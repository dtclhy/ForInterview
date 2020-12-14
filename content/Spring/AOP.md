# 基本概念

面向切面编程。可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术。

**切面(Aspect)：**切面是通知和切点的结合

**连接点(Joinpoint)：**程序执行过程中的某一行为，例如，MemberService .get 的调用

**通知(Advice)：**切面对于某个连接点所产生的动作

**切入点(Pointcut)：**匹配连接点的断言，在 AOP 中通知和一个切入点表达式关联

**AOP 代理：**TargetObject 实现了接口时采用 JDK 动态代理，反之，采用 CGLib 代理

# AOP生成

## 1.入口

Spring 的 AOP 是通过接入 BeanPostProcessor 后置处理器开始的。 后置处理器是一个监听器，可以监听容器触发的 Bean 声明周期事件。后置处 理器向容器注册以后，容器中管理的 Bean 就具备了接收 IOC 容器事件回调的能力。

```java
public interface BeanPostProcessor {
  //为在 Bean 的初始化前提供回调入口
  @Nullable
  default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {     
    return bean;  
  }
  //为在 Bean 的初始化之后提供回调入口
  @Nullable
  default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
     return bean;
   }
}
```

BeanPostProcessor 后置处理器的调用发生在 Spring IOC 容器完成对 Bean 实例对象的创建和属性的依赖注入完成之后。

createBeanInstance()实例化----->populateBean()依赖注入---->initializeBean()初始化

 initializeBean()实现了初始化前回调、调用 Bean 实例对象初始化方法(可以用init-Method指定)、初始化后回调。

AOP代理对象在初始化后回调中生成。

## 2.选择代理策略

最终调用的是 proxyFactory.getProxy()，proxyFactory 有 JDK 和 CGLib。

# AOP调用

 InvocationHandler 是 JDK 动态代理的核心，生成的代理对象的方法调用都会委托到 InvocationHandler.invoke()方法。

主要实现思路可以简述为：首先获取应用到此方法上的通知链(Interceptor Chain)。如果有通知，则应用通知，并执行 JoinPoint。如果没有通知，则直接反射执行 JoinPoint

## 通知链

从提供的配置中获取 advisor列表，遍历处理这些 advisor。判断此 Advisor能否应用到目标类上。如果是 PointcutAdvisor，则判断此 Advisor能否应用到目标方法 Method上。将满足条件的 Advisor 通过 AdvisorAdaptor 转化成 *Interceptor* 列表返回，同时会被缓存起来。

如果得到的拦截器链为空，则直接反射调用目标方法，否则创建 MethodInvocation，递归调用其 proceed()方法，触发拦截器链的执行。