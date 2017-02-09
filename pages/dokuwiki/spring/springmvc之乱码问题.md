title: springmvc之乱码问题 

#  springmvc之乱码问题 
web.xml配置:这个配置放在最前面.` 但是这只能解决POST中文乱码。而$.ajax默认为GET方式 `，所以需要使用时显示指定method:"POST"
```

<!-- Spring相关 -->
	<servlet>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring-mvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

```