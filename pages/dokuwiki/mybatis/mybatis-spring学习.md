title: mybatis-spring学习 

#  mybatis:mybatis-spring学习

MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中。 使用这个类库中的类, Spring 将会加载必要的 MyBatis 工厂类和 session 类。 这个类库也提供一个简单的方式来注入 MyBatis 数据映射器和 SqlSession 到业务层的 bean 中。 而且它也会处理事务, 翻译 MyBatis 的异常到 Spring 的 DataAccessException 异常(数据访问异)中。最终,它并 不会依赖于 MyBatis,Spring 或 MyBatis-Spring 来构建应用程序代码。

#  Quick Start 

要和 Spring 一起使用 MyBatis,你需要在 Spring 应用上下文中定义至少两样东西:一个 SqlSessionFactory 和至少一个数据映射器类。
```

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
  <property name="dataSource" ref="dataSource" />  
</bean>  

```
要注意 SqlSessionFactory 需要一个 DataSource(数据源,译者注) 。这可以是任意 的 DataSource,配置它就和配置其它 Spring 数据库连接一样。
```

public interface UserMapper {  
  @Select("SELECT * FROM users WHERE id = #{userId}")  
  User getUser(@Param("userId") String userId);  
}   
那么可以使用 MapperFactoryBean,像下面这样来把接口加入到 Spring 中:
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">  
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />  
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />  
</bean>  

```
要注意,所指定的映射器类必须是一个接口,而不是具体的实现类。在这个示例中,注 解被用来指定 SQL 语句,` 但是 MyBatis 的映射器 XML 文件也可以用。 `
一旦配置好,你可以用注入其它任意 Spring 的 bean 相同的方式直接注入映射器到你的 business/service 对象中。MapperFactoryBean 处理 SqlSession 的创建和关闭它。如果使用 了 Spring 的事务,那么当事务完成时,session 将会提交或回滚。最终,任何异常都会被翻 译成 Spring 的 DataAccessException 异常。

调用 MyBatis 数据方法现在只需一行代码:
```

public class FooServiceImpl implements FooService {  
  
private UserMapper userMapper;  
  
public void setUserMapper(UserMapper userMapper) {  
  this.userMapper = userMapper;  
}  
  
public User doSomeBusinessStuff(String userId) {  
  return this.userMapper.getUser(userId);  
}  

```
#  SqlSessionFactoryBean属性 

SqlSessionFactory 有一个单独的必须属性,就是 JDBC 的 DataSource。这可以是任意 的 DataSource,其配置应该和其它 Spring 数据库连接是一样的。

一个通用的属性是 ` configLocation `,它是用来指定 MyBatis 的 XML 配置文件路径的。**如果基本的 MyBatis 配置需要改变, 那么这就是一个需要它的地方。 通常这会是<settings> 或<typeAliases>的部分。**` 要注意这个配置文件不需要是一个完整的 MyBatis 配置。确切地说,任意环境,数据源 和 MyBatis 的事务管理器都会被忽略 `。SqlSessionFactoryBean 会创建它自己的,使用这些 值定制 MyBatis 的 Environment 时是需要的。

如果 MyBatis 映射器 XML 文件在和映射器类相同的路径下不存在,那么另外一个需要 配置文件的原因就是它了。使用这个配置,有两种选择。第一是手动在 MyBatis 的 XML 配 置文件中使用<mappers>部分来指定类路径。第二是使用工厂 bean 的 mapperLocations 属 性。

**mapperLocations** 属性使用一个资源位置的 list。 这个属性可以用来指定 MyBatis 的 XML 映射器文件的位置。 它的值可以包含 Ant 样式来加载一个目录中所有文件, 或者从基路径下 递归搜索所有路径。比如:
```

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
  <property name="dataSource" ref="dataSource" />  
  <property name="mapperLocations" value="classpath*:sample/config/mappers/**/*.xml" />  
</bean>  

```
这会从类路径下加载在 sample.config.mappers 包和它的子包中所有的 MyBatis 映射器 XML 文件。

#  事务 

一个使用 MyBatis-Spring 的主要原因是它允许 MyBatis 参与到 Spring 的事务管理中。而 不是给 MyBatis 创建一个新的特定的事务管理器,MyBatis-Spring 利用了存在于 Spring 中的 DataSourceTransactionManager。

一旦 Spring 的 PlatformTransactionManager 配置好了,你可以在 Spring 中以你通常的做 法来配置事务。@Transactional 注解和 AOP(Aspect-Oriented Program,面向切面编程,译 者注)样式的配置都是支持的。在事务处理期间,一个单独的 SqlSession 对象将会被创建 和使用。当事务完成时,这个 session 会以合适的方式提交或回滚。

一旦事务创建之后,MyBatis-Spring 将会透明的管理事务。在你的 DAO 类中就不需要额 外的代码了。

<note important>你 不 能 在 Spring 管 理 的 SqlSession 上 调 用 SqlSession.commit() , SqlSession.rollback() 或 SqlSession.close() 方 法 。 如 果 这 样 做 了 , 就 会 抛 出 UnsupportedOperationException 异常。注意在使用注入的映射器时不能访问那些方法。</note>
##  方式一：标准方式 

要 开 启 Spring 的 事 务 处 理 , 在 Spring 的 XML 配 置 文 件 中 简 单 创 建 一 个 DataSourceTransactionManager 对象:
```

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
  <property name="dataSource" ref="dataSource" />  
</bean>  

```
指定的 DataSource 一般可以是你使用 Spring 的任意 JDBC DataSource。这包含了连接 池和通过 JNDI 查找获得的 DataSource。

要注意, 为事务管理器指定的 DataSource 必须和用来创建 SqlSessionFactoryBean 的 是同一个数据源,否则事务管理器就无法工作了。

##  方式二：JEE容器管理事务 

如果你正使用一个 JEE 容器而且想让 Spring 参与到容器管理事务(Container managed transactions,CMT,译者注)中,那么 Spring 应该使用 JtaTransactionManager 或它的容 器指定的子类来配置。做这件事情的最方便的方式是用 Spring 的事务命名空间:
<blockquote><tx:jta-transaction-manager /> </blockquote> 
在这种配置中,MyBatis 将会和其它由 CMT 配置的 Spring 事务资源一样。Spring 会自动 使用任意存在的容器事务,在上面附加一个 SqlSession。如果没有开始事务,或者需要基 于事务配置,Spring 会开启一个新的容器管理事务。

注 意 , 如 果 你 想 使 用 CMT , 而 不 想 使 用 Spring 的 事 务 管 理 , 你 就 必 须 配 置 SqlSessionFactoryBean 来使用基本的 MyBatis 的 ManagedTransactionFactory 而不是其 它任意的 Spring 事务管理器:
```

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
  <property name="dataSource" ref="dataSource" />  
  <property name="transactionFactory">  
    <bean class="org.apache.ibatis.transaction.managed.ManagedTransactionFactory" />  
  </property>    
</bean>  

```
#  使用SqlSession 

使用 MyBatis-Spring 之后, 你不再需要直接使用 SqlSessionFactory 了,因为你的 bean 可以通过一个线程安全的 SqlSession 来注入,基于 Spring 的事务配置 来自动提交,回滚,关闭 session。注意通常不必直接使用 SqlSession。 在大多数情况下 MapperFactoryBean, 将会在 bean 中注入所需要的映射器。

##  SqlSessionTemplate 

**SqlSessionTemplate 是 MyBatis-Spring 的核心**。 这个类负责管理 MyBatis 的 SqlSession, 调用 MyBatis 的 SQL 方法, 翻译异常。` SqlSessionTemplate 是线程安全的, 可以被多个 DAO 所共享使用。 `

当调用 SQL 方法时, 包含从映射器 getMapper()方法返回的方法, SqlSessionTemplate 将会保证使用的 SqlSession 是和当前 Spring 的事务相关的。此外,它管理 session 的生命 周期,包含必要的关闭,提交或回滚操作。

SqlSessionTemplate 实现了 SqlSession 接口,这就是说,在代码中无需对 MyBatis 的 SqlSession 进行替换。 SqlSessionTemplate 通常是被用来替代默认的 MyBatis 实现的 DefaultSqlSession , 因为模板可以参与到 Spring 的事务中并且被多个注入的映射器类所使 用时也是线程安全的。相同应用程序中两个类之间的转换可能会引起数据一致性的问题。

SqlSessionTemplate 对象可以使用 SqlSessionFactory 作为构造方法的参数来创建。
```

<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">  
  <constructor-arg index="0" ref="sqlSessionFactory" />  
</bean>  

```
SqlSessionTemplate 有一个使用 ExecutorType 作为参数的构造方法。这允许你用来 创建对象,比如,一个**批量 SqlSession**,但是使用了下列 Spring 配置的 XML 文件:
```

<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">  
  <constructor-arg index="0" ref="sqlSessionFactory" />  
  <constructor-arg index="1" value="BATCH" />  
</bean>  
这个 bean 现在可以直接注入到 DAO bean 中。你需要在 bean 中添加一个 SqlSession 属性,就像下面的代码:
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">  
  <property name="sqlSession" ref="sqlSession" />  
</bean>  
public class UserDaoImpl implements UserDao {  
  
  private SqlSession sqlSession;  
  
  public void setSqlSession(SqlSession sqlSession) {  
    this.sqlSession = sqlSession;  
  }  
  
  public User getUser(String userId) {  
    return (User) sqlSession.selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);  
  }  
}  
如果使用了批量操作的陪着孩子，下面这个也可以使用：
public void insertUsers(User[] users) {  
   for (User user : users) {  
     sqlSession.insert("org.mybatis.spring.sample.mapper.UserMapper.insertUser", user);  
   }  
 }  

```

##  SqlSessionDaoSupport 

SqlSessionDaoSupport 是 一 个 抽象 的支 持 类, 用来 为你 提供 SqlSession 。 调 用 getSqlSession()方法你会得到一个 SqlSessionTemplate,之后可以用于执行 SQL 方法, 就像下面这样:
```

public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {  
  public User getUser(String userId) {  
    return (User) getSqlSession().selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);  
  }  
}  

```
<WRAP center round tip 60%>
通常 MapperFactoryBean 是这个类的首选,因为它不需要额外的代码。但是,如果你 需要在 DAO 中做其它非 MyBatis 的工作或需要具体的类,那么这个类就很有用了。
</WRAP>

SqlSessionDaoSupport 需要一个 sqlSessionFactory 或 sqlSessionTemplate 属性来 设 置 。 这 些 被 明 确 地 设 置 或 由 Spring 来 自 动 装 配 。 如 果 两 者 都 被 设 置 了 , 那 么 SqlSessionFactory 是被忽略的。假设类 UserMapperImpl 是 SqlSessionDaoSupport 的子类,它可以在 Spring 中进行如 下的配置:
```

<bean id="userMapper" class="org.mybatis.spring.sample.mapper.UserDaoImpl">  
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />  
</bean>  

```
#  注入映射器MapperFactoryBean 
```

<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">  
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />  
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />  
</bean>  

```
MapperFactoryBean 创建的代理类实现了 UserMapper 接口,并且注入到应用程序中。 因为代理创建在运行时环境中(Runtime,译者注) ,那么` 指定的映射器必须是一个接口, `而 不是一个具体的实现类。
` 如果 UserMapper 有一个对应的 MyBatis 的 XML 映射器文件, 如果 XML 文件在类路径的 位置和映射器类相同时, 它会被 MapperFactoryBean 自动解析。 ` 没有必要在 MyBatis 配置文 件 中 去 指 定 映 射 器 ,` 除 非 映 射 器 的 XML 文 件 在 不 同 的 类 路 径 下 。 可 以 参 考 SqlSessionFactoryBean 的 configLocation 属性 `来获取更多信息。
可以直接在 business/service 对象中以和注入任意 Spring bean 的相同方式直接注入映 射器:
```

<bean id="fooService" class="org.mybatis.spring.sample.mapper.FooServiceImpl">  
  <property name="userMapper" ref="userMapper" />  
</bean>  
public class FooServiceImpl implements FooService {  
  
  private UserMapper userMapper;  
  
  public void setUserMapper(UserMapper userMapper) {  
    this.userMapper = userMapper;  
  }  
  
  public User doSomeBusinessStuff(String userId) {  
    return this.userMapper.getUser(userId);  
  }  
}  

```

##  MapperScannerConfigurer 

没有必要在 Spring 的 XML 配置文件中注册所有的映射器。相反,你可以使用一个 MapperScannerConfigurer , 它 将 会 查 找 类 路 径 下 的 映 射 器 并 自 动 将 它 们 创 建 成 MapperFactoryBean。
要创建 MapperScannerConfigurer,可以在 Spring 的配置中添加如下代码:
```

<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
  <property name="basePackage" value="org.mybatis.spring.sample.mapper" />  
</bean>  

```
basePackage 属性是让你为映射器接口文件设置基本的包路径。 你可以使用分号或逗号 作为分隔符设置多于一个的包路径。每个映射器将会在指定的包路径中递归地被搜索到。