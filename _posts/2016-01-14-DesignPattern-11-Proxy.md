---
layout: post
title: 《研磨设计模式》代理模式-Proxy
categories: [设计模式]
description: 设计模式|代理模式
keywords: 设计模式,代理,Proxy
autotoc: true
---

本文是《研磨设计模式》第十一章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##一、使用场景

查询所有员工信息

###不使用代理模式的方法

```java

public class UserModel {
    private String userId;
    private String name;
    private String depId;
    private String sex;
    public String getUserId() {
        return userId;
    }
    public void setUserId(String userId) {
        this.userId = userId;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getDepId() {
        return depId;
    }
    public void setDepId(String depId) {
        this.depId = depId;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
}  

```

```java

public class UserManager {
    public Collection<UserModel> getUserById(String depId) throws SQLException {
        Collection<UserModel> col = new ArrayList<UserModel>();
        Connection conn = null;
        conn = this.getConnection();
        String sql = "SELECT * FROM USER u,DEPT d WHERE u.deptId=d.deptId AND d.deptId LIKE ?";
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setString(0, "%" + depId + "%");
        ResultSet rs = ps.executeQuery();
        while (rs.next()) {
            UserModel um = new UserModel();
            um.setDepId(depId);
            um.setName(rs.getString("name"));
            um.setSex(rs.getString("sex"));
            um.setUserId(rs.getString("userId"));
            col.add(um);
        }
        return col;
    }
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection("", "", "");
    }
}  

```

上述代码存在的问题：当数据量较大时，将会消耗较多的内存

- 解决方法：先只查询用户ID和用户名两个字段，如果需要具体的用户信息，再去查询具体信息

##代理模式定义：

**为其他对象提供一种代理以控制对这个对象的访问。**

![](/images/posts/designpattern/Proxy-1.png)

标准的代码模式：

```java

/**
 * 抽象的目标接口，定义具体的目标对象和代理公用的接口
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-27下午09:16:47
 */
public interface Subject {
    public void request();
}  

```

```java

public class RealSubject implements Subject {
    @Override
    public void request() {
      //具体的业务逻辑
    }
}  

```

```java

/**
 * 代理对象
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-27下午09:18:17
 */
public class Proxy implements Subject {
    // 持有被代理的具体的目标对象
    private RealSubject realSubject;
    public Proxy(RealSubject realSubject) {
        this.realSubject = realSubject;
    }
    @Override
    public void request() {
        // 转调之前可以执行一些功能
        // 转调具体的目标对象的方法
        realSubject.request();
        // 转调之后可以执行一些功能
    }
}  

```

使用代理模式重写案例：

```java

/**
 * 定义数据对象的接口
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-27下午09:26:14
 */
public interface UserModelAPI {
    public String getUserId();
    public void setUserId(String userId);
    public String getName();
    public void setName(String name);
    public String getDepId();
    public void setDepId(String depId);
    public String getSex();
    public void setSex(String sex);
}  

```

```java

public class Proxy implements UserModelAPI {
    private UserModel userModel = null;
    public Proxy(UserModel userModel) {
        this.userModel = userModel;
    }
    // 是否加载数据
    private boolean loaded = false;
    @Override
    public String getDepId() {
        if (!loaded) {
            reload();
            loaded = true;
        }
        return userModel.getDepId();
    }
    @Override
    public String getName() {
        return userModel.getName();
    }
    @Override
    public String getSex() {
        if (!loaded) {
            reload();
            loaded = true;
        }
        return userModel.getSex();
    }
    @Override
    public String getUserId() {
        return userModel.getUserId();
    }
    @Override
    public void setDepId(String depId) {
        userModel.setDepId(depId);
    }
    @Override
    public void setName(String name) {
        userModel.setName(name);
    }
    @Override
    public void setSex(String sex) {
        userModel.setSex(sex);
    }
    @Override
    public void setUserId(String userId) {
        userModel.setUserId(userId);
    }
    private void reload() {
        Connection conn = null;
        try {
            conn = this.getConnection();
            String sql = "SELECT * FROM USER u WHERE u.userId=?";
            PreparedStatement ps = conn.prepareStatement(sql);
            ps.setString(0, userModel.getUserId());
            ResultSet rs = ps.executeQuery();
            while (rs.next()) {
                userModel.setDepId(rs.getString("depId"));
                userModel.setSex(rs.getString("sex"));
            }
            rs.close();
            ps.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection("", "", "");
    }
}

```

```java

public class UserManager {
    public Collection<UserModelAPI> getUserById(String depId) throws SQLException {
        Collection<UserModelAPI> col = new ArrayList<UserModelAPI>();
        Connection conn = null;
        conn = this.getConnection();
        // 只要查询UserId和name两个字段就可以了
        String sql = "SELECT u.userId,u.name FROM USER u,DEPT d WHERE u.deptId=d.deptId AND d.deptId LIKE ?";
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setString(0, "%" + depId + "%");
        ResultSet rs = ps.executeQuery();
        while (rs.next()) {
            Proxy proxy = new Proxy(new UserModel());
            proxy.setUserId(rs.getString("userId"));
            proxy.setName(rs.getString("name"));
            col.add(proxy);
        }
        return col;
    }
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection("", "", "");
    }
} 

```

##二、保护代理

场景需求：一旦订单被创建，只有订单的创建人才可以修改订单。

```java

public interface OrderApi {
    public String getProductName();
    public void setProductName(String productName,String user);
    public int getOrderNum();
    public void setOrderNum(int orderNum,String user);
    public String getOrderUser();
    public void setOrderUser(String orderUser,String user);
}  

```

业务对象实现类：

```java

public class Order implements OrderApi {
    private String productName;
    private int orderNum;
    private String orderUser;
    public Order(String productName, int orderNum, String orderUser) {
        this.productName = productName;
        this.orderNum = orderNum;
        this.orderUser = orderUser;
    }
    public String getProductName() {
        return productName;
    }
    public void setProductName(String productName,String user) {
        this.productName = productName;
    }
    public int getOrderNum() {
        return orderNum;
    }
    public void setOrderNum(int orderNum,String user) {
        this.orderNum = orderNum;
    }
    public String getOrderUser() {
        return orderUser;
    }
    public void setOrderUser(String orderUser,String user) {
        this.orderUser = orderUser;
    }
}  

```

代理对象类：
 
 ```java

public class OrderProxy implements OrderApi {
    // 持有目标对象
    private Order order = null;
    public OrderProxy(Order order) {
        this.order = order;
    }
    @Override
    public int getOrderNum() {
        return order.getOrderNum();
    }
    @Override
    public String getOrderUser() {
        return order.getOrderUser();
    }
    @Override
    public String getProductName() {
        return order.getProductName();
    }
    @Override
    public void setOrderNum(int orderNum, String user) {
        // 控制访问权限，只有创建订单的人才能修改
        if (user != null && user.equals(this.getOrderUser())) {
            order.setOrderNum(orderNum, user);
        } else {
            System.out.println("对不起，" + user + "您无权修改本订单的数量！");
        }
    }
    @Override
    public void setOrderUser(String orderUser, String user) {
        // 控制访问权限，只有创建订单的人才能修改
        if (user != null && user.equals(this.getOrderUser())) {
            order.setOrderUser(orderUser, user);
        } else {
            System.out.println("对不起，" + user + "您无权修改本订单的采购人！");
        }
    }
    @Override
    public void setProductName(String productName, String user) {
        // 控制访问权限，只有创建订单的人才能修改
        if (user != null && user.equals(this.getOrderUser())) {
            order.setProductName(productName, user);
        } else {
            System.out.println("对不起，" + user + "您无权修改本订单的产品名称！");
        }
    }
    public String toString() {
        return "productName=" + getProductName() + ",orderNum=" + getOrderNum() + ",orderUser=" + getOrderUser();
    }
}

```

客户端调用：

```java

public class Client {
    public static void main(String[] args) {
        OrderApi order = new OrderProxy(new Order("设计模式", 100, "老孙"));
        // 如果老陈想修改，没门
        order.setOrderNum(20, "老陈");
        System.out.println("老陈改不了订单，还是原样：" + order);
        // 老孙就可以修改
        order.setProductName("老孙的设计模式", "老孙");
        System.out.println("老孙修改了订单：" + order);
    }
}  

```

以上的代理实现方式称之为：**“静态代理”**

- 静态代理的一个缺点是：如果Subject接口发生变化，那么代理类和具体的目标实现都要发生变化。

- 通常把使用Java内建的对代理模式支持的功能来实现的代理称为“动态代理”。

- 动态代理在实现的时候，虽然Subject接口上定义了很多方法，但是动态代理类始终只有一个invoke方法。这样当subject接口发生变化时，动态代理的接口就不需要跟着发生变化了。

```java

public class DynamicProxy implements InvocationHandler {
    private OrderApi order = null;
    /**
     * 获取绑定好代理和具体目标对象后的目标对象的接口
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-7-28下午10:05:56
     * @param order
     *            :具体的订单对象，相当于具体目标对象
     * @return：绑定好代理个具体目标对象后的目标对象的接口
     */
    public OrderApi getProxyInterface(Order order) {
        // 设置好被代理的对象，好方便invoke的操作
        this.order = order;
        // 把真正的订单对象和动态代理类关联起来
        OrderApi orderApi = (OrderApi) Proxy.newProxyInstance(order.getClass().getClassLoader(), order.getClass().getInterfaces(), this);
        return orderApi;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 如果是setter方法就要检查权限
        if (method.getName().startsWith("set")) {
            // 如果不是创建人，就不能修改
            if (order.getOrderUser() != null && order.getOrderUser().equals(args[1])) {
                return method.invoke(order, args);
            } else {
                System.out.println("对不起，" + args[1] + "您无权修改本订单的数据！");
            }
        } else {
            // 不是set方法就继续运行
            return method.invoke(order, args);
        }
        return null;
    }
}  

```

```java

public class Client {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-7-28下午10:13:46
     * @param args
     */
    public static void main(String[] args) {
        Order order = new Order("设计模式", 100, "老孙");
        DynamicProxy dynamicProxy = new DynamicProxy();
        // 把订单和动态代理类关联起来
        OrderApi orderApi = dynamicProxy.getProxyInterface(order);
        orderApi.setOrderNum(200, "老陈");
        System.out.println("老陈改不了订单，还是原样：" + order);
        // 老孙就可以修改
        orderApi.setProductName("老孙的设计模式", "老孙");
        System.out.println("老孙修改了订单：" + order);
    }
} 

```

##三、代理模式的本质

 **控制对象访问**

 *其实，也可以将以上精态代理的方式简化：去掉OrderApi接口，只有Order类。而OrderProxy集成Order类*





