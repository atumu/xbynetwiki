title: hibernate_session_transaction详解 

#  Hibernate Session & Transaction详解 
本文参考：http://blog.163.com/magicc_love/blog/static/1858536622012092478227/?suggestedreading&wumii
http://blog.163.com/magicc_love/blog/static/185853662201291610427970/?suggestedreading&wumii
HIbernate中的` Session `
　　Session是JAVA应用程序和Hibernate进行交互时使用的主要接口，它也是持久化操作核心API. ` Session不是线程安全的 `。
　　注意这里的Session的含义，它与传统意思上web层的HttpSession并没有关系，Hibernate Session之与Hibernate，相当于JDBC Connection相对与JDBC。 
　　Session对象是有生命周期的，它以Transaction对象的事务开始和结束边界 
　　Session作为贯穿Hibernate的持久化管理器核心，提供了众多的持久化的方法，如save(), update ,delete等，通过这些方法我们可以透明的完成对象的增删改查（` CRUD `-- create read update delete），这里所谓的透明是指，Session在读取，创建和删除影射的实体对象的实例时，这一系列的操作将被转换为对数据库表中数据的增加，修改，查询和删除操作。

` SessionFactory `负责创建Session，` SessionFactory是线程安全的 `，多个并发线程可以同时访问一个SessionFactory 并从中获取Session实例。而` Session并非线程安全 `，也就是说，如果多个线程同时使用一个Session实例进行数据存取，则将会导致Session 数据存取逻辑混乱.因此创建的Session实例必须在本地存取空上运行，使之总与当前的线程相关。


Session有以下的特点
　　1,` 不是线程安全的 `，应该避免多个线程共享同一个Session实例 
　　2,` Session实例是轻量级的 `，所谓轻量级：是指他的创建和删除不需要消耗太多资源。` SessionFactory不是轻量级的 `，作为全局变量保存
　　3,` Session对象内部有一个缓存 `，被称为Hibernate**第一缓存**，他存放被当前工作单元中加载的对象，**每个Session实例都有自己的缓存。**

##  保证session的线程安全 
` ThreadLocal `，在很多种Session 管理方案中都用到了它.ThreadLocal 是Java中一种较为特殊的线程绑定机制，通过ThreadLocal存取的数据，总是与当前线程相关，也就是说，JVM 为每个运行的线程，绑定了私有的本地实例存取空间，从而为多线程环境常出现的并发访问问题提供了一种隔离机制，ThreadLocal并不是线程本地化的实现，而是线程局部变量。

一个很实用的HibernateUtil类
```

public class HibernateUtil {  
  //SessionFactory作为重量级选手，应该对于整个应用来说保存为全局变量。
public static final SessionFactory sessionFactory;  
 //采用ThreadLocal模式的Session
public static final ThreadLocal session = new ThreadLocal();  
  //静态初始化SessionFactory
static{  
	try{  
		Configuration configuration=new Configuration().configure();   
		sessionFactory = configuration.buildSessionFactory();  
	}catch (Throwable ex){  
		System.err.println("Initial SessionFactory creation failed." + ex);  
		throw new ExceptionInInitializerError(ex);  
	}  
}  
 /**返回当前线程上下文相关的Session*/
public static Session currentSession() throws HibernateException{  
	Session s = (Session) session.get();  
	if (s == null)  
	{  
		s = sessionFactory.openSession();  
		session.set(s);  
	}  
	return s;  
}  
 
public static void closeSession() throws HibernateException {  
	Session s = (Session) session.get();  
	if (s != null)  
		s.close();  
		session.set(null);  
	}  
} 

```
不安全解释，例如：Servlet 运行是多线程的，而应用服务器并不会为每个线程都创建一个Servlet实例，也就是说，TestServlet在应用服务器中只有一个实例（在Tomcat中是这样，其他的应用服务器可能有不同的实现），而这个实例会被许多个线程并发调用，doGet 方法也将被不同的线程反复调用，可想而知，每次调用doGet 方法，这个唯一的TestServlet 实例的session 变量都会被重置

 在代码中，只要借助上面这个工具类获取Session 实例，我们就可以实现线程范围内的Session 共享，从而避免了在线程中频繁的创建和销毁Session 实例。不过注意在线程结束时关闭Session。同时值得一提的是，新版本的Hibernate在处理Session的时候已经内置了延迟加载机制，只有在真正发生数据库操作的时候，才会从数据库连接池获取数据库连接，我们不必过于担心Session的共享会导致整个线程生命周期内数据库连接被持续占用。

##  Hibernate Session缓存 

**Hibernate Session缓存被称为Hibernate的第一级缓存。SessionFactory的外置缓存称为Hibernate的二级缓存**。这两个缓存都位于持久层，它们存放的都是数据库数据的拷贝。SessionFactory的内置缓存 存放元数据和预定义SQL， SessionFactory的内置缓存是只读缓存。

**Hibernate Session缓存的三大作用：**
1，减少数据库的访问频率，提高访问性能。
2，保证缓存中的对象与数据库同步，位于缓存中的对象称为持久化对象。
3，当持久化对象之间存在关联时，Session 保证不出现对象图的死锁。
**Session 如何判断持久化对象的状态的改变呢？**
Session 加载对象后会为对象值类型的属性复制一份快照。当Session 清理缓存时，比较当前对象和它的快照就可以知道那些属性发生了变化。

**Session 什么时候清理缓存？**
1，**commit（）** 方法被调用时
2，**查询时**会清理缓存，保证查询结果能反映对象的最新状态。
3，显示的调用session 的 **flush**方法。
session 清理缓存的特例：
当对象使用 native 生成器 时 会立刻清理缓存向数据库中插入记录。

##  保证事务在发生异常时安全地回滚 

```

try{
 session=sessionFactory.openSession();
 tx.session.beginTransaction();
 ....
 tx.commit()
}catch(RuntimeException ex){//注意捕获的是RuntimeException
 try{
	tx.rollback();//回滚过程中也可能发生RuntimeException
 }catch(RuntimeException rex){
    log.error("can't roll back transaction ",rex);//将错误记录下来
 }
 throw ex; //记得再次抛出
}finally{
 session.close();//finally块中关闭session
}

```

##  Transaction 
` Transanction `接口是Hibernate的数据库事务接口，用于管理事务，他对底层的事务作出了封装，用户可以使用Transanction对象定义自己的对数据库的原子操作，底层事务包括：JDBC API ,JTA(Java Transaction API)。。。。。 
　　一个Transaction对象的事务可能会包括多个对数据库进行的操作 
　　org.hibernate Interface Transaction 
　　public interface Transaction
常用方法：
　　public void commit() throws HibernateException 刷新当前的Session以及结束事务的工作，这个方法将迫使数据库对当前的事务进行提交 
　　public void rollback() throws HibernateException ：强迫回滚当前事务 
　　public boolean isActive() throws HibernateException： 这个事务是否存活 

Session：当中包含一个` Connection `对象 
　　Connection c =session.getConnection(); 
　　Session的缓存用于临时保存持久化的对象，等到一定时候，再将缓存中的对象保存到数据库中。 
　　应用程序事务：如果一个Session中包含有多个Transaction（数据库事务）,这些Transaction的集合称为应用程序事务 

Hibernate不是盏省油的灯，也不是想像的射来射去很简单的事。有很多细节处理不好会让你很不舒服的，这方面最突出的表现在两方面：
  * 一是事务管理，是JTA事务还是JDBC事务？幸亏有了Spring和J2EE容器；
  * 二是胡乱映射，模型关系建立不合理或者错误导致，或者是映射策略和技术不过关导致。这样的最终结果是抛出一堆HibernateException，摸不着头脑
##  主要借口和方法、状态转换 

**1、Configuration/SessionFactory/Session**

Configuration实例代表了一个应用程序中Java类型 到SQL数据库映射的完整集合. Configuration被用来构建一个(不可变的 (immutable))SessionFactory.
**SessionFactory是线程安全的**，**创建代价很高**。
**Session是非线程安全的，轻量级的**。一个Session对应一个JDBC连接，
Session的connection()会获取Session与之对应的数据库连接Connection对象。
Session的功能就是操作对象的，这些对象和数据库表有映射关系。
Session操作的对象是有状态的，分三类：
  * 自由状态（transient）: 未持久化，未与任何Session相关联，数据库表中没有对应的记录。
  * 持久化状态（persistent）: 与一个Session相关联，对应数据库表中一条记录。
  * 游离状态（detached）: 已经进行过持久化，但当前未与任何Session相关联，数据库表中曾经有一条记录，现在还有没有就不知道了。

自由状态的实例可以通过调用save()、persist()或者saveOrUpdate()方法进行持久化。持久化实例可以通过调用 delete()变成自由状态。通过get()或load()方法得到的实例都是持久化状态的。游离状态的实例可以通过调用 ` update()、saveOrUpdate()、lock()或者replicate() `进行持久化。游离或者自由状态下的实例可以通过调用` merge() `方法成为一个**新的持久化实例**。

**2、Session的save()/persist()/update()/saveOrUpdate()/merge()/delete()方法**
save()方法将指定对象保存，插入表中一条数据；
persist()方法将指定对象保存，插入表中一条数据，和save()的区别是如果事务已经关闭，它永远也不会执行INSERT操作，而save方法将会产生额外的事务插入。不过persist方法无法保证马上执行INSERT操作，只有显示调用flush()的时候才能保证它已经执行。persist不会返回自动生成的ID，因为此时实体的ID属性可能还没有被设置。
replicate()方法完全使用给定对象各个属性的值（包括标识id）来持久化给定的游离状态（Transient）的实体，很暴力啊，其中还需要指定存储模式（有四种保存策略供选择）。
update()方法将指定对象更新，更新表中一条数据；如果实体已经被附着在会话上，将抛出异常。
saveOrUpdate()方法接收一个实体对象，根据实体对象的id判断是否已经存在进行保存或更新操作，这样保存和更新方法就统一了；
merge()方法将给定的对象的状态复制到具有相同标识的持久化对象上。与update的区别是无论实体是否已经被附在会话上，它都可以正常工作。**一般推荐使用merge()**
delete()方法将指定对象删除，删除表中一条数据；
flush()刷新会话，它会使挂起的语句立即执行，但是不会结束事务。可以多次被调用。不过关闭会话或者提交事务的时候将会自动执行刷新。需要刷新会话背后的原因是：Hibernate将会把语句添加到队列中，在一个涉及多个操作的事务中，操作实际上可能无法按照指定的顺序执行。在刷新时将保证此时的操作被立即执行。

特别注意：为了使用saveOrUpdate()方法，在由定义映射文件时，通过设定<id>标签的unsaved-value="null"来判断执行什么操作： 当id属性等于unsaved-value的值（在此为null）时，则认为还没有保存，应该执行保存操作，否则执行更新操作。这样设定之后，可以使用saveOrUpdate()方法来统一保存和更新的方法。
<id name="id" column="id" type="java.lang.Integer" unsaved-value="null">
<generator class="native"/>
</id>
unsaved-value可以设定的值有四个：
  * any：总是储存
  * none：总是更新
  * null：id为null时储存（预设）
  * valid：id为null或是指定值时储存


**3、Session的get()/load()方法**
` get() `立即加载，方法会总是查询实体对象，**不存在时候返回null；**
` load() `延迟加载，方法也是获取一个实体对象，**不存在时候抛空指针异常。**
当使用load方法来得到一个对象时，此时hibernate会使用延迟加载的机制来加载这个对象，即：当我们使用session.load()方法来加载一个对象时，此时并不会发出sql语句，当前得到的这个对象其实是一个代理对象，这个代理对象只保存了实体对象的id值，只有当我们要使用这个对象，得到其它属性时，这个时候才会发出sql语句，从数据库中去查询我们的对象。
**4、Session的clear()/evict()方法**
clear()方法清除Session级别缓存中的所有实体（包括各种状态）对象，目的是释放内存。
evict()方法清除Session级别缓存中的指定的实体（包括各种状态）对象。
当然，Session关闭后，这些缓存也就不存在了，会等待JVM回收。

**5、Session的flush()方法**
flush()强制持久化Session缓存中的实体对象。一般还会调用clear()或evict()，目的是赶紧保存，释放宝贵内存资源。

**6、Session的commit()/rollback()方法**
commit()方法用于提交Session上的事务，否则工作单元不会对数据库产生影响。如果执行出现异常（也就是commit()失败了），则之前的操作取消，执行rollback()可撤消之前的操作。

**7、Session的close()/isOpen()/isConnected()/reconnect()方法**
close()方法关闭Session所对应数据库连接，与其相关联的对象生命周期结束。
isOpen()方法检查Session是否仍然打开，如果Session已经断开，则可以使用reconnect(Connection connection)来重新让Session关联一个JDBC连接。
isConnected()方法检查当前Session是否处于连接状态。

**8、Criteria、DetchedCriteria和Query接口**
Criteria和Query的实例都是和Session绑定的，其生命周期跟随着Session结束而结束。
DetchedCriteria实例相当于一个SQL模板，目的是为了复用。其中的getExecutableCriteria(session)方法接收一个Session对象，并与之绑定，返回一个Criteria对象。

**9、Hibernate类的initialize()方法**
initialize()方法强制Hibernate立即加载指定实体所关联的对象和集合。Hibernate类中还有其他几个很有用但不适很常用的方法。

**10、映射文件中的lazy属性**
在Hibernate3中，class元素的lazy属性默认是true，如果不需要，则需要显示指定为lazy="false"，否则，操作load返回的对象会抛异常。另外Hibernate3中还可以为实体属性指定lazy属性。

##  JDBC事务和JTA事务 
Hibernate本身没有事务管理功能，它依赖于JDBC或JTA的事务管理功能，在Hibernate配置文件中，如果不显式指定Transaction的工厂类别属性hibernate.transaction.factory_class的配置，**则默认为JDBC事务：**
```

<property name="hibernate.transaction.factory_class">org.hibernate.transaction.JDBCTransactionFactory</property>。

```
在通过SessionFactory获取到Session后，与Session相关联的JDBC Connection实例就被设定为false。
特别注意：如果数据库不支持事务，比如MySQL的MyISAM引擎的表就不支持事务，声明事务也不会起作用。要使MySQL5的表支持事务，则可以指定表的引擎类型为InnoDB。如果是学习或者研究，目前最好还是使用PostgreSQL 8.3或DB2、Oracle。
**JDBC事务总是和一个数据库连接（或一个Session）相关联的。**
**JTA事务则可以跨越多个数据连接（或多个Session），这些连接还可以是不同数据库的连接，JTA事务一般由容器进行管理。编程只要在多个操作单元的开始和结束定义JTA事务的边界即可。**
特别注意：如果使用了JTA事务，则不能再用在JDBC式的事务来管理每个Session的操作，否则会出错。为了程序的的通用性，**一般来说，都是使用JTA事务来构建应用，这使用任何环境。**当然，也可以使用事务代理为每个JDBC的操作方法加入事务控制。这样也为程序以后移植到JTA容器事务上带来很大方便。其实现在可以使用Spring的事务管理，与Hibernate结合的非常完美。