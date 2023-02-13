---
layout:  post
title:   使用Session+Cookie实现7天免登录
date:   2-03-16 22:05:06 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-服务器
-前端
-http

---


>session存在于服务器端，cookie存在于浏览器端。

如果我们在服务器端使用Session保存用户信息，那么在用户登录某个页面时，访问服务器的时候，就会生成一个cookie返回到浏览器端，如果没有设置cookie的过期时间，浏览器关闭的时候cookie失效。

关闭浏览器重新打开这个网站，就要重新登录，这是用户不愿意看到的。

在浏览器没有关闭的情况下，用户访问服务器，是不需要频繁登录的。服务器是通过浏览器携带的cookie找到对应的session对象。具体就是根据cookie的JSESSIONID对应的值，找到服务器得session。


具体的原理如图： ![img](https://img-blog.csdnimg.cn/1f6fb30c327b46cdbe010ccd10679bc7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

所以我们可以设置cookie得过期时间，就算浏览器关闭，cookie也不会失效。

```java
Cookie cookie = new Cookie("JSESSIONID", req.getSession().getId());
cookie.setMaxAge(60 * 60 * 24 * 7);
session.setMaxInactiveInterval(60 * 60 * 24 * 7);
//默认情况下，如果不设置Cookie的path，默认是“/项目名/当前路径的上一层地址”
//我这里省略，因为我在http://localhost:8080/userLogin 根路径下，直接绑定了根路径
resp.addCookie(cookie);
```

这样当我们关闭浏览器，如果cookie没有过期，我们就不需要登录，因为浏览器会携带cookie发送给服务器，服务器帮我们解析找到session，判断用户身份。

