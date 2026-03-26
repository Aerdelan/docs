## 配置 main.go

```go

func main() {
	app := cli.NewApp() //创建一个实例
	app.Name = "staffsubsidybackend" //实例名称
	app.Version = VERSION //版本号
	app.Usage = "Staffsubsidybackend API service" //备注
	app.Commands = []*cli.Command{
		cmd.StartCmd(), //加载cmd模块下StartCmd配置启动指令方法
		cmd.StopCmd(),
		cmd.VersionCmd(VERSION),
	}
	err := app.Run(os.Args)
	if err != nil {
		panic(err)
	}
}

```

StartCmd 方法中需要配置
启动命令名称，启动命令备注以及携带参数

```go
        Name:  "start", //执行指令名称
		Usage: "Start server", //指令备注
		Flags: []cli.Flag{ //指令携带参数
			&cli.StringFlag{
				Name:        "workdir",
				Aliases:     []string{"d"},
				Usage:       "Working directory",
				DefaultText: "configs",
				Value:       "configs",
			},
			&cli.StringFlag{
				Name:        "config",
				Aliases:     []string{"c"},
				Usage:       "Runtime configuration files or directory (relative to workdir, multiple separated by commas)",
				DefaultText: "dev",
				Value:       "dev",
			},
			&cli.StringFlag{
				Name:    "static",
				Aliases: []string{"s"},
				Usage:   "Static files directory",
			},
			&cli.BoolFlag{
				Name:  "daemon",
				Usage: "Run as a daemon",
			},
		},
```

Action 方法为启动指令方法

```go
          Action: func(c *cli.Context) error {
			workDir := c.String("workdir") //获取命令行携带参数
			staticDir := c.String("static")
			configs := c.String("config")

			if c.Bool("daemon") { //判断是否为安全模式，daemon也是携带参数
				bin, err := filepath.Abs(os.Args[0])
				if err != nil {
					fmt.Printf("failed to get absolute path for command: %s \n", err.Error())
					return err
				}

				args := []string{"start"} //创建一个新的启动指令
				args = append(args, "-d", workDir) //将上面携带参数赋值给当前启动指令
				args = append(args, "-c", configs)
				args = append(args, "-s", staticDir)
				fmt.Printf("execute command: %s %s \n", bin, strings.Join(args, " "))
				command := exec.Command(bin, args...) //启动一个新的进程

				// Redirect stdout and stderr to log file
				stdLogFile := fmt.Sprintf("%s.log", c.App.Name) //生成日志文件
				file, err := os.OpenFile(stdLogFile, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666) //打开该文件
				if err != nil {
					fmt.Printf("failed to open log file: %s \n", err.Error())
					return err
				}
				defer file.Close()

				command.Stdout = file //输出进程内容
				command.Stderr = file //输出进程错误

				err = command.Start()
				if err != nil {
					fmt.Printf("failed to start daemon thread: %s \n", err.Error())
					return err
				}

				// Don't wait for the command to finish
				// The main process will exit, allowing the daemon to run independently
				fmt.Printf("Service %s daemon thread started successfully\n", config.C.General.AppName)

				pid := command.Process.Pid
				_ = os.WriteFile(fmt.Sprintf("%s.lock", c.App.Name), []byte(fmt.Sprintf("%d", pid)), 0666)
				fmt.Printf("service %s daemon thread started with pid %d \n", config.C.General.AppName, pid)
				os.Exit(0) //向某个锁定文件输出进程pid
			}

			err := bootstrap.Run(context.Background(), bootstrap.RunConfig{
				WorkDir:   workDir,
				Configs:   configs,
				StaticDir: staticDir,
			}) //指定Run方法
			if err != nil {
				panic(err)
			}
			return nil
		},
```

bootstrap.Run 主要是进行配置项的写入以及启动 http 模块

```go
func Run(ctx context.Context, runCfg RunConfig) error {
	defer func() {
		if err := zap.L().Sync(); err != nil {
			fmt.Printf("failed to sync zap logger: %s \n", err.Error())
		}
	}() // 延时函数，在方法执行完之后执行，输出日志文件

	// Load configuration.
	workDir := runCfg.WorkDir  //写入配置项
	staticDir := runCfg.StaticDir
	config.MustLoad(workDir, strings.Split(runCfg.Configs, ",")...)
	config.C.General.WorkDir = workDir
	config.C.Middleware.Static.Dir = staticDir
	config.C.Print()
	config.C.PreLoad()

	// Initialize logger.
	cleanLoggerFn, err := logging.InitWithConfig(ctx, &config.C.Logger, initLoggerHook) //输出log文件
	if err != nil {
		return err
	}
	ctx = logging.NewTag(ctx, logging.TagKeyMain)

	logging.Context(ctx).Info("starting service ...",
		zap.String("version", config.C.General.Version),
		zap.Int("pid", os.Getpid()),
		zap.String("workdir", workDir),
		zap.String("config", runCfg.Configs),
		zap.String("static", staticDir),
	) //记录进程启动信息

	// Start pprof server.
	if addr := config.C.General.PprofAddr; addr != "" {
		logging.Context(ctx).Info("pprof server is listening on " + addr)
		go func() {
			err := http.ListenAndServe(addr, nil)
			if err != nil {
				logging.Context(ctx).Error("failed to listen pprof server", zap.Error(err))
			}
		}()
	}

	// Build injector.
	injector, cleanInjectorFn, err := wirex.BuildInjector(ctx) //读取wire_gen中BuildInjector方法读取各种依赖注入
	if err != nil {
		return err
	}

	if err := injector.M.Init(ctx); err != nil {
		return err
	} //完成各模块初始化工作

	// Initialize global prometheus metrics.
	prom.Init()

	return util.Run(ctx, func(ctx context.Context) (func(), error) {
		cleanHTTPServerFn, err := startHTTPServer(ctx, injector) //启动http服务
		if err != nil {
			return cleanInjectorFn, err
		}

		return func() {  // 关闭http服务，清楚依赖注入和日志
			if err := injector.M.Release(ctx); err != nil {
				logging.Context(ctx).Error("failed to release injector", zap.Error(err))
			}

			if cleanHTTPServerFn != nil {
				cleanHTTPServerFn()
			}
			if cleanInjectorFn != nil {
				cleanInjectorFn()
			}
			if cleanLoggerFn != nil {
				cleanLoggerFn()
			}
		}, nil
	})
}
```

http 模块：负责项目相应客户端接口请求，配置请求中间件

定义 startHTTPServer
启动方法内配置 http 模式

```go
if config.C.IsDebug() {
		gin.SetMode(gin.DebugMode) //开发
	} else {
		gin.SetMode(gin.ReleaseMode) //生产
	}
```

调用框架的 gin.New()方法启动一个实例

```go
	e := gin.New()
	e.GET("/health", func(c *gin.Context) {
		util.ResOK(c)
	})
	e.Use(middleware.RecoveryWithConfig(middleware.RecoveryConfig{
		Skip: config.C.Middleware.Recovery.Skip,
	}))
	e.NoMethod(func(c *gin.Context) {
		util.ResError(c, errors.MethodNotAllowed("", "Method Not Allowed"))
	})
	e.NoRoute(func(c *gin.Context) {
		util.ResError(c, errors.NotFound("", "Not Found"))
	})


	// ResOK函数 返回成功状态
func ResOK(c *gin.Context) {
	ResJSON(c, http.StatusOK, ResponseResult{
		Success: true,
	})
}
    // Use
```

其中 Get 创建了一个/health 接口，可供检测系统运行状态
Use 用于捕获 http 请求产生的 panic 防止系统崩溃
NoMethod 用于处理指定格式的请求 如果只定义了 Get 但是前端传递了 Post 请求则会在这里被拦截并且返回 405
NoRoute 用于处理请求 url 与路由定义不匹配的问题，也即是 404

```go
allowedPrefixes := injector.M.RouterPrefixes()
```

以上操作执行完毕后使用 RouterPrefixes 方法获取所有路由入口文件 allowedPrefixes，

```go
	if err := useHTTPMiddlewares(ctx, e, injector, allowedPrefixes); err != nil {
		return nil, err
	}

	//useHTTPMiddlewares 统一调用中间件操作

   func useHTTPMiddlewares(_ context.Context, e *gin.Engine, injector *wirex.Injector, allowedPrefixes []string) error {
   	e.Use(middleware.AuthWithConfig(middleware.AuthConfig{
   		AllowedPathPrefixes: allowedPrefixes,
   		SkippedPathPrefixes: config.C.Middleware.Auth.SkippedPathPrefixes,
   		ParseUserID:         injector.M.RBAC.LoginAPI.LoginBIZ.ParseUserID,
   		RootID:              config.C.General.Root.ID,
   	}))
   		if config.C.Util.Prometheus.Enable {
   		e.Use(prom.GinMiddleware)
   	}

   	return nil

   }
```

然后是中间件注册调用，useHTTPMiddlewares 为需要执行的中间件操作,比如接口认证中间件

在 AuthWithConfig 中有一个判断，

```go
        if !AllowedPathPrefixes(c, config.AllowedPathPrefixes...) ||
			SkippedPathPrefixes(c, config.SkippedPathPrefixes...) ||
			(config.Skipper != nil && config.Skipper(c)) {
			c.Next()
			return
		}
	   userID, err := config.ParseUserID(c)
		if err != nil {
			util.ResError(c, err)
			return
		}
```

AllowedPathPrefixes 是哪些接口需要进行验证，而 SkippedPathPrefixes 则为哪些接口不需要进行验证，接口验证通过后开始解析用户 id，成功之后将用户信息更新至上下文

CORS 配置

```go
package middleware

import (
	"time"

	"github.com/gin-contrib/cors"
	"github.com/gin-gonic/gin"
)

type CORSConfig struct {
	Enable          bool //是否启用
	AllowAllOrigins bool //是否允许所有来源
	// AllowOrigins is a list of origins a cross-domain request can be executed from.
	// If the special "*" value is present in the list, all origins will be allowed.
	// Default value is []
	AllowOrigins []string //指定可跨域的域名列表
	// AllowMethods is a list of methods the client is allowed to use with
	// cross-domain requests. Default value is simple methods (GET, POST, PUT, PATCH, DELETE, HEAD, and OPTIONS)
	AllowMethods []string //指定可跨域的请求方式
	// AllowHeaders is list of non simple headers the client is allowed to use with
	// cross-domain requests.
	AllowHeaders []string //允许前端可选的请求头
	// AllowCredentials indicates whether the request can include user credentials like
	// cookies, HTTP authentication or client side SSL certificates.
	AllowCredentials bool //是否允许token等请求信息
	// ExposeHeaders indicates which headers are safe to expose to the API of a CORS
	// API specification
	ExposeHeaders []string //允许前端访问的响应头
	// MaxAge indicates how long (with second-precision) the results of a preflight request
	// can be cached
	MaxAge int //浏览器缓存检测时间
	// Allows to add origins like http://some-domain/*, https://api.* or http://some.*.subdomain.com
	AllowWildcard bool //是否允许通配符
	// Allows usage of popular browser extensions schemas
	AllowBrowserExtensions bool //是否允许浏览器插件发起的请求（例如 Chrome 扩展程序）
	// Allows usage of WebSocket protocol
	AllowWebSockets bool //是否允许跨域的 WebSocket 连接
	// Allows usage of file:// schema (dangerous!) use it only when you 100% sure it's needed
	AllowFiles bool //是否允许 file:// 协议来源
}

var DefaultCORSConfig = CORSConfig{
	AllowOrigins: []string{"*"},
	AllowMethods: []string{"GET", "POST", "PUT", "PATCH", "DELETE", "HEAD", "OPTIONS"},
} //未使用

func CORSWithConfig(cfg CORSConfig) gin.HandlerFunc {
	if !cfg.Enable {
		return Empty()
	}

	return cors.New(cors.Config{
		AllowAllOrigins:        cfg.AllowAllOrigins,
		AllowOrigins:           cfg.AllowOrigins,
		AllowMethods:           cfg.AllowMethods,
		AllowHeaders:           cfg.AllowHeaders,
		AllowCredentials:       cfg.AllowCredentials,
		ExposeHeaders:          cfg.ExposeHeaders,
		MaxAge:                 time.Second * time.Duration(cfg.MaxAge),
		AllowWildcard:          cfg.AllowWildcard,
		AllowBrowserExtensions: cfg.AllowBrowserExtensions,
		AllowWebSockets:        cfg.AllowWebSockets,
		AllowFiles:             cfg.AllowFiles,
	})
}

```
