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

## 七、线程阻塞工具类：LockSupport

LockSupport可以在线程内任意位置让线程阻塞。它与**suspend()**方法相比弥补了由于**resume()**在前发生，而导致线程无法继续执行的情况。与**Object.wait()**相比，它不需要获得某个对象的锁，也不会抛出**InterruptedException**.

LockSupport的静态方法**park()**可以阻塞当前线程，类似的还有**parkNanos()**,**parkUntil()**等方法。它们实现了一个限时的等待。

```java
public class LockSupportDemo {

	public static Object u = new Object();
	static ChangeObjectThread t1 = new ChangeObjectThread("t1");
	static ChangeObjectThread t2 = new ChangeObjectThread("t2");

	public static class ChangeObjectThread extends Thread {

		public ChangeObjectThread(String name) {
			super.setName(name);
		}

		public void run() {
			synchronized (u) {
				System.out.println("in " + getName());
				LockSupport.park();
				if (Thread.interrupted()) {
					System.out.println(getName() + " 被中断了!");
				}
			}
			System.out.println(getName() + " 执行结束");
		}
	}

	public static void main(String[] args) throws InterruptedException {
		t1.start();
		Thread.sleep(1000);
		t2.start();
		t1.interrupt();
		LockSupport.unpark(t2);
	}

}
```

打印信息：

```
in t1
t1 被中断了!
t1 执行结束
in t2
t2 执行结束
```

上述代码不会用为park()方法而被永久性的挂起。这是因为LockSupport用了类似信号量的机制。为每个线程都准备了一个许可，如果许可可用，则park()立即返回，如果不可用则阻塞。

**即使unpark()发生在park()之前，它也可以使得下一次的park()操作立即返回。**


##八、线程复用：线程池

一种最为简单的线程创建和回收的方法：

```java
new Thread(new Runnable(){
	
	public void run(){
		//do sth
	}
}).start();
```

**实际应用中，必须对线程的数量加以控制。盲目的大量创建线程对系统的性能是有伤害的。**

### 1. JDK对线程池的支持

Executor框架提供对各种类型的线程池：

```java
public static ExecutorService newFixedThreadPool(int nThreads)
public static ExecutorService newSingleThreadExecutor() 
public static ExecutorService newCachedThreadPool()
public static ScheduledExecutorService newSingleThreadScheduledExecutor()
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) 
```

- newFixedThreadPool：返回一个固定线程数量的线程池。如果没有空闲线程，则新的任务会被暂存在一个任务队列中；
- newSingleThreadExecutor：返回只有一个线程的线程池
- newCachedThreadPool：返回一个可调整数量的线程池。
- newSingleThreadScheduledExecutor：返回一个ScheduledExecutorService，线程池大小为1。ScheduledExecutorService接口在ExecutorService接口上扩展了在给定时间执行某任务的功能。
- newScheduledThreadPool：也返回一个ScheduledExecutorService对象，但线程池大小为1

### 2. ScheduledExecutorService

```java
public ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit);
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,long delay,TimeUnit unit);
```
**scheduleAtFixedRate**与**scheduleWithFixedDelay**两者之间有点小小的区别：

- scheduleAtFixedRate：任务的调度频率是一定的。它是以上个任务**开始执行时间**为起点，之后的period时间，调度下一次任务。**是根据周期和任务调度时间中取最大值执行的**
- scheduleWithFixedDelay：是在上一个任务**结束**后，再经过delay时间进行调度。**是按照周期+调度时间为整个周期执行的**

```java
public class ScheduledExecutorServiceDemo {

	public static void main(String[] args) {

		ScheduledExecutorService ses = Executors.newScheduledThreadPool(10);
		// 如果前面的任务没有完成，则调度也不会启动
		ses.scheduleAtFixedRate(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(1000);
					System.out.println("Rate==>>"+System.currentTimeMillis() / 1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}

		}, 0, 2, TimeUnit.SECONDS);
		
		ses.scheduleWithFixedDelay(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(1000);
					System.out.println("Delay==>>"+System.currentTimeMillis() / 1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}

		}, 0, 2, TimeUnit.SECONDS);
	}
}
```

上面代码的打印输出：

```
Rate==>>1453641356
Delay==>>1453641356
Rate==>>1453641358
Delay==>>1453641359
Rate==>>1453641360
Rate==>>1453641362
Delay==>>1453641362
Rate==>>1453641364
```


###3. 核心线程池的内部实现

Executors创建的线程池对象，内部实现均使用了**ThreadPoolExecutor**。以下是三种线程的内部实现方式：

```java
 public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
 }


public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
```

其实都是**ThreadPoolExecutor**类的封装。看下ThreadPoolExecutor类的构造函数：

```java
  public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) 
```

- corePoolSize：线程池中线程数量；
- maximumPoolSize：线程池中最大线程数量；
- keepAliveTime：线程池线程数量超过corePoolSize时，多余的空闲线程的存活时间；
- workQueue：即等待队列
- threadFactory：创建线程的线程工厂，一般用默认即可
- handler：拒绝策略。当线程太多来不及处理，如何拒绝任务。

以下对**workQueue**和**handler**进行详细说明：

workQueue是一个**BlockingQueue**接口，它可以分成以下几类：

- SynchronousQueue：**直接提交的队列**。它没有容量，每一个插入操作都要等待一个删除操作，反之，每一个删除操作都要等待对应的插入操作。SynchronousQueue使用通常需要设置很大的maximumPoolSize，否则很容易执行拒绝策略。
- ArrayBlockingQueue：**有界的任务队列**。它有一个构造函数

```java
public ArrayBlockingQueue(int capacity)
```

当新的任务需要执行，首先判断当前线程数是否大于corePoolSize，如果小于，则优先创建新线程。如果大于，则放入等待队列。如果等待队列满，则判断总线程数是否大于maximumPoolSize，如果大于，则执行拒绝策略。如果不大于，则创建新的进程执行任务。
- LinkedBlockingQueue：**无界的任务队列**。同上，如果当前线程数大于corePoolSize，就不继续增加线程。若后续仍有新的任务加入，而又没有空闲的线程资源，则任务进入队列等待。
- PriorityBlockingQueue：**优先任务队列**。有界、无界都是按照先进先出算法实现，优先任务队列则增加了线程的优先顺序。

###4. 自定义线程创建：ThreadFactory

线程池是为了线程复用，避免线程频繁创建。但是这些最初的线程是从哪里来的呢？答案就是：**ThreadFactory**

ThreadFactory是一个接口，它只有一个方法用来创建线程：

```java
Thread newThread(Runnable r)
```

###5. 优化线程池线程数量

**Ncpu = CPU的数量** <br/>
**Ucpu = 目标CPU的使用率，0<=Ucpu<=1****<br/><br/>
**W/C = 等待时间与计算时间的比率**<br/>

为保持处理器达到期望的使用率，最优的池的大小等于：
**Nthreads = Ncpu * Ucpu *(1 + W/C)**

Java中可以通过**System.getRuntime().availableProcessors();**获得可用的CPU的数量


##九、JDK并发容器

###1.并发集合工具类

- ConcurrentHashMap：线程安全的高效并发的HashMap
- CopyOnWriteArrayList：在读多写少的场合，这个List的性能远远好于Vector
- ConcurrentLinkedQueue：高效的并发队列，使用链表实现。可以看做是线程安全的LinkedList
- BlockingQueue：阻塞队列，非常适合用于作为数据共享的通道
- ConcurrentSkipListMap：跳表的实现。这是一个Map，使用跳表的数据结构进行快速查找

###2.线程安全的HashMap

- 一种可行的办法产生一个线程安全的Map：

```java
public static Map m = Collections.synchronizedMap(new HashMap());

```
这行代码会生成一个SynchronizedMap，这个Map底层使用了信号量mutex加同步锁的机制，保证线程安全。因此性能可能并不是太好。

- ConcurrentHashMap：性能更好。具体在**第四章、锁的优化及注意事项**会详细讲到

###3.List的线程安全

可以使用：

```java
public static List<String> l = Collections.synchronizedList<new LinkedList<String>>();
```

###4.高效的读写队列：深入剖析ConcurrentLinkedQueue

ConcurrentLinkedQueue算是在高并发环境中性能最好的队列

ConcurrentLinkedQueue作为链表，定义Node的核心代码如下：

```java
private static class Node<E> {
        volatile E item;
        volatile Node<E> next;
...

```

- item：目标元素
- next：当前Node的下一个元素

```java
boolean casItem(E cmp, E val) {
    return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
}

void lazySetNext(Node<E> val) {
    UNSAFE.putOrderedObject(this, nextOffset, val);
}

boolean casNext(Node<E> cmp, Node<E> val) {
    return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
}
```

- casItem：用来设置当前Node的item值，cmp是期望值，val是目标值。当当前值=期望值时，就把Node值改成目标值；

ConcurrentLinkedQueue有两个重要字段：head、tail

tail一般来说，我们期望tail总是为链表的末尾，但实际上，tail的更新并不是及时的，而是可能会产生拖延现象。**每次更新会跳跃2个元素**

以下是添加元素的offer()方法：

```java
public boolean offer(E e) {
        checkNotNull(e);
        final Node<E> newNode = new Node<E>(e);

        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
            if (q == null) {
                // p is last node
                if (p.casNext(null, newNode)) {
                    // Successful CAS is the linearization point
                    // for e to become an element of this queue,
                    // and for newNode to become "live".
                    if (p != t) // hop two nodes at a time
                        casTail(t, newNode);  // Failure is OK.
                    return true;
                }
                // Lost CAS race to another thread; re-read next
            }
            else if (p == q)
                // We have fallen off list.  If tail is unchanged, it
                // will also be off-list, in which case we need to
                // jump to head, from which all live nodes are always
                // reachable.  Else the new tail is a better bet.
                p = (t != (t = tail)) ? t : head;
            else
                // Check for tail updates after two hops.
                p = (p != t && t != (t = tail)) ? t : q;
        }
    }

```
该方法没有任何的锁，线程安全完全由CAS操作和队列的算法保证。**具体算法解释请参看书本P126**

###5. 高效读取：不变模式下的CopyOnWriteArrayList

CopyOnWriteArrayList是为了在读取的情况下将性能发挥到极致。对它来说，**读取是完全不加锁的，而且：写入也不会阻塞读取操作，只有写入和写入之间需要进行同步等待**

CopyOnWriteArrayList是在写入操作时，进行一次自我复制。换句话说，当这个List需要修改时，我并不修改原有的内容（这对于保证当前在读线程的数据一致性非常重要），而是对原有的数据进行一次复制，将修改的内容写入副本中。写完之后，再将修改完的副本替换原来的数据。这样就保证写操作不会影响读了。

