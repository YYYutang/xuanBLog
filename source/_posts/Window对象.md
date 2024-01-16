---
title: Javascript Windows对象
categories: Javascript
tags: [Javascript,前端]
cover: /picture/javascript.jpg
date: 2024-01-16 20:17
---



## JavaScript的组成

-   ECMAScript:
    -   规定了js基础语法核心知识。
    -   比如：变量、分支语句、循环语句、对象等等
-   Web APIs :
    -   DOM 文档对象模型， 定义了一套操作HTML文档的API
    -   BOM 浏览器对象模型，定义了一套操作浏览器窗口的API

![](1676047389456_33gqey4Bkw.png)

## JS执行机制

### 同步任务

同步任务都在主线程上执行，形成一个执行栈。

![](image_5wiBTQqbj7.png)

### 异步任务

JS 的异步是通过回调函数实现的。

一般而言，异步任务有以下三种类型:

1、普通事件，如 click、resize 等

2、资源加载，如 load、error 等

3、定时器，包括 setInterval、setTimeout 等

异步任务相关添加到任务队列中（任务队列也称为消息队列）。

![](image_fhaWyLY9Mw.png)

### 事件循环

1.  先执行执行栈中的同步任务。
2.  异步任务放入任务队列中。
3.  一旦执行栈中的所有同步任务执行完毕，系统就会按次序读取任务队列中的异步任务，于是被读取的异步任务结束等待状态，进入执行栈，开始执行。

![](image_POygugmxWz.png)

![](image_HGFhfC6n7-.png)

## BOM对象

**BOM** (Browser Object Model ) 是浏览器对象模型

-   window对象是一个全局对象，也可以说是JavaScript中的顶级对象
-   像document、alert()、console.log()这些都是window的属性，基本BOM的属性和方法都是window的
-   所有通过var定义在全局作用域中的变量、函数都会变成window对象的属性和方法
-   window对象下的属性和方法调用的时候可以省略window

![](1676047436362_Sn5b0Y1wJ7.png)

## 定时器-延迟函数

JavaScript 内置的一个用来让代码延迟执行的函数，叫 setTimeout

**语法：**

```javascript
setTimeout(回调函数, 延迟时间)
```

setTimeout 仅仅只执行一次，所以可以理解为就是把一段代码延迟执行, 平时省略window

间歇函数 setInterval : 每隔一段时间就执行一次, 平时省略window

清除延时函数：

```javascript
clearTimeout(timerId)
```

> 注意点
> 1\.  延时函数需要等待,所以后面的代码先执行
> 2\.  返回值是一个正整数，表示定时器的编号

```javascript
<body>
  <script>
    // 定时器之延迟函数

    // 1. 开启延迟函数
    let timerId = setTimeout(function () {
      console.log('我只执行一次')
    }, 3000)

    // 1.1 延迟函数返回的还是一个正整数数字，表示延迟函数的编号
    console.log(timerId)

    // 1.2 延迟函数需要等待时间，所以下面的代码优先执行

    // 2. 关闭延迟函数
    clearTimeout(timerId)

  </script>
</body>
```

## location对象

location (地址) 它拆分并保存了 URL 地址的各个组成部分， 它是一个对象

| 属性/方法    | 说明                            |
| -------- | ----------------------------- |
| href     | 属性，获取完整的 URL 地址，赋值时用于地址的跳转    |
| search   | 属性，获取地址中携带的参数，符号 ？后面部分        |
| hash     | 属性，获取地址中的哈希值，符号 # 后面部分        |
| reload() | 方法，用来刷新当前页面，传入参数 true 时表示强制刷新 |

```javascript
<body>
  <form>
    <input type="text" name="search"> <button>搜索</button>
  </form>
  <a href="#/music">音乐</a>
  <a href="#/download">下载</a>

  <button class="reload">刷新页面</button>
  <script>
    // location 对象  
    // 1. href属性 （重点） 得到完整地址，赋值则是跳转到新地址
    console.log(location.href)
    // location.href = 'http://www.itcast.cn'

    // 2. search属性  得到 ? 后面的地址 
    console.log(location.search)  // ?search=笔记本

    // 3. hash属性  得到 # 后面的地址
    console.log(location.hash)

    // 4. reload 方法  刷新页面
    const btn = document.querySelector('.reload')
    btn.addEventListener('click', function () {
      // location.reload() // 页面刷新
      location.reload(true) // 强制页面刷新 ctrl+f5
    })
  </script>
</body>
```

## navigator对象

navigator是对象，该对象下记录了浏览器自身的相关信息

常用属性和方法：

-   通过 userAgent 检测浏览器的版本及平台

```javascript
// 检测 userAgent（浏览器信息）
(function () {
  const userAgent = navigator.userAgent
  // 验证是否为Android或iPhone
  const android = userAgent.match(/(Android);?[\s\/]+([\d.]+)?/)
  const iphone = userAgent.match(/(iPhone\sOS)\s([\d_]+)/)
  // 如果是Android或iPhone，则跳转至移动站点
  if (android || iphone) {
    location.href = 'http://m.itcast.cn'
  }})();
```

## histroy对象

history (历史)是对象，主要管理历史记录， 该对象与浏览器地址栏的操作相对应，如前进、后退等

**使用场景**

history对象一般在实际开发中比较少用，但是会在一些OA 办公系统中见到。

常见方法：

![](1676047846593_aXdUWYlxO1.png)

```javascript
<body>
  <button class="back">←后退</button>
  <button class="forward">前进→</button>
  <script>
    // histroy对象

    // 1.前进
    const forward = document.querySelector('.forward')
    forward.addEventListener('click', function () {
      // history.forward() 
      history.go(1)
    })
    // 2.后退
    const back = document.querySelector('.back')
    back.addEventListener('click', function () {
      // history.back()
      history.go(-1)
    })
  </script>
</body>

```
