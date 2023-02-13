---
layout:  post
title:   Juc19_从字节码角度看synchronize、Monitor类、monitorenter、monitorexit、深入理解同步方法
date:   22-03-31 22:21:26 修改
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JUC并发编程
-容器
-kubernetes
-docker

---








-  ①. synchronized属于重量级锁,效率低下,因为监视器锁(monitor)是依赖于底层的操作系统的Mutex Lock来实现的,挂起线程和恢复线程都需要转入内核态去完成,阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成,这种状态切换需要耗费处理器时间,如果同步代码块中内容过于简单,这种切换的时间可能比用户代码执行的时间还长”,时间成本相对较高,这也是为什么早期的synchronized效率低的原因,后期进行了锁的优化,锁升级,对应偏向锁、轻量级锁的讲解会在下一节进行叙述  
-  ②. synchronized有三种应用方式 (反编译:javap -v -p *.class > 类.txt 将进行输出到txt中) 

1. 作用于代码块,对括号里配置的对象加锁 
2. 作用于实例方法,当前实例加锁,进入同步代码前要获得当前实例的锁 
3. 作用于静态方法,当前类加锁,进去同步代码前要获得当前类对象的锁

- ③. synchronized同步代码块



1. 实现使用的是monitorenter和monitorexit指令 ![img](https://img-blog.csdnimg.cn/20210314181051606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70) 
<li>一定是一个enter和两个exit吗? (不一定,如果方法中直接抛出了异常处理,那么就是一个monitorenter和一个monitorexit) ![img](https://img-blog.csdnimg.cn/20210314181153474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)</li>



- ④. synchronized普通同步方法 (调用指令将会检查方法的ACC_SYNCHRONIZED访问标志是否被设置,如果设置了,执行线程会将先持有monitor然后再执行方法,最后再方法完成(无论是正常完成还是非正常完成)时释放minotor) ![img](https://img-blog.csdnimg.cn/2021031418130816.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70) 
- ⑤. synchronized静态同步方法 (ACC_STATIC、ACC_SYNCHRONIZED访问标志区分该方法是否静态同步方法) ![img](https://img-blog.csdnimg.cn/20210314181650733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)


-  ①. 任何一个对象都可以成为一个锁,在HotSpot虚拟机中,monitor采用ObjectMonitor实现  
-  ②. 上述C++源码解读 


1. ObjectMonitor.java — ObjectMonitor.cpp — ObjectMonitor.hpp 
2. ObjectMonitor.hpp(底层源码解析) ![img](https://img-blog.csdnimg.cn/20210314181905750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)

- ③. 阿里开发手册说明 高并发时,同步调用应该去考量锁的性能损耗。能用无锁数据结构,就不要用锁;能锁区块,就不要锁整个方法体;能用对象锁,就不要用类锁


-  ①. 前言:是在Java中万物皆是对象,所以这些指令(monitorenter、 monitorexit)会不会和某些对象有关系呢？ 果然可以和一个叫Monitor类联系到一块  
-  ②. monitor相当于一个对象的钥匙,只有拿到此对象的monitor,才能访问该对象的同步代码。相反未获得monitor的只能阻塞来等待持有monitor的线程释放monitor。可以这样比喻吧,monitorenter 和monitorexit 对应的就是拿钥匙和还钥匙  
-  ③. Monitor与java对象以及线程是如何关联 ？ 

1. 首先,每一个对象都有一个属于自己的monitor,其次如果线程未获取到singal (许可),则线程阻塞。object可以比作医院的诊室,monitor 就是负责喊病人的护士,线程则是就诊的病人。 
2. 通过护士(监视器)的调度,诊室(synchronized锁住的对象)内只允许进入一个病人(线程),此病人(线程)在当前时间就拥有此诊室(对象)的使用权,也就是获取了许可。病人就诊完毕,则表明归还了诊室的使用权。然后护士再调度下一个等待的病人进入诊室(被阻塞的线程)。走廊当中等待的病人们

- ④. 通过上面两段描述,我们应该能很清楚的看出Synchronized的实现原理,Synchronized的语义底层是通过一个monitor的对象来 完成,其实wait/notify等方法也依赖于monitor对象,这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法,否则会抛出java.lang.IllegalMonitorState Exception的异常的原因


- ①. 每一个对象都会和一个监视器monitor关联。监视器被占用时会被锁住,其他线程无法来获取该monitor。当JVM执行某个线程的某个方法内部的monitorenter时,它会尝试去获取当前对象对应的monitor的所有权。其过程如下:

1. 若monior的进入数为0,线程可以进入monitor,并将monitor的进数置为1。当前线程成为monitor的owner(所有者) 
2. 若线程已拥有monitor的所有权,允许它重入monitor,则进入monitor的进入数加1 
3. 若其他线程已经占有monitor的所有权,那么当前尝试获取monitor的所有权的线程会被阻塞,直到monitor的进入数变为0,才能重新尝试获取monitor的所有权

- ②. synchronized的锁对象会关联一个monitor,这个monitor不是我们主动创建的,是JVM的线程执行到这个同步代码块,发现锁对象没有monitor就会创建monitor,monitor内部有两个重要的成员变量owner:拥有这把锁的线程,recursions会记录线程拥有锁的次数,当一个线程拥有monitor后其他线程只能等待


-  ①. 能执行monitorexit指令的线程一定是拥有当前对象的monitor的所有权的线程  
-  ②. 执行monitorexit时会将monitor的进入数减1。当monitor的进入数减为0时,当前线程退出monitor,不再拥有monitor的所有权,此时其他被这个monitor阻塞的线程可以尝试去获取这个monitor的所有权  
-  ③. monitorexit释放锁 (monitorexit,指令出现了两次,第1次为同步正常退出释放锁；第2次为发生异步退出释放锁) monitorexit插入在方法结束处和异常处,JVM保证每个monitorenter必须有对应的monitorexit  
-  ④. 面试题synchroznied出现异常会释放锁吗? 会释放锁 


-  ①. synchronized普通同步方法 (调用指令将会检查方法的ACC_SYNCHRONIZED访问标志是否被设置,如果设置了,执行线程会将先持有monitor然后再执行方法,最后再方法完成(无论是正常完成还是非正常完成)时释放minotor)  
-  ②. 从编译的结果来看,方法的同步并没有通过指令monitorenter和monitorexit来完成(理论上其实也可以通过这两条指令来实现),不过相对于普通方法,其常量池中多了ACC_SYNCHRONIZED标示符。JVM就是根据该标示符来实现方法的同步的。当方法调用时,调用指令将会检查方法的 ACC_SYNCHRONIZED访问标志是否被设置,如果设置了,执行线程将先获取monitor,获取成功之后才能执行方法体,方法执行完后再释放monitor。在方法执行期间,其他任何线程都无法再获得同一个monitor对象  
-  ③. 两种同步方式本质上没有区别,只是方法的同步是一种隐式的方式来实现,无需通过字节码来完成。两个指令的执行是JVM通 过调用操作系统的互斥原语mutex来实现,被阻塞的线程会被挂起、等待重新调度,会导致"用户态和内核态"两个态之间来回切换,对性能有较大影响 

