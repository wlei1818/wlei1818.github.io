---
layout: post
title: 《研磨设计模式》抽象工厂模式-AbstractFactory
categories: [设计模式]
description: 设计模式|抽象工厂模式
keywords: 设计模式
autotoc: true
---

本文是《研磨设计模式》第七章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 抽象工厂模式

1. 定义：提供一个创建一系列相关或相互依赖对象的接口，而无需指定他们的类
 
2. 解决思路：一个是只知道所需要的一系列对象的接口，而不知道具体实现，或者是不知道具体使用哪一个实现；另外一个是这一系列对象是相关或者相互依赖的，也就是说既要创建接口的对象，还要约束他们之间的关系。
 
3. 案例：组装电脑
 
4. 与工厂方法模式、简单工厂方法的区别：<br/>
    **前者只关注单个产品对象的创建，比如创建CPU的工厂方法，就只负责创建CPU，创建主板的工厂方法，就只负责创建主板。**
 
5. 抽象工厂的模式和结构说明

![](/images/posts/designpattern/AbstractFactory-1.png)

![](/images/posts/designpattern/AbstractFactory-2.png)

![](/images/posts/designpattern/AbstractFactory-3.png)

- 从某种意义上看，抽象工厂其实是一个产品系列，或者是产品簇；
 
- AbstractFactory在Java中通常被实现成接口；
 
- AbstractFactory定义了创建产品所需要的接口，具体的实现是在实现类里面，通常在实现类里面需要多种更具体的实现。所以AbstractFactory定义的创建产品的方法可以看成是工厂方法，而这些工厂方法的具体实现就延迟到了具体的工厂里面，也就是说使用工厂方法实现抽象工厂。

![](/images/posts/designpattern/AbstractFactory-4.png)

![](/images/posts/designpattern/AbstractFactory-5.png)

![](/images/posts/designpattern/AbstractFactory-6.png)

## 抽象工厂模式的本质

**选择产品簇的实现**<br/>
 
1、工厂方法是选择单个产品的实现。虽然一个类里面可以有多个工厂方法，但是这些方法之间一般是没有联系的，即使看起来像有联系抽象工厂着重的就是为一个产品簇选择实现，定义在抽象工厂里面的方法通常是有联系的，它们都是产品的某一部分或者是相互依赖的。如果抽象工厂里面只定义一个方法，直接创建产品，那就退化成工厂方法了。

![](/images/posts/designpattern/AbstractFactory-7.png)

![](/images/posts/designpattern/AbstractFactory-8.png)