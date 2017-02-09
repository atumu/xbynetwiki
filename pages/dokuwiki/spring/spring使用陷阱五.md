title: spring使用陷阱五 

#  Spring陷阱之多个视图解析器的配置 
参考:http://www.cnblogs.com/rollenholt/archive/2012/12/27/2836035.html
在Spring MVC应用程序中，我们经常需要应用一些视图解析器策略来解析视图名称。例如，联合使用三个视图解析器：InternalResourceViewResolver、ResourceBundleViewResolver和XmlViewResolver。
但是，如果返回了一个视图的名称，那么，使用哪一个视图解析器策略?
解决方法
**如果应用了多个视图解析器策略，那么就必须通过“order”属性来声明优先级，` order值越低，则优先级越高 `**。例如：

<note important>注意：**InternalResourceViewResolver必须总是赋予最低的优先级（最大的order值）****，因为不管返回什么视图名称，它都将解析视图**。**如果它的优先级高于其它解析器的优先级的话，它将使得其它具有较低优先级的解析器没有机会解析视图。**</note>
```

 <bean class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
        <property name="basename">
            <value>spring-views</value>
        </property>
        <property name="order" value="0" />
    </bean>
 
    <bean class="org.springframework.web.servlet.view.XmlViewResolver">
        <property name="location">
            <value>/WEB-INF/spring-views.xml</value>
        </property>
        <property name="order" value="1" />
    </bean>
 
    <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
        <property name="prefix">
            <value>/WEB-INF/pages/</value>
        </property>
        <property name="suffix">
            <value>.jsp</value>
        </property>
        <property name="order" value="2" />
    </bean>

```