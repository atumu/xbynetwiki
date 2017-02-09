title: spring与hibernate整合 

#  Spring与Hibernate JPA整合 
参考：http://www.cnblogs.com/chenmfly/p/4295578.html
http://blog.csdn.net/lengyueqiufeng/article/details/5900735
版本：Spring4.x,Hibernate 4.x,JPA2.x , C3P0 0.9.12

整合spring和hibernate JPA需要五个要素，分别包括
  * 持久化的实体类，
  * 数据源，采用C3P0
  * EntityManagerFactory，
  * TransactionManager和
  * 持久化操作的DAO类。
##  web.xml配置 
```

<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
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
	<servlet>
		<servlet-name>springmvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>

	</servlet>
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	<!-- 解决乱码问题-->
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
 
<listener>
    <listener-class>org.apache.tiles.extras.complete.CompleteAutoloadTilesListener</listener-class>
</listener>
<session-config>
  		<session-timeout>30</session-timeout>
</session-config>
</web-app>

```
##  第一种方式 
保留classpath下的META-INF/persistence.xml.
persistence.xml还是和原来一样：
```

<persistence xmlns="http://java.sun.com/xml/ns/persistence"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             
        xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
        version="2.0">
	<!--事务类型：local的还是global(JTA)的事务 -->  
   <persistence-unit name="mysqldb" transaction-type="RESOURCE_LOCAL"> <!-- 这里新加了transaction-type="RESOURCE_LOCAL" -->
       <!-- <provider>org.hibernate.ejb.HibernatePersistence</provider>   -->
       <!-- The provider only needs to be set if you use several JPA providers
       <provider>org.hibernate.ejb.HibernatePersistence</provider>
        -->
        <!-- A managed datasource provided by the application server, in this
            case the microcontainer, see caveatemptor-beans.xml definition.
       
       <jta-data-source>java:/caveatemptorTestingDatasource</jta-data-source>
      -->
       <!-- This is required to be spec compliant, Hibernate however supports
            auto-detection even in JSE.
       <class>hello.Message</class>
        -->
	<!--   厂商专有属性(可选)   -->  
      <properties>
          <!-- Scan for annotated classes and Hibernate mapping XML files -->
          <property name="hibernate.archive.autodetection" value="class, hbm"/>

        
          <property name="hibernate.show_sql" value="true"/>
          <property name="hibernate.format_sql" value="true"/>
          <property name="use_sql_comments" value="true"/>
       

          <property name="hibernate.connection.driver_class"
                    value="com.mysql.jdbc.Driver"/>
          <property name="hibernate.connection.url"
                    value="jdbc:mysql://localhost/mydb"/>
          <property name="hibernate.connection.username"
                    value="root"/>
	  <property name="hibernate.connection.password" value="1234"/>
         <!-- hibernate的c3p0连接池配置（需要jar包：c3p0-0.9.0.4.jar）
	 <property name="hibernate.connection.provider_class" value="org.hibernate.connection.C3P0ConnectionProvider"/> 可以省略。会自动检测。
	--> 
          <property name="hibernate.c3p0.min_size"
                    value="5"/>
          <property name="hibernate.c3p0.max_size"
                    value="20"/>
         <!-- 获得连接的超时时间为3秒,如果超过这个时间,会抛出异常，单位毫秒 -->       
          <property name="hibernate.c3p0.timeout"
                    value="3000"/>
         <!--最大空闲时间,300秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->     
        <property name="c3p0.maxIdleTime" value="300"/>   
          <!-- 最大的PreparedStatement的数量 --> 
          <property name="hibernate.c3p0.max_statements"
                    value="50"/>
         <!-- 每隔3000秒检查连接池里的空闲连接 ，单位是秒。可以解决mysql8小时问题-->   
          <property name="hibernate.c3p0.idle_test_period"
                    value="3000"/>
<!-- 当连接池里面的连接用完的时候，C3P0一下获取的新的连接数 
        <property name="c3p0.acquire_increment" value="3"/>   
-->    
          <property name="hibernate.dialect"
                    value="org.hibernate.dialect.MySQL5InnoDBDialect"/>
<!--Automatically validates or exports schema DDL to the database when the SessionFactory is created. With create-drop, the database schema will be dropped when the SessionFactory is closed explicitly.     e.g. validate | update | create | create-drop
validate 加载hibernate时，验证创建数据库表结构
create 每次加载hibernate，重新创建数据库表结构，这就是导致数据库表数据丢失的原因。
create-drop 加载hibernate时创建，退出是删除表结构
update 加载hibernate自动更新数据库结构,以前的行不会被删除。
以上4个属性对同一配置文件下所用有的映射表都起作用
总结：
1.请慎重使用此参数，没必要就不要随便用。
2.如果发现数据库表丢失，请检查hibernate.hbm2ddl.auto的配置
-->
           <property name="hibernate.hbm2ddl.auto" value="update"/>
      </properties>
   </persistence-unit>

</persistence>

```
spring配置文件：
```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:jpa="http://www.springframework.org/schema/data/jpa"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xmlns:tx="http://www.springframework.org/schema/tx"
xmlns:util="http://www.springframework.org/schema/util"
 xmlns:p="http://www.springframework.org/schema/p"
 xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd
http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.2.xsd
http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.2.xsd
http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-3.2.xsd">
  <!-- 指定需要扫描的包并注册为bean，该元素可以有多个-->
    <context:component-scan base-package="com.habuma" />
  <aop:aspectj-autoproxy proxy-target-class="true" />
  <!-- 为了在Spring中使用@PersistenceContext实现EntityManager注入-->
  <bean class=
  "org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>
   <!--指定JPA厂商适配器bean -->
  <bean id="jpaVendorAdapter" 
      class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
    <property name="database" value="MYSQL" />
    <property name="showSql" value="true"/>
    <property name="formatSql" value="true"/>
    <property name="generateDdl" value="false"/>
    <property name="databasePlatform" 
              value="org.hibernate.dialect.MySQL5InnoDBDialect" />
  </bean>
 <!--指定EntityManagerFactory bean--> 
  <bean id="emf" class=
      "org.springframework.orm.jpa.LocalEntityManagerFactoryBean">
    <property name="persistenceUnitName" value="mysqldb"></property>  
  </bean>
  <!--指定事物管理器 bean-->
   <bean id="txManager"
      class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="emf" />
  </bean>
  <bean class=
 "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"
/>
  <!-- 开启Spring提供的基于注解的声明式事务管理 -->  
    <tx:annotation-driven transaction-manager="txManager" proxy-target-class="true"/>
    <!-- 任务调度 -->
  <task:annotation-driven/>
</beans>

```
##  第二种方式： 
persistence.xml 这个文件可以省略了，**全部配置在applicationContext.xml** 里面：
```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:jpa="http://www.springframework.org/schema/data/jpa"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xmlns:tx="http://www.springframework.org/schema/tx"
xmlns:util="http://www.springframework.org/schema/util"
 xmlns:p="http://www.springframework.org/schema/p"
 xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd
http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.2.xsd
http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.2.xsd
http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-3.2.xsd">
  <!-- 指定需要扫描的包并注册为bean，该元素可以有多个-->
    <context:component-scan base-package="com.habuma" />
  <aop:aspectj-autoproxy proxy-target-class="true" />
  <!-- 引入配置文件 -->
   <context:property-placeholder location="classpath:common.properties,classpath:datasource.properties"/>
  
  <!-- 为了在Spring中使用@PersistenceContext实现EntityManager注入-->
  <bean class=
  "org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>
   <!--数据源配置C3P0 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"  
       init-method="init" destroy-method="close">  
        <property name="driverClass" value="com.mysql.jdbc.Driver">  
        </property>  
        <property name="jdbcUrl" value="jdbc:mysql://localhost/mydb"></property>  
        <property name="user" value="HIBERNATE" />  
        <property name="password" value="HIBERNATE" />  
  
        <property name="minPoolSize" value="10" />  
        <property name="maxPoolSize" value="100" />  
        <property name="maxIdleTime" value="1800" />  
        <property name="acquireIncrement" value="3" />  
        <property name="maxStatements" value="1000" />  
        <property name="initialPoolSize" value="10" />  
        <property name="idleConnectionTestPeriod" value="600" />  
        <property name="acquireRetryAttempts" value="30" />  
        <property name="breakAfterAcquireFailure" value="true" />  
        <property name="testConnectionOnCheckout" value="false" />  
    </bean>  
  
  <!-- JPA实体工厂配置 -->  
    <bean id="emf"  
        class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">  
        <property name="dataSource" ref="dataSource" />  
        <!-- 扫描实体路径 -->  
        <property name="packagesToScan" value="net.xby1993.springmvc.entity" />  
        <property name="jpaVendorAdapter">  
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">   
                <property name="database" value="MYSQL" />
    			<property name="showSql" value="true"/>
    			<property name="generateDdl" value="false"/>
    			<property name="databasePlatform" value="org.hibernate.dialect.MySQL5InnoDBDialect" />
            </bean>  
        </property> 
        <property name="jpaProperties">
			<props>
				<!-- 命名规则 My_NAME->MyName java驼峰变db下划线命名-->
				<prop key="hibernate.ejb.naming_strategy">org.hibernate.cfg.ImprovedNamingStrategy</prop>
				<prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate4.SpringSessionContext</prop>
			</props>
		</property> 
        
    </bean>  
  <!--指定事物管理器 bean-->
   <bean id="txManager"
      class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="emf" />
  </bean>
  <bean class=
 "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"
/>
  <!-- 开启Spring提供的基于注解的声明式事务管理 -->  
   <tx:annotation-driven transaction-manager="txManager" proxy-target-class="true"/>
    <!-- 任务调度 -->
<task:annotation-driven executor="myExecutor" scheduler="myScheduler"/>
<task:executor id="myExecutor" pool-size="5"/>
<task:scheduler id="myScheduler" pool-size="10"/>
</beans>      

```       
然后我们就可以这样编写DAO了：` @Repository @Transactional @PersistenceContext `
```

编写基于JPA的POJO:
@Repository("spitterDao")//表明这是一个数据库处理bean
@Transactional//表明这个dao类中所有数据库处理方法都在事物上下文中执行
public class JpaSpitterDao implements SpitterDao {
  private static final String RECENT_SPITTLES = 
      "SELECT s FROM Spittle s";
  private static final String ALL_SPITTERS =
      "SELECT s FROM Spitter s";
  private static final String SPITTER_FOR_USERNAME = 
      "SELECT s FROM Spitter s WHERE s.username = :username";
  private static final String SPITTLES_BY_USERNAME = 
      "SELECT s FROM Spittle s WHERE s.spitter.username = :username";

  @PersistenceContext  //配合前面的配置文件，它会被spring自动注入。
  private EntityManager em; //<co id="co_injectEM"/>

```
```

  //启动Spring容器  
ApplicationContext beans=new ClassPathXmlApplicationContext("applicationContext.xml");

```  