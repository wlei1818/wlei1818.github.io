---
layout: post
title: 《研磨设计模式》迭代器模式-Iterator
categories: [设计模式]
description: 设计模式|迭代器模式
keywords: 设计模式,迭代器,Iterator
autotoc: true
---

本文是《研磨设计模式》第十四章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 案例场景

将新旧两种工资单整合到一起

核心：如何能够以一个统一的方式来访问内部实现不同的聚合对象；

## 迭代器模式

定义：**提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示；**

![](/images/posts/designpattern/Iterator-1.png)

- Iterator：迭代器接口。定义访问和遍历元素的接口；

- ConcreteIterator：具体的迭代器实现对象。实现对聚合对象的遍历，并跟踪遍历时的当前位置；

- Aggregate：聚合对象。定义创建相应迭代器对象的接口；

- ConcreteAggregate：具体聚合对象。实现创建相应的迭代器对象；







