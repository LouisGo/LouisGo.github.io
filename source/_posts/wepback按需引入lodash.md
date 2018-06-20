---
title: wepback按需引入lodash
date: 2018-05-28 17:21:15
tags: 
- webpack
- 工具
- lodash
categories: 
- 前端
keywords: webpack,工具,lodash,按需引入
---

# 前言

在前端项目的开发过程中，经常需要引入 [lodash](https://github.com/lodash/lodash) 库，这样就不用自己去费心费力定义一些（可能会坑）的工具函数，使用久经考验的`lodash`，可以帮助我们更好更快的开发。

# 简单的使用方式

通常情况下，我们直接：

```javascript
<script src="lodash/js"></script>
```

然后将`lodash`挂载到`window`：

```javascript
window._ = lodash
```

这种方式虽然简单粗暴，但是一点也不优雅，因为会污染全局命名空间。所以我们使用`webpack`，模块化是解决此类问题的最佳思路。

<!--more-->

# 通过npm引入lodash模块

```
npm i -S lodash

or 

npm i --save lodash
```
然后我们就可以在各个文件中去动态引入`lodash`，你可以引入全部：

```javascript
import _ from 'lodash'
_.debouce(...)
_.chunk(...)
```

或者按需引入某些用到的模块，这减少了打包后的体积（也增加了代码数量...）：

```javascript
import debounce from 'lodash/debounce'
import throttle from 'lodash/throttle'

debounce(...)
throttle(...)
```

# 全局引用lodash

上述两种方法都需要在每个文件中单独引用，那么有没有一种方法可以一次加载，全局直接引用呢？

有，还有两种。

以vue-cli搭建的项目为例，我们在`main.js`中引入`lodash`：

```javascript
// main.js
import _ from 'lodash'

window._ = _
```

是不是很熟悉呢？这跟开头我们写的似乎没有区别，而且在服务器端渲染的项目中就不可行了，因为服务器端并没有全局的`window`对象。

让我们看看第二种方法，同样是在`main.js`：

```javascript
// main.js
import vue from 'vue'
import _ from 'lodash'

vue.prototype._ = _
```

我们通过将`lodash`挂载到`vue`原型上的方式，就可以在各个`.vue`文件中，通过`this._.lodashFunction`的形式去欢快的使用了，当然如果你觉得`this._.blabla`的形式太丑了，你可以把`_`换成`lodash`或者其他好记的单词。

# 再进一步

以上两种全局引入的方式，都有一个问题：**在打包过程中，会产生冗余的`lodash`代码**。

因为我们不可能使用`lodash`的全部方法，虽然官方号称`Full build`之后再经过`gzip`体积只有大概`24kb`，但是在寸土寸金的带宽费面前，我们还是要有些追求的。

那么可不可以将全局引入和按需引入的方式结合一些呢？当然有，代价就是每个文件还是要引入一次`lodash`，优点是`webpack`会自动将你用到的方法进行打包。

此刻，我们站在了 [lodash-webpack](https://github.com/lodash/lodash-webpack-plugin) 和 [babel-plugin-lodash](https://www.npmjs.com/package/babel-plugin-lodash) 的肩膀上。

1. 首先安装依赖

```
npm install -S lodash
npm install -D lodash-webpack-plugin babel-plugin-lodash
```

2. 修改`.babelrc`文件

在`plugins`中添加`lodash`：

```
"plugins": ["transform-vue-jsx", "transform-runtime", "lodash"]
```

3. 修改`wepback.prod.config.js`

其实我们只需要精简打包时的体积，所以只修改`wepback.prod.config.js`这个文件

```javascript
const LodashModuleReplacementPlugin = require('lodash-webpack-plugin')

//...

plugins: [
    new LodashModuleReplacementPlugin(),
    // ...
]
```

这个时候我们在每个`.vue`文件中去引入一次`lodash`：

```javascript
import _ from 'lodash'

export default {
    data () {

    },
    created () {
        const array = [1, 2, 3, 4, 5]
        console.log(_.chunk(array, 2)) // [[1, 2], [3, 4], [5]]
    }
}
```

这样虽然好像全局引入了`lodash`，但是实际上在打包过程中只会打包`chunk`这个文件。

# 再再进一步

其实现在的解决方法还是没有到达完美的地步，因为要完成按需打包的方式，还是需要在每个文件中去引入一次`lodash`，期待后续能有更加完美的解决方案。