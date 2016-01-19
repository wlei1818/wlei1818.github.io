---
layout: post
title: 《研磨设计模式》职责链模式-Chain Of Responsibility
categories: [设计模式]
description: 设计模式|职责链模式
keywords: 设计模式,职责链,Chain Of Responsibility
autotoc: true
comments: true
---

本文是《研磨设计模式》第二十三章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、使用场景

OA审批，不同的权限审批不同金额；

### 不使用职责链模式的代码

```java
public class FeeRequest {
    /**
     * 提交经费申请给项目经理
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-4下午9:42:49
     * @param user
     * @param fee
     * @return
     */
    public String requsetToProjectManager(String user, double fee) {
        String str = "";
        if (fee < 500) {// 项目经理只能审批500元以内的
            str = projectHandler(user, fee);
        } else if (fee < 1000) {// 部门经理的权限在1000元以内
            str = depMangagerHandler(user, fee);
        } else if (fee >= 1000) {// 总经理的权限最大
            str = generalManagerHandler(user, fee);
        }
        return str;
    }
    public String projectHandler(String user, double fee) {
        String str = "";
        if ("小李".equals(user)) {
            str = "项目经理同意" + user + "聚餐费用" + fee + "的请求";
        } else {
            str = "项目经理不同意" + user + "聚餐费用" + fee + "的请求";
        }
        return str;
    }
    public String depMangagerHandler(String user, double fee) {
        String str = "";
        if ("小李".equals(user)) {
            str = "部门经理同意" + user + "聚餐费用" + fee + "的请求";
        } else {
            str = "部门经理不同意" + user + "聚餐费用" + fee + "的请求";
        }
        return str;
    }
    public String generalManagerHandler(String user, double fee) {
        String str = "";
        if ("小李".equals(user)) {
            str = "总经理同意" + user + "聚餐费用" + fee + "的请求";
        } else {
            str = "总经理不同意" + user + "聚餐费用" + fee + "的请求";
        }
        return str;
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        FeeRequest request = new FeeRequest();
        String ret1 = request.requsetToProjectManager("小李", 300);
        System.out.println("ret1====>>>>>" + ret1);
        String ret2 = request.requsetToProjectManager("小张", 300);
        System.out.println("ret2====>>>>>" + ret2);
        String ret3 = request.requsetToProjectManager("小李", 600);
        System.out.println("ret3====>>>>>" + ret3);
        String ret4 = request.requsetToProjectManager("小李", 6000);
        System.out.println("ret4====>>>>>" + ret4);
    }
}  
```

## 二、职责链模式的定义

使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求,知道有一个对象处理它为止。

- 职责链的基本思路就是：**动态构造流程步骤**；

![](/images/posts/designpattern/Responsibility-1.png)

- Handler：定义职责的接口，通常在这里定义处理请求的方法，可以在这里实现后继链；

- ConcreteHandler：职责实现类。实现在他范围内请求的处理，如果不处理，就继续转发请求给后继者；

- Client

```java
public abstract class Handler {
    protected Handler successor;// 下一个继承者
    /**
     * 设置后继的职责对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午8:59:47
     * @param successor
     */
    public void setSuccessor(Handler successor) {
        this.successor = successor;
    }
    
    /**
     * 示意请求的方法，虽然这个示意方法是没有传入参数的
     * 但是实际是可以传入参数的，根据具体需求来选择是否传入参数
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:00:19
     */
    public abstract void handlerRequest();
}  
```

```java
/**
 * 具体的职责对象，用来处理具体的请求
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-5下午9:02:14
 */
public class ConcreteHandler extends Handler {
    /**
     * 根据某些判断条件去判断是否为当前等级来请求，如果不是则转发到下一节点。 这些判断条件可以看具体情况分析
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:02:14
     */
    @Override
    public void handlerRequest() {
        boolean somecondition = false;
        if (somecondition) {
            // 属于自己的处理范围，就在这里处理
            System.out.println("ConcreteHandler1 handle this request");
        } else {
            // 如果不是自己的处理范文，则判断是否还有后继对象，如果有，就转发请求给后继对象
            // 如果没有，就什么都不做，自然结束
            if (this.successor != null) {
                this.successor.handlerRequest();
            }
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Handler handler1 = new ConcreteHandler();
        // Handler handler2=new ConcreteHandler2();
        // 设置下一个处理节点（因为没有写ConcreteHandler2所以就注释了）
        // handler1.setSuccessor(handler2);
        handler1.handlerRequest();
    }
}  
```

### 使用职责链模式重写案例

```java
public abstract class Handler {
    protected Handler successor = null;
    /**
     * 设置下一节点
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:13:37
     * @param successor
     */
    public void setSuccessor(Handler successor) {
        this.successor = successor;
    }
    /**
     * 处理餐费的申请
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:14:18
     * @param user
     * @param fee
     * @return
     */
    public abstract String handlerFeeRequest(String user, double fee);
} 
```

```java 
public class ProjectManager extends Handler {
    /**
     * 部门经理处理请求
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:15:09
     * @param user
     * @param fee
     * @return
     */
    @Override
    public String handlerFeeRequest(String user, double fee) {
        String str = "";
        if (fee < 500) {
            if ("小李".equals(user)) {
                str = "项目经理同意" + user + "聚餐费用" + fee + "的请求";
            } else {
                str = "项目经理不同意" + user + "聚餐费用" + fee + "的请求";
            }
        } else {
            if (successor != null) {
                return successor.handlerFeeRequest(user, fee);
            }
        }
        return str;
    }
}  
```

```java
public class DepManager extends Handler {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:19:11
     * @param user
     * @param fee
     * @return
     */
    @Override
    public String handlerFeeRequest(String user, double fee) {
        String str = "";
        if (fee < 1000) {
            if ("小李".equals(user)) {
                str = "部门经理同意" + user + "聚餐费用" + fee + "的请求";
            } else {
                str = "部门经理不同意" + user + "聚餐费用" + fee + "的请求";
            }
        } else {
            if (this.successor != null) {
                successor.handlerFeeRequest(user, fee);
            }
        }
        return str;
    }
}  
```

```java
public class GeneralManager extends Handler {
    @Override
    public String handlerFeeRequest(String user, double fee) {
        String str = "";
        if (fee >= 1000) {
            if ("小李".equals(user)) {
                str = "总经理同意" + user + "聚餐费用" + fee + "的请求";
            } else {
                str = "总经理不同意" + user + "聚餐费用" + fee + "的请求";
            }
        } else {
            if (this.successor != null) {
                successor.handlerFeeRequest(user, fee);
            }
        }
        return str;
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        // 组装职责链
        Handler handler1 = new ProjectManager();
        Handler handler2 = new DepManager();
        Handler handler3 = new GeneralManager();
        handler1.setSuccessor(handler2);
        handler2.setSuccessor(handler3);
        String ret = handler1.handlerFeeRequest("小李", 600);
        System.out.println(ret);
    }
}  
```

当遇到多个请求时，怎么处理？

- 第一种方式是：可以在Handler类中新增方法，处理其他的请求。当然这种方法有弊端：每次新增请求都要新增方法，很不灵活；

- 第二种方法是：用一个通用的请求对象来封装请求传递的参数；然后定义一个通用的调用方法，这个方法不区分具体业务。所有的业务都是这个方法，在通用的请求对象中有业务的标记字段。

```java
public abstract class Handler {
    protected Handler successor;
    public void setSuccessor(Handler successor) {
        this.successor = successor;
    }
    /**
     * 通用的请求处理方法
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:37:18
     * @param rm
     * @return
     */
    public Object handleRequest(RequestModel rm) {
        if (successor != null) {
            return this.successor.handleRequest(rm);
        } else {
            System.out.println("没有后续的处理节点了");
            return false;
        }
    }
} 
```

```java
public class ProjectManager extends Handler {
    /**
     * 处理请求
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:42:04
     * @param rm
     * @return
     */
    public Object handleRequest(RequestModel rm) {
        if (FeeRequestModel.FEE_TYPE.equals(rm.getType())) {
            // 项目经理可以处理聚餐费用的申请
            return handleFeeRequest(rm);
        } else {
            // 其他的，项目经理暂时不想处理
            //~当然，其他的比如请假申请，肯定有同样的类继承自RequestModel
            return super.handleRequest(rm);
        }
    }
    /**
     * 处理自身可以处理的业务
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-5下午9:44:04
     * @param rm
     * @return
     */
    private Object handleFeeRequest(RequestModel rm) {
        FeeRequestModel frm = (FeeRequestModel) rm;
        String str = "";
        if (frm.getFee() < 500) {
            if ("小李".equals(frm.getUser())) {
                str = "项目经理同意" + frm.getUser() + "聚餐费用" + frm.getFee() + "的请求";
            } else {
                str = "项目经理不同意" + frm.getUser() + "聚餐费用" + frm.getFee() + "的请求";
            }
        } else {
            if (this.successor != null) {
                successor.handleRequest(rm);
            }
        }
        return str;
    }
}  
```

```java
 
/**
 * 通用的请求对象
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-5下午9:35:18
 */
public class RequestModel {
    public String type;
    public RequestModel(String type) {
        this.type = type;
    }
    public String getType() {
        return type;
    }
    public void setType(String type) {
        this.type = type;
    }
}  
```

```java
public class FeeRequestModel extends RequestModel {
    public final static String FEE_TYPE = "fee";
    private String user;
    private double fee;
    // 设置自身级别（业务类型）为：费用申请
    // ~这里当然可以有其他的扩充类去继承RequestModel,然后把业务类型设置成其他
    // ~当然这个业务类型可以放到父类去定义
    public FeeRequestModel() {
        super(FEE_TYPE);
    }
    public String getUser() {
        return user;
    }
    public void setUser(String user) {
        this.user = user;
    }
    public double getFee() {
        return fee;
    }
    public void setFee(double fee) {
        this.fee = fee;
    }
}  
```

职责链模式的本质：**分离职责，动态组合**