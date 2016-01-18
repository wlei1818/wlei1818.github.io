---
layout: post
title: 《设计模式之禅》迭代器模式
categories: [设计模式]
description: 设计模式之禅|迭代器模式
keywords: 设计模式
autotoc: true
---

本文是《设计模式之禅》第二十章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、案例场景

将所有项目整理到一起

未使用迭代器模式的写法：

```java
public interface IProject {
    public String getProjectInfo();
} 
```

```java
public class Project implements IProject {
    private String name = "";
    private int num = 0;
    private int cost = 0;
    public Project(String name, int num, int cost) {
        this.name = name;
        this.num = num;
        this.cost = cost;
    }
    @Override
    public String getProjectInfo() {
        String info = "项目名称是：" + name + "。项目人数是：" + num + "项目费用：" + cost;
        return info;
    }
}  
```

```java
public class Boss {
    public static void main(String[] args) {
        ArrayList<IProject> projectList = new ArrayList<IProject>();
        projectList.add(new Project("星球大战项目", 10, 1000));
        projectList.add(new Project("扭转时空项目", 20, 2000));
        projectList.add(new Project("超人改造项目", 30, 3000));
        // 老的10个项目
        for (int i = 4; i < 14; i++) {
            projectList.add(new Project("第" + i + "个项目", i * 10, i * 1000));
        }
        for (IProject p : projectList) {
            System.out.println(p.getProjectInfo());
        }
    }
}  
```

使用迭代器模式写：

```java
public interface IProject {
    //获取项目信息
    public String getProjectInfo();
    // 增加项目
    public void add(String name, int num, int cost);
    // 获得一个可以被遍历的对象
    public IProjectIterator iterator();
}  
```

```java
public class Project implements IProject {
    private String name = "";
    private int num = 0;
    private int cost = 0;
    private ArrayList<IProject> projectList = new ArrayList<IProject>();
    
    public Project(){
        
    }
    public Project(String name, int num, int cost) {
        this.name = name;
        this.num = num;
        this.cost = cost;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午05:05:47
     * @param name
     * @param num
     * @param cost
     */
    @Override
    public void add(String name, int num, int cost) {
        projectList.add(new Project(name, num, cost));
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午05:05:47
     * @return
     */
    @Override
    public String getProjectInfo() {
        String info = "项目名称是：" + name + "。项目人数是：" + num + "项目费用：" + cost;
        return info;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午05:07:05
     * @return
     */
    @Override
    public IProjectIterator iterator() {
        return new ProjectIterator(projectList);
    }
}
```

```java
 public interface IProjectIterator extends Iterator {
}

```

```java
public class ProjectIterator implements IProjectIterator {
    private ArrayList<IProject> projectList = new ArrayList<IProject>();
    private int currentItem = 0;
    public ProjectIterator(ArrayList<IProject> projectList) {
        this.projectList = projectList;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午05:09:57
     * @return
     */
    @Override
    public boolean hasNext() {
        boolean b = true;
        if (currentItem >= projectList.size() || projectList.get(currentItem) == null)
            b = false;
        return b;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午05:09:57
     * @return
     */
    @Override
    public Object next() {
        return projectList.get(currentItem + 1);
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-6下午05:09:57
     */
    @Override
    public void remove() {
        // 暂时没用到
    }
}  
```

```java
public class Boss {
    public static void main(String[] args) {
        IProject project = new Project();
        project.add("星球大战项目", 10, 1000);
        project.add("扭转时空项目", 20, 2000);
        project.add("超人改造项目", 30, 3000);
        // 老的10个项目
        for (int i = 4; i < 14; i++) {
            project.add("第" + i + "个项目", i * 10, i * 1000);
        }
        // 遍历取数据
        IProjectIterator projectIterator = project.iterator();
        while (projectIterator.hasNext()) {
            IProject p = (IProject) projectIterator.next();
            System.out.println(p.getProjectInfo());
        }
    }
}  

```

迭代器模式目前已经是没落的模式，因此也就不深究；