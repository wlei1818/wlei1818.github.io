---
layout: post
title: 线程的基本操作
categories: [Java,多线程]
description: Thread
keywords: Java, 线程
autotoc: true
---
本文记录是《Java高并发程序设计》第二章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

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

**以上代码，虽然对t1做了中断，但是并没有对中断做处理，因此这个中断并没有任何作用。**

```java
public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            public void run() {
                while (true) {
                    if (Thread.currentThread().interrupted()) {
                        System.out.println("Interrupted");
                        break;
                    }
                    Thread.yield();
                }
            }
        };
        t1.start();
        Thread.sleep(2000);
        t1.interrupt();
    }  
```
*注意：Thread.sleep()会由于中断而抛出异常，此时它会**清除中断标记**，如果不加处理，那么在下一次循环开始时，就无法捕捉这个异常。故在异常处理中，再次设置中断标记位。*

```java
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            public void run() {
                while (true) {
                    if (Thread.currentThread().interrupted()) {
                        System.out.println("Interrupted");
                        break;
                    }
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        System.out.println("Interrupted When Sleep");
                        // 设置中断状态
                        Thread.currentThread().interrupt();
                    }
                }
            }
        };
        t1.start();
        Thread.sleep(2000);
        t1.interrupt();
    }  
```

##4.wait和notify
- wait和notify用于多线程协作；是属于Object类的方法；
- 当一个对象调用了wait方法后，当前线程就会在这个对象上等待；
- wait方法并不是随便可以调用的，它必须包含在对应的synchronized语句中，无论是wait或者是notify都需要首先获取目标对象的一个监视器。

```java
public class WaitAndNotify {
    final static Object obj = new Object();
    // T1
    public static class T1 extends Thread {
        public void run() {
            synchronized (obj) {
                System.out.println(System.currentTimeMillis() + "：T1 Started！");
                try {
                    System.out.println(System.currentTimeMillis() + "：T1 wait for object !");
                    obj.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + "：T1 end ！");
            }
        }
    }
    // T2
    public static class T2 extends Thread {
        public void run() {
            synchronized (obj) {
                System.out.println(System.currentTimeMillis() + "：T2 start,notify one thread ");
                obj.notify();
                System.out.println(System.currentTimeMillis() + "：T2 end ");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    public static void main(String[] args) {
        Thread t1 = new T1();
        Thread t2 = new T2();
        t1.start();
        t2.start();
    }
}  
```

*打印信息：*

```java
1452694743902：T1 Started！    
1452694743902：T1 wait for object !      
1452694743902：T2 start,notify one thread         
1452694743902：T2 end        
1452694745902：T1 end ！     
```

- 可以发现，T1线程由于obj的wait方法后阻塞，T2线程nofity后，将T2线程sleep2秒，以便看到效果。 说明T1必须等T2释放obj锁之后，获取obj锁之后才能继续执行！

##5.挂起(suspend)和继续（resume）

- 被挂起的线程，一定要等resume()操作后才能继续制定
- 不推荐使用suspend去挂起线程，因为suspend并不会去释放任何锁资源。此时，其他任何线程想要访问被他暂用的锁时，都会被牵连。可能导致死锁
- 如果resume()操作意外地在suspend()之前就执行了，那么被挂起的线程可能很难有机会被继续执行。并且，更严重的是，它所占用的锁不会被释放。而且对于被挂起的线程，从它的线程状态来看，还是RUNNABLE的。

```java
public class BadSuspend {
    public static Object u = new Object();
    static ChangeObjectThread t1 = new ChangeObjectThread("t1");
    static ChangeObjectThread t2 = new ChangeObjectThread("t1");
    public static class ChangeObjectThread extends Thread {
        public ChangeObjectThread(String name) {
            super.setName(name);
        }
        public void run() {
            synchronized (u) {
                System.out.println("in " + getName());
                Thread.currentThread().suspend();
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        t1.start();
        Thread.sleep(1000);
        t2.start();
        t1.resume();
        t2.resume();
        t1.join();
        t2.join();
    }
}
```

- 线程t2其实是被挂起的，但它的状态确实是RUNNABLE;
- 同时，虽然主函数中调用了resume，但是由于时间先后顺序的缘故，resume并未生效。这就导致了t2永远被挂起，并且永远占用了U对象的锁。
- 不建议使用suspend去挂起线程，可以利用wait和notify进行挂起和继续操作。

```java
public class GoodSuspend {
    public static Object u = new Object();
    public static class ChangeObjectThread extends Thread {
        volatile boolean suspendme = false;
        public void suspendMe() {
            suspendme = true;
        }
        public void resumeMe() {
            suspendme = false;
            synchronized (this) {
                this.notify();
            }
        }
        public void run() {
            while (true) {
                synchronized (this) {
                    while (suspendme) {
                        try {
                            wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
                synchronized (u) {
                    System.out.println("in ChangeObjectThread");
                }
                Thread.yield();
            }
        }
    }
    public static class ReadObjectThread extends Thread {
        public void run() {
            while (true) {
                synchronized (u) {
                    System.out.println("in ReadObjectThread");
                }
                Thread.yield();
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        ChangeObjectThread t1 = new ChangeObjectThread();
        ReadObjectThread t2 = new ReadObjectThread();
        t1.start();
        t2.start();
        Thread.sleep(1000);
        t1.suspendMe();
        System.out.println("suspend t1 2 sec");
        Thread.sleep(2000);
        System.out.println("resume t1");
        t1.resumeMe();
    }
}  
```

##6.等待线程结束（join）和谦让（yield）

- 很多时候，一个线程的输入可能非常依赖于另外一个或者多个线程的输出。此时，这个线程就需要等待依赖线程执行完毕，才能继续执行。

```java
public final void join() throws InterruptedException

public final synchronized void join （long mills）throws InterruptedException
```

- 第一个方法：无线等待，一直阻塞当前线程，直到目标线程执行完毕。
- 第二个方法：给出一个最大等待时间，如果超过指定时间目标线程还在执行，当前线程等不及而继续往下执行；

```java
public class JoinMain {
    public volatile static int i = 0;
    public static class AddThread extends Thread {
        public void run() {
            for (i = 0; i < 1000000; i++)
                ;
        }
    }
    public static void main(String[] args) throws InterruptedException {
        AddThread at = new AddThread();
        at.start();
        at.join();
        System.out.println(i);
    }
}  
```

这里如果不使用join()方法，可能得到的是0或者很小的值，因为at线程还来不及执行，就被主线程打印出来i了。
而用了join()方法之后，就表明主线程愿意等待at线程执行完毕后再打印。

- join()方法的核心源码实现：其实是调用wait方法
- Thread.yield ：它会使当前线程让出CPU。但是让出后，还是会继续进行CPU的资源争夺。

##7.volatile和Java内存模型

- volatiel：告诉虚拟机，这个变量既可能会被某些线程或者程序修改。为了确保这个变量被修改后应用程序范围内的所有线程都能够“看到”这个改动，虚拟机就会采取一定的手段去保证这个变量的可见性。
- volatile不能代替锁，也无法保证一些复合操作的原子性。
- valatile保证数据的可见性和有序性；

```java
public class NoVisibility {
    private static boolean ready;
    private static int number;
    public static class ReaderThread extends Thread {
        public void run() {
            while (!ready)
                ;
            System.out.println(number);
        }
    }
   
    public static void main(String[] args) throws InterruptedException {
        ReaderThread t = new ReaderThread();
        t.start();
        Thread.sleep(1000);
        number = 42;
        ready = true;
        Thread.sleep(10000);
    }
}  
```

在JVM server模式下，ReaderThread线程无法看到主线程中的修改，导致ReaderThread**永远无法退出**；这就是典型的可见性问题。
我们只要使用**volatile**来声明这个变量，那么就可以解决这个问题。

##8.线程组

```java
public class ThreadGroupName implements Runnable {
    @Override
    public void run() {
        String groupAndName = Thread.currentThread().getThreadGroup().getName() + "-" + Thread.currentThread().getName();
        while (true) {
            System.out.println("I am " + groupAndName);
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) {
        ThreadGroup tg = new ThreadGroup("PrintGroup");
        Thread t1 = new Thread(tg, new ThreadGroupName(), "t1");
        Thread t2 = new Thread(tg, new ThreadGroupName(), "t2");
        t1.start();
        t2.start();
        System.out.println(tg.activeCount());
        tg.list();
    }
}  
```

##9.驻守后台：守护线程（Daemon）

```java
public class DaemonDemo {
    public static class DeamonT extends Thread {
        public void run() {
            while (true) {
                System.out.println("I am alive");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        DeamonT t = new DeamonT();
        t.setDaemon(true);
        t.start();
        Thread.sleep(2000);
    }
}  
```

- setDaemon(true)必须在start之前。否则会报错：*java.lang.IllegalThreadStateException*
- 由于t被设置为守护线程，因此系统中只有主线程main为用户线程，因此在main线程休眠2秒钟后退出时，整个程序也随之退出。但是如果不把t设置为守护现场，main线程结束后，t线程还是会不断的打印。

##10.线程优先级

```java
public class PriorityDemo {
    public static class HighPriority extends Thread {
        private int count = 0;
        public void run() {
            while (true) {
                // 加锁竞争资源，使得差异显现的更加明显些
                synchronized (PriorityDemo.class) {
                    count++;
                    if (count > 100000000) {
                        System.out.println("HighPriority finished！");
                        break;
                    }
                }
            }
        }
    }
    public static class LowPriority extends Thread {
        private int count = 0;
        public void run() {
            while (true) {
                // 加锁竞争资源，使得差异显现的更加明显些
                synchronized (PriorityDemo.class) {
                    count++;
                    if (count > 100000000) {
                        System.out.println("LowPriority finished！");
                        break;
                    }
                }
            }
        }
    }
    public static void main(String[] args) {
        HighPriority high = new HighPriority();
        LowPriority low = new LowPriority();
        high.setPriority(Thread.MAX_PRIORITY);
        low.setPriority(Thread.MIN_PRIORITY);
        low.start();
        high.start();
    }
}  
```

- 在大多数情况下，高优先级的线程都比低优先级的线程先执行完！

##11.ArrayList线程不安全

```java
public class ArrayListMultiThread {
    static ArrayList<Integer> al = new ArrayList<Integer>(10);
    public static class AddThread extends Thread {
        public void run() {
            for (int i = 0; i < 1000000; i++) {
                al.add(i);
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        AddThread t1=new AddThread();
        AddThread t2=new AddThread();
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(al.size());
    }
}  
```

**上述程序可能产生3种情况：**
- 得到正确的结果，数组长度为20000；           
- 报错，java.lang.ArrayIndexOutOfBoundsException             这是由于数组在扩容过程中，内部一致性被破坏，由于没有锁的保护，另外一个线程访问到了不一致的内部状态，导致数组越界；    
- 出现了一个其他的数组长度，比如：1670968 这是由于线程对数组同一位置进行覆盖导致的；                   
- 解决方法：使用**Vector**代替ArrayList即可；          

##12.诡异的HashMap

```java
public class HashMapMultiThread {
    static Map<String, String> map = new HashMap<String, String>();
    public static class AddThread extends Thread {
        int start = 0;
        public AddThread(int start) {
            this.start = start;
        }
        public void run() {
            for (int i = start; i < 100000; i += 2) {
                map.put(Integer.toString(i), Integer.toBinaryString(i));
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        AddThread t1 = new AddThread(0);
        AddThread t2 = new AddThread(1);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(map.size());
    }
}  
```

**上述程序可能产生3种情况：**     
- 输出正常结果；      
- 程序正常结束，但是输出一个小于预期值的数；     
- 程序永远无法结束：这个需要参考HashMap源码的put方法，当并发时的迭代遍历，把链表变成了一个环，因此永远是死循环了！     
- 解决方法：使用**ConcurrentHashMap**代替HashMap；     