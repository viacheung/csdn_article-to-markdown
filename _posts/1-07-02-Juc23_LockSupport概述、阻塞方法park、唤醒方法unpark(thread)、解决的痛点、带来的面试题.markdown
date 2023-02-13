---
layout:  post
title:   Juc23_LockSupport概述、阻塞方法park、唤醒方法unpark(thread)、解决的痛点、带来的面试题
date:   1-07-02 09:05:30 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JUC并发编程

---








-  ①. 通过park()和unpark(thread)方法来实现阻塞和唤醒线程的操作  
-  ②. LockSupport是一个线程阻塞工具类,所有的方法都是静态方法,可以让线程在任意位置阻塞,阻塞之后也有对应的唤醒方法。归根结底,LockSupport调用的Unsafe中的native代码  
-  ③. 官网解释: LockSupport是用来创建锁和其他同步类的基本线程阻塞原语 LockSupport类使用了一种名为Permit(许可)的概念来做到阻塞和唤醒线程的功能,每个线程都有一个许可(permit),permit只有两个值1和零,默认是零 可以把许可看成是一种(0,1)信号量(Semaphore),但与Semaphore不同的是,许可的累加上限是1 


-  ①. permit默认是0,所以一开始调用park()方法,当前线程就会阻塞,直到别的线程将当前线程的permit设置为1时, park方法会被唤醒,然后会将permit再次设置为0并返回  
-  ②. static void park( ):底层是unsafe类native方法  
-  ③. static void park(Object blocker) 


![img](https://img-blog.csdnimg.cn/20201023152750416.png#pic_center)


-  ①. 调用unpark(thread)方法后,就会将thread线程的许可permit设置成1(注意多次调用unpark方法,不会累加,permit值还是1)会自动唤醒thread线程,即之前阻塞中的LockSupport.park()方法会立即返回  
-  ②. static void unpark( ) 


-  ①. LockSupport不用持有锁块,不用加锁,程序性能好  
-  ②. 先后顺序,不容易导致卡死(因为unpark获得了一个凭证,之后再调用park方法,就可以名正言顺的凭证消费,故不会阻塞)  
-  ③. 代码演示: 

```java
/*
(1).阻塞
 (permit默认是O,所以一开始调用park()方法,当前线程就会阻塞,直到别的线程将当前线程的permit设置为1时,
 park方法会被唤醒,然后会将permit再次设置为O并返回)
 static void park()
 static void park(Object blocker)
(2).唤醒
static void unpark(Thread thread)
 (调用unpark(thread)方法后,就会将thread线程的许可permit设置成1(注意多次调用unpark方法,不会累加,
 permit值还是1)会自动唤醒thread线程,即之前阻塞中的LockSupport.park()方法会立即返回)
 static void unpark(Thread thread)
* */
public class LockSupportDemo {
   
    public static void main(String[] args) {
   

        Thread t1=new Thread(()->{
   
            System.out.println(Thread.currentThread().getName()+"\t"+"coming....");
            LockSupport.park();
            /*
            如果这里有两个LockSupport.park(),因为permit的值为1,上一行已经使用了permit
            所以下一行被注释的打开会导致程序处于一直等待的状态
            * */
            //LockSupport.park();
            System.out.println(Thread.currentThread().getName()+"\t"+"被B唤醒了");
            },"A");
        t1.start();

        //下面代码注释是为了A线程先执行
        //try { TimeUnit.SECONDS.sleep(3);  } catch (InterruptedException e) {e.printStackTrace();}

        Thread t2=new Thread(()->{
   
            System.out.println(Thread.currentThread().getName()+"\t"+"唤醒A线程");
            //有两个LockSupport.unpark(t1),由于permit的值最大为1,所以只能给park一个通行证
            LockSupport.unpark(t1);
            //LockSupport.unpark(t1);
        },"B");
        t2.start();
    }
}
```


-  ①. 为什么可以先唤醒线程后阻塞线程?(因为unpark获得了一个凭证,之后再调用park方法,就可以名正言顺的凭证消费,故不会阻塞)  
-  ②. 为什么唤醒两次后阻塞两次,但最终结果还会阻塞线程?(因为凭证的数量最多为1,连续调用两次unpark和调用一次unpark效果一样,只会增加一个凭证;而调用两次park却需要消费两个凭证,证不够,不能放行) 

