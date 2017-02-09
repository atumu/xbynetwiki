title: java_api 

#  mybatis:java API 

#  应用目录结构 
```

/my_application  
  /bin  
  /devlib  
  /lib                <-- MyBatis *.jar文件在这里。  
  /src  
    /org/myapp/  
      /action  
      /data           <-- MyBatis配置文件在这里, 包括映射器类, XML配置, XML映射文件。  
        /mybatis-config.xml  
        /BlogMapper.java  
        /BlogMapper.xml  
      /model  
      /service  
      /view  
    /properties       <-- 在你XML中配置的属性 文件在这里。  
  /test  
    /org/myapp/  
      /action  
      /data  
      /model  
      /service  
      /view  
    /properties  
  /web  
    /WEB-INF  
      /web.xml  

```
#  SqlSessionFactoryBuilder 

#  SqlSessionFactory 
SqlSessionFactory 有六个方法可以用来创建 SqlSession 实例。通常来说,如何决定是你 选择下面这些方法时:

Transaction (事务): 你想为 session 使用事务或者使用自动提交(通常意味着很多 数据库和/或 JDBC 驱动没有事务)?
Connection (连接): 你想 MyBatis 获得来自配置的数据源的连接还是提供你自己
Execution (执行): 你想 MyBatis 复用预处理语句和/或批量更新语句(包括插入和 删除)?
```

SqlSession openSession()  
SqlSession openSession(boolean autoCommit)  
SqlSession openSession(Connection connection)  
SqlSession openSession(TransactionIsolationLevel level)  
SqlSession openSession(ExecutorType execType,TransactionIsolationLevel level)  
SqlSession openSession(ExecutorType execType)  
SqlSession openSession(ExecutorType execType, boolean autoCommit)  
SqlSession openSession(ExecutorType execType, Connection connection)  
Configuration getConfiguration();  

```
默认的 openSession()方法没有参数,它会创建有如下特性的 SqlSession:

  * 会开启一个事务(也就是不自动提交)
  * 连接对象会从由活动环境配置的数据源实例中得到。
  * 事务隔离级别将会使用驱动或数据源的默认设置。
  * 预处理语句不会被复用,也不会批量处理更新。
还有一个可能对你来说是新见到的参数,就是 ExecutorType。这个枚举类型定义了 3 个 值:

  * ExecutorType.SIMPLE: 这个执行器类型不做特殊的事情。它为每个语句的执行创建一个新的预处理语句。
  * ExecutorType.REUSE: 这个执行器类型会复用预处理语句。
  * ExecutorType.BATCH: 这个执行器会批量执行所有更新语句,如果 SELECT 在它们中间执行还会标定它们是 必须的,来保证一个简单并易于理解的行为。
注意 在 SqlSessionFactory 中还有一个方法我们没有提及,就是 getConfiguration()。这 个方法会返回一个 Configuration 实例,在运行时你可以使用它来自检 MyBatis 的配置。

#  SqlSession 

使用 MyBatis 的主要 Java 接口就是 SqlSession。你可以使用这个接口执行命令,获 取映射器和管理事务。SqlSessions 是由 SqlSessionFactory 实例创建的。SqlSessionFactory 对 象 包 含 创 建 SqlSession 实 例 的 所 有 方 法 。 而 SqlSessionFactory 本 身 是 由 SqlSessionFactoryBuilder 创建的,它可以从 XML 配置,注解或手动配置 Java 来创建 SqlSessionFactory。
<note important>注意：如果与Spring结合时, SqlSessions are created and injected by the DI framework so you don't need to use the SqlSessionFactoryBuilder or SqlSessionFactory and can go directly to the SqlSession section.故而此时需要使用Mybatis-Spring这个项目。</note>

##  语句执行方法 

这些方法被用来执行定义在 SQL 映射的 XML 文件中的 SELECT,INSERT,UPDA E T 和 DELETE 语句。
它们都会自行解释,每一句都使用语句的 ID 属性和参数对象,` 参数可以 是原生类型(自动装箱或包装类) ,JavaBean,POJO 或 Map。 `
```

<T> T selectOne(String statement, Object parameter)  
<E> List<E> selectList(String statement, Object parameter)  
<K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey)  
int insert(String statement, Object parameter)  
int update(String statement, Object parameter)  
int delete(String statement, Object parameter)  

```
**selectOne 和 selectList 的不同仅仅是 selectOne 必须返回一个对象。** 如果多余一个, 或者 没有返回 (或返回了 null) 那么就会抛出异常。  
**如果你不知道需要多少对象, 使用 selectList。**

如果你想检查一个对象是否存在,那么最好返回统计数(0 或 1) 。因为并不是所有语句都需 要参数,这些方法都是有不同重载版本的,它们可以不需要参数对象。
  * <T> T selectOne(String statement)
  * <E> List<E> selectList(String statement)
  * <K,V> Map<K,V> selectMap(String statement, String mapKey)
  * int insert(String statement)
  * int update(String statement)
  * int delete(String statement)

最后,还有查询方法的三个高级版本,它们` 允许你限制返回行数的范围,或者提供自定 义结果控制逻辑, `这通常用于大量的数据集合。
```

<E> List<E> selectList (String statement, Object parameter, RowBounds rowBounds)  
<K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey, RowBounds rowbounds)  
void select (String statement, Object parameter, ResultHandler handler)  
void select (String statement, Object parameter, RowBounds rowBounds, ResultHandler handler)  

```
**RowBounds 参数会告诉 MyBatis 略过指定数量的记录,还有限制返回结果的数量。** RowBounds 类有一个构造方法来接收 offset 和 limit,否则是不可改变的。
int offset = 100;  
int limit = 25;  
RowBounds rowBounds = new RowBounds(offset, limit);  
不同的驱动会实现这方面的不同级别的效率。对于最佳的表现,使用结果集类型的 SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE(或句话说:不是 FORWARD_ONLY)。
**ResultHandler 参数允许你按你喜欢的方式处理每一行。** 
它的接口很简单。
```

package org.apache.ibatis.session;
public interface ResultHandler<T> {
  void handleResult(ResultContext<? extends T> context);
}

```
ResultContext 参数给你访问结果对象本身的方法, 大量结果对象被创建, 你可以使用布 尔返回值的 stop()方法来停止 MyBatis 加载更多的结果。

**` Batch update批量更新 `**
当时使用ExecutorType.BATCH as ExecutorType获取sqlsession
**Batch update statement Flush Method**
There is method for flushing(executing) batch update statements that stored in a JDBC driver class at any timing. This method can be used when you use the ExecutorType.BATCH as ExecutorType.
**List<BatchResult> flushStatements()**
```

public void batchUpdate(String str, List objs )throws Exception{
		SqlSessionFactory sqlSessionFactory = sqlSessionTemplate.getSqlSessionFactory();
		//批量执行器
		SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH,false);//
		try{
			if(objs!=null){
				for(int i=0,size=objs.size();i<size;i++){
					sqlSession.update(str, objs.get(i));
				}
				sqlSession.flushStatements(); //
				sqlSession.commit();//
				sqlSession.clearCache();//
			}
		}finally{
			sqlSession.close();
		}
	}

```


##  事务控制方法 
```

void commit()  
void commit(boolean force)  
void rollback()  
void rollback(boolean force)  

```
默认情况下 MyBatis 不会自动提交事务, 除非它侦测到有插入, 更新或删除操作改变了 数据库。如果你已经做出了一些改变而没有使用这些方法,那么你可以传递 true 到 commit 和 rollback 方法来保证它会被提交(注意,你不能在自动提交模式下强制 session,或者使用 了外部事务管理器时) 。
` 注意：Mybatis-Spring采用声明式的事务处理。所以与Spring集成时查看MyBatis-Spring文档 `
**清理 Session 级的缓存**
void clearCache()
SqlSession 实例有一个本地缓存在执行 update,commit,rollback 和 close 时被清理。要 明确地关闭它(获取打算做更多的工作) ,你可以调用 clearCache()。
**确保 SqlSession 被关闭**
void close()
` 确保 SqlSession 被关闭 `
```

SqlSession session = sqlSessionFactory.openSession();  
try {  
    // following 3 lines pseudocod for "doing some work"  
    session.insert(...);  
    session.update(...);  
    session.delete(...);  
    session.commit();  
} finally {  
    session.close();  
}  
使用映射器
public interface AuthorMapper {  
  // (Author) selectOne("selectAuthor",5);  
  Author selectAuthor(int id);   
  // (List<Author>) selectList(“selectAuthors”)  
  List<Author> selectAuthors();  
  // (Map<Integer,Author>) selectMap("selectAuthors", "id")  
  @MapKey("id")  
  Map<Integer, Author> selectAuthors();  
  // insert("insertAuthor", author)  
  int insertAuthor(Author author);  
  // updateAuthor("updateAuthor", author)  
  int updateAuthor(Author author);  
  // delete("deleteAuthor",5)  
  int deleteAuthor(int id);  
}  

```
#  映射器注解 

` 查看官方文档http://www.mybatis.org/mybatis-3/zh/java-api.html `