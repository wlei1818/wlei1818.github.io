---
layout: post
title: 线程的基本操作
categories: [Java,多线程]
description: Thread
keywords: Java, 线程
autotoc: true
---

## 1.新建线程
线程Thread调用start()方法，就会新建一个线程并让这个线程执行run()方法；

注意：不要用run()来开启新的线程，它只会在当前线程中，串行执行run()中的代码；

```java
public class CreateThread {
    public static void main(String[] args) {
        Thread t1 = new Thread() {
            public void run() {
                System.out.println("Hello!");
            }
        };
        t1.start();
    }
}  
```

* 如果没有特殊情况，都可以通过继承Thread，重载run方法进行自定义线程。但是考虑到继承也是一种宝贵的资源，因此我们可以使用Runnable接口来实现同样的操作；
* Thread有一个非常重要的构造方法：

```java
public Thread(Runnable target)

```

实际上，默认的Thread.run()就是直接调用内部的Runnable接口，因此使用Runnable接口告诉线程该做什么，更为合理。

```java
public class CreateThread2 implements Runnable {
    public static void main(String[] args) {
        Thread t1 = new Thread(new CreateThread2());
        t1.start();
    }
    @Override
    public void run() {
        System.out.println("use runnable");
    }
} 
```

## 2.终止线程
* 一般来说，线程执行完成之后就会自动关闭。但也有些线程常住后台，不会正常终结。终止线程可以用stop()方法，但是现在已经被废弃。因为stop()过于暴力，强制把执行一半的线程终止，可能会引起数据不一致的情况。
* Thread.stop()方法在结束线程时，会直接终止线程，并且会立即释放这个线程所持有的锁。而这些锁恰恰是用来维护对象一致性的。如果此时，写线程写入数据进行到一半时，被强行终止，那这个线程就会被写坏，同时由于锁已经被释放，另外一个等待该锁的线程就顺理成章的读到了这个不一致的对象。 

##3.线程中断
* 严格的讲，线程中断并不会使线程立即退出，而是给线程发送一个通知，告诉目标线程，有人希望你退出！至于目标线程接到通知后如何处理，则完全由目标线程自行决定
与线程中断有关的三个方法：

```java
public void Thread.interrupt(）；                                 //中断线程

public boolean Thread.isInterrupted( );                        //判断是否被中断

public static boolean Thread.interrupted( );                 //判断是否被中断，并清除当前中断状态

```

**Thread.interrupt**：通知目标线程中断，并设置中断标志位；
**Thread.isInterrupted**：通过判断中断标志位判断线程是否被中断；
**Thread.interrupted**：判断当前线程的中断状态，但是会清除中断标志位状态；

```java
public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            public void run() {
                while (true) {
                    Thread.yield();
                }
            }
        };
        t1.start();
        Thread.sleep(2000);
        t1.interrupt();
    }  
```
