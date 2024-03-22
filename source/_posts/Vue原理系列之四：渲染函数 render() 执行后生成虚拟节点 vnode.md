---
title: Vue原理系列之四：渲染函数 render() 执行后生成虚拟节点 vnode
categories: Vue
tags: [Vue2, 前端]
cover: /picture/vue.jpg
date: 2024-03-22 10:54
---

## VNode

### 概念

虚拟节点（virtual node， 简写VNode），一个vnode就是一个js对象（通过VNode实例而来），用来描述一个真实DOM元素，它所包含的信息会告诉Vue页面上需要渲染什么样的节点，包括及其子节点的描述信息。 而虚拟DOM是对Vue组件树建立起来的整个VNode树的称呼。

先来看看VNode类的代码：

```javascript
var VNode = function VNode(
  tag,
  data,
  children,
  text,
  elm,
  context,
  componentOptions,
  asyncFactory
) {
  this.tag = tag;    // 节点标签名
  this.data = data;  // 节点数据（VNodeData类型）
  this.children = children;    // 子节点
  this.text = text;  // 节点文本
  this.elm = elm;   // 节点对应的真实DOM节点
  this.ns = undefined;   // 命名空间
  this.context = context;  // 上下文
  this.fnContext = undefined;  // 函数化组件上下文
  this.fnOptions = undefined;  // 函数化组件配置项
  this.fnScopeId = undefined;  // 函数化组件ScopeId
  this.key = data && data.key; // 节点的key属性，用于作为节点的标识，有利于patch的优化
  this.componentOptions = componentOptions;  // 组件配置项
  this.componentInstance = undefined;  // 组件实例
  this.parent = undefined;  // 父节点
  this.raw = false;  // 是否为原生HTML或只是普通文本
  this.isStatic = false;  // 静态节点标志 keep-alive
  this.isRootInsert = true;   // 是否作为根节点插入
  this.isComment = false;  // 是否为注释节点
  this.isCloned = false;   // 是否为克隆节点
  this.isOnce = false;   // 当前节点是否有v-once指令
  this.asyncFactory = asyncFactory;  // 异步工厂方法 
  this.asyncMeta = undefined;   // 异步Meta
  this.isAsyncPlaceholder = false;  // 是否为异步占位
};
```

### VNode的作用

由于每次渲染视图时都是先创建vnode，然后使用它创建的真实DOM插入到页面中，所以可以将上一次渲染视图时先所创建的vnode先缓存起来，之后每当需要重新渲染视图时，将新创建的vnode和上一次缓存的vnode对比，查看他们之间有哪些不一样的地方，找出不一样的地方并基于此去修改真实的DOM。
Vue.js目前对状态的侦测策略采用了中等粒度。当状态发生变化时，只通知到组件级别，然后组件内使用虚拟DOM来渲染视图。

![](20210728151514840_dvdNgpEaXc.png)

如果组件只有一个节点发生了变化，那么重新渲染整个组件的所有节点，很明显会造成很大的性能浪费。因此，对vnode进行缓存，并将上一次的缓存和当前创建的vnode对比，只更新有差异的节点就变得很重要。这也是vnode最重要的一个作用。

### VNode的类型

vnode类型有以下几种：

#### 注释节点

```javascript
var createEmptyVNode = function (text) {
  if (text === void 0) text = "";
 
  var node = new VNode();
  node.text = text;
  node.isComment = true;
  return node;
};
```

一个注释节点只有两个有效属性text 和isComment，其余属性全是默认undefined或者false

```javascript
// <!-- 注释节点 -->
{
    text: "注释节点",
    isComment: true
}
```

#### 文本节点

```javascript
function createTextVNode(val) {
  return new VNode(undefined, undefined, undefined, String(val));
}
```

#### 元素节点

元素节点通常会存在以下4中有效属性。

- tag：tag就是一个节点的名称，例如 p、ul、li和div等。
- data：改属性包含了一些节点上的数据，比如attrs、class和style等。
- children：当前节点的子节点列表。
- context：它是当前组件的Vue.js实例

比如

```javascript
<p><span>Hello</span><span>World</span></p>
```

对应的vnode：

```javascript
{
  children: [VNode, VNode],
  context: {...},
  data: {...},
  tag: "p",
  ...
}
```

#### 组件节点

组件节点有以下两个独有的属性：

componentOptions：组件节点的选项参数，其中包含了propsData、tag和children等信息
componentInstance：组件的实例，也就是Vue.js的实例。事实上，在Vue.js中，每个组件都有一个Vue.js实例
比如\<child>\</child>，对应的vnode

```javascript
{
    componentInstance: {...},
    componentOptions: {...},
    context: {...},
    data: {...},
    tag: "vue-component-1-child",
    ...    
}
```

#### 函数式组件

函数式节点有两个独有的属性functionalContext和functionalOptions

```javascript
{
   functionalContext: {...},
   functionalOptions: {...},
   context: {...},
   data: {...},
   tag: "div"
 }
```

#### 克隆节点

克隆节点是将现有节点的属性赋值到新节点中，让新创建的节点和被克隆的节点的属性保持一致，从而实现克隆效果。它的作用是优化静态节点和插槽节点（slot node）。

以静态节点为例，当组件内某个状态发生变化后，当前组件会通过虚拟DOM重新渲染视图，静态节点因为它的内容不会改变，所以除了首次渲染需要执行渲染函数获取vnode之外，后续更新不需要执行渲染函数重新生成vnode。

因此，这是就会使用创建克隆节点的方法将vnode克隆一份，使用克隆节点进行渲染。这样就不需要执行渲染函数生成新的静态节点的vnode，从而提升一定的性能。

```javascript
function cloneVNode(vnode) {
  var cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children && vnode.children.slice(),
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  );
  cloned.ns = vnode.ns;
  cloned.isStatic = vnode.isStatic;
  cloned.key = vnode.key;
  cloned.isComment = vnode.isComment;
  cloned.fnContext = vnode.fnContext;
  cloned.fnOptions = vnode.fnOptions;
  cloned.fnScopeId = vnode.fnScopeId;
  cloned.asyncMeta = vnode.asyncMeta;
  cloned.isCloned = true;
  return cloned;
}
```

## 生成一个新的vnode的过程

通常使用createElement用来创建一个虚拟节点。

当data上已经绑定\_\_ob\_\_的时候，代表该对象已经被Oberver过了，所以创建一个空节点。tag不存在的时候同样创建一个空节点。当tag不是一个String类型的时候代表tag是一个组件的构造类，直接用new VNode创建。当tag是String类型的时候，如果是保留标签，则用new VNode创建一个VNode实例，如果在vm的option的components找得到该tag，代表这是一个组件，否则统一用new VNode创建。
