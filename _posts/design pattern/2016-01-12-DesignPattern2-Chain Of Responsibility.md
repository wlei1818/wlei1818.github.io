---
layout: post
title: 《设计模式之禅》责任链模式
categories: [设计模式]
description: 设计模式之禅|责任链方法
keywords: 设计模式
autotoc: true
comments: true
---

本文是《设计模式之禅》第十六章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、使用场景

女子“三从四德”

```java
public interface IWomen {
    /**
     * 获取个人状况
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:47:27
     * @return
     */
    public int getType();
    /**
     * 获取请示：要出去干嘛
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:47:45
     * @return
     */
    public String getRequest();
}  
```

```java
public class Women implements IWomen {
    private int type = 0;// 1-未出嫁;2-出嫁;3-夫死
    private String request = "";
    public Women(int type, String request) {
        this.type = type;
        this.request = request;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:48:26
     * @return
     */
    @Override
    public String getRequest() {
        return this.request;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:48:26
     * @return
     */
    @Override
    public int getType() {
        return this.type;
    }
}  
```

```java
public interface IHandler {
    /**
     * 处理女性请求
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:52:03
     * @param women
     */
    public void handleMessage(IWomen women);
}  
```

```java
public class Father implements IHandler {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:52:30
     * @param women
     */
    @Override
    public void handleMessage(IWomen women) {
        System.out.println("女儿的请示是：" + women.getRequest());
        System.out.println("父亲的答复是：同意");
    }
}  
```

```java
public class Husband implements IHandler {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:53:30
     * @param women
     */
    @Override
    public void handleMessage(IWomen women) {
        System.out.println("妻子的请示是：" + women.getRequest());
        System.out.println("父丈夫的答复是：同意");
    }
}
```

```java
public class Son implements IHandler {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:54:02
     * @param women
     */
    @Override
    public void handleMessage(IWomen women) {
        System.out.println("母亲的请示是：" + women.getRequest());
        System.out.println("儿子的答复是：同意");
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        // 随机挑选几个女性
        Random rand = new Random();
        ArrayList<IWomen> arrayList = new ArrayList<IWomen>();
        for (int i = 0; i < 5; i++) {
            arrayList.add(new Women(rand.nextInt(4), "我要出去逛街"));
        }
        IHandler father = new Father();
        IHandler husband = new Husband();
        IHandler son = new Son();
        for (IWomen women : arrayList) {
            if (women.getType() == 1) {// 请示父亲
                father.handleMessage(women);
            } else if (women.getType() == 2) {
                husband.handleMessage(women);
            } else if (women.getType() == 3) {
                son.handleMessage(women);
            }
        }
    }
}
```

##二、使用责任链模式重写

关键之处在于在Hanlder类中需要指定下一个处理者。

```java
public abstract class Handler {
    public final static int FATHER_LEVEL_REQUEST = 1;
    public final static int HUSBAND_LEVEL_REQUEST = 2;
    public final static int SON_LEVEL_REQUEST = 3;
    private int level = 0;
    // 每个类都要说明下一个责任类是谁
    private Handler nextHandler;
    public Handler(int level) {
        this.level = level;
    }
    public final void handlerMessage(IWomen women) {
        if (women.getType() == level) {
            this.response(women);
        } else {// 不是当前的人处理
            if (nextHandler != null) {
                nextHandler.handlerMessage(women);
            } else {
                System.out.println("无人处理，按同意处理");
            }
        }
    }
    protected abstract void response(IWomen women);
    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }
} 
```

```java 
public class Father extends Handler {
    public Father() {
        super(FATHER_LEVEL_REQUEST);// 设置自己为父亲
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午10:25:10
     * @param women
     */
    @Override
    protected void response(IWomen women) {
        System.out.println("女儿的请示是：" + women.getRequest());
        System.out.println("父亲的答复是：同意");
    }
}  
```

```java
public class Husband extends Handler {
    public Husband() {
        super(HUSBAND_LEVEL_REQUEST);
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午10:27:58
     * @param women
     */
    @Override
    protected void response(IWomen women) {
        System.out.println("妻子的请示是：" + women.getRequest());
        System.out.println("父丈夫的答复是：同意");
    }
}  
```

```java
public class Son extends Handler {
    public Son() {
        super(SON_LEVEL_REQUEST);
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午10:28:32
     * @param women
     */
    @Override
    protected void response(IWomen women) {
        System.out.println("母亲的请示是：" + women.getRequest());
        System.out.println("儿子的答复是：同意");
    }
}  
```

```java
public interface IWomen {
    /**
     * 获取个人状况
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:47:27
     * @return
     */
    public int getType();
    /**
     * 获取请示：要出去干嘛
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4上午09:47:45
     * @return
     */
    public String getRequest();
}  
```

```java
public class Women implements IWomen {
    private int type = 0;// 1-未出嫁;2-出嫁;3-夫死
    private String request = "";
    @Override
    public String getRequest() {
        return this.request;
    }
    @Override
    public int getType() {
        return this.type;
    }
    public Women(int type, String request) {
        this.type = type;
        switch (type) {
        case 1:
            this.request = "女儿的请求是：" + request;
            break;
        case 2:
            this.request = "妻子的请求是：" + request;
            break;
        case 3:
            this.request = "母亲的请求是：" + request;
            break;
        }
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        // 随机挑选几个女性
        Random rand = new Random();
        ArrayList<IWomen> arrayList = new ArrayList<IWomen>();
        for (int i = 0; i < 5; i++) {
            arrayList.add(new Women(rand.nextInt(4), "我要出去逛街"));
        }
        Handler father = new Father();
        Handler husband = new Husband();
        Handler son = new Son();
        //设置请求顺序
        father.setNextHandler(husband);
        husband.setNextHandler(son);
        for(IWomen women:arrayList){
            father.handlerMessage(women);
        }
        
    }
}  
```

##三、责任链模式的定义

使多个对象都有机会处理请求，从而避免了请求的发送者和接受者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它为止；

责任链标准代码：

```java
/**
 * 抽象处理者
 * 
 * @Package Responsibility.zen.three
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-4 下午04:11:03
 */
public abstract class Handler {
    private Handler nextHandler;
    private final Response handleMessage(Request request) {
        Response resp = null;
        if (this.getHandlerLevel().equals(request.getRequestLevel())) {// 判断是否是自己处理
            resp = this.echo(request);
        } else {
            if (nextHandler != null) {// 是否有下一个处理者
                resp = nextHandler.handleMessage(request);
            } else {
                // 无合适的处理者，默认处理方式
            }
        }
        return resp;
    }
    /**
     * 设置下一个责任人
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午04:14:37
     * @param nextHandler
     */
    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }
    /**
     * 获取处理级别
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午04:15:16
     * @return
     */
    protected abstract Level getHandlerLevel();
    /**
     * 每个处理者都必须实现的处理任务
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午04:15:54
     * @param request
     * @return
     */
    protected abstract Response echo(Request request);
}
```

Handler的实现类：

```java
public class Concrete1Handler extends Handler {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午04:23:41
     * @param request
     * @return
     */
    @Override
    protected Response echo(Request request) {
        // 完成处理逻辑
        return null;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午04:23:41
     * @return
     */
    @Override
    protected Level getHandlerLevel() {
        // 设置自己的处理级别
        return null;
    }
}  
```

```java
public class Level {
}

```java
public class Request {
    /**
     * 获取请求的等级
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午04:18:11
     * @return
     */
    public Level getRequestLevel() {
        return null;
    }
}  
```

```java

public class Response {
}

```

```java

public class Client {
    public static void main(String[] args) {
        // 声明所有节点
        Handler handler1 = new Concrete1Handler();
        Handler handler2 = new Concrete2Handler();
        // 设置责任链
        handler1.setNextHandler(handler2);
        // 提交请求，返回结果
        Response resp = handler1.handleMessage(new Request());
    }
}  
```

**抽象处理者实现三个职责：**

- 定义一个请求的处理方法handlerMessage,唯一对外开放的方法；
- 定义一个链的编排方法 setNext
- 定义了具体的请求者必须实现的两个方法：getHandlerLevel和echo;