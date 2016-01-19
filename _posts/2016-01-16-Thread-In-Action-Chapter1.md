---
layout: post
title: 《Java高并发程序设计》走入并行世界
categories: [多线程]
description: Thread,多线程
keywords: Java, 多线程，Thread
autotoc: true
comments: true
---

本文是《Java高并发程序设计》第一章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

#Java并发的几个基本概念原则

##原子性

- 原子性是指一个操作是不可中断的，即使是多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。
- 32位系统中，两个线程如果同时对long进行读写的话，线程之间的结果是会有干扰的。因为long有64位。

```java
public class MultiThreadLong {
    public static long t = 0;
    public static class ChangeT implements Runnable {
        private long to;
        public ChangeT(long to) {
            this.to = to;
        }
        public void run() {
            while (true) {
                MultiThreadLong.t = to;
                Thread.yield();
            }
        }
    }
    public static class ReadT implements Runnable {
        public void run() {
            while (true) {
                long tmp = MultiThreadLong.t;
                if (tmp != 111L && tmp != -999L && tmp != 333L && tmp != -444L)
                    System.out.println(tmp);
                Thread.yield();
            }
        }
    }
    public static void main(String[] args) {
        new Thread(new ChangeT(111L)).start();
        new Thread(new ChangeT(-999L)).start();
        new Thread(new ChangeT(333L)).start();
        new Thread(new ChangeT(-444L)).start();
        new Thread(new ReadT()).start();
    }
}
```

正常来说，值应该是这4个值中的一个，但是很可惜，这个程序运行后，会输出大量的其余值。

##可见性

- 可见性指的时候当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道这个修改。
这个问题对串行程序来说，不会发生。但是对并行程序来说，就不一定了。因为如果一个线程修改了一个全局变量，那么其他线程未必可以马上知道这个修改。因为通过硬件，会可能把修改后的值缓存到cache中或者寄存器里。
硬件缓存、指令重排都会导致这个问题。

##有序性

线程A的指令顺序在线程B看来是没有保证的。如果运气好的话，线程B也许真的可以看到和线程A一样的执行顺序。
不过对于一个线程来说，它看到指令执行顺序一定是一致的。指令重排可以保证串行语义的一致性，但没有义务保证多线程间的语义也一致。

Happen-Before规则：**哪些指令不能重排**   

- 程序顺序原则：一个线程内保证语义的串行性；<br/>
- volatile规则：volatile变量的写，先发生于读，这保证了volatile变量的可见性；<br/>
- 锁规则：解锁(unlock)必然发生在虽有的加锁(lock)前；<br/>
- 传递性：A先于B，B先于C，那么A必然先于C；<br/>
- 线程的start()方法先于它的每一个动作；<br/>
- 线程的所有操作先于线程的终结(Thread.join())；<br/>
- 线程的中断(interrupt())先于先于被中断线程的代码：<br/>
- 对象的构造函数执行、结束方法先于finalize()方法；<br/>
