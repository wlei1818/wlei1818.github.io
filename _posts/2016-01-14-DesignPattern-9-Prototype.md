---
layout: post
title: 《研磨设计模式》原型模式-Prototype
categories: [设计模式]
description: 设计模式|原型模式
keywords: 设计模式,原型,Prototype
autotoc: true
---

本文是《研磨设计模式》第九章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##一、使用场景

将订单中拆分，直至每个订单中的产品数量不超过1000

##二、代码实战

1、不使用原型模式的写法：

```java
package Prototype.one;
public interface OrderApi {
    /**
     * 获取订单产品数量
     * 
     * @description
     * @title getOrderProductNum
     * @author sunhanbin
     * @date 2015年7月11日 下午2:22:41
     * @param @return
     * @return int
     */
    public int getOrderProductNum();
    /**
     * 设置订单产品数量
     * 
     * @description
     * @title setOrderProductNum
     * @author sunhanbin
     * @date 2015年7月11日 下午2:23:05
     * @param @param num
     * @return void
     */
    public void setOrderProductNum(int num);
}
```

```java
package Prototype.one;
/**
 * 个人订单对象
 * 
 * @description
 * 
 * @classname PersonalOrder
 * @author sunhanbin
 * @date 2015年7月11日 下午2:23:49
 * @version 1.0
 */
public class PersonalOrder implements OrderApi {
    private String customerName;
    private String productId;
    private int orderProductNum = 0;
    public int getOrderProductNum() {
        return this.orderProductNum;
    }
    public void setOrderProductNum(int num) {
        this.orderProductNum = num;
    }
    public String getCustomerName() {
        return customerName;
    }
    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }
    public String getProductId() {
        return productId;
    }
    public void setProductId(String productId) {
        this.productId = productId;
    }
    public String toString() {
        return "该订单的订购人是" + this.customerName + ",订购的产品是" + this.productId
                + ",订购数量为" + this.orderProductNum;
    }
}
```

```java
package Prototype.one;
/**
 * 企业订单
 * 
 * @description
 * 
 * @classname EnterpriseOrder
 * @author sunhanbin
 * @date 2015年7月11日 下午2:28:32
 * @version 1.0
 */
public class EnterpriseOrder implements OrderApi {
    private String enterpriseName;
    private String productId;
    private int orderProductNum = 0;
    public int getOrderProductNum() {
        return this.orderProductNum;
    }
    public void setOrderProductNum(int num) {
        this.orderProductNum = num;
    }
    public String getEnterpriseName() {
        return enterpriseName;
    }
    public void setEnterpriseName(String enterpriseName) {
        this.enterpriseName = enterpriseName;
    }
    public String getProductId() {
        return productId;
    }
    public void setProductId(String productId) {
        this.productId = productId;
    }
    
    public String toString() {
        return "该订单的订购企业是" + this.enterpriseName + ",订购的产品是" + this.productId
                + ",订购数量为" + this.orderProductNum;
    }
}
```

处理订单拆分的业务类：

```java
package Prototype.one;
/**
 * 处理订单的业务对象
 * 
 * @description
 * 
 * @classname OrderBuisness
 * @author sunhanbin
 * @date 2015年7月11日 下午2:31:26
 * @version 1.0
 */
public class OrderBusiness {
    /**
     * 保存订单的方法
     * 
     * @description
     * @title saveOrder
     * @author sunhanbin
     * @date 2015年7月11日 下午2:32:00
     * @param @param order
     * @return void
     */
    public void saveOrder(OrderApi order) {
        while (order.getOrderProductNum() > 1000) {
            OrderApi newOrder = null;
            // 当产品数量大于1000时，就必须拆分新订单。但是无法知道订单究竟是个人订单还是企业订单！
            // 下面先提供一个解决方式
            if (order instanceof PersonalOrder) {
                PersonalOrder p2 = new PersonalOrder();
                PersonalOrder p1 = (PersonalOrder) order;
                p2.setCustomerName(p1.getCustomerName());
                p2.setProductId(p1.getProductId());
                p2.setOrderProductNum(1000);
                newOrder = p2;
            } else if (order instanceof EnterpriseOrder) {
                EnterpriseOrder e2 = new EnterpriseOrder();
                EnterpriseOrder e1 = (EnterpriseOrder) order;
                e2.setEnterpriseName(e1.getEnterpriseName());
                e2.setProductId(e1.getProductId());
                e2.setOrderProductNum(1000);
                newOrder = e2;
            }
            // 原来的订单数量减少1000
            order.setOrderProductNum(order.getOrderProductNum() - 1000);
            System.out.println("拆分生成的订单为==" + newOrder);
        }
        System.out.println("订单为===" + order);
    }
}
```

客户端调用：

```java
package Prototype.one;
public class Client {
    public static void main(String[] args) {
        PersonalOrder op = new PersonalOrder();
        op.setCustomerName("孙汉斌");
        op.setOrderProductNum(2900);
        op.setProductId("P001");
        OrderBusiness ob = new OrderBusiness();
        ob.saveOrder(op);
    }
}
```

##三、原型模式的定义

- 用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象

- 通过原型实例创建新的对象，就不需要在关心这个实例本身的类型，也不关心它的具体实现，只要它克隆了自身的方法，就可以通过这个方法来获取新的对象，而无需通过new来创建；

![](/images/posts/designpattern/Prototype-1.png)

###原型模式改写代码

在订单接口里增加克隆方法：

```java
package Prototype.two;
public interface OrderApi {
    /**
     * 获取订单产品数量
     * 
     * @description
     * @title getOrderProductNum
     * @author sunhanbin
     * @date 2015年7月11日 下午2:22:41
     * @param @return
     * @return int
     */
    public int getOrderProductNum();
    /**
     * 设置订单产品数量
     * 
     * @description
     * @title setOrderProductNum
     * @author sunhanbin
     * @date 2015年7月11日 下午2:23:05
     * @param @param num
     * @return void
     */
    public void setOrderProductNum(int num);
    /**
     * 实例的克隆方法
     * 
     * @description
     * @title cloneOrder
     * @author sunhanbin
     * @date 2015年7月11日 下午2:55:51
     * @param @return
     * @return OrderApi
     */
    public OrderApi cloneOrder();
}
```

```java
package Prototype.two;
/**
 * 个人订单对象
 * 
 * @description
 * 
 * @classname PersonalOrder
 * @author sunhanbin
 * @date 2015年7月11日 下午2:23:49
 * @version 1.0
 */
public class PersonalOrder implements OrderApi {
    private String customerName;
    private String productId;
    private int orderProductNum = 0;
    public int getOrderProductNum() {
        return this.orderProductNum;
    }
    public void setOrderProductNum(int num) {
        this.orderProductNum = num;
    }
    public String getCustomerName() {
        return customerName;
    }
    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }
    public String getProductId() {
        return productId;
    }
    public void setProductId(String productId) {
        this.productId = productId;
    }
    public String toString() {
        return "该订单的订购人是" + this.customerName + ",订购的产品是" + this.productId
                + ",订购数量为" + this.orderProductNum;
    }
    /**
     * 实例的克隆方法
     * 
     * @description
     * @title cloneOrder
     * @author sunhanbin
     * @date 2015年7月11日 下午2:55:13
     * @param @return
     * @return PersonalOrder
     */
    public PersonalOrder cloneOrder() {
        PersonalOrder order = new PersonalOrder();
        order.setCustomerName(this.customerName);
        order.setOrderProductNum(this.orderProductNum);
        order.setProductId(this.productId);
        return order;
    }
}
```

业务类里无需再判断是属于哪个子类了：

```java
package Prototype.two;
/**
 * 处理订单的业务对象
 * 
 * @description
 * 
 * @classname OrderBuisness
 * @author sunhanbin
 * @date 2015年7月11日 下午2:31:26
 * @version 1.0
 */
public class OrderBusiness {
    /**
     * 保存订单的方法
     * 
     * @description
     * @title saveOrder
     * @author sunhanbin
     * @date 2015年7月11日 下午2:32:00
     * @param @param order
     * @return void
     */
    public void saveOrder(OrderApi order) {
        while (order.getOrderProductNum() > 1000) {
            // 当产品数量大于1000时，就必须拆分新订单。
            // 由于实现了实例的自我克隆方法，因此这里无需在判断是个人订单还是企业订单
            OrderApi newOrder = order.cloneOrder();
            newOrder.setOrderProductNum(1000);
            // 原来的订单数量减少1000
            order.setOrderProductNum(order.getOrderProductNum() - 1000);
            System.out.println("拆分生成的订单为==" + newOrder);
        }
        System.out.println("订单为===" + order);
    }
}
```

使用Java原生的clone方法来实现原型模式：
- 接口中不再定义克隆方法，而是由实现类去实现Java的克隆接口：

```java
package Prototype.three;
public interface OrderApi {
    /**
     * 获取订单产品数量
     * 
     * @description
     * @title getOrderProductNum
     * @author sunhanbin
     * @date 2015年7月11日 下午2:22:41
     * @param @return
     * @return int
     */
    public int getOrderProductNum();
}
package Prototype.three;
import Prototype.one.OrderApi;
public class PersonalOrder implements OrderApi, Cloneable {
    private String customerName;
    private String productId;
    private int orderProductNum = 0;
    public String getCustomerName() {
        return customerName;
    }
    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }
    public String getProductId() {
        return productId;
    }
    public void setProductId(String productId) {
        this.productId = productId;
    }
    public int getOrderProductNum() {
        return orderProductNum;
    }
    public void setOrderProductNum(int orderProductNum) {
        this.orderProductNum = orderProductNum;
    }
    public String toString() {
        return "该订单的订购人是" + this.customerName + ",订购的产品是" + this.productId
                + ",订购数量为" + this.orderProductNum;
    }
    /**
     * 克隆方法
     */
    public Object clone() {
        Object obj = null;
        // 直接调用父类的克隆方法就可以真正实现克隆
        try {
            obj = super.clone();
        } catch (CloneNotSupportedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return obj;
    }
}
```

以上只是浅度克隆，浅度克隆与深度克隆的区别：<br/>
- 浅度克隆：只负责克隆按值传递的数据（比如基本数据类型、String类型）；
- 深度克隆：除了浅度克隆要克隆的值外，还负责克隆引用类型的数据，基本上就是被克隆实例所有的属性数据都会被克隆出来；
- 深度克隆如果被克隆的对象里面的属性数据也是引用类型，也就是说属性的类型也是对象，则需要一直递归的克隆下去。也就是说，要想深度克隆，必须整个克隆所涉及的对象都要正确实现克隆方法，如果其中一个没有实现，那么就会导致克隆失败。
- 深度克隆例子：【用Java的clone方法来实现】

```java
package Prototype.four;
/**
 * 使用Java的clone方法来实现深度克隆
 * 
 * @description
 * 
 * @classname Product
 * @author sunhanbin
 * @date 2015年7月12日 上午10:27:48
 * @version 1.0
 */
public class Product implements Cloneable {
    private String productId;
    private String name;
    public String getProductId() {
        return productId;
    }
    public void setProductId(String productId) {
        this.productId = productId;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String toString() {
        return "该产品编号==" + this.productId + ",产品名称==" + this.name;
    }
    /**
     * 克隆对象
     */
    public Object clone() {
        Object obj = null;
        try {
            obj = super.clone();
        } catch (CloneNotSupportedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return obj;
    }
}
```

*注意：下面的PersonalOrder对象持有了Product对象属性，那么在PersonalOrder的克隆方法中，也要把这个把这个对象属性进行**递归克隆**！！！！！*

```java
package Prototype.four;
import Prototype.one.OrderApi;
public class PersonalOrder implements Cloneable, OrderApi {
    private String customerName;
    private int orderProductNum = 0;
    private Product product;// 注意：这里持有对象属性！！！
    public String getCustomerName() {
        return customerName;
    }
    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }
    public int getOrderProductNum() {
        return orderProductNum;
    }
    public void setOrderProductNum(int orderProductNum) {
        this.orderProductNum = orderProductNum;
    }
    public Product getProduct() {
        return product;
    }
    public void setProduct(Product product) {
        this.product = product;
    }
    public String toString() {
        return "该订单的订购人是" + this.customerName + ",订购的产品是"
                + this.product.getProductId() + ",订购数量为" + this.orderProductNum;
    }
    public Object clone() {
        PersonalOrder obj = null;
        try {
            obj = (PersonalOrder) super.clone();
            // 注意！！！这么这句话十分重要，一定要把持有的属性对象也进行递归克隆！！！！
            obj.setProduct((Product) this.product.clone());
        } catch (CloneNotSupportedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return obj;
    }
}
```

为什么不能少了这句话：**obj.setProduct((Product) this.product.clone());**<br/>
因为在调用super.clone()这个方法时，Java是先开辟一片内存的空间，然后把对象的值原样拷贝过去，对于基本类型这样做是没有问题，而属性product是引用类型，把值拷贝过去的意思就是把对应的内存地址拷贝过去，也就是说克隆对象实例的product和原型对象实例的product指向的是同一块内存空间，是同一个实例。<br/>
也就是说，如果不对**属性对象**进行克隆的话，那么会影响到原型实例！！

