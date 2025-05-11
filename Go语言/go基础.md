## Gd

官方文档：https://go.dev/doc/tutorial/getting-started#install

下载安装后 go 会自动配置环境变量

# makefile 构建工具

make build # 编译
make run # 运行
make test # 测试
make clean # 清理

# 类型命名

参数定义类型名应在参数后面

int 为类型

```js
package main

import "fmt"

func swap(x, y,z string) (string, string,string) {
	return x, y,z
}

func main() {
	a, b,c := swap("hello","my", "world",)
	fmt.Println(a, b,c)
}
```

以上函数中 main 为主函数，调用 swap，如果 swap 中 return 空则会返回所有已定义返回值

# 项目结构与数库连接

go 测试项目结构：

my-gin-app-1/
├── controller/
│ └── hello.go # 定义路由处理函数，例如 HelloHandler 和 GetUsers
├── model/
│ └── db.go # 数据库初始化和连接管理
├── router/
│ └── router.go # 路由配置（未提供，但通常用于定义路由）
├── main.go # 项目入口，初始化数据库和路由
├── go.mod # Go 模块定义文件，包含模块名和依赖
├── go.sum # Go 模块依赖的校验文件
└── Makefile # 定义常用命令（如 run、build、clean）

项目安装 mysql 驱动：go get -u github.com/go-sql-driver/mysql

启动 mysql：net start MySQL80 --本机使用 8.0 版本

<!-- 配置数据库连接服务 -->

```go
package model

import (
	"database/sql"
	"fmt"
	"log"

	_ "github.com/go-sql-driver/mysql"
)

var DB *sql.DB

func InitDB() {
	// 配置数据库连接信息
	dsn := "root:123456@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"

	var err error
	DB, err = sql.Open("mysql", dsn)
	if err != nil {
		log.Fatalf("Failed to connect to database: %v", err)
	}

	// 测试数据库连接
	if err := DB.Ping(); err != nil {
		log.Fatalf("Failed to ping database: %v", err)
	}

	fmt.Println("成功")
}
```
