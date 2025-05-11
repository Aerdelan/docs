# Electron

## 一. 配置

官方文档：https://www.electronjs.org/zh/

配置Electron的main.js文件：

```js
const { app, BrowserWindow } = require('electron')
const path = require('path')

const createWindow = () => {
    // Create the browser window.
    const mainWindow = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
            preload: path.join(__dirname, 'preload.js')
        }
    })
    mainWindow.loadFile(path.join(__dirname, './dist/index.html'))
    //打包地址时
    mainWindow.loadURL('http://localhost:8000');
}

// 这段程序将会在 Electron 结束初始化
// 和创建浏览器窗口的时候调用
// 部分 API 在 ready 事件触发后才能使用。
app.whenReady().then(() => {
    createWindow()

    app.on('activate', () => {
        if (BrowserWindow.getAllWindows().length === 0) createWindow()
    })
})

// 除了 macOS 外，当所有窗口都被关闭的时候退出程序。
app.on('window-all-closed', () => {
    if (process.platform !== 'darwin') app.quit()
})
```



使用electron-builder进行项目打包：

```bash
npm install electron-builder --save-dev  //安装electron-builder打包工具

```

```json
{
  "name": "your-app",
  "version": "1.0.0", // 版本号
  "main": "main.js",  // 指定 Electron 主进程的入口文件
  "scripts": {
    "start": "umi dev",  // 启动 Umi 开发服务器
    "build": "umi build",  // 构建 Umi 项目
    "electron": "electron .",  // 启动 Electron 应用
    "pack": "electron-builder --dir",  // 打包应用到目标目录
    "dist": "electron-builder"  // 创建可分发的安装包
  },
  "dependencies": {
    "react": "^latest_version",
    "react-dom": "^latest_version",
    "umi": "^latest_version",
    "electron": "^latest_version"
  },
  "devDependencies": {
    "electron-builder": "^latest_version",
    "umi": "^latest_version"
  },
  "build": {
    "appId": "com.example.app",
    "mac": {
      "target": "dmg",
      "icon": "icon.icns"
    },
    "win": {
      "target": "nsis",
      "icon": "icon.ico"
    }
  }
}

```

*需要注意：使用electron-builder首次打包项目时需要挂全局vpn，或者手动下载缺失文件至c盘user/AppData/Local/electron-builder/Cache.

使用electron-forge进行项目打包：

```bash
npm install --save-dev @electron-forge/cli //安装electron-forge打包工具
npx electron-forge import

```

```json
{
  "name": "your-app",
  "version": "1.0.0",
  "main": "main.js",
  "scripts": {
    "start": "npm run dev",  // 启动 Umi 开发服务器
    "build": "umi build",  // 构建 Umi 项目
    "electron": "electron .",  // 启动 Electron 应用
    "package": "electron-forge package",  // 打包应用
    "make": "electron-forge make"  // 创建安装包
  },
  "devDependencies": {
    "electron": "^latest_version",
    "@electron-forge/cli": "^latest_version",
    "@electron-forge/maker-deb": "^latest_version",
    "@electron-forge/maker-rpm": "^latest_version"
  },
  "electron-forge": {
    "packagerConfig": {
      "icon": "path/to/icon"
    },
    "makers": [
      {
        "name": "@electron-forge/maker-squirrel",
        "config": {
          "name": "your-app"
        }
      }
    ]
  }
}

```





## 二.mac与win配置



## 三.项目所遇问题

####   1.打包后的系统需要调起本机默认浏览器

```js
// main.js中配置
 mainWindow = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
            preload: path.join(__dirname, 'preload.js'),
            nodeIntegration: false, // 必须为 false 以保证安全
            contextIsolation: true  // 必须为 true 以启用上下文隔离
        },
    });
// 此处需要配置preload文件
```

```js
// preload.js
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld(
    'electron',
    {
        openUrl: (url) => ipcRenderer.send('open-url', url),
    }
)
// 挂载electron全局方法通过contextBridge暴露出来openUrl，渲染程序通过 window.electron.openUrl()调用
// window.electron.openUrl()方法调起main.js主程序内对应方法
```

```js
// main.js
const {  shell } = require('electron');
function getUrl(url) {
    shell.openExternal(url);  // 正确的传递 url
}
```

#### 2.electron隐藏顶部工具栏

main.js添加以下代码：

```js
const {  Menu  } = require('electron');
// Menu创建工具栏和修改工具栏，传递null可隐藏工具栏
Menu.setApplicationMenu(null)

```

