---
title: Promise
categories: Javascript
tags: [Javascript,前端]
cover: /picture/ES6.jpg
date: 2024-03-03 14:34
---

## 概述

1.  什么是 Promise ？
    -   Promise 对象用于表示一个异步操作的最终完成（或失败）及其结构值
2.  Promise 的好处是什么？
    -   逻辑更清晰（成功或失败会关联后续的处理函数）
    -   了解 axios 函数内部运作的机制
    -   解决回调函数地狱问题



```javascript
/**
 * 目标：使用Promise管理异步任务
*/
// 1. 创建Promise对象
const p = new Promise((resolve, reject) => {
  // 2. 执行异步代码
  setTimeout(() => {
    // resolve('模拟AJAX请求-成功结果')
    reject(new Error('模拟AJAX请求-失败结果'))
  }, 2000)
})

// 3. 获取结果
p.then(result => {
  console.log(result)
}).catch(error => {
  console.log(error)
})
```

## promise的状态

Promise 有哪三种状态？

> 每个 Promise 对象必定处于以下三种状态之一

1.  待定（pending）：初始状态，既没有被兑现，也没有被拒绝
2.  已兑现（fulfilled）：操作成功完成
3.  已拒绝（rejected）：操作失败

Promise 的状态改变有什么用？

调用对应函数，改变 Promise 对象状态后，内部触发对应回调函数传参并执行

![](image_e9l2XI1110.png)

注意：每个 Promise 对象一旦被兑现/拒绝，那就是已敲定了，状态无法再被改变

## 创建promise实例

**一般情况下都会使用**`new Promise()`**来创建promise对象，但是也可以使用**`promise.resolve`**和**`promise.reject`**这两个方法：**

-   **Promise.resolve**

`Promise.resolve(value)`的返回值也是一个promise对象，可以对返回值进行.then调用，代码如下：

```javascript
Promise.resolve(11).then(function(value){
  console.log(value); // 打印出11
});

```

-   **Promise.reject**

`Promise.reject` 也是`new Promise`的快捷形式，也创建一个promise对象。代码如下

```javascript
Promise.reject(new Error(“我错了！！”));

```

## Promise的实例方法和静态方法

### 实例方法

**then**

`then`方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为`resolved`时调用，第二个回调函数是Promise对象的状态变为`rejected`时调用。其中第二个参数可以省略。 `then`方法返回的是一个新的Promise实例（不是原来那个Promise实例）。因此可以采用链式写法，即`then`方法后面再调用另一个then方法。

**catch**

该方法相当于`then`方法的第二个参数，指向`reject`的回调函数。不过`catch`方法还有一个作用，就是在执行`resolve`回调函数时，如果出现错误，抛出异常，不会停止运行，而是进入`catch`方法中。

**finally**

`finally`方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。

下面是一个例子，服务器使用 Promise 处理请求，然后使用`finally`方法关掉服务器。

```javascript
server.listen(port)
  .then(function () {
    // ...
  })
  .finally(server.stop);

```

`finally`方法的回调函数不接受任何参数，这意味着没有办法知道，前面的 Promise 状态到底是`fulfilled`还是`rejected`。这表明，`finally`方法里面的操作，应该是与状态无关的，不依赖于 Promise 的执行结果。

### 静态方法

**all**

`all`方法可以完成并发任务， 它接收一个数组，数组的每一项都是一个`promise`对象，返回一个[Promise](https://link.juejin.cn?target=https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise "Promise")实例。当数组中所有的`promise`的状态都达到`resolved`的时候，`all`方法的状态就会变成`resolved`，如果有一个状态变成了`rejected`，那么`all`方法的状态就会变成`rejected`。

**race**

`race`方法和`all`一样，接受的参数是一个每项都是`promise`的数组，但是与`all`不同的是，当最先执行完的事件执行完之后，就直接返回该`promise`对象的值。如果第一个`promise`对象状态变成`resolved`，那自身的状态变成了`resolved`；反之第一个`promise`变成`rejected`，那自身状态就会变成`rejected`。

**any**

它接收一个数组，数组的每一项都是一个`promise`对象，该方法会返回一个新的 `promise`，数组内的任意一个 `promise` 变成了`resolved`状态，那么由该方法所返回的 `promise` 就会变成`resolved`状态。如果数组内的 `promise` 状态都是`rejected`，那么该方法所返回的 `promise` 就会变成`rejected`状态，

**resolve、reject**

用来生成对应状态的Promise实例

### Promise.all、Promise.race、Promise.any的区别

**all：** 成功的时候返回的是**一个结果数组**，而失败的时候则返回**最先被reject失败状态的值**。

**race：** 哪个结果获得的快，就返回那个结果，不管结果本身是成功状态还是失败状态。

**any：** 返回最快的成功结果，如果全部失败就返回失败结果

