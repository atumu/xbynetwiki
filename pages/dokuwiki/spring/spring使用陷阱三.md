title: spring使用陷阱三 

#  Spring陷阱之spring获取webapplicationcontext,applicationcontext几种方法详解 
 Bean工厂（com.springframework.beans.factory.BeanFactory）是Spring框架最核心的接口，它提供了高级IoC的配置机制。BeanFactory使管理不同类型的Java对象成为可能，应用上下文（com.springframework.context.ApplicationContext）建立在BeanFactory基础之上，提供了更多面向应用的功能，它提供了国际化支持和框架事件体系，更易于创建实际应用。我们一般称BeanFactory为IoC容器，而称ApplicationContext为应用上下文。但有时为了行文方便，我们也将ApplicationContext称为Spring容器。
 对于两者的用途，我们可以进行简单划分：BeanFactory是Spring框架的基础设施，面向Spring本身；ApplicationContext面向使用Spring框架的开发者，几乎所有的应用场合我们都直接使用ApplicationContext而非底层的BeanFactory。
ApplicationContext的初始化和BeanFactory有一个重大的区别：BeanFactory在初始化容器时，并未实例化Bean，直到第一次访问某个Bean时才实例目标Bean；而ApplicationContext则在初始化应用上下文时就实例化所有单实例的Bean。因此ApplicationContext的初始化时间会比BeanFactory稍长一些

方法一：在初始化时保存ApplicationContext对象 
方法二：通过Spring提供的` WebApplicationContextUtils `类获取ApplicationContext对象。 一般通过实现ServletContextListener保存为全局变量。
方法三：继承自抽象类ApplicationObjectSupport 缺点无法采用static形式
方法四：继承自抽象类WebApplicationObjectSupport  缺点无法采用static形式
` 方法五：实现接口ApplicationContextAware  Web应用中强烈推荐此种方法 `
方法六：通过Spring提供的ContextLoader 经实验没有用。
方法七：实现` BeanFactoryAware `接口，把该接口配置到spring中，然后把getbean方法写成静态的. 未实验，理论可行。

**以上不管是继承类还是实现借口一定要注意**：` 一定要在Spring 的配置文件文件中进行配置。否则获取的ApplicationContext对象将为null `例如：
```

<bean class="net.xby1993.springmvc.util.SpringContextUtil"/>

```

获取spring中bean的方式总结： 
**方法一：在初始化时保存ApplicationContext对象**
```

ApplicationContext ac = new ClassPathXMlApplicationContext("applicationContext.xml"); 
ac.getBean("beanId"); 

``` 
说明：这种方式适用于采用Spring框架的独立应用程序，需要程序通过配置文件手工初始化Spring的情况。

**方法二：通过Spring提供的工具类获取ApplicationContext对象**
注意：当使用WebApplicationContextUtils获取ApplicationContext实例时，需要在` web.xml `配置文件中添加` org.springframework.web.context.ContextLoaderListener `监听器，否则获取不到ApplicationContext对象，返回Null。
```


ApplicationContext ac1 = WebApplicationContextUtils.getRequiredWebApplicationContext(ServletContext sc); 
ApplicationContext ac2 = WebApplicationContextUtils.getWebApplicationContext(ServletContext sc); 
ac1.getBean("beanId"); 
ac2.getBean("beanId"); 

``` 
```

public class MyApplicationListener
        implements ServletContextListener
{
    @Inject SessionRegistry sessionRegistry;
   
    @Override
    public void contextInitialized(ServletContextEvent event)
    {
        WebApplicationContext context =
                WebApplicationContextUtils.getRequiredWebApplicationContext(
                        event.getServletContext());
       
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) { }
}


```
说明：这种方式适合于采用Spring框架的B/S系统，通过ServletContext对象获取ApplicationContext对象，然后在通过它获取需要的类实例。
上面两个工具方式的区别是，前者在获取失败时抛出异常，后者返回null。

**方法三：继承自抽象类ApplicationObjectSupport**
说明：抽象类ApplicationObjectSupport提供getApplicationContext()方法，可以方便的获取ApplicationContext。
Spring初始化时，会通过该抽象类的setApplicationContext(ApplicationContext context)方法将ApplicationContext 对象注入。

**方法四：继承自抽象类WebApplicationObjectSupport**
通过继承org.springframework.web.context.support.WebApplicationObjectSupport使用` getWebApplicationContext() ` 获取到org.springframework.web.context.` WebApplicationContext `
由于Web应用比一般的应用拥有更多的特性，因此WebApplicationContext扩展了ApplicationContext。` WebApplicationContext定义了一个常量ROOT_WEB_APPLICATION_ CONTEXT_ATTRIBUTE `，在上下文启动时，WebApplicationContext实例即以此为键放置在` ServletContext的属性列表 `中，因此我们可以直接通过以下语句从Web容器中获取WebApplicationContext：
```

WebApplicationContext wac = (WebApplicationContext)servletContext.getAttribute(
WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);

```

**方法五：实现接口ApplicationContextAware**
说明：实现该接口的setApplicationContext(ApplicationContext context)方法，并保存ApplicationContext 对象。Spring初始化时，会通过该方法将ApplicationContext对象注入。
以下是实现ApplicationContextAware接口方式的代码，前面两种方法类似：
```

public class SpringContextUtil implements ApplicationContextAware {  
  
    // Spring应用上下文环境  
    private static ApplicationContext applicationContext;  
  
    /** 
     * 实现ApplicationContextAware接口的回调方法，设置上下文环境 
     *  
     * @param applicationContext 
     */  
    public void setApplicationContext(ApplicationContext applicationContext) {  
        SpringContextUtil.applicationContext = applicationContext;  
    }  
  
    /** 
     * @return ApplicationContext 
     */  
    public static ApplicationContext getApplicationContext() {  
        return applicationContext;  
    }  
  
    /** 
     * 获取对象 
     *  
     * @param name 
     * @return Object
     * @throws BeansException 
     */  
    public static Object getBean(String name) throws BeansException {  
        return applicationContext.getBean(name);  
    }  
}

```
虽然，spring提供的后三种方法可以实现在普通的类中继承或实现相应的类或接口来获取spring 的ApplicationContext对象，
但是在使用是一定要注意实现了这些类或接口的普通java类**一定要在Spring 的配置文件文件中进行配置。否则获取的ApplicationContext对象将为null。**

方法六：通过Spring提供的ContextLoader
```

WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();
wac.getBean(beanID);

```
最后提供一种不依赖于servlet,不需要注入的方式**。但是需要注意一点，在服务器启动时，Spring容器初始化时，不能通过以下方法获取Spring 容器，**细节可以查看spring源码org.springframework.web.context.ContextLoader。

##  通用方式 
方法五、用` BeanFactoryAware `接口,用BEAN的名称来获取BEAN对象
写一个类，实现BeanFactoryAware接口，把该接口配置到spring中，然后**把getbean方法写成静态的**，就可以动态获取了。下面是示例： 
```

public class Springfactory implements BeanFactoryAware {  
  
    private static BeanFactory beanFactory;  
  
    // private static ApplicationContext context;  
  
    public void setBeanFactory(BeanFactory factory) throws BeansException {  
        this.beanFactory = factory;  
    }  
  
    /** 
     * 根据beanName名字取得bean 
     *  
     * @param beanName 
     * @return 
     */  
    public static <T> T getBean(String beanName) {  
        if (null != beanFactory) {  
            return (T) beanFactory.getBean(beanName);  
        }  
        return null;  
    }  
  
}  

```
```

<bean id="beanFactoryHelper" class="com.cyjch.base.Springfactory"/>

```
使用的时候，通过Springfactory.getBean("beanName"),就可以获取到bean了。注意：这个是静态方法，直接通过类名去调用。



参考：http://www.meiriyouke.net/?p=154
http://www.blogjava.net/Todd/archive/2010/04/22/295112.html
http://www.meiriyouke.net/?p=154
http://www.iteye.com/problems/85363
http://www.cnblogs.com/cyjch/archive/2012/02/06/2340417.html
