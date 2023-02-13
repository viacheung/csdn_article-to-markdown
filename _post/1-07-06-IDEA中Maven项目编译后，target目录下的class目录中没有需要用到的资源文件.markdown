---
layout:  post
title:   IDEA中Maven项目编译后，target目录下的class目录中没有需要用到的资源文件
date:   1-07-06 16:32:42 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Java
-java
-maven
-mybatis

---


IDEA中Maven项目编译后，target目录下的class目录中没有需要用到的资源文件 检查是否因自己操作所导致的，具体步骤如下：（以 .xml 资源文件为例）

首先确定资源是不是放在了 resources 资源目录下，该标识的文件才是资源目录；

如果没有放在该目录下，则需要在 pom.xml 文件中配置资源插件，在 build 标签下加入如下代码（如果没有 build 标签创建一个，再将下边代码复制进 build 标签中）

```java
<resources>
  <resource>
    <directory>src/main/java</directory> <!-- .xml资源所在目录 -->
    <includes> <!-- 包括目录下的 .xml 文件-->
      <include>**/*.xml</include>
      </includes>
    <filtering>false</filtering> <!-- 暂时不需要管，加上就是了 -->
  </resource>
</resources>
```

如果上述操作正确，我们的代码没有问题。 那问题就是idea所导致的，有以下几种方法

## 标题方法1：点击 IDEA 右侧栏中的 Maven Projects – 选择你的项目 – Lifecycle – clean（清理掉以前的东西） – compile（重新编译）


![img](https://img-blog.csdnimg.cn/20210706162817673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1Nzc0NjQ1,size_16,color_FFFFFF,t_70)

## 标题方法2：点击IDEA上方的Build – Rebuild Project（重新构建项目）


![img](https://img-blog.csdnimg.cn/20210706162838844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1Nzc0NjQ1,size_16,color_FFFFFF,t_70)

## 标题方法3：File – Invalidate Caches / Restart…（清除无效缓存并重启IDEA）


![img](https://img-blog.csdnimg.cn/20210706162858197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1Nzc0NjQ1,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/20210706162920791.png)

## 标题方法4：直接将资源复制到target目录下的class目录下的相应位置

