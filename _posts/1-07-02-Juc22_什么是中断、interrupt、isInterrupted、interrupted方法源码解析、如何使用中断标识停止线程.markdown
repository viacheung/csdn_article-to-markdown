---
layout:  post
title:   Juc22_什么是中断、interrupt、isInterrupted、interrupted方法源码解析、如何使用中断标识停止线程
date:   1-07-02 08:51:41 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JUC并发编程

---








-  ①. 一个线程不应该由其他线程来强制中断或停止,而是应该由线程自己自行停止,所以,Thread.stop、Thread.suspend、Thread. resume都已经被废弃了  
-  ②. 在Java中没有办法立即停止一条线程,然而停止线程却显得尤为重要,如取消一个耗时操作。因此,Java提供了一种用于停止线程的机制 — 中断  
-  ③. 中断只是一种协作机制,Java没有给中断增加任何语法,中断的过程完全需要程序员自己实现  
-  ④. 若要中断一个线程,你需要手动调用该线程的interrupt方法,该方法也仅仅是将线程对象的中断标识设为true  
-  ⑤. 每个线程对象中都有一个标识,用于标识线程是否被中断;该标识位为true表示中断,为false表示未中断;通过调用线程对象的interru pt方法将线程的标识位设为true;可以在别的线程中调用,也可以在自己的线程中调用 


- ①. void interrupt( )实例方法

1. interrupt( )仅仅是设置线程的中断状态未true,不会停止线程 
2. 源码解读 (如果这个线程因为wait()、join()、sleep()方法在用的过程中被打断(interupt),会抛出Interrupte dException)

- ②. boolean isInterrupted( )实例方法 判断当前线程是否被中断(通过检查中断标识位) 实例方法 
- ③. static boolean interrupted( )静态方法

1. 判断线程是否被中断,并清楚当前中断状态,这个方法做了两件事 (返回当前线程的中断状态 | 将当前线程的中断状态设为false) 
2. 原理:假设有两个线程A、B,线程B调用了interrupt方法,这个时候我们连接调用两次isInterrupted方法,第一次会返回true,然后这个方法会将中断标识位设置位false,所以第二次调用将返回false

```java
System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
  System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
  System.out.println("111111");
  Thread.currentThread().interrupt();///----false---> true
  System.out.println("222222");
  System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
  System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
  /**
   main---false
   main---false
   111111
   222222
   main---true
   main---false
   * */
```

- ④. 比较静态方法interrupted和实例方法isInterrupted

1. 静态方法interrupted将会清除中断状态(传入的参数ClearInterrupted位true) 
2. 实例方法isInterrupted则不会(传入的参数ClearInterrupted为false)


-  ①. 在需要中断的线程中不断监听中断状态,一旦发生中断,就执行型对于的中断处理业务逻辑  
-  ②. 三种中断标识停止线程的方式 

1. 通过一个volatile变量实现 
2. 通过AtomicBoolean 
3. 通过Thread类自带的中断API方法实现

```java
public class InterruptDemo{
   
    static volatile boolean isStop = false;
    static AtomicBoolean atomicBoolean = new AtomicBoolean(false);

    public static void m3(){
   
    Thread t1 = new Thread(() -> {
   
        while (true) {
   
            if (Thread.currentThread().isInterrupted()) {
   
                System.out.println("-----isInterrupted() = true,程序结束。");
                break;
            }
            System.out.println("------hello Interrupt");
        }
    }, "t1");
    t1.start();

    try {
    TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) {
    e.printStackTrace(); }

    new Thread(() -> {
   
        t1.interrupt();//修改t1线程的中断标志位为true
    },"t2").start();
}

    /**
     * 通过AtomicBoolean
     */
    public static void m2(){
   
        new Thread(() -> {
   
            while(true)
            {
   
                if(atomicBoolean.get())
                {
   
                    System.out.println("-----atomicBoolean.get() = true,程序结束。");
                    break;
                }
                System.out.println("------hello atomicBoolean");
            }
        },"t1").start();

        //暂停几秒钟线程
        try {
    TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) {
    e.printStackTrace(); }

        new Thread(() -> {
   
            atomicBoolean.set(true);
        },"t2").start();
    }

    /**
     * 通过一个volatile变量实现
     */
    public static void m1(){
   
        new Thread(() -> {
   
            while(true)
            {
   
                if(isStop)
                {
   
                    System.out.println("-----isStop = true,程序结束。");
                    break;
                }
                System.out.println("------hello isStop");
            }
        },"t1").start();

        //暂停几秒钟线程
        try {
    TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) {
    e.printStackTrace(); }

        new Thread(() -> {
   
            isStop = true;
        },"t2").start();
    }
}
```

- ③. 当前线程的中断标识为true,是不是就立刻停止？

1. 线程调用interrupt()时 (①. 如果线程处于正常活动状态,那么会将线程的中断标志设置位true,仅此而已。被设置中断标志的线程将继续正常运行,不受影响。所以,interrupt( )并不能真正的中断线程,需要被调用的线程自己进行配合才行②. 如果线程处于被阻塞状态(例如处于sleep、wait、join等状态),在别的线程中调用当前线程对象的interrupt方法,那么线程立即被阻塞状态,并抛出一个InterruptedException异常) 
2. 中断只是一种协同机制,修改中断标识位仅此而已,不是立即stop打断 
3. sleep方法抛出InterruptedException后,中断标识也被清空置为false,我们在catch没有通过调用th.interrupt( )方法再次将中断标识位设置位true,这就是导致无限循环了

```java
public static void m5()
    {
   
        Thread t1 = new Thread(() -> {
   
            while (true) {
   
                if (Thread.currentThread().isInterrupted()) {
   
                    System.out.println("-----isInterrupted() = true,程序结束。");
                    break;
                }
                try {
   
                    Thread.sleep(500);
                } catch (InterruptedException e) {
   
                    //线程的中断标志位重新设置为false,无法停下,需要再次掉interrupt()设置true
                    Thread.currentThread().interrupt();//???????
                    e.printStackTrace();
                }
                System.out.println("------hello Interrupt");
            }
        }, "t1");
        t1.start();

        try {
    TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) {
    e.printStackTrace(); }

        new Thread(() -> {
   
            t1.interrupt();//修改t1线程的中断标志位为true
        },"t2").start();
    }

    /**
     *中断为true后,并不是立刻stop程序
     */
    public static void m4()
    {
   
        //中断为true后,并不是立刻stop程序
        Thread t1 = new Thread(() -> {
   
            for (int i = 1; i <= 300; i++) {
   
                System.out.println("------i: " + i);
            }
            System.out.println("t1.interrupt()调用之后02: "+Thread.currentThread().isInterrupted());
        }, "t1");
        t1.start();

        System.out.println("t1.interrupt()调用之前,t1线程的中断标识默认值: "+t1.isInterrupted());
        try {
    TimeUnit.MILLISECONDS.sleep(3); } catch (InterruptedException e) {
    e.printStackTrace(); }
        //实例方法interrupt()仅仅是设置线程的中断状态位设置为true,不会停止线程
        t1.interrupt();
        //活动状态,t1线程还在执行中
        System.out.println("t1.interrupt()调用之后01: "+t1.isInterrupted());

        try {
    TimeUnit.MILLISECONDS.sleep(3000); } catch (InterruptedException e) {
    e.printStackTrace(); }
        //非活动状态,t1线程不在执行中,已经结束执行了。
        System.out.println("t1.interrupt()调用之后03: "+t1.isInterrupted());
    }
```

