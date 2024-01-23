---
title: hexo使用笔记
categories: blog搭建
tags: [blog,踩坑记录]
cover: /picture/blog.jpg
date: 2024-01-23 14:52
---

## 1、文章内加图片（stellar主题）

\_config.yml里配置post\_asset\_folder。

```javascript
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

在\_posts文件夹下新建一个与markdown文件同名的文件夹，放置图片资源。

在markdown文件内，使用'/test01.jpg'的路径引入。

## 2、配色方案

[https://www.colorhunt.co/](https://www.colorhunt.co/ "https://www.colorhunt.co/")

提供赏心悦目的色彩搭配方案，可以直接复制十六进制。

## 3、文章的toc（stellar主题）

默认从H2开始，分级要严格按照2→3→4→5这样降级，否则会显示错误。

## 4、配置文章的默认信息

如果是手动把已经写好的markdown文件拖入post文件夹上传，需要在markdown文件开始的地方配置默认YAML信息，快捷键为三个➖加换行。本博客使用的配置信息格式为：

```javascript
title: hexo使用笔记
categories: blog搭建
tags: [blog,踩坑记录]
cover: /picture/blog.jpg
date: 2024-01-23 14:52
```
