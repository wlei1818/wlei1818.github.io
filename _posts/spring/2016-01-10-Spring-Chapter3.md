---
layout: post
title: 《Spring 3.X企业应用开发实战》IoC容器概述
categories: [Java, Spring]
description: IoC容器概述
keywords: Spring, IoC
autotoc: true
comments: true
---

本文是《Spring 3.X企业应用开发实战》第三章学习笔记

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、Resource接口进行资源的访问

![](images/posts/springspring-chapter3-1.jpg)

**包含的方法：**

```java
Boolean exists();
boolean isOpen();
URL getURL();
fILE getFile();
InputStream getInputStream();
```

```java
public class FileSourceExample {
    public static void main(String[] args) {
        try {
            String filePath = "D:/HTDZWorkspace/SpringDevelop/WebRoot/WEB-INF/classes/conf/file1.txt";
            // 1、使用系统文件路径方式加载文件
            Resource res1 = new FileSystemResource(filePath);
            // 2、用类路径方式加载文件
            Resource res2 = new ClassPathResource("conf/file1.txt");
            InputStream ins1 = res1.getInputStream();
            InputStream ins2 = res2.getInputStream();
            System.out.println("res1=" + res1.getFilename());
            System.out.println("res2=" + res2.getFilename());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}  
```

```java
public class EncodedResourceExample {
    public static void main(String[] args) {
        try {
            Resource res = new ClassPathResource("conf/file1.txt");
            EncodedResource encRes = new EncodedResource(res, "UTF-8");
            String content = FileCopyUtils.copyToString(encRes.getReader());
            System.out.println(content);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}  
```

2、classpath
**classpath**:从根路径加载资源，也可以在jar包或者zip包中；如果存在多个，只会找第一个
**classpath***：会从多个文件中去找；

## 二、资源加载器

![](images/posts/springspring-chapter3-2.jpg)

```java
public class PatternResolverTest {
    public static void main(String[] args) {
        try {
            // ResourceLoader只支持资源类型的前缀表达式不支持ant
            // ResourcePatternResolver支持ant
            ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
            // 加载包下面所有xml结尾的资源
            Resource resources[] = resolver.getResources("classpath*:com/sun/spring/**/*.xml");
            for (Resource res : resources) {
                System.out.println(res.getDescription());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}  
```

## 三、BeanFactory

![](images/posts/springspring-chapter3-3.jpg)

BeanFactory启动IoC容器时，不会初始化配置文件中定义的bean，会在第一次调用时初始化；

## 三、ApplicationContext

![](images/posts/springspring-chapter3-4.jpg)

```java
public class ApplicationContextTest {
    // 放在类路径下时：
    ApplicationContext ctx = new ClassPathXmlApplicationContext("com/sun/spring/beans.xml");
    // 放在文件路径下时
    ApplicationContext ctx2 = new FileSystemXmlApplicationContext("com/sun/spring/beans.xml");
    // 还可以指定一组配置文件
    ApplicationContext ctx3 = new ClassPathXmlApplicationContext(new String[] { "conf/beans.xml", "conf/benas2.xml" });
}  
```

**ApplicationContext** 与 **BeanFactory** 的区别在于： 

ApplicationContext在初始化应用上下文时就实例化所有单实例的bean

## 四、Bean的生命周期

![](images/posts/springspring-chapter3-5.jpg)

## 五、ApplicationContext中Bean的生命周期

![](images/posts/springspring-chapter3-6.jpg)

**ApplicationContext**与 **BeanFactory** 最大的一处不同在于：

- 前者利用Java的反射机制自动识别出配置文件中定义的BeanPostProcessor、InstantiationAwareBeanPostProcessor和BeanFactoryPostProcessor，并自动将它们注册到应用上下文中。
- 而后者需要在代码中通过手工调用addBeanPostProcessor（）方法进行注册
