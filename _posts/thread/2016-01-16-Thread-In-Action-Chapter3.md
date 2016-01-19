---
layout: post
title: 《Java高并发程序设计》Java并法包
categories: [多线程]
description: Thread,多线程
keywords: Java, 多线程，Thread
autotoc: true
comments: true
---
本文是《Java高并发程序设计》第三章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##一、synchronized的功能扩展：重入锁

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

**中断响应**

对于synchronized来说，如果一个线程在等待锁，那么结果只有两种，要么它获取这么锁继续执行，要么它就保持等待。

而使用重入锁，则提供另外一种可能，那就是线程可以被中断。也就是在你等待锁的过程中，程序可以根据需要取消对锁的请求。

```java
public class IntLock implements Runnable {

    public static ReentrantLock lock1 = new ReentrantLock();
    public static ReentrantLock lock2 = new ReentrantLock();
    int lock;

    /**
     * 控制加锁顺序,方便构造死锁
     * 
     * @param lock
     */
    public IntLock(int lock) {
        this.lock = lock;
    }

    public void run() {
        try {
            if (lock == 1) {
                lock1.lockInterruptibly();
                Thread.sleep(1000);
                lock2.lockInterruptibly();
            } else {
                lock2.lockInterruptibly();
                Thread.sleep(1000);
                lock1.lockInterruptibly();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            if (lock1.isHeldByCurrentThread()) {
                lock1.unlock();
            }
            if (lock2.isHeldByCurrentThread()) {
                lock2.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        IntLock r1 = new IntLock(1);
        IntLock r2 = new IntLock(2);
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        t2.start();
        Thread.sleep(1000);
        // 中断其中一个线程
        t2.interrupt();
    }

}
```

上述代码我们首先构造了一个死锁程序。t1占用lock1，再占用lock2；t2先占用lock2，再请求lock1。在这里，对锁的请求，统一使用 **lockInterruptibly()**。这是一个可以对中断进行响应的锁申请动作，即在等待锁的过程中，可以响应中断。

main主线程中，处于休眠时，此时两个线程已经处于死锁状态。当t2被中断，t2会放弃lock1的申请，同时会释放已经占用的lock2锁。这样可以使得t1线程可以顺利得到lock2而继续执行下去。

**锁申请等待限时**

另一种避免死锁的方法是，限时等待。可以使用**tryLock**进行一次限时的等待。

```java
public class TimeLock implements Runnable {

    private ReentrantLock lock = new ReentrantLock();

    public void run() {
        try {
            if (lock.tryLock(5, TimeUnit.SECONDS)) {
                Thread.sleep(6000);
            } else {
                System.out.println("Get lock failed！");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        TimeLock tl = new TimeLock();
        Thread t1 = new Thread(tl);
        Thread t2 = new Thread(tl);
        t1.start();
        t2.start();
    }
}
```

tryLock()还有不带参的方法，这个方法在尝试获得锁时，如果锁未被占用，则申请成功，返回true；如果锁被占用，则立即返回false，不会等待。因此不会产生死锁。

tryLock()不产生死锁：

```java
public class TryLock implements Runnable {

    public static ReentrantLock lock1 = new ReentrantLock();
    public static ReentrantLock lock2 = new ReentrantLock();
    int lock;

    public TryLock(int lock) {
        this.lock = lock;
    }

    public void run() {

        if (lock == 1) {

            while (true) {
                try {
                    if (lock1.tryLock()) {
                        Thread.sleep(500);

                        if (lock2.tryLock()) {
                            try {
                                System.out.println(Thread.currentThread().getId() + "My Job is Done！");
                                return;
                            } finally {
                                lock2.unlock();
                            }
                        }
                    }
                } catch (InterruptedException e) {

                } finally {
                    lock1.unlock();
                }
            }
        } else {

            while (true) {
                if (lock2.tryLock()) {
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    if (lock1.tryLock()) {
                        try {
                            System.out.println(Thread.currentThread().getId() + "My Job is Done！");
                            return;
                        } finally {
                            lock1.unlock();
                        }
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        TryLock l1 = new TryLock(1);
        TryLock l2 = new TryLock(2);
        Thread t1 = new Thread(l1);
        Thread t2 = new Thread(l2);
        t1.start();
        t2.start();
    }
}
```

上述代码，是非常容易出现死锁的代码。但是使用tryLock()后情况大大改善，由于tryLock不会傻傻等待，所以只要执行足够长的时间，总能使得2个线程得到顺利的执行。

**公平锁**

默认情况下，锁的申请都是不公平的。是随机从等待队列中获取的。

公平锁则会按照时间的先后顺序，依次获取锁。它的一大特点就是：**不会产生饥饿现象**。synchronized就保证了公平锁。而重入锁也可以对公平性进行设置。

```java
public ReentrantLock(boolean fair);
```

参数为true，就是公平。公平锁因为需要系统维护有序队列，因此实现难度较高，成本较
大。

```java
public class FairLock implements Runnable {

    public static ReentrantLock fairLock = new ReentrantLock(true);// 公平锁

    /**
     * @Description
     * @author sunhanbin
     * @date 2016-1-19下午10:20:51
     */
    @Override
    public void run() {
        while (true) {
            try {
                fairLock.lock();
                System.out.println(Thread.currentThread().getName() + " get lock");
            } finally {
                fairLock.unlock();
            }
        }

    }

    public static void main(String[] args) {
        FairLock f1 = new FairLock();
        Thread t1 = new Thread(f1, "Thread_1");
        Thread t2 = new Thread(f1, "Thread_2");
        t1.start();
        t2.start();

    }

}
```
**ReentrantLock重要方法：**

- lock()：获得锁，如果锁已经被占用，则等待。
- lockInterruptibly()：获得锁，但先优先响应中断
- tryLock()
- tryLock(Long time,TimeUnit unit)
- unlock()

##二、重入锁的好搭档：Condition条件

wait()和notify()是跟synchronized搭配使用。而Condition是与重入锁搭配使用。利用Condition对象，可以让线程在合适的时间等待，或者在某一个特定的时刻得到通知，继续执行。

**Condition接口的主要方法：**

```java
void await() throws InterruptedException;
void awaitUninterruptibly();
long awaitNanos(long nanosTimeout) throws InterruptedException;
boolean await(long time, TimeUnit unit) throws InterruptedException;
boolean awaitUntil(Date deadline) throws InterruptedException;
void signal();
void signalAll();
```

- **await()**：使当前线程等待，同时释放当前锁。当其他线程中使用 **signal()** 或者 **signalAll()** 时，线程会重新获得锁并继续执行。与Object.wait()方法很相似。
- **awaitUninterruptibly()**：与await()基本相同，但是不会在等待过程中响应中断；
- **signal()**：与notify()方法类似，用于唤醒一个在等待中的线程。

```java
public class ReentrantConditionLock implements Runnable {

    public static ReentrantLock lock = new ReentrantLock();
    public static Condition condition = lock.newCondition();

    @Override
    public void run() {
        try {
            lock.lock();
            condition.await();
            System.out.println("Thread is going on");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void main(String[] args) throws InterruptedException {
        ReentrantConditionLock tc = new ReentrantConditionLock();
        Thread t1 = new Thread(tc);
        t1.start();
        Thread.sleep(2000);
        // 通知线程1继续执行
        lock.lock();
        condition.signal();
        lock.unlock();
    }

}
```

signal()方法之后，一般要释放相关的锁，谦让给被唤醒的线程，让它可以继续执行。

ArrayBlockingQueue就用到了Condition。


