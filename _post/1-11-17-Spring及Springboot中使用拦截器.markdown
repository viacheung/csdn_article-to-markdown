---
layout:  post
title:   Spring及Springboot中使用拦截器
date:   1-11-17 16:18:36 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Springboot
-spring
-java
-java-ee

---



![img](https://img-blog.csdnimg.cn/img_convert/5064e5e7aeeb682a2b7796c72a54b434.png)

开发流程


```java
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>4.0.1</version>
  <scope>provided</scope>
</dependency>
```


```java
public class MyInterceptor implements HandlerInterceptor {
   

    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
   
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
   

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
   

    }
}
```


```java
过滤器不受这个标签限制
<mvc:default-servlet-handler/>

<mvc:interceptors>
  <mvc:intercepter >
    <mvc:mapping path="/**"/>
    //排除静态资源
    <mvc:exclude-mapping path="/resource/**"/>
    <bean class="com.liu.MyInterceptor"/>
  </mvc:intercepter>
</mvc:interceptors>
```


Springboot中使用拦截器，只需要加入web启动依赖。

还是要实现HandlerInterceptor，只不过在配置的时候有所不同。

新建配置类实现WebMvcConfigurer

```java
@Configuration
public class MyAppConfig implements WebMvcConfigurer {
   
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
   
        //创建拦截器对象
        MyInterceptor interceptor=new MyInterceptor();
        //指定拦截的uri
        String[] path ={
   "/user/**"};
        //不拦截的uri
        String[] excludePath ={
   "/user/login"};
        registry.addInterceptor(interceptor).
                addPathPatterns(path).
                excludePathPatterns(excludePath);
    }
}
```



![img](https://img-blog.csdnimg.cn/img_convert/defb67e21d28cf966e905281cbac16f3.png)


使用logback日志保存

加入依赖

```java
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.2.5</version>
</dependency>
```

logback配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%thread] %d %level %logger{10} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="accessHistoryLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>d:/logs/history.%d.log</fileNamePattern>
        </rollingPolicy>
        <encoder>
            <pattern>[%thread] %d %level %logger{10} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="debug">
        <appender-ref ref="console"/>
    </root>
    <logger name="com.imooc.restful.interceptor.AccessHistoryInterceptor"
            level="INFO" additivity="false">
        <appender-ref ref="accessHistoryLog"/>
    </logger>
</configuration>
```

拦截器类

```java
package com.imooc.restful.interceptor;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class AccessHistoryInterceptor implements HandlerInterceptor {
   
    private Logger logger = LoggerFactory.getLogger(AccessHistoryInterceptor.class);

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
   
        StringBuilder log = new StringBuilder();
        log.append(request.getRemoteAddr());
        log.append("|");
        log.append(request.getRequestURL());
        log.append("|");
        log.append(request.getHeader("user-agent"));
        logger.info(log.toString());
        return true;
    }
}
```

springmvc配置拦截规则

```java
<mvc:interceptors>
  <mvc:interceptor>
    <mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/resources/**"/>
    <bean class="com.imooc.restful.interceptor.AccessHistoryInterceptor"/>
  </mvc:interceptor>
</mvc:interceptors>
```



![img](https://img-blog.csdnimg.cn/img_convert/53ac6f9540c5fe539c14fff694ef654b.png)

