---
title: 理解js的调用堆栈和event-loop
date: 2018-08-24 19:02:16
tags:
  - JavaScript
  - 原理
categories:
  - 前端
keywords: 调用堆栈,时间循环,宏任务,微任务
---

# 先上一张图

![](https://raw.githubusercontent.com/LouisGo/image-hosting/master/event-loop.png)

```javascript
function multiply(x, y) {
  return x * y;
}
function printSquare(x) {
  var s = multiply(x, x);
  console.log(s);
}
printSquare(5);

```

![](https://raw.githubusercontent.com/LouisGo/image-hosting/master/call-stack.png)

# 宏任务和微任务

macro-task(宏任务)：包括整体代码script，setTimeout，setInterval, setImmediate, I/O, UI rendering

micro-task(微任务)：Promise，process.nextTick, Object.observe, MutationObserver


**事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。听起来有点绕，我们用文章最开始的一段代码说明**


```javascript
setTimeout(function() {
    console.log('setTimeout');
})

new Promise(function(resolve) {
    console.log('promise');
}).then(function() {
    console.log('then');
})

console.log('console');
```

1. 这段代码作为宏任务，进入主线程。
2. 先遇到setTimeout，那么将其回调函数注册后分发到宏任务Event Queue。(注册过程与上同，下文不再描述)
3. 接下来遇到了Promise，new Promise立即执行，then函数分发到微任务Event Queue。
4. 遇到console.log()，立即执行。
5. 好啦，整体代码script作为第一个宏任务执行结束，看看有哪些微任务？我们发现了then在微任务Event Queue里面，执行。
6. ok，第一轮事件循环结束了，我们开始第二轮循环，当然要从宏任务Event 
7. Queue开始。我们发现了宏任务Event 
8. Queue中setTimeout对应的回调函数，立即执行。
9. 结束。

# 其他

- 函数的直接调用实际上是call的一种语法糖
```javascript
function say(word) { console.log(world); } 
say("Hello world"); // == say.call(window, "Hello world"); 
```