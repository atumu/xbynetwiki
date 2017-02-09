title: tomcat中配置使用c3p0 

#  tomcat中配置使用c3p0 

tomcat中配置数据源的方式有以下几种：
1.配置全局数据源：在conf/server.xml中的<GlobalNamingResources>下增加
```

<Resource   name="jdbc/myDB"
              auth="Container"
               description="DB Connection"    
               driverClass="com.mysql.jdbc.Driver"
               user="test"
               password="ready2go"
               type="com.mchange.v2.c3p0.ComboPooledDataSource"
             jdbcUrl="jdbc:mysql://localhost:3306/test
               acquireIncrement="1"
               maxPoolSize="50"
               minPoolSize="10"
               initialPoolSize="10"
               automaticTestTable="C3P0TestTable"
               idleConnectionTestPeriod="60"
               maxIdleTime="60"
               checkoutTimeout="30000"/>
  </code>
  然后在config/context.xm文件中<context>下增加： 
<ResourceLink global="jdbc/mydb" name="jdbc/myDB" type="javax.sql.DataSource"/>
` l或者webapp下的META-INF中的context.xml中的<context>元素下增加： `
<ResourceLink name="jdbc/myDB" global="jdbc/myDB" type="javax.sql.DataSource"/>

然后在webapp下的web.xml中添加：
<resource-ref>
<res-ref-name>jdbc/myDB</res-ref-name>
<res-type>javax.sql.DataSource</res-type>
<res-auth>Container</res-auth>
</resource-ref>
2.配置局部数据源，` 只要在META-INF下的context.xml中添加： `
<code>
<Resource   name="jdbc/myDB"
              auth="Container"
               description="DB Connection"    
               driverClass="com.mysql.jdbc.Driver"
               user="test"
               password="ready2go"
               type="com.mchange.v2.c3p0.ComboPooledDataSource"
             jdbcUrl="jdbc:mysql://localhost:3306/test
               acquireIncrement="1"
               maxPoolSize="50"
               minPoolSize="10"
               initialPoolSize="10"
               automaticTestTable="C3P0TestTable"
               idleConnectionTestPeriod="60"
               maxIdleTime="60"
               checkoutTimeout="30000"/>

```
其他照旧。

3.配置局部数据源的另外一种方式：
conf/Catalina/localhost/myapp.xml中的context元素下添加上面的配置。

二、配置好之后就是使用，使用很简单。就是使用常规的DataSource接口进行操作。
InitialContext ic = new InitialContext();
DataSource ds = (DataSource) ic.lookup("java:comp/env/jdbc/pooledDS");
参考：C3P0官方文档
http://my.oschina.net/artshell/blog/199669
http://qiufubin.blog.sohu.com/55457392.html