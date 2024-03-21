---
title: Vue原理系列之三：AST编译为render（）
categories: Vue
tags: [Vue2, 前端]
cover: /picture/vue.jpg
date: 2024-03-21 22:42
---

渲染函数是 AST 到虚拟 DOM 节点的中间媒介，本质上就是 JS 的函数，执行后会基于『运行时』返回虚拟节点的对象。

在 Vue.js 2 中，通过执行「渲染函数」获得了虚拟 DOM 节点，用于虚拟节点 Diff 并最终生成真实 DOM。

```javascript
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```

上述3行源码中，调用的`vm._render()`即是「渲染函数」，其返回值即为「虚拟 DOM 节点」。

将虚拟 DOM 节点作为参数传给`vm._update()`后，就开始了著名的『虚拟 DOM Diff』。

## generate

`generate` 会将 AST 转化成 render funtion 字符串，最终得到 render 的字符串以及 staticRenderFns 字符串。

render函数的就是返回一个\_c('tagName',data,children)的方法

1.  第一个参数是标签名
2.  第二个参数是他的一些数据，包括属性/指令/方法／表达式等等。
3.  第三个参数是当前标签的子标签,同样的，每一个子标签的格式也是\_c('tagName',data,children)。

```javascript
function render() {
  with(this) {
   return _c('div',[_v(_s(msg))])
  }
}
```

generate就是通过不断递归形成了这么一种树形结构。

![](image_INCQFDamrb.png)

这里对后面会用到的方法以及函数做一个简单的介绍

genElement：用来生成基本的render结构或者叫createElement结构
genData: 处理ast结构上的一些属性，用来生成data
genChildren:处理ast的children,并在内部调用genElement,形成子元素的\_c()方法

render字符串内部有几种方法：

-   [ ] \_c：对应的是 createElement 方法，顾名思义，它的含义是创建一个元素(Vnode)
-   [ ] \_v：创建一个文本结点。
-   [ ] \_s：把一个值转换为字符串。（eg: {{data}}）
-   [ ] \_m：渲染静态内容

```javascript
<template>
  <div id="app">
    {{val}}
    <img src="http://xx.jpg">
  </div>
</template>

{
  render: with(this) {
    return _c('div', {
      attrs: {
        "id": "app"
      }
    }, [_v("\n" + _s(val) + "\n"),
        _c('img', {
              attrs: {
                "src": ""
              }
            })
        ]
    )
  }
}

```

## 核心原理

### 1. 把字符串函数体转化为函数

写 JS 时，我们可以通过`声明`或`表达式`的形式创造函数。

但是在 JS 的执行过程中「创造函数」我们需要[new Function()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function "new Function()")[ API](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function " API")，即JS中函数的构造函数。

通过调用函数的构造函数，我们可以将「字符串」类型的函数体，转化为一个可执行的JS函数：

```javascript
const func = new Function('console.log(`新函数`)')
/* 
func ===
ƒ anonymous() {
  console.log(`新函数`)
}
*/
func() // 打印 `新函数`
```

通过`new Function()`API，我们就拥有了在 JS 执行过程中生成函数体，并最终声明函数的能力。

### 2. 基于AST生成字符串格式的函数体

有了声明函数的能力，我们就可以把 AST 编译为「字符串格式的函数体」，再将之转化为可执行的函数。

例如，我们有一个`<div />`对应的 AST：

```javascript
{
    "type": 1,
    "tag": "div",
    "children": [],
}
```

想要把 AST 编译为渲染函数的函数体：`_c('div')`。

我们只需要对 AST 进行遍历，根据`tag`属性就可以拼接出想要的函数体：

```javascript
function generate(ast) {
  if (ast.tag) {
    return `_c('${ast.tag}')`
  }
}
```

如果 AST 的`children`属性不为空，我们继续对其进行深度优先递归搜索，就可继续增加渲染函数的函数体，最终生成各种复杂的渲染函数，渲染出复杂的 DOM，例如：

```javascript
const render = function () {
  with (this) {
    return _c(
      'div', {attrs: {"id": "app"}},
      [
        _c('h1', [_v("Hello vue-template-babel-compiler")]),
        _v(" "),
        (optional?.chaining)
          ? _c('h2', [_v("\\n      Optional Chaining enabled: " + _s(optional?.chaining) + "\\n    ")])
          : _e()
      ]
    )
  }
}
```

> 如果有兴趣，可以找到自己项目中的`node_modules/vue-template-compiler/build.js`第4815行：`var code = generate(ast, options);
> `加上`console.log(code)`，`npm run serve`运行后，就可以在控制台中看到自己写的`.vue`文件编译出的渲染函数。

## 具体步骤

### 1. 增加`CodeGenerator`类及其调用

我们用`CodeGenerator`封装编译AST为渲染函数的逻辑，其带有一个`generate(ast)`方法，

传入 AST 作为参数，调用后会返回带有 render() 函数作为属性值的对象：

```javascript
class CodeGenerator {
    generate(ast) {
      debugger
      var code = this.genElement(ast)

      return {
        render: ("with(this){return " + code + "}"),
      }
    }
}
```

### 2. 编译 AST 中的父元素

我们再为类添加一个`genElement`方法，

这个方法接受一个 AST 节点，做2件事：

-   [ ] 继续编译 AST 节点的子节点`children`
-   [ ] 拼接字符串，将当前 AST 节点编译为渲染函数

```javascript
genElement(el) {
  var children = this.genChildren(el)
  const code = `_c('${el.tag}'${children ? `,${children}` : ''})`
  return code
}
```

genElement用于将AST：

```javascript
{
    "type": 1,
    "tag": "div",
    "children": [],
}
```

编译为字符串函数体：`_c('div')`

### 3. 编译 AST 中的子元素

接下来我们编译子元素`ast.children`

`children`是一个数组，可能有多个子元素，所以我们需要对其进行`.map()`遍历，分别处理每一个子元素。

```javascript
genChildren (el, state) {
  var children = el.children
  if (children.length) {
    return `[${children.map(c => this.genNode(c, state)).join(',')}]`
  }
}
```

我们再为类添加一个`genElement`方法，用于调用`genChildren`：

```javascript
  genElement(el) {
    debugger
    var children = this.genChildren(el)
    const code = `_c('${el.tag}'${children ? `,${children}` : ''})`
    return code
  }
```

### 4. 分别处理每一个子元素

我们用`genNode(node)`方法处理子元素，

生产环境中，子元素有多种，可能是文本、注释、HTML元素，所以需要用`if (node.type === 2)`判断类型，在分情况处理。

```javascript
genNode(node) {
  if (node.type === 2) {
    return this.genText(node)
  }
  // TODO else if (node.type === otherType) {}
}
```

我们此次需要处理的只有「文本」（`node.type === 2`）这一种，所以我们再增加一个`genText(text)`来处理。

```javascript
genText(text) {
  return `_v(${text.expression})`
}
```

在编译 AST 阶段，我们已经把`{{msg}}`编译为了一个 JS 对象：

```javascript
  {
    "type": 2,
    "expression": "_s(msg)",
    "tokens": [
        {
           "@binding": "msg"
        }
    ],
    "text": "{{msg}}"
  }
```

现在我们只要取`expression`属性，就是其对应的渲染函数。

简而言之`_s()`是 Vue.js 内置的一个方法，可以把传入的字符串生成一个对应的虚拟 DOM 节点。

后续我们将详细介绍`_s(msg)`的含义及其实现。

### 5. 拼接为字符串函数体、生成渲染函数

经过以上各步骤，我们已将 AST 对象解析成了渲染函数的函数体字符串：`with(this){return _c('div',[_v(_s(msg))])}`，

为了将仍然是字符串函数体的`render`属性，转化为可执行的函数，我们再增加一段`new Function(code)`逻辑，

并把`createFunction (code)`声明到`VueCompiler`类，以便于最终调用：

```javascript
createFunction (code) {
  try {
    return new Function(code)
  } catch (err) {
    throw err
  }
}
```

最后我们来统一调用。

在`VueCompiler`类的`compile(template)`中添加`CodeGenerator`实例及`this.CodeGenerator.generate(ast)`调用：

```javascript
class VueCompiler {
  HTMLParser = new HTMLParser()
  CodeGenerator = new CodeGenerator()

  compile(template) {
    const ast = this.parse(template)
    console.log(`一、《template 字符串编译为抽象语法树 AST》`)
    console.log(`ast = ${JSON.stringify(ast, null, 2)}`)

    const code = this.CodeGenerator.generate(ast)
    const render = this.createFunction(code.render)
    console.log(`二、《抽象语法树 AST 编译为渲染函数 render()》`)
    console.log(`render() = ${render}`)
    return render
  }
}
```
