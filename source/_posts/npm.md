---
title: Nodejs npm
categories: Node.js
tags: [Node.js, npm, 前端]
cover: /picture/nodejs.webp
date: 2024-03-11 10:50
---

## 简介

1.  npm 简介[链接](http://dev.nodejs.cn/learn/an-introduction-to-the-npm-package-manager "链接")： 软件包管理器，用于下载和管理 Node.js 环境中的软件包
2.  npm 使用步骤：
    1.  初始化清单文件： npm init -y （得到 package.json 文件，有则跳过此命令）
        > 注意 -y 就是所有选项用默认值，所在文件夹不要有中文/特殊符号，建议英文和数字组成，因为 npm 包名限制建议用英文和数字或者下划线中划线
    2.  下载软件包：npm i 软件包名称
    3.  使用软件包
3.  需求：使用 npm 下载 dayjs 软件包到本地项目文件夹中，引入到 index.js 中格式化日期打印，运行观察效果
4.  具体使用流程图：

![](image-20230331155537983_LbotGQ-TUJ.png)

## npm全局软件包-nodemon

1.  软件包区别：
    -   本地软件包：当前项目内使用，作用在**当前项目**，封装的**属性/方法**，供项目调用编写业务需求，封装属性和方法，存在于 node\_modules
    -   全局软件包：本机所有项目使用，作用在**所有项目**，一般封装的**命令/工具**，支撑项目运行，封装命令和工具，存在于系统设置的位置
    ![](image-20230331170539931_UNo17y5yYB.png)
2.  nodemon 作用：替代 node 命令，检测代码更改，自动重启程序
3.  使用：
    1.  安装：npm i nodemon -g （-g 代表安装到全局环境中）
    2.  运行：nodemon 待执行的目标 js 文件
4.  需求：使用 nodemon 命令来启动素材里准备好的项目，然后修改代码保存后，观察终端重启应用程序
