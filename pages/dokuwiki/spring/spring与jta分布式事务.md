title: spring与jta分布式事务 

#  Spring与JTA分布式事务 
参考：http://364434006.iteye.com/blog/981819 http://www.ibm.com/developerworks/cn/java/j-lo-jta/ 
http://www.cnblogs.com/wuyifu/p/3474756.html   http://blog.csdn.net/z69183787/article/details/19829927
使用SpringJTA进行分布式事务管理,需要引入第三方UserTransaction。例如:JOTM、Atomikos
spring3.0已经不再支持jtom了，不过我们可以用第三方开源软件atomikos(http://www.atomikos.com/)来实现． 
**Atomikos**是一个公司的名字，AtomikosTransactionsEssentials是其开源的分布式事务软件包，而ExtremeTransactions是商业的分布式事务软件包。TransactionsEssentials是基于apache-license的，是JTA/XA的开源实现，支持Java Application和J2EE应用。
分布式事务是指操作多个数据库之间的事务，spring的org.springframework.transaction.jta.JtaTransactionManager，提供了分布式事务支持。
 在tomcat下，是没有分布式事务的，不过可以借助于第三方软件jotm（Java Open Transaction Manager ）和AtomikosTransactionsEssentials实现，
在spring中分布式事务是通过jta（jotm，atomikos）来进行实现。 
##  JTA分布式事务管理配置 

```

  <!-- spring atomikos 配置　开始-->
 <!-- mysql数据源 -->
    <bean id="mysqlDS" class="com.atomikos.jdbc.AtomikosDataSourceBean"
        init-method="init" destroy-method="close">
        <description>mysql xa datasource</description>
        <property name="uniqueResourceName">
            <value>mysql_ds</value>
        </property>
        <property name="xaDataSourceClassName" value="com.mysql.jdbc.jdbc2.optional.MysqlXADataSource" />
        <property name="xaProperties">
            <props>
                <prop key="user">userName</prop>
                <prop key="password">password</prop>
                <prop key="URL">jdbc:mysql://127.0.0.1:3306/dataBaseName?autoReconnect=true</prop>
            </props>
        </property>
        <property name="poolSize">  
            <value>2</value>  
        </property>  
        <property name="maxPoolSize">  
            <value>30</value>  
        </property>  
    </bean>  

 <!-- oracle数据源 -->
    <bean id="oracleDS" class="com.atomikos.jdbc.AtomikosDataSourceBean"
        init-method="init" destroy-method="close">
        <description>oracle xa datasource</description>
        <property name="uniqueResourceName">
            <value>oracle_ds</value>
        </property>
        <property name="xaDataSourceClassName">
            <value>oracle.jdbc.xa.client.OracleXADataSource</value>
        </property>
        <property name="xaProperties">
            <props>
                <prop key="user">userName</prop>
                <prop key="password">password</prop>
                <prop key="URL">jdbc\:oracle\:thin\:@127.0.0.1\:1521\:dataBaseName</prop>
            </props>
        </property>
        <!-- 连接池里面连接的个数? --> 
        <property name="poolSize" value="3"/>
    </bean>

 <!-- atomikos事务管理器 -->
    <bean id="atomikosTransactionManager" class="com.atomikos.icatch.jta.UserTransactionManager"
        init-method="init" destroy-method="close">
        <description>UserTransactionManager</description>
        <property name="forceShutdown">
            <value>true</value>
        </property>
    </bean>

    <bean id="atomikosUserTransaction" class="com.atomikos.icatch.jta.UserTransactionImp">
        <property name="transactionTimeout" value="300" />
    </bean>

    <!-- spring 事务管理器 -->
    <bean id="springTransactionManager"
        class="org.springframework.transaction.jta.JtaTransactionManager">
        <property name="transactionManager">
            <ref bean="atomikosTransactionManager" />
        </property>
        <property name="userTransaction">
            <ref bean="atomikosUserTransaction" />
        </property>
    </bean>
<tx:annotation-driven transaction-manager="springTransactionManager"/> 

    <!-- spring　事务模板 我在项目当中用的是编程式事务-->
    <bean id="transactionTemplate"
        class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager">
            <ref bean="springTransactionManager" />
        </property>
    </bean>


    <bean id="simpleJdbcTemplate" class="org.springframework.jdbc.core.simple.SimpleJdbcTemplate">
        <constructor-arg>
            <ref bean="mysqlDS" />
        </constructor-arg>
    </bean>
    
    <bean id="simplejdbcTemplateOra" class="org.springframework.jdbc.core.simple.SimpleJdbcTemplate">
        <constructor-arg>
            <ref bean="oracleDS" />
        </constructor-arg>
    </bean>

    <!-- spring atomikos 配置　结束-->
  <!-- 
配置SessionFactory:
      <bean id="oracleSessionFactory" class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">  
    <property name="dataSource" ref="ds1"></property>  
    <property name="packagesToScan" value="org/liny/entity"/>  
    <property name="hibernateProperties">  
        <props>  
            <prop key="hibernate.show_sql">true</prop>  
            <prop key="hibernate.format_sql">true</prop>  
            <prop key="hibernate.dialect">org.hibernate.dialect.Oracle10gDialect</prop>  
        </props>  
    </property>  
</bean>  
-->

```
##  JTA分布式事务管理 
事务是计算机应用中不可或缺的组件模型，它保证了用户操作的原子性 ( Atomicity )、一致性 ( Consistency )、隔离性 ( Isolation ) 和持久性 ( Durabilily )。
J2EE 事务处理方式
**1. 本地事务**：紧密依赖于底层资源管理器（例如数据库连接 )，事务处理局限在当前事务资源内。此种事务处理方式不存在对应用服务器的依赖，因而部署灵活却无法支持多数据源的分布式事务。在数据库连接中使用本地事务示例如下：
```

public void transferAccount() { 
		 Connection conn = null; 
		 Statement stmt = null; 
		 try{ 
			 conn = getDataSource().getConnection(); 
			 // 将自动提交设置为 false，
			 //若设置为 true 则数据库将会把每一次数据更新认定为一个事务并自动提交
			 conn.setAutoCommit(false);
			
			 stmt = conn.createStatement(); 
			 // 将 A 账户中的金额减少 500 
			 stmt.execute("\
             update t_account set amount = amount - 500 where account_id = 'A'");
			 // 将 B 账户中的金额增加 500 
			 stmt.execute("\
             update t_account set amount = amount + 500 where account_id = 'B'");
			
			 // 提交事务
		     conn.commit();
			 // 事务提交：转账的两步操作同时成功
		 } catch(SQLException sqle){ 			
			 try{ 
				 // 发生异常，回滚在本事务中的操做
                conn.rollback();
				 // 事务回滚：转账的两步操作完全撤销
                 stmt.close(); 
                 conn.close(); 
			 }catch(Exception ignore){ 
				
			 } 
			 sqle.printStackTrace(); 
		 } 
	 }

```
**2. 分布式事务处理 :** Java 事务编程接口（JTA：Java Transaction API）和 Java 事务服务 (JTS；Java Transaction Service) 为 J2EE 平台提供了分布式事务服务。
分布式事务（Distributed Transaction）包括**事务管理器（Transaction Manager）和一个或多个支持 XA 协议的资源管理器 ( Resource Manager )**。我们可以将资源管理器看做任意类型的持久化数据存储；事务管理器承担着所有事务参与单元的协调与控制。**JTA 事务有效的屏蔽了底层事务资源，使应用可以以透明的方式参入到事务处理中；但是与本地事务相比，XA 协议的系统开销大，**在系统开发过程中应慎重考虑是否确实需要分布式事务。若确实需要分布式事务以协调多个事务资源，则应实现和配置所支持 XA 协议的事务资源，如 JMS、JDBC 数据库连接池等。
开发人员使用开发人员接口，实现应用程序对全局事务的支持；各提供商（数据库，JMS 等）依据提供商接口的规范提供事务资源管理功能；事务管理器（ TransactionManager ）将应用对分布式事务的使用映射到实际的事务资源并在事务资源间进行协调与控制。 下面，本文将对包括`  UserTransaction、Transaction 和 TransactionManager ` 在内的三个主要接口以及其定义的方法进行介绍。
面向开发人员的接口为 UserTransaction （使用方法如上例所示），开发人员通常只使用此接口实现 JTA 事务管理，其定义了如下的方法：
  * begin()- 开始一个分布式事务，（在后台 TransactionManager 会创建一个 Transaction 事务对象并把此对象通过 ThreadLocale 关联到当前线程上 )
  * commit()- 提交事务（在后台 TransactionManager 会从当前线程下取出事务对象并把此对象所代表的事务提交）
  * rollback()- 回滚事务（在后台 TransactionManager 会从当前线程下取出事务对象并把此对象所代表的事务回滚）
  * getStatus()- 返回关联到当前线程的分布式事务的状态 (Status 对象里边定义了所有的事务状态，感兴趣的读者可以参考 API 文档 )
  * setRollbackOnly()- 标识关联到当前线程的分布式事务将被回滚

使用 JTA 处理事务的示例如下（**注意：connA 和 connB 是来自不同数据库的连接**）
```

public void transferAccount() { 
		
		 UserTransaction userTx = null; 
		 Connection connA = null; 
		 Statement stmtA = null; 
				
		 Connection connB = null; 
		 Statement stmtB = null; 
    
		 try{ 
		       // 获得 Transaction 管理对象
			 userTx = (UserTransaction)getContext().lookup("\
			       java:comp/UserTransaction"); 
			 // 从数据库 A 中取得数据库连接
			 connA = getDataSourceA().getConnection(); 
			
			 // 从数据库 B 中取得数据库连接
			 connB = getDataSourceB().getConnection(); 
      
                        // 启动事务
			 userTx.begin();
			
			 // 将 A 账户中的金额减少 500 
			 stmtA = connA.createStatement(); 
			 stmtA.execute("
            update t_account set amount = amount - 500 where account_id = 'A'");
			
			 // 将 B 账户中的金额增加 500 
			 stmtB = connB.createStatement(); 
			 stmtB.execute("\
             update t_account set amount = amount + 500 where account_id = 'B'");
			
			 // 提交事务
			 userTx.commit();
			 // 事务提交：转账的两步操作同时成功（数据库 A 和数据库 B 中的数据被同时更新）
		 } catch(SQLException sqle){ 
			
			 try{ 
		  	       // 发生异常，回滚在本事务中的操纵
                  userTx.rollback();
				 // 事务回滚：转账的两步操作完全撤销 
				 //( 数据库 A 和数据库 B 中的数据更新被同时撤销）
				
				 stmt.close(); 
                 conn.close(); 
				 ... 
			 }catch(Exception ignore){ 
				
			 } 
			 sqle.printStackTrace(); 
			
		 } catch(Exception ne){ 
			 e.printStackTrace(); 
		 } 
	 }

```