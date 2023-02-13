---
layout:  post
title:   Springboot解决跨域问题的三种方式以及引入SpringSecurity跨域解决方案失效的情况
date:   22-09-12 09:56:57 修改
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Springboot
-java

---


参考： [什么是跨域？跨域解决方法](https://blog.csdn.net/qq_38128179/article/details/84956552)

## 为什么会出现跨域

>出于浏览器的同源策略限制。同源策略（Sameoriginpolicy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。同源策略会阻止一个域的javascript脚本和另外一个域的内容进行交互。所谓同源（即指在同一个域）就是两个页面具有相同的协议（protocol），主机（host）和端口号（port）

## 什么是跨域

>当一个请求url的<strong>协议、域名、端口</strong>三者之间任意一个与当前页面url不同即为跨域


![img](https://img-blog.csdnimg.cn/img_convert/597a6e9d6370ad20c44814bd503e907f.png)

## SpringBoot解决跨域的三种方式

### 第一种：添加@CrossOrigin注解

在Controller层对应的方法上添加@Controller

```java
@RestController
@RequestMapping("index")
public class IndexController {
   

    @GetMapping("/test")
    @CrossOrigin
    public String index() {
   
        return "hello world";
    }
}
```

也可以加在类上

### 第二种：添加CORS过滤器

新建配置类CorsConfig，创建CorsFilter过滤器，允许跨域

```java
@Configuration
public class CorsConfig {
   
    // 跨域请求处理
    @Bean
    public CorsFilter corsFilter() {
   
        CorsConfiguration config = new CorsConfiguration();
        //允许所有域名进行跨域调用
        config.addAllowedOrigin("*");
        //允许所有请求头
        config.addAllowedHeader("*");
        //允许所有方法
        config.addAllowedMethod("*");
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```

### 第三种：实现WebMvcConfigurer，重写addCorsMappings方法

```java
@Configuration
public class CorsConfiguration implements WebMvcConfigurer{
   

    @Override
    public void addCorsMappings(CorsRegistry registry) {
   
        registry.addMapping("/**")
                .allowedOriginPatterns("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                .maxAge(3600);
    }
}
```

### 第四种 引入SpringSecurity之后的解决方案

当项目中引入SpringSecurity，我们前面的解决方案第一种和第三中会失效。这两种方案的原理是使用Spring中的拦截器。我们知道过滤器的执行顺序在拦截器之前。加入SpringSecurity后，就会加入很多过滤器，会导致以前的配置失效。想要第二种方案有效，需要这个过滤器执行时间在SpringSecurity过滤器之前。需要使用@Order指定顺序。但是这种方案需要我们知道SpringSecurity过滤器执行的时机，不利于项目维护。 那么怎么解决呢？ 加入SpringSecurity后，我们可以使用SpringSecurity来解决跨域问题。

```java
public class SecurityConfig extends WebSecurityConfigurerAdapter{
   
@Override
    protected void configure(HttpSecurity http) throws Exception {
   
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and().cors().configurationSource(corsConfigurationSource())
                .and()
                .csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());//将csrf令牌存储在cookie中 允许cookie前端获取
    }

    CorsConfigurationSource corsConfigurationSource(){
   
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedHeaders(Arrays.asList("*"));
        config.setAllowedMethods(Arrays.asList("*"));
        config.setAllowedOrigins(Arrays.asList("*"));
        config.setMaxAge(3600L);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
  }
```

