---
layout:  post
title:   SpringBoot中的知识点
date:   22-09-12 09:23:48 修改
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Springboot
-spring boot

---


## yml中变量引用：


![img](https://img-blog.csdnimg.cn/img_convert/f3cad34069accfbf750c5e7ca4f0ff2e.png)

## SpringBoot将所有的配置信息装配到Environment对象中


![img](https://img-blog.csdnimg.cn/img_convert/d484828be3dc27058367cb817e5966ad.png)


![img](https://img-blog.csdnimg.cn/img_convert/c7f294068c965758740974c9e6b74c9f.png)

## SpringBoot封住配置信息为对象：


![img](https://img-blog.csdnimg.cn/img_convert/ee0e76c2b7d05c9f54928abf5ea7c8c6.png)


![img](https://img-blog.csdnimg.cn/img_convert/d574d827f420899308dff69743c9bdd4.png)

## SpringBoot整合MyBatis


![img](https://img-blog.csdnimg.cn/img_convert/6b8d1b8f51d4432f7b43ca1dd5f08881.png)

导入starter


![img](https://img-blog.csdnimg.cn/img_convert/91aab2d2d88f0d3730bedea5fa0a154e.png)


![img](https://img-blog.csdnimg.cn/img_convert/e1c6408dba1db22f7dc940450a0756b4.png)

## 临时属性

打包SpringBoot程序运行之后修改端口


![img](https://img-blog.csdnimg.cn/img_convert/d528c10d77677b6cb9c554b9ca16557a.png)


![img](https://img-blog.csdnimg.cn/img_convert/62eb9b273943e50a1a3dfbe3a761a715.png)


![img](https://img-blog.csdnimg.cn/img_convert/23845569cedd9622e8b807897805714f.png)

## 配置文件分类


![img](https://img-blog.csdnimg.cn/img_convert/25f5eac75d3fd881c08001c6aed0c8f5.png)

## 多环境开发


![img](https://img-blog.csdnimg.cn/img_convert/3dadc761f76b99c39206240dd9f37d2d.png)

## 配置日志级别：

```java
debug: true #默认info，修改为debug
# 整体上是info级别，可以细粒度设置某个包的级别
logging:
  level:
   root: info
   com.liu.controller: debug
```

### 输出到文件


![img](https://img-blog.csdnimg.cn/img_convert/e53f826acc5fc2b0412b935c28845ea5.png)

## @ConfigurationProperties


![img](https://img-blog.csdnimg.cn/img_convert/705e52c9b23c6cdab5d728f8dc433a68.png)

### 配置单位：


![img](https://img-blog.csdnimg.cn/img_convert/23eb413a4998eab110e825783fddf8f4.png)


![img](https://img-blog.csdnimg.cn/img_convert/4435756302269fc65fb998850d538750.png)

## 数据校验


![img](https://img-blog.csdnimg.cn/img_convert/6080876fdfade5ef10fad9c9cc4de62f.png)


![img](https://img-blog.csdnimg.cn/img_convert/1c29edc53c6f3e8b7aa71045964514ee.png)

## SpringBoot进制转换

当配置文件中使用0开头的数字时候，会被转化为八进制计算。

## @Import导入配置类

## SpringBoot发送虚拟请求


![img](https://img-blog.csdnimg.cn/img_convert/094f01fa00c34bec4aa5277bdcab946d.png)

## 测试业务层加@Transaction注解

可以防止产生垃圾数据


![img](https://img-blog.csdnimg.cn/img_convert/86fa90bb26a26c34606aac32b014c572.png)

## 测试生成随机值


![img](https://img-blog.csdnimg.cn/img_convert/05182d615080c0efe92b01c09e0f2c6b.png)


![img](https://img-blog.csdnimg.cn/img_convert/0eec339b2fb5f6780f8aa1293cbd4925.png)

## SpringBoot数据源配置

默认是Hikari数据源


![img](https://img-blog.csdnimg.cn/img_convert/5db6bc00941f1c6b135aa02e3735507a.png)

