---
layout: post
title: 《设计模式之禅》访问者模式
categories: [设计模式]
description: 设计模式之禅|访问者方法
keywords: 设计模式
autotoc: true
comments: true
---

本文是《设计模式之禅》第二十五章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。
 
 一、案例场景

统计公司所有员工信息。注意：不同的员工职位需要打印不同的信息，如普通员工关系职位、经理关心绩效等；

```java
/**
 * 抽象员工基类
 * 
 * @Package Visitor.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-19 下午01:30:14
 */
public abstract class Employee {
    public final static int MALE = 0;
    public final static int FEMALE = 1;
    private String name;
    private int salary;
    private int sex;
    // 打印员工信息
    public void report() {
        String info = "姓名：" + name + "\n";
        info += "性别：" + sex + "\n";
        info += "薪水：" + salary + "\n";
        info += getOtherInfo();
        System.out.println(info);
    }
    // 其他信息
    protected abstract String getOtherInfo();
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getSalary() {
        return salary;
    }
    public void setSalary(int salary) {
        this.salary = salary;
    }
    public int getSex() {
        return sex;
    }
    public void setSex(int sex) {
        this.sex = sex;
    }
}  
```

```java
/**
 * 普通员工
 * 
 * @Package Visitor.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-19 下午01:33:32
 */
public class CommonEmployee extends Employee {
    private String job;
    public String getJob() {
        return job;
    }
    public void setJob(String job) {
        this.job = job;
    }
    protected String getOtherInfo() {
        return "工作：" + job + "\n";
    }
}
```

```java
/**
 * 管理层人员
 * 
 * @Package Visitor.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-19 下午01:37:18
 */
public class Manager extends Employee {
    private String performance;
    protected String getOtherInfo() {
        return "绩效：" + performance + "\n";
    }
    public String getPerformance() {
        return performance;
    }
    public void setPerformance(String performance) {
        this.performance = performance;
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        for (Employee emp : mockEmployee()) {
            emp.report();
        }
    }
    public static List<Employee> mockEmployee() {
        List<Employee> emplist = new ArrayList<Employee>();
        // 普通员工
        CommonEmployee zhangsan = new CommonEmployee();
        zhangsan.setJob("Java程序员");
        zhangsan.setName("张三");
        zhangsan.setSalary(5000);
        zhangsan.setSex(Employee.MALE);
        CommonEmployee lisi = new CommonEmployee();
        lisi.setJob("美工");
        lisi.setName("李四");
        lisi.setSalary(5000);
        lisi.setSex(Employee.MALE);
        // 管理人员
        Manager manager = new Manager();
        manager.setName("王五");
        manager.setPerformance("绩效很差");
        manager.setSalary(100000);
        manager.setSex(Employee.MALE);
        emplist.add(zhangsan);
        emplist.add(lisi);
        emplist.add(manager);
        return emplist;
    }
}  
```

使用访问者模式重写案例：

```java
/**
 * 抽象员工基类
 * 
 * @Package Visitor.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-19 下午01:30:14
 */
public abstract class Employee {
    public final static int MALE = 0;
    public final static int FEMALE = 1;
    private String name;
    private int salary;
    private int sex;
    // 允许一个访问者访问
    public abstract void accept(IVisitor visitor);
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getSalary() {
        return salary;
    }
    public void setSalary(int salary) {
        this.salary = salary;
    }
    public int getSex() {
        return sex;
    }
    public void setSex(int sex) {
        this.sex = sex;
    }
}
```

```java
/**
 * 普通员工
 * 
 * @Package Visitor.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-19 下午01:33:32
 */
public class CommonEmployee extends Employee {
    private String job;
    public String getJob() {
        return job;
    }
    public void setJob(String job) {
        this.job = job;
    }
    // 允许访问者访问
    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
}  
```

```java
public class Manager extends Employee {
    private String performance;
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-19下午02:10:03
     * @param visitor
     */
    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
    public String getPerformance() {
        return performance;
    }
    public void setPerformance(String performance) {
        this.performance = performance;
    }
}
```

```java
/**
 * 访问者接口：定义可以访问哪些对象
 * 
 * @Package Visitor.zen.two
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-19 下午02:01:55
 */
public interface IVisitor {
    // 可以访问普通员工
    public void visit(CommonEmployee commonEmployee);
    // 可以访问经理
    public void visit(Manager manager);
}  
```

```java
public class Visitor implements IVisitor {
    // 访问经理
    public void visit(Manager manager) {
        System.out.println(getManagerInfo(manager));
    }
    // 访问普通员工
    public void visit(CommonEmployee commonEmployee) {
        System.out.println(getCommonEmployee(commonEmployee));
    }
    private String getBasicInfo(Employee employee) {
        String info = "姓名：" + employee.getName() + "\n";
        info += "性别：" + (employee.getSex() == Employee.MALE ? "男" : "女") + "\n";
        info += "薪水：" + employee.getSalary() + "\n";
        return info;
    }
    private String getManagerInfo(Manager manager) {
        String info = "绩效：" + manager.getPerformance() + "\n";
        return getBasicInfo(manager) + info;
    }
    private String getCommonEmployee(CommonEmployee commonEmployee) {
        String info = "工作：" + commonEmployee.getJob() + "\n";
        return getBasicInfo(commonEmployee) + info;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        for (Employee emp : mockEmployee()) {
            emp.accept(new Visitor());
        }
    }
    public static List<Employee> mockEmployee() {
        List<Employee> emplist = new ArrayList<Employee>();
        // 普通员工
        CommonEmployee zhangsan = new CommonEmployee();
        zhangsan.setJob("Java程序员");
        zhangsan.setName("张三");
        zhangsan.setSalary(5000);
        zhangsan.setSex(Employee.MALE);
        CommonEmployee lisi = new CommonEmployee();
        lisi.setJob("美工");
        lisi.setName("李四");
        lisi.setSalary(5000);
        lisi.setSex(Employee.MALE);
        // 管理人员
        Manager manager = new Manager();
        manager.setName("王五");
        manager.setPerformance("绩效很差");
        manager.setSalary(100000);
        manager.setSex(Employee.MALE);
        emplist.add(zhangsan);
        emplist.add(lisi);
        emplist.add(manager);
        return emplist;
    }
}  
```

##二、访问者模式的定义

**封装一些作用于某种数据结构中的各元素的操作，它可以在不改变数据结构的前提下定义作用于这些元素的新的操作；**

- Visitor：抽象访问者。声明访问者可以访问哪些元素。也就是visit()方法中参数定义哪些对象可以被访问；
- ConcreteVisitor：影响访问者访问到一个类后该怎么干；
- Element：声明接受哪一类访问者访问，用accept（）方法的参数来定义；
- ConcreteElement：实现accept（）方法，通常是visitor.visit(this)就结束了；
- ObjectStruture：元素产生者；

```java
/**
 * 抽象元素
 * 
 * @Package Visitor.zen.three
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-19 下午03:22:08
 */
public abstract class Element {
    // 定义自身业务逻辑
    public abstract void doSomething();
    // 允许谁访问
    public abstract void accept(IVisitor visitor);
}  
```

```java
public class ConcreteElement1 extends Element {
    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
    public void doSomething() {
    }
}
```

```java
public class ConcreteElement2 extends Element {
    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
    public void doSomething() {
    }
}  
```

```java
public interface IVisitor {
    // 定义可以访问哪些对象
    public void visit(ConcreteElement1 ce1);
    public void visit(ConcreteElement2 ce2);
}  
```

```java
public class Visitor implements IVisitor {
    // 访问元素1
    public void visit(ConcreteElement1 ce1) {
        ce1.doSomething();
    }
    // 访问元素2
    public void visit(ConcreteElement2 ce2) {
        ce2.doSomething();
    }
}  
```

```java
public class ObjectStruture {
    /**
     * 对象生成器，生产Element对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-19下午03:29:06
     * @return
     */
    public static Element createElement() {
        Random rand = new Random();
        if (rand.nextInt(100) > 50) {
            return new ConcreteElement1();
        } else {
            return new ConcreteElement2();
        }
    }
}  
```

```java
public class Client {   
    public static void main(String[] args) {
        for(int i=0;i<10;i++){
            //获得元素对象
            Element el=ObjectStruture.createElement();
            el.accept(new Visitor());
        }
    }
}  
```