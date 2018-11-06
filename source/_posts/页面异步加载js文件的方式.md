---
title: 页面异步加载js文件的方式
date: 2018-08-08 22:01:56
tags:
  - JavaScript
  - 优化
categories:
  - 前端
keywords: 页面加载
---

# 当浏览器碰到script标签时

1. `<script src="script.js"></script>` 没有`defer`或者`async`，浏览器会立即加载并执行js文件，立即指的是该`script`标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。
2. `<script src="test.js" async></script>` 有`async`，加载和渲染后续文档元素的过程将和`script.js`的加载与执行并行进行（异步）
3. `<script src="test.js" defer></script>` 有`defer`，加载后续文档元素的过程将和`script.js`的加载并行进行（异步），**但是`script.js`的执行要在所有元素解析完成之后，`DOMContentLoaded`事件触发之前完成。**

# 注意的几点

1. 渲染引擎遇到`script`标签会停下来，等到执行完脚本，继续向下渲染
2. `defer`是“渲染完再执行”，`async`是“下载完就执行”，`defer`如果有多个脚本，**会按照在页面中出现的顺序加载**，多个`async`脚本不能保证加载顺序
3. `async`适合不依赖任何脚本或者不被任何脚本依赖的情况，比如`Google Analytics`
4. 加载es6模块的时候设置`type=module`，异步加载不会造成阻塞浏览器，页面渲染完再执行，可以同时加上`async`属性，异步执行脚本（利用顶层的`this`等于`undefined`这个语法点，可以侦测当前代码是否在es6模块之中）

# 图解

![](https://raw.githubusercontent.com/LouisGo/image-hosting/master/page-loading-timing.png)