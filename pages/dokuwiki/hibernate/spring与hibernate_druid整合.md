title: spring与hibernate_druid整合 

#  Spring与Hibernate+Druid整合 
参考：http://blog.csdn.net/zhanyuanlin/article/details/9983171 http://www.verydemo.com/demo_c146_i25712.html
```

	<properties>
		<org.hibernate.version>4.2.8.Final</org.hibernate.version>
		<org.springframework.version>3.2.5.RELEASE</org.springframework.version>
		<jackson.version>1.8.9</jackson.version>
		<jodd.version>3.5.2</jodd.version>
	</properties>
	<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.14</version>
		</dependency>
	<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>${org.hibernate.version}</version>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${org.hibernate.version}</version>
		</dependency>

		<dependency>
			<groupId>javassist</groupId>
			<artifactId>javassist</artifactId>
			<version>3.13.0.GA</version>
		</dependency>

		<dependency>
			<groupId>Mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.30</version>
		</dependency>
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-classic</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-core</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.9</version>
		</dependency>

```
```

	<!-- 为了在Spring中使用@PersistenceContext实现EntityManager注入 -->
	<bean
		class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor" />
 <!-- 数据源配置 -->  
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"  
        init-method="init" destroy-method="close">  
        <!-- 驱动名称 -->  
        <property name="DriverClassName" value="com.mysql.jdbc.Driver" />  
        <!-- JDBC连接串 -->  
        <property name="url"  
            value="jdbc:mysql://localhost:3306/spring" />  
        <!-- 数据库用户名称 -->  
        <property name="username" value="root" />  
        <!-- 数据库密码 -->  
        <property name="password" value="1234" />  
        <!-- 连接池最大使用连接数量 -->  
        <property name="maxActive" value="20" />  
        <!-- 初始化大小 -->  
        <property name="initialSize" value="5" />  
        <!-- 获取连接最大等待时间 -->  
        <property name="maxWait" value="60000" />  
        <!-- 连接池最小空闲 -->  
        <property name="minIdle" value="2" />  
        <!-- 逐出连接的检测时间间隔 -->  
        <property name="timeBetweenEvictionRunsMillis" value="3000" />  
        <!-- 最小逐出时间 -->  
        <property name="minEvictableIdleTimeMillis" value="300000" />  
        <!-- 测试有效用的SQL Query -->  
        <property name="validationQuery" value="SELECT 'x'" />  
        <!-- 连接空闲时测试是否有效 -->  
        <property name="testWhileIdle" value="true" />  
        <!-- 获取连接时测试是否有效 -->  
        <property name="testOnBorrow" value="false" />  
        <!-- 归还连接时是否测试有效 -->  
        <property name="testOnReturn" value="false" />  
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
        
    </bean>  
	<!--指定事物管理器 bean -->
	<bean id="txManager" class="org.springframework.orm.jpa.JpaTransactionManager">
		<property name="entityManagerFactory" ref="emf" />
	</bean>
	<bean
		class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />
	<!-- 开启Spring提供的基于注解的声明式事务管理 -->
	<tx:annotation-driven transaction-manager="txManager" proxy-target-class="true"/>

```

##  实战示列 
```

<!-- 连接池配置 -->
	<bean id="writeDataSource" class="com.alibaba.druid.pool.DruidDataSource"
		init-method="init" destroy-method="close">
		<!-- 基本属性 url、user、password -->
		<property name="url" value="${connection.url}" />
		<property name="username" value="${connection.username}" />
		<property name="password" value="${connection.password}" />
		<!-- 配置初始化大小、最小、最大 -->
		<property name="initialSize" value="${druid.initialSize}" />
		<property name="minIdle" value="${druid.minIdle}" />
		<property name="maxActive" value="${druid.maxActive}" />
		<!-- 配置获取连接等待超时的时间 -->
		<property name="maxWait" value="${druid.maxWait}" />
		<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
		<property name="timeBetweenEvictionRunsMillis" value="${druid.timeBetweenEvictionRunsMillis}" />
		<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
		<property name="minEvictableIdleTimeMillis" value="${druid.minEvictableIdleTimeMillis}" />

		<property name="validationQuery" value="${druid.validationQuery}" />
		<property name="testWhileIdle" value="${druid.testWhileIdle}" />
		<property name="testOnBorrow" value="${druid.testOnBorrow}" />
		<property name="testOnReturn" value="${druid.testOnReturn}" />

		<!-- 打开PSCache，并且指定每个连接上PSCache的大小 如果用Oracle，则把poolPreparedStatements配置为true，mysql可以配置为false。 -->
		<property name="poolPreparedStatements" value="${druid.poolPreparedStatements}" />
		<property name="maxPoolPreparedStatementPerConnectionSize"
			value="${druid.maxPoolPreparedStatementPerConnectionSize}" />

		<!-- 配置监控统计拦截的filters -->
		<property name="filters" value="${druid.filters}" />
	</bean>
	<bean id="readDataSource" class="com.alibaba.druid.pool.DruidDataSource"
		init-method="init" destroy-method="close">
		<!-- 基本属性 url、user、password -->
		<property name="url" value="${connection.url}" />
		<property name="username" value="${connection.username}" />
		<property name="password" value="${connection.password}" />
		<!-- 配置初始化大小、最小、最大 -->
		<property name="initialSize" value="${druid.initialSize}" />
		<property name="minIdle" value="${druid.minIdle}" />
		<property name="maxActive" value="${druid.maxActive}" />
		<!-- 配置获取连接等待超时的时间 -->
		<property name="maxWait" value="${druid.maxWait}" />
		<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
		<property name="timeBetweenEvictionRunsMillis" value="${druid.timeBetweenEvictionRunsMillis}" />
		<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
		<property name="minEvictableIdleTimeMillis" value="${druid.minEvictableIdleTimeMillis}" />

		<property name="validationQuery" value="${druid.validationQuery}" />
		<property name="testWhileIdle" value="${druid.testWhileIdle}" />
		<property name="testOnBorrow" value="${druid.testOnBorrow}" />
		<property name="testOnReturn" value="${druid.testOnReturn}" />

		<!-- 打开PSCache，并且指定每个连接上PSCache的大小 如果用Oracle，则把poolPreparedStatements配置为true，mysql可以配置为false。 -->
		<property name="poolPreparedStatements" value="${druid.poolPreparedStatements}" />
		<property name="maxPoolPreparedStatementPerConnectionSize"
			value="${druid.maxPoolPreparedStatementPerConnectionSize}" />

		<!-- 配置监控统计拦截的filters -->
		<property name="filters" value="${druid.filters}" />
	</bean>
	<bean id="dataSource" class="com.css.sdc.framework.datasource.MultiDataSource">
		<property name="targetDataSources">
			<map key-type="java.lang.String">
				<entry key="write" value-ref="writeDataSource" />
				<entry key="read" value-ref="readDataSource" />
			</map>

		</property>
		<property name="defaultTargetDataSource" ref="writeDataSource" />
	</bean>
	<!-- Jpa Entity Manager 配置 -->
	<bean id="entityManagerFactory"
		class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="jpaVendorAdapter">
			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
				<property name="showSql" value="false" />
				<property name="generateDdl" value="false" />
				<property name="database" value="MYSQL" />
                          	<property name="databasePlatform" value="org.hibernate.dialect.MySQL5InnoDBDialect" />
			</bean>
		</property>
		<!--Weblogic/Jboss这些自带JPA支持的应用服务器有时候会扫描persistence.xml，因此彻底删除掉这个文件需要加，否则会报错 -->
		<property name="packagesToScan" value="com.css.**.entity" />
		<property name="jpaProperties">
			<props>
				<!-- 命名规则 My_NAME->MyName java驼峰变db下划线命名-->
				<prop key="hibernate.ejb.naming_strategy">org.hibernate.cfg.ImprovedNamingStrategy</prop>
				<prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate4.SpringSessionContext</prop>
			</props>
		</property>
	</bean>
	<!-- Spring Data Jpa配置 -->
	<jpa:repositories base-package="com.css.**.dao"
		transaction-manager-ref="transactionManager"
		entity-manager-factory-ref="entityManagerFactory"
		factory-class="com.css.sdc.framework.repository.SDCRepositoryFactoryBean" />
	<!-- Jpa 事务配置 -->
	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
		<property name="entityManagerFactory" ref="entityManagerFactory" />
	</bean>
	<!-- 使用annotation定义事务 -->
	<tx:annotation-driven transaction-manager="transactionManager"  proxy-target-class="true"/>

```
**datasource.propertis:**
```

application-root.xml
#dataSource configure
#connection.url=jdbc:oracle:thin:@10.20.20.9:1521:wyjg
#connection.url=jdbc:oracle:thin:@localhost:1521:orcl
connection.url=jdbc:mysql://10.1.100.162/wyjg
connection.username=wyjg
connection.password=wyjg

#druid datasource
druid.initialSize=10
druid.minIdle=10
druid.maxActive=30
druid.maxWait=60000
druid.timeBetweenEvictionRunsMillis=60000
druid.minEvictableIdleTimeMillis=300000
druid.validationQuery=SELECT 1 FROM DUAL
druid.testWhileIdle=true
druid.testOnBorrow=false
druid.testOnReturn=false
druid.poolPreparedStatements=true
druid.maxPoolPreparedStatementPerConnectionSize=20
druid.filters=wall,stat

#shiro
#password.algorithmName=md5
#password.hashIterations=2

```