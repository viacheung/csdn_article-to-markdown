---
layout:  post
title:   SpringBoot自动配置原理
date:   2-04-02 18:49:29 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Springboot
-SpringMVC
-spring
-idea

---



聊Springboot的自动装配原理，首先要看的就是启动类，在启动的时候发生了什么。Spring Boot的启动类上有一个@SpringBootApplication注解，这个注解是Spring Boot项目必不可少的注解.

```java
@SpringBootApplication
public static void main(String[] args) {
   
        ApplicationContext run = SpringApplication.run(EmosVxApiApplication.class, args);
}
```

启动类中的返回值实际上就是Spring的容器，这在Spring中是一致的。 然后就是看@SpringBootApplication都做了什么工作。 @SpringBootApplication

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {
   @Filter(
    type = FilterType.CUSTOM,
    classes = {
   TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {
   AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
   
}
```

发现SpringBootApplication注解是一个复合注解，上面有三个关键的注解分别是：@SpringBootConfiguration @EnableAutoConfiguration @ComponentScan

## @SpringBootConfiguration


@SpringBootConfiguration实际上是一个配置类注解，被这个注解修饰的注解也是一个配置类。 ![img](https://img-blog.csdnimg.cn/img_convert/e7e016a6606bb7ee3de36ac693fe340c.png)

## @ComponentScan

@ComponentScan这个注解应该都知道吧，就是扫描哪些包，忽略哪些包。 @EnableAutoConfiguration最重要的就是这个注解了。看一下这个注解都包括什么：

```java
@Inherited
@AutoConfigurationPackage
@Import({
   AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
   
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {
   };

    String[] excludeName() default {
   };
}
```

这个注解有一个@Import注解，这个注解有两个作用。 静态加载：将其他类的对象导入IOC容器

```java
@Configuration
@Import({
   Student.class})
public class JavaConfig {
   
    public static void main(String[] args) {
   
        ApplicationContext context =new AnnotationConfigApplicationContext(JavaConfig.class);
        Student student = context.getBean(Student.class);
        System.out.println(student);
    }
}
```


动态加载：如果导入的类实现了ImportSelector类，那么就将ImportSelector类selectImports方法的返回值导入到IOC容器，实际上它的返回值就是类的全限定类名。 ![img](https://img-blog.csdnimg.cn/img_convert/213a5a178da7e3744caec3b3b23d854b.png) 所以我们只要实现这个类，就可以动态的添加一些对象到容器中。

```java
@Import({
   MySelector.class})
public class JavaConfig {
   
    public static void main(String[] args) {
   
        ApplicationContext context =new AnnotationConfigApplicationContext(JavaConfig.class);
        Student student = context.getBean(Student.class);
        System.out.println(context.getBean(Person.class));
        System.out.println(student);
    }
}
```

```java
public class MySelector implements ImportSelector {
   
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
   
        return new String[]{
   Person.class.getName(), Logger.class.getName(), Student.class.getName()};
    }

    @Override
    public Predicate<String> getExclusionFilter() {
   
        return ImportSelector.super.getExclusionFilter();
    }
}
```




![img](https://img-blog.csdnimg.cn/img_convert/a623e94c4965ad21bb0a5a1401723c8b.png) SpringBoot已经为我们实现了，再来看@EnableAutoConfiguration注解。 ![img](https://img-blog.csdnimg.cn/img_convert/9fb1b7e6c06d4068f454b92306fb9250.png) Import为我们导入了AutoConfigurationImportSelector类，看类名大家也应该想到了，这个类实现ImportSelector接口 ![img](https://img-blog.csdnimg.cn/img_convert/20456b1ab4757aa21432cdc3a57d9521.png) 所以这个类也应该实现了接口中的方法，看一下SpringBoot是如何实现的。

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
   
    if (!this.isEnabled(annotationMetadata)) {
   
        return NO_IMPORTS;
    } else {
   
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

我们主要看else里面的代码就可以了。在这个方法中调用了this.getAutoConfigurationEntry(annotationMetadata);这个方法的返回值其实就是我们需要的类的全限定类目。所以我们要看一下它是如何实现的，进入方法查看。

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
   
    if (!this.isEnabled(annotationMetadata)) {
   
        return EMPTY_ENTRY;
    } else {
   
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        //获取所有的候选配置类，其实就是扫描META-INF/spring.factories
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        //排除掉重复的
        configurations = this.removeDuplicates(configurations);
        //排除不需要的 就是我们在@SpringBootApplication(exclude =xxx.class )写的
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        this.checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        //过滤用不到的的配置类
        configurations = this.getConfigurationClassFilter().filter(configurations);
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
    }
}
```


spring.factories会有很多，比如我们引入其他starter，就会有对应的spring.factories ![img](https://img-blog.csdnimg.cn/img_convert/50db8d667e4b54f130257fd040fe1188.png)


![img](https://img-blog.csdnimg.cn/img_convert/65889a296d89ca487e800c7bd45c2d91.png)


我们发现有100多个配置类被扫描到了，但是实际上我们用不到那么多，如果全部加载到内存到话，会占用内存空间，所以怎么样排除我们不需要的呢？看this.getConfigurationClassFilter().filter(configurations);这个方法是如何实现的。 ![img](https://img-blog.csdnimg.cn/img_convert/e7e0b9290e065c82368e2fd3b4ce3804.png)

这个方法很长，我们只看关键的部分，while循环判断所有的配置类，根据this.autoConfigurationMetadata这个条件来判断，这个条件是什么呢？


![img](https://img-blog.csdnimg.cn/img_convert/688b1db3d5898070b7e35bf524f5e0ff.png)

通过加载Metadata来判断，所以继续看什么是Metadata

```java
static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader) {
   
    return loadMetadata(classLoader, "META-INF/spring-autoconfigure-metadata.properties");
}
```


发现它是加载META-INF/spring-autoconfigure-metadata.properties文件进行判断的， ![img](https://img-blog.csdnimg.cn/img_convert/e449f98a28fee1d7517fc85d938d9c9b.png) 现在我们能理解了，配置类是按需加载的。

## 总结

Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作，并且根据当前项目的环境排除一些我们不需要的配置，减少内存消耗。以前我们需要自己配置的东西，自动配置类都帮我们完成了。

