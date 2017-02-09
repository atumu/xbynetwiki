title: spring使用陷阱四 

#  Spring陷阱之spring-webmvc和spring-web区别 
参考：http://stackoverflow.com/questions/13533700/maven-dependency-spring-web-vs-spring-webmvc
` spring-web ` provides core HTTP integration, including some handy servlet filters, Spring HTTP Invoker, infrastructure to integrate with other web frameworks and HTTP technologies (Hessian, Burlap).
` spring-webmvc ` **is an implementation of Spring MVC**.** spring-webvc depends on on spring-web**, thus including it will transitively add spring-web. You don't have to add spring-web explicitly.
You should depend only on spring-web if you don't use Spring MVC but want to take advantage of other web-related technologies that Spring support.
总之就是：
  * spring-web为spring的web开发提供很多便利特性，此时的spring-web可以结合Struts等一起使用。对于你不想使用spring-mvc，但是你可以使用spring-web简化开放。
  * spring-webmvc是SpringMVC的实现。依赖于spring-web。

一般即使你不使用SpringMVC也应该在你的web项目中加入spring-web.因为Spring web容器的自启动依赖于它。
当你在web.xml中配置Spring容器自启动时就用到了spring-web:org.springframework.web.context.ContextLoaderListener是spring-web下的类。
```

	<!-- 启动Spring容器 -->
  	<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

```