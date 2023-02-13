---
layout:  post
title:   An attempt was made to call a method that does not exist. The attempt was ma
date:   2-04-20 11:31:35 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-MyBatis

---



SpringBoot整合MyBatis出错。 ![img](https://img-blog.csdnimg.cn/cf1a889e05f4481182b7b98f8c7aefbb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 原因：我同时引入了这两个依赖。

```java
<dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis</artifactId>
           <version>3.2.2</version>
</dependency>
 <dependency>
           <groupId>org.mybatis.spring.boot</groupId>
           <artifactId>mybatis-spring-boot-starter</artifactId>
           <version>2.1.2</version>
</dependency>
```

删除

```java
<dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis</artifactId>
           <version>3.2.2</version>
</dependency>
```

然后就好了，不太明白为什么这么做

