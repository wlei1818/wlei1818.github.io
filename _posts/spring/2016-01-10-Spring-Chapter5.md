---
layout: post
title: 《Spring 3.X企业应用开发实战》Spring容器高级主题
categories: [Java, Spring]
description: Spring容器高级主题
keywords: Spring
autotoc: true
comments: true
---

本文是《Spring 3.X企业应用开发实战》第五章学习笔记

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、属性编辑

**PropertyEditor接口**

- Object getValue()：返回属性当前值；
- void setValue(Object newValue):设置属性值；
- String getAsText():将属性对象用字符串表示
- void setAsText(String text)：用字符串去更新属性的内部值；

  实现类：PropertyEditorSupport

## 二、Spring的属性编辑器

1、引入外部属性文件：
 
<contex:property-placeholder location="com/sun/spring/a.properties"/>

2、基于注解的引用属性

```java
/**
 * 通过注解引用属性文件值
 * 
 * @Package com.sun.spring.chapter5
 * @Description: TODO(这里用一句话描述这个类的作用)
 * @author Frank
 * @date 2015-9-6 下午3:16:54
 * 
 */
@Component
public class MyDataSource {
    @Value("${driverClassName}")
    private String driverClassName;
    @Value("${url}")
    private String url;
    @Value("${userName}")
    private String userName;
    @Value("${password}")
    private String password;
    public String toString() {
        return ToStringBuilder.reflectionToString(this);
    }
}
```
属性文件加密：

jdbc.driverClassName=oracle.jdbc.driver.OracleDriver<br/>
jdbc.url=jdbc\:oracle\:thin\:@10.251.1.16\:1521\:payment1<br/>
jdbc.username=E8F753256206C7CB<br/>
jdbc.password=DD8E1DB58620BE621538FD7CB52FBB4C<br/>
dbcp.maxActive=50<br/>
dbcp.maxIdle=200 <br/>

```java
/**
 * 属性文件信息加密配置
 * 
 * @Package com.htdz.qfb.oms.common.util
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-26 上午10:36:11
 */
public class EncryptPropertyPlaceholderConfigurer extends PropertyPlaceholderConfigurer {
    // 属性名
    private String[] encryptProNames = { "jdbc.username", "jdbc.password" };
    /**
     * 属性转换：解密
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-26上午10:45:20
     * @param propertyName
     * @param propertyValue
     * @return
     */
    protected String convertProperty(String propertyName, String propertyValue) {
        if (isEncryptProp(propertyName)) {
            try {
                propertyValue = StringEncryptUtil.decrypt(propertyValue);
                System.out.println("decryptValue==>>>" + propertyValue);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return propertyValue;
    }
    /**
     * 判断是否是需要进行解密的属性
     * 
     * @Description
     * @author sunhanbin
     * @date 2015-8-26上午10:41:15
     * @param propertyName
     * @return
     */
    private boolean isEncryptProp(String propertyName) {
        for (String encryptPropName : encryptProNames) {
            if (encryptPropName.equals(propertyName)) {
                return true;
            }
        }
        return false;
    }
}  
```

```xml
<bean class="com.htdz.qfb.oms.common.util.EncryptPropertyPlaceholderConfigurer">
        <property name="ignoreResourceNotFound" value="true" />
        <property name="locations" value="classpath*:/jdbc.properties"/>
    </bean>  
```

3、属性文件自身引用

dbName=samledb;<br/>
url="jdbc:mysql://localhost:3306/${dbName}"<br/>

4、属性配置在bean中，引用bean的值
 
```java
public class SysConfig {
    private int sessionTimeOut;
    private int maxTabPageNum;
    private DataSource dataSource;
    // 模拟从数据库动态获取配置信息
    public void initFormDB() {
        // ....
        this.sessionTimeOut = 30;
        this.maxTabPageNum = 10;
    }
    public int getSessionTimeOut() {
        return sessionTimeOut;
    }
    public void setSessionTimeOut(int sessionTimeOut) {
        this.sessionTimeOut = sessionTimeOut;
    }
    public int getMaxTabPageNum() {
        return maxTabPageNum;
    }
    public void setMaxTabPageNum(int maxTabPageNum) {
        this.maxTabPageNum = maxTabPageNum;
    }
    public DataSource getDataSource() {
        return dataSource;
    }
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

```xml
<bean id="sysConfig" class="com.sun.spring.chapter5.two.SysConfig"
    init-method="initFromDB"
    p:dataSource-ref="dataSource"/>
  
  
  <bean class="com.sun.spring.ApplicationManager"
        p:maxTabPageNum="#{sysConfig.maxTabPageNum}"
        p:sessionTimeOut="#{sessionTimeOut}"/>  
```

基于注解获取：

```java
@Component
public class ApplicationManager {
    @Value("#{sysConfig.sessionTimeOut}")
    private int sessionTimeOut;
    @Value("#{sysConfig.maxTabPageNum}")
    private int maxTabPageNum;
}  
```