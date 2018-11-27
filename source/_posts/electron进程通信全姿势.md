---
title: electron进程通信全姿势
date: 2018-10-14 13:11:17
tags:
  - JavaScript
  - Electron
categories:
  - 前端
keywords: Electron,前端,客户端
---

# 背景

最近的项目有用到 electron，刚开始对 main 和 renderer 进程的概念还是有点懵逼的，渐渐熟悉之后准备写篇 blog 加深一下印象。

# 进程分类

## 主进程（main process）

一个 Electron 应用**只有一个主进程**

当我们`electron .`命令后， Electron 会运行当前目录下的`package.json`文件中`main`字段指定的文件。而运行该文件的进程就是主进程。

运行在主进程中的脚本可以通过创建一个窗口，并传入 URL，让这个窗口加载一个网页来展示图形界面。

与创建 GUI 相关的接口**只应该由主进程来调用**

## 渲染进程（renderer process）

在 Electron 里的每个页面都有它自己的进程，叫作渲染进程。主进程通过实例化 `BrowserWindow`，每个`BrowserWindow`实例都在它自己的渲染进程内返回一个 web 页面。当`BrowserWindow`实例销毁时，相应的渲染进程也会终止。

渲染进程由主进程进行管理。每个渲染进程都是相互独立的，控制页面渲染、脚本执行、事件处理等，它们只关心自己所运行的 web 页面。

每个渲染进程是多线程的，可以有以下几类子线程：

- GUI 线程
- JS 引擎线程
- 事件触发线程
- 定时器线程
- 网络请求线程

<!--more-->

# 主进程和渲染进程之间的通信

## 利用`ipcMain`和`ipcRenderer`

以下例子为官网示例：

```javascript
// main process
const { ipcMain } = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.sender.send('asynchronous-reply', 'pong')
})

ipcMain.on('synchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.returnValue = 'pong'
})
```

```javascript
// In renderer process (web page).
const { ipcRenderer } = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // prints "pong"
})
ipcRenderer.send('asynchronous-message', 'ping')
```

渲染进程可以通过`ipcRenderer`模块的`send`方法向主进程发送消息。在主进程中，通过`ipcMain`模块设置监听`asynchronous-message`和`synchronous-message`两个事件，当渲染进程发送时就可以针对不同的事件进行处理。

主进程监听事件的回调函数中，会传递`event`对象及`arg`对象。arg 对象中保存渲染进程传递过来的参数。通过`event.sender`对象，主进程可以向渲染进程发送消息。如果主进程执行的是同步方法，还可以通过设置`event.returnValue`来返回信息

上面说了渲染进程如何向主进程发送消息，但**主进程也可以主动向渲染进程发送消息**

在主进程中，我们会创建一个`BrowserWindow`对象，这个对象有`webContents`属性。`webContets`提供了`send`方法来实现向渲染进程发送消息

以下例子为官网示例：

```javascript
// In the main process.
const { app, BrowserWindow } = require('electron')
let win = null

app.on('ready', () => {
  win = new BrowserWindow({ width: 800, height: 600 })
  win.loadURL(`file://${__dirname}/index.html`)
  win.webContents.on('did-finish-load', () => {
    win.webContents.send('ping', 'whoooooooh!')
  })
})
```

```html
<!-- index.html -->
<html>
  <body>
    <script>
      require('electron').ipcRenderer.on('ping', (event, message) => {
        console.log(message) // Prints 'whoooooooh!'
      })
    </script>
  </body>
  </html>
```

`webContents.on`监听的是**已经定义好的事件**，如上面的`did-finish-load`。要监听自定义的事件还是通过`ipcMain`和`ipcRenderer`。

渲染进程的监听事件回调函数中，也可以通过`event.sender`来向主进程发送消息。这个对象只是`ipcRenderer`的引用`(event.sender === ipcRenderer)`。因此，`event.sender`发送的消息在主进程中还是需要通过`ipcMain.on`方法来监听，而不是通过`webContents.on`方法

## 利用`electron.remote`模块

在渲染进程中，可以通过`const {remote} = require('electron')`获取到`remote`对象，通过`remote`对象可以让渲染进程访问/使用主进程的模块。

比如，通过`remote`在渲染进程中新建一个窗口：

```javascript
const { BrowserWindow } = require('electron').remote
let win = new BrowserWindow({ width: 800, height: 600 })
win.loadURL('https://github.com')
```

也可以通过`remote`对象访问到`app`对象。这样就可以访问到在主进程中挂载到`electron.app`对象上的方法

```javascript
// main process
const { app } = require('electron')
const utils = require('./utils')
app.utils = utils
```

```javascript
// renderer process
const { remote } = require('electron');
function() {
  // remote.app.utils 对象与上述文件中的 utils 对象是一样的。
  remote.app.utils.test();
}
```
