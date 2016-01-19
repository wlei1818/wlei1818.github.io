---
layout: post
title: 《Spring 3.X企业应用开发实战》Spring AOP基础
categories: [Java, Spring]
description: Spring AOP基础
keywords: Spring, Aop
autotoc: true
comments: true
---

本文是《Spring 3.X企业应用开发实战》第六章学习笔记

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

## 一、相关术语

- 连接点（JoinPoint）：Spring仅支持方法的连接点，即仅能在方法调用前、后、方法抛出异常时以及方法调用前后这些程序执行织入增强；

- 切点（Pointcut）：AOP通过“切点”定位特定接连点；连接点相当于数据库中的记录，而切点相当于查询条件。切点和连接点不是一对一的关系，一个切点可以匹配多个连接点；

- 增强（Advisor）：织入到目标类连接点上的一段程序代码；

- 目标对象（Target）：目标类，这个类只需实现非横切逻辑的程序逻辑；横切逻辑由AOP动态织入到特定的连接点上；

- 引入（Introduction）：引介是一种特殊的增强，它为类添加一些属性和方法；

- 织入（Weaving）：AOP有三种织入方式：①编译期织入；②类装载期织入；③动态代理织入；

- 代理（Proxy）：一个类被AOP织入增强后，就产生了一个结果类，它是融合了原类和增强逻辑的代理类。根据不同的代理方式，代理类既可能是和原类具有相同接口的类，也可能就是原类的子类；

- 切面（Aspect）

## 二、基础知识

Spring AOP使用了两种动态代理机制：

- 基于JDK的动态代理；

- 基于CGLib的动态代理；因为JDK本身只提供接口的代理，而不支持类的代理；

**JDK动态代理**

JDK动态代理设计到java.lang.reflect包中的 **Proxy** 和 **InvocationHandler**.

**InvocationHandler** 是一个接口，可以通过实现该接口定义横切逻辑，并通过反射机制调用目标类的代码，动态将横切逻辑和业务逻辑编织在一起。

**Proxy**利用 **InvocationHandler** 动态创建一个符合某一接口的实例，生成目标类的代理对象。

使用代理前：

```java
public interface ForumService {
    public void removeiTopic(int topicId);
    public void removeForum(int forumId);
}  
```

```java
public class ForumServiceImpl implements ForumService {
    public void removeForum(int forumId) {
        // 1、开始对该方法进行性能监控
        PerformanceMonitor.begin("com.sun.spring.chapter6.one.ForumServiceImpl.removeForum");
        System.out.println("模拟删除Forum记录：" + forumId);
        try {
            Thread.currentThread().sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public void removeiTopic(int topicId) {
        // 1、开始对该方法进行性能监控
        PerformanceMonitor.begin("com.sun.spring.chapter6.one.ForumServiceImpl.removeiTopic");
        System.out.println("模拟删除Topic记录：" + topicId);
        try {
            Thread.currentThread().sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}  
```

```java
public class MethodPermance {
    private long begin;
    private long end;
    private String serviceMethod;
    public MethodPermance(String serviceMethod) {
        super();
        this.serviceMethod = serviceMethod;
        this.begin = System.currentTimeMillis();
    }
    public void printPerformance() {
        this.end = System.currentTimeMillis();
        long diff = end - begin;
        System.out.println(serviceMethod + "耗时：" + diff + "毫秒");
    }
    public long getBegin() {
        return begin;
    }
    public void setBegin(long begin) {
        this.begin = begin;
    }
    public long getEnd() {
        return end;
    }
    public void setEnd(long end) {
        this.end = end;
    }
    public String getServiceMethod() {
        return serviceMethod;
    }
    public void setServiceMethod(String serviceMethod) {
        this.serviceMethod = serviceMethod;
    }
}  
```

```java
/**
 * 性能监控类：将该类织入到业务类
 * 
 * @Package com.sun.spring.chapter6.one
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-27 上午11:11:25
 */
public class PerformanceMonitor {
    // 保存线程相关的监控信息
    private static ThreadLocal<MethodPermance> performanceRecord = new ThreadLocal<MethodPermance>();
    // 启动对某一目标方法的性能监控
    public static void begin(String method) {
        System.out.println("begin monitor...");
        MethodPermance mp = new MethodPermance(method);
        performanceRecord.set(mp);
    }
    public static void end() {
        System.out.println("end monitor...");
        MethodPermance mp = performanceRecord.get();
        mp.printPerformance();
    }
}  
```

```java
public class Test {
    
     public static void main(String[] args) {
        ForumService fs = new ForumServiceImpl();
        fs.removeForum(10);
        fs.removeiTopic(20);
    }
}  
```

使用代理后：

```java
public class ForumServiceImpl implements ForumService {
    public void removeForum(int forumId) {
        System.out.println("模拟删除Forum记录：" + forumId);
        try {
            Thread.currentThread().sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public void removeiTopic(int topicId) {
        System.out.println("模拟删除Topic记录：" + topicId);
        try {
            Thread.currentThread().sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}  
```

```java
public class PerformanceHandler implements InvocationHandler {
    private Object target;// 目标的业务类
    public PerformanceHandler(Object target) {
        super();
        this.target = target;
    }
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-27上午11:26:03
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        PerformanceMonitor.begin(target.getClass().getName() + "." + method.getName());
        Object obj = method.invoke(target, args);// 通过反射方法调用业务类的目标方法
        PerformanceMonitor.end();
        return obj;
    }
}  
```

```java
public class Test {
    public static void main(String[] args) {
        // 1、希望被代理的目标业务类
        ForumService target = new ForumServiceImpl();
        // 2、将目标业务类和横切代码编织到一起
        PerformanceHandler handler = new PerformanceHandler(target);
        // 3、创建代理实例
        ForumService proxy = (ForumService) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), handler);
        // 4、调用代理实例
        proxy.removeForum(10);
        proxy.removeiTopic(20);
    }
}  
```

**CGLib动态代理**

解决了JDK代理只能代理接口的问题。CGLib采用非常底层的字节码技术，可以为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，并顺势织入横切逻辑。

```java
/**
 * 对于没有通过接口定义业务方法的类，无法使用Proxy方式，可以通过cglib的动态代理实现
 * 
 * @Package com.sun.spring.chapter6.three
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-28 上午09:47:46
 */
public class CGLibProxy implements MethodInterceptor {
    private Enhancer enhancer = new Enhancer();
    public Object getProxy(Class clazz) {
        enhancer.setSuperclass(clazz);//设置需要创建的子类
        enhancer.setCallback(this);
        return enhancer.create();//通过字节码技术创建子类实例
    }
    /**
     * @Description 拦截父类所有方法的调用
     * @author sunhanbin
     * @date 2015-8-28上午09:52:52
     * @param arg0
     * @param arg1
     * @param arg2
     * @param arg3
     * @return
     * @throws Throwable
     */
    public Object intercept(Object target, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        PerformanceMonitor.begin(target.getClass().getName() + "." + method.getName());
        Object obj = proxy.invokeSuper(target, args);// 通过代理类调用父类中的方法
        PerformanceMonitor.end();
        return obj;
    }
}
```

**Spring AOP**

通过Pointcut指定在哪些类的哪些方法上织入横切逻辑，通过Advice（增强）描述横切逻辑和方法的具体织入点（方法前、方法后、方法的两端等）。
   
CGLib所创建的动态代理对象的性能比JDK创建的性能高不少，但CGLib在创建对象时所花费的时间却比JDK代理多；

增强类型：

![](/images/posts/spring/spring-chapter6-1.jpg)

- 前置增强：BeforeAdvice.表示在目标方法执行前实施增强；
- 后置增强：AfterReturningAdvice：目标方法执行后实施增强；
- 环绕增强：MethodInterceptor,目标方法执行前后实施增强；
- 引介增强：IntroductionInterceptor，在目标方法中添加一些新的方法和属性；
- 异常抛出增强：ThrowsAdvice,目标方法抛出异常后实施增强

**前置增强**

```java
public interface Waiter {
    void greetTo(String name);
    void serveTo(String name);
}  
```

```java
public class NaviWaiter implements Waiter {
    public void greetTo(String name) {
        System.out.println(name + "，欢迎光临！");
    }
    public void serveTo(String name) {
        System.out.println(name + "有什么需要服务的吗？");
    }
}
```

```java
/**
 * 前置通知，在方法执行前调用
 * 
 * @Package com.sun.spring.chapter6.four.beforeAdvice
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-28 上午10:17:56
 */
public class GreetingBeforeAdvice implements MethodBeforeAdvice {
    public void before(Method arg0, Object[] arg1, Object arg2) throws Throwable {
        String clientName = (String) arg1[0];
        System.out.println("早上好，" + clientName);
    }
}  
```

```java
public class Test {
    public static void main(String[] args) {
        // 1、代理目标
        Waiter waiter = new NaviWaiter();
        // 前置通知
        BeforeAdvice advice = new GreetingBeforeAdvice();
        // 2、Spring提供的代理工厂
        ProxyFactory pf = new ProxyFactory();
        // 3、设置代理目标
        pf.setTarget(waiter);
        pf.addAdvice(advice);
        // 4、生成代理实例
        Waiter proxy = (Waiter) pf.getProxy();
        proxy.greetTo("大徐");
        proxy.serveTo("小孙");
    }
}  
```

```xml
<!--设置前置通知  -->
 <bean id="greetingAdvice" class="com.sun.spring.chapter6.four.beforeAdvice.GreetingBeforeAdvice"/>
 <bean id="target" class="com.sun.spring.chapter6.four.beforeAdvice.NaviWaiter"/>
 <bean id="waiter" class="org.springframework.aop.framework.ProxyFactoryBean"
       p:proxyInterfaces="com.sun.spring.chapter6.four.beforeAdvice.Waiter"
       p:interceptorNames="greetingAdvice"
       p:target-ref="target"
 />  
```

```java
public class TestAdviceXml {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("com/sun/spring/chapter6/four/beforeAdvice/before-advice.xml");
        Waiter waiter = (Waiter) ctx.getBean("waiter");
        waiter.greetTo("Tom");
    }
}  
```

**后置增强**

```java
public class GreetingAfterAdvice implements AfterReturningAdvice {
    // 在目标类方法调用后执行
    public void afterReturning(Object arg0, Method arg1, Object[] arg2, Object arg3) throws Throwable {
        System.out.println("尽情享用!");
    }
}  
```

```xml
<!--设置前置通知  -->
 <bean id="greetingAdvice" class="com.sun.spring.chapter6.four.beforeAdvice.GreetingBeforeAdvice"/>
 <!--设置后置通知  -->
 <bean id="greetingAfter" class="com.sun.spring.chapter6.four.afterAdvice.GreetingAfterAdvice"/>
 <bean id="target" class="com.sun.spring.chapter6.four.beforeAdvice.NaviWaiter"/>
 <bean id="waiter" class="org.springframework.aop.framework.ProxyFactoryBean"
       p:proxyInterfaces="com.sun.spring.chapter6.four.beforeAdvice.Waiter"
       p:interceptorNames="greetingBefore,greetingAdvice"
       p:target-ref="target"
 />  
```

**环绕增强**

```java
public class GreetingInterceptor implements MethodInterceptor {
    /**
     * @Description
     * @author sunhanbin
     * @date 2015-8-28下午03:01:01
     * @param arg0
     * @return
     * @throws Throwable
     */
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Object[] args = invocation.getArguments();// 目标方法入参
        String clientName = (String) args[0];
        // 目标方法执行前调用
        System.out.println("开始了，" + clientName);
        // 反射机制执行目标方法
        Object obj = invocation.proceed();
        // 目标方法执行后调用
        System.out.println("结束了，" + clientName);
        return obj;
    }
}  
```

```xml
<!--设置环绕通知  -->
 <bean id="greetingAround" class="com.sun.spring.chapter6.four.interceptor.GreetingInterceptor"/>
 <bean id="target" class="com.sun.spring.chapter6.four.beforeAdvice.NaviWaiter"/>
 <bean id="waiter" class="org.springframework.aop.framework.ProxyFactoryBean"
       p:proxyInterfaces="com.sun.spring.chapter6.four.beforeAdvice.Waiter"
       p:interceptorNames="greetingAround"
       p:target-ref="target"
 />  
 ```

**异常抛出增强**

ThrowsAdvice只是一个标识接口，没有定义任何方法。必须采用以下的签名形式定义异常抛出的增强方法：

```java
void afterThrowing([Method method,Object[]args,Object target],Throwable);
```

- 方法名必须为afterThrowing；

- [Method method,Object[]args,Object target]为可选参数，要不全部提供要不全部不提供；

```java
/**
 * 异常增强:ThrowsAdvice未定义任何方法，只是一个标识接口
 * 
 * @Package com.sun.spring.chapter6.four.throwAdvice
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-28 下午04:39:17
 */
public class TransactionManager implements ThrowsAdvice {
    public void afterThrowing(Method method, Object[] args, Object target, Exception e) throws Throwable {
        System.out.println("=====");
        System.out.println("method:" + method.getName());
        System.out.println("抛出异常：" + e.getMessage());
        System.out.println("成功回滚事务");
    }
}  
```

**引介增强**

引介增强不是在目标方法周围织入增强，而是为目标类创建新的方法和属性，所以引介增强的连接点是类级别的，而非方法级别。通过引介增强，可以为目标类添加一个接口的实现。即原来目标类未实现某个接口，通过引介增强可以为目标类创建实现某接口的代理。

```java
/**
 * 可以手工设置是否打开性能监视的切面程序
 * 
 * @Package com.sun.spring.chapter6.four.introductionAdvice
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-28 下午04:52:49
 */
public interface Monitorable {
    void setMonitorActive(boolean isActive);
}
```

```java  
/**
 * 引介增强：可以为目标类创建新的方法或者属性
 * 选择性的织入
 * 
 * @Package com.sun.spring.chapter6.four.introductionAdvice
 * @Description: TODO
 * @author sunhanbin
 * @date 2015-8-28 下午04:54:03
 */
public class ControllablePerformanceMonitor extends DelegatingIntroductionInterceptor implements Monitorable {
    private ThreadLocal<Boolean> MonitorStatusMap = new ThreadLocal<Boolean>();
    public void setMonitorActive(boolean isActive) {
        MonitorStatusMap.set(isActive);
    }
    // 拦截方法
    public Object invoke(MethodInvocation inv) throws Throwable {
        Object obj = null;
        // 对于设置了性能可控的方法开启性能监控
        if (MonitorStatusMap.get() != null && MonitorStatusMap.get()) {
            PerformanceMonitor.begin(inv.getClass().getName() + "." + inv.getMethod().getName());
            obj = super.invoke(inv);
            PerformanceMonitor.end();
        } else {
            obj = super.invoke(inv);
        }
        return obj;
    }
}
```

```xml
<!--设置引介通知  -->
 <bean id="pmonitor" class="com.sun.spring.chapter6.four.introductionAdvice.ControllablePerformanceMonitor"/>
 <bean id="target" class="com.sun.spring.chapter6.one.ForumService"/>
 <bean id="forumService" class="org.springframework.aop.framework.ProxyFactoryBean"
       p:proxyTargetClass="true"
       p:interceptorNames="pmonitor"
       p:target-ref="target"
       p:interfaces="com.sun.spring.chapter6.four.introductionAdvice.Monitorable"
 />  
 ```

## 三、创建切面

增强提供了连接点方位信息，而切点进一步描述织入到哪些类的哪些方法上。
   
Spring通过org.springframework.aop.Pointcut接口描述切点，Pointcut由ClassFilter和MethodMatcher构成，它通过ClassFilter定位到某些特定类上，通过MethodMatcher定位到某些特定方法上。
   
Spring支持两种方法匹配器：**静态方法匹配器**、**动态方法匹配器**。

- 静态方法匹配器：它仅对方法名签名（包括方法名和入参类型及顺序）进行匹配；静态匹配仅判别一次

- 动态方法匹配器：在运行期间检查方法入参的值。动态匹配因为每次调用方法的入参都可能不一样，所以每次调用方法都必须判断；因此动态匹配对性能的影响很大；

##四、切点类型

- 静态方法切点：org.springframework.aop.support.StaticMethodMatcherPointcut。默认匹配所有类.

其中两个主要的子类有：NameMatchMethodPointcut和AbstractRegexpMethodPointcut

- 动态方法切点：org.springframework.aop.support.DynamicMethodMatcherPointcut

- 注解切点：org.springframework.aop.support.annotation.AnnotationMatchingPointcut

- 表达式切点：org.springframework.aop.support.annotation.ExpressionPointcut.支持AspectJ切点表达式语法而定义的接口

- 流程切点：org.springframework.aop.support.annotation.ControlFlowPointcut

- 复合切点：org.springframework.aop.support.annotation.ComposablePointcut

**切面类型**

切点结合增强，即可创建切面

- 一般切面：Advisor.范围太广一般不直接使用；

- **PointcutAdvisor**：代表具有切点的切面，包含Advice和Pointcut

- **IntroductionAdvisor**：引介切面。使用ClassFilter进行定义

![](/images/posts/spring/spring-chapter6-2.jpg)

**PointcutAdvisor具体的实现类：**

- DefaultPointcutAdvisor：不支持引介切面；
- NameMatchMethodPointcutAdvisor：按方法名定义切点的切面
- RegexpMethodPointcutAdvisor：按正则表达式匹配方法名进行切点定义的切面；
- StaticMethodMatcherPointcutAdvisor：静态方法匹配器切点定义的切面；
- AspectJExpressionPointcutAdvisor：用于AspectJ切点定义表达式定义切点的切面；
- AspectJPointcutAdvisor：用于AspectJ语法定义切点

**静态普通方法名匹配切面**

```java
/**
 * 静态普通方法名匹配切面：StaticMethodMatherPintcutAdvisor
 * 
 * @ClassName: GreetingAdvisor
 * @Description: TODO(这里用一句话描述这个类的作用)
 * @author Frank
 * @date 2015-9-2 下午2:41:54
 * 
 */
public class GreetingAdvisor extends StaticMethodMatcherPointcutAdvisor {
    /**
     * 切点方法匹配规则
     * 
     * @Description: TODO
     * @author Frank
     * @date 2015-9-2 下午2:44:16
     * @param @param arg0
     * @param @param arg1
     * @param @return
     * @throws
     */
    public boolean matches(Method method, Class<?> clazz) {
        return "greetTo".equals(method.getName());
    }
    // 切点类的匹配规则：Waiter类或者子类
    // 因为默认去匹配所有的类
    public ClassFilter getClassFilter() {
        return new ClassFilter() {
            public boolean matches(Class<?> clazz) {
                return Waiter.class.isAssignableFrom(clazz);
            }
        };
    }
}  
```

```java
/**
 * 前置通知，在方法执行前调用
 * 
 * @Package com.sun.spring.chapter6.four.beforeAdvice
 * @Description: 这里用来配合GreetingAdvisor切点类来进行advisor的织入
 * @author sunhanbin
 * @date 2015-8-28 上午10:17:56
 */
public class GreetingBeforeAdvice implements MethodBeforeAdvice {
    public void before(Method arg0, Object[] arg1, Object arg2) throws Throwable {
        String clientName = (String) arg1[0];
        System.out.println("早上好，" + clientName);
    }
}
```

```xml
<bean id="waiterTarget" class="com.sun.spring.chapter6.four.beforeAdvice.Waiter"/>
 <!--设置前置通知  -->
 <bean id="greetingAdvice" class="com.sun.spring.chapter6.five.staticPointcutAdvisor.GreetingBeforeAdvice"/>
 <!--向切面注入一个前置增强  -->
 <bean id="greetingAdvisor" class="com.sun.spring.chapter6.five.staticPointcutAdvisor.GreetingAdvisor"
       p:advice-ref="greetingAdvice"/>
 <!-- 通过父bean定义公共的配置信息 -->
 <bean id="parent" abstract="true" class="org.springframework.aop.framework.ProxyFactory"
       p:interceptorNames="greetingAdvisor"
       p:proxyTargetClass="true"/>
<!-- 设置代理，把两者的父信息提取到公共 -->
<bean id="waiter" parent="parent" p:target-ref="waiterTarget"/>
<bean id="seller" parent="parent" p:target-ref="sellerTarget"/> 
```

**静态正则表达式方法切面**

```xml 
<bean id="waiterTarget" class="com.sun.spring.chapter6.four.beforeAdvice.Waiter"/>
 <!--设置前置通知  -->
 <bean id="greetingAdvice" class="com.sun.spring.chapter6.five.staticPointcutAdvisor.GreetingBeforeAdvice"/>
 <!-- 通过正则定义一个切面  -->
 <bean id="regexpAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor"
       p:advice-ref="greetingAdvice">
       <property name="patterns">
          <list><value>.*greet.*</value></list><!-- 注意这里匹配的是类方法的全限定名，即带类名的方法名 -->
       </property>
  </bean>
 <bean id="waiter1" abstract="true" class="org.springframework.aop.framework.ProxyFactory"
       p:interceptorNames="regexpAdvisor"
       p:proxyTargetClass="true"
       p:target-ref="waiterTarget"/>  
```

![](/images/posts/spring/spring-chapter6-3.jpg)


![](/images/posts/spring/spring-chapter6-3.jpg)


**动态切面**

```java
/**
 * 动态切面
 * 
 * @Package com.sun.spring.chapter6.five.dynamicPointcutAdvisor
 * @Description: TODO(这里用一句话描述这个类的作用)
 * @author Frank
 * @date 2015-9-2 下午3:25:15
 * 
 */
public class GreetingDynamicPointcut extends DynamicMethodMatcherPointcut {
    private static List<String> specialClientList = new ArrayList<String>();
    static {
        specialClientList.add("John");
        specialClientList.add("Tom");
    }
    // 对类进行静态切点检查:Waiter类或者子类
    public ClassFilter getClassFilter() {
        return new ClassFilter() {
            public boolean matches(Class<?> clazz) {
                System.out.println("调用getClassFilter()对" + clazz.getName() + "做静态检查");
                return Waiter.class.isAssignableFrom(clazz);
            }
        };
    }
    /**
     * 静态检查
     * 由于动态检查性能开销很大，因此在最好配合静态检查，排除一大部分非法内容
     * @author Frank
     * @date 2015-9-2 下午3:29:09
     * @param method
     * @param targetClass
     * @param args
     * @return
     */
    public boolean matches(Method method, Class<?> targetClass) {
        System.out.println("调用matches(method,clazz)" + targetClass.getName() + "." + method.getName() + "做静态检查");
        return "greetTo".equals(method.getName());
    }
    /**
     * 动态检查 
     * 
     * @author Frank
     * @date 2015-9-2 下午3:30:33
     * @param method
     * @param targetClass
     * @param args
     * @return
     */
    public boolean matches(Method method, Class<?> targetClass, Object[] args) {
        System.out.println("调用matches(method,clazz)" + targetClass.getName() + "." + method.getName() + "做动态检查");
        String clientName = (String) args[0];
        return specialClientList.contains(clientName);
    }
}  
```

由于动态切点检查会对性能造成很大的影响，尽量避免在运行时每次都对目标类的各个方法进行动态检查。

Spring采用这样的机制：*在创建代理时对目标类的每个连接点使用静态切点检查，如果仅通过静态切点检查就可以知道连接点是不匹配的，则在运行时就不再进行动态检查了；如果静态切点检查是匹配的，在运行时才进行动态切点检查；*
   
所以在定义动态切点时，切勿忘记同时覆盖 *getClassFilter()* 和 *matches(Method method,Class clazz)*,通过静态切点检查排除大部分方法；