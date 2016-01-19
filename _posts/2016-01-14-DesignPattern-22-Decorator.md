---
layout: post
title: 《研磨设计模式》装饰器模式-Decorator
categories: [设计模式]
description: 设计模式|装饰器模式
keywords: 设计模式,装饰器,Decorator
autotoc: true
comments: true
---

本文是《研磨设计模式》第二十二章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。


## 装饰器模式定义

动态的给一个对象增加一些额外的职责。就增加功能来说，装饰模式比生成子类更灵活。

为了能够实现和原来被装饰对象的代码无缝结合，是通过定义一个抽象类。让这个类实现与被装饰对象相同的接口，然后在具体的实现类中，转调被装饰的对象，在转调的前后添加新的功能，这就实现了给被装饰的对象增加功能。

![](/images/posts/designpattern/Decorator-1.png)

![](/images/posts/designpattern/Decorator-2.png)

## 代码示例

组件接口：

```java
public abstract class   Component{
    public abstract void a();
}
```

组件接口实现类：

```java
public class ConcreteComponent{
    public void a();
}
```

装饰器对象：

```java
public  abstract class Decorator extends Component{
     protected Component c；//持有组件对象
    
    // 构造函数传入组件对象
    public Decorator (Componet c){
      this.c= c
    }
    public void a(){
       c.a();//转发给组件对象，可以在转发前后执行一些附加动作
    }
}
```

装饰器具体实现类：向组件对象添加职责

```java
public class ConcreateDecoratorA extends Decorator {
    public  ConcreateDecoratorA (Componet c){
       super(c);
    }

//添加的状态
private String addedState;
set.get*****
public void a(){
      //调用父类的方法，可以在转发前后执行一些附加动作
     //在这里进行处理时，可以用新添加的状态
       super.a();
    }
}
```






