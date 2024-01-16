---
title: hexo+stellar+github actions实现自动部署
categories: blog搭建
tags: [blog,踩坑记录]
cover: /picture/blog.jpg
date: 2024-01-16 20:49
---

## 博客的持续部署

抛开定义，直观上，持续部署，顾名思义，就是持续不断地去部署，部署自动紧跟代码改变：你的提交了源码修改，部署上就自动更新了。对于我们的博客系统，也就是新建/修改/删除了文章，博客站点就自动更新、修改对应内容。从效果上来说，就是我们不用再去手动 `hexo g -d` 生成、部署了。

用持续部署，首先提交源码，然后在云端就自动生成(编译)、部署，这个生成、部署的工作是不需要在本地完成的，由github提供的 CI/CD 服务的服务器自动来完成。GitHub 免费提供的这项服务叫做 [GitHub Actions](https://github.com/features/actions "GitHub Actions")。

## 1. 前提工作

本教程在以下环境搭建。

```
hexo: 6.3.0
hexo-cli: 4.3.0
os: win32 10.0.19042
node: 16.17.1
hexo-deployer-git: 4.0.0
```

### 1.1创建所需仓库

1.  创建 `blog` 仓库用来存放 Hexo 项目
2.  创建 `your.github.io` 仓库用来存放静态博客页面

### 1.2生成部署密钥

```javascript

$ ssh-keygen -f github-deploy-key
```

一路按回车直到生成成功

当前目录下会有 `github-deploy-key` 和 `github-deploy-key.pub` 两个文件。

### 1.3 配置部署密钥

复制 `github-deploy-key` 文件内容，在 `blog` 仓库 `Settings -> Secrets and variables -> Actions` 页面上添加。

1.  在 `Name` 输入框填写 `HEXO_DEPLOY_PRI`。
2.  在 `Value` 输入框填写 `github-deploy-key` 文件内容。

复制 `github-deploy-key.pub` 文件内容，在 `your.github.io` 仓库 `Settings -> Deploy keys -> Add deploy key` 页面上添加。

1.  在 `Title` 输入框填写 `HEXO_DEPLOY_PUB`。
2.  在 `Key` 输入框填写 `github-deploy-key.pub` 文件内容。
3.  勾选 `Allow write access` 选项。

## 2. 配置workflow

在 `blog` 仓库根目录下创建 `.github/workflows/deploy.yml` 文件。

在 `deploy.yml` 文件中粘贴以下内容。

配置文件

```javascript
name: CI

on:
  push:
    branches:
      - main

env:
  GIT_USER: YYYutang
  GIT_EMAIL: 1036503272@qq.com
  THEME_REPO: YYYutang/hexo-theme-stellar
  THEME_BRANCH: main
  DEPLOY_REPO: YYYutang/YYYutang.github.io
  DEPLOY_BRANCH: main

jobs:
  build:
    name: Build on node ${{ matrix.node_version }} and ${{ matrix.os }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os: [ubuntu-latest]
        node_version: [16.x]

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Checkout theme repo
        uses: actions/checkout@v2
        with:
          repository: ${{ env.THEME_REPO }}
          ref: ${{ env.THEME_BRANCH }}
          path: themes/stellar

      - name: Checkout deploy repo
        uses: actions/checkout@v2
        with:
          repository: ${{ env.DEPLOY_REPO }}
          ref: ${{ env.DEPLOY_BRANCH }}
          path: .deploy_git

      - name: Use Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node_version }}

      - name: Configuration environment
        env:
          HEXO_DEPLOY_PRI: ${{secrets.HEXO_DEPLOY_PRI}}
        run: |
          sudo timedatectl set-timezone "Asia/Shanghai"
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name $GIT_USER
          git config --global user.email $GIT_EMAIL
          cp _config.theme.yml themes/stellar/_config.yml

      - name: Install dependencies
        run: |
          npm install

      - name: Deploy hexo
        run: |
          npm run deploy
```

### 模版参数说明

-   *name* 为此 Action 的名字
-   *on* 触发条件，当满足条件时会触发此任务，这里的 `on.push.branches.$.master` 是指当 `master` 分支收到 `push` 后执行任务。
-   *env* 为环境变量对象
    -   *env.GIT\_USER* 为 Hexo 编译后使用此 git 用户部署到仓库。
    -   *env.GIT\_EMAIL* 为 Hexo 编译后使用此 git 邮箱部署到仓库。
    -   *env.THEME\_REPO* 为您的 Hexo 所使用的主题的仓库，这里为 `sanonz/hexo-theme-concise`。
    -   *env.THEME\_BRANCH* 为您的 Hexo 所使用的主题仓库的版本，可以是：branch、tag 或者 SHA。
    -   *env.DEPLOY\_REPO* 为 Hexo 编译后要部署的仓库，例如：`sanonz/sanonz.github.io`。
    -   *env.DEPLOY\_BRANCH* 为 Hexo 编译后要部署到的分支，例如：master。
-   *jobs* 为此 Action 下的任务列表
    -   *jobs.{job}.name* 任务名称
    -   *jobs.{job}.runs-on* 任务所需容器，可选值：`ubuntu-latest`、`windows-latest`、`macos-latest`。
    -   *jobs.{job}.strategy* 策略下可以写 `array` 格式，此 job 会遍历此数组执行。
    -   *jobs.{job}.steps* 一个步骤数组，可以把所要干的事分步骤放到这里。
        -   *jobs.{job}.steps.\$.name* 步骤名，编译时会会以 LOG 形式输出。
        -   *jobs.{job}.steps.\$.uses* 所要调用的 Action，可以到 [https://github.com/actions](https://github.com/actions "https://github.com/actions") 查看更多。
        -   *jobs.{job}.steps.\$.with* 一个对象，调用 Action 传的参数，具体可以查看所使用 Action 的说明。

这里踩的坑有：

1、path: themes/stellar的主题地址忘记换成自己的，导致部署上去之后，所有的html文件均为空文件，并且页面加载不出来。

2、部署成功，但在workflow里deploy那一步有大量报错，说找不到avatar属性，一开始以为是部署流程只识别了\_config.theme.yml,所以没有识别\_config.yml里设置的avatar，遂修改\_config.theme.yml，但没有起作用。

仔细检查发现主题是通过THEME\_REPO设置的github连接引入的，而我github上fork的是一个比较早的版本的stellar，本地使用的\_config.theme.yml是最新版本的stellar里提供的配置文件。将github上的主题仓库更新至最新版，问题解决。

![](image_XvDrAt19Cq.png)

3、报大量的语法错误

![](image_TGTchTWkX1.png)

是指定的node\_version过低的问题，最新版的stellar不适配12.x的版本，修改为15.x以上即可。

4、报错ERROR Deployer not found: git，需要安装 hexo-deployer-git。

```
 npm install hexo-deployer-git --save
```

## 3. 修改\_config.yml里的deploy配置

```javascript
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: 'git'
  repo: git@github.com:yyyutang/YYYutang.github.io.git
  branch: main
  message: ${{ github.event.head_commit.message }}
```

## 4. 配置theme的\_config文件

复制一份 使用的theme根目录下的\_config.yml，放到 `blog` 根目录下，名为 `_config.theme.yml`，如果您已经配置过此文件，只需要把您的复制过来就行。

这里注意需要把网站的基础信息，如头像Avatar、标题Title等设置好。

## 5.执行任务

写一篇文章，`push` 到 `blog` 仓库的 `main` 分支，在此仓库 `Actions` 页面查看当前 task。

当任务完成后查看您的博客 `https://your.github.io`，如果不出意外的话已经可以看到新添加的文章了。
