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

**ArrayBlockingQueue**就用到了**Condition**。


## 三、允许多个线程同时访问：Semaphore

无论是内部锁synchronized还是重入锁RenntrantLock，一次都只允许一个线程访问一个资源。Semaphore可以指定多个线程，同时访问某一个资源。

Semaphore提供2个构造函数：

```java
 public Semaphore(int permits)

public Semaphore(int permits, boolean fair)  //第二个参数指定是否公平
```

信号量的主要方法有：

```java
public void acquire() throws InterruptedException
public void acquireUninterruptibly() 
public boolean tryAcquire() 
public boolean tryAcquire(long timeout, TimeUnit unit)
public void release() 
```

- acquire()：尝试获得一个准入的许可。若无法获得，则线程会等待，直到有线程释放一个许可或者当前线程被中断；
- acquireUninterruptibly() ：与acquire()类似，但是不响应中断；
- tryAcquire()：尝试获得一个许可，直接返回布尔值，不会进行等待，立即返回；
- release()：释放一个许可

信号量代码示例：

```java
public class SemaDemo implements Runnable {

    final Semaphore semp = new Semaphore(5);

    public void run() {

        try {
            semp.acquire();
            // 以下是临界区
            // 模拟耗时操作
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getId() + " done");
            semp.release();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newFixedThreadPool(20);
        final SemaDemo demo = new SemaDemo();
        for (int i = 0; i < 20; i++) {
            exec.submit(demo);
        }

    }

}

```

以上代码表明同时有5个线程可以进入临界区。打印结果是以5个线程为单位输出。
使用了acquire()后必须使用release()释放。如果不幸发生了信号量的泄露，即获取了但没释放，那么进入临界区的线程就会越来越少，直到所有线程都不可访问。

## 四、ReadWriteLock读写锁

如果使用重入锁或者内部锁，则理论上所有读之间、读与写之间。写和写之间都是串行操作。读写锁使得读写可以分离。

**读写锁的访问约束情况**

- 读-读不互斥：读读之间不阻塞
- 读-写互斥：读阻塞写，写也会阻塞读
- 写-写互斥：写写阻塞

如果在系统中，读操作次数远远大于写操作，那么读写锁可以发挥最大功效，提升系统的性能。

```java
public class ReadWriteLockDemo {

    private static Lock lock = new ReentrantLock();
    private static ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    private static Lock readLock = readWriteLock.readLock();
    private static Lock writeLock = readWriteLock.writeLock();
    private int value;

    public Object handleRead(Lock lock) {

        try {
            lock.lock();// 模拟读操作
            Thread.sleep(1000);// 读操作越耗时,读写锁的优势就越明显
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return value;
    }

    public void handleWrite(Lock lock, int index) {
        try {
            lock.lock();
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        value = index;
    }

    public static void main(String[] args) {
        final ReadWriteLockDemo demo = new ReadWriteLockDemo();
        Runnable readRunnable = new Runnable() {

            @Override
            public void run() {
                demo.handleRead(readLock);
                // demo.handleRead(lock);
            }
        };

        Runnable writeRunnable = new Runnable() {
            @Override
            public void run() {
                demo.handleWrite(writeLock, new Random().nextInt());
                //demo.handleWrite(lock, new Random().nextInt());
            }
        };

        // 读线程
        for (int i = 0; i < 18; i++) {
            new Thread(readRunnable).start();
        }

        // 写线程
        for (int i = 18; i < 20; i++) {
            new Thread(writeRunnable).start();
        }
    }
}
```
上述代码中，如果使用可重入锁，那么所有的读写线程必须相互等待；而使用读写锁的话，所有的读线程都
能并行，而写会阻塞读。

## 五、倒计时器：CountDownLatch

这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。
典型的场景就是火箭发射，使用CountDownLatch使得点火线程在所有检查工作做完后再执行。

```java
public CountDownLatch(int count)  //计时器的计数个数
```

代码演示：

```java
public class CountDownLatchDemo implements Runnable {

    public static final CountDownLatch end = new CountDownLatch(10);
    public static final CountDownLatchDemo demo = new CountDownLatchDemo();

    @Override
    public void run() {
        // 模拟检查任务
        try {
            Thread.sleep(new Random().nextInt(10) * 1000);
            System.out.println("finish check");
            end.countDown();// 该项任务检查完成，计数器减一
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            exec.submit(demo);
        }
        // 等待检查任务
        end.await();
        // 发射火箭
        System.out.println("Fire");
        exec.shutdown();
    }

}
```
使用countDown()方法说明该任务以及完成，计数器可以减一。await() 方法要求主线程等待所有10个任务检全部查完成。
主线程在CountDownLatch上等待，当所有的检查任务都完成后，主线程方能继续执行。

## 六、循环栅栏：CyclicBarrier

CyclicBarrier与CountDownLatch类似，但是比CountDownLatch强大。栅栏的意思就是用来阻止线程继续执行，要求线程在栅栏处等待。CyclicBarrier表明这个计数器可以反复使用。到数后计数器就会归零。

CyclicBarrier可接收一个参数作为barrierAction。所谓barrierAction就是当计数器**一次计数**完成后，系统就会执行的动作。

```java
public CyclicBarrier(int parties, Runnable barrierAction) 
```

代码演示：

```java
public class CyclicBarrierDemo {

    public static class Soldier implements Runnable {
        private String solider;
        private CyclicBarrier cylic;

        public Soldier(String solider, CyclicBarrier cylic) {
            this.solider = solider;
            this.cylic = cylic;
        }

        @Override
        public void run() {
            // 等待所有士兵到齐
            try {
                cylic.await();
                doWork();
                // 等待所有士兵完成工作
                cylic.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }

        void doWork() {
            try {
                Thread.sleep(Math.abs(new Random().nextInt() % 10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(solider + ":任务完成");
        }
    }

    public static class BarrierRun implements Runnable {

        boolean flag;
        int N;

        public BarrierRun(boolean flag, int n) {
            super();
            this.flag = flag;
            N = n;
        }

        @Override
        public void run() {
            if (flag) {
                System.out.println("司令:[士兵" + N + "个，任务完成！]");
            } else {
                System.out.println("司令:[士兵" + N + "个，集合完成！]");
                flag = true;
            }
        }
    }

    public static void main(String[] args) {
        final int N = 10;
        Thread[] soilders = new Thread[N];
        boolean flag = false;
        CyclicBarrier cyclic = new CyclicBarrier(N, new BarrierRun(flag, N));
        // 设置屏障点，主要为了执行这个方法
        System.out.println("集合队伍！");
        for (int i = 0; i < N; i++) {
            System.out.println("士兵" + i + "报道！");
            soilders[i] = new Thread(new Soldier("士兵" + i, cyclic));
        }
    }

}
```


