title: c3p0数据源的使用初步及mysql8小时问题解决 

#  c3p0数据源的使用初步及mysql8小时问题解决 

c3p0的配置方式分为三种，分别是
1.setters一个个地设置各个配置项
2.类路径下提供一个c3p0.properties文件
3.类路径下提供一个c3p0-config.xml文件
我们主要说下c3p0-config.xml配置：

<c3p0-config>
//默认池配置
<default-config>  
<property name="user">root</property>
<property name="password">java</property>
<property name="driverClass">com.mysql.jdbc.Driver</property>
<property name="jdbcUrl">jdbc:mysql://localhost:3306/jdbc</property>

   <property name="initialPoolSize">10</property>
   <property name="maxIdleTime">30</property>
   <property name="maxPoolSize">100</property>
   <property name="minPoolSize">10</property>
 </default-config>
 //命名池配置
 <named-config name="myApp">
<property name="user">root</property>
<property name="password">java</property>
<property name="driverClass">com.mysql.jdbc.Driver</property>
<property name="jdbcUrl">jdbc:mysql://localhost:3306/jdbc</property>
   <property name="initialPoolSize">10</property>
   <property name="maxIdleTime">30</property>
   <property name="maxPoolSize">100</property>
   <property name="minPoolSize">10</property>
 </named-config>
</c3p0-config>

可见c3p0可以配置在一个应用中使用多个池。换言之可以使用多个数据库。
使用：
//使用默认池：
private static ComboPooledDataSource ds = new ComboPooledDataSource();
//使用命名池
private static ComboPooledDataSource ds = new ComboPooledDataSource("myApp");
基本配置项的介绍：

##  1.基本配置 

acquireIncrement
default : 3
连接池在无空闲连接可用时一次性创建的新数据库连接数
initialPoolSize
default : 3
连接池初始化时创建的连接数
maxPoolSize
default : 15
连接池中拥有的最大连接数，如果获得新连接时会使连接总数超过这个值则不会再获取新连接，而是等待
其他连接释放，所以这个值有可能会设计地很大
maxIdleTime
default : 0 单位 s
连接的最大空闲时间，如果超过这个时间，某个数据库连接还没有被使用，则会断开掉这个连接
如果为0，则永远不会断开连接
minPoolSize
default : 3
连接池保持的最小连接数，后面的maxIdleTimeExcessConnections跟这个配合使用来减轻连接池的负载

##  2.管理连接池的大小和连接的生存时间 

maxConnectionAge
default : 0 单位 s
配置连接的生存时间，超过这个时间的连接将由连接池自动断开丢弃掉。当然正在使用的连接不会马上断开，而是等待
它close再断开。配置为0的时候则不会对连接的生存时间进行限制。
maxIdleTimeExcessConnections
default : 0 单位 s
这个配置主要是为了减轻连接池的负载，比如连接池中连接数因为某次数据访问高峰导致创建了很多数据连接
但是后面的时间段需要的数据库连接数很少，则此时连接池完全没有必要维护那么多的连接，所以有必要将
断开丢弃掉一些连接来减轻负载，必须小于maxIdleTime。配置不为0，则会将连接池中的连接数量保持到minPoolSize。
为0则不处理。
##  3.配置连接测试 
：因为连接池中的数据库连接很有可能是维持数小时的连接，很有可能因为数据库服务器的问题，网络问题等导致实际连接已经无效，但是连接池里面的连接还是有效的，如果此时获得连接肯定会发生异常，所以有必要通过测试连接来确认连接的有效性。
下面的前三项用来配置如何对连接进行测试，后三项配置对连接进行测试的时机。
automaticTestTable
default : null
用来配置测试连接的一种方式。配置一个表名，连接池根据这个表名创建一个空表，
并且用自己的测试sql语句在这个空表上测试数据库连接
这个表只能由c3p0来使用，用户不能操作，同时用户配置的preferredTestQuery 将会被忽略。
preferredTestQuery
default : null
用来配置测试连接的另一种方式。与上面的automaticTestTable二者只能选一。
如果要用它测试连接，千万不要设为null，否则测试过程会很耗时，同时要保证sql语句中的表在数据库中一定存在。
connectionTesterClassName
default :  com.mchange.v2.c3p0.impl.DefaultConnectionTester
连接池用来支持automaticTestTable和preferredTestQuery测试的类，必须是全类名，就像默认的那样，
可以通过实现UnifiedConnectionTester接口或者继承AbstractConnectionTester来定制自己的测试方法
idleConnectionTestPeriod
default : 0
用来配置测试空闲连接的间隔时间。测试方式还是上面的两种之一，可以用来解决MySQL8小时断开连接的问题。因为它
保证连接池会每隔一定时间对空闲连接进行一次测试，从而保证有效的空闲连接能每隔一定时间访问一次数据库，将于MySQL
8小时无会话的状态打破。为0则不测试。
testConnectionOnCheckin
default : false
如果为true，则在close的时候测试连接的有效性。为了提高测试性能，可以与idleConnectionTestPeriod搭配使用，
配置preferredTestQuery或automaticTestTable也可以加快测试速度。
testConnectionOnCheckout
default : false
性能消耗大。如果为true，在每次getConnection的时候都会测试，为了提高性能，
可以与idleConnectionTestPeriod搭配使用，
配置preferredTestQuery或automaticTestTable也可以加快测试速度。
##  4.配置PreparedStatement缓存 

maxStatements
default : 0
连接池为数据源缓存的PreparedStatement的总数。由于PreparedStatement属于单个Connection,所以
这个数量应该根据应用中平均连接数乘以每个连接的平均PreparedStatement来计算。为0的时候不缓存，
同时maxStatementsPerConnection的配置无效。
maxStatementsPerConnection
default : 0
连接池为数据源单个Connection缓存的PreparedStatement数，这个配置比maxStatements更有意义，因为
它缓存的服务对象是单个数据连接，如果设置的好，肯定是可以提高性能的。为0的时候不缓存。
注意当数据池不再使用时需要调用close()方法手动关闭它配置举例：
##  我的c3p0配置实例 
```

<c3p0-config>
<!--解决Mysql 8小时问题 -->
<default-config>
<property name="automaticTestTable">C3P0Test</property>
<property name="idleConnectionTestPeriod">60</property>
<!--设置为2个小时 -->
<property name="maxIdleTime">60</property> 
<!--获取连接时测试是否有效 ,消耗较大，不建议使用-->
<!-- <property name="testConnectionCheckin">true</property> -->
<!--  -->
<property name="initialPoolSize">10</property>
<property name="maxPoolSize">100</property> 
<property name="minPoolSize">10</property> 
<property name="maxStatements">100</property>

<!--获取连接超时毫秒为单位--->
<property name="checkoutTimeout">30000</property>
</default-config>
</c3p0-config>

```


#  解决MYSQL 8小时问题 

Mysql服务器默认的“wait_timeout”是8小时，也就是说一个connection空闲超过8个小时，Mysql将自动断开该 connection。这就是问题的所在，在C3P0 pools中的connections如果空闲超过8小时，Mysql将其断开，而C3P0并不知道该connection已经失效，如果这时有 Client请求connection，C3P0将该失效的Connection提供给Client，将会造成上面的异常。
解决的方法有3种：
   增加wait_timeout的时间。
   减少Connection pools中connection的lifetime。
   测试Connection pools中connection的有效性。
C3P0增加以下配置信息:
   //自动测试的table名称
   automaticTestTable=C3P0TestTable
   //set to something much less than wait_timeout, prevents connections from going stale。每个小时测试一次
   idleConnectionTestPeriod = 3600
   //set to something slightly less than wait_timeout, preventing 'stale' connections from being handed out最大空闲时间为2个小时
   maxIdleTime = 7200
//获取connnection时测试是否有效
testConnectionOnCheckin = true

#  整体说明 

<c3p0-config>
<default-config>
<!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
<property name="acquireIncrement">3</property>
<!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
<property name="acquireRetryAttempts">30</property>
<!--两次连接中间隔时间，单位毫秒。Default: 1000 -->
<property name="acquireRetryDelay">1000</property>
<!--连接关闭时默认将所有未提交的操作回滚。Default: false -->
<property name="autoCommitOnClose">false</property>
<!--c3p0将建一张名为Test的空表，并使用其自带的查询语句进行测试。如果定义了这个参数那么
属性preferredTestQuery将被忽略。你不能在这张Test表上进行任何操作，它将只供c3p0测试
使用。Default: null-->
<property name="automaticTestTable">Test</property>
<!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效
保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试
获取连接失败后该数据源将申明已断开并永久关闭。Default: false-->
<property name="breakAfterAcquireFailure">false</property>
<!--当连接池用完时客户端调用getConnection()后等待获取新连接的时间，超时后将抛出
SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
<property name="checkoutTimeout">100</property>
<!--通过实现ConnectionTester或QueryConnectionTester的类来测试连接。类名需制定全路径。
Default: com.mchange.v2.c3p0.impl.DefaultConnectionTester-->
<property name="connectionTesterClassName"></property>
<!--指定c3p0 libraries的路径，如果（通常都是这样）在本地即可获得那么无需设置，默认null即可
Default: null-->
<property name="factoryClassLocation">null</property>
<!--Strongly disrecommended. Setting this to true may lead to subtle and bizarre bugs.
（文档原文）作者强烈建议不使用的一个属性-->
<property name="forceIgnoreUnresolvedTransactions">false</property>
<!--每60秒检查所有连接池中的空闲连接。Default: 0 -->
<property name="idleConnectionTestPeriod">60</property>
<!--初始化时获取三个连接，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->
<property name="initialPoolSize">3</property>
<!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
<property name="maxIdleTime">60</property>
<!--连接池中保留的最大连接数。Default: 15 -->
<property name="maxPoolSize">15</property>
<!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements
属于单个connection而不是整个连接池。所以设置这个参数需要考虑到多方面的因素。
如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0-->
<property name="maxStatements">100</property>
<!--maxStatementsPerConnection定义了连接池内单个连接所拥有的最大缓存statements数。Default: 0 -->
<property name="maxStatementsPerConnection"></property>
<!--c3p0是异步操作的，缓慢的JDBC操作通过帮助进程完成。扩展这些操作可以有效的提升性能
通过多线程实现多个操作同时被执行。Default: 3-->
<property name="numHelperThreads">3</property>
<!--当用户调用getConnection()时使root用户成为去获取连接的用户。主要用于连接池连接非c3p0
的数据源时。Default: null-->
<property name="overrideDefaultUser">root</property>
<!--与overrideDefaultUser参数对应使用的一个参数。Default: null-->
<property name="overrideDefaultPassword">password</property>
<!--密码。Default: null-->
<property name="password"></property>
<!--定义所有连接测试都执行的测试语句。在使用连接测试的情况下这个一显著提高测试速度。注意：
测试的表必须在初始数据源的时候就存在。Default: null-->
<property name="preferredTestQuery">select id from test where id=1</property>
<!--用户修改系统配置参数执行前最多等待300秒。Default: 300 -->
<property name="propertyCycle">300</property>
<!--因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的
时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable
等方法来提升连接测试的性能。Default: false -->
<property name="testConnectionOnCheckout">false</property>
<!--如果设为true那么在取得连接的同时将校验连接的有效性。Default: false -->
<property name="testConnectionOnCheckin">true</property>



#  补充一点 
，与c3p0配置无关，但是对于tomcat中配置DBCP很重要。
对于tomcat中的DBCP，也需要解决Mysql 8小时问题，所以需要这样：
  //set to 'SELECT 1'
  validationQuery = "SELECT 1"
  //set to 'true'
  testWhileIdle = "true"
  //some positive integer
  timeBetweenEvictionRunsMillis = 3600000
  //set to something smaller than 'wait_timeout'
  minEvictableIdleTimeMillis = 18000000
  //if you don't mind a hit for every getConnection(), set to "true"
  testOnBorrow = "true"
参考：http://my.oschina.net/lyzg/blog/55133
http://www.blogjava.net/Alpha/archive/2009/03/29/262789.html
