---
layout:     post
title:      重识synchronized：线程安全和线程同步
subtitle:   线程安全的本质
date:       2019-04-30
author:     Jim
header-img: img/avatar-cmb.jpg
catalog: true
tags:
    - Java
---

先看一段代码，按照预想的结果，应该打印出来的结果是 1_000_000 * 10 ，但是结果并非我们想象的那样。
                                                                                         
    public class Demo    {
    private  int count = 0;
    public static void main(String[] args) {
         new Demo().work();
    }

    public  void work() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 1_000_000; i++) {
                    count();
                }
                System.out.println("Thread 1  x = " + count);
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 1_000_000; i++) {
                    count();
                }
                System.out.println("Thread 2  x = " + count);

            }
        }).start();
    }
    private void count() {
        count++;
      }
    }


![运行结果.png](https://upload-images.jianshu.io/upload_images/4066959-202786405b974b29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#线程安全
什么是线程安全，对于一份共享的数据资源，在多个线程访问这个资源的时候，线程A对数据进行了写数据的操作（正在修改但还没修改完）的同时，线程B对此时这个正在被线程A修改的数据进行了读操作，或者对其进行了写操作（此时线程A正在修改），这时候就会出现数据错误即线程不安全。为了让我们的线程变得安全，我们很多时候会使用Synchronized关键字来实现线程同步。

![使用Synchronized关键字.png](https://upload-images.jianshu.io/upload_images/4066959-c8b4af03bf005edc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![线程同步之后的结果.png](https://upload-images.jianshu.io/upload_images/4066959-c6e6da6bff07f4df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# Synchronized与对象锁（monitor）机制
在一个方法或者代码块被加上了Synchronized之后，当某个线程访问这个方法或者代码块的时候，会获取一个Monitor（可以理解为监视器，此时资源被监视了，保证线程之间对监视资源的数据同步）。此时，别的线程如果也想要访问这个资源，就需要等待获取了Monitor的那个线程释放掉。

任何线程在获取到 Monitor 后的第⼀时间，会先将共享内存中的数据复制到⾃自⼰己的缓存中
任何线程在释放 Monitor 的第⼀ 时间，会先将缓存中的数据复制到共享内存中

关于Synchronized的使用方法

######对于普通方法，Monitor默认指定为当前实例对象
![运行结果](./images/01.png)

######对于方法块同步，Monitor 是 Synchronized 括号里的对象
使用这个方式我们可以自己指定Monitor ，当业务比较比较复杂的时候我们需要加两把锁，通过锁住代码块比较灵活
![Synchronized 同步代码块.png](https://upload-images.jianshu.io/upload_images/4066959-e00c40a29c97cd4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


######对于静态方法同步，锁是当前类的 Class 对象
如下图锁是Demo.class

       public class Demo {
         private static int count = 0;
         private static synchronized void count() {
                count++;
          }
       }

总结一下
######Synchronized 的本质：
保证⽅方法内部或代码块内部资源（数据）的互斥访问。即同一时间、由同一个 Monitor 监视的代码，最多只能有⼀个线程在访问。

######线程安全 的本质：
在多个线程访问共同的资源时，在某⼀个线程对资源进⾏写操作的中途（写⼊已经开始，但还没结束），另一个线程对这个已经被写了⼀半的资源进行了读操作，或者基于这个写了⼀半的资源再次进行了写 操作，导致出现数据错误。

######锁机制的本质：
 通过对共享的资源进行访问限制，让同一时间只有一个线程可以访问资源，确保数据的准确性。








