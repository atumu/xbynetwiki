title: hibernate4新特性 

#  Hibernate4新特性 
参考：http://www.docin.com/p-774870874.html
http://blog.sina.com.cn/s/blog_3e20fc040100yhzq.html
先从代码上区分以下与Hibernate3的一些改变：
  * 数据库方言的设置
  * buildSessionFactory
  * annotation
  * 事务，hibernateTemplate
  * 自动建表
  * 二级缓存配置
  * 与spring整合配置文件

**1、数据库方言的设置：**
<property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>
3.x版本是MySQLDialect

**2、buildSessionFactory**
4.1版本中buildSessionFactory()已经被buildSessionFactory(ServiceRegistry servoceRegistry)取代
解决办法：
```

Configuration cfg=new Configuration();
ServiceRegistry serviceRegistry=new ServiceRegistryBuilder().applySettings(cfg.getProperties()).buildServiceRegistry();
Session Factory sf=sf.configure.buildSessionFactory(serviceRegistry);

```

**3、annotation**
org.hibernate.cfg.AnnotationConfiguration已经被废弃
4.x版本读取配置不需要特别注明是注解，直接用Configuration cfg=new Configuration();就可以读取注解了。
Hibernate4.1版本中推荐使用annotation配置，所以在引进jar包时把requested里面的包全部引进来就已经包含了annotation必须包

**4、事务，hibernateTemplate**
hibernate4已经完全可以实现事务了，与Spring3.0中的hibernatedao,hibernateTemplate等有冲突，所以Spring3.1里不再提供hibernatedaosupport,hibernateTemplate

**5、自动建表**
Hibernate4.1已经可以自动建表，所以开发时只需要自己开发类然后配置好就ok。不需要考虑怎么建表。

**6、.二级缓存配置**
原来3.3+：
```

        <property name="cache.use_query_cache">true</property>        
        <property name="cache.use_second_level_cache">true</property>        
        <property name="cache.provider_class">org.hibernate.cache.EhCacheProvider</property>

```
现在4.1+:
```

        <property name="cache.use_query_cache">true</property>        
        <property name="cache.use_second_level_cache">true</property>        
        <property name="cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>

```
        
**7、与spring整合配置文**
hibernate4的sessionFactory配置 需要使用org.springframework.orm.` hibernate4 `.LocalSessionFactoryBean
hibernate4的transanctionManager配置需要使用org.springframework.orm.` hibernate4 `.HibernateTransactionManager
```

<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
<property name="dataSource" ref="dataSource"/>
...
</bean>


<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
<property name="sessionFactory" ref="sessionFactory"/>
</bean>

```

hibernate4必须配置为开启事务 否则 getCurrentSession()获取不到
```

<tx:advice id="baseServiceAdvice" transaction-manager="transactionManager">
   <tx:attributes>
      <!--hibernate4必须配置为开启事务 否则 getCurrentSession()获取不到--> 
      <tx:method name="get*" read-only="true" propagation="REQUIRED"/><!--之前是NOT_SUPPORT-->
      <tx:method name="find*" read-only="true" propagation="REQUIRED"/><!--之前是NOT_SUPPORT-->
      <tx:method name="save*" propagation="REQUIRED"/>
      <tx:method name="update*" propagation="REQUIRED"/>
      <tx:method name="remove*" propagation="REQUIRED"/>
      <tx:method name="add*" propagation="REQUIRED"/>
      <!--默认其他方法都是REQUIRED-->
      <tx:method name="*"/>
   </tx:attributes>
</tx:advice>

```
此处一定注意** 使用 hibernate4，在不使用OpenSessionInView模式时，在使用getCurrentSession()时会有如下问题：**
当有一个方法list 传播行为为Supports，当在另一个方法getPage()（无事务）调用list方法时会抛出org.hibernate.HibernateException: No Session found for current thread 异常。
这是因为getCurrentSession()在没有session的情况下不会自动创建一个，不知道这是不是Spring3.1实现的bug，欢迎大家讨论下。
因此最好的解决方案是使用REQUIRED的传播行为

Spring3.1集成Hibernate4不再需要HibernateDaoSupport和HibernateTemplate了，**直接使用原生API获取Session即可。**
##  附录：相关整合问题解决 

参考：http://blog.csdn.net/iaiti/article/details/9336703
**Hibernate4的改动较大只有spring3.1以上版本能够支持**，Spring3.1取消了HibernateTemplate，**因为Hibernate4的事务管理已经很好了**，不用Spring再扩展了。这里简单介绍了hibernate4相对于hibernate3配置时出现的错误，只列举了问题和解决方法，详细原理如果大家感兴趣还是去自己搜吧，网上很多。

1、Spring3.1去掉了HibernateDaoSupport类**。hibernate4需要通过getCurrentSession()获取session。**并且设置
    <prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate4.SpringSessionContext</prop>
    （在hibernate3的时候是thread和jta）。

2、缓存设置改为<prop key="hibernate.cache.provider_class">net.sf.ehcache.hibernate.EhCacheProvider</prop>
                    <prop key="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</prop>

3、Spring对hibernate的事务管理，不论是注解方式还是配置文件方式统一改为：
    <bean id="txManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager" >  
    <property name="sessionFactory"><ref bean="sessionFactory"/>
    </property> 
    </bean>

4、getCurrentSession()事务会自动关闭，所以在有所jsp页面查询数据都会关闭session。要想在jsp查询数据库需要加入：
    org.springframework.orm.hibernate4.support.OpenSessionInViewFilter过滤器。

    Hibernate分页出现 ResultSet may only be accessed in a forward direction 需要设置hibernate结果集滚动
     <prop key="jdbc.use_scrollable_resultset">false</prop>


--------------------------------------------------------------------
找到篇好文章,我之前遇到的问题都在这都能找到。其实出现这些问题的关键就是hibernate4和hibernate3出现了session管理的变动。
spring也作出相应的变动....
错误1：java.lang.NoClassDefFoundError: org/hibernate/cache/CacheProvider
原因：**spring的sessionfactory和transactionmanager与支持hibernate3时不同**。
解决：
 
```

<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
<property name="dataSource" ref="dataSource"/>
...
</bean>


<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
<property name="sessionFactory" ref="sessionFactory"/>
</bean>

```


错误2：java.lang.NoSuchMethodError:org.hibernate.SessionFactory.openSession()Lorg/hibernate/classic/Session
原因：hibernate4之后，spring31把HibernateDaoSupport去除，包括数据访问都不需要hibernatetemplate，这意味着dao需要改写，**直接使用hibernate的session和query接口。**
解决：为了改写dao，足足花了一天时间，然后一个个接口进行单元测试，这是蛋疼的一个主要原因。

错误3：nested exception is org.hibernate.HibernateException: No Session found for current thread
原因：发现一些bean无法获得当前session，**需要把之前一些方法的事务从NOT_SUPPORT提升到required,readonly=true**
见https://jira.springsource.org/browse/SPR-9020， http://www.iteye.com/topic/1120924
解决：
 
```

<tx:advice id="baseServiceAdvice" transaction-manager="transactionManager">
   <tx:attributes>
      <tx:method name="get*" read-only="true" propagation="REQUIRED"/><!--之前是NOT_SUPPORT-->
      <tx:method name="find*" read-only="true" propagation="REQUIRED"/><!--之前是NOT_SUPPORT-->
      <tx:method name="save*" propagation="REQUIRED"/>
      <tx:method name="update*" propagation="REQUIRED"/>
      <tx:method name="remove*" propagation="REQUIRED"/>
      <tx:method name="add*" propagation="REQUIRED"/>
      <!--默认其他方法都是REQUIRED-->
      <tx:method name="*"/>
   </tx:attributes>
</tx:advice>

```


错误4:与错误3报错类似，java.lang.NoSuchMethodError:org.hibernate.SessionFactory.openSession()Lorg/hibernate/classic/Session;
        at org.springframework.orm.hibernate3.SessionFactoryUtils.doGetSession(SessionFactoryUtils.java:324) [spring-orm-3.1.1.RELEASE.jar:3.1.1.RELEASE]
        at org.springframework.orm.hibernate3.SessionFactoryUtils.getSession(SessionFactoryUtils.java:202) [spring-orm-3.1.1.RELEASE.jar:3.1.1.RELEASE]
        at org.springframework.orm.hibernate3.support.OpenSessionInViewFilter

原因：因为**opensessioninview filter**的问题，如果你的配置还是hibernate3，需要改为hibernate4 
```

<filter>
    <filter-name>openSessionInViewFilter</filter-name>
   <filter-class>org.springframework.orm.hibernate4.support.OpenSessionInViewFilter</filter-class>
</filter>

```

--------------------------------------------------------------------
**由于Hibernate4已经完全可以实现事务了**， 与Spring3.1中的hibernatedao,hibernateTemplete等有冲突，所以Spring3.1里已经不提供Hibernatedaosupport,HibernateTemplete了，**只能用Hibernate原始的方式用session:**
        Session session = sessionFactory.openSession();
        Session session = sessionFactory.getCurrentSession();
在basedao里可以用注入的sessionFactory获取session.
注意, 配置事务的时候必须将父类baseServiceImpl也配上，要不然会出现错误：No Session found for currentthread, 以前是不需要的
SessionFactory.getCurrentSession()的后台实现是可拔插的。因此，引入了新的扩展接口 (org.hibernate.context.spi.CurrentSessionContext)和
新的配置参数(hibernate.current_session_context_class)，以便对什么是“当前session”的范围和上下文(scope and context)的定义进行拔插。

在Spring @Transactional声明式事务管理,”currentSession”的定义为: 当前被 Spring事务管理器 管理的Session,此时应配置:
` hibernate.current_session_context_class=org.springframework.orm.hibernate4.SpringSessionContext。 `

此处一定注意** 使用 hibernate4**，在不使用OpenSessionInView模式时，在使用getCurrentSession()时会有如下问题： 当有一个方法list 传播行为为Supports，当在另一个方法getPage()（无事务）调用list方法时会抛出org.hibernate.HibernateException: No Session found for current thread 异常。 这是因为**getCurrentSession()在没有session的情况下不会自动创建一个**，不知道这是不是Spring3.1实现的bug。 **因此最好的解决方案是使用REQUIRED的传播行为。**
