title: xml配置 

#  mybatis:xml配置 

```

<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
  "http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  
  <environments default="development">  
    <environment id="development">  
      <transactionManager type="JDBC"/>  
      <dataSource type="POOLED">  
        <property name="driver" value="${driver}"/>  
        <property name="url" value="${url}"/>  
        <property name="username" value="${username}"/>  
        <property name="password" value="${password}"/>  
      </dataSource>  
    </environment>  
  </environments>  
  <mappers>  
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>  
  </mappers>  
</configuration>  

```
#  文档结构层次 

MyBatis 的配置文件包含了影响 MyBatis 行为甚深的设置（settings）和属性（properties）信息。文档的顶层结构如下：
configuration 配置
  * properties 属性
  * settings 设置
  * typeAliases 类型命名
  * typeHandlers 类型处理器
  * objectFactory 对象工厂
  * plugins 插件
  * environments 环境
    * environment 环境变量
      * transactionManager 事务管理器
      * dataSource 数据源
  * databaseIdProvider 数据库厂商标识
  * mappers 映射器

#  properties 

这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。
```

<properties resource="org/mybatis/example/config.properties">  
  <property name="username" value="dev_user"/>  
  <property name="password" value="F2Fa3!33TYyg"/>  
</properties>  
其中的属性就可以在整个配置文件中使用来替换需要动态配置的属性值。
<dataSource type="POOLED">  
  <property name="driver" value="${driver}"/>  
  <property name="url" value="${url}"/>  
  <property name="username" value="${username}"/>  
  <property name="password" value="${password}"/>  
</dataSource>  

```
属性也可以被传递到 SqlSessionBuilder.build()方法中。
##  属性加载顺序 

如果属性在不只一个地方进行了配置，那么 MyBatis 将按照下面的顺序来加载：

在 properties 元素体内指定的属性首先被读取。
然后会读取从类路径下资源或 properties 元素中的 url 属性（url attributes）中加载的属性，它会覆盖已读取的同名属性。
最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。
因此，通过方法参数传递的属性具有最高优先级，资源文件及 url 属性配置的次之，最低优先级的是 properties 属性中指定的属性。

#  settings 

调整 settings 中的设置是非常关键的，它们会改变 MyBatis 的运行时行为。具体设置参数查看官方文档。示例：
```

<settings>  
  <setting name="cacheEnabled" value="true"/>  
  <setting name="lazyLoadingEnabled" value="true"/>  
  <setting name="multipleResultSetsEnabled" value="true"/>  
  <setting name="useColumnLabel" value="true"/>  
  <setting name="useGeneratedKeys" value="false"/>  
  <setting name="autoMappingBehavior" value="PARTIAL"/>  
  <setting name="defaultExecutorType" value="SIMPLE"/>  
  <setting name="defaultStatementTimeout" value="25"/>  
  <setting name="safeRowBoundsEnabled" value="false"/>  
  <setting name="mapUnderscoreToCamelCase" value="false"/>  
  <setting name="localCacheScope" value="SESSION"/>  
  <setting name="jdbcTypeForNull" value="OTHER"/>  
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>  
</settings>  

```
#  typeAliases 

类型别名是为 Java 类型命名的一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。例如：
```

<typeAliases>  
  <typeAlias alias="Author" type="domain.blog.Author"/>  
  <typeAlias alias="Blog" type="domain.blog.Blog"/>  
  <typeAlias alias="Comment" type="domain.blog.Comment"/>  
</typeAliases>  
使用这个配置，“Blog”可以用在任何使用“domain.blog.Blog”的地方。

也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如
<typeAliases>  
  <package name="domain.blog"/>  
</typeAliases>  

```
这样的话，每一个在包 domain.blog 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。比如 domain.blog.Author 的别名为 author；若有注解，则别名为其注解值。看下面的例子：
```

@Alias("author")  
public class Author {  
    ...  
}  

```
已经为普通的 Java 类型内建了许多相应的类型别名。它们都是大小写不敏感的.
![](/data/dokuwiki/mybatis/pasted/20150511-050204.png)

#  typeHandlers 

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时，都会用类型处理器将获取的值以合适的方式转换成 Java 类型。一些默认的类型处理器查看官方文档即可

你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。具体做法为：实现 org.apache.ibatis.type.TypeHandler 接口，或继承一个很便利的类 org.apache.ibatis.type.BaseTypeHandler，然后可以将它映射到一个 JDBC 类型。
```

@MappedJdbcTypes(JdbcType.VARCHAR)  
public class ExampleTypeHandler extends BaseTypeHandler<String> {  
  
  @Override  
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {  
    ps.setString(i, parameter);  
  }  
  
  @Override  
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {  
    return rs.getString(columnName);  
  }  
  
  @Override  
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {  
    return rs.getString(columnIndex);  
  }  
  
  @Override  
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {  
    return cs.getString(columnIndex);  
  }  
}  
  
  
<!-- mybatis-config.xml -->  
<typeHandlers>  
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>  
</typeHandlers>  

```
要注意 MyBatis 不会窥探数据库元信息来决定使用哪种类型，所以你必须在参数和结果映射中指明那是 VARCHAR 类型的字段，以使其能够绑定到正确的类型处理器上。这基于一个事实：MyBatis 直到语句被执行才会去关心数据类型。

MyBatis 会通过窥探属性的原始类型（generic type）来推断由类型处理器处理的 Java 类型，不过这种行为可以通过两种方法改变：

在类型处理器的配置元素（typeHandler element）上增加一个 javaType 属性（比如：javaType="String"）；
在类型处理器的类上（TypeHandler class）增加一个 @MappedTypes 注解来指定与其关联的 Java 类型列表。 如果在 javaType 属性中也同时指定，则注解方式将被忽略。
可以通过两种方式来指定被关联的 JDBC 类型：

在类型处理器的配置元素上增加一个 javaType 属性（比如：javaType="VARCHAR"）；
在类型处理器的类上（TypeHandler class）增加一个 @MappedJdbcTypes 注解来指定与其关联的 JDBC 类型列表。 如果在 javaType 属性中也同时指定，则注解方式将被忽略。
最后，可以让 MyBatis 为你查找类型处理器：
<!-- mybatis-config.xml -->
<typeHandlers>
<package name="org.mybatis.example"/>
</typeHandlers>
` 注意在使用自动检索（autodiscovery）功能的时候，只能通过注解方式来指定 JDBC 的类型。 `

###  处理枚举类型 

若想映射枚举类型 Enum，则需要从 EnumTypeHandler 或者 EnumOrdinalTypeHandler 中选一个来使用。

#  对象工厂（objectFactory） 

` MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成 `。默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现。比如：
```

public class ExampleObjectFactory extends DefaultObjectFactory {  
  public Object create(Class type) {  
    return super.create(type);  
  }  
  public Object create(Class type, List<Class> constructorArgTypes, List<Object> constructorArgs) {  
    return super.create(type, constructorArgTypes, constructorArgs);  
  }  
  public void setProperties(Properties properties) {  
    super.setProperties(properties);  
  }  
  public <T> boolean isCollection(Class<T> type) {  
    return Collection.class.isAssignableFrom(type);  
  }}  
<!-- mybatis-config.xml -->  
<objectFactory type="org.mybatis.example.ExampleObjectFactory">  
  <property name="someProperty" value="100"/>  
</objectFactory>  

```

#  插件（plugins） 

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

  * Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
  * ParameterHandler (getParameterObject, setParameters)
  * ResultSetHandler (handleResultSets, handleOutputParameters)
  * StatementHandler (prepare, parameterize, batch, update, query)

通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定了想要拦截的方法签名即可。
```
  
@Intercepts({@Signature(  
  type= Executor.class,  
  method = "update",  
  args = {MappedStatement.class,Object.class})})  
public class ExamplePlugin implements Interceptor {  
  public Object intercept(Invocation invocation) throws Throwable {  
    return invocation.proceed();  
  }  
  public Object plugin(Object target) {  
    return Plugin.wrap(target, this);  
  }  
  public void setProperties(Properties properties) {  
  }  
}  
<!-- mybatis-config.xml -->  
<plugins>  
  <plugin interceptor="org.mybatis.example.ExamplePlugin">  
    <property name="someProperty" value="100"/>  
  </plugin>  
</plugins>  
  </code>
上面的插件将会拦截在 Executor 实例中所有的“update”方法调用，这里的 Executor 是负责低层映射语句执行的内部对象。
#  配置环境（environments） 

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中。

不过要记住：` 尽管可以配置多个环境，每个 SqlSessionFactory 实例只能选择其一。 `

` 所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。 `而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：

每个数据库对应一个 SqlSessionFactory 实例
为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。可以接受环境配置的两个方法签名是：
<code xml>
SqlSessionFactory factory = sqlSessionFactoryBuilder.build(reader, environment);  
SqlSessionFactory factory = sqlSessionFactoryBuilder.build(reader, environment,properties);  
环境元素定义了如何配置环境。
<environments default="development">  
  <environment id="development">  
    <transactionManager type="JDBC">  
      <property name="..." value="..."/>  
    </transactionManager>  
    <dataSource type="POOLED">  
      <property name="driver" value="${driver}"/>  
      <property name="url" value="${url}"/>  
      <property name="username" value="${username}"/>  
      <property name="password" value="${password}"/>  
    </dataSource>  
  </environment>  
</environments>  

```
##  事务管理器（transactionManager） 

在 MyBatis 中有两种类型的事务管理器（也就是 type=”[JDBC|MANAGED]”）：

JDBC – 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务范围。
MANAGED – 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。例如:
```

<transactionManager type="MANAGED">  
  <property name="closeConnection" value="false"/>  
</transactionManager>  

```
` 如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。 `

##  数据源（dataSource） 

dataSource 元素使用基本的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

许多 MyBatis 的应用程序将会按示例中的例子来配置数据源。` 然而它并不是必须的。要知道为了方便使用延迟加载，数据源才是必须的 `。
有三种内建的数据源类型（也就是 type=”???”）：UNPOOLED| POOLED| JNDI

###  无连接池（UNPOOLED） 
– 这个数据源的实现是每次被请求时简单打开和关闭连接
  * UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：
  * driver – 这是 JDBC 驱动的 Java 类的完全限定名（如果你的驱动包含，它也不是数据源类）。
  * url – 这是数据库的 JDBC URL 地址。
  * username – 登录数据库的用户名。
  * password – 登录数据库的密码。
  * defaultTransactionIsolationLevel – 默认的连接事务隔离级别。
作为可选项，你也可以传递 properties 给数据库驱动。要这样做，属性的前缀以“driver.”开头，例如：
driver.encoding=UTF8
这样就会传递以值 “UTF8” 来传递属性“encoding”，它是通过DriverManager.getConnection(url,driverProperties)方法传递给数据库驱动的。

###  有连接池（POOLED） 
– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。这是一种使得并发 Web 应用快速响应请求的流行处理方式。

除了上述无连接池（UNPOOLED）的属性外，会有更多属性用来配置有连接池（POOLED）的数据源：

  * poolMaximumActiveConnections – 在任意时间可以存在的活动（也就是正在使用）连接数量，默认值：10
  * poolMaximumIdleConnections – 任意时间可能存在的空闲连接数。
  * poolMaximumCheckoutTime – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
  * poolTimeToWait – 这是一个低层设置，如果获取连接花费的相当长的时间，它会给连接池打印日志并重新尝试获取一个连接的机会（避免在误配置的情况下一直安静的失败），默认值：20000 毫秒（即 20 秒）。
  * poolPingQuery – 发送到数据的侦测查询，用来检验连接是否处在正常工作秩序中并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息。
  * poolPingEnabled – 是否启用侦测查询。若开启，也必须使用一个可执行的 SQL 语句设置 poolPingQuery 属性（最好是一个非常快的 SQL），默认值：false。
  * poolPingConnectionsNotUsedFor – 配置 poolPingQuery 的使用频度。这可以被设置成匹配标准的数据库连接超时时间，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。

###  JNDI 
– 这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。这种数据源配置只需要两个属性：
  * initial_context – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么 data_source 属性将会直接从 InitialContext 中寻找。
  * data_source – 这是引用数据源实例位置的上下文的路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。
和其他数据源配置类似，可以通过名为“env.”的前缀直接向初始上下文发送属性（send properties）。比如：
env.encoding=UTF8
这就会在初始化时以值“UTF8”向初始上下文的构造方法传递名为“encoding”的属性。

##  自定义数据源（以C3p0为例） 

也可使用任何第三方数据源，需要实现这个接口 org.apache.ibatis.datasource.DataSourceFactory，但是已经有一个便利类了，那就是
org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory 可被用作父类来构建新的数据源适配器，
以自定义C3p0为例：
```

import org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory;  
import com.mchange.v2.c3p0.ComboPooledDataSource;  
          
public class C3P0DataSourceFactory extends UnpooledDataSourceFactory {  
  
  public C3P0DataSourceFactory() {  
    this.dataSource = new ComboPooledDataSource();  
  }  
}  

```
为了令其工作，可在每个需要 MyBatis 调用的 setter 方法中增加一个属性。下面是一个可以连接至 MySQL 数据库的例子：
```

<dataSource type="com.mybatis.bean.C3P0DataSourceFactory">  
           <property name="driverClass" value="com.mysql.jdbc.Driver" />  
           <property name="jdbcUrl" value="jdbc:mysql://127.0.0.1:3306/study" />  
           <property name="user" value="root" />  
           <property name="password" value="abcde2008" />  
           <property name="idleConnectionTestPeriod" value="60" />  
           <property name="maxPoolSize" value="20" />  
           <property name="maxIdleTime" value="600" />  
           <property name="preferredTestQuery" value="SELECT 1" />  
       </dataSource>  

```

databaseIdProvider
MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 databaseId 属性的。
为支持多厂商特性只要像下面这样在 mybatis-config.xml 文件中加入 databaseIdProvider 即可：
<databaseIdProvider type="DB_VENDOR" />  
这里的 DB_VENDOR 会通过 DatabaseMetaData#getDatabaseProductName() 返回的字符串进行设置。由于通常情况下这个字符串都非常长而且相同产品的不同版本会返回不同的值，所以最好通过设置属性别名来使其变短，如下
```

<databaseIdProvider type="DB_VENDOR">  
  <property name="SQL Server" value="sqlserver"/>  
  <property name="DB2" value="db2"/>          
  <property name="Oracle" value="oracle" />  
</databaseIdProvider>  

```

#  映射器（mappers） 
```

你可以使用相对于类路径的资源引用，或完全限定资源定位符（包括 file:/// 的 URL），或类名和包名等。例如：
<!-- Using classpath relative resources -->  
<mappers>  
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>  
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>  
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>  
</mappers>  
<!-- Using url fully qualified paths -->  
<mappers>  
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>  
  <mapper url="file:///var/mappers/BlogMapper.xml"/>  
  <mapper url="file:///var/mappers/PostMapper.xml"/>  
</mappers>  
<!-- Using mapper interface classes -->  
<mappers>  
  <mapper class="org.mybatis.builder.AuthorMapper"/>  
  <mapper class="org.mybatis.builder.BlogMapper"/>  
  <mapper class="org.mybatis.builder.PostMapper"/>  
</mappers>  
<!-- Register all interfaces in a package as mappers -->  
<mappers>  
  <package name="org.mybatis.builder"/>  
</mappers>  

```