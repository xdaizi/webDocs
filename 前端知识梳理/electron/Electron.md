# Electron
## 一.简介
可以使用js，html，css开发桌面端产品
### 1.组成
- Chromium: 通过web技术编写UI，Chrom的开源浏览器
- node.js：提供底层能力，比如文件的读写，集成C++等
- 内置的Native API：跨平台，原生能力
	- 统一的原生界面：窗口，托盘
	- 系统能力：系统通知
	- 应用的基础能力：软件更新，崩溃监控
	
### 2.跟普通web的区别
- 可以调用底层能力以及系统的能力
- 不用考虑兼容性问题，很多新语法都能使用、
	- async awit
	- img标签的lazy-loading
## 二.通信
### 1.主进程，子进程通信 ipc
#### 主进程
``` js
const { ipcMain } = require('electron')
// 监听
ipcMain.on('asynchronous-message', (event, arg) => {
	console.log(arg) // prints "ping"
	// 发送，event：子窗口的句柄
	event.reply('asynchronous-reply', 'pong')
})
```
#### 子进程
``` js
//在渲染器进程 (网页) 中。
const { ipcRenderer } = require('electron')
// 监听
ipcRenderer.on('asynchronous-reply', (event, arg) => {
	console.log(arg) // prints "pong"
})
// 发送
ipcRenderer.send('asynchronous-message', 'ping')

```

#### invoke,handle
``` js
// Main process
ipcMain.handle('my-invokable-ipc', async (event, ...args) => {
	const result = await somePromise(...args)
	return result
})
// Renderer process
async () => {
	const result = await ipcRenderer.invoke('my-invokable-ipc', arg1, arg2)
	// ...
}
```

### 2.子进程之间通信

####  通过主进程转发
#### sendTo
ipcRenderer.sendTo(webContentsId, channel, ...args)
-   `webContentsId` Number 进程id
-   `channel` String 事件名
-   `...args` any[] 参数



### 3.remote
[remote](https://www.electronjs.org/docs/api/remote#remote)

不建议使用
- 底层时通过调用ipc的同步通信方法
- Electron 确保只要渲染进程中的远程对象一直存在（换句话说，没有被回收），主进程中的相应对象就不会被释放。 当远程对象被垃圾回收后，主进程中的相应对象将被解除引用。容易造成内存泄漏










- 远程控制
- node调用exe文件，截图
- 快捷键
- 升级，打包
- log


