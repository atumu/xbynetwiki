title: apache_shiro_ini配置 

#  apache Shiro Ini配置 
INI配置文件是一种key/value的键值对配置，**提供了分类的概念**，每一个类中的key不可重复。在这个示例中我们使用一个INI文件来配置Shiro SecurityManager，首先，在pom.xml同目录中创建一个src/main/resources子目录，在该子目录中创建一个shiro.ini文件，内容如下：
```

# # ## =
# Shiro INI 配置
# # ## =

[main]
# 对象和它们的属性在这里定义
# 例如 SecurityManager, Realms 等。

[users]
# 用户在这里定义，如果只是一小部分用户。（实际使用中，使用这种配置显然不合适）

[roles]
# 角色在这里定义，（实际应用中也不会在这里定义角色，一般都是存储于数据库中）


[urls]
# web系统中，基于url的权限配置，web章节会介绍。

```

##  根对象SecurityManager 
从之前的Shiro架构图可以看出，Shiro是从根对象SecurityManager进行身份验证和授权的；也就是所有操作都是自它开始的，**这个对象是线程安全且真个应用只需要一个即可，**因此Shiro提供了SecurityUtils让我们绑定它为全局的，方便后续操作。
 
因为Shiro的类都是POJO的，因此都很容易放到任何IoC容器管理。但是和一般的IoC容器的区别在于，**Shiro从根对象securityManager开始导航**；Shiro支持的依赖注入：public空参构造器对象的创建、setter依赖注入。
 
**1、纯Java代码写法**（com.github.zhangkaitao.shiro.chapter4.NonConfigurationCreateTest）： 
```

DefaultSecurityManager securityManager = new DefaultSecurityManager();  
//设置authenticator  
ModularRealmAuthenticator authenticator = new ModularRealmAuthenticator();  
authenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());  
securityManager.setAuthenticator(authenticator);  
  
//设置authorizer  
ModularRealmAuthorizer authorizer = new ModularRealmAuthorizer();  
authorizer.setPermissionResolver(new WildcardPermissionResolver());  
securityManager.setAuthorizer(authorizer);  
  
//设置Realm  
DruidDataSource ds = new DruidDataSource();  
ds.setDriverClassName("com.mysql.jdbc.Driver");  
ds.setUrl("jdbc:mysql://localhost:3306/shiro");  
ds.setUsername("root");  
ds.setPassword("");  
  
JdbcRealm jdbcRealm = new JdbcRealm();  
jdbcRealm.setDataSource(ds);  
jdbcRealm.setPermissionsLookupEnabled(true);  
securityManager.setRealms(Arrays.asList((Realm) jdbcRealm));  
  
//将SecurityManager设置到SecurityUtils 方便全局使用  
SecurityUtils.setSecurityManager(securityManager);  
  
Subject subject = SecurityUtils.getSubject();  
UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");  
subject.login(token);  
Assert.assertTrue(subject.isAuthenticated()); 

``` 
  
**2.1、等价的INI配置**（shiro-config.ini） 
```

[main]  
#authenticator  
authenticator=org.apache.shiro.authc.pam.ModularRealmAuthenticator  
authenticationStrategy=org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy  
authenticator.authenticationStrategy=$authenticationStrategy  
securityManager.authenticator=$authenticator  
  
#authorizer  
authorizer=org.apache.shiro.authz.ModularRealmAuthorizer  
permissionResolver=org.apache.shiro.authz.permission.WildcardPermissionResolver  
authorizer.permissionResolver=$permissionResolver  
securityManager.authorizer=$authorizer  
  
#realm  
dataSource=com.alibaba.druid.pool.DruidDataSource  
dataSource.driverClassName=com.mysql.jdbc.Driver  
dataSource.url=jdbc:mysql://localhost:3306/shiro  
dataSource.username=root  
#dataSource.password=  
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm  
jdbcRealm.dataSource=$dataSource  
jdbcRealm.permissionsLookupEnabled=true  
securityManager.realms=$jdbcRealm  

``` 
即使没接触过IoC容器的知识，如上配置也是很容易理解的：
1、对象名=全限定类名  相对于调用public无参构造器创建对象
2、对象名.属性名=值    相当于调用setter方法设置常量值
3、对象名.属性名=$对象引用    相当于调用setter方法设置对象引用
 
2.2、Java代码（com.github.zhangkaitao.shiro.chapter4.ConfigurationCreateTest） 
```

Factory<org.apache.shiro.mgt.SecurityManager> factory =  
         new IniSecurityManagerFactory("classpath:shiro-config.ini");  
  
org.apache.shiro.mgt.SecurityManager securityManager = factory.getInstance();  
  
//将SecurityManager设置到SecurityUtils 方便全局使用  
SecurityUtils.setSecurityManager(securityManager);  
Subject subject = SecurityUtils.getSubject();  
UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");  
subject.login(token);  
  
Assert.assertTrue(subject.isAuthenticated());  

``` 
2、IniSecurityManagerFactory是创建securityManager的工厂，其需要一个ini配置文件路径，**其支持“classpath:”（类路径）、“file:”（文件系统）、“url:”（网络）三种路径格式**，默认是文件系统；
3、接着获取SecuriyManager实例，后续步骤和之前的一样。

##  INI配置 
ini配置文件类似于Java中的properties（key=value），不过提供了将key/value分类的特性，key是每个部分不重复即可，而不是整个配置文件。如下是INI配置分类： 
```

[main]  
#提供了对根对象securityManager及其依赖的配置  
securityManager=org.apache.shiro.mgt.DefaultSecurityManager  
…………  
securityManager.realms=$jdbcRealm  
  
[users]  
#提供了对用户/密码及其角色的配置，用户名=密码，角色1，角色2  
username=password,role1,role2  
  
[roles]  
#提供了角色及权限之间关系的配置，角色=权限1，权限2  
role1=permission1,permission2  
  
[urls]  
#用于web，提供了对web url拦截相关的配置，url=拦截器[参数]，拦截器  
/index.html = anon  
/admin/** = authc, roles[admin], perms["permission1"] 

``` 
 
[main]部分
提供了对根对象securityManager及其依赖对象的配置。
创建对象 
securityManager=org.apache.shiro.mgt.DefaultSecurityManager  
其构造器必须是public空参构造器，通过反射创建相应的实例。
 
常量值setter注入 
dataSource.driverClassName=com.mysql.jdbc.Driver  
jdbcRealm.permissionsLookupEnabled=true   
会自动调用jdbcRealm.setPermissionsLookupEnabled(true)，对于这种常量值会自动类型转换。
 
对象引用setter注入 
authenticator=org.apache.shiro.authc.pam.ModularRealmAuthenticator  
authenticationStrategy=org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy  
authenticator.authenticationStrategy=$authenticationStrategy  
securityManager.authenticator=$authenticator   
会自动通过securityManager.setAuthenticator(authenticator)注入引用依赖。
 
嵌套属性setter注入 
securityManager.authenticator.authenticationStrategy=$authenticationStrategy   
也支持这种嵌套方式的setter注入。
 
byte数组setter注入 
#base64 byte[]  
authenticator.bytes=aGVsbG8=  
#hex byte[]  
authenticator.bytes=0x68656c6c6f   
默认需要使用Base64进行编码，也可以使用0x十六进制。
 
Array/Set/List setter注入 
authenticator.array=1,2,3  
authenticator.set=$jdbcRealm,$jdbcRealm   
多个之间通过“，”分割。
 
Map setter注入
authenticator.map=$jdbcRealm:$jdbcRealm,1:1,key:abc  
即格式是：map=key：value，key：value，可以注入常量及引用值，常量的话都看作字符串（即使有泛型也不会自动造型）。        
 
实例化/注入顺序 
realm=Realm1  
realm=Realm12  
  
authenticator.bytes=aGVsbG8=  
authenticator.bytes=0x68656c6c6f   
后边的覆盖前边的注入。
 
测试用例请参考配置文件shiro-config-main.ini。 
 
[users]部分
配置用户名/密码及其角色，格式：“用户名=密码，角色1，角色2”，角色部分可省略。如：
[users]  
zhang=123,role1,role2  
wang=123   
密码一般生成其摘要/加密存储，后续章节介绍。
 
[roles]部分
配置角色及权限之间的关系，格式：“角色=权限1，权限2”；如：
[roles]  
role1=user:create,user:update  
role2=*   
如果只有角色没有对应的权限，可以不配roles，具体规则请参考授权章节。
 
[urls]部分
配置url及相应的拦截器之间的关系，格式：“url=拦截器[参数]，拦截器[参数]，如：  
```

[urls]  
/admin/** = authc, roles[admin], perms["permission1"]  

``` 
具体规则参见web相关章节。 