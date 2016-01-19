---
layout: post
title: 《设计模式之禅》模板方法模式
categories: [设计模式]
description: 设计模式之禅|模板方法
keywords: 设计模式
autotoc: true
comments: true
---

本文是《设计模式之禅》第十章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、使用场景

悍马模型的设计

```java
public abstract class HummerModel {
    public abstract void start();
    public abstract void stop();
    public abstract void alarm();
    public abstract void engineBoom();
    public void run() {
        this.start();
        this.engineBoom();
        this.alarm();
        this.stop();
    }
}  
```

```java
public class HummerH1Model extends HummerModel {
    public void alarm() {
        System.out.println("悍马1鸣笛....");
    }
    public void engineBoom() {
        System.out.println("悍马1发动引擎....");
    }
    public void start() {
        System.out.println("悍马1启动....");
    }
    public void stop() {
        System.out.println("悍马1熄火....");
    }
} 
```

```java 
public class Client {
    public static void main(String[] args) {
        HummerModel h1 = new HummerH1Model();
        h1.run();
    }
}  
```

##二、模板模式的定义

1、模板方法模式知识用了Java的继承机制。

2、为了防止恶意操作，一般模板方法都加上**final**关键字，不允许被覆写；

3、抽象模板中的基本方法尽量设计为**protected**类型；
