---
layout:  post
title:   CentOS7安装VMware Tools详细步骤
date:   1-08-29 17:05:37 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-linux
-centos
-linux

---






![img](https://img-blog.csdnimg.cn/868c10f057e44a55bbfe3df70307369a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 有些小伙伴安装VMware Tools显示灰色，可以看一下这个 [博客](https://blog.csdn.net/cph77777/article/details/79565695)。进入系统之后会发现，多了一个光驱，里面有vm tools ![img](https://img-blog.csdnimg.cn/fd542e302a984577ad790cb77d1dcb23.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)






![img](https://img-blog.csdnimg.cn/33275f20ef1242859152ae5011e82f8d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 点击**其他位置**-&gt;**计算机**-&gt;**opt** ![img](https://img-blog.csdnimg.cn/464a59ceb320431da1a78d294f59088e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) ![img](https://img-blog.csdnimg.cn/eaaf5fc020a94b85b966c3c84ed0f117.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 粘贴到这里 ![img](https://img-blog.csdnimg.cn/f1eed5aa23484cc590e1531a23f0b515.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)




![img](https://img-blog.csdnimg.cn/bf91fb948b794d2e9948e75d2f9fa0d8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 这个窗口相当于windows 的dos窗口，我们在这里使用命令安装vm tools软件 ![img](https://img-blog.csdnimg.cn/82e153cdc312448f838d7bfbc2f5a8a1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)


```java
tar -zxvf VMwareTools-10.3.23-17030940.tar.gz
```





回车 ![img](https://img-blog.csdnimg.cn/dbe6bcbea1394a9a97ff0b9f07d68eaa.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 会发现多了一个文件夹。 ![img](https://img-blog.csdnimg.cn/bee863b10e7147f6b95dc2434963b563.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 使用终端进入这个文件夹 ![img](https://img-blog.csdnimg.cn/6e27b942724d4de28825cd9250047800.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) ![img](https://img-blog.csdnimg.cn/f953a3e43b7b43c2a71d65e2a674f532.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)



使用ls命令查看，有一个后缀.pl的文件，这个文件就是要安装的 ![img](https://img-blog.csdnimg.cn/c5880a0a16294747a1cb1b002023fa3e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 使用命令

```java
./文件名
```


接下来就是一系列回车操作。知道看到这个就说明安装成功了。 ![img](https://img-blog.csdnimg.cn/8ffc44977f3c440aa2cbd5b89111a1ae.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)





![img](https://img-blog.csdnimg.cn/f5ca5714957744839edae923d663accc.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) ![img](https://img-blog.csdnimg.cn/3b4ab3dc524d49229e0230f43c04b954.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 选择要共享的文件夹，我在D盘创建了一个共享文件夹myshare，新建了一个文本测试。 ![img](https://img-blog.csdnimg.cn/264ead6a1e344437b7c595b034b0b07c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)







![img](https://img-blog.csdnimg.cn/124d8f3e28db4b6db782cf1a7c7f40b7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) ![img](https://img-blog.csdnimg.cn/9320e608297d48f19e039cfb82715778.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) ![img](https://img-blog.csdnimg.cn/740d053e515a481e87963cadff26cd4e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 这个文件夹就是我们共享的文件夹了。 ![img](https://img-blog.csdnimg.cn/2f050f38b9d442519d4a4f66d04e8be3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 可以看到我们共享的文件 ![img](https://img-blog.csdnimg.cn/31b3319add5d4c49b5170482d324fe86.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

