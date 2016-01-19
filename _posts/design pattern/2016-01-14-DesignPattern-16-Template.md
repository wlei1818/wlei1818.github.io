---
layout: post
title: 《研磨设计模式》模板方法模式-Template
categories: [设计模式]
description: 设计模式|模板方法模式
keywords: 设计模式,模板,Template
autotoc: true
comments: true
---

本文是《研磨设计模式》第十六章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。


## 案例场景

用户登录：前台登录与后台登录

### 不用模板方法模式

```java

/**
 * 描述登录人员登录时填写的信息的数据模型
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-10下午8:37:51
 */
public class LoginModel {
    
    private String userId,pwd;
    public String getUserId() {
        return userId;
    }
    public void setUserId(String userId) {
        this.userId = userId;
    }
    public String getPwd() {
        return pwd;
    }
    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
}
```

```java
public class UserModel {
    private String uuid, userId, pwd, name;
    public String getUuid() {
        return uuid;
    }
    public void setUuid(String uuid) {
        this.uuid = uuid;
    }
    public String getUserId() {
        return userId;
    }
    public void setUserId(String userId) {
        this.userId = userId;
    }
    public String getPwd() {
        return pwd;
    }
    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

```java
/**
 * 普通登录
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-10下午8:36:37
 */
public class NormalLogin {
    /**
     * 后台登录从数据库校验
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午8:41:13
     * @param lm
     * @return
     */
    public boolean login(LoginModel lm) {
        UserModel um = this.findUserByUserId(lm.getUserId());
        if (um != null) {
            if (um.getUserId().equals(lm.getUserId()) && um.getPwd().equals(lm.getPwd())) {
                return true;
            }
        }
        return false;
    }
    private UserModel findUserByUserId(String userId) {
        UserModel um = new UserModel();
        um.setUserId(userId);
        um.setName("test");
        um.setPwd("test");
        um.setUuid("User001");
        return um;
    }
}
```

```java
public class LoginModel2 {
    private String workerId, pwd;
    public String getWorkerId() {
        return workerId;
    }
    public void setWorkerId(String workerId) {
        this.workerId = workerId;
    }
    public String getPwd() {
        return pwd;
    }
    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
}
```

```java
public class WorkerModel {
    private String uuid, workerId, pwd, name;
    public String getUuid() {
        return uuid;
    }
    public void setUuid(String uuid) {
        this.uuid = uuid;
    }
    public String getWorkerId() {
        return workerId;
    }
    public void setWorkerId(String workerId) {
        this.workerId = workerId;
    }
    public String getPwd() {
        return pwd;
    }
    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

```java
/**
 * 工作人员登录
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-10下午8:45:32
 */
public class WorkerLogin {
    public boolean login(LoginModel2 lm) {
        WorkerModel wm = findWorkerByWorkerId(lm.getWorkerId());
        if (wm != null) {
            if (lm.getPwd().equals(wm.getPwd()) && lm.getWorkerId().equals(wm.getWorkerId())) {
                return true;
            }
        }
        return false;
    }
    private WorkerModel findWorkerByWorkerId(String workerId) {
        WorkerModel wm = new WorkerModel();
        wm.setWorkerId(workerId);
        wm.setName("Worker1");
        wm.setPwd("test");
        wm.setUuid("Worker001");
        return wm;
    }
}
```

## 模板方法的定义

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重新定义该算法的某些特定步骤。

- AbstractClass：抽象类，用来定义算法骨架和原语操作

- ConcreteClass：具体实现类

模板方法示例代码：

```java
/**
 * 定义模板方法、原语操作等的抽象类
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-10下午8:57:52
 */
public abstract class AbstractClass {
    // 原语操作1：即抽象操作，必须由子类提供具体实现
    public abstract void doPrimitiveOperation1();
    // 原语操作2
    public abstract void doPrimitiveOperation2();
    // 模板方法，定义算法骨架
    public final void templateMethod() {
        doPrimitiveOperation1();
        doPrimitiveOperation2();
    }
}  
```

```java
public class ConcreteClass extends AbstractClass {
    @Override
    public void doPrimitiveOperation1() {
        // 具体实现
    }
    @Override
    public void doPrimitiveOperation2() {
        // 具体实现
    }
}  
```

使用模板方法重写案例：

```java
public class LoginModel {
    private String loginId;
    private String pwd;
    public String getLoginId() {
        return loginId;
    }
    public void setLoginId(String loginId) {
        this.loginId = loginId;
    }
    public String getPwd() {
        return pwd;
    }
    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
}  
```

```java
/**
 * 定义公共的登录控制的算法骨架
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-10下午9:09:57
 */
public abstract class LoginTemplate {
    public final boolean login(LoginModel lm) {
        LoginModel dblm = findLoginUser(lm.getLoginId());
        if (dblm != null) {
            lm.setPwd(lm.getPwd());
            return this.match(lm, dblm);
        }
        return false;
    }
    public abstract LoginModel findLoginUser(String loginId);
    /**
     * 判断用户登录信息是否与数据库匹配
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午9:16:33
     * @param lm
     * @param dblm
     * @return
     */
    public boolean match(LoginModel lm, LoginModel dblm) {
        if (lm.getLoginId().equals(dblm.getLoginId()) && lm.getPwd().equals(dblm.getPwd())) {
            return true;
        }
        return false;
    }
}
```

```java
public class NormalLogin extends LoginTemplate {
    @Override
    public LoginModel findLoginUser(String loginId) {
        // 这里只是做个示意，因此简单返回
        LoginModel lm = new LoginModel();
        lm.setLoginId(loginId);
        lm.setPwd("test");
        return lm;
    }
}
```

```java
public class WorkerLogin extends LoginTemplate {
    @Override
    public LoginModel findLoginUser(String loginId) {
        // 这里只是做个示意，因此简单返回
        LoginModel lm = new LoginModel();
        lm.setLoginId(loginId);
        lm.setPwd("test");
        return lm;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        LoginModel lm = new LoginModel();
        lm.setLoginId("admin");
        lm.setPwd("workerpwd");
        LoginTemplate lt = new WorkerLogin();
        LoginTemplate lt2 = new NormalLogin();
        // 进行登录测试
        boolean flag = lt.login(lm);
        System.out.println("可以登录工作平台=" + flag);
        boolean flag2 = lt2.login(lm);
        System.out.println("可以进行普通的登录=" + flag2);
    }
}
```

模板方法中，作为父类的模板会在需要的时候，调用子类相应的方法，也就是由父类来找子类，而不是让子类来找父类。
因为Java的动态绑定采用的是**“后期绑定”**技术。一句话：**new谁就调用谁的方法。**

一个完成的模板方法示例：

```java
public abstract class AbstractTemplate {
    public final void templateMethod() {
        // 第一步
        this.operation1();
        // 第二步
        this.operation2();
        // 第三步
        this.doPrimitiveOperation1();
        // 第四步
        this.doPrimitiveOperation2();
        // 第五步
        this.hookOperation();
    }
    /**
     * 算法1：固定实现，而且子类不需要访问
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午9:38:49
     */
    private void operation1() {
        //
    }
    /**
     * 具体操作2，固定实现，子类可能需要访问.当然也可以定义成protected，不可以被覆盖，因此是final
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午9:39:35
     */
    private final void operation2() {
    }
    /**
     * 子类的公共功能 但通常不是具体的算法步骤
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午9:40:42
     */
    protected void commonOperation() {
    }
    /**
     * 原语操作1，算法中的必要不在背后。父类无法确定如何真正实现，需要子类实现
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午9:41:32
     */
    protected abstract void doPrimitiveOperation1();
    /**
     * 原语操作2，算法中的必要不在背后。父类无法确定如何真正实现，需要子类实现
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午9:42:19
     */
    protected abstract void doPrimitiveOperation2();
    /**
     * 钩子操作，算法中的步骤，不一定需要，提供默认实现 .由子类选择并具体实现
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午9:43:02
     */
    protected void hookOperation() {
        // 提供默认实现
    }
    /**
     * 工厂方法，创建某个对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-10下午9:44:14
     * @return
     */
    protected abstract Object createOneObject();
}  
```

## Java回调与模板方法模式

使用Java回调方式来实现模板方法模式，即使用匿名内部类方式

1. 先定义一个模板方法需要的回调接口

```java
/**
 * 登录控制的模板方法需要的回调接口，需要把所有需要的接口方法都定义出来。 或者说是所有可以被扩展的方法都需要被定义出来
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-10下午9:48:44
 */
public interface LoginCallback {
    // 根据编号来查找和获取中的相应数据
    public LoginModel findLoginUser(String userId);
    // 数据加密
    public String encryptPwd(String pwd, LoginTemplate template);
    // 登录匹配
    public boolean match(LoginModel lm, LoginModel dblm, LoginTemplate tempalte);
}  
```

```java
public class LoginTemplate {
    public final boolean login(LoginModel lm, LoginCallback callback) {
        LoginModel dblm = callback.findLoginUser(lm.getLoginId());
        if (dblm != null) {
            String encryptPwd = callback.encryptPwd(lm.getPwd(), this);
            lm.setPwd(encryptPwd);
            return callback.match(lm, dblm, this);
        }
        return false;
    }
    public String encryptPwd(String pwd) {
        return pwd;
    }
    public boolean match(LoginModel lm, LoginModel dblm) {
        if (lm.getLoginId().equals(dblm.getLoginId()) && lm.getPwd().equals(dblm.getPwd())) {
            return true;
        }
        return false;
    }
}  
public class Client {
    public static void main(String[] args) {
        LoginModel lm = new LoginModel();
        lm.setLoginId("admin");
        lm.setPwd("workerpwd");
        LoginTemplate lt = new LoginTemplate();
        boolean flag = lt.login(lm, new LoginCallback() {
            @Override
            public LoginModel findLoginUser(String userId) {
                LoginModel lm = new LoginModel();
                lm.setLoginId(userId);
                lm.setPwd("testpwd");
                return lm;
            }
            // 自己不需要实现，转调模板方法中的方法
            public String encryptPwd(String pwd, LoginTemplate template) {
                return template.encryptPwd(pwd);
            }
            @Override
            public boolean match(LoginModel lm, LoginModel dblm, LoginTemplate tempalte) {
                return tempalte.match(lm, dblm);
            }
        });
        System.out.println("普通人员可以登录=" + flag);
    }
}  
```

模板方法的典型应用：*排序*

```java
public class Client {
    public static void main(String[] args) {
        UserModel u1 = new UserModel("u1", "user1", 10);
        UserModel u2 = new UserModel("u2", "user2", 20);
        UserModel u3 = new UserModel("u3", "user3", 30);
        UserModel u4 = new UserModel("u4", "user4", 40);
        UserModel u5 = new UserModel("u5", "user5", 50);
        List<UserModel> list = new ArrayList<UserModel>();
        list.add(u1);
        list.add(u3);
        list.add(u2);
        list.add(u5);
        list.add(u4);
        System.out.println("排序前=======");
        printList(list);
        // 实现一个比较器，也可以单独用类实现
        // ~也可以直接在sort()方法中用匿名内部类实现
        Comparator c = new Comparator() {
            @Override
            public int compare(Object o1, Object o2) {
                // 按照年龄升序排序
                UserModel tempU1 = (UserModel) o1;
                UserModel tempU2 = (UserModel) o2;
                if (tempU1.getAge() > tempU2.getAge()) {
                    return 1;
                } else if (tempU1.getAge() == tempU2.getAge()) {
                    return 0;
                } else if (tempU1.getAge() < tempU2.getAge()) {
                    return -1;
                }
                return 0;
            }
        };
        // 排序
        Collections.sort(list, c);
        System.out.println("排序后");
        printList(list);
    }
    private static void printList(List<UserModel> list) {
        for (UserModel um : list) {
            System.out.println(um);
        }
    }
}
```






