---
layout: post
title: 《研磨设计模式》命令模式-Command
categories: [设计模式]
description: 设计模式|观察者模式
keywords: 设计模式,命令,Command
autotoc: true
comments: true
---

本文是《研磨设计模式》第十三章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。


##一、命令模式的定义

1. 场景：开机按下按钮电脑启动，按钮实际上是通过主板才去启动各个硬件的

2. 定义

**将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作；**

![](/images/posts/designpattern/Command-1.png)

- Command:定义命令的接口，声明执行的方法；

- ConcreteCommand：命令接口实现对象，是“虚”的实现；通常会持有接收者，并调用接收者的功能来完成命令要执行的操作；

- Receiver：接收者，真正执行命令的对象。任何类都可能成为一个接收者，只要它能够实现命令要求实现的相应功能；

- Invoker：要求命令对象执行请求，通常会持有命令对象，可以持有很多的命令对象。这个是客户端真正触发命令并要求命令执行相应操作的地方；

- Client：创建具体的命令对象，并且设置命令对象的接收者。即装配者。

## 二、代码实战

### 命令模式标准模式

```java

/**
 * 命令接口，声明执行的操作
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-1下午08:06:49
 */
public interface Command {
    /**
     * 执行命令对应的操作
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-1下午08:07:14
     */
    public void execute();
}

```

```java

/**
 * 具体的命令实现对象
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-1下午08:07:50
 */
public class ConcreteCommand implements Command {
    private Receiver receiver = null;// 持有相应的接收者对象
    private String state;// 示意，命令对象可以有自己的状态
    public ConcreteCommand(Receiver receiver) {
        this.receiver = receiver;
    }
    public void execute() {
        // 通常会转调接收者对象的相应方法，让接收者来真正执行功能
        receiver.action();
    }
}  

```

```java

public class Receiver {
    public void action() {
        // 真正执行命令操作的功能代码
    }
}

```

```java

/**
 * 调用者
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-1下午08:11:50
 */
public class Invoker {
    private Command command = null;
    public void runCommand() {
        command.execute();
    }
    public Command getCommand() {
        return command;
    }
    public void setCommand(Command command) {
        this.command = command;
    }
}  

```

```java

public class Client {
    public void assemble() {
        Receiver receiver = new Receiver();
        Command command = new ConcreteCommand(receiver);
        Invoker invoker = new Invoker();
        invoker.setCommand(command);
    }
}  

```

### 使用命令模式重写案例：

```java

/**
 * 主板接口：是接收者，真正去做事的
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-1下午08:22:58
 */
public interface MainBoardApi {
    // 开机
    public void open();
}  

```

```java
/**
 * 技嘉主板，开机命令的真正实现者，在Command模式中充当Receiver
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-1下午08:23:55
 */
public class GigaMainBoard implements MainBoardApi {
    @Override
    public void open() {
        System.out.println("技嘉主板现在正在开机，请稍后...");
        System.out.println("接通电源....");
        System.out.println("设备检查...");
        System.out.println("装载系统...");
        System.out.println("机器正常运转起来...");
        System.out.println("机器已经打开，请操作");
    }
}  
```

```java
/**
 * 命令接口，声明执行的操作
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-1下午08:06:49
 */
public interface Command {
    /**
     * 执行命令对应的操作
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-1下午08:07:14
     */
    public void execute();
}
```

```java
/**
 * 开机命令的实现，实现Command接口 持有开机命令的真正实现，通过调用接收者的方法来实现命令
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-1下午08:33:37
 */
public class OpenCommand implements Command {
    private MainBoardApi mainBoard = null;
    public OpenCommand(MainBoardApi mainBoard) {
        this.mainBoard = mainBoard;
    }
    @Override
    public void execute() {
        // 对于命令对象，根不不知道如何开机，会转调主板对象
        // 让主板去完成开机的功能
        mainBoard.open();
    }
}  
```

```java
/**
 * 机箱对象，就是命令的调用者。
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-1下午08:38:02
 */
public class Box {
    public Box(Command openCommand) {
        this.openCommand = openCommand;
    }
    private Command openCommand = null;
    public void setOpenCommand(Command openCommand) {
        this.openCommand = openCommand;
    }
    public void openButtonPressed() {
        openCommand.execute();
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        // 1、把命令和真正的实现组合起来，相当于在组装机器
        // 把机箱上的按钮的连接线插到主板上
        MainBoardApi mainBoard = new GigaMainBoard();
        OpenCommand command =   new OpenCommand(mainBoard);
        // 2、为机箱上的按钮设置对应的命令，让按钮知道该干什么
        Box box = new Box(command);
        // 3、模拟按下机箱上的按钮
        box.openButtonPressed();
    }
}
```

### 命令模式深入

- 命令模式的关键是在于把请求封装成为对象，也就是命令对象。

- 命令模式的参数化配置：可以用不同的命令对象，去参数化配置客户的请求；

比如前面的例子，按下机箱按钮可能是开机也可能是重启：这时可以在MainBoardApi中新增重启的方法，同时新增一个重启的命令对象，实现命令接口；

- 可撤销的操作

实现可撤销的操作，有两种方式，一种是：**补偿式**，又叫**反操作式**；另一种是：**存储恢复式**；

- 宏命令：一组命令。是一个命令的组合。

基本上把它当命令对象处理，但是它跟普通的命令对象又有所不同：宏命令包含多个普通的命令对象。

```java

/**
 * 接收者：厨师接口
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4下午8:40:20
 */
public interface CookApi {
    // 做菜方法
    public void cook(String name);
}  

```

```java

/**
 * 做热菜的厨师对象-接收者实现类
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4下午8:41:36
 */
public class HotCook implements CookApi {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午8:41:37
     * @param name
     */
    @Override
    public void cook(String name) {
        System.out.println("本厨师正在做热菜：" + name);
    }
}  

```

```java
/**
 * 做凉菜厨师
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4下午8:44:01
 */
public class CoolCook implements CookApi {
    @Override
    public void cook(String name) {
        System.out.println("凉菜" + name + "已经做好，正在装盘");
    }
}  
```

```java
/**
 * 命令对象：绿豆排骨煲
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4下午8:45:55
 */
public class ChopCommand implements Command {
    // 持有接收者，接收者真正转调执行方法
    private CookApi cookApi = null;
    @Override
    public void execute() {
        cookApi.cook("绿豆排骨煲");
    }
    public void setCookApi(CookApi cookApi) {
        this.cookApi = cookApi;
    }
}  
```

```java
/**
 * 菜单对象：宏命令
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4下午8:49:38
 */
public class MenuCommand implements Command {
    // 多个命令对象
    private Collection<Command> col = new ArrayList<Command>();
    public void addCommand(Command cmd) {
        col.add(cmd);
    }
    @Override
    public void execute() {
        // 循环执行菜单里的每个菜.
        for (Command cmd : col) {
            cmd.execute();
        }
    }
}
```

```java
/**
 * 服务员：负责组合菜单，负责组装每个菜和具体的实现者
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4下午8:52:21
 */
public class Waiter {
    private MenuCommand menuCommand = new MenuCommand();
    /**
     * 客户点菜
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午9:02:29
     * @param cmd
     */
    public void orderDish(Command cmd) {
        // 客户传过来的命令对象是没有和接收者组装的
        CookApi hotCook = new HotCook();
        CookApi coolCook = new CoolCook();
        // 这里简单点 判断是热菜还是凉菜
        if (cmd instanceof ChopCommand) {// 绿豆炖排骨
            ((ChopCommand) cmd).setCookApi(hotCook);
        } else {
            // 其他多种例子
        }
        menuCommand.addCommand(cmd);
    }
    /**
     * 点完单后，表示要执行命令了。这里就是执行的命令组合
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午9:19:50
     */
    public void orderOver() {
        menuCommand.execute();
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        // 创建服务员
        Waiter waiter = new Waiter();
        // 创建命令对象
        Command chop = new ChopCommand();
        // Command duck=new DuckCommand();
        // Command pork=new PorkCommand();
        // 点菜
        waiter.orderDish(chop);
        // waiter.orderDish(duck);
        // waiter.orderDish(pork);
        // 点菜完毕
        waiter.orderOver();
    }
}  
```

###命令模式的本质

**封装请求**






