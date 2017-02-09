title: spring加载多配置文件 

#  Spring加载多个配置文件的几种方式 
### 方式一Spring配置文件导入： 
```

<import resource="services.xml"l>
<import resource="resources/messageSource.xml"l>
<!-- 导入第二份配置文件: resourcesl themeSource.xml -->
<import resource="/resources/themeSource.xml"l>

```
##  方式二：配置上下文加载参数： 
<listener> 
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class> 
</listener> 
** 非SpringMVC项目需要启用上面的ContextLoaderListener，而SpringMVC则不需要**
spring是如何加载配置文件肯定也跟 ` ContextLoaderListener `类有关，该类可以作为weblistener 使用，它会在创建时**自动查找WEB-INF/ 下的applicationContext.xml** 文件。因此，如果只有一个配置文件，并且文件名为applicationContext.xml ，则只需在web.xml加上面代码即可。 
如果有多个配置文件需要载入，则考虑使用` <context-param> `即元素来确定配置文件的文件名。由于ContextLoaderListener加载时，会查找名为` contextConfigLocation `的参数。因此，配置context-param时参数名字应该是contextConfigLocation。**所以context-param参数的名字是固定的contextConfigLocation.** 
` 多个配置文件用","分开，也可以使用通配符"*"加载多个配置文件。 `
```


	<!-- 通过配置ContextLoaderListener与全局的contextConfigLocation参数用于启用一个全局的Spring应用上下文，
	作为所有的DispatcherServlet应用上下文的父亲，可以共享这个父亲的所有bean与配置 -->
  	<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
           /WEB-INF/spitter-security.xml，
            classpath:service-context.xml，
         classpath:spring-hibernate.xml
        </param-value>
      </context-param>

  	<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

```
##  方式三：java代码方式加载多个: 
```

假设有3 个配置文件: a.xml , b.xml , c.xml
String[] configLocations = {"a.xml" , "b.xml" , "c.xml"}
以配置文件数组为参数，创建ApplicationContext
ApplicationContext ctx = new ClassPathXmlApplicationContext(configLocations);

```
