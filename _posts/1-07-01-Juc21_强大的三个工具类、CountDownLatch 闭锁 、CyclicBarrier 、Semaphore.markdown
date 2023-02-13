---
layout:  post
title:   Juc21_强大的三个工具类、CountDownLatch 闭锁 、CyclicBarrier 、Semaphore
date:   1-07-01 23:35:18 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JUC并发编程

---







>为什么这里要介绍下JUC强大的工具类？<br> CountDownLatch | CyclicBarrier | Semaphore 底层都是AQS来实现的


-  ①. CountDownLatch主要有两个方法,当一个或多个线程调用await方法时,这些线程会阻塞  
-  ②. 其它线程调用countDown方法会将计数器减1(调用countDown方法的线程不会阻塞)  
-  ③. 计数器的值变为0时,因await方法阻塞的线程会被唤醒,继续执行 

```java
//需求:要求6个线程都执行完了,mian线程最后执行
public class CountDownLatchDemo {
   
    public static void main(String[] args) throws Exception{
   

        CountDownLatch countDownLatch=new CountDownLatch(6);
        for (int i = 1; i <=6; i++) {
   
            new Thread(()->{
   
                System.out.println(Thread.currentThread().getName()+"\t");
                countDownLatch.countDown();
            },i+"").start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t班长关门走人,main线程是班长");
    }
}
```

- ④. 利用枚举减少if else的判断

```java
public enum CountryEnum {
   

    one(1,"齐"),two(2,"楚"),three(3,"燕"),
    four(4,"赵"),five(5,"魏"),six(6,"韩");

    private Integer retCode;
    private String retMessage;

    private CountryEnum(Integer retCode,String retMessage){
   
        this.retCode=retCode;
        this.retMessage=retMessage;
    }

    public static CountryEnum getCountryEnum(Integer index){
   
        CountryEnum[] countryEnums = CountryEnum.values();
        for (CountryEnum countryEnum : countryEnums) {
   
            if(countryEnum.getRetCode()==index){
   
                return countryEnum;
            }
        }
        return null;
    }

    public Integer getRetCode() {
   
        return retCode;
    }

    public String getRetMessage() {
   
        return retMessage;
    }
}
```

```java
/*
	楚	**国,被灭
	魏	**国,被灭
	赵	**国,被灭
	燕	**国,被灭
	齐	**国,被灭
	韩	**国,被灭
	main	**秦国一统江湖
* */
public class CountDownLatchDemo {
   
    public static void main(String[] args) throws Exception{
   
        CountDownLatch countDownLatch=new CountDownLatch(6);
        for (int i = 1; i <=6; i++) {
   
            new Thread(()->{
   
                System.out.println(Thread.currentThread().getName()+"\t"+"**国,被灭");
                countDownLatch.countDown();
            },CountryEnum.getCountryEnum(i).getRetMessage()).start();

        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t"+"**秦国一统江湖");
    }
}
```

- ⑤. 实验CountDownLatch去解决时间等待问题

```java
public class AtomicIntegerDemo {
   
    AtomicInteger atomicInteger=new AtomicInteger(0);
    public void addPlusPlus(){
   
        atomicInteger.incrementAndGet();
    }
    public static void main(String[] args) throws InterruptedException {
   
        CountDownLatch countDownLatch=new CountDownLatch(10);
        AtomicIntegerDemo atomic=new AtomicIntegerDemo();
        // 10个线程进行循环100次调用addPlusPlus的操作,最终结果是10*100=1000
        for (int i = 1; i <= 10; i++) {
   
            new Thread(()->{
   
               try{
   
                   for (int j = 1; j <= 100; j++) {
   
                       atomic.addPlusPlus();
                   }
               }finally {
   
                   countDownLatch.countDown();
               }
            },String.valueOf(i)).start();
        }
        //(1). 如果不加上下面的停顿3秒的时间,会导致还没有进行i++ 1000次main线程就已经结束了
        //try { TimeUnit.SECONDS.sleep(3);  } catch (InterruptedException e) {e.printStackTrace();}
        //(2). 使用CountDownLatch去解决等待时间的问题
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t"+"获取到的result:"+atomic.atomicInteger.get());
    }
}
```


-  ①. CyclicBarrier的字面意思是可循环(Cyclic) 使用的屏障(barrier)。它要做的事情是,让一组线程到达一个屏障(也可以叫做同步点)时被阻塞,知道最后一个线程到达屏障时,屏障才会开门,所有被屏障拦截的线程才会继续干活,线程进入屏障通过CyclicBarrier的await()方法  
-  ②. 代码验证： 

```java
//集齐7颗龙珠就能召唤神龙
public class CyclicBarrierDemo {
   
    public static void main(String[] args) {
   
        // public CyclicBarrier(int parties, Runnable barrierAction) {}
        CyclicBarrier cyclicBarrier=new CyclicBarrier(7,()->{
   
            System.out.println("召唤龙珠");
        });
        for (int i = 1; i <=7; i++) {
   
            final int temp=i;
            new Thread(()->{
   
                System.out.println(Thread.currentThread().getName()+"\t收集到了第"+temp+"颗龙珠");
                try {
   
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
   
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
   
                    e.printStackTrace();
                }
            }).start();
        }

    }
}
```


-  ①. acquire(获取)当一个线程调用acquire操作时,它要么通过成功获取信号量(信号量减1),要么一直等下去,直到有线程释放信号量,或超时。  
-  ②. release(释放)实际上会将信号量的值加1,然后唤醒等待的线程。  
-  ③. 信号量主要用于两个目的,一个是用于多个共享资源的互斥使用,另一个用于并发线程数的控制。  
-  ④. 代码验证 

```java
public class SemaphoreDemo {
   
    public static void main(String[] args) {
   
        Semaphore semaphore=new Semaphore(3);
        for (int i = 1; i <=6; i++) {
   
            new Thread(()->{
   
                try {
   
                    System.out.println(Thread.currentThread().getName()+"\t抢占了车位");
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"\t离开了车位");
                } catch (InterruptedException e) {
   
                    e.printStackTrace();
                }finally {
   
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

