# 基本概念

DI(Dependency Injection)依赖注入：就是指对象是被动接受依赖类而不是自己主动去找，换句话说就是指对象不是从容器中查找它依赖的类，而是在容器实例化对象的时候主动将它依赖的类注入给它。

# 发生时间

当 Spring IOC 容器完成了 Bean 定义资源的定位、载入和解析注册以后，IOC 容器中已经管理类 Bean 定义的相关数据，但是此时 IOC 容器还没有对所管理的 Bean 进行依赖注入，依赖注入在以下两种情况 发生:

1. 用户第一次调用 getBean()方法时，IOC 容器触发依赖注入。 

2. 当用户在配置文件中将\<bean>元素配置了 lazy-init=false 属性，即让容器在解析注册 Bean 定义时进行预实例化，触发依赖注入。

# getBean方式流程

## 1.入口

getBean()---->doGetBean()----->createBean()----->createBeanInstance()方法，生成 Bean 所包含的 java 对象实例。populateBean()方法，对 Bean 属性的依赖注入进行处理。

如果 Bean 定义的单例模 式(Singleton)，则容器在创建之前先从缓存中查找，以确保整个容器中只存在一个实例对象。如果 Bean 定义的是原型模式(Prototype)，则容器每次都会创建一个新的实例对象。

## 2.选择初始化策略

使用工厂方法和自动装配特性的 Bean 的实例化调用相应的工厂方法或者参数匹配的构造方法即可完成实例化对象的工作。

最常使用的默认无参构造方法就需要使用相应的初始化策略(JDK 的反射机制或者 CGLib)来进行初始化了

## 3.执行实例化

如果 Bean 有方法被覆盖了，则使用 JDK 的反射机制进行实例化，否 则使用 CGLib 进行实例化。

## 4.准备依赖注入

1. 属性值类型不需要强制转换时，不需要解析属性值，直接准备进行依赖注入。 

2. 属性值需要进行类型强制转换时，如对其他对象的引用等，首先需要解析属性值，然后对解析后的属性值进行依赖注入。

## 5.解析属性

解析引用类型的属性值时，首先获取引用的 Bean 名称。如果引用的对象在父类容器中则从父类容器中获取指定的引用对象。

从当前的容器中获取指定的引用 Bean 对象，如果指定的 Bean 没有被实例化，则会递归调用getbean()触发引用 Bean 的初始化和依赖注入

## 6.依赖注入

对容器中完成初始化的 Bean 实例对象进行属性的依赖注入，即把 Bean 对象设置到它所依赖的另一个 Bean 的属性中去。

1. 对于集合类型的属性，将其属性值解析为目标类型的集合后直接赋值给属性。 
2. 对于非集合类型的属性，大量使用了 JDK 的反射机制，通过属性的 getter()方法获取指定属性注入以前的值，同时调用属性的 setter()方法为属性设置注入后的值。

# @Autowired

在bean实例化之后，调用MergedBeanDefinitionPostProcessor 后处理器，Autowire等注解信息就是在这一步完成预解析，并且将注解需要的信息放入缓存。

在依赖注入阶段，IOC 容器根据 Bean 名称或者类型进行自动依赖注入

# 延时加载

当 Bean 定义资源的\<Bean>元素中配置了 lazy-init=false 属性时，容器将会在初始化的时候对所配置 的 Bean 进行预实例化，Bean 的依赖注入在容器初始化的时候就已经完成。

容器在完成 Bean 定义的注册之后，在refresh()方法中调用finishBeanFactoryInitialization(beanFactory)，通过 getBean 方法，触发对指定 Bean 的初始化和依赖注入过程。

# 循环依赖

先从一级缓存singletonObjects中去获取。如果获取到就直接return。

如果获取不到或者对象正在创建中(isSingletonCurrentlyInCreation())，那就再从二级缓存earlySingletonObjects中获取。如果获取到就直接return。

如果还是获取不到，且允许singletonFactories(allowEarlyReference=true)通过getObject()获取。就从三级缓存singletonFactory.getObject()获取。如果获取到了就从singletonFactories中移除，并且放进earlySingletonObjects。其实也就是从三级缓存移动到了二级缓存

加入singletonFactories三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决

### 三级缓存的目的

singletonFactories这个三级缓存，里面都是ObjectFactory。先实例化的bean会通过ObjectFactory半成品提前暴露在三级缓存中。

如果没有AOP的话确实可以两级缓存就可以解决循环依赖的问题，如果加上AOP，两级缓存是无法解决的，不可能每次执行singleFactory.getObject()方法都给我产生一个新的代理对象，所以还要借助另外一个缓存来保存产生的代理对象

