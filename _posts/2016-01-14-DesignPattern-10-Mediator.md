---
layout: post
title: 《研磨设计模式》中介者模式-Mediator
categories: [设计模式]
description: 设计模式|中介者模式
keywords: 设计模式,中介者,Mediator
autotoc: true
---

本文是《研磨设计模式》第十章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##一、使用场景

电脑主板和内存、CPU等之间的数据交互；

##二、中介者模式定义

用一个中介对象来封装一系列的对象交互。中介者使得各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变他们之间的交互。

![](/images/posts/designpattern/Mediator-1.png)

##三、中介者模式标准代码

Colleague:

```java
/**
 * 同事类的抽象父类
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-20下午08:48:43
 */
public abstract class Colleague {
    
    //持有中介者对象，每个同事类都知道它的中介者对象
    private Mediator mediator;
    
    public Colleague(Mediator mediator){
        this.mediator=mediator;
    }
    public Mediator getMediator() {
        return mediator;
    }  
```

ConcreteColleague:

```java
/**
 * 同事类的具体实现类
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-20下午08:51:51
 */
public class ConcreteColleagueA extends Colleague {
    public ConcreteColleagueA(Mediator mediator) {
        super(mediator);
    }
    public void someOperation() {
        // 在需要跟其他同事通信的时候，通知中介者对象
        getMediator().changed(this);
    }
}  
```

Mediator：中介者接口

```java
/**
 * 中介者，定义各个同事对象通信的接口
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-20下午09:38:31
 */
public interface Mediator {
    /**
     * 同事对象在自身改变的时候去通知中介者的方法 ,让中介者去负责相应的与其他同事对象的交互
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-20下午09:39:43
     * @param colleague
     */
    public void changed(Colleague colleague);
}  

```

ConcreteMediator：中介者具体实现类

```java

package Mediator.one;
/**
 * 具体的中介者实现
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-20下午09:41:16
 */
public class ConcreteMediator implements Mediator {
    
    
    public ConcreteColleagueA colleagueA;
    public ConcreteColleagueB colleagueB;
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-7-20下午09:41:16
     * @param colleague
     */
    @Override
    public void changed(Colleague colleague) {
        // TODO Auto-generated method stub
    }
    public ConcreteColleagueA getColleagueA() {
        return colleagueA;
    }
    public void setColleagueA(ConcreteColleagueA colleagueA) {
        this.colleagueA = colleagueA;
    }
    public ConcreteColleagueB getColleagueB() {
        return colleagueB;
    }
    public void setColleagueB(ConcreteColleagueB colleagueB) {
        this.colleagueB = colleagueB;
    }
}

```


##四、广义中介者模式

实际应用中对中介者模式的简化：

- 通常会去掉同事对象的父类；
- 通常不定义Mediator接口；
- 同事对象不再持有中介者，而是在需要的时候直接获取中介者对象并调用。
- 中介者对象也不再持有同事对象，而是在具体处理方法里面去创建，或者获取，或者从参数传入需要的同事对象；

案例：<br/>

部门与员工：多对多的关系

最简单的设计（也是耦合性最高的）：

```java

public class Dep {
    private Collection<Dep> colUser=new ArrayList<Dep>();
} 

```

```java

 public class User {
    
    private Collection<User> colUser=new ArrayList<User>();
}

```

耦合性很高，当部门撤销或者合并时，需要将下面所有的员工进行操作。同时人员离职或入职，需要通知所有的部门更新；

使用中介者模式进行改善：

```java

public class Dep {
    private String depId;
    private String depName;
    public String getDepId() {
        return depId;
    }
    public void setDepId(String depId) {
        this.depId = depId;
    }
    public String getDepName() {
        return depName;
    }
    public void setDepName(String depName) {
        this.depName = depName;
    }
    /**
     * 撤销部门，通知中介者进行人员的更新
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-22下午08:33:48
     * @return
     */
    public boolean deleteDep() {
        // 注意在实际的开发中，这些业务功能可能会被做到业务层去
        DepUserMediatorImpl mediator = DepUserMediatorImpl.getInstance();
        mediator.deleteDep(depId);
        return true;
    }
}

```

```java

public class User {
    private String userId;
    private String userName;
    public String getUserId() {
        return userId;
    }
    public void setUserId(String userId) {
        this.userId = userId;
    }
    public String getUserName() {
        return userName;
    }
    public void setUserName(String userName) {
        this.userName = userName;
    }
    /**
     * 人员离职
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-22下午09:01:11
     * @return
     */
    public boolean dismission() {
        DepUserMediatorImpl mediator = DepUserMediatorImpl.getInstance();
        mediator.deleteUser(userId);
        return true;
    }  
```

中介者：已经实现了单例模式

```java

public class DepUserMediatorImpl {
    private static DepUserMediatorImpl mediator = new DepUserMediatorImpl();
    private DepUserMediatorImpl() {
        initTestData();// 初始化数据
    }
    // 单例模式
    public static DepUserMediatorImpl getInstance() {
        return mediator;
    }
    // 测试用，存放部门和员工的对应关系
    private Collection<DepUserModel> depUserCol = new ArrayList<DepUserModel>();
    private void initTestData() {
        DepUserModel du1 = new DepUserModel();
        du1.setDepId("d1");
        du1.setUserId("u1");
        du1.setDepUserId("du1");
        depUserCol.add(du1);
        DepUserModel du2 = new DepUserModel();
        du2.setDepId("d2");
        du2.setUserId("u2");
        du2.setDepUserId("du2");
        depUserCol.add(du2);
        DepUserModel du3 = new DepUserModel();
        du3.setDepId("d3");
        du3.setUserId("u3");
        du3.setDepUserId("du3");
        depUserCol.add(du3);
    }
    /**
     * 删除部门后，完成与之相关的业务操作
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-22下午08:42:49
     * @param depId
     * @return
     */
    public boolean deleteDep(String depId) {
        Collection<DepUserModel> tempCol = new ArrayList<DepUserModel>();
        for (DepUserModel du : depUserCol) {
            if (du.getDepId().equals(depId)) {
                tempCol.add(du);
            }
        }
        depUserCol.removeAll(tempCol);
        return true;
    }
    /**
     * 员工离职后，完成与之相关的操作
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-22下午08:46:35
     * @param userId
     * @return
     */
    public boolean deleteUser(String userId) {
        Collection<DepUserModel> tempCol = new ArrayList<DepUserModel>();
        for (DepUserModel du : depUserCol) {
            if (du.getUserId().equals(userId)) {
                tempCol.add(du);
            }
        }
        depUserCol.removeAll(tempCol);
        return true;
    }
    /**
     * 测试用：显示部门所有员工
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-22下午08:49:03
     * @param dep
     */
    public void showDepUsers(Dep dep) {
        for (DepUserModel du : depUserCol) {
            if (du.getDepId().equals(dep.getDepId())) {
                System.out.println("部门编号==" + dep.getDepId() + "拥有的员工==" + du.getUserId());
            }
        }
    }
    /**
     * 测试用：显示员工所在的部门
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-22下午08:50:43
     * @param user
     */
    public void showUserDeps(User user) {
        for (DepUserModel du : depUserCol) {
            if (du.getUserId().equals(user.getUserId())) {
                System.out.println("员工编号==" + user.getUserId() + "所在的部门==" + du.getDepId());
            }
        }
    }
}  

```

客户端调用：

```java

public class Client {
    public static void main(String[] args) {
        DepUserMediatorImpl mediator = DepUserMediatorImpl.getInstance();
        Dep dep1 = new Dep();
        dep1.setDepId("d1");
        Dep dep2 = new Dep();
        dep2.setDepId("d2");
        User u1 = new User();
        u1.setUserId("u1");
        System.out.println("部门撤销前==========");
        mediator.showUserDeps(u1);
        dep1.deleteDep();// 删除部门
        System.out.println("部门撤销后==========");
        mediator.showUserDeps(u1);
    }
}  

```

中介者模式的本质：**封装交互**





