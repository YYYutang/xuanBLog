---
title: Nodejs 模块化
categories: Node.js
tags: [Node.js, 前端]
cover: /picture/nodejs.webp
date: 2024-03-11 11:22
---

## 模块化简介

- 在 Node.js 中每个文件都被当做是一个独立的模块，模块内定义的变量和函数都是独立作用域的，因为 Node.js 在执行模块代码时，将使用如下所示的函数封装器对其进行封装

![](image-20230331150152299_7h7aNpcf2h.png)

- 而且项目是由多个模块组成的，每个模块之间都是独立的，而且提高模块代码复用性，按需加载，独立作用域

![](image-20230331150407659_SmFc_qZ1rO.png)

- 但是因为模块内的属性和函数都是私有的，如果对外使用，需要使用标准语法导出和导入才可以，而这个标准叫 CommonJS 标准，接下来我们在一个需求中，体验下模块化导出和导入语法的使用

  需求：定义 utils.js 模块，封装基地址和求数组总和的函数，导入到 index.js 使用查看效果

![](image-20230331150506876_cgzAyIhRm5.png)

- 导出语法：

```javascript
module.exports = {
  对外属性名: 模块内私有变量
}
```

- 导入语法：

```javascript
const 变量名 = require('模块名或路径')
// Node.js 环境内置模块直接写模块名（例如：fs，path，http）
// 自定义模块：写模块文件路径（例如：./utils.js)
```

变量名的值接收的就是目标模块导出的对象

代码实现

-   utils.js：导出

```javascript
/**
 * 目标：基于 CommonJS 标准语法，封装属性和方法并导出
 */
const baseURL = 'http://hmajax.itheima.net'
const getArraySum = arr => arr.reduce((sum, item) => sum += item, 0)

// 导出
module.exports = {
  url: baseURL,
  arraySum: getArraySum
}
```

-   index.js：导入使用

```javascript
/**
 * 目标：基于 CommonJS 标准语法，导入工具属性和方法使用
 */
// 导入
const obj = require('./utils.js')
console.log(obj)
const result = obj.arraySum([5, 1, 2, 3])
console.log(result)
```

## ECMAScript标准-默认导出和导入

CommonJS 规范是 Node.js 环境中默认的，后来官方推出 ECMAScript 标准语法，我们接下来在一个需求中，体验下这个标准中默认导出和导入的语法要如何使用

1.  需求：封装并导出基地址和求数组元素和的函数，导入到 index.js 使用查看效果
2.  导出语法：

```javascript
export default {
  对外属性名: 模块内私有变量
}
```

3. 导入语法：

```javascript
import 变量名 from '模块名或路径'
```

变量名的值接收的就是目标模块导出的对象

注意：Node.js 默认只支持 CommonJS 标准语法，如果想要在当前项目环境下使用 ECMAScript 标准语法，请新建 package.json 文件设置 type: 'module' 来进行设置

```javascript
{ “type”: "module" }
```

1.  代码实现：

-   utils.js：导出

```javascript
/**
 * 目标：基于 ECMAScript 标准语法，封装属性和方法并"默认"导出
 */
const baseURL = 'http://hmajax.itheima.net'
const getArraySum = arr => arr.reduce((sum, item) => sum += item, 0)

// 默认导出
export default {
  url: baseURL,
  arraySum: getArraySum
}
```

-   index.js：导入

```javascript
/**
 * 目标：基于 ECMAScript 标准语法，"默认"导入，工具属性和方法使用
 */
// 默认导入
import obj from './utils.js'
console.log(obj)
const result = obj.arraySum([10, 20, 30])
console.log(result)
```

## ECMAScript标准-命名导出和导入

1.  ECMAScript 标准的语法有很多，常用的就是默认和命名导出和导入，现在来学习下命名导出和导入的使用
2.  需求：封装并导出基地址和数组求和函数，导入到 index.js 使用查看效果
3.  命名导出语法：

```javascript
export 修饰定义语句
```

4. 命名导入语法：

```javascript
import { 同名变量 } from '模块名或路径'
```

代码示例：

-   utils.js 导出

```javascript
/**
 * 目标：基于 ECMAScript 标准语法，封装属性和方法并"命名"导出
 */
export const baseURL = 'http://hmajax.itheima.net'
export const getArraySum = arr => arr.reduce((sum, item) => sum += item, 0)
```

-   index.js 导入

```javascript
/**
 * 目标：基于 ECMAScript 标准语法，"命名"导入，工具属性和方法使用
 */
// 命名导入
import {baseURL, getArraySum} from './utils.js'
console.log(obj)
console.log(baseURL)
console.log(getArraySum)
const result = getArraySum([10, 21, 33])
console.log(result)
```

5. 与默认导出如何选择：

-   按需加载，使用命名导出和导入
-   全部加载，使用默认导出和导入
