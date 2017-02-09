title: spring使用陷阱二 

#  Spring陷阱之applicationContext.xml与SpringMVC的xxx-servlet.xml的区别 
` 如果使用了SpringMVC，事实上，bean的配置完全可以在xxx-servlet.xml中进行配置。而不用添加applicationContext.xml文件的。 `  

**那为什么需要applicationContext.xml？**
  * 第一、非web的JavaSE项目中可以使用。不过是通过ClasspathXmlApplicationContext初始化容器。
  * 第二、采用非SpringMVC技术，让Spring与其它框架结合使用。这里也要用到applicationContext.xml
使用applicationContext.xml文件时是需要在` web.xml `中添加listener的：
```

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

```

还有最重要的一点。即使是采用SpringMVC，但是在web.xml中定义了多个DispatcherServlet的时候有多个对应的spring应用上下文，那么这些上下文之间如何共享bean，那么可以通过配置这个全局配置来解决该问题。
这个全局配置会作为所有DispatcherServlet应用上下文的父亲，所以可以共享父亲的bean：
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
 <!--    注意，这个Servlet名很重要，它决定了Spring应用上下文的加载文件名为spitter-servlet.xml-->
     <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc-servlet.xml</param-value>
		</init-param>
        <load-on-startup>1</load-on-startup>
       
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

<!--  解决SpringMVC post中文乱码 -->
    <filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
		<init-param>  
            <param-name>forceEncoding</param-name>  
            <param-value>true</param-value>  
        </init-param>  
	</filter>
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

 <!--  用于普通类中获取session和request -->   
 <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
 </listener>

```

参考：http://blog.csdn.net/zb0567/article/details/7930642