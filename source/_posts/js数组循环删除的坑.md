---
title: js数组循环删除的坑
date: 2018-08-01 21:11:02
tags:
  - JavaScript
  - 数组
categories:
  - 前端
keywords: 数组,循环,删除
---

改项目上一个遗留的 bug 时，“不小心”掉进了数组循环删除的坑，之前的开发中其实有解决过这类问题，为加深印象，特意再手打一次。

<!--more-->

# 实例

假设有如下一个数组：

```javascript
let arr = [
  {
    name: 'yes',
    id: 1
  },
  {
    name: 'yes',
    id: 2
  },
  {
    name: 'no',
    id: 3
  },
  {
    name: 'no',
    id: 4
  },
  {
    name: 'yes',
    id: 5
  },
  {
    name: 'no',
    id: 6
  }
]
```

现在需要循环删除其中`name`为`no`的对象，错误示范如下：

```javascript
for (let index = 0; index < arr.length; index++) {
  let item = arr[index]
  if (item.name === 'no') {
    arr.splice(index, 1)
  }
}

// arr =>

/*
[
    {
        name: "yes",
        id: 1
    },
    {
        name: "yes",
        id: 2
    },
    {
        name: "no",
        id: 4
    },
    {
        name: "yes",
        id: 5
    }
]
*/
```

错误分析：

1. 当循环到 index 为 2，id 为 3 的符合条件的对象时，从 length 为 6 的`arr`数组中删除了该对象，则`arr`的 length - 1
2. 此时原先 index 为 3 的应该匹配的对象因为数组 length 变化跑到了 index 为 2 的位置，而循环跳过了该对象，接着去判断此时 index 为 3，id 为 5 的对象，不匹配，继续进行循环
3. index 为 4，id 为 6 的对象被删除
4. 循环结束

可以看出，主要的问题出在删除操作**改变了原数组**

# 解决方案

1. 逆向循环不考虑 length 减少带来的 index 错乱

```javascript
for (let index = arr.length - 1; index >= 0; index--) {
  let item = arr[index]
  if (item.name === 'no') {
    arr.splice(index, 1)
  }
}
```

2. 把数组看成队列巧妙处理

```javascript
for (let index = 0; index < arr.length; index++) {
  if (arr[0].name !== 'no') {
    // 永远只判断数组第一位，满足条件则push到arr尾部，不满足直接删除首位
    arr.push(arr[0])
  }
  arr.shift()
}
```

3. 重置 index 值，获得正确的迭代顺序

```javascript
for (let index = 0; index < arr.length; index++) {
  if (arr[index].name === 'no') {
    arr.splice(index, 1)
    index-- // 手动重置
  }
}
```
