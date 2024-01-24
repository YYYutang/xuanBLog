---
title: Javascript DOM操作
categories: Javascript
tags: [Javascript,前端]
cover: /picture/javascript.jpg
date: 2024-01-16 20:12
---



DOM（Document Object Model）是将整个 HTML 文档的每一个标签元素视为一个对象，这个对象下包含了许多的属性和方法，通过操作这些属性或者调用这些方法实现对 HTML 的动态更新，为实现网页特效以及用户交互提供技术支撑。

简言之 DOM 是用来动态修改 HTML 的，其目的是开发网页特效及用户交互。

## 概念

### DOM 树

![](web-api_d_ZJuQ9Ou4.jpg)

### DOM 节点

节点是文档树的组成部分，**每一个节点都是一个 DOM 对象**，主要分为元素节点、属性节点、文本节点等。

1.  【元素节点】其实就是 HTML 标签，如上图中 `head`、`div`、`body` 等都属于元素节点。
2.  【属性节点】是指 HTML 标签中的属性，如上图中 `a` 标签的 `href` 属性、`div` 标签的 `class` 属性。
3.  【文本节点】是指 HTML 标签的文字内容，如 `title` 标签中的文字。
4.  【根节点】特指 `html` 标签。
5.  其它...

### document

`document` 是 JavaScript 内置的专门用于 DOM 的对象，该对象包含了若干的属性和方法，`document` 是学习 DOM 的核心。

```javascript
<script>  
// document 是内置的对象  
// console.log(typeof document);  
// 1. 通过 document 获取根节点  
console.log(document.documentElement); // 对应 html 标签  
// 2. 通过 document 节取 body 节点  
console.log(document.body); // 对应 body 标签  
// 3. 通过 document.write 方法向网页输出内容  
document.write('Hello World!');  
</script>

```

上述列举了 `document` 对象的部分属性和方法，我们先对 `document` 有一个整体的认识

## 获取dom对象

1.  querySelector 满足条件的第一个元素
2.  querySelectorAll 满足条件的元素集合 返回伪数组
3.  了解其他方式
    1.  getElementById
    2.  getElementsByTagName

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DOM - 查找节点</title>
</head>
<body>
  <h3>查找元素类型节点</h3>
  <p>从整个 DOM 树中查找 DOM 节点是学习 DOM 的第一个步骤。</p>
  <ul>
      <li>元素</li>
      <li>元素</li>
      <li>元素</li>
      <li>元素</li>
  </ul>
  <script>
    const p = document.querySelector('p')  // 获取第一个p元素
    const lis = document.querySelectorAll('li')  // 获取第一个p元素
  </script>
</body>
</html>
```

总结：

-   document.getElementById 专门获取元素类型节点，根据标签的 `id` 属性查找
-   任意 DOM 对象都包含 nodeType 属性，用来检检测节点类型

## 操作元素内容

### 操作元素内容

通过修改 DOM 的文本内容，动态改变网页的内容。

1.  innerText 将文本内容添加/更新到任意标签位置，文本中包含的标签不会被解析。
    ```javascript
    <script>
    // innerText 将文本内容添加/更新到任意标签位置
    const intro = document.querySelector('.intro')
    // intro.innerText = '嗨~ 我叫李雷！'
    // intro.innerText = '<h4>嗨~ 我叫李雷！</h4>'
    </script>

    ```
2.  innerHTML 将文本内容添加/更新到任意标签位置，文本中包含的标签会被解析。
    ```javascript
    <script>
    // innerHTML 将文本内容添加/更新到任意标签位置

    const intro = document.querySelector('.intro')
    intro.innerHTML = '嗨~ 我叫韩梅梅！'
    intro.innerHTML = '<h4>嗨~ 我叫韩梅梅！</h4>'
    </script>

    ```

总结：如果文本内容中包含 html 标签时推荐使用 innerHTML，否则建议使用 innerText 属性。

操作元素属性

有3种方式可以实现对属性的修改：

常用属性修改

1.  直接能过属性名修改，最简洁的语法
    ```javascript
    <script>
    // 1. 获取 img 对应的 DOM 元素

    const pic = document.querySelector('.pic')
    // 2. 修改属性
    pic.src = './images/lion.webp'
    pic.width = 400;
    pic.alt = '图片不见了...'
    </script>

    ```

### 控制样式属性

1.  **应用【修改样式】，通过修改行内样式 style 属性，实现对样式的动态修改。**

通过元素节点获得的 style 属性本身的数据类型也是对象，如 box.style.color、box.style.width 分别用来获取元素节点 CSS 样式的 color 和 width 的值。

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>练习 - 修改样式</title>
</head>
<body>
  <div class="box">随便一些文本内容</div>
  <script>
    // 获取 DOM 节点
    const box = document.querySelector('.intro')
    box.style.color = 'red'
    box.style.width = '300px'
    // css 属性的 - 连接符与 JavaScript 的 减运算符
    // 冲突，所以要改成驼峰法
    box.style.backgroundColor = 'pink'
  </script>
</body>
</html>
```

任何标签都有 style 属性，通过 style 属性可以动态更改网页标签的样式，如要遇到 css 属性中包含字符 - 时，要将 - 去掉并将其后面的字母改成大写，如 background-color 要写成 box.style.backgroundColor

1.  **操作类名(className) 操作CSS**

如果修改的样式比较多，直接通过style属性修改比较繁琐，我们可以通过借助于css类名的形式。

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>练习 - 修改样式</title>
    <style>
        .pink {
            background: pink;
            color: hotpink;
        }
    </style>
</head>
<body>
  <div class="box">随便一些文本内容</div>
  <script>
    // 获取 DOM 节点
    const box = document.querySelector('.intro')
    box.className = 'pink'
  </script>
</body>
</html>
```

注意：

1.由于class是关键字, 所以使用className去代替

2.className是使用新值换旧值, 如果需要添加一个类,需要保留之前的类名

1.  **通过 classList 操作类控制CSS**

为了解决className 容易覆盖以前的类名，我们可以通过classList方式追加和删除类名

```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
      div {
        width: 200px;
        height: 200px;
        background-color: pink;
      }
      .active {
        width: 300px;
        height: 300px;
        background-color: hotpink;
        margin-left: 100px;
      }
    </style>
  </head>
  <body>
    <div class="one"></div>
    <script>
      // 1.获取元素
      // let box = document.querySelector('css选择器
      let box = document.querySelector('div')
      // add是个方法 添加  追加
      // box.classList.add('active')
      // remove() 移除 类
      // box.classList.remove('one')
      // 切换类，有就删掉，没有就加上
      box.classList.toggle('one')
    </script>
  </body>
  </html>
```

### 操作表单元素属性

表单很多情况，也需要修改属性，比如点击眼睛，可以看到密码，本质是把表单类型转换为文本框

正常的有属性有取值的跟其他的标签属性没有任何区别

获取:DOM对象.属性名

设置:DOM对象.属性名= 新值

```javascript
<!DOCTYPE html>

<html lang="en"
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <input type="text" value="请输入">
  <button disabled>按钮</button>
  <input type="checkbox" name="" id="" class="agree">
  <script>
    // 1. 获取元素
    let input = document.querySelector('input')
    // 2. 取值或者设置值  得到input里面的值可以用 value
    // console.log(input.value)
    input.value = '小米手机'
    input.type = 'password'
    // 2. 启用按钮
    let btn = document.querySelector('button')
    // disabled 不可用   =  false  这样可以让按钮启用
    btn.disabled = false
    // 3. 勾选复选框
    let checkbox = document.querySelector('.agree')
    checkbox.checked = false
  </script>
</body> 
</html>
```

#### 自定义属性

标准属性: 标签天生自带的属性 比如class id title等, 可以直接使用点语法操作比如： disabled、checked、selected

自定义属性：

在html5中推出来了专门的data-自定义属性 &#x20;

在标签上一律以data-开头

在DOM对象上一律以dataset对象方式获取

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div data-id="1"> 自定义属性 </div>
    <script>
      // 1. 获取元素
      let div = document.querySelector('div')
      // 2. 获取自定义属性值
        console.log([div.dataset.id](http://div.dataset.id))
    </script>
</body>
</html>
```

## 定时器

### 间歇函数

知道间歇函数的作用，利用间歇函数创建定时任务。

setInterval 是 JavaScript 中内置的函数，它的作用是间隔固定的时间自动重复执行另一个函数，也叫定时器函数。

```javascript
<script>
  // 1. 定义一个普通函数
  function repeat() {
    console.log('不知疲倦的执行下去....')
  }
  // 2. 使用 setInterval 调用 repeat 函数
  // 间隔 1000 毫秒，重复调用 repeat
  setInterval(repeat, 1000)
  clearInterval(repeat)//关闭定时器
</script>
```
