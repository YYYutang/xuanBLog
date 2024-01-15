---
title: 虚拟DOM创建
categories: react
tags: [react,前端]
cover: /picture/react.jpg
date: 2023-06-18 15:45
---


# 1.用jsx创建虚拟DOM

```javascript
const VDOM=(
 /* 此处一定不要写引号，因为不是字符串 */
 <h1 id="title">  <span>Hello,React</span> </h1>)
ReactDOM.render(VDOM,document.querySelector('.test'))
```

# 2.用js创建虚拟DOM

```javascript
//1.创建虚拟DOM,创建嵌套格式的dom
const VDOM=React.createElement('h1',{id:'title'},React.createElement('span',{},'hello,React'))
//2.渲染虚拟DOM到页面
ReactDOM.render(VDOM,document.querySelector('.test')).
```

# 3.虚拟DOM与真实DOM的区别

1、虚拟DOM本质上就是Object类型的对象

2、虚拟DOM较为轻量级，真实DOM较为重量级，因为虚拟DOM运用于React内部，无需真实DOM上的过多属性。

3、虚拟DOM最终会被React转化为真实DOM。

