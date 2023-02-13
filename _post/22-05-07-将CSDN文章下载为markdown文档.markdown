---
layout:  post
title:   将CSDN文章下载为markdown文档
date:   22-05-07 17:40:56 修改
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-工具
-intellij-idea
-java

---


很多情况下，我们需要将CSDN中的文章转化为markdown文档，直接复制全文是不可以的，CSDN不支持。 那有什么办法快速得到md文档：

## 原理：

>利用jsoup解析网页，将网页中的内容转化为md文档。

直接将文章的url放入文本框，就可以解析获得md文档。 gitee仓库地址：https://gitee.com/liushili888/csdn-is---mark-down 在我上传的资源里面也可以下载到，免费的。

为了简单操作，特意加了Java的GUI界面。

## 使用：

1. 可以将文章url直接放进去，会生成md文档 
2. 也可以将作者的uname放进去，获取作者的所有博客


![img](https://img-blog.csdnimg.cn/05d6bf788d9041e1834433fdd25f04a8.png) 代码就不放这里了，可以看gitee仓库或者我的csdn发布的资源

## 演示：


md文档生成目录 ![img](https://img-blog.csdnimg.cn/a64c22bd11f74960a92a3ff0c31a7ffc.png)

