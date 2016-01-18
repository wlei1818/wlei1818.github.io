---
layout: post
title: 《设计模式之禅》组合模式
categories: [设计模式]
description: 设计模式之禅|组合模式
keywords: 设计模式
autotoc: true
---

本文是《设计模式之禅》第二十一章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、案例场景

数据结构中的二叉树遍历：如公司的人事组织架构 就是一个典型的树结构。

不使用模式的写法：

```java
/**
 * 根节点
 * 
 * @Package Composite.zen.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-7 上午10:28:38
 */
public interface IRoot {
    public String getInfo();
    // 增加分支节点
    public void add(IBranch branch);
    // 增加叶子节点
    public void add(ILeaf leaf);
    // 遍历
    public ArrayList getSubordinateInfo();
}  
```

```java
public class Root implements IRoot {
    private ArrayList subordinateList = new ArrayList();
    private String name = "";
    private String position = "";
    private int salary = 0;
    public Root(String name, String position, int salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7上午10:31:01
     * @param branch
     */
    @Override
    public void add(IBranch branch) {
        subordinateList.add(branch);
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7上午10:31:01
     * @return
     */
    @Override
    public String getInfo() {
        String info = "";
        info = "名称：" + name;
        info += "\t职位：" + position;
        info += "\t薪水：" + salary;
        return info;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7上午10:31:01
     * @return
     */
    @Override
    public ArrayList getSubordinateInfo() {
        return this.subordinateList;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7上午10:31:06
     * @param leaf
     */
    @Override
    public void add(ILeaf leaf) {
        subordinateList.add(leaf);
    }
}
```

```java
public interface IBranch {
    // 获得信息
    public String getInfo();
    // 增加数据节点，例如研发部下设的研发一组
    public void add(IBranch branch);
    // 增设叶子节点
    public void add(ILeaf leaf);
    // 获得下级信息
    public ArrayList getSubordinateInfo();
}  
```

```java
public class Branch implements IBranch {
    private ArrayList subordinateList = new ArrayList();
    private String name = "";
    private String position = "";
    private int salary = 0;
    public Branch(String name, String position, int salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7上午10:46:47
     * @param branch
     */
    @Override
    public void add(IBranch branch) {
        subordinateList.add(branch);
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7上午10:46:47
     * @param leaf
     */
    @Override
    public void add(ILeaf leaf) {
        subordinateList.add(leaf);
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7上午10:46:47
     * @return
     */
    @Override
    public String getInfo() {
        String info = "";
        info = "名称：" + name;
        info += "\t职位：" + position;
        info += "\t薪水：" + salary;
        return info;
    }
    @Override
    public ArrayList getSubordinateInfo() {
        return subordinateList;
    }
}
```

```java
public interface ILeaf {
    // 获得自己的信息
    public String getInfo();
}  
```

```java
public class Leaf implements ILeaf {
    private ArrayList subordinateList = new ArrayList();
    private String name = "";
    private String position = "";
    private int salary = 0;
    public Leaf(String name, String position, int salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    @Override
    public String getInfo() {
        String info = "";
        info = "名称：" + name;
        info += "\t职位：" + position;
        info += "\t薪水：" + salary;
        return info;
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        // 首先产生了一个根节点
        IRoot ceo = new Root("王大麻子", "总经理", 200000);
        // 产生三个部门经理，也就是树枝节点
        IBranch developDep = new Branch("刘大瘸子", "研发经理", 10000);
        IBranch salesDep = new Branch("马二拐子", "销售部门经理", 20000);
        IBranch financeDep = new Branch("赵三驼子", "财务部经理", 30000);
        // 三个小组长
        IBranch firstDevGroup = new Branch("杨三", "开发一组组长", 5000);
        IBranch secondDevGroup = new Branch("吴大锤子", "开发二组组长", 6000);
        // 剩下的叶子节点，小罗罗
        ILeaf a = new Leaf("a", "开发人员", 2000);
        ILeaf b = new Leaf("b", "开发人员", 2000);
        ILeaf c = new Leaf("c", "开发人员", 2000);
        ILeaf d = new Leaf("d", "开发人员", 2000);
        ILeaf e = new Leaf("e", "开发人员", 2000);
        ILeaf f = new Leaf("f", "销售人员", 2000);
        ILeaf g = new Leaf("g", "销售人员", 2000);
        ILeaf h = new Leaf("h", "财务人员", 2000);
        ILeaf i = new Leaf("i", "财务人员", 2000);
        ILeaf j = new Leaf("i", "CEO秘书", 2000);
        ILeaf laoliu = new Leaf("郑老六", "研发部副总", 20000);
        // 定义总经理下面的三个部门经理
        ceo.add(developDep);
        ceo.add(salesDep);
        ceo.add(financeDep);
        // 总经理下面有一个秘书
        ceo.add(j);
        // 定义研发部门下的结构
        developDep.add(firstDevGroup);
        developDep.add(secondDevGroup);
        // 研发部下面还有一个副总
        developDep.add(laoliu);
        // 小组下面的开发人员
        firstDevGroup.add(a);
        firstDevGroup.add(b);
        firstDevGroup.add(c);
        secondDevGroup.add(d);
        secondDevGroup.add(e);
        // 销售部下面的销售人员
        salesDep.add(f);
        salesDep.add(g);
        // 财务下面的财务人员
        financeDep.add(h);
        financeDep.add(i);
        // 打印出树的完整形状
        System.out.println(ceo.getInfo());
        getAllSubordinateInfo(ceo.getSubordinateInfo());
    }
    private static void getAllSubordinateInfo(ArrayList subordinateList) {
        int len = subordinateList.size();
        for (int m = 0; m < len; m++) {
            Object s = subordinateList.get(m);
            if (s instanceof Leaf) {// 叶子节点
                ILeaf employee = (Leaf) s;
                System.out.println(((Leaf) s).getInfo());
            } else {
                IBranch branch = (IBranch) s;
                System.out.println(branch.getInfo());
                getAllSubordinateInfo(branch.getSubordinateInfo());
            }
        }
    }
}  
```

使用组合模式的写法：对类进行了抽象而已

```java
public interface ICorp {
    
    public String getInfo();
} 
```

```java
public interface IBranch extends ICorp {
    // 能增加树叶节点或者树枝节点
    public void addSubordinate(ICorp corp);
    // 获取下属信息
    public ArrayList<ICorp> getSubordinate();
}  
```

```java
public class Branch implements IBranch {
    private String name = "";
    private String position = "";
    private int salary = 0;
    private ArrayList subordinateList = new ArrayList();
    public Branch(String name, String position, int salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    @Override
    public void addSubordinate(ICorp corp) {
        subordinateList.add(corp);
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7下午04:59:07
     * @return
     */
    @Override
    public ArrayList<ICorp> getSubordinate() {
        return subordinateList;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-7下午04:59:07
     * @return
     */
    public String getInfo() {
        String info = "";
        info = "名称：" + name;
        info += "\t职位：" + position;
        info += "\t薪水：" + salary;
        return info;
    }
}
```

```java
public interface ILeaf extends ICorp {
    
}  
```

```java
public class Leaf implements ILeaf {
    private String name = "";
    private String position = "";
    private int salary = 0;
    public Leaf(String name, String position, int salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    public String getInfo() {
        String info = "";
        info = "名称：" + name;
        info += "\t职位：" + position;
        info += "\t薪水：" + salary;
        return info;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Branch ceo = compositeCorpTree();
        System.out.println(ceo.getInfo());
        System.out.println(getTreeInfo(ceo));
    }
    public static Branch compositeCorpTree() {
        // 总经理
        Branch root = new Branch("王大麻子", "总经理", 200000);
        // 三个部门经理
        IBranch developDep = new Branch("刘大瘸子", "研发经理", 10000);
        IBranch salesDep = new Branch("马二拐子", "销售部门经理", 20000);
        IBranch financeDep = new Branch("赵三驼子", "财务部经理", 30000);
        // 三个小组长
        IBranch firstDevGroup = new Branch("杨三", "开发一组组长", 5000);
        IBranch secondDevGroup = new Branch("吴大锤子", "开发二组组长", 6000);
        // 剩下的叶子节点，小罗罗
        ILeaf a = new Leaf("a", "开发人员", 2000);
        ILeaf b = new Leaf("b", "开发人员", 2000);
        ILeaf c = new Leaf("c", "开发人员", 2000);
        ILeaf d = new Leaf("d", "开发人员", 2000);
        ILeaf e = new Leaf("e", "开发人员", 2000);
        ILeaf f = new Leaf("f", "销售人员", 2000);
        ILeaf g = new Leaf("g", "销售人员", 2000);
        ILeaf h = new Leaf("h", "财务人员", 2000);
        ILeaf i = new Leaf("i", "财务人员", 2000);
        ILeaf j = new Leaf("i", "CEO秘书", 2000);
        ILeaf laoliu = new Leaf("郑老六", "研发部副总", 20000);
        // ceo下面
        root.addSubordinate(j);
        root.addSubordinate(developDep);
        root.addSubordinate(salesDep);
        root.addSubordinate(financeDep);
        // 研发经理下面
        developDep.addSubordinate(laoliu);
        developDep.addSubordinate(firstDevGroup);
        developDep.addSubordinate(secondDevGroup);
        // 研发小组下面
        firstDevGroup.addSubordinate(a);
        firstDevGroup.addSubordinate(b);
        firstDevGroup.addSubordinate(c);
        secondDevGroup.addSubordinate(d);
        secondDevGroup.addSubordinate(e);
        // 销售部下面的销售人员
        salesDep.addSubordinate(f);
        salesDep.addSubordinate(g);
        // 财务下面的财务人员
        financeDep.addSubordinate(h);
        financeDep.addSubordinate(i);
        return root;
    }
    public static String getTreeInfo(Branch root) {
        ArrayList<ICorp> subordinateList = root.getSubordinate();
        String info = "";
        for (ICorp s : subordinateList) {
            if (s instanceof Leaf) {
                info += s.getInfo() + "\n";
            } else {
                info += s.getInfo() + "\n" + getTreeInfo((Branch) s);
            }
        }
        return info;
    }
}
```

继续对代码进行优化，对getInfo()方法进行抽取：

```java
public abstract class Corp {
    private String name = "";
    private String position = "";
    private int salary = 0;
    public Corp(String name, String position, int salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    public String getInfo() {
        String info = "";
        info = "名称：" + name;
        info += "\t职位：" + position;
        info += "\t薪水：" + salary;
        return info;
    }
}
```

```java
public class Branch extends Corp {
    ArrayList subordinateList = new ArrayList();
    public Branch(String name, String position, int salary) {
        super(name, position, salary);
    }
    public void addSubordinate(ICorp corp) {
        subordinateList.add(corp);
    }
    public ArrayList<ICorp> getSubordinate() {
        return subordinateList;
    }
}  
```

```java
public class Leaf extends Corp {
    public Leaf(String name, String position, int salary) {
        super(name, position, salary);
    }
}
```

## 二、组合模式定义

将对象组合成树形结构以表示“部分-整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性；

组合模式的标准定义：

```java
public abstract class Component {
    public void doSomething() {
        // 业务逻辑
    }
} 
```

```java 
public class Composite extends Component {
    // 构件容器
    private ArrayList<Component> componentList = new ArrayList<Component>();
    // 增加一个叶子构件或者树枝构件
    public void add(Component com) {
        componentList.add(com);
    }
    // 删除一个叶子构件或者树枝构件
    public void remove(Component com) {
        componentList.remove(com);
    }
    public ArrayList<Component> getChildren() {
        return this.componentList;
    }
}  
```

```java
public class Leaf extends Component {
    // 可以覆写父类方法
    public void doSomething() {
        // 业务逻辑
    }
}  
```

```java
public class Client {
    public static void main(String[] args) {
        // 创建根节点
        Composite root = new Composite();
        root.doSomething();
        // 创建树枝构件
        Composite branch = new Composite();
        // 创建叶子构件
        Leaf leaf = new Leaf();
        // 建立整理
        root.add(branch);
        root.add(leaf);
    }
    // 遍历树结构
    public static void display(Composite root) {
        for (Component c : root.getChildren()) {
            if (c instanceof Leaf) {
                c.doSomething();
            } else {
                display((Composite) c);
            }
        }
    }
}  
```

组合模式分为：**安全模式**和**透明模式**。以上讲的都是*安全模式*。

- 安全模式：把树枝节点和叶子节点彻底分开，树枝节点单独拥有用来组合的方法，这种方法比较安全；
- 透明模式：把用来组合使用的方法放到抽象类中，如add()、remove()、以及getChildren()方法，不管叶子对象还是树枝对象都有相同的结构，通过判断getChildren返回值确认是叶子节点还是树枝节点。<br/>
也就是透明模式下，树枝类和树叶类拥有相同的方法，这些方法因为定义在Component抽象父类中。

透明模式代码：

```java
public abstract class Component {
    public void doSomething() {
        // 业务逻辑
    }
    // 增加一个叶子构件或者树枝构件
    public abstract void add(Component component);
    // 删除一个叶子构件或者树枝构件
    public abstract void remove(Component component);
    // 获得分支下的所有叶子构件和树枝构件
    public abstract ArrayList<Component> getChildren();
}  
```

```java
public class Composite extends Component {
    // 构件容器
    private ArrayList<Component> componentList = new ArrayList<Component>();
    // 增加一个叶子构件或者树枝构件
    public void add(Component com) {
        componentList.add(com);
    }
    // 删除一个叶子构件或者树枝构件
    public void remove(Component com) {
        componentList.remove(com);
    }
    public ArrayList<Component> getChildren() {
        return this.componentList;
    }
}
```

```java
public class Leaf extends Component {
    @Deprecated
    public void add(Component component) throws UnsupportedOperationException {
        throw new UnsupportedOperationException();
    }
    @Deprecated
    public ArrayList<Component> getChildren() throws UnsupportedOperationException {
        throw new UnsupportedOperationException();
    }
    @Deprecated
    public void remove(Component component) throws UnsupportedOperationException {
        throw new UnsupportedOperationException();
    }
}
```

```java
public class Client {
    // 递归遍历树
    public static void display(Component root) {
        for (Component c : root.getChildren()) {
            if (c instanceof Leaf) {// 叶子节点
                c.doSomething();
            } else {// 树枝节点
                display(c);
            }
        }
    }
}  
```
组合模式从上到下遍历已经清楚，但是如何从下到上遍历？

增加了父节点属性！！

```java
public abstract class Corp {
    private String name = "";
    private String position = "";
    private int salary = 0;
    private Corp parent;
    public Corp(String name, String position, int salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    public String getInfo() {
        String info = "";
        info = "名称：" + name;
        info += "\t职位：" + position;
        info += "\t薪水：" + salary;
        return info;
    }
    public Corp getParent() {
        return parent;
    }
    public void setParent(Corp parent) {
        this.parent = parent;
    }
}  
```

```java
public class Branch extends Corp {
    ArrayList subordinateList = new ArrayList();
    public Branch(String name, String position, int salary) {
        super(name, position, salary);
    }
    public void addSubordinate(Corp corp) {
        corp.setParent(corp);// 设置父节点
        subordinateList.add(corp);
    }
    public ArrayList<ICorp> getSubordinate() {
        return subordinateList;
    }
}  
```

```java
public class Leaf extends Corp {
    public Leaf(String name, String position, int salary) {
        super(name, position, salary);
    }
}
```

