---
layout:  post
title:   Juc18_Java内存模型、对象头Mark Word、实例数据、对齐填充、谈谈new Object( )占多大内存
date:   1-06-23 09:09:36 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JUC并发编程

---








-  ①. 对象内部结构分为:对象头、实例数据、对齐填充(保证8个字节的倍数)。  
-  ②. 对象头分为对象标记(markOop)和类元信息(klassOop),类元信息存储的是指向该对象类元数据(klass)的首地址。 


![img](https://img-blog.csdnimg.cn/20210406193020801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)


## ①. 对象标记Mark Word

- ①. 默认存储 (哈希值(HashCode )、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳)等信息


1. 这些信息都是与对象自身定义无关的数据,所以MarkWord被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。 
2. 它会根据对象的状态复用自己的存储空间,也就是说在运行期间MarkWord里存储的数据会随着锁标志位的变化而变化。 ![img](https://img-blog.csdnimg.cn/20210406193237787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)


- ②. 在64位系统中,Mark Word占了8个字节,类型指针占了8个字节,一共是16个字节 ![img](https://img-blog.csdnimg.cn/20210406193345417.png)

## ②. 类元信息(又叫类型指针)

- 对象指向它的类元数据的指针,虚拟机通过这个指针来确定这个对象是哪个类的实例

## ③. 对象头多大

- ①. 在64位系统中,Mark Word占了8个字节,类型指针占了8个字节,一共是16个字节


- 说明:它是对象真正存储的有效信息,包括程序代码中定义的各种类型的字段(包括从父类继承下来的和本身拥有的字段) 规则:

1. 相同宽度的字段总被分配在一起 
2. 父类中定义的变量会出现在子类之前 
3. 如果CompactFields参数为true(默认为true),子类的窄变量可能插入到父类变量的空隙


-  ①. 虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的,仅仅是为了字节对齐。这部分内存按8字节补充对齐。  
-  ②. 不是必须的,也没特别含义,仅仅起到占位符作用  
-  ③. 解释如下图: 


![img](https://img-blog.csdnimg.cn/20200825093157382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70#pic_center)


- ①. 代码演示

```java
public class CustomerTest {
   
    public static void main(String[] args) {
   
        Customer cust = new Customer();
    }
}
```


- ②. 图解👆代码 ![img](https://img-blog.csdnimg.cn/20200825093342514.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70#pic_center)





- ①. 32位(看一下即可,不用学了,以64位为准) ![img](https://img-blog.csdnimg.cn/20210406194316778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70) 
- <font color="red">②. 64位重要 markword(64位)分布图,对象布局、GC回收和后面的锁升级就是:对象标记MarkWord里面标志位的变化 <img src="https://img-blog.csdnimg.cn/20210406194338481.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> <img src="https://img-blog.csdnimg.cn/20210406194431981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"></font> 
- ③. oop.hpp ![img](https://img-blog.csdnimg.cn/2021040619452321.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70) 
- ④. markOop.hpp hash: 保存对象的哈希码 age: 保存对象的分代年龄 biased_lock: 偏向锁标识位 lock: 锁状态标识位 JavaThread* :保存持有偏向锁的线程ID epoch: 保存偏向时间戳 ![img](https://img-blog.csdnimg.cn/2021040619453782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)


- ①. 引入pom,代码如下

```java
<!--
	JAVA object layout
	官网:http://openjdk.java.net/projects/code-tools/jol/
	定位:分析对象在JVM的大小和分布
	-->
	<dependency>
	    <groupId>org.openjdk.jol</groupId>
	    <artifactId>jol-core</artifactId>
	    <version>0.9</version>
	</dependency>
```

```java
public class JOLDemo{
   
    public static void main(String[] args){
   
        Object o = new Object();
        System.out.println( ClassLayout.parseInstance(o).toPrintable());
    }
}
```

- ②. 结果呈现说明


![img](https://img-blog.csdnimg.cn/20210406194855666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/20210406194839664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)

- ③. GC年龄采用4位bit存储,最大为15,例如MaxTenuringThreshold参数默认值就是15 -XX:MaxTenuringThreshold=16 因为GC年龄占4位,最大就是1111=15


![img](https://img-blog.csdnimg.cn/20210406195102482.png)



- ④. 手动关闭压缩再看看 -XX:-UseCompressedClassPointers ![img](https://img-blog.csdnimg.cn/2021040619523912.png)![img](https://img-blog.csdnimg.cn/20210406195529908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/20210406195400750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)

- ⑤. 说到这里,如果Object是个空对象,那么new Object 就是占16个字节

