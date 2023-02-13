---
layout:  post
title:   Springboot中使用swagger2
date:   2-04-14 18:15:47 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Springboot
-spring
-spring boot

---



```java
<!--swagger-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<!--swagger ui-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
<!--swagger2  增强版接口文档-->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.4</version>
</dependency>
```


```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.ParameterBuilder;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.schema.ModelRef;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Parameter;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.List;

/**
 * Swagger2配置信息
 */
@Configuration
@EnableSwagger2
public class Swagger2Config {
   
    @Bean
    public Docket webApiConfig(){
   

        //添加head参数start
        List<Parameter> pars = new ArrayList<>();
        ParameterBuilder tokenPar = new ParameterBuilder();
        tokenPar.name("userId")
                .description("用户ID")
                .defaultValue("1")
                .modelRef(new ModelRef("string"))
                .parameterType("header")
                .required(false)
                .build();
        pars.add(tokenPar.build());
        
        ParameterBuilder tmpPar = new ParameterBuilder();
        tmpPar.name("userTempId")
                .description("临时用户ID")
                .defaultValue("1")
                .modelRef(new ModelRef("string"))
                .parameterType("header")
                .required(false)
                .build();
        pars.add(tmpPar.build());
        //添加head参数end
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("webApi")
                .apiInfo(webApiInfo())
                .select()
                //可以测试请求头中：输入token
                //.apis(RequestHandlerSelectors.withClassAnnotation(ApiOperation.class))
            	//扫描controller层生成接口文档 需要改成你的
                .apis(RequestHandlerSelectors.basePackage("com.liu.controller"))
                .build()
                .globalOperationParameters(pars);

    }
   
    private ApiInfo webApiInfo(){
   

        return new ApiInfoBuilder()
                .title("网站-API文档")
                .description("本文档描述了网站微服务接口定义")
                .version("1.0")
                .contact(new Contact("liu", "http://liushili.icu", "1728502937@qq.com"))
                .build();
    }
    private ApiInfo adminApiInfo(){
   
        return new ApiInfoBuilder()
                .title("后台管理系统-API文档")
                .description("本文档描述了后台管理系统微服务接口定义")
                .version("1.0")
                .contact(new Contact("liu", "http://liushili.icu", "1728502937@qq.com"))
                .build();
    }
}
```

>注意: 这里使用扫描controller层的方式生成swagger文档，需要改写你的controller包名路径，才能生成swagger文档


测试之前，先了解几个swagger提供的注解

>@Api：修饰整个类，描述Controller的作用<br> @ApiOperation：描述一个类的一个方法，或者说一个接口<br> @ApiParam：单个参数描述<br> @ApiModel：用对象来接收参数<br> @ApiModelProperty：用对象接收参数时，描述对象的一个字段<br> @ApiImplicitParam：一个请求参数<br> @ApiImplicitParams：多个请求参数

在com.liu.controller中新建TestController类

```java
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiParam;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@Api(tags = "测试接口")
@RestController
public class TestController {
   

    @GetMapping("/test")
    public String test(@ApiParam("用户id") Integer id) {
   
        return "hello swagger2";
    }

}
```

启动springboot工程，在浏览器输入

http://localhost:xxx/swagger-ui.html就可以看到生成的接口文档了。


![img](https://img-blog.csdnimg.cn/img_convert/ff2383746bf4ea3264a0b08c66271e6c.png)

这样就可以算集成完成了。

