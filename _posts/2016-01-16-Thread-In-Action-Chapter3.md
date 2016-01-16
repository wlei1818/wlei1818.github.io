---
layout: post
title: Java并法包
categories: [多线程]
description: Thread
keywords: Java, 多线程，Thread
autotoc: true
---
本文是《Java高并发程序设计》第三章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##1.synchronized的功能扩展：重入锁

```java
public class ReenterLock implements Runnable {
    public static ReentrantLock lock = new ReentrantLock();
    public static int i = 0;
    @Override
    public void run() {
        for (int j = 0; j < 100000; j++) {
            lock.lock();
            try {
                i++;
            } finally {
                lock.unlock();
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        ReenterLock r1 = new ReenterLock();
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r1);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}  
```

- 重入锁对于同一个线程来说，是可以反复进入的；加锁代码甚至可以写成如下：

```java
lock.lock();
lock.lock();
try {
    i++;
 } finally {
    lock.unlock();
    lock.unlock();
 }  
```

这种情况下，一个线程连续两次获取同一把锁，这是允许的！释放的时候，只要同时释放多个锁即可！
