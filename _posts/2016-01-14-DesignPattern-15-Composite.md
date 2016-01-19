---
layout: post
title: 《研磨设计模式》组合模式-Composite
categories: [设计模式]
description: 设计模式|组合模式
keywords: 设计模式,组合,Composite
autotoc: true
comments: true
---

本文是《研磨设计模式》第十五章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。


## 案例场景

商品类别数：叶子节点、根节点

```java
/**
 * 叶子对象
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-6下午8:56:06
 */
public class Leaf {
    private String name = "";
    public Leaf(String name) {
        this.name = name;
    }
    /**
     * 输出叶子对象的结构，叶子对象没有子对象，也就是输出叶子对象的名字
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午8:58:01
     * @param preStr
     */
    public void printStruct(String preStr) {
        System.out.println(preStr + "-" + name);
    }
}
```

```java  
/**
 * 组合对象，可以包含其他组合对象或者叶子对象
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-6下午9:18:37
 */
public class Composite {
    private Collection<Composite> childComposite = new ArrayList<Composite>();
    private Collection<Leaf> childLeaf = new ArrayList<Leaf>();
    private String name = "";
    public Composite(String name) {
        this.name = name;
    }
    /**
     * 向组合对象加入被它包含的其他组合对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午9:02:30
     * @param c
     */
    public void addComposite(Composite c) {
        childComposite.add(c);
    }
    /**
     * 向组合对象加入被它包含的叶子对象
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午9:03:22
     * @param leaf
     */
    public void addLeaf(Leaf leaf) {
        childLeaf.add(leaf);
    }
    /**
     * 输出组合对象自身的结构
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午9:04:34
     * @param preStr
     */
    public void printStruct(String preStr) {
        System.out.println(preStr + "-" + name);
        //
        preStr += "";
        for (Leaf leaf : childLeaf) {
            leaf.printStruct(preStr);
        }
        // 输出当前对象的子对象
        for (Composite c : childComposite) {
            c.printStruct(preStr);
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        // 定义所有的组合对象
        Composite root = new Composite("服装");
        Composite c1 = new Composite("男装");
        Composite c2 = new Composite("女装");
        // 定义所有的叶子对象
        Leaf leaf1 = new Leaf("衬衣");
        Leaf leaf2 = new Leaf("夹克");
        Leaf leaf3 = new Leaf("裙子");
        Leaf leaf4 = new Leaf("套装");
        // 按照树的结构来组合对象和叶子对象
        root.addComposite(c1);
        root.addComposite(c2);
        c1.addLeaf(leaf1);
        c1.addLeaf(leaf2);
        c2.addLeaf(leaf3);
        c2.addLeaf(leaf4);
        // 调用根对象的输出功能来输出整棵树
        root.printStruct("");
    }
}
```

## 组合模式的定义

- 组合模式的定义：将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

- 组合模式通过引入一个抽象的组件对象，作为组合对象和叶子对象的父对象，这样就把叶子对象和组合对象统一起来了。用户使用的时候，始终是在操作组件对象，而不再去区分是在操作组合对象还是叶子对象；

- Component：抽象的组件对象，为组合中的对象声明接口，让客户通过这个接口来访问和管理整个对象结构。

- Leaf：叶子节点对象。

- Composite：组合对象，通常会存储子组件。







