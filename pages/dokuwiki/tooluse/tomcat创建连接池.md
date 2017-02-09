title: tomcat创建连接池 

#  tomcat创建连接池 
JNDI——Java Naming and Directory Interface是一套提供naming和 directory功能的 API，
Java应用程式开发者透过使用 JNDI，在naming和 directory方面的应用上就有了共通的准则.
第一步：
将驱动程序(jar包)放到tomcat安装目录下的common\lib文件夹下
第二步:
在Tomcat的webapps目录随便创建一个工程目录，例如myjdbc。在myjdbc目录下创建META-INF目录，在此目录下创建一个context.xml文件，或者直接配置tomcat/conf/context.xml文件。里面的内容如下： 
```

<?xml version="1.0" encoding="UTF-8"?> 
<Context>
 <Resource name="jdbc/test" 
  auth="Container" 
  type="javax.sql.DataSource"
         maxActive="100" 
  maxIdle="30" 
  maxWait="10000"
         username="sa" password="" 
  driverClassName="com.mysql.jdbc.Driver"
         url="jdbc:mysql://localhost:3306/mydb"/>
 </Context>

```
 

附注如下：
Tomcat标准数据源资源工厂配置项如下：
* driverClassName - 所使用的JDBC驱动类全称。
* maxActive - 同一时刻可以自数据库连接池中被分配的最大活动实例数。
* maxIdle - 同一时刻数据库连接池中处于非活动状态的最大连接数。
* maxWait - 当连接池中没有可用连接时，连接池在抛出异常前将等待的最大时间，单位毫秒。
* password - 传给JDBC驱动的数据库密码。
* url - 传给JDBC驱动的连接URL。
* user - 传给JDBC驱动的数据库用户名。
* validationQuery - 一个SQL查询语句，用于在连接被返回给应用前的连接池验证。
* 如果指定了该属性，则必为至少返回一行记录的SQL SELECT语句。


jdbc/test是数据源的名称(随意写，要和web.xml文件中 <res-ref-name>jdbc/test</res-ref-name> 一样即可)，
其他的参数按照自己的实际情况进行修改，例如数据库的名称、账号、密码。

第三步:
在myjdbc目录下创建WEB-INF目录，创建**web.xml**文件，内容如下: 

```

<?xml version="1.0" encoding="UTF-8"?> 
<web-app xmlns="http://java.sun.com/xml/ns/j2ee" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" 
version="2.4"> 

    <resource-ref> 
        <description>DB Connection</description> 
        <res-ref-name>jdbc/test</res-ref-name> 
        <res-type>javax.sql.DataSource</res-type> 
        <res-auth>Container</res-auth> 
    </resource-ref> 
</web-app>

``` 

说明:
<resource-ref>
<descrtiption>引用资源说明</descrtiption>
<res-ref-name>引用资源的JNDI名</res-ref-name>
<res-type>引用资源的类名</res-type>
<res-auth>管理者（Container）</res-auth><!--Container－容器管理 Application－Web应用管理-->
</resource-ref>

```

 try
  {
   //初始化查找命名空间
   Context ctx = new InitialContext(); 
   //找到DataSource,对名称进行定位java:comp/env是必须加的,后面跟你的DataSource名
   DataSource ds = (DataSource)ctx.lookup("java:comp/env/jdbc/test");
   //取出连接
   Connection conn = ds.getConnection();
System.out.println("connection pool connected !!");   
  } catch (NamingException e) {
   System.out.println(e.getMessage());
  } catch (SQLException e) {
   e.printStackTrace();
  }finally
  {
   //注意不是关闭,是放回连接池.
   conn.close();
  }

```

参考：http://www.blogjava.net/supercrsky/articles/174931.html