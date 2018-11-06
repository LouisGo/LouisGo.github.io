---
title: v-for循环中使用自定义属性值进行事件代理性能优化
date: 2018-09-23 10:21:15
tags:
  - JavaScript
  - Vue
  - 优化
categories:
  - 前端
keywords: 性能优化
---

原文：[vue组件通信全揭秘(共7章)](https://juejin.im/post/5bd97e7c6fb9a022852a71cf)

# 官方定义

默认情况下父作用域的不被认作 props 的特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。通过设置 inheritAttrs 到 false，这些默认行为将会被去掉。而通过 (同样是 2.4 新增的) 实例属性 $attrs 可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上。

**这个选项不影响 class 和 style 绑定**

# 事件代理

当我们用 v-for 渲染大量的同样的 DOM 结构时，但是每个上面都加一个点击事件，这个会导致性能问题，那我们可以通过 HTML5 的 data 的自定义属性做事件代理。

<!--more-->

父组件：

```javascript
<template>
  <div class="hello" @click="ff">
     <demo :first="firstMsg" :second="secondMessage" :data-second="secondMsg"></demo>
     <demo :first="firstMsg" :second="secondMessage" :data-second="secondMsg"></demo>
     <demo :first="firstMsg" :second="secondMessage" :data-second="secondMsg"></demo>
     <demo :first="firstMsg" :second="secondMessage" :data-second="secondMsg"></demo>
  </div>
</template>

<script>
  import Demo from './Demo.vue'
  export default {
    name: 'hello',
    components: {
      Demo
    },
    data () {
      return {
        firstMsg: 'first props',
        secondMsg: 'secondMsg'
      }
    },
    methods: {
      ff (e) {
        if(e.target.dataset.second == 'secondMsg') {
            // do something...
            console.log('通过事件委托拿到了自定义属性')
        }
      }
    }
  }
</script>
```

子组件：

```javascript
<template>
   <div>
      {{first}}
   </div>
</template>
<script>
export default {
   name: 'demo',
   props: ['first'],
}
</script>
```

在父组件中，把向子组件传递的参数名改成了 HTML 自定义的 data-second 属性，同样在子组件中不进行 props 接收，就顺其自然的成为了子组件每一个根节点的自定义属性。
通过事件冒泡的原理，然而可以从e.target.dataset.second 就能找对应的 Dom 节点进行逻辑操作。
同样，在子组件模版上可以绑定多个自定义属性，在子组件包裹的外层进行一次监听，通过 data 自定义属性拿到循环出来组件的对应的数据，进行逻辑操作。

`interitAttrs = false`发生了什么 ？

```javascript
<template>
   <div>
      {{first}}
   </div>
</template>
<script>
export default {
   name: 'demo',
   props: ['first'],
   inheritAttrs: false,
   create () {
       console.log(this.$attrs) // {second: "secondMsg", third: "thirdMsg"}
   }
}
</script>
```

想要通 $attr 接收，但必须要保证设置选项 inheritAttrs: false，不然会默认变成根元素的属性节点，最有用的情况则是在深层次组件运用的时候，创建第三层孙子组件，作为第二层父组件的子组件，在子组件引入的孙子组件，在模版上把整个 $attr 当数作数据传递下去，中间则并不用通过任何方法去手动转换数据。

子组件：

```javascript
<template>
   <div>
      <next-demo v-bind="$attrs"></next-demo>
   </div>
</template>
```

孙子组件：
```javascript
<template>
  <div>
      {{second}}{{third}}
  </div>
</template>

<script>
  export default {
     props : [ 'second' , 'third']
  }
</script>
```