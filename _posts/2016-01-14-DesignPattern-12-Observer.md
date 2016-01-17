---
layout: post
title: 《研磨设计模式》观察者模式-Observer
categories: [设计模式]
description: 设计模式|观察者模式
keywords: 设计模式,观察,Observer
autotoc: true
---

本文是《研磨设计模式》第十二章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。


## 一、观察者模式的定义

定义对象间的一种一对多的依赖关系。当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

### 观察者模式结构

![](/images/posts/designpattern/Observer-1.png)


#### 观察者标准代码模式

```java
/**
 * 目标对象，它知道观察它的观察者，并提供注册和删除观察者的接口.即观察者观察的对象（报社）
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-29下午09:24:40
 */
public class Subject {
    private List<Observer> observers = new ArrayList<Observer>();
    /**
     * 注册观察者对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午09:26:47
     * @param observer
     */
    public void attach(Observer observer) {
        observers.add(observer);
    }
    /**
     * 删除观察者对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午09:27:23
     * @param observer
     */
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    /**
     * 通知所有注册的观察者对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午09:28:29
     */
    protected void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(this);
        }
    }
}  

````

```java

/**
 * 具体的目标对象，负责把有关状态存入到相应的观察者对象 ,并在自己状态发生改变时，通知各个观察者
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-29下午09:29:14
 */
public class ConcreteSubject extends Subject {
    private String subjectStatus;
    public String getSubjectStatus() {
        return subjectStatus;
    }
    public void setSubjectStatus(String subjectStatus) {
        this.subjectStatus = subjectStatus;
        // 状态发生了改变，通知各个观察者
        this.notifyObservers();
    }
} 

```

```java

/**
 * 观察者接口，定义一个更新的接口给那些在目标发生改变的时候被通知的对象 .即：订阅者
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-29下午09:31:22
 */
public interface Observer {
    /**
     * 更新的接口
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午09:32:16
     * @param subject
     */
    public void update(Subject subject);
}  

```

```java

public class ConcreteObserver implements Observer {
    private String observerState;
    @Override
    public void update(Subject subject) {
        // 具体的更新实现
        // 这里可能需要更新观察者的状态，使其与目标的状态一致
        observerState = ((ConcreteSubject) subject).getSubjectStatus();
    }
}  

```

3、业务场景实例：用户订阅报纸

```java

public class Subject {
    private List<Observer> observers = new ArrayList<Observer>();
    /**
     * 注册观察者对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午09:26:47
     * @param observer
     */
    public void attach(Observer observer) {
        observers.add(observer);
    }
    /**
     * 删除观察者对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午09:27:23
     * @param observer
     */
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    /**
     * 通知所有注册的观察者对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午09:28:29
     */
    protected void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(this);
        }
    }
}  

```
 
```java

/**
 * 报纸对象，具体的目标实践
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-29下午10:36:40
 */
public class NewsPaper extends Subject {
    private String content;
    public String getContent() {
        return content;
    }
    public void setContent(String content) {
        this.content = content;
        // 内容有了，说明出新报纸了，通知所有的读者（观察者）
        notifyObservers();
    }
}

```

```java

/**
 * 观察者接口，定义一个更新的接口给那些在目标发生改变的时候被通知的对象 .即：订阅者
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-29下午09:31:22
 */
public interface Observer {
    /**
     * 更新的接口
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午09:32:16
     * @param subject
     */
    public void update(Subject subject);
}

```

```java

/**
 * 真正的读者对象
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-29下午10:38:32
 */
public class Reader implements Observer {
    private String name;// 读者姓名
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-7-29下午10:38:32
     * @param subject
     */
    @Override
    public void update(Subject subject) {
        System.out.println(name + "收到报纸了，阅读它。内容是====" + ((NewsPaper) subject).getContent());
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}

```

客户端调用：

```java

public class Client {
    public static void main(String[] args) {
        // 报纸，被观察者
        NewsPaper subject = new NewsPaper();
        // 读者，观察者
        Reader reader1 = new Reader();
        reader1.setName("老孙");
        Reader reader2 = new Reader();
        reader2.setName("老陈");
        // 注册读者
        subject.attach(reader1);
        subject.attach(reader2);
        subject.setContent("最新一期环球时报！");
    }
}  

```

4、单向依赖：只有观察者依赖于目标，而目标是不会依赖于观察者的。

- 双方的联系的主动权是掌握在目标手中，只有目标知道什么时候需要通知观察者。在整个过程中，观察者始终是被动的。被动的等待目标的通知，等待目标传值给他。

- 触发时机：一般情况下，是在完成了状态维护后触发，不能先通知后改数据。

- 多个观察者之间的功能是平行的，相互不应该有先后的依赖关系。 

5、观察者模式的推模型和拉模型 

- 推模型：目标对象主动向观察者推送目标的详细信息。其实就是将具体的信息字段传入；

- 拉模型：即将整个目标对象传送给观察者。观察者根据自己的需要从对象中获取信息；

##二、Java中的观察者模式

使用Java的观察者模式，目标对象必须**继承java.util.Observable类。** 

观察者对象必须实现Observer接口。 

```java

/**
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-29下午11:07:18
 */
public class NewsPaper extends Observable {
    private String content;
    public String getContent() {
        return content;
    }
    public void setContent(String content) {
        this.content = content;
        // 内容有更新，通知所有读者
        // 注意在用Java的Observer模式的时候，下面这句话必不可少
        this.setChanged();
        // 推的方式
        this.notifyObservers(this.content);
        // 如果用拉的方式，这么调用
        // this.notifyObservers();
    }
}  

```

```java

public class Reader implements Observer {
    private String name;
    public void update(Observable o, Object arg) {
        // 采用推的方式
        System.out.println(name + "收到报纸了，阅读先。目标推过来的内容是==" + arg);
        // 采用拉的方式
        //System.out.println(name + "收到报纸了，阅读先。目标推过来的内容是==" + ((NewsPaper) o).getContent());
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}  

```

客户端调用：

```java

public class Client {
    public static void main(String[] args) {
        // 报纸，被观察者
        NewsPaper subject = new NewsPaper();
        // 读者，观察者
        Reader reader1 = new Reader();
        reader1.setName("老孙");
        Reader reader2 = new Reader();
        reader2.setName("老陈");
        // 注册读者
        subject.attach(reader1);
        subject.attach(reader2);
        subject.setContent("最新一期环球时报！");
    }
}  

```

##三、观察者模式的思考

1. 观察者模式的本质：触发联动。<br/>

- 当修改目标对象的状态，就会触发相应的通知，然后会循环调用所有注册的观察者对象的相应方法，其实就相当于联动调用这些观察者的方法。

2. 何时选用？<br/>

- 当一个抽象模型的两个方面，其中一个方面的操作依赖于另一个方面的状态变化时。

- 如果在更改一个对象的时候，需要同时连带改变其他的对象，而且不知道究竟应该有多少对象需要被连带改变时；

- 当一个对象必须通知其他的对象，但是又希望这个对象和其他被它通知的对象是松耦合的时

##四、设计模式的变型

案例：对水质的观测，水中杂质为正常时，只通知监测人员，当轻度污染时，通知监测人员和预警人员，当重度污染时，通知监测人员、预警人员、以及部门领导；

```java

public interface WatcherObserver {
    /**
     * 被通知的方法
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-30下午08:58:20
     * @param subject
     */
    public void update(WaterQualitySubject subject);
    /**
     * 设置观察人员的职务
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-30下午08:58:42
     * @param job
     */
    public void setJob(String job);
    public String getJob();
}  

```

```java

public class Watcher implements WatcherObserver {
    private String job;
    @Override
    public String getJob() {
        return job;
    }
    @Override
    public void setJob(String job) {
        this.job = job;
    }
    @Override
    public void update(WaterQualitySubject subject) {
        System.out.println(job + "获取到通知，当前污染级别为：" + subject.getPolluteLevel());
    }
}

```

```java

public abstract class WaterQualitySubject {
    protected List<WatcherObserver> observers = new ArrayList<WatcherObserver>();
    public void attach(WatcherObserver observer) {
        observers.add(observer);
    }
    public void detach(WatcherObserver observer) {
        observers.remove(observer);
    }
    // 根据不同的业务通知不同的观察者
    public abstract void notifyWatchers();
    // 获取水质污染等级
    public abstract int getPolluteLevel();
}  

```

```java

public class WaterQuality extends WaterQualitySubject {
    // 0-正常,1-轻度污染,2-重度污染
    private int polluteLevel = 0;
    @Override
    public int getPolluteLevel() {
        return polluteLevel;
    }
    @Override
    public void notifyWatchers() {
        for (WatcherObserver watcher : observers) {
            if (polluteLevel >= 0) {
                // 仅仅通知监测人员
                if ("监测人员".equals(watcher.getJob()))
                    watcher.update(this);
            }
            
            if(polluteLevel>=1){
                //同时也要通知预警人员
                if("预警人员".equals(watcher.getJob()))
                    watcher.update(this);
            }
            
            if(polluteLevel>=2){
                //同时要通知领导
                if("监测部门领导".equals(watcher.getJob()))
                    watcher.update(this);
            }
        }
    }
    public void setPolluteLevel(int polluteLevel) {
        this.polluteLevel = polluteLevel;
        // 水质数据得到，通知人员
        notifyWatchers();
    }
}

```

```java

public class Client {
    public static void main(String args[]) {
        WatcherObserver observer1 = new Watcher();
        observer1.setJob("监测人员");
        WatcherObserver observer2 = new Watcher();
        observer2.setJob("预警人员");
        WatcherObserver observer3 = new Watcher();
        observer3.setJob("监测部门领导");
        WaterQuality subject = new WaterQuality();
        subject.attach(observer1);
        subject.attach(observer2);
        subject.attach(observer3);
        
        subject.setPolluteLevel(0);
        subject.setPolluteLevel(1);
        subject.setPolluteLevel(2);
        
    }
}  

```




