---
layout:  post
title:   RabbitMQ连接超时Timed out
date:   22-04-05 16:22:58 修改
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-rabbitmq
-java
-linux

---



客户端连接RabbitMQ时出现一下异常信息：

>Exception in thread “main” java.net.ConnectException: Connection timed out: connect

在本地连接云服务器的RabbitMQ时，需要开启5672端口。 原因是没有开放5672端口，这个是RabbitMQ的通信端口 5672：client端通信端口 15672：管理界面ui端口

云服务器放行端口


![img](https://img-blog.csdnimg.cn/8d04ceb33fd54c039a5db8ab7d915709.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

在服务器中开启端口

```java
# 开启5672端口
firewall-cmd --add-port=5672/tcp --permanent 
# 更新防火墙规则
firewall-cmd --reload
```

