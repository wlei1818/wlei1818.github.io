---
layout: post
title: 《Java高并发程序设计》锁的优化及注意事项
categories: [多线程]
description: Thread,多线程
keywords: Java, 多线程，Thread
autotoc: true
comments: true
---

本文是《Java高并发程序设计》第四章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##一、有助于提高“锁”性能的几点建议

###1、减少锁持有时间

```java

public synchronized void syncMethod(){
	otherCode1();
	mutexMethod();
	otherCode2();
}

```

上述同步方法，假设只有metuxMethod()需要同步，那么当otherCode1()和otherCode2()分别是重量级的方法时，则会花费较长的CPU时间。