---
layout: post
title: 《设计模式之禅》备忘录模式
categories: [设计模式]
description: 设计模式之禅|备忘录模式
keywords: 设计模式
autotoc: true
comments: true
---

本文是《设计模式之禅》第二十四章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##一、案例场景

男孩追求女孩，状态可多次恢复。

```java
public class Boy {
    // 某一时间点状态
    private String state = "";
    /**
     * 改状态
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-11下午02:57:39
     */
    public void changeState() {
        this.state = "心情可能很不好";
    }
    public void setState(String state) {
        this.state = state;
    }
    public String getState() {
        return state;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Boy boy = new Boy();
        boy.setState("心情很棒！");
        System.out.println("男孩现在的状态===" + boy.getState());
        // 记录下当前时刻状态
        Boy backup = new Boy();
        backup.setState(boy.getState());
        boy.changeState();
        System.out.println("男孩追女孩后的状态===" + boy.getState());
        // 追求失败，恢复原状
        boy.setState(backup.getState());
        System.out.println("男孩恢复后的状态===" + boy.getState());
    }
}  
```

根据单一职责原则，状态记录交给单独的类去完成。使用备忘录模式重写案例：

```java
public class Boy {
    
    private String state="";
    
    public void changeState(){
        this.state="心情可能很不好";
    }
    public String getState() {
        return state;
    }
    public void setState(String state) {
        this.state = state;
    }
    
    //保留一个备份
    public Memento createMemento(){
        return new Memento(this.state);
    }
    
    //恢复一个备份
    public void restoreMemento(Memento _memento){
        this.setState(_memento.getState());
    }
}  
```

```java
public class Memento {
    
    private String state="";
    public Memento(String state) {
        this.state = state;
    }
    public String getState() {
        return state;
    }
    public void setState(String state) {
        this.state = state;
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        Boy boy = new Boy();
        boy.setState("心情很棒！");
        System.out.println("男孩现在的状态===" + boy.getState());
        // 记录下当前状态
        Memento mem = boy.createMemento();
        // 男孩追求女孩，状态改变
        boy.changeState();
        System.out.println("男孩现在的状态===" + boy.getState());
        // 恢复到原来状态
        boy.restoreMemento(mem);
        System.out.println("男孩恢复后的状态===" + boy.getState());
    }
}  
```

继续进行优化，新增备忘录管理者：

```java
public class Client {
    public static void main(String[] args) {
        Boy boy = new Boy();
        boy.setState("心情很棒！");
        System.out.println("男孩现在的状态===" + boy.getState());
        // 记录下当前状态
        Caretaker caretaker = new Caretaker();
        caretaker.setMemento(boy.createMemento());
        // 男孩追求女孩，状态改变
        boy.changeState();
        System.out.println("男孩现在的状态===" + boy.getState());
        // 恢复到原来状态
        boy.restoreMemento(caretaker.getMemento());
        System.out.println("男孩恢复后的状态===" + boy.getState());
    }
}
```

## 二、备忘录模式的定义

**在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。**

- Originator：发起人角色<br/>
  记录当前时刻的内部状态，负责定义哪些属于备份范围的状态，负责创建和恢复备忘录数据。
- Memento：备忘录角色<br/>
  负责存储Originator发起人对象的内部状态，在需要的时候提供发起人需要的内部状态
- Caretaker：备忘录管理员角色

```java
public class Originator {
    private String state = "";
    // 创建一个备忘录
    public Memento createMemento() {
        return new Memento(state);
    }
    // 恢复一个备忘录
    public void restoreMemento(Memento _memento) {
        this.setState(_memento.getState());
    }
    public String getState() {
        return state;
    }
    public void setState(String state) {
        this.state = state;
    }
}
```

```java
public class Memento {
    
    private String state = "";
    public String getState() {
        return state;
    }
    public Memento(String state) {
        super();
        this.state = state;
    }
    public void setState(String state) {
        this.state = state;
    }
}  
```

```java
public class Caretaker {
    private Memento memento;
    public Memento getMemento() {
        return memento;
    }
    public void setMemento(Memento memento) {
        this.memento = memento;
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        // 定义出发起人
        Originator originator = new Originator();
        // 定义备忘录管理员
        Caretaker caretaker = new Caretaker();
        // 创建一个备忘录
        caretaker.setMemento(originator.createMemento());
        // 恢复一个备忘录
        originator.restoreMemento(caretaker.getMemento());
    }
}
```

## 三、扩展后的备忘录模式

备忘录模式可以进行扩展，即把发起人角色进行自身克隆，这样就可以保存当时的快照状态；
由发起者自主备份当前状态、自主恢复：

```java
/**
 * 发起人自主备份、自主恢复
 * 
 * @Package Memento.zen.five
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-11 下午04:13:22
 */
public class Originator implements Cloneable {
    private String state;
    private Originator backup;
    // 创建一个备忘录
    public void createMemento() {
        this.backup = this.clone();
    }
    // 恢复一个备忘录
    public void restoreMemento() {
        this.setState(this.backup.getState());
    }
    // 克隆当前对象
    protected Originator clone() {
        try {
            return (Originator) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }
    public String getState() {
        return state;
    }
    public void setState(String state) {
        this.state = state;
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        // 定义出发起人
        Originator originator = new Originator();
        // 建立初始状态
        originator.setState("初始状态");
        // 建立备份
        originator.createMemento();
        // 修改状态
        originator.setState("修改后的状态");
        // 恢复先前状态
        originator.restoreMemento();
    }
}  
```

多状态的备忘录模式：

```java
public class Originator {
    private String state1 = "";
    private String state2 = "";
    private String state3 = "";
    // 创建一个备忘录
    public Memento createMemento() {
        return new Memento(BeanUtils.backupProp(this));
    }
    // 恢复一个备忘录
    public void restoreMemento(Memento _memento) {
        BeanUtils.restoreProp(this, _memento.getStateMap());
    }
    public String getState1() {
        return state1;
    }
    public void setState1(String state1) {
        this.state1 = state1;
    }
    public String getState2() {
        return state2;
    }
    public void setState2(String state2) {
        this.state2 = state2;
    }
    public String getState3() {
        return state3;
    }
    public void setState3(String state3) {
        this.state3 = state3;
    }
    public String toString() {
        return "state1=" + state1 + "  state2=" + state2 + " state3=" + state3;
    }
}  
```

```java
public class Memento {
    private HashMap<String, Object> stateMap;
    public Memento(HashMap<String, Object> stateMap) {
        super();
        this.stateMap = stateMap;
    }
    public HashMap<String, Object> getStateMap() {
        return stateMap;
    }
    public void setStateMap(HashMap<String, Object> stateMap) {
        this.stateMap = stateMap;
    }
}  
```

```java
public class Caretaker {
    private Memento memento;
    public Memento getMemento() {
        return memento;
    }
    public void setMemento(Memento memento) {
        this.memento = memento;
    }
}  
```

```java
public class BeanUtils {
    /**
     * 把bean的所有属性及数值放入到Hashmap中
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-11下午04:32:51
     * @param bean
     * @return
     */
    public static HashMap<String, Object> backupProp(Object bean) {
        HashMap<String, Object> result = new HashMap<String, Object>();
        try {
            // 获取bean描述
            BeanInfo beanInfo = Introspector.getBeanInfo(bean.getClass());
            // 获取属性描述
            PropertyDescriptor[] descriptors = beanInfo.getPropertyDescriptors();
            // 遍历所有属性
            for (PropertyDescriptor des : descriptors) {
                // 属性名称
                String fieldName = des.getName();
                // 读取属性的方法
                Method getter = des.getReadMethod();
                // 读取属性值
                Object fieldValue = getter.invoke(bean, new Object[] {});
                if (!fieldName.equalsIgnoreCase("class")) {
                    result.put(fieldName, fieldValue);
                }
            }
        } catch (IntrospectionException e) {
            e.printStackTrace();
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
        return result;
    }
    /**
     * 把Hashmap中的值返回到bean中
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-11下午04:33:20
     * @param bean
     * @param proMap
     */
    public static void restoreProp(Object bean, HashMap<String, Object> propMap) {
        // 获取bean描述
        BeanInfo beanInfo;
        try {
            beanInfo = Introspector.getBeanInfo(bean.getClass());
            // 获取属性描述
            PropertyDescriptor[] descriptors = beanInfo.getPropertyDescriptors();
            // 遍历所有属性
            for (PropertyDescriptor des : descriptors) {
                // 属性名称
                String fieldName = des.getName();
                if (propMap.containsKey(fieldName)) {
                    Method setter = des.getWriteMethod();
                    setter.invoke(bean, new Object[] { propMap.get(fieldName) });
                }
            }
        } catch (IntrospectionException e) {
            e.printStackTrace();
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Originator originator=new Originator();
        originator.setState1("中国");
        originator.setState2("强盛");
        originator.setState3("繁荣");
        System.out.println(originator.toString());
        //备份当前状态
        Caretaker caretaker=new Caretaker();
        caretaker.setMemento(originator.createMemento());
         
        //修改状态
        originator.setState1("美国");
        originator.setState2("牛逼");
        originator.setState3("吊炸");
        System.out.println(originator.toString());
        
        //恢复状态
        originator.restoreMemento(caretaker.getMemento());
        System.out.println(originator.toString());
        
    }
}
```