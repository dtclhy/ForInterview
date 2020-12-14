# 基本概念

IOC(Inversion of Control)控制反转。所谓控制反转，就是把原先我们代码里面需要实现的对象创建、依赖的代码，反转给容器来帮忙实现。那么必然的我们需要创建一个容器，同时需要一种描述来让容器知道需要创建的对象与对象的关系。这个描述最具体表现就是我们所看到的配置文件。

# 核心类

## BeanFactory

BeanFactory 作为最顶层的一个接口类，它定义了 IOC 容器的基本功能规范。只对 IOC 容器的基本行为作了定义。同时Spring 提供了许多 IOC 容器的实 现 ，ClasspathXmlApplication。

BeanFactory 有三 个重要的子类:ListableBeanFactory、HierarchicalBeanFactory 和 AutowireCapableBeanFactory。主要是为了区分在 Spring 内部在操作过程中对象的传递和转化过程时，对对象的数据访问所做的限制。

## BeanDefinition

SpringIOC 容器管理了我们定义的各种 Bean 对象及其相互的关系，Bean 对象在 Spring 实现中是 以 BeanDefinition 来描述的。

## BeanDefinitionReader

Bean 的解析主要就是对 Spring 配置文件的解析。这个解析过程主要通过 BeanDefintionReader 来完成。

# 基于Xml的IOC容器初始化

IOC 容器的初始化包括定位、加载和注册这三个基本的过程。

1. 定位。定位配置文件和扫描相关的注解。
2. 加载。将配置信息 载入到内存中。
3. 注册。根据载入的信息，将对象初始化到IOC容中。

## 1.获得配置路径

Spring IOC容器在初始化时将配置的 Bean 配置信息定位为 Spring 封装的 Resource。

## 2.开始启动

ClassPathXmlApplicationContext、XmlWebApplicationContext 等都继承自父容器 AbstractApplicationContext。主要用到了装饰器模式和策略模式，最终都是调用同一个refresh()方法。

refresh()是一个模板方法，规定了 IOC 容器的启动流程。

```java
public void refresh() throws BeansException, IllegalStateException {        
  synchronized (this.startupShutdownMonitor) {
    //STEP 1、调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识
    prepareRefresh();
    // STEP 2:
    //a) 创建IoC容器(DefaultListableBeanFactory)
    //b) 加载解析XML文件(最终存储到Document对象中)
    //c) 读取Document对象，并完成BeanDefinition的加载和注册工作
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
    // STEP 3: 为 BeanFactory 配置容器特性，例如类加载器、事件处理器等 
    prepareBeanFactory(beanFactory);
    // STEP 4:为容器的某些子类指定特殊的 BeanPost 事件处理器
    postProcessBeanFactory(beanFactory);
    // STEP 5: 调用BeanFactoryPostProcessor后置处理器对BeanDefinition处理
    //@
    invokeBeanFactoryPostProcessors(beanFactory);
    // STEP 6: 注册BeanPostProcessor后置处理器 
    registerBeanPostProcessors(beanFactory);
    // STEP 7: 初始化一些消息源(比如处理国际化的i18n等消息源) 
    initMessageSource();
    // STEP 8: 初始化容器事件传播器 
    initApplicationEventMulticaster();
    // STEP 9: 初始化一些特殊的bean
    onRefresh();
    // STEP 10: 注册一些监听器
    registerListeners();
    // STEP 11: 实例化剩余的单例bean(非懒加载方式)
    finishBeanFactoryInitialization(beanFactory);
    // STEP 12: 完成刷新时，需要发布对应的事件 
    finishRefresh();
}
```

## 3.创建容器

在这个方法中，先判断 BeanFactory 是否存在，如果存在则先销毁 beans 并关闭 beanFactory，接着创建 DefaultListableBeanFactory，并调用 loadBeanDefinitions(beanFactory)装载 bean 定义。

## 4.载入配置路径

## 5.解析配置文件路径

## 6.读取配置内容

载入 Bean 配置信息，将 Bean 配置信息转换为 Document 对象

## 7.解析配置文件

在完成通用的 XML 解析之后，按照 Spring Bean 的定义规则对 Document 对象进行解析。首先解析\<bean>元素，然后解析\<property>、\<list>等元素。

在解析\<Bean>元素过程中没有创建和实例化 Bean 对象，只是创建了 Bean 对象的定义类 BeanDefinition，将\<Bean>元素中的配置信息设置到 BeanDefinition 中。

## 8.向容器注册

DefaultListableBeanFactory 中使用一个 HashMap 的集合对象存放 IOC 容器中注册解析的 BeanDefinition。

现在 IOC 容器中已经建立了整个 Bean 的配置信息，这些 BeanDefinition 信息已经可以使用，并且可以被检索，IOC 容器的作用就是对这些注册的 Bean 定义信息进行处理和维护。

# 基于 Annotation 的 IOC 初始化

包括定位bean扫描路径、读取元数据、解析、注册这四个步骤

解析主要内容为 Bean 中关于作用域的配置、处理注解 Bean 定义类中通用的注解、创建对于作用域的代理对象

