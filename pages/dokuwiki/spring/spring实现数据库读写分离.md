title: spring实现数据库读写分离 

#  spring使用AbstractRoutingDataSource实现数据库读写分离 
关于mysql主从master-slave分离搭建请参考[[database:mysql主从读写分离配置]]
此次不在啰嗦

##  背景知识 

现在大型的电子商务系统，**在数据库层面大都采用读写分离技术，就是一个Master数据库，多个Slave数据库。Master库负责数据更新和实时数据查询，Slave库当然负责非实时数据查询。**因为在实际的应用中，数据库都是**读多写少**（读取数据的频率高，更新数据的频率相对较少），**而读取数据通常耗时比较长，占用数据库服务器的CPU较多，**从而影响用户体验。我们通常的做法就是**把查询从主库中抽取出来，采用多个从库，使用负载均衡，减轻每个从库的查询压力。**
采用**读写分离技术**的目标：有效减轻Master库的压力，又可以把用户查询数据的请求分发到不同的Slave库，从而保证系统的健壮性。我们看下采用读写分离的背景。
随着网站的业务不断扩展，数据不断增加，用户越来越多，数据库的压力也就越来越大，采用传统的方式，比如：数据库或者SQL的优化基本已达不到要求，这个时候可以采用读写分离的策 略来改变现状。

具体到开发中，**如何方便的实现读写分离呢?目前常用的有两种方式：**
　　1 第一种方式是我们最常用的方式，就是定义2个数据库连接，一个是MasterDataSource,另一个是SlaveDataSource。更新数据时我们读取MasterDataSource，查询数据时我们读取SlaveDataSource。这种方式很简单，我就不赘述了。
　　2 第二种方式**` 动态数据源切换 `**，就是在程序运行时，把数据源动态织入到程序中，从而选择读取主库还是从库。**主要使用的技术是：annotation，Spring AOP ，反射。**下面会详细的介绍实现方式。
　　　在介绍实现方式之前，我们先准备一些必要的知识，` spring 的AbstractRoutingDataSource ` 类
　　   AbstractRoutingDataSource这个类 是spring2.0以后增加的， ** AbstractRoutingDataSource继承了AbstractDataSource** ，而AbstractDataSource 又是DataSource 的子类。
![](/data/dokuwiki/spring/pasted/20151022-211811.png)

##  二、实现原理 
1、扩展Spring的` AbstractRoutingDataSource `抽象类（该类充当了DataSource的路由中介,** 能有在运行时, 根据某种key值来动态切换到真正的DataSource上**。）
从AbstractRoutingDataSource的源码中：
```

public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean
   我们可以看到，它继承了AbstractDataSource，而AbstractDataSource不就是javax.sql.DataSource的子类，So我们可以分析下它的getConnection方法：
public Connection getConnection() throws SQLException {  
    return determineTargetDataSource().getConnection();  
}    
public Connection getConnection(String username, String password) throws SQLException {  
     return determineTargetDataSource().getConnection(username, password);  
}

```
获取连接的方法中，重点是` determineTargetDataSource() `方法，看源码：
```

    protected DataSource determineTargetDataSource() {  
        Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");  
        Object lookupKey = determineCurrentLookupKey();  
        DataSource dataSource = this.resolvedDataSources.get(lookupKey);  
        if (dataSource == null && (this.lenientFallback || lookupKey == null)) {  
            dataSource = this.resolvedDefaultDataSource;  
        }  
        if (dataSource == null) {  
            throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");  
        }  
        return dataSource;  
    }

```
 上面这段源码的重点在于` determineCurrentLookupKey() `方法，这是AbstractRoutingDataSource类中的一个**抽象方法**，而**它的返回值是你所要用的数据源dataSource的key值**，**有了这个key值，resolvedDataSource（这是个map,由配置文件中设置好后存入的）就从中取出对应的DataSource，如果找不到，就用配置默认的数据源。**
看完源码，应该有点启发了吧，没错！` 你要扩展AbstractRoutingDataSource类，并重写其中的determineCurrentLookupKey()方法，来实现数据源的切换： `
```

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
 
/**
 * 获取数据源（依赖于spring）
 * 
 */
public class DynamicDataSource extends AbstractRoutingDataSource{
    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceRouteHolder.getDataSourceKey();
    }
}

```
DataSourceHolder这个类则是我们自己封装的对数据源进行操作的类：
```

/**
 * 数据源操作
 * 
 */
public class DataSourceRouteHolder {
    //线程本地环境
    private static final ThreadLocal<String> dataSources = new ThreadLocal<String>();
    //设置数据源
    public static void setDataSourceKey(String customerType) {
        dataSources.set(customerType);
    }
    //获取数据源
    public static String getDataSourceKey() {
        return (String) dataSources.get();
    }
    //清除数据源
    public static void clearDataSourceKey() {
        dataSources.remove();
    }
 
}

```
 2、有人就要问，那你setDataSource这方法是要在什么时候执行呢？当然是在你需要切换数据源的时候执行啦。手动在代码中调用写死吗？
这是多蠢的方法，**当然要让它动态咯。所以我们可以应用spring aop来设置，把配置的数据源类型都设置成为注解标签，在service层中需要切换数据源的方法上，写上注解标签，调用相应方法切换数据源咯**（就跟你设置事务一样）：
```

@DataSource(name=DataSource.slave1)
public List getProducts(){

```
```

import java.lang.annotation.*;
 
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSource {
    String name() default DataSource.master;
 
    public static String master = "dataSource1";
 
    public static String slave1 = "dataSource2";
 
    public static String slave2 = "dataSource3";
 
}

```
使用**spring 的aop编程在业务逻辑方法运行前将当前方法使用数据源的key从业务逻辑方法上自定义注解@DataSource中解析数据源key并添加到DataSourceRouteHolder中**
```

/**
 * 执行dao方法之前的切面
 * 获取datasource对象之前往DataSourceRouteHolder中指定当前线程数据源路由的key
 * @author Administrator
 *
 */
public class DataSourceAspect
{
	/**
	 * 在dao层方法之前获取datasource对象之前在切面中指定当前线程数据源路由的key
	 */
	public void beforeDaoMethod(JoinPoint point)
	{
		//dao方法上配置的注解
		DataSource datasource = ((MethodSignature)point.getSignature()).getMethod().getAnnotation(DataSource.class);
          	if(datasource!=null){
                  dataSourceRouteHolder.setDataSourceKey(datasource.value());
                }
	
	}
}

```
也可以采用注解方式，那样就不用进行配置了。仅供参考。
```

/**
 * 在所有service之前定义这个切面，根据service的事务注解 readOnly来决定向本地线程中设置一个数据源的key<br>
 */
@Component
@Aspect
public class DataSourceAspect {

	// 配置切入点,该方法无方法体,主要为方便同类中其他方法使用此处配置的切入点
	@Pointcut("execution(* com.c..service..*(..))")
	public void aspect() {
	}

	/**
	 * 在dao层方法之前获取datasource对象之前在切面中指定当前线程数据源路由的key
	 */
	@Before("aspect()")
	public void doBefore(JoinPoint point) {
	//dao方法上配置的注解
		DataSource datasource = ((MethodSignature)point.getSignature()).getMethod().getAnnotation(DataSource.class);
          	if(datasource!=null){
                  dataSourceRouteHolder.setDataSourceKey(datasource.value());
                }
	}
}


```
业务逻辑方法
```

@Service("userService")
public class UserService 
{
	@Autowired
	private UserDao userDao;
	
	@DataSource("master")
	@Transactional(propagation=Propagation.REQUIRED)
	public void updatePasswd(int userid,String passwd)
	{
		User user = new User();
		user.setUserid(userid);
		user.setPassword(passwd);
		userDao.updatePassword(user);
	}
	@DataSource("slave")
	@Transactional(propagation=Propagation.REQUIRED)
	public User getUser(int userid)
	{
		User user = userDao.getUserById(userid);
		System.out.println("username------:"+user.getUsername());
		return user;
	}
}

```
spring的配置文件
```
	
<bean id="master" class="org.apache.commons.dbcp.BasicDataSource">  
        <property name="driverClassName" value="${jdbc.driverclass}" />  
        <property name="url" value="${jdbc.masterurl}" />  
        <property name="username" value="${jdbc.username}" />  
        <property name="password" value="${jdbc.password}" />  
        <property name="maxActive" value="${jdbc.maxActive}"></property>  
        <property name="maxIdle" value="${jdbc.maxIdle}"></property>  
        <property name="maxWait" value="${jdbc.maxWait}"></property>  
</bean>  
    
<bean id="slave" class="org.apache.commons.dbcp.BasicDataSource">  
        <property name="driverClassName" value="${jdbc.driverclass}" />  
        <property name="url" value="${jdbc.slaveurl}" />  
        <property name="username" value="${jdbc.username}" />  
        <property name="password" value="${jdbc.password}" />  
        <property name="maxActive" value="${jdbc.maxActive}"></property>  
        <property name="maxIdle" value="${jdbc.maxIdle}"></property>  
        <property name="maxWait" value="${jdbc.maxWait}"></property>  
 </bean>
 <bean id="dataSource" class="com.westone.datasource.DbRouteDataSource">
	<property name="targetDataSources">
		<map key-type="java.lang.String">
		   <!-- write -->
		   <entry key="master" value-ref="master"></entry>
		   <!-- read -->
		   <entry key="slave" value-ref="slave"></entry>
		</map>
           <!-- 默认目标数据源为你主库数据源 -->
        <property name="defaultTargetDataSource" ref="master"/>
	</property>	
</bean>     
<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">com.datasource.test.util.database.ExtendedMySQLDialect</prop>
                <prop key="hibernate.show_sql">${SHOWSQL}</prop>
                <prop key="hibernate.format_sql">${SHOWSQL}</prop>
                <prop key="query.factory_class">org.hibernate.hql.classic.ClassicQueryTranslatorFactory</prop>
                <prop key="hibernate.connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</prop>
                <prop key="hibernate.c3p0.max_size">30</prop>
                <prop key="hibernate.c3p0.min_size">5</prop>
                <prop key="hibernate.c3p0.timeout">120</prop>
                <prop key="hibernate.c3p0.idle_test_period">120</prop>
                <prop key="hibernate.c3p0.acquire_increment">2</prop>
                <prop key="hibernate.c3p0.validate">true</prop>
                <prop key="hibernate.c3p0.max_statements">100</prop>
            </props>
        </property>
    </bean>
    <bean id="hibernateTemplate" class="org.springframework.orm.hibernate3.HibernateTemplate">
        <property name="sessionFactory" ref="sessionFactoryHibernate"/>
    </bean>
 
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
   
   
   <!-- 为业务逻辑层的方法解析@DataSource注解  为当前线程的routeholder注入数据源key -->
   	<bean id="aspectBean" class="com.westone.datasource.aspect.DataSourceAspect"></bean>
    
   <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="insert*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="add*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="update*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="modify*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="edit*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="del*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="save*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="send*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="query*" read-only="true"/>
            <tx:method name="search*" read-only="true"/>
            <tx:method name="select*" read-only="true"/>
            <tx:method name="count*" read-only="true"/>
        </tx:attributes>
    </tx:advice>
 
    <aop:config>
        <aop:pointcut id="servicePoint" expression="execution(* com.datasource..*.service.*.*(..))"/>
        <!-- 关键配置，切换数据源一定要比持久层代码更先执行（事务也算持久层代码）,order 值越大优先值越低-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="servicePoint" order="2"/>

        <aop:aspect id="dsAspect" ref="aspectBean"
            order="0">
            <aop:before method="beforeDaoMethod" pointcut-ref="servicePoint" />
        </aop:aspect>
    </aop:config>
	<!-- 开启事务注解驱动    在业务逻辑层上使用@Transactional 注解 为业务逻辑层管理事务 -->  
    <tx:annotation-driven  transaction-manager="transactionManager"/>

 <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <property name="defaultEncoding" value="${mail.smtp.encoding}" />
        <property name="host" value="${mail.smtp.host}" />
        <property name="username" value="${mail.smtp.username}" />
        <property name="password" value="${mail.smtp.password}" />
        <property name="javaMailProperties">
            <props>
                <!-- 是否开启验证 -->
                <prop key="mail.smtp.auth">${mail.smtp.auth}</prop>
                <prop key="mail.debug">true</prop>
                <!-- 设置发送延迟 -->
                <prop key="mail.smtp.timeout">${mail.smtp.timeout}</prop>
            </props>
        </property>
    </bean>
 
    <!-- 设置计划 -->
    <task:annotation-driven />

```
事务管理配置一定要配置在往DataSourceRouteHolder中注入数据源key之前 否则会报 Could not open JDBC Connection for transaction; nested exception is java.lang.IllegalStateException: Cannot determine target DataSource for lookup key [null] 找不到数据源错误
最后测试业务逻辑层的方法发现 可以根据方法上的@DataSource("master") 注解配置不同的数据源key 使用动态数据源。 不需要再业务逻辑方法层中定义不同的数据源对象，人为的使用不同的数据源操作数据。

对于SpringMVC**写拦截器**代替AOP方式动态切换数据源也是一种不错的方式
写自己的拦截器实现MethodInterceptor。在此拦截器中根据请求的方法名字设置key.然后在自己的Rout类中返回就可以了。
或者也可以更实现URL拦截器


##  更进一步，多个读库随机分配 
```

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSource {
    String name() default DataSource.master;
    //写库类型
    public static final String master = "master";
    //读库类型
    public static final String slave = "slave";
 
}

```
```

/**
 * 执行dao方法之前的切面
 * 获取datasource对象之前往DataSourceRouteHolder中指定当前线程数据源路由的key
 * @author Administrator
 *
 */
public class DataSourceAspect
{
  	@Autowired
  	private DataSourceKey dataSourcekey;
	/**
	 * 在dao层方法之前获取datasource对象之前在切面中指定当前线程数据源路由的key
	 */
	public void beforeDaoMethod(JoinPoint point)
	{
		//dao方法上配置的注解
		DataSource datasource = ((MethodSignature)point.getSignature()).getMethod().getAnnotation(DataSource.class);
          	if(datasource!=null){
                  if(datasource.value().equals(datasource.master)){
                    DataSourceRouteHolder.setDataSourceKey(dataSourceKey.getMaster());
                  }else if(datasource.value().equals(datasource.slave)){
                    DataSourceRouteHolder.setDataSourceKey(dataSourceKey.getSlave());
                  }
                }
	
	}
}

```
```

public class DataSourceKey{
		  private  String writeKey;
		  private  List<String> readKeys;
		  
		  public void setWriteKey(String writeKey){
		  	this.writeKey=writeKey;
		  }
		  public String getMaster(){
		    return writeKey;
		  }
		  public void setReadKeys(List keys){
		    readKeys =keys;
		  }
		  public String getSlave(){
		    int index=new Random().nextInt(readKeys.size());
		    return readKeys.get(index);
		  }
}

```
```

<bean id="dataSource" class="com.westone.datasource.DbRouteDataSource">
	<property name="targetDataSources">
		<map key-type="java.lang.String">
		   <!-- write -->
		   <entry key="master" value-ref="master"></entry>
		   <!-- read -->
		   <entry key="slave1" value-ref="slave"></entry>
                  <entry key="slave2" value-ref="slave1"></entry>
		</map>
           <!-- 默认目标数据源为你主库数据源 -->
        <property name="defaultTargetDataSource" ref="master"/>
	</property>	
</bean>     
 <bean id="emf"  class="net.xby1993.DataSourceKey">  
        <property name="writeKey" value="master" />
        <property name="readKeys">
        <list>
        	<value>slave1</value>
        	<value>slave2</value>
        </list> 
        </property>

```
参考：
http://www.aichengxu.com/view/35238
http://www.cnblogs.com/davidwang456/p/4318303.html
http://www.oschina.net/code/snippet_170632_48518
http://blog.csdn.net/fairyhawk/article/details/7937597