---
layout: post
title: 《Spring 3.X企业应用开发实战》Spring的事务管理
categories: [Java, Spring]
description: Spring的事务管理
keywords: Spring,事务
autotoc: true
comments: true
---

本文是《Spring 3.X企业应用开发实战》第九章学习笔记

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##一、ThreadLocal
 
ThreadLocal：不是一个线程，而是线程的一个本地化的对象。当工作与多线程中的对象使用ThradLocal维护变量时，ThreadLocal会为每个使用该变量的线程保存一个独立的变量副本。所以每一个线程都可以独立的改变自己的副本，而不会影响其他线程所对应的副本。

```java
protected T initialValue()：返回线程局部变量的初始值；
public T get()
public void set(T value)
public void remove()
```

```java
public class SequenceNumber {
    // 通过匿名内部类覆盖initialValue()方法，指定初始值
    public static ThreadLocal<Integer> seqNum = new ThreadLocal<Integer>() {
        public Integer initialValue() {
            return 0;
        }
    };
    // 获取下一个序列值
    public int getNextNum() {
        seqNum.set(seqNum.get() + 1);
        return seqNum.get();
    }
    public static void main(String[] args) {
        SequenceNumber sn = new SequenceNumber();
        TestClient t1=new TestClient(sn);
        TestClient t2=new TestClient(sn);
        TestClient t3=new TestClient(sn);
        t1.start();
        t2.start();
        t3.start();
    }
    private static class TestClient extends Thread {
        private SequenceNumber sn;
        public TestClient(SequenceNumber sn) {
            this.sn = sn;
        }
        public void run() {
            for (int i = 0; i < 3; i++) {
                System.out.println("Thread[" + Thread.currentThread().getName() + "]  sn=[" + sn.getNextNum() + "]");
            }
        }
    }
}  
```

打印结果：

```
Thread[Thread-0]  sn=[1]
Thread[Thread-0]  sn=[2]
Thread[Thread-0]  sn=[3]
Thread[Thread-1]  sn=[1]
Thread[Thread-1]  sn=[2]
Thread[Thread-1]  sn=[3]
Thread[Thread-2]  sn=[1]
Thread[Thread-2]  sn=[2]
Thread[Thread-2]  sn=[3] 
```

虽然每个线程共享一个SequenceNumber实例，但是之间没有发生相互干扰的情况，而是各自独自产生独立的序列号，这是由于ThreadLocal为每个线程提供了一个单独的副本； 