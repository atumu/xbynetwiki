title: hibernate_jpa_c3p0配置 

#  Hibernate+JPA+C3P0配置： 
```

<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>3.6.10.Final</version>
</dependency>
<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${org.hibernate.version}</version>
		</dependency>
<!-- Hibernate c3p0 connection pool -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-c3p0</artifactId>
			<version>${org.hibernate.version}</version>
		</dependency>
<dependency>
	<groupId>javassist</groupId>
	<artifactId>javassist</artifactId>
	<version>3.13.0.GA</version>
</dependency>

<!-- end-->
		<dependency>
			<groupId>c3p0</groupId>
			<artifactId>c3p0</artifactId>
			<version>0.9.1.2</version>
		</dependency>
		<dependency>
			<groupId>Mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.30</version>
		</dependency>

```
	
版本：Hibernate3.x+JPA2.x+C3P0 0.9.1.2
` persistence.xml必须位于classpath下的META-INF/目录下，而不是默认的META-INF `
```

<persistence xmlns="http://java.sun.com/xml/ns/persistence"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
        version="2.0">

   <persistence-unit name="helloworld">

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