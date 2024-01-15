---
title: jsx基础知识
categories: react
tags: [react,前端,jsx]
cover: /picture/react.jpg
date: 2023-06-18 15:51
---

# jsx语法

JSX是类似于XML的js扩展语法，其本质是React.createElement(component,props,...children)方法的语法糖

注：XML早期用于存储和传输数据。

XML格式如下：

```
<student>
<name>Tom</name>
<age>19</age>
</student>
```

JSON格式如下：

```
"{"name"："Tom","age":19}"
```

# jsx语法规则

1、定义虚拟DOM时，不要写引号。

```
const VDOM=(
 /* 此处一定不要写引号，因为不是字符串 */
 <h1 id="test">  <span>Hello,React</span> </h1>)
//2.渲染虚拟DOM到页面
ReactDOM.render(VDOM,document.getElementById('test')).
```

2、标签里如果要用JS的表达式，要用{}。

```
const myID="Test";
const string1="Hello,React"
const VDOM=(
 /* 此处一定不要写引号，因为不是字符串 */
 <h1 id={myID.toLowerCase()}>  <span>{string1.toLowerCase()}</span> </h1>)
//2.渲染虚拟DOM到页面
ReactDOM.render(VDOM,document.getElementById('test')).
```

3、写样式时指定类名不用class，用className。

```
const myID="Test";
const string1="Hello,React"
const VDOM=(
 /* 此处一定不要写引号，因为不是字符串 */
 <h1 className="title" id={myID.toLowerCase()}>  <span>{string1.toLowerCase()}</span> </h1>)
//2.渲染虚拟DOM到页面
ReactDOM.render(VDOM,document.getElementById('test')).
```

```
 <style>
 .title{
 background:red;
 }
 </style>
```

4、写内联样式时用style={% raw %}"{{kay:value}}"{% endraw %}的样式。

```
const myID="Test";
const string1="Hello,React"
const VDOM=(
 /* 此处一定不要写引号，因为不是字符串 */
 <h1 className="title" id={myID.toLowerCase()}>  <span style={{color:'white',fontSize:'29px'}}>{string1.toLowerCase()}</span> </h1>)
//2.渲染虚拟DOM到页面
ReactDOM.render(VDOM,document.getElementById('test')).
```

5、虚拟DOM必须只有一个根标签。

6、标签必须闭合。

7、标签首字母

1）若小写字母开头，则将标签转为html中同名元素，若html中不存在同名元素，则报错。

2）若大写字母开头，react就去渲染对应的组件，若组件没有定义，则报错。

注：区分js语句和js表达式

1、表达式：会产生一个值，可以放在任何一个需要值的地方。如a、a+b、demo(1)、arr.map()、function test（）{}

2、语句（代码）：

比如if(){}、for(){}、switch（）{case: ....}