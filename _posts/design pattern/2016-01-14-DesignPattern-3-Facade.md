---
layout: post
title: 《研磨设计模式》外观模式-Facade
categories: [设计模式]
description: 设计模式|外观模式
keywords: 设计模式,外观
autotoc: true
comments: true
---

本文是《研磨设计模式》第三章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##使用场景

    代码生成工具，需要生成表现层、逻辑层、数据层、配置层的代码

## 代码实战：

 1. 不用模式的操作方式

 ```java
public class ConfigModel {
    private boolean needGenPresentation = true;// 是否生成表现层数据
    private boolean needGenBusiness = true;// 是否生成逻辑层
    private boolean neddGenDAO = true;// 是否需要生成DAO
    public boolean isNeedGenPresentation() {
        return needGenPresentation;
    }
    public void setNeedGenPresentation(boolean needGenPresentation) {
        this.needGenPresentation = needGenPresentation;
    }
    public boolean isNeedGenBusiness() {
        return needGenBusiness;
    }
    public void setNeedGenBusiness(boolean needGenBusiness) {
        this.needGenBusiness = needGenBusiness;
    }
    public boolean isNeddGenDAO() {
        return neddGenDAO;
    }
    public void setNeddGenDAO(boolean neddGenDAO) {
        this.neddGenDAO = neddGenDAO;
    }
}
public class ConfigManager {
    private static ConfigManager manager = null;
    private static ConfigModel cm = null;
    public static ConfigManager getInstance() {
        if (manager == null) {
            manager = new ConfigManager();
            cm = new ConfigModel();
            // 读取配置文件，把值设置到ConfigModel中，此处省略
        }
        return manager;
    }
    public ConfigModel getConfigModel() {
        return cm;
    }
}  
/**
 * 示意生成表现层的模块
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-7-23下午08:47:46
 */
public class Presentation {
    public void generate() {
        ConfigModel cm = ConfigManager.getInstance().getConfigModel();
        if (cm.isNeedGenPresentation())
            System.out.println("正在生成表现层的代码！！！");
    }
}
其他业务层、数据层的代码生成类似
public class Client {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-7-23下午08:51:36
     * @param args
     */
    public static void main(String[] args) {
        new Presentation().generate();
    }
}  
```

##外观模式的定义
为子系统中的一组接口提供一个一致的界面，Facade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。