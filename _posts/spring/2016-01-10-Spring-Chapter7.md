---
layout: post
title: 《Spring 3.X企业应用开发实战》基于@AspectJ和Schema的AOP
categories: [Java, Spring]
description: 基于@AspectJ和Schema的AOP
keywords: Spring,AspectJ,Schema
autotoc: true
comments: true
---

本文是《Spring 3.X企业应用开发实战》第七章学习笔记

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、注解类

```java
//声明注解的保留期限
@Retention(RetentionPolicy.RUNTIME)
// 声明可以使用该注解的目标类型
@Target(ElementType.METHOD)
// 定义注解
public @interface NeedTest {
    // 声明注解成员
    boolean value() default true;
}  
```

**使用@interface定义注解类**

- 成员方法无法抛出异常、无法带入参；
- 可以通过default为成员指定一个默认值；
- 成员类型是受限的：原始类型及其封装类、String、 Class、 enums ，以及它们的数组类型；
- 如果注解只有一个成员，则成员必须取名为value（）.
- 如果对多个参数赋值必须使用“=”，如value(a="1',b="2")；数组类型赋值为{a,b,c }；

##二、@AspectJ切面

```java
public class NaviWaiter implements Waiter {
    public void greetTo(String name) {
        System.out.println(name + "，欢迎光临！");
    }
    public void serveTo(String name) {
        System.out.println(name + "有什么需要服务的吗？");
    }
}  
/**
 * 基于@AepectJ的切面
 * 
 * @Package com.guozha.oms.web.controller.user
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-9-3下午4:44:07
 */
// 注解标记为切面
@Aspect
public class PreGreetingAspect {
    // 定义切点和增强逻辑
    @Before("execution(* greetTo(..))")
    public void beforeGreeting() {// 增强的横切逻辑
        System.out.println("How are you!");
    }
}  
public class AspectJProxyTest {
    public static void main(String[] args) {
        Waiter target = new NaviWaiter();
        AspectJProxyFactory factory = new AspectJProxyFactory();
        // 设置目标对象
        factory.setTarget(target);
        // 添加切面类
        factory.addAspect(PreGreetingAspect.class);
        // 生成织入切面的代理对象
        Waiter proxy = factory.getProxy();
        proxy.greetTo("John");
        proxy.greetTo("Tom");
    }
}  
```

如果使用配置的话，就使用以下的配置：

```xml
<!-- 如果用这个 就不需要③ -->
 <aop:aspectj-autoproxy/>
 <!--①目标bean  -->
 <bean id="waiter" class="com.sun.spring.chapter6.four.beforeAdvice.Waiter"/>
 <!--②使用AspectJ注解的切面类  -->
 <bean   class="com.sun.spring.chapter7.two.PreGreetingAspect"/>
 <!--③自动创建代理器，自动将AspectJ注解切面类织入到目标Bean中  -->
 <bean   class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator"/>  
 ```

## 三、切点函数

![](/images/posts/spring/spring-chapter7-1.jpg)

![](/images/posts/spring/spring-chapter7-2.jpg)

**AspectJ通配符**

*：匹配任意字符，但只能匹配上下文中的一个元素；

..：匹配任意字符，可匹配上下文中多个元素。在表示类时，必须和*联合使用，表示入参时单独使用；

+：按类型匹配制定类的所有类，必须跟在类名后面；

**增强类型**

- @Before
- @AfterReturning
- @Around
- @AfterThrowing
- @After：不管是抛出异常或者是正常退出，都会执行增强；一般用于释放资源。
- @DeclareParents：引介增强


## 四、命名切点

```java
/**
 * 通过@Pointcut对切点进行命名，从而可以在其他切面类中共同使用这些切点
 * 
 * @Package com.sun.spring.chapter7.three
 * @Description: TODO(这里用一句话描述这个类的作用)
 * @author Frank
 * @date 2015-9-6 上午9:44:22
 * 
 */
public class TestNamePointcut {
    @Pointcut("within(com.sun.spring.*)")
    private void inPackage() {
    };
    @Pointcut("execution(* greetTo(..))")
    protected void greetTo() {
    };
    @Pointcut("inPackage() and greetTo()")
    public void inPkgGreetTo() {
    };
}  
```

```java
@Aspect
public class TestAspect {
    
    @Before("TestNamePointcut.inPkgGreetTo()")
    public void pkgGreetTo(){
        System.out.println("===pkgGreetTo executed!!");
    }
    
    @Before("!target(com.sun.spring.chapter6.four.beforeAdvice.NaviWaiter) && TestNamePointcut.inPkgGreetTo()")
    public void pkgGreetToNotNaviWaiter(){
        System.out.println("==pkgGreetToNotNaviWaiter executed!!==");
    }
}  
```

## 五、访问连接点信息

1、**ProceedingJoinPoint**

- Object proceed()：通过反射执行目标对象的连接点处的方法；
- Object proceed(Object[] args)：同上，但使用新的入参替换原来的入参；

```java
@Aspect
public class TestAspect {
    @Around("execution(* greetTo(..)) && target(com.sun.spring.chapter7.two.NaviWaiter)")
    public void joinPointAccess(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("===joinPointAccess===");
        // 访问连接点信息
        System.out.println("args[0]=" + pjp.getArgs()[0]);
        System.out.println("signature=" + pjp.getTarget().getClass());
        // 通过连接点执行目标对象的方法
        pjp.proceed();
        System.out.println("===joinPointAccess===");
    }
}  
```

## 六、Schema配置切面

```java
public class AdviceMethods {
    public void preGreeting() {
        System.out.println("How are you ");
    }
}  
```

```xml
<!-- 使用cglib动态代理。可定义多个config，可使用不同的代理 -->
   <aop:config proxy-target-class="true">
      <aop:aspect ref="adviceMethod">
        <aop:before pointcut="target(com.sun.spring.chapter7.two.NaviWaiter) and execution(* greetTo(..))" method="preGreeting"/>
      </aop:aspect>
   </aop:config>
   
   <bean id="adviceMethod" class="com.sun.spring.chapter7.five.AdviceMethods"/>
   <bean id="navieWaiter" class="com.sun.spring.chapter7.two.NaviWaiter"/>

```

其余增强类型详见P253

## 七、切面总结

![](/images/posts/spring/spring-chapter7-3.jpg)


