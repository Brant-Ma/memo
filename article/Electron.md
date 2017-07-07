## Electron
> electron 让写 desktop application 变的很有趣

### 基本组成

Electron 是一个专注于桌面应用的运行时。它既是一个可以访问操作系统的 Node.js 变体，又是一个可以呈现 web 页面的小型 Chromium 浏览器。最重要的是，它实现了 JavaScript 对桌面应用的占领。

Electron 主要包含了 Main 进程和 Renderer 进程：

- Main 进程负责创建页面。它通过创建 web 页面来呈现 GUI
- Renderer 进程负责呈现页面。它使用 Chromium 及其多进程架构来呈现 web 页面

具体的，Main 进程通过创建 `BrowserWindow` 实例来创建 web 页面，每个实例在它自己的 Renderer 进程中运行一个 web 页面。实例被销毁时，对应的 Renderer 进程也将终止。Main 负责管理所有的 web 页面及其对应的 Renderer 进程，而每个独立的 Renderer 进程只关心运行其中的 web 页面。

### 进程通信

为了防止资源泄漏，web 页面不能直接操作本地的 GUI 资源。Renderer 进程需要请求 Main 进程来执行这些操作。具体而言，可以通过 `ipcRenderer` 模块、`ipcMain` 模块和 `remote` 模块来进行 Main 进程和 Renderer 进程之间的通信。

web 页面之间的分享数据有两种方式。一种是使用 HTML5 API，比如 Storage API 和 IndexedDB；另一种是使用 IPC 系统，它能将对象作为全局变量存储在 Main 进程中，然后 Renderer 进程就可以通过 `electron` 模块的 `remote` 属性来访问这些对象：

	// main process
	global.sharedObject = { version: '1.0.0' }
	
	// in some page
	let version = require('electron').remote.getGlobal('sharedObject').version

### 书写应用

Electron 应用的基本结构是三个文件：

- `package.json`：配置文件的 `main` 指定了应用的启动脚本，它将启动 main 进程
- `main.js`：负责创建 window，处理系统事件
- `index.html`：作为要呈现的 web 页面

基本的 `package.json` 文件如下：

	{
	  "name": "demo",
	  "version": "1.0.0",
	  "main": "main.js"
	} 

基本的 `main.js` 文件如下：

	const {app, BrowserWindow} = require('electron')
	const path = require('path')
	const url = require('url')
	
	// 通过保持对 window object 的引用，防止因垃圾回收而导致页面关闭
	let win
	
	function createWindow() {
	  // 创建 window
	  win = new BrowserWindow({width: 800, height: 600})
	  
	  // 加载 index.html
	  win.loadURL(url.format({
	    pathname: path.join(__dirname, 'index.html'),
	    protocol: 'file:',
	    slashes: true
	  }))
	  
	  // 打开 devtools
	  win.webContents.openDevTools()
	  
	  // 关闭 window 时解除引用，多窗口 app 则存储在数组中再操作
	  win.on('closed', () => { win = null })
	}
	
	// app 在完成初始化后，开始创建 window
	app.on('ready', createWindow)
	
	// app 退出，比如键入 Meta + Q
	app.on('window-all-closed', () => {
	  if (process.platform !== 'darwin') app.quit()
	})
	
	// app 启动，比如点击 dock icon
	app.on('activate', () => {
	  if (win === null) createWindow()
	})
 
基本的 `index.html` 文件如下：

	<!DOCTYPE html>
	<html>
	  <head>
	    <meta charset="UTF-8">
	    <title>demo</title>
	  </head>
	  <body>
	    <script>
	      let {node, chrome, electron} = process.versions
	      let str = `node ${node}, chrome ${chrome}, electron ${electron}`
	      document.write(str)
	    </script>
	  </body>
	</html>

### 测试应用

三个核心文件创建好后，就可以运行 app 来进行本地测试了。这里需要一个包含已经预编译的 Electron 的名叫 `electron` 的 npm 模块。 

	// 全局安装 electron
	$ npm install -g electron
	
	// 本地运行 application
	$ electron .

### 发布应用

如果应用运行正常就可以打包发布，然后获得一个可运行的最终形态了。官方提供的方法比较繁琐，最便捷的方法是使用第三方工具，比如 [electron-packager](https://github.com/electron-userland/electron-packager)，之后就可以按照流程进行操作。

首先，确定目录结构没有问题：

	electron-demo
	├── package.json
	├── main.js
	├── [... other files ...]
	└── index.html

然后，确定以下事项没有问题：

- `electron-packager` 已经全局安装
- `package.json` 的 `productName` 或 `name` 已经设置
- `electron` 的某个确切版本已经本地安装，且保存在了 `package.json` 的 `devDependencies`
- `npm install` 已经在本地运行过

最后，在本地目录执行 `$ electron-packager .` 命令。

大功告成！优雅的桌面应用已经默认保存在了本地路径下的 `[app name]-darwin-x64` 路径下。


### 参考
1. [Quick Start | Electron](https://electron.atom.io/docs/tutorial/quick-start/)
2. [Electron FAQ | Electron](https://electron.atom.io/docs/faq/)
3. [Application Distribution | Electron](https://electron.atom.io/docs/tutorial/application-distribution/)
4. [electron-userland/electron-packager | GitHub](https://github.com/electron-userland/electron-packager)
