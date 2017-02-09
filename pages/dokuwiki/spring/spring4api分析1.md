title: spring4api分析1 

#  Spring4 API分析之IoC容器部分 
##  org.springframework.beans.factory包分析 
org.springframework.beans.factory——Spring轻量级IoC容器实现的核心包。
BeanFactory——访问Spring bean容器的顶级接口。
  * HierarchicalBeanFactory
  * ListableBeanFactory——可以枚举所有的bean实例，而不是一个一个地通过名字去查找。

Aware——标记接口，用于表示会被Spring容器回调通知的对象。
  * BeanClassLoaderAware——回调，用于获取当前的bean factory所使用的class loader
  * BeanFactoryAware——回调，用于获取当前的所用的BeanFactory对象
  * BeanNameAware——回调，用于获取bean name

FactoryBean<T>——用于在BeanFactory内部使用的bean实现的接口。
SmartFactoryBean<T>

NamedBean——与BeanNameAware对应。定义bean的name
ObjectFactory<T>

BeanFactoryUtils
###  BeanFactory分析 
BeanFactory接口的实现保持着大量的bean definitions,每一个bean definitions的名字都是唯一的。依赖于bean definitions,BeanFactory可以返回单例(Singleton)或多例(Prototype)形式的bean实例。(在web环境下还有request,session范围(scope))
Spring的依赖注入功能实现了BeanFactory和它的子接口。
BeanFactory会从配置源记载bean definitions.然后使用org.springframework.beans包中的类去配置beans。BeanFactory接口并没有约束配置源的存储方式(XML,JDBC，Code etc.)。具体由实现觉得。
HierarchicalBeanFactory的实现维护一个父factory引用关系。
ListableBeanFactory里面的方法会在调用时检查parent factories是否存在一个HierarchicalBeanFactory实现。如果某个bean在当前工厂中没有找到，那么它就会到父bean factory里去查找。
###  bean生命周期回调接口 
初始化回调顺序：
1. BeanNameAware's setBeanName
2. BeanClassLoaderAware's setBeanClassLoader
3. BeanFactoryAware's setBeanFactory
4. ResourceLoaderAware's setResourceLoader (only applicable when running in an application context)
5. ApplicationEventPublisherAware's setApplicationEventPublisher (only applicable when running in an application context)
6. MessageSourceAware's setMessageSource (only applicable when running in an application context)
7. ApplicationContextAware's setApplicationContext (only applicable when running in an application context)
8. ServletContextAware's setServletContext (only applicable when running in a web application context)
9. postProcessBeforeInitialization methods of BeanPostProcessors
10. InitializingBean's afterPropertiesSet
11. a custom init-method definition
12. postProcessAfterInitialization methods of BeanPostProcessors

bean销毁时回调顺序：
1. DisposableBean's destroy
2. a custom destroy-method definition

##  org.springframework.context包分析 
基于beans包构建，添加了国际化消息支持、基于观察者模式的事件发布订阅机制、资源加载能力等
  * ApplicationContext	
  * ApplicationContextAware
  * ApplicationContextInitializer<C extends ConfigurableApplicationContext>：Callback interface for initializing a Spring ConfigurableApplicationContext prior to being refreshed.通常用在web应用中，如果需要编程方式的application context初始化工作时。	
  * ApplicationEventPublisher
  * ApplicationEventPublisherAware
  * ApplicationListener<E extends ApplicationEvent>
  * ConfigurableApplicationContext
  * EnvironmentAware：org.springframework.core.env.Environment是一个代表当前运行应用的环境接口。它保存了两个关键来源的配置:**profiles和properties**.它扩展了` PropertyResolver ` 接口。通过PropertyResolver 接口中的方法访问property。1、一个profile时一个命名的，属于一个逻辑组的bean definitions。只有在当前profile is active时才将其注册到容器。可以通过XML或注解@Profile来配置.2、properties来源于很多方式:properties文件、JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc Properties objects, Maps, and so on.由ApplicationContext管理的Bean可以通过**实现EnvironmentAware接口或者通过@Inject Environment来获取Enviroment实例**，进而获取profile配置或者properties。大多数情况下，应用层级的beans不需要直接和Environment直接交互。而是使用 ${..}(会被property placeholder configurer替换。例如` PropertySourcesPlaceholderConfigurer `这是一个实现了 EnvironmentAware接口的类，在Spring3.1开始通过使用` <context:property-placeholder/> `来实现默认注册。）
  * MessageSource 策略接口，用于处理消息。支持国际化和参数化消息。
  * MessageSourceAware
  * Lifecycle 通用接口，定义了一些方法用于start/stop lifecycle control.
  * LifecycleProcessor策略接口，用于处理ApplicationContext中的beans的生命周期
  * Phased
  * ResourceLoaderAware
  * SmartLifecycle

类:
ApplicationEvent
PayloadApplicationEvent<T>	
###  ApplicationContext 
ApplicationContext接口扩展EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver(ResourceLoader的子接口)
提供应用配置的核心接口。在运行时是只读的，不过如果实现支持的话可以reload。
ApplicationContext提供如下功能:
  * Bean factory方法来访问应用组件.(继承自 ListableBeanFactory.)
  * 加载文件资源的能力(继承自ResourcePatternResolver(ResourceLoader的子接口))。而DefaultResourceLoader提供了独立于ApplicationContext之外的资源加载能力。
  * 发布事件给注册的监听器的能力。(继承自ApplicationEventPublisher接口)
  * 接收处理消息的能力,包括国际化消息支持(继承自ResourcePatternResolver(ResourceLoader的子接口))
  * 继承自parent context.子代的Definitions拥有较高的优先级。这意味着` 你可以用一个单独的parent context配置整个web应用，然后再给各个servlet单独配置互相独立的child context. `
  * 除了继承了标准的BeanFactory生命周期能力之外，还提供了调用ApplicationContextAware、ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware的生命周期回调

关于ApplicationContextAware实现有了简单的便利抽象类可供继承ApplicationObjectSupport或WebApplicationObjectSupport.