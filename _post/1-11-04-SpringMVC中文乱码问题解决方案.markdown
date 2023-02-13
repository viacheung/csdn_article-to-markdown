---
layout:  post
title:   SpringMVC中文乱码问题解决方案
date:   1-11-04 18:47:40 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-SpringMVC
-tomcat
-java
-服务器

---




浏览器向服务器请求 ![img](https://img-blog.csdnimg.cn/ea5c3464b4fd489dac0e0cfc457c828f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

## Get请求乱码


修改Tomcat 文件夹conf 中server.xml配置文件 ![img](https://img-blog.csdnimg.cn/745049fca025445f92a559a07be14edd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

```java
在这个标签中添加URIEncoding="UTF-8" 属性 ，网址请求中乱码
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8" />
```

注意：在Tomcat8.0以后，默认就是UTF-8

## Post请求乱码

修改web.xml

```java
<filter>
  <filter-name>characterEncodingFilter</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
  <init-param>
    <param-name>forceEncoding</param-name>
    <param-value>true</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>characterEncodingFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

注意：这里为什么使用/*，而不是/,这两个有什么区别？

>其中/和/<em>的区别：<br> &lt; url-pattern &gt; / &lt;/ url-pattern &gt; 不会匹配到</em>.jsp，即：<em>.jsp不会进入spring的 DispatcherServlet类 。<br> &lt; url-pattern &gt; /</em> &lt;/ url-pattern &gt; 会匹配*.jsp，会出现返回jsp视图时再次进入spring的DispatcherServlet 类，导致找不到对应的controller所以报404错。<br> ​

>总之，关于web.xml的url映射的小知识:<br> &lt; url-pattern&gt;/ 会匹配到/login这样的路径型url，不会匹配到模式为*.jsp这样的后缀型url<br> &lt; url-pattern&gt;/<em> 会匹配所有url：路径型的和后缀型的url(包括/login,</em>.jsp,<em>.js和</em>.html等)

我们编写servlet程序时，会为每一个自定义的servlet配置一个url-pattern，使得它可以处理对应的请求。但是我们是否想过，当我们请求jsp页面或者html、css、js时，这些请求是如何处理的呢？ 这些请求的处理就是由tomcat的内置servlet来完成的，所以如果使用/*，就不会使用Tomcat默认sevlet处理这些请求，然而SpringMVC又没有这些servlet，所以访问这些静态资源会出错。 ​


服务器向浏览器响应 ​

在SpringMVC配置文件中设置

```java
<mvc:annotation-driven> //启用Spring MVC注解开发模式
      <mvc:message-converters>//响应类型转换器，可配置多个
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
          <property name="supportedMediaTypes">
            <list>
              //相当于在servlet中写的response.setContentType("text/html;charset=utf-8")
              <value>text/plain;charset=UTF-8</value>
              <value>text/html;charset=UTF-8</value>
              <value>application/json;charset=UTF-8</value>
            </list>
          </property>
        </bean>
      </mvc:message-converters>
  </mvc:annotation-driven>
```

>注意：<br> 如果你使用的spring4.3及后续版本，且返回值为json，如果全局配置，只需要在xml中配置&lt;mvc:annotation-driven /&gt; 即可，中文不会乱码

