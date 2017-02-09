title: spring使用陷阱六 

#  Spring陷阱之context:component-scan扫描与事务失效+Hibernate4 getCurrentSession 

近日，使用Spring+Hibernate4,安装Hibernate4在spring当中的配法配好后，发现还是报错
` No Session found for current thread `

##  Hibernate4的主要配置变动: 

```

<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
  
<prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate4.SpringSessionContext</prop> 
  
 <bean id="txManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
 <!--hibernate4必须配置为开启事务 否则 getCurrentSession()获取不到-->
            <tx:method name="get*" propagation="REQUIRED" read-only="true" />
            <tx:method name="count*" propagation="REQUIRED" read-only="true" />
            <tx:method name="find*" propagation="REQUIRED" read-only="true" />
            <tx:method name="list*" propagation="REQUIRED" read-only="true" />

```
当有一个方法list 传播行为为Supports，当在另一个方法getPage()（无事务）调用list方法时会抛出org.hibernate.HibernateException: No Session found for current thread 异常。
这是因为getCurrentSession()在没有session的情况下不会自动创建一个，不知道这是不是Spring3.1实现的bug，欢迎大家讨论下。
**因此最好的解决方案是使用REQUIRED的传播行为。**

以下为详细配置文件
```

	<!--Hibernate SessionFatory-->
	<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="packagesToScan">
		     <list>
               <value>com.xs.demo.entity</value>
		     </list>
		</property>
 		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">${hibernate.dialect}</prop>
				<prop key="hibernate.show_sql">${hibernate.show_sql}</prop>
                <prop key="hibernate.format_sql">${hibernate.format_sql}</prop>
                <prop key="hibernate.hbm2ddl.auto">${hibernate.hbm2ddl.auto}</prop>
                <prop key="hibernate.cache.use_query_cache">${hibernate.cache.use_query_cache}</prop>
                <prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate4.SpringSessionContext</prop> 
                <prop key="hibernate.autoReconnect">true</prop> 
			</props>
		</property>
	
		
	</bean>

```
```


 <bean id="txManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
	<bean class="com.xs.demo.dao.UserDao">
		 <property name="sessionFactory" ref="sessionFactory"/>
	</bean>
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED" />
            <tx:method name="add*" propagation="REQUIRED" />
            <tx:method name="create*" propagation="REQUIRED" />
            <tx:method name="insert*" propagation="REQUIRED" />
            <tx:method name="update*" propagation="REQUIRED" />
            <tx:method name="modify*" propagation="REQUIRED" />
            <tx:method name="upload*" propagation="REQUIRED" />
            <tx:method name="merge*" propagation="REQUIRED" />
            <tx:method name="del*" propagation="REQUIRED" />
            <tx:method name="remove*" propagation="REQUIRED" />
            <tx:method name="move*" propagation="REQUIRED" />
            <tx:method name="change*" propagation="REQUIRED" />
            <tx:method name="put*" propagation="REQUIRED" />
            <tx:method name="use*" propagation="REQUIRED"/>
            <tx:method name="log*" propagation="REQUIRED"/>
            <tx:method name="sh*" propagation="REQUIRED"/>
            <tx:method name="bh*" propagation="REQUIRED"/>
            <tx:method name="sf*" propagation="REQUIRED"/>
            <tx:method name="bj*" propagation="REQUIRED"/>
            <tx:method name="tf*" propagation="REQUIRED"/>
            <tx:method name="mobileLogin" propagation="REQUIRED"/>
            <tx:method name="register*" propagation="REQUIRED"/>
            <tx:method name="goto*" propagation="REQUIRED"/>
            <tx:method name="active*" propagation="REQUIRED"/>
            <tx:method name="send*" propagation="REQUIRED"/>
            <tx:method name="handel*" propagation="REQUIRED"/>
            <tx:method name="attendance*" propagation="REQUIRED"/>
            <tx:method name="batch" propagation="REQUIRED"/>
            <!--hibernate4必须配置为开启事务 否则 getCurrentSession()获取不到-->
            <tx:method name="get*" propagation="REQUIRED" read-only="true" />
            <tx:method name="count*" propagation="REQUIRED" read-only="true" />
            <tx:method name="find*" propagation="REQUIRED" read-only="true" />
            <tx:method name="list*" propagation="REQUIRED" read-only="true" />
            <tx:method name="*" read-only="true" />
        </tx:attributes>
    </tx:advice>
    <aop:config expose-proxy="true">
        <!-- 只对业务逻辑层实施事务 -->
        <aop:pointcut id="txPointcut" expression="execution(* com.xs.demo.service.*.*(..))" />
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>

```
##  原因查找 

但是还是提示：No Session found for current thread
后来认真看了http://jinnianshilongnian.iteye.com/blog/1762632 http://outofmemory.cn/java/spring/spring-DI-with-annotation-context-component-scan  http://jinnianshilongnian.iteye.com/blog/1423971  ,才明白是由于我的配置导致的事务不起作用。
如下方式可以成功扫描到@Controller注解的Bean**，不会扫描@Service/@Repository的Bean。**正确
```

 <context:component-scan base-package="org.bdp.system.test.controller">   
     <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>   
</context:component-scan> 

``` 
  
但是如下方式，**不仅仅扫描@Controller，还扫描@Service/@Repository的Bean**，可能造成一些问题
```

 <context:component-scan base-package="org.bdp">   
     <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>   
</context:component-scan> 

``` 
这个尤其在springmvc+spring+hibernate等集成时最容易出问题的地，最典型的错误就是：**` 事务不起作用 `**
默认` use-default-filters `默认为true，所以即使配置了context:include-filter为@Controller,它还会自动注册对@Component、@ManagedBean、@Named注解的Bean进行扫描。如果细心，到此我们就找到问题根源了。
即
**use-default-filters="true"(默认值就是true)**
它的作用就是扫描一些相关的注解，不管是否设置白名单，都对包括了@controller,@Component,@Service,@Repository等进行扫描
而use-default-filters="false",仅仅使用白名单context:include-filter对指定的注解进行扫描
如：
```

 <context:component-scan base-package="org.bdp.system.test.controller" use-default-filters="false">     
     <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>     
</context:component-scan>  

``` 
这就表示只扫描"org.bdp.system.test.controller"下的Controller注解。因为此时use-default-filters="false"

applicationContext.xml中已经扫描过@Service和@Repository,而在springmvc-servlet.xml扫描@Controller时就不应该再去扫描@Service和@Repository，不然就会导致事物失效的情况
如果在springmvc配置文件，不使用cn.javass.demo.web.controller前缀，而是使用cn.javass.demo，**则service、dao层的bean可能也重新加载了，但事务的AOP代理没有配置在springmvc配置文件中，从而造成新加载的bean覆盖了老的bean，造成事务失效。只要使用use-default-filters=“false”禁用掉默认的行为就可以了。**

##  解决办法： 
组件扫描(context:component-scan)是分离应用上下文层次的关键（根应用上下文和DispatcherServlet应用上下文）。如果启用了组件扫描就必须正确的配置。否则可能出现重复的bean定义，甚至导致事务失效等。
组件扫描基于两个准则进行工作：包扫描(base-package)和类过滤(context:exclude-filter和context:include-filter)。
**applicationContext.xml下：**
用于扫描@Component,@Service,@Repository等，除了@Controller
```

 <context:component-scan base-package="com.xs.demo">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

```
` 错误的配置： `
```

 <context:component-scan base-package="com.xs.demo" />

```

**springmvc-servlet.xml下：**
用于扫描@Controller除了@Component,@Service,@Repository等,以避免覆盖而导致事务失效。
第一种方式：
```

    <context:component-scan base-package="com.xs.demo.controller">
    	<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

```
第二种方式：
```

    <context:component-scan base-package="com.xs.demo" use-default-filters="false">
    	<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

```
此节点有两个属性base-package属性告诉spring要扫描的包，**use-default-filters="false"表示不要使用默认的过滤器，此处的默认过滤器，会扫描包含Service,Component,Repository,Controller注解修饰的类**，而此处我们处于示例的目的，故意将use-default-filters属性设置成了false。

` 错误的配置： `
```

    <context:component-scan base-package="com.xs.demo">
    	<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

```

##  附录：<context:component-scan> 
<context:component-scan>提供两个子标签：<context:include-filter>和<context:exclude-filter>各代表引入和排除的过滤。
如：<context:component-scan base-package="com.xhlx.finance.budget" >
<context:include-filter type="regex" expression=".service.*"/>
</context:component-scan>
filter标签在Spring3有五个type，如下
![](/data/dokuwiki/spring/pasted/20151024-091214.png)
将filter的type设置成了正则表达式，regex，注意在正则里面.表示所有字符，**而\.才表示真正的.字符**。我们的正则表示以Dao或者Service结束的类。