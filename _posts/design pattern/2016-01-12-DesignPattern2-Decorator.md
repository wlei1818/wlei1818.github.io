---
layout: post
title: 《设计模式之禅》装饰器模式
categories: [设计模式]
description: 设计模式之禅|装饰器方法
keywords: 设计模式
autotoc: true
comments: true
---

本文是《设计模式之禅》第十七章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、案例场景

对成绩单的美化，让家长签字；

```java
/**
 * 抽象成绩单
 * 
 * @Package Decorator.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4 下午05:02:08
 */
public abstract class SchoolReport {
    /**
     * 展示成绩情况
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午05:02:56
     */
    public abstract void report();
    /**
     * 家长签字
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午05:03:08
     */
    public abstract void sign(String name);
}
```

```java
/**
 * 四年级成绩单
 * 
 * @Package Decorator.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4 下午05:03:57
 */
public class FouthGradeSchoolReport extends SchoolReport {
    /**
     * 我的成绩单
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午05:03:57
     */
    public void report() {
        System.out.println("尊敬的家长：");
        System.out.println("......");
        System.out.println("语文62  数学65 体育 90");
        System.out.println("家长签名：");
    }
    /**
     * 家长签名
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午05:07:22
     * @param name
     */
    public void sign(String name) {
        System.out.println("家长签名为:" + name);
    }
}  
```

```java
/**
 * 修饰成绩单
 * 
 * @Package Decorator.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4 下午05:07:58
 */
public class SugarFouthGradeSchoolReport extends FouthGradeSchoolReport {
    // 美化成绩单，先报最高分
    public void reportHighScore() {
        System.out.println("考试语文最高分是75 数学78 ");
    }
    // 美化成绩单，再报排名
    public void reportShort() {
        System.out.println("我的排名是30名");
    }
    // 美化后的报告
    public void report() {
        reportHighScore();
        super.report();
        reportShort();
    }
}  
```

```java
public class Father {
    public static void main(String[] args) {
        SchoolReport sr = new SugarFouthGradeSchoolReport();
        sr.report();
        sr.sign("老孙");
    }
}  
```

以上的设计暴露出一个问题：当汇报的内容需要自由改变时，该如何处理？也就是说report()方法需要改变时，怎么办？如果多个继承，会造成代码的冗余和高耦合。

##二、装饰模式

```java
/**
 * 抽象的装饰类
 * 
 * @Package Decorator.zen.two
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4 下午05:17:47
 */
public abstract class Decorator extends SchoolReport {
    private SchoolReport sr;// 对成绩单进行装饰
    public Decorator(SchoolReport sr) {
        this.sr = sr;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午05:17:47
     */
    @Override
    public void report() {
        sr.report();
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午05:17:47
     * @param name
     */
    @Override
    public void sign(String name) {
        sr.sign(name);
    }
}  
```

```java
public class HighSoreDecorator extends Decorator {
    public HighSoreDecorator(SchoolReport sr) {
        super(sr);
    }
    private void reportHighScore() {
        System.out.println("这次考试语文最高分是78 数学是75 ");
    }
    public void report() {
        reportHighScore();
        report();
    }
}
```

```java
public class SortDecorator extends Decorator {
    public SortDecorator(SchoolReport sr) {
        super(sr);
    }
    private void reportSort() {
        System.out.println("我的排名是38名");
    }
    public void report() {
        reportSort();
        report();
    }
}
```

```java
public class Father {
    public static void main(String[] args) {
        SchoolReport sr = new FouthGradeSchoolReport();
        Decorator d = new HighSoreDecorator(sr);
        d.report();
    }
} 
```

##三、装饰模式的定义

**动态地给一个对象添加一些额外的职责。就增加功能来说，装饰模式相比生成子类更灵活。**

- Componet：抽象类或者接口，定义我们最核心的对象，也就是最原始的对象，如上面的成绩单；
- ConcreteComponent：Component的实现类
- Decorator：装饰角色
- ConcreteDecorator

```java
public abstract class Componet {
    
    public abstract void operate();
}  
```

```java
public class ConcreteComponent extends Componet {
    @Override
    public void operate() {
        System.out.println("do something");
    }
}  
```

```java
public class Decorator extends Componet {
    private Componet comp;
    public Decorator(Componet comp) {
        this.comp = comp;
    }
    @Override
    public void operate() {
        comp.operate();
    }
}  
```

```java
public class ConcreteDecorator extends Decorator {
    public ConcreteDecorator(Componet comp) {
        super(comp);
    }
    public void method1() {
        System.out.println("用这个方法来装饰");
    }
    public void operate() {
        method1();// 装饰方法
        super.operate();
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        Componet comp = new ConcreteComponent();
        // 用这个装饰类装饰
        comp = new ConcreteDecorator(comp);
        // 装饰完运行
        comp.operate();
    }
} 
``` 
