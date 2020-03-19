# Spring 面试题

### 什么是 Spring Bean？

Spring beans 就是被 Spring 容器所管理的 Java 对象。

### 什么是 Spring 容器？

Spring 容器负责实例化，配置和装配 Spring beans。

`org.springframework.context.ApplicationContext`表示Spring IoC容器，负责实例化，配置和组装上述 bean。容器通过读取配置元数据获取有关要实例化，配置和组装的对象的指令。配置元数据通常以XML，Java 注解或代码的形式表示。

### BeanFactory 和 ApplicationContext 的不同点

**BeanFactory 接口**

通常情况，BeanFactory 的实现是使用**懒加载**的方式，这意味着 beans 只有在我们通过 getBean() 方法直接调用它们时才进行实例化

**ApplicationContext 接口**

它继承了 BeanFactory 接口，所以 ApplicationContext 包含 BeanFactory 的所有功能以及更多功能！它的主要功能是支持大型的业务应用的创建**特性：**

- Bean instantiation/wiring
- Bean 的实例化/串联
- 自动的 BeanPostProcessor 注册
- 自动的 BeanFactoryPostProcessor 注册
- 方便的 MessageSource 访问（i18n）
- ApplicationEvent 的发布

与 BeanFactory 懒加载的方式不同，它是**预加载**，所以，每一个 bean 都在 ApplicationContext 启动之后实例化

### BeanDefinition 与 BeanFactory  区别？

BeanDefinition 用来描述 bean 实例的信息体。BeanFactory  对 Bean 的一些操作，getBean 获取 bean，或者 containBean 是否存在 bean。

### BeanFactoryPostProcessor 与 BeanPostProcessor 区别？

BeanFactoryPostProcessor是在spring容器加载了bean的定义文件之后**，在bean实例化之前**，可以修改 bean 的相关信息，比如bean的scope从singleton改为prototype。

BeanPostProcessor（拦截器），可以在spring容器实例化bean之后，在执行bean的初始化方法前后，添加一些自己的处理逻辑。这里说的初始化方法，指的是下面两种：
1）bean实现了InitializingBean接口，对应的方法为afterPropertiesSet

2）在bean定义的时候，通过init-method设置的方法

注意：BeanPostProcessor是在spring容器加载了bean的定义文件并且实例化bean之后执行的。BeanPostProcessor的执行顺序是在BeanFactoryPostProcessor之后。

spring中，有内置的一些BeanPostProcessor实现类，例如：

org.springframework.context.annotation.CommonAnnotationBeanPostProcessor：支持@Resource注解的注入
org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor：支持@Required注解的注入
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor：支持@Autowired注解的注入
org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor：支持@PersistenceUnit和@PersistenceContext注解的注入
org.springframework.context.support.ApplicationContextAwareProcessor：用来为bean注入ApplicationContext等容器对象
这些注解类的BeanPostProcessor，在spring配置文件中，可以通过这样的配置 <context:component-scan base-package="*.*" /> ，自动进行注册。（spring通过ComponentScanBeanDefinitionParser类来解析该标签）

### InitializingBean 与 init-method 区别？

InitializingBean 直接调用接口
init-method 反射方式

### IOC 和依赖注入 DI 关系

控制反转（Inversion of Control） 就是依赖倒置原则的一种代码设计的思路。具体采用的方法就是所谓的依赖注入（Dependency Injection）。上层需要什么，底层提供什么。

控制反转是一种以给予应用程序中目标组件更多控制为目的设计范式，并在我们的实际工作中起到 了有效的作用。

依赖注入是在编译阶段尚未知所需的功能是来自哪个的类的情况下，将其他对象所依赖的功能对象 实例化的模式。

1. 构造器注入

2. Setter 方法注入 
3. 接口注入

### Spring 初始化过程

Spring 框架提供了以下四种方式来管理 bean 的生命周期事件:

- InitializingBean 和 DisposableBean 回调接口
- 针对特殊行为的其他 Aware 接口
- Bean配置文件中的Custom init()方法和destroy()方法

@PostConstruct 和@PreDestroy 注解方式

1. 通过构造器或工厂方法创建 Bean 实例
2. 为 Bean 的属性设置值和对其他 Bean 的引用
3. 将 Bean 实 例 传 递 给 Bean 后 置 处 理 器 的 postProcessBefore.Initialization 方法
4. 调用 Bean 的初始化方法(init-method)
5. 将 Bean 实 例 传 递 给 Bean 后 置 处 理 器 的 postProcessAfter.Initialization 方法 
6. Bean 可以使用了
7. 当容器关闭时, 调用 Bean 的销毁方法(destroy-method)

### 三种初始化方式？

InitializingBean、@PostConstruct（推荐）、init-method三种。

执行顺序：@PostConstruct》InitializingBean》init-method

https://mp.weixin.qq.com/s/I7zgbOoCgAUnRFy4avipiQ

### 三种销毁 Spring Bean 的方式

DisposableBean、@PreDestroy、destroy-method

执行顺序：@PreDestroy》DisposableBean》destroy-method

### Spring中用了哪些设计模式？

工厂模式： Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象
模板模式：Spring 中 jdbcTemplate、hibernateTemplate 
观察者模式：事件驱动
适配器模式 :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller
代理模式：Spring AOP 功能的实现
单例设计模式 : Spring 中的 Bean 默认都是单例的。
包装器设计模式 : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。

### **Spring Bean** 的作用域之间有什么区别?

1. singleton:这种 bean 范围是默认的，这种范围确保不管接受到多少个请求，每个容器中只有一个 bean 的实例，单例的模式由 bean factory 自身来维护。

2. prototype:原形范围与单例范围相反，为每一个 bean 请求提供一个实例。

3. request:在请求 bean 范围内会每一个来自客户端的网络请求创建一个实例，在请求完成以后，

   bean 会失效并被垃圾回收器回收。

4. Session:与请求范围类似，确保每个 session 中有一个 bean 的实例，在 session 过期后，bean

   会随之失效。

5. global- session:global-session 和 Portlet 应用相关。当你的应用部署在 Portlet 容器中工作 时，它包含很多 portlet。如果 你想要声明让所有的 portlet 共用全局的存储变量的话，那么这**全局变量需要存储在 global-session 中**。

### 什么是 **Spring inner beans**?

在 Spring 框架中，无论何时 bean 被使用时，当仅被调用了一个属性。一个明智的做法是将这个 bean 声明为内部 bean。内部 bean 可以用 setter 注入“属性”和构造方法注入“构造参数”的方式来 实现。

比如，在我们的应用程序中，一个 Customer 类引用了一个 Person 类，我们的要做的是创建一个 Person 的实例，然后在 Customer 内部使用。

### **Spring** 框架中的单例 **Beans** 是线程安全的么?

 Spring 框架并没有对单例 bean 进行任何多线程的封装处理。关于单例 bean 的线程安全和并发问题需要**开发者自行去搞定**。但实际上，大部分的 Spring bean 并**没有可变的状态**(比如 Service 类 和 DAO 类)，所以在某种程度上说 Spring 的单例 bean 是线程安全的。如果你的 bean 有多种状 态的话(比如 View Model 对象)，就需要自行保证线程安全。
 最浅显的解决办法就是**将多态 bean 的作用域由“singleton”变更为“prototype”。**

### 请解释 **Spring Bean** 的自动装配?

在 Spring 框架中，在配置文件中设定 bean 的依赖关系是一个很好的机制，Spring 容器还可以自动装配合作关系 bean 之间的关联关系。这意味着 Spring 可以通过向 Bean Factory 中注入的方式自动搞定 bean 之间的依赖关系。自动装配可以设置在每个 bean 上，也可以设定在特定的 bean 上。

### 请解释自动装配模式的区别?

1. no:这是 Spring 框架的默认设置，在该设置下自动装配是关闭的，开发者需要自行在 bean 定义 中用标签明确的设置依赖关系。
2. byName:该选项可以根据 bean 名称设置依赖关系。当向一个 bean 中自动装配一个属性时，容 器将根据 bean 的名称自动在在配置文件中查询一个匹配的 bean。如果找到的话，就装配这个属 性，如果没找到的话就报错。
3. byType:该选项可以根据 bean 类型设置依赖关系。当向一个 bean 中自动装配一个属性时，容器 将根据 bean 的类型自动在在配置文件中查询一个匹配的 bean。如果找到的话，就装配这个属性， 如果没找到的话就报错。
4. constructor:造器的自动装配和 byType 模式类似，但是仅仅适用于与有构造器相同参数的 bean，如果在容器中没有找到与构造器参数类型一致的 bean，那么将会抛出异常。
5. autodetect:该模式自动探测使用构造器自动装配或者 byType 自动装配。首先，首先会尝试找合 适的带参数的构造器，如果找到的话就是用构造器自动装配，如果在 bean 内部没有找到相应的构 造器或者是无参构造器，容器就会自动选择 byTpe 的自动装配方式。

### 请举例解释**@Required** 注解?

被该注解修饰的set方法会被要求该属性必须被设置值。

### 简述 **AOP** 和 **IOC** 概念

 **AOP:**Aspect Oriented Program, 面向(方面)切面的编程;Filter(过滤器) 也是一种 AOP. AOP 是一种 新的方法论, 是对传统 OOP(Object-Oriented Programming, 面向对象编程) 的补充. AOP 的 主要编程对象是切面(aspect), 而切面模块化横切关注点.可以举例通过事务说明.

IOC: Invert Of Control, 控制反转. 也成为 DI(依赖注入)其思想是反转资源获取的方向. 传统 的资源查找方式要求组件向容器发起请求查找资源.作为回应, 容器适时的返回资源. **而应用了 IOC 之后, 则是容器主动地将资源推送给它所管理的组件**,组件所要做的仅是选择一种合适的方式 来接受资源. 这种行 为也被称为查找的被动形式