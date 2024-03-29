---
title: Vue原理系列之一：Vue的运行机制
categories: Vue
tags: [Vue2, 前端]
cover: /picture/vue.jpg
date: 2024-03-21 12:09
---

开始前首先贴上个人看到的比较好的一些讲述了vue底层原理的文章，大佬们写的非常详细，由浅入深。本系列是站在各位大佬的肩膀上进行的一个总结。

[纯干货！图解Vue响应式原理 - 掘金 (juejin.cn)](https://juejin.cn/post/7074422512318152718?searchId=20240322002254657FCB2BDC7F699EAE05#heading-2)

[https://juejin.cn/user/2647279730694029/posts](https://juejin.cn/user/2647279730694029/posts "https://juejin.cn/user/2647279730694029/posts")

[https://juejin.cn/post/7076002091570823181#heading-12](https://juejin.cn/post/7076002091570823181#heading-12 "https://juejin.cn/post/7076002091570823181#heading-12")

[https://segmentfault.com/a/1190000041689097](https://segmentfault.com/a/1190000041689097 "https://segmentfault.com/a/1190000041689097")

## 引入

首先简单贴一下vue2的 运行机制图

![](image_lSXbEtfbBA.png)

## 初始化及挂载

![](image_UEehMs81Vi.png)

在 `new Vue()` 之后。 Vue 会调用 `_init` 函数进行初始化，也就是这里的 `init` 过程，它会初始化生命周期、事件、 props、 methods、 data、 computed 与 watch 等。

其中最重要的是通过 `Object.defineProperty` 设置 `setter` 与 `getter` 函数，用来实现「**响应式**」以及「**依赖收集**」，后面会详细讲到，这里只要有一个印象即可。

初始化之后调用 `$mount` 会挂载组件，如果是运行时编译，即不存在 render function 但是存在 template 的情况，需要进行「**编译**」步骤。

## 编译

![](image_WcbyAuq7cY.png)

**1. parse**

`parse` 会用正则等方式解析 `template` 模板中的指令、`class`、`style`等数据，形成`AST`。

**2. optimize**

> `optimize` 的主要作用是标记 `static` 静态节点，这是 Vue 在编译过程中的一处优化，后面当 `update` 更新界面时，会有一个 `patch` 的过程， diff 算法会直接跳过静态节点，从而减少了比较的过程，优化了 `patch` 的性能。

**3. generate**

> `generate` 是将 `AST` 转化成 `render function` 字符串的过程，得到结果是 `render` 的字符串以及 `staticRenderFns` 字符串。

-   在经历过 `parse`、`optimize` 与 `generate` 这三个阶段以后，组件中就会存在渲染 `VNode` 所需的 `render function` 了。

## 响应式

![](image_ImjgN5RC2g.png)

这里的 `getter` 跟 `setter` 已经在之前的文章已经介绍过了，在 `init` 的时候通过 `Object.defineProperty` 进行了绑定，它使得当被设置的对象被读取的时候会执行 `getter` 函数，而在当被赋值的时候会执行 `setter` 函数。

-   当 `render function` 被渲染的时候，因为会读取所需对象的值，所以会触发 `getter` 函数进行「**依赖收集**」，「**依赖收集**」的目的是将观察者 `Watcher` 对象存放到当前闭包中的订阅者 `Dep` 的 `subs` 中。形成如下所示的这样一个关系。

![](image_557Exj1wGP.png)

在修改对象的值的时候，会触发对应的 `setter`， `setter` 通知之前「**依赖收集**」得到的 `Dep` 中的每一个 `Watcher`，告诉它们自己的值改变了，需要重新渲染视图。这时候这些 `Watcher` 就会开始调用 `update` 来更新视图，当然这中间还有一个 `patch` 的过程以及使用队列来异步更新的策略，这个我们后面再讲。

## Virtual DOM

> 我们知道，`render function` 会被转化成 `VNode` 节点。`Virtual DOM` 其实就是一棵以 JavaScript 对象（ VNode 节点）作为基础的树，用对象属性来描述节点，实际上它只是一层对真实 DOM 的抽象。最终可以通过一系列操作使这棵树映射到真实环境上。由于 Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Weex、Node 等。

比如说下面这样一个简单的例子：

```javascript
{
    tag: 'div',                 /*说明这是一个div标签*/
    children: [                 /*存放该标签的子节点*/
        {
            tag: 'a',           /*说明这是一个a标签*/
            text: 'click me'    /*标签的内容*/
        }
    ]
}

```

渲染后可以得到

```javascript
<div>
    <a>click me</a>
</div>


```

## 更新视图

![](image_3JAXlV5wbA.png)

-   前面我们说到，在修改一个对象值的时候，会通过 `setter -> Watcher -> update` 的流程来修改对应的视图，那么最终是如何更新视图的呢？
-   当数据变化后，执行 `render function` 就可以得到一个新的 `VNode` 节点，我们如果想要得到新的视图，最简单粗暴的方法就是直接解析这个新的 `VNode` 节点，然后用 `innerHTML` 直接全部渲染到真实 `DOM` 中。但是其实我们只对其中的一小块内容进行了修改，这样做似乎有些「**浪费**」。
-   那么我们为什么不能只修改那些「改变了的地方」呢？这个时候就要介绍我们的「**`patch`**」了。我们会将新的 `VNode` 与旧的 `VNode` 一起传入 `patch` 进行比较，经过 `diff` 算法得出它们的「**差异**」。最后我们只需要将这些「**差异**」的对应 `DOM` 进行修改即可。

## 关于nextTick

通过以上的了解我们知道，vue整个内部运行的机制。那么，现在这里存在一个问题，我们每次数据更新不可能只更新一个数据或者说当某个数据持续更新时，难道我们每更新一次数据都会生成一个DOM吗？

答案当然是否定的。Vue.js在默认情况下，每次触发某个数据的setter方法后，对应的 Watcher 对象会被 push 进一个队列中，它会在下一次tick的时候将这些watcher拿出来，执行一遍对应的patch操作。

那么什么是下一个tick呢？且听随后分解。



