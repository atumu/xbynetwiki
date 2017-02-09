title: chapter4_数据库编程 

#  Chapter4 数据库编程 
Created Thursday 26 March 2015
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080258.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080314.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080338.png)
#  SQL数据类型 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080350.png)

#  JDBC配置 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080403.png)

##  数据库URL 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080415.png)

##  驱动程序jar文件 

##  注册驱动器类 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080424.png)

##  连接到数据库 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080437.png)

###  对于Mysql数据库： 
说明文档：http://dev.mysql.com/doc/connector-j/en/connector-j-reference-configuration-properties.html
```

com.mysql.jdbc.Driver
jdbc:mysql://[host][,failoverhost...][:port]/[database] [?propertyName1][=propertyValue1][&propertyName2][=propertyValue2]...
If the host name is not specified, it defaults to 127.0.0.1. If the port is not specified, it defaults to 3306, the default port number for MySQL servers.

jdbc:mysql://[host:port],[host:port].../[database] [?propertyName1][=propertyValue1][&propertyName2][=propertyValue2]...
Here is a sample connection URL:
jdbc:mysql://localhost:3306/sakila?profileSQL=true

```
示例：
```

jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/corejava
jdbc.user=root
jdbc.password=123

	try(Connection conn=getConnection()){
			Statement stat=conn.createStatement();
			stat.executeUpdate("create table greetings (message char(20))");
			stat.executeUpdate("insert into greetings values('hello!')");
			try(ResultSet rs=stat.executeQuery("select * from greetings")){
				if(rs.next()){
					System.out.println(rs.getString(1));
					System.out.println(rs.getString("message"));
				}
			}
			stat.executeUpdate("drop table greetings");
		}
	}

	public static Connection getConnection() throws IOException, SQLException, ClassNotFoundException {
		Properties prop = new Properties();
		try (InputStream inStream = Files.newInputStream(Paths
				.get("db.properties"))) {
			prop.load(inStream);
		}
		Class.forName(prop.getProperty("jdbc.driver"));
		
		String url=prop.getProperty("jdbc.url");
		String user=prop.getProperty("jdbc.user");
		String password=prop.getProperty("jdbc.password");
		
		return DriverManager.getConnection(url,user,password);
	 }


```
#  执行SQL语句 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080537.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080550.png)
##  分析SQL异常 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080602.png)

##  组装数据库 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080614.png)

#  执行查询操作 

##  预编译语句 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080630.png)

##  读写LOB 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080644.png)

##  多结果集 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080659.png)

##  获取自动生成键 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080705.png)

#  可滚动和可更新的结果集 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080711.png)

##  可滚动结果集 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080727.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080742.png)
##  可更新结果集 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080802.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080817.png)
#  行集 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080857.png)
注意：在jdk7中添加了如下创建行集的方式：
RowSetFactory factory=RowSetProvider.newFactory();
CachedRowSet crs=factory.createCachedRowSet();
而在jdk7之前使用如下方式：
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080846.png)

##  被缓存的行集 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080932.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-080946.png)
#  元数据 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-081004.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-081011.png)
#  事务 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-081028.png)
##  保存点 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-081034.png)

##  批量更新 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-081057.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-081119.png)
##  类型对应 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-081126.png)

#  Web与企业应用中的连接管理 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-081154.png)

