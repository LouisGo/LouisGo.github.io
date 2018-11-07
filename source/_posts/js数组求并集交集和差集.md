---
title: js数组求并集交集和差集
date: 2018-10-13 15:21:10
tags:
  - JavaScript
  - 数组
categories:
  - 前端
keywords: 数组,交集,并集,差集
---

项目中有遇到一些涉及数学集的运算，在网上寻找解决方法的时候顺便总结一下。

假设有数组`a = [1, 2, 3]`和`b = [2, 4, 5]`

# ES7 includes

```javascript
// 并集
let union = a.concat(b.filter(n => !a.includes(n))) // [1, 2, 3, 4, 5]

// 交集
let inter = a.filter(n => b.includes(n)) // [2]

// 差集
let diff = a.concat(b).filter(n => !a.inclueds(n) || !b.inclueds(n)) // [1, 3, 4, 5]
```

<!--more-->

# ES6 Array.from Set

```javascript
let aSet = new Set(a)
let bSet = new Set(b)

// 并集
let union = Array.from(new Set(a.concat(b))) // [1, 2, 3, 4, 5]

// 交集
let inter = Array.from(new Set(a.filter(n => bSet.has(n)))) // [2]

// 差集
let diff = Array.from(
  new Set(a.concat(b).filter(n => !aSet.has(n) || !bSet.has(n)))
) // [1, 3, 4, 5]
```

# ES5 filter indexOf

```javascript
// 并集
let union = a.concat(
  b.filter(function(n) {
    return a.indexOf(n) === -1
  })
) // [1, 2, 3, 4, 5]

// 交集
let inter = a.fitler(function (n) {
  return b.indexOf(n) > -1
}) // [2]

// 差集
let diff = a.filter(function (n) {
  return b.indexOf(n) === -1
}).concat(b.filter(function (n) {
  return a.index(n) === -1
})) // [1, 3, 4, 5]
```

ps：有NaN的情况则需另外判断`a.some(function(n){return isNaN(n)})`
