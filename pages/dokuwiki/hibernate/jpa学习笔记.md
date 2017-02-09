title: jpa学习笔记 

#  JPA学习笔记 
参考：http://docs.oracle.com/cd/E11035_01/kodo41/full/html/ejb3_langref.html
http://blog.csdn.net/chjttony/article/details/6086305
http://www.cnblogs.com/holbrook/archive/2012/12/30/2839842.html

本wiki还有一篇[JPA介绍文章](/pages/dokuwiki/database/jpa学习笔记):
JPA（Java Persistence API，Java持久化API），定义了对象-关系映射（ORM）以及实体对象持久化的标准接口。JPA是JSR-220（EJB3.0）规范的一部分
JPA维护一个Persistence Context（持久化上下文），在持久化上下文中维护实体的生命周期。主要包含三个方面的内容：
  * ORM元数据。JPA支持annotion或xml两种形式描述对象-关系映射。
  * 实体操作API。实现对实体对象的CRUD操作。
  * 查询语言。约定了面向对象的查询语言JPQL（Java Persistence Query Language）。

^org.hibernate	^javax.persistence	^说明^
|Configuration	|Persistence	|读取配置信息|
|SessionFactory	|EntityManagerFactory	|用于创建会话/实体管理器的工厂类|
|Session	EntityManager	|提供实体操作API，|管理事务，创建查询|
|Transaction	|EntityTransaction	|管理事务|
|Query	|Query	|执行查询|
|save、update、saveOrUpdate、persist|persist|持久化，涉及级联时hibernate的一些方法会有区别|
|delete|remove|先加载后移除|
|get、load|find、getReference|加载,前者可能返回null,立即加载，后者可能报异常，延迟加载|
|merge、update|merge|持久化托管对象|
|flush|flush|清除持久化上下文缓存|

![](/data/dokuwiki/hibernate/pasted/20150909-153815.png)

Maven：
```

 <dependency>
            <groupId>org.eclipse.persistence</groupId>
            <artifactId>javax.persistence</artifactId>
            <version>2.1.0</version>
            <scope>compile</scope>
        </dependency>

        <dependency>
            <groupId>javax.transaction</groupId>
            <artifactId>javax.transaction-api</artifactId>
            <version>1.2</version>
            <scope>compile</scope>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>4.3.6.Final</version>
            <scope>runtime</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.hibernate.javax.persistence</groupId>
                    <artifactId>hibernate-jpa-2.1-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.jboss.spec.javax.transaction</groupId>
                    <artifactId>jboss-transaction-api_1.2_spec</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
   

```
**2.JPA的持久化策略文件：**
根据JPA规范要求，在实体bean应用中，需要在应用` 类路径(classpath)的META-INF目录下 `加入持久化配置文件——` persistence.xml `，该文件就是持久化策略文件。其内容如下：
```

<?xml version="1.0" encoding="UTF-8"?>  
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
        version="2.0">
  <persistence-unit name="持久化单元名" transaction-type="RESOURCE_LOCAL">  <!--可以有多个持久化单元 transaction-type表示持久化单元将使用Java Transaction API(JTA)事务或者本地事务RESOURCE_LOCAL-->
      <non-jta-data-source>使用数据源JNDI</non-jta-data-source>  <!--当transaction-type为JTA时才能使用jta-data-source元素，如果为RESOURCE_LOCAL时，替换为non-jta-data-source -->
        <!--指定JPA javax.persistence.spi.PersistenceProvider实现提供者，不指定就采用JavaEE服务器默认的JPA提供者，这里以Hibernate为例-->  
	<provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>  
	<class>实体bean全路径名</class>  
	……  
    <properties>  
              <!--配置标准属性以及JPA实现提供者的一些信息-->  
    </properties>  
    <mapping-file> <!--可以指定多个 -->
    </mapping-file>
  </persistence-unit>  
</persistence>  

```
注意：可以在持久化策略文件中配置多个持久化单元persistence-unit，在使用分布式数据库时经常配置多个持久化单元。

##  2 实体生命周期 
实体生命周期是JPA中非常重要的概念，描述了实体对象从创建到受控、从删除到游离的状态变换。对实体的操作主要就是改变实体的状态。
JPA中实体的生命周期如下图：
![](/data/dokuwiki/hibernate/pasted/20150909-153841.png)
  * New，新创建的实体对象，没有主键(identity)值
  * Managed，对象处于Persistence Context(持久化上下文）中，被EntityManager管理
  * Detached，对象已经游离到Persistence Context之外，进入Application Domain
  * Removed, 实体对象被删除。
` EntityManager `提供一系列的方法管理实体对象的生命周期，包括：（` 注意，数据库中实体的唯一性标识为表的ID字段 `）
  * ` find `查找：相当于Hibernate中的get操作,` 如果查找不到会返回null `。根据**主键**从数据库中查询一个实体，这个方法首先从缓存中去查找，如果找不到，就从数据库中去找，并把它加入到缓存中。 
  * ` getReference `查找：相当于Hibernate中的load操作，如果一级缓存查找不到并不立即去数据库查找，而是去查找二级缓存，返回实体对象的代理，` 如果查找不到会抛出ObjectNotFindException。 `
  * ` persist `, 使实体类从new状态或者removed转变到managed状态，并将数据保存到底层数据库中。**如果实体已存在，则抛出EntityExistsException异常**，缓存则不存在了。 
  * ` remove `，将实体变为removed状态，当实体管理器关闭或者刷新时，才会真正地删除数据。根据主键从数据库中删除一个对象，**这个对象的状态必须是managed，否则会抛出IllegalArgumentException,**
  * ` flush `：将实体和底层数据库进行同步，当调用persist、merge或者remove方法时，更新并不会立刻同步到数据库中，直到容器决定刷新到数据库中时才会执行，可以调用flush强制刷新。
  *`  merge `，将**游离实体转变为Managed状态**，数据存入数据库。把一个对象加入到当前的持久化上下文中，就是把一个对象从detach转变为managed，并返回这个对象。当一个对象设置了主键，并调用此方法，就会从数据库中根据主键查找到该对象把它放到持久化上下文中，当事物提交的时候，如果对象发生了改变，更新该对象的改变到数据库中，如果对象没有改变，则什么也不做，**如果对象没有设置主键，则插入该对象到数据库中。** (数据库中对象是否已存在是` 根据主键 `判断)
  * ` createQuery `：根据JPA QL定义查询对象。它还可以执行insert，update等。
  * ` createNativeQuery `：允许开发人员根据特定数据库的SQL语法来进行查询操作，只有JPA QL不能满足要求时才使用，不推荐使用，因为增加了程序可移植性。
  * ` createNamedQuery `：根据实体中标注的命名查询创建查询对象。

**detach()、remove()、em.createQuery("DELETE FROM Country").exceuteUpdate()的区别：**
参考：http://stackoverflow.com/questions/18514455/difference-between-detach-and-remove-entitymanagers-methods
```

void detach(java.lang.Object entity)
Remove the given entity from the persistence context, causing a managed entity to become detached.从持久化上下文，以分离的管理实体中删除给定的实体。于该实体的更改（如果有）（包括除去实体的），将不会同步到数据库。其中引用的分离实体将继续实体引用它。

void remove(java.lang.Object entity)
Remove the entity instance. The database is affected right away.

em.createQuery("DELETE FROM Country").exceuteUpdate();
Does the Delete directly to database, if you have that object, for example, saved in any list or it is a simple referenced object, it wont get the changes, and surely raise an error if you try to merge, or do something with that.尽量不要做一个删除这个样子，至少它是你最后的选择。

```

如果使用了事务管理，则事务的` commit/rollback `也会改变实体的状态。
```

@WebServlet(
        name = "entityServlet",
        urlPatterns = "/entities",
        loadOnStartup = 1
)
public class EntityServlet extends HttpServlet
{
    private final Random random;
    private EntityManagerFactory factory;

    public EntityServlet()
    {
        try
        {
            this.random = SecureRandom.getInstanceStrong();
        }
        catch(NoSuchAlgorithmException e)
        {
            throw new IllegalStateException(e);
        }
    }

    @Override
    public void init() throws ServletException
    {
        super.init();
        this.factory = Persistence.createEntityManagerFactory("EntityMappings");
    }

    @Override
    public void destroy()
    {
        super.destroy();
        this.factory.close();
    }

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
    {
        EntityManager manager = null;
        EntityTransaction transaction = null;
        try
        {
            manager = this.factory.createEntityManager();
            transaction = manager.getTransaction();
            transaction.begin();

            CriteriaBuilder builder = manager.getCriteriaBuilder();

            CriteriaQuery<Publisher> q1 = builder.createQuery(Publisher.class);
            request.setAttribute("publishers", manager.createQuery(
                    q1.select(q1.from(Publisher.class))
            ).getResultList());

            CriteriaQuery<Author> q2 = builder.createQuery(Author.class);
            request.setAttribute("authors", manager.createQuery(
                    q2.select(q2.from(Author.class))
            ).getResultList());

            CriteriaQuery<Book> q3 = builder.createQuery(Book.class);
            request.setAttribute("books", manager.createQuery(
                    q3.select(q3.from(Book.class))
            ).getResultList());

            transaction.commit();

            request.getRequestDispatcher("/WEB-INF/jsp/view/entities.jsp")
                    .forward(request, response);
        }
        catch(Exception e)
        {
            if(transaction != null && transaction.isActive())
                transaction.rollback();
            e.printStackTrace(response.getWriter());
        }
        finally
        {
            if(manager != null && manager.isOpen())
                manager.close();
        }
    }


```

##  3.Spring集成JPA： 
JPA不但可以在JavaEE环境中使用，也可以还JavaSE环境中使用，spring现在在java开发中应用非常广泛，Spring同样提供了对JPA的强大支持。Spring集成JPA的步骤如下：
(1).在classpath的MATE-INF下添加JPA的持久化策略文件persistence.xml，开发实体bean。
(2).为工程引入Spring支持。添加Spring相关依赖包和spring配置文件。
(3).在spring配置文件中加入JPA配置如下：
```

<bean id=”entityManagerFactory”
	class=”org.springframework.orm.jpa.LocalEntityManagerFactoryBean”>
<property name=”persistenceUnitName” value=”持久化单元名称”/>
</bean>
<bean id=”transactionManager”
	class=”org.springframework.orm.jpa.JpaTransactionManager”>
<property name=”entityManagerFactory” ref=” entityManagerFactory”/>
</bean>

<bean class=
  "org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/> 
<tx:annotation-driven transaction-manager=” transactionManager”/>

```
至此，Spring和JPA的简单集成完成。
```

@Repository("spitterDao")
@Transactional
public class JpaSpitterDao implements SpitterDao {
  private static final String RECENT_SPITTLES = 
      "SELECT s FROM Spittle s";
  private static final String ALL_SPITTERS =
      "SELECT s FROM Spitter s";
  private static final String SPITTER_FOR_USERNAME = 
      "SELECT s FROM Spitter s WHERE s.username = :username";
  private static final String SPITTLES_BY_USERNAME = 
      "SELECT s FROM Spittle s WHERE s.spitter.username = :username";

  @PersistenceContext //@PersistenceContext注解表明需要将一个EntityManager实例注入到em上，为了在Spring中实现EntityManager注入，我们需要在Spring应用上下文中配置一个<bean class=
//  "org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/> 
  private EntityManager em;

```
##  5.JPA分页查询： 
(1).为实体管理器创建的查询对象设置返回最大结果数：` setMaxResults(int maxResult); `
(2).为实体管理器创建的查询对象设置返回结果开始下标：` setFirstResult(int firstResult); `

##  JPQL查询语法 
JPA提供两种**查询方式**:
一种是**根据主键查询**，使用EntityManager的find方法：
```

T find(Class entityClass, Object primaryKey)

```
另一种就是使用**JPQL查询语言**。JPQL是完全面向对象的，具备继承、多态和关联等特性，和hibernate HQL很相似。
使用EntityManager的createQuery方法：
```

Query createQuery(String qlString)

```

JPA的**命名参数**：语法：“:参数名”。
```

Query query = entityManager.createQuery(”select u from User u where u.name=:name”);
//设置查询中的命名参数
Query.setParameter(“name”, testName);

```
注意：不允许在同一个查询中使用两个相同名字的命名参数。

JPA的**位置参数**：语法：“？位置参数索引”。
```

Query query = entityManager.createQuery(”select u from User u where u.name=?1”);
//设置查询中的位置参数
Query.setParameter(1, testName);

```
**注意：JPA中的位置参数索引值从` 1 `开始。**

JPA的**集合查询**：
我们知道在SQL语句中，查找某元素是否在某个集合中时常用类似：` in(a,b,c) `的查询语句，在JDBC中我们常常需要将集合元素拼接成以逗号(“,”)分隔的字符串，在JPA中可以方便地进行判断元素是否在集合中的查询：
设置集合类型的参数：
查询对象的setParameter方法支持Object类型，因此可以传入一个集合类型的参数，如数组或者ArrayList等。
```

Query query = em.createQuery(“select * fromUsers where name in(?names)”);
query.setParameter(“name”, names);//names是一个name集合

```

**5.3 排序**
JPQL也支持排序，类似于SQL中的语法。例如： ` Query query = em.createQuery("select p from Person p order by p.age, p.birthday desc"); `

**5.4 聚合查询**
JPQL支持` AVG、SUM、COUNT、MAX、MIN `五个聚合函数。例如：
```

Query query = em.createQuery("select max(p.age) from Person p"); 
Object result = query.getSingleResult(); 
String maxAge = result.toString();

```

**5.5 更新和删除**
JPQL不仅用于查询，还可以用于批量更新和删除。
如
```

Query query = em.createQuery("update Order as o set o.amount=o.amount+10"); 
int result = query.executeUpdate();//update 的记录数

Query query = em.createQuery("delete from OrderItem item where item.order in(from Order as o where o.amount<100)"); 
query.executeUpdate();

query = em.createQuery("delete from Order as o where o.amount<100"); 
query.executeUpdate();//delete的记录数

```

**5.6具名查询**
如果**某个JPQL语句需要在多个地方使用**，还可以使用` @NamedQuery ` 或者 ` @NamedQueries `**在实体对象上预定义命名查询。**
在需要调用的地方只要引用该查询的名字即可。
例如
```

@NamedQuery(name="getPerson", query= "FROM Person WHERE personid=?1")
@NamedQueries({ @NamedQuery(name="getPerson1", query= "FROM Person WHERE personid=?1"), @NamedQuery(name="getPersonList", query= "FROM Person WHERE age>?1") })
Query query = em.createNamedQuery("getPerson");

```
##  JPA QL标准函数 
^ 函数                                    ^ 适用性  ^
| UPPER(s),LOWER(s)                           ||
| CONCAT(s1,s2)                               ||
| SUBSTRING(s,offset,length)                  ||
| TRIM()                                      ||
| LENGTH(s)                                   ||
| LOCATE(search,s,offset)                     ||
| ABS(s),SQRT(s),MOD(dividend,divisor)        ||
| SIZE( c )                                   ||
##  3 实体关系映射（ORM） 
**3.1 基本映射**
^对象端	^数据库端	^annotion	^可选annotion^
|Class	|Table	|@Entity	|@Table(name="tablename")|
|property	|column	|–	|@Column(name = "columnname")|
|property	|primary key	|@Id	|@GeneratedValue 详见ID生成策略|
|property	|NONE	|@Transient|	 
**3.2 ID生成策略**
ID对应数据库表的主键，是保证唯一性的重要属性。JPA提供了以下几种ID生成策略
  * GeneratorType.AUTO ，由JPA自动生成
  * GenerationType.IDENTITY，使用数据库的自增长字段，需要数据库的支持（如SQL Server、MySQL、DB2、Derby等）
  * GenerationType.SEQUENCE，使用数据库的序列号，需要数据库的支持（如Oracle）
  * GenerationType.TABLE，使用指定的数据库表记录ID的增长 需要定义一个TableGenerator，在@GeneratedValue中引用。例如：
@TableGenerator( name="myGenerator", table="GENERATORTABLE", pkColumnName = "ENTITYNAME", pkColumnValue="MyEntity", valueColumnName = "PKVALUE", allocationSize=1 )
@GeneratedValue(strategy = GenerationType.TABLE,generator="myGenerator")
```

@Entity(name = "PublisherEntity")
@Table(name = "Publishers", indexes = {
        @Index(name = "Publishers_Names", columnList = "PublisherName")
})
public class Publisher implements Serializable
{
    private long id;
    private String name;
    private String address;

    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
            generator = "PublisherGenerator")
    @TableGenerator(name = "PublisherGenerator", table = "SurrogateKeys",
            pkColumnName = "TableName", pkColumnValue = "Publishers",
            valueColumnName = "KeyValue", initialValue = 11923,
            allocationSize = 1)
    @Column(name = "PublisherId")
    public long getId()
    {
        return this.id;
    }


```
**3.3 关联关系**
JPA定义了one-to-one、one-to-many、many-to-one、many-to-many 4种关系。
对于数据库来说，通常在一个表中记录对另一个表的**外键关联**；对应到实体对象，**持有关联数据的一方称为owning-side，另一方称为inverse-side。**
为了编程的方便，我们经常会希望在inverse-side也能引用到owning-side的对象，此时就构建了双向关联关系。** 在双向关联中，需要在inverse-side定义mappedBy属性.**

**3.4延迟加载与级联**
` 关联关系还可以定制延迟加载和级联操作的行为（ `owning-side和inverse-side可以分别设置）：
通过设置` fetch=FetchType.LAZY ` 或 fetch=FetchType.EAGER来决定关联对象是延迟加载或立即加载。
通过设置` cascade={options} `可以设置级联操作的行为，其中options可以是以下组合：
  * CascadeType.MERGE 级联更新
  * CascadeType.PERSIST 级联保存
  * CascadeType.REFRESH 级联刷新
  * CascadeType.REMOVE 级联删除
  * CascadeType.ALL 级联上述4种操作

**3.5 继承关系**
JPA通过在**父类**增加` @Inheritance(strategy=InheritanceType.xxx) `来声明继承关系。A支持3种**继承策略：**
  * 单表继承（InheritanceType.SINGLETABLE），所有继承树上的类共用一张表，在父类指定（` @DiscriminatorColumn `）声明并在每个类指定` @DiscriminatorValue `来区分类型。
  * 类表继承（InheritanceType.JOINED），父子类共同的部分公用一张表，其余部分保存到各自的表，通过` join `进行关联。
  * 具体表继承（InheritanceType.TABLEPERCLASS)，每个具体类映射到自己的表。
其中1和2能够支持多态，但是1需要允许字段为NULL，2需要多个JOIN关系；3最适合关系数据库，对多态支持不好。具体应用时根据需要取舍。

**3.5枚举类型作为实体属性**
```

public enum Gender
{
    MALE,
    FEMALE,
    UNSPECIFIED
}


```
默认情况下，映射到库变成了0、1、2.解决办法如下：
```

 @Enumerated(EnumType.STRING)
    public Gender getGender()
    {
        return this.gender;
    }

```

**3.6将大属性映射为CLOB，BLOB**
LOB可以存储数百万甚至数十亿的字节。
```

    @Lob
    public String getPreviewHtml()
    {
        return previewHtml;
    }

```
##  4 事件及监听 
![](/data/dokuwiki/hibernate/pasted/20150909-155022.png)
通过在实体的方法上标注@PrePersist，@PostPersist等声明即可在事件发生时触发这些方法。
JPA中，实体类支持一些回调方法，可以通过如下的注解指定回调方法：（注意：这些注解不支持Hibernate的Session,只支持JPA的EntityManager）
(1).@PrePersist：在persist方法调用后立刻发生，级联保存也会发生该事件，此时数据还没有真实插入数据库中。
(2).@PostPersist：数据已经插入进数据库中后触发。
(3).@PreRemove：在实体从数据库删除之前触发。级联删除也会触发该事件，此时数据还没有真实从数据库中删除。
(4).@PostRemove：在实体已经从数据库中删除后触发。
(5).@PreUpdate：在实体的状态同步到数据库之前触发，此时数据还没有真实更新到数据库中。
(6).@PostUpdate：在实体的状态同步到数据库后触发，同步在事务提交时发生。
(7).@PostLoad：在以下情况触发：a.执行find或者getReference方法载入一个实体之后。b.执行JPA QL查询之后。c.refresh方法被调用之后。
##  6 事务管理 
JPA支持**本地事务管理（RESOURCELOCAL）和容器事务管理（JTA）**，**容器事务管理只能用在EJB/Web容器环境中。**
事务管理的类型可以在` persistence.xml文件中的“transaction-type” `元素配置。
JPA中通过` EntityManager的getTransaction() `方法获取事务的实例（EntityTransaction），之后可以调用事务的` begin()、commit()、rollback() `方法。

**1、事务管理特性：**
事务所数据库的一个逻辑工作单元，包含一系列的操作，事务有四个特性，即` ACID特性 `：
(1).` Atomic(原子性) `：事务中的各个操作不可分割，事务中所包含的操作被看作一个逻辑单元，这个逻辑单元中的操作要么全部成功，要么全部失败。
(2).` Consistency(一致性) `：一致性意味着，只有合法的数据才可以被写入数据库，如果数据有任何不符合，则事务应该将其回滚。
(3).` Isolation(隔离性) `：事务允许多个用户对同一个数据的并发访问，而不破坏数据的正确性和完整性。同时，并行事务的修改必须与其他并行事务的修改相互独立。按照比较严格隔离逻辑来讲，一个事务看到的数据要么是另外一个事务修改这些数据之前的状态，要么是一个事务已经修改完成的数据，决不能是其他事务正在修改的数据。
(4).` Durability(持久性) `：事务结束后，事务处理的结果必须能够得到持久化。

**2.数据操作的3中不确定情况：**
(1).脏读取：一个事务读取了另一个并行事务未提交的数据。
(2).不可重复读：一个事务再次读取之前曾经读过的数据时，发现该数据已经被另一个已提交的事务修改。
(3).幻读：同一查询在同一事务中多次进行，由于其他事务提交所做的操作，使得每次返回不同的结果集，从而产生换读。

**3.事务的隔离级别：**
事务的隔离，是指数据库(或其他事务系统)通过某种机制，在并行的多个事务之间进行分割，使得每个事务在其执行过程中保持独立，如同当前只有一个事务在单独运行。为了避免12中3中不确定情况的发生，标准的SQL规范中定义了如下4中事务隔离级别：
(1).为提交读：最低等级的事务隔离，仅仅保证了读取过程中不会读取到非法数据。
(2).已提交读：该级别的事务隔离保证了一个事务不会读取到另一个并行事务已修改但未提交的数据，避免了脏读。大多数主流数据库默认的事务隔离等级是已提交读。
(3).重复读：该级别的事务隔离避免了脏读和不可重复读，但是也意味着，一个事务部可能更新已经由另一个事务读取但未提交的数据。
(4).序列化：最高等级的事务隔离，也提供了最严格的隔离机制，上面3种不确定情况都可以被避免。该级别将模拟事务的串行执行，逻辑上如同所有的事务都处于一个执行队列中，依次串行执行，而非并行执行。
4种事务隔离级别总结：
隔离等级              脏读              不可重复读           幻读
未提交读              可能              可能                     可能
提交读                  不可能           可能                     可能
重复读                  不可能           不可能                  可能
序列化                  不可能           不可能                  不可能

**4、JDBC事务和JTA事务：**
  * JDBC事务：**只能支持一个数据库，单数据源**，一个由数据库本身来执行提交或回滚，单阶段提交，本地事务。**JDBC事务由Connection管理**，事务周期局限于Connection的生命周期之内。
  * JTA事务：**JTA提供了跨Session的事务管理能力，支持多数据源的分布式事务**，两阶段提交。**JTA事务管理由JTA容器实现**，JTA容器对当前加入事务的众多Connection进行调度，实现其事务性要求，JTA的事务周期可横跨多个JDBC Connection生命周期。

**5、事务的传播特性：**
事务传播特性是用于指定当进行操作时，如何使用事务，JPA中有以下6种事务传播特性：
(1).Not Support：不支持，若当前事务有上下文，则挂起。
(2).Support：支持，若有事务，则使用事务，若无事务，则不使用事务。
(3).Required：需要，若有事务，则使用使用，若无事务，则创建新的事务。
(4).Required New：需要新事务，每次都会创建新的事务。
(5).Mandatory：必须有事务，若无事务，则将抛出异常。
(6).Never：必须不能有事务，若有事务，则会抛出异常。

**16.锁的机制：**
所谓锁，就是给选定的目标对象加上限制，使其无法被其他程序所修改，**JPA支持两种锁机制：乐观锁和悲观锁。**
**(1).悲观锁**：对数据被外界修改持保守态度，在整个数据处理过程中，将数据处于锁定状态，悲观锁的实现，往往依靠数据库提供的锁机制。最常用的悲观锁是在查询时加上“for update“，用法如下：
Select * from user where name=”test” forupdate
在JPA中，可以通过给Query对象设置锁模式(query.setLockMode)来制定悲观锁的模式：
  * a.LockMode.NONE：无锁机制。
  * b.LockMode.WRITE：在insert和update时会自动加锁。
  * c.LockMode.READ：在读取记录时会自动加锁。
  * d.LockMode.UPGRADE：利用数据库的for update字句加锁。
**(2).乐观锁**：相对于悲观锁，乐观锁机制采用较为宽松的加锁机制，悲观锁大多数情况下依靠数据库锁的机制实现，以保证操作最大程度的独占性，但是数据库的性能开销比较大。而乐观锁大多数基于数据版本(version)记录或者时间戳机制实现。
基于版本的乐观锁在读取数据时将数据的版本号一同读出，之后更新则对版本号加1，提交时对数据的版本与数据库中对应的版本进行比较，如果提交数据数据版本号打于数据库中当前版本号，则予以更新，否则认为是过期数据。乐观锁大大减小了数据库开销，提高了程序的性能。
JPA中可以在实体中使用` @Version `注解指定其使用乐观锁，JPA会在数据库对应的表中自动生成和维护一列版本号。

