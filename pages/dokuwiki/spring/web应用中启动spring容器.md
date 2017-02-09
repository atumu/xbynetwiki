title: web应用中启动spring容器 

#  WEB应用中创建并启动加载Spring容器 
**这是针对非SpringMVC的web项目。SpringMVC项目无需考虑这一点。一旦配置好SpringMVC会自动启动。**

**使用spring的web应用时，不用手动创建spring容器，而是通过配置文件声明式地创建spring容器**，因此，在web应用中创建spring容器有如下两种方式：
**为了让spring容器随web的应用的启动而自动启动**，有如下两种方法
  * 1.利用org.springframework.web.context.` ContextLoaderListener `实现
  * 2.采用load-on-startup Servlet ` Spring的ContextLoaderServlet `实现
##   1.` ContextLoaderListener `实现 
该类可以作为listener 使用，它会在创建时自动查找` WEB-INF/ 下的applicationContext.xrnl ` 文件。因此，如果只有一个配置文件，并且文件名为` applicationContext.xml ` ，则只需在` web.xml `
文件中增加如下代码即可:
```

<listener>
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

```
如果有多个配置文件需要载入，则考虑使用` <context-param> `即元素来确定配置文件的文件名。由于` ContextLoaderListener `加载时，会查找名为` contextConfigLocation `的参数。
因此，配置context-param时参数名字应该是contextConfigLocation。
```

<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
          <!--多个配置文件之间以，隔开，要么/WEB-INF要么classpath:-->
            /WEB-INF/spitter-security.xml,classpath:service-context.xml,
        </param-value>
    </context-param>
<listener>
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

```

##  2、采用load-on-startup Servlet 实现。 
该Servlet 在启动时，会自动查找` WEB-IN日下的applicationContext.xml ` 文件。
当然，为了让ContextLoaderServlet 随应用启动而启动，**应将此Servlet 配置成` load-on-startup ` 的Servlet**.load-on-startup 的值小一点比较合适，因为要保证Application
Context 优先创建。如果只有一个配置文件，并且文件名为` applicationContext.xml ` ，则在` web.xml ` 文件中增加如下代码即可:
```

<servlet>
<servlet-name>context</servlet-name>
<servlet-class>org.springframework.web.context.ContextLoaderServlet</
servlet-class>
<load-on-startup>l</load-on-startup>
</servlet>

```

参考：
http://blog.163.com/bison_001/blog/static/54262580200710190448853/
http://blog.csdn.net/zsm653983/article/details/8156750
