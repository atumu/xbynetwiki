title: spring_framework介绍 

#  spring framework介绍与配置、应用上下文、注解组件扫描机制 
Spring Framework是一个Java应用程序容器。它提供了许多有用的特性，如反转控制IoC，依赖注入DI,抽象数据访问，AOP，事物管理等。它创建于2002年。Spring3.2发布于2012年。2013年12月发布Spring Framework 4.0
Spring Framework容器可以运行于任何JavaEE应用服务器、JavaWeb服务器、JavaSE独立应用程序。
Spring Framework是Java工具箱中最强大的工具。

##  主要特性 
1、反转控制和依赖注入
**反转控制IoC是一个软件设计模式**，组装器将在运行时绑定对象而不是编译时绑定对象。例如ServiceA依赖于另一个ServiceB，该依赖对象将在运行时实现而不是由ServiceA直接实例化ServiceB。通过这种方式，应用可以针对一组接口进行编程，这样可以在不同的环境中进行切换，而无需重新编译代码。
尽管理论上可以有多种方式实现IoC，但是DI是最常见的技术。
**依赖注入DI:**是反转控制IoC这种软件设计模式的一种技术实现。通过使用DI，一段程序代码可以什么它依赖于另一个程序代码（一个接口），然后组装器可以在运行时注入它依赖的实现**（通常但不总是单例）**。

2、面向切面编程
SpringAOP：分离横切关注点。面向切面编程。

3、数据访问与事务管理

4、应用程序消息
Spring Framework提供了一个松耦合的消息系统，使用发布-订阅模式。

5、SpringMVC

##  了解应用上下文 
Spring Framework**容器**以**一个或多个**应用上下文的形式存在。由ApplicationContext接口表示。一个应用上下文管理着一组bean、执行业务逻辑的Java对象，执行任务、持久化、web得等。由Spring管理的bean可以自动进行依赖注入，消息通知，定时方法执行、bean验证和执行其他关键的Spring服务。

一个Spring应用至少需要一个应用上下文。**它也可以是由多个应用上下文组成的层次结构。**在这样的层次结构中，**任何由Spring管理的bean都可以访问相同的应用上下文，父亲应用上下文，祖先应用上下文中的bean。但它们不能访问兄弟或孩子应用上下文的bean**，这对于定义一组共享的应用模块来说是非常有用的。
```

<!-- 通过配置ContextLoaderListener与全局的contextConfigLocation参数用于启用一个全局的Spring应用上下文，
	作为所有的DispatcherServlet应用上下文的父亲，可以共享这个父亲的所有bean与配置 -->
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
         classpath:spring-hibernate.xml
        </param-value>
      </context-param>

  	<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<!-- 该servlet应用上下文的父亲为前面的全局应用上下文，它可以共享全局应用上下文里的配置与bean等-->
	<servlet>
		<servlet-name>springmvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
             	<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc-servlet.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>

	</servlet>
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern> <!-- 注意不要配置为/*,不然JSP机制不能直接处理JSP请求等，这不是我们期望的结果-->
	</servlet-mapping>

	<!--静态资源配置到default servlet这比使用SpringMVC的<mvc:resources mapping="/static/**" location="/static/" />要有效，要更好
	更具体的URL模式总是会覆盖/，所以静态资源URL映射可以覆盖前面的配置不走DispatcherServlet，而走由Web服务器容器初始化的default servlet。
	-->
     <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/resource/*</url-pattern>
        <url-pattern>/static/*</url-pattern>
    </servlet-mapping>

```

许多接口都继承了ApplicationContext，也有许多类实现了它：
  * ConfigurableApplicationContext接口。
  * WebApplicationContext和ConfigurableWebApplicationContext：提供了对底层ServletContext和ServletConfig的访问
  * ClassPathXmlApplicationContext和FileSystemXmlApplicationContext被设计在独立的JavaSE应用中从XML文件中加载配置。而XmlWebApplicationContext被设计为在JavaEE Web应用程序中实现相同的目标。
对于SpringMVC应用，每个DispatcherServlet实例都有自己的应用上下文。其中包含了对Web应用的ServletContext和它自己的ServletConfig的引用。我们可以创建多个和DispatcherServlet。` **通常我们需要配置一个全局根应用上下文。它将作为所有DispatcherServlet应用上下文的父亲。** `这个全局根应用上下文是通过配置org.springframework.web.context.` ContextLoaderListener `这个监听器创建的。它也有Web应用的ServletContext的引用。

注意上面两个contextConfigLocation参数并不冲突，前者作用域整个Servlet上下文，而后者只是作用于它所指定的Servlet。

##  ServletContainerInitializer 
Java EE6提供了一个新的接口` ServletContainerInitializer `,实现了这个接口的类将在应用程序开始时启动（比实现ServletContextListener要早），并在所有监听器启动之前调用它们的onStartup方法。这是应用程序生命周期中最早可以使用的时间点。不能在web.xml中配置ServletContainerInitializer。相反，需要使用Java的服务提供系统声明实现了ServletContainerInitializer接口的一个或多个类，在文件/META-INF/services/ServletContainerInitializer中列出他们，每行一个类。例如:
com.test.MyServletContainerInitializer1
com.test.MyServletContainerInitializer2

这种方式不利的一面是不支持WAR文件打包。只能打包成jar包。
SpringFramework提供了一个桥接口，使这种方式更加容易实现。SpringServletContainerInitializer类实现了ServletContainerInitializer接口。它在启动时会扫描实现了WebApplicationInitializer接口的类，并调用所有匹配实现类的onStartup方法。在WebApplicationInitializer实现类中，可以通过编程的方式配置监听器、Servlet、过滤器等，也可以从该类中启动Spring。例如下面这段代码：（仅作为示例）
```

import org.springframework.web.WebApplicationInitializer;
import org.springframework.web.context.ContextLoaderListener;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration;

@SuppressWarnings("unused")
public class Bootstrap implements WebApplicationInitializer
{
    @Override
    public void onStartup(ServletContext container) throws ServletException
    {
        container.getServletRegistration("default").addMapping("/resource/*");

        AnnotationConfigWebApplicationContext rootContext =
                new AnnotationConfigWebApplicationContext();
        rootContext.register(RootContextConfiguration.class);
        container.addListener(new ContextLoaderListener(rootContext));

        AnnotationConfigWebApplicationContext servletContext =
                new AnnotationConfigWebApplicationContext();
        servletContext.register(ServletContextConfiguration.class);
        ServletRegistration.Dynamic dispatcher = container.addServlet(
                "springDispatcher", new DispatcherServlet(servletContext)
        );
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");
    }
}


```
##  注解与配置、Aware接口 
组件注解：
  * @Component
  * @Controller
  * @Repository
  * @Service
  * @Aspect:@Before,@After,@Around
  * JPA注解
  * 事务注解

自动注入
  * @Autowired:通过类型   对应于Java标准的@Inject
  * @Qualifier：通过bean名字  对应于Java标准的@Named
  * @Primary:存在多个实现某个依赖接口的类时指定优先级

org.springframework.beans.factory.Aware接口实现：
  * ApplicationContextAware
  * **BeanFactoryAware：用于获得BeanFactory，通过它可以手动获得或创建bean。**
  * EnvironmentAware：用于获得Environment对象，通过它可以从属性源中获得属性。
  * ServletContextAware用于获取JavaWeb应用的ServletContext
  * ServletConfigAware

注解扫描与bean自动配置
```

<!--注解扫描 -->
<mvc:annotation-driven/>  <!-- 用于SpringMVC-->
<aop:aspectj-autoproxy proxy-target-class="true" /> <!--用于AOP -->
<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor" /> <!-- JPA注解支持 -->

<task:annotation-driven /><!-- 任务调度 -->
<context:property-placeholder location="classpath:cfg.properties"/>	<!-- 引入配置文件 -->
<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" /> <!--事务异常处理 -->
<!-- 开启Spring提供的基于注解的声明式事务管理 -->
<tx:annotation-driven transaction-manager="txManager" proxy-target-class="true"/>

```

##  注解组件扫描与component-scan 
在多个spring配置文件存在时需要注意component-scan扫描的bean不能相互覆盖。不然有可能导致事务失效等诡异现象。
可以通过缩小base-package范围，使用exclude-filter，include-filter结合use-default-filters="false"等方式来解决。如下：
Spring-Global.xml
```

	<context:component-scan base-package="net.xby1993.springmvc">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>

```
Spring-mvc.xml
```

	<context:component-scan base-package="net.xby1993.springmvc.controller" use-default-filters="false">
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>

```
use-default-filters="false"告诉spring忽略它的标准扫描模式，它告诉Spring只扫描@Controller，不然默认会扫描所有标准组件注解，这就有可能导致全局配置中的bean被覆盖，尤其是事务。



