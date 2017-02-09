title: chapter9_servlet动态注册和容器初始化 

#  Chapter9 Servlet动态注册和容器初始化 
##  动态注册 
为了使动态注册成为可能，** ServletContext** 接口中还添加了以下方法，用来动态地创建 Web 对象：
<T extends Filter>**createFilter**(java.lang.Class<T> clazz)
<T extends java.util.EventListener>** createListener**(java.lang.Class<T> clazz)
<T extends Servlet>** createServlet**(java.lang.Class<T> clazz)

例如，假设 MyServlet 是一个可以直接或间接实现 javax.servlet.Servlet 的类，那么可以通过调用 createServlet 方法将 MyServlet 实例化：

Servlet myServlet = createServlet(MyServlet.class)

创建好一个 Web 对象之后，可以利用以下任意一个方法将它添加到 ServletContext 中，这也是 Servlet3 的一项新特性。

FilterRegistration.Dynamic** addFilter**(java.lang.String filterName, Filter filter)

<T extends java.util.EventListener> void **addListener**(T t)

ServletRegistration.Dynamic **addServlet**(java.lang.String servletName, Servlet servlet)

同样地，也可以通过在 ServletContext 中调用以下任意一个方法，在创建 Web 对象的同时，将它添加到 ServletContext 中。

在创建或添加监听器时，传给第一个 addListener 覆盖方法的类必须实现以下其中一个或多个接口：

1、ServletContextAttributeListener

2、ServletRequestListener

3、ServletRequestAttributeListener

4、HttpSessionListener

5、HttpSessionAttributeListener

如果将 ServletContext 传给** ServletContextInitializer** 的 onStartup 方法，那么监听器类也可以**实现 ServletContextListener 接口**。

下面这个例子没有用注解，也没有用部署描述符进行声明。

```

@WebListener
public class DynRegListener implements ServletContextListener {

  @Override
  public void contextDestroyed(ServletContextEvent arg0) {
  }

  @Override
  public void contextInitialized(ServletContextEvent sce) {
    ServletContext servletContext = sce.getServletContext() ;
    Servlet firstServlet = null ;
    try {
      firstServlet = servletContext.createServlet(FirstServlet.class) ;
    } catch (ServletException e) {
      e.printStackTrace();
    }
    if(firstServlet != null && firstServlet instanceof FirstServlet){
      ((FirstServlet) firstServlet).setName("Dynamically registered servlet");
    }
    ServletRegistration.Dynamic dynamic = servletContext.addServlet("firstServlet", firstServlet) ;
    dynamic.addMapping("/dynamic") ;
  }

}

```
**当应用程序启动时，容器会调用监听器的contextInitialized方法**，结果是创建了一个FirstServlet实例，并注册和映射到/dynamic。可以利用下面这个路径访问FirstServlet
http://localhost:8080/filedowmload/dynamic
##   Servlet 容器初始化 
如果你使用过像 Struts 或 Struts2 这类 Java Web 框架，就应该知道，在使用框架之前必须先配置应用程序。一般来说，需要通过修改部署描述符，告诉 Servlet 容器你正在使用一个框架。例如，要想在应用程序中使用 Struts2 ，可以将以下标签添加到部署描述符中：
```

<filter>
  <filter-name>struts2</filter-name>
  <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>struts2</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>

```
**但在Servlet3中，就不再需要这些了。框架可以进行打包，自动完成Web对象的首次注册。**

**Servlet容器初始化的核心是javax.servlet.ServletContainerInitializer接口**。这是一个简单的接口，它只有一个方法：**onStartup**。在执行任何ServletContext监听器之前，由Servlet容器调用这个方法。

实现ServletContainerInitializer的类必须用**@HandleTypes**注解进行标注，以便声明初始化程序可以处理这些类型的类。

下面是个例子。要把下面的类和文件打成jar包。
```

@HandlesTypes({FirstServlet.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {

  @Override
  public void onStartup(Set<Class<?>> classes, ServletContext servletContext)
      throws ServletException {
    System.out.println("onStartup");
    ServletRegistration registration = servletContext.addServlet("firstServlet", "servlet.FirstServlet");
    registration.addMapping("/f");
    System.out.println("leaving onStartup");
  }

}

```
参考：http://www.tuicool.com/articles/ZRBvMr