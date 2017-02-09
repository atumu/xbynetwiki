title: hibernate学习笔记1 

#  Hibernate学习笔记总结进阶 
参考：http://blog.csdn.net/tanxiang21/article/details/8034105系列文章
http://blog.163.com/magicc_love/blog/static/185853662201282141433406/系列文章
安装：
```

<properties>
 <org.hibernate.version>4.2.8.Final</org.hibernate.version>
 </properties>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>${org.hibernate.version}</version>
</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${org.hibernate.version}</version>
		</dependency>
<dependency>
	<groupId>javassist</groupId>
	<artifactId>javassist</artifactId>
	<version>3.13.0.GA</version>
</dependency>

<!-- end-->
		<dependency>
			<groupId>c3p0</groupId>
			<artifactId>c3p0</artifactId>
			<version>0.9.1.2</version>
		</dependency>
		<dependency>
			<groupId>Mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.30</version>
		</dependency>
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-classic</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-core</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.9</version>
		</dependency>

```
##  核心接口和类 
**①Hibernate的5个核心接口**
` Session接口 `:Session接口负责执行被持久化对象的CRUD操作。` Session对象是非线程安全的 `。
同时，Hibernate的session不同于JSP应用中的HttpSession。这里当使用session这个术语时，其实**指的是Hibernate中的session**，而以后会将HttpSesion对象称为用户session。
` Session通过SessionFactory `打开，在所有的工作完成后，**需要关闭**。Session介于Transaction和Connection接口之间

这是SessionFactory中定义的` openSession() `:
继承hibernate3中的Session。在hibernate3中的Session接口中你可以找到如下两个方法的声明：
public Connection ` connection() ` throws HibernateException;
public Transaction ` beginTransaction() ` throws HibernateException;
第一个方法使得你**可以使用jdbc的方式操作数据库**，通常用来调用存储过程等
第二个是在session中获得**对事务进行操作。**

所以整个过程应该是：
1、先建立连接，然后进行一系列会话。
2、如果涉及到并发、一致性等问题，要进行事务操作的时候先打开事务，然后在执行一系列session中的方法对数据库进行操作。

` SessionFactory接口 `:SessionFactroy接口负责初始化Hibernate。它充当数据存储源的代理，并**负责创建Session对象**。这里用到了工厂模式。需要` 注意的是SessionFactory并不是轻量级的 `，因为一般情况下，` 一个项目通常只需要一个SessionFactory就够，作为全局变量进行保存 `当需要操作多个数据库时，可以为每个数据库指定一个SessionFactory。会话工厂缓存了生成的SQL语句和Hibernate在运行时使用的映射元数据。

` Configuration接口 `:Configuration接口负责**配置并启动**Hibernate，` 创建SessionFactory对象 `。在Hibernate的启动的过程中，Configuration类的实例首先定位映射文档位置、读取配置，然后创建SessionFactory对象。它包括如下内容：Hibernate运行的底层信息：数据库的URL、用户名、密码、JDBC驱动类，数据库Dialect(方言),数据库连接池等。Hibernate映射文件（*.hbm.xml描述实体到表之间映射对应关系）。

` Transaction接口 `:Transaction接口负责**事务相关**的操作。它是可选的，可发人员也可以设计编写自己的底层事务处理代码。
它将应用代码从底层的事务实现中抽象出来，这可能是一个JDBC事务，一个JTA用户事务或者甚至是一个公共对象请求代理结构（CORBA），允许应用通过一组一致的API控制事务边界。这有助于保持Hibernate应用在不同类型的执行环境或容器中的可移植性。

` Query和Criteria接口 `:Query和Criteria接口负责执行各种**数据库查询**。它**可以使用HQL语言或SQL语句两种**表达方式。

②创建上述接口对象的方法：
```

Configuration cfg = new Configuration();//查找classpath下的hibernate.properties
Configuration cfg = new Configuration().configure();//在classpath根目录下搜索hibernate.cfg.xml文件.可以显式配置路径.configure(“/your_path”)
SessionFactory sessionFactory = cfg.buildSessionFactory();
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

```
③Hibernate配置的两种方法：
属性文件（hibernate.properties）。
调用代码：Configuration cfg = new Configuration();
Xml文件（hibernate.cfg.xml）。
调用代码：Configuration cfg = new Configuration().configure();

##  持久化类原则 
(persistence classes)
五个原则：
①Hibernate对javabeans风格的属性实行持久化，属性不一定需要声明为public，还可以是default，protected或private的get/set方法
②设定一个**唯一标识(id)**
③实现一个**默认的无参构造函数(**可以不是public)，这样的话Hibernate就可以使用Constructor.newInstance()来实例化它们。
④**集合必须声明为接口类型**如List list = new ArrayList();
5、最好实现Serilizable接口

eclipse下创建映射XML文件的步骤可参考：http://blog.163.com/magicc_love/blog/static/18585366220128214940312/

##  基础配置篇 
一、首先找hibernate.properties文件
```

hibernate.dialect=org.hibernate.dialect.MySQLDialect  
hibernate.connection.driver_class=com.mysql.jdbc.Driver  
hibernate.connection.url=jdbc:mysql://192.168.18.184:3306/SAMPLEDB  
hibernate.connection.username=root  
hibernate.connection.password=  
hibernate.show_sql=true  

```
二、其次找hibernate.cfg.xml
```

<?xml version="1.0"encoding="utf-8" ?>  
<!DOCTYPE hibernate-configuration  
 PUBLIC "-//Hibernate/HibernateConfiguration DTD//EN"  
 "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">  
<hibernate-configuration>  
<session-factory >  
<property name="dialect">org.hibernate.dialect.MySQLDialect</property>  
<property name="connection.driver_class">com.mysql.jdbc.Driver</property>  
<property name="connection.url">jdbc:mysql://localhost:3306/sampledb</property>  
<property name="connection.username">root</property>  
<property name="connection.password">1234</property>  
<property name="show_sql">true</property>  
<mapping resource="mypack/Monkey.hbm.xml" />  
</session-factory>  
</hibernate-configuration>  

```
```

public static SessionFactory sessionFactory;  
       static {  
              try {  
       Configuration config = newConfiguration();  
       config.configure();  
           sessionFactory = config.buildSessionFactory();  
              } catch (RuntimeException e) {  
                     // TODO: handle exception  
                     e.printStackTrace();  
                     throw e;  
              }  
       }  

```
三、找类配置文件 例如：Monkey.hbm.xml
几个需要注意的属性
  * 1.dynamic-insert="true" dynamic-update="true"  动态版插入更新  性能优化需要
  * 2.access="field" 不是要get set直接采用属性  ，就是Hibernate底部通过反射属性field进行赋值或读取。同时setAccesible(true)
  * 3.formula 无属性也可以直接查数据库组装数据  ,直接写表达式查询
  * 4.在class中声明mutable=”false” 或 @Immutable  
      这意味着对该类的更新将会被忽略，不过不会抛出异常，只允许有增加和删除操作。  
     在class中声明mutable=”false”：insert=允许,delete=允许,update=不允许  
     在集合中声明mutable=”false” 或 @Immutable  
      这意味着在这个集合中插入记录或删除孤行是不允许的，否则会抛出异常。只允许更新操作。  
     不过，如果启用级联删除的话，当父类被删除时，其所有子类也将被删除，即使它是mutable的。  
     在集合中声明mutable=”false”：insert=不允许,孤行删除=不允许,delete=允许,update=允许  

以下为实例：
```

<?xml version="1.0"?>  
<!DOCTYPE hibernate-mapping  
PUBLIC "-//Hibernate/HibernateMapping DTD 3.0//EN"  
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">  
<hibernate-mapping>  
  <class name="mypack.Monkey" table="MONKEYS" dynamic-insert="true" dynamic-update="true"  > <!-- mutable="ture" -->  
    <id name="id">  
      <generator class="increment"/>  
    </id>  
    <property name="name" column="NAME" />  
    <property name="gender" column="GENDER" access="field" />  
    <property name="age" column="AGE" />  
    <property name="avgAge"  
      formula="(select avg(m.AGE) from MONKEYS m)" />  
   <property name="description"  type="text" column="`MONKEY  DESCRIPTION`"/>  
 </class>  
</hibernate-mapping>  

```

###  transient 
防止属性序列化
transient关键字
@Transient JPA注解
###  hbm2dll.auto的四种取值 
①create-drop : hibernate一开始就发送一个drop语句，再执行create语句创建表，当sessionFactory关闭时又执行drop语句删除表。结果就是，当表存在时，首先会被drop掉，然后又创建，最终被drop，如果表不存在，一开始虽然hibernate还是会发送drop语句，当时因为表不存在而drop失败。
②create：hibernate首先会发送一个drop语句，如果表存在则会被drop掉，然后hibernate再发送create语句创建表。
③update：hibernate会首先查询数据库看是否存在此表，如果存在则不管，如果不存在则会先发送一个create语句创建一个表。
④validate：每次插入数据之前都会验证数据库中的表结构和hbm文件的结构是否一致。如果表不存在，则报错。
###  Hibernate方言 
![](/data/dokuwiki/hibernate/pasted/20150817-041226.png|)
##  Hibernate三种对象状态 
后面还会提到[这点](http://wiki.xby1993.net/doku.php?id=hibernate:hibernate%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B01#java对象在hibernate持久化层的状态)
  * Transient:瞬态
  * Persistent:持久态
  * Detached:游离态
![](/data/dokuwiki/hibernate/pasted/20150907-090805.png)![](/data/dokuwiki/hibernate/pasted/20150907-090815.png)
##  标识符及基本级联配置 
###  标识符 

  * increment 用于代理主键.由Hibernate自增方式生成标识符,每次增量为1  
  * identity  同上。由底层数据库生成标识符.前提是要数据库支持自增  
  * sequence  同上。由底层数据库生成标识符.前提是要数据库支持序列  
  * hilo      同上。由Hibernate根据high/low算法生成标识符  
  * native    同上。根据数据库的支持，选择identity,sequence或hilo  
  * uuid.hex  同上。由Hibernate采用UUID算法生成字符串形式的标识符  
  * assigned  用于自然主键。由Java代码通过setId()生成标识符。  

**数据库可使用的标识符**
几种常用数据库可以使用的标识符生成器:  
  * 1.      MySQL:identity,increment,hilo,native  
  * 2.      MS SQL Server:identity,increment,hilo,native  
  * 3.      Oracle:sequence,seqhilo,hilo,increment,native  
  * 4.      跨平台：native  
```

id生成策略
①：hilo(oracle|mysql都支持) 高低位算法；跟数据库无关的；
步骤:准备一张表并且含有一列默认值，还有最大低位值
(建完表后一定要commit，切记切记！) 
        <id name="id" column="id">
                <generator class="hilo">
                    <param name="table">high_value</param>
                    <!--设置高位值取值的表-->
                    <param name="column">next_value</param>
                    <!--设置高位值取值的字段-->
                    <param name="max_lo">50</param>
                    <!--指定低位最大值，当取到最大值时会再取一个高位值再运算-->
                </generator>
        </id>

②:sequence(oralce支持 |mysql不支持sequence)
    建立sequence语法:create sequence sequence_name start with 1 increment by 1
            <generator class="sequence">
                    <param name="sequence">test_seq</param>
                </generator>
③:seqhilo(oralce支持 |mysql不支持sequence)
可以不用自己建sequence;默认用数据库里的hibernate_sequence;
            <generator class="seqhilo">
                <param name="sequence">test_seq</param>
                <param name="max_lo">5</param>
            </generator>

④:increment（oracle|mysql都支持）不适合多线程；不多用；
            <generator class="increment">
                </generator>
⑤: identity <generator class="identity">（mysql支持|oracle不支持）
                </generator>
⑥: native <generator class="native">(oracle|mysql都支持)
                </generator>
            oracle:默认去取hibernate_sequence序列的值
⑦：uuid(id为String型，要修改表和实体类中id的类型，oracle表中修改为varchar2(>=32);mysql表中修改为varchar(>=32))
<generator class="uuid">
</generator>
⑧assigned:手动赋值

```
###  基础级联 
 
Monkey属于Team
Team.hbm.xml       【inverse="true"  cascade="all-delete-orphan"】
```

    <class name="mypack.Team" table="TEAMS">         
        <id name="id" type="long" column="ID">        
            <generator class="increment"/>  
        </id>  
        <property name="name" type="string" column="NAME"/>             
              <set name="monkeys"  
                   cascade="all-delete-orphan"  
                      inverse="true">  
                      <key column="TEAM_ID"/>  
                      <one-to-many class="mypack.Monkey"/>  
              </set>            
    </class>  
</hibernate-mapping>

```
Monkey.hbm.xml      【 none delete all delete-orphan all-delete-orphan 】
```

<hibernate-mapping>  
    <class name="mypack.Monkey" table="MONKEYS">  
        <id name="id" type="long" column="ID">  
            <generator class="increment"/>  
        </id>  
        <property name="name" type="string" column="NAME"/>         
        <!--mapping with cascade -->        
        <many-to-one  
         name="team"  
        column="TEAM_ID"  
         class="mypack.Team"  
         cascade="save-update" <!— none delete all delete-orphan all-delete-orphan  -->   
         lazy="false"  <!-- proxy -->      
        />                
    </class>  
</hibernate-mapping> 

```
操作Service类代码片段
```

tx = session.beginTransaction();  
              Team team = new Team("BULL", new HashSet<Monkey>());  
              Monkey monkey = new Monkey();  
              monkey.setName("Tom");  
              monkey.setTeam(team);  
              team.getMonkeys().add(monkey);  
              session.save(team);  
              tx.commit();  
              tx =session.beginTransaction();  
              Team team =(Team) session.load(Team.class, teamId);  
              Monkey monkey =(Monkey) team.getMonkeys().iterator().next();  
              // 解除team和Monkey的关联关系  
              team.getMonkeys().remove(monkey);  
              monkey.setTeam(null);  
              tx.commit(); 

```
##  【配置详解】 
###  Hibernate配置翻译 

```

<hibernate-mapping>  
<class name="项目路径" table="库中对应表名" schema="dbo" catalog="netoa">  
      <meta attribute="class-description">指定描述类的javaDoc</meta>  
      <meta attribute="class-scope">指名类的修饰类型</meta>  
      <meta attribute="extends">指定继承类</meta>  
         <id name="bgrkbh" type="long">  
            <column name="BGRKBH" precision="15" scale="0" sql-type="库中类型" check="BGRKBH>10"/>  
            <meta attribute="scope-set">指定类,类属性的getxxx(),setxxx()方法的修饰符  
             包括:static,final,abstract,public,protected,private  
            </meta>  
            <generator class="assigned" />  
        </id>  
         <property name="Class.fileName" type="long">  
                <column name="YSLX" precision="精度" scale="刻度" not-null="默认false" sql-type="数据库中类型"/>  
                附加属性不会影响Hibernate的运行行为  
                <meta attribute="field-description">指定描述类的javaDoc</meta>  
                指定描述类属性的javaDoc  
         </property>  
</class>   
</hibernate-mapping>

```  
###  <meta>元素属性 

```

    属性                                                描述  
class-description                            指定描述类的javaDoc  
field-description                            指定描述类的属性javaDoc  
interface                                    如果为true,表明生成接口而非类,默认false  
implements                                   指定类所实现的接口  
extends                                      指定继承的父类名  
generated-class                              重新指定生成的类名  
scope-class                                  指定类的修饰符,默认public  
scope-set                                    指定set方法的修饰符,默认public  
scope-get                                    指定get方法的修饰符,默认public  
scope-field                                  指定类的属性的修饰符,默认private  
use-in-toString                              如果为true,表示在toString()方法中包含此属性  
gen-property                                 如果为false,不会在java类中生成此属性,默认true  
finder-method                                指定find方法名 

```
###  <column>元素属性 
```

name                 设定字段名字  
length               设定字段长度  
not-null             如为true,指名该字段不允许为null,默认false  
unique               如为true,指名该字段具有唯一约束,默认false  
index                给一个或多个字段建立索引  
unique-key           为多个字段设定唯一约束  
foreign-key          为外键约束命名,在<many-to-many><one-to-one><key><many-to-one>元素中包含  
                     foreign-key属性,在双向关联中,inverse属性为true的一端不能设置foreign-key  
sql-type             设定字段sql类型  
check                设定sql检查约束 

``` 
###  控制insert or update 语句的映射属性 

```

<property>元素的insert属性                如为false,在insert中不包含该字段,默认为true  
<property>元素的update属性                如为false,在update中不包含该字段,默认为true  
<class>元素的mutable属性                  如为false,等价于所有字段的update属性为false,默认为true  
<property>元素的dunameic-insert属性       如为true,表明动态生成insert语句,只有不为null,才会包含insert语句中,默认false  
<property>元素的dunameic-update属性       如为true,表明动态生成update语句,只有不为null,才会包含insert语句中,默认false  
<class>元素的dunameic-insert属性          如为true,表明等价于所有字段动态生成insert语句,只有不为null,才会包含insert语句中 ,默认false  
<class>元素的dunameic-update属性          如为true,表明等价于所有字段动态生成update语句,只有不为null,才会包含insert语句中 ,默认false

```
###  cascade属性 

```

描述  
none                              在保存更新时,忽略其他关联对象,他是cascade默认属性  
save-update                       当通过Session的save(),update()以及saveOrUpdate()方法来保存或更新当前对象时,级联保存所有关联的新建的临时对象,并且级联更新所有关联的游离对象  
delete                            当通过session的delete()方法删除当前对象时,及联删除所有对象  
lock                              把游离对象加入缓存当中，关联对象也加入  
evict                             把持久化对象从缓存中移除，关联对象也移除  
refresh                           刷新当前缓存中对象，级联对象也会刷新  
all                               包含save-update、 delete、 evict、lock及refresh的行为  
delete-orphan                     删除所有和当前对象解除关联关系的对象  
all-delete-orphan                 包含all和delete-orphan 

```
###  Hibernate映射类型 
对应的java基本类型及对应的标准SQL类型
```

 Hibernate 映射类型               java类型                     标准SQL类型  
  integer或者int                  int                          INTEGER  
  long                            long                         BIGINT  
  short                           short                        SMALLINT  
  byte                            byte                         TINYINT  
  float                           float                        FLOAT  
  double                          double                       DOUBLE  
  big_decimal                     java.math.BigDecimal         NUMERIC  
  character                       char and string              CHAR  
  string                          string                       VARCHAR  
  boolean                         boolean                      BIT  
  
 Hibernate映射类型,对应的java时间和日期类型及对应的标准SQL类型  
映射类型           java类型                     标准SQL类型             描述  
 date       java.util.Date或者java.sql.Date        DATE         代表日期,YYYY-MM-DD  
 time       java.util.Date或者java.sql.Date        TIME         代表时间,形式为HH:MM:SS  
 timestamp java.util.Date或者java.sql.Timestamp   TIMESTAMP    代表日期和时间,YYYYMMDDHHMMSS  
 calendar   java.util.Calendar                     TIMESTAMP    同上  
lendar_date java.util.Calendar                     DATE         代表日期,YYYY-MM-DD 

JAVA大对象类型的Hibernate映射类型
映射类型        java类型             标准SQL类型         MYSQL类型           ORALCE类型  
binary           byte[]            VARBINARY(或BLOB)      BLOB                  BLOB  
text             string               CLOB                TEXT                  CLOB  
serializable  实现Serializable       VARBINARY(或BLOB)    BLOB                  BLOB  
              接口任意一个java类  
clob            java.sql.Clob          CLOB               TEXT                  CLOB              
blob            java.sql.Blob          BLOB               BLOB                  BLOB  

```
##  【操纵对象篇】 
###  Hibernate Session缓存 

**A     Session的缓存作用**
1.1  减少访问数据库的频率  
1.2  当缓存中的持久化对象间存在循环关联关系时，保证不死循环  
1.3  保证数据库中相关记录与缓存中的相应对象保持同步  

**B     脏检查及刷新缓存的机制**
1.1  刷新缓存Flush时，执行SQL顺序
a)        调用session.save()方法的先后顺序，实体执行insert语句  
b)       实体执行update语句  
c)        集合进行delete语句  
d)       集合进行删除、更新或者插入sql语句  
e)        集合进行insert语句  
f)        按照调用session.delete()方法的先后顺序，执行实体删除delete语句  
1.2  刷新缓存时间点
a)      Transaction的commit()方法  
b)     执行查询操作时，缓存中持久化对象属性已经变化，实施同步  
c)      调用Session的flush()方法  

1.3  session.setFlushMode(FlushMode.COMMIT)  
 		各种查询方法	commit()	flush()
FlusMode.AUTO默认	清理	清理	清理
FlushMode.COMMIT	不清理	清理	清理
FlushMode.NEVER	不清理	不清理	清理

###  Java对象在Hibernate持久化层的状态 

  * 临时状态(transient)：          刚刚new，未被持久化，不存于Session缓存  
  * 持久化状态(persistent)：       已经被持久化，并且加入Session缓存  
  * 删除状态(removed)：            不在处理Session缓存中并且Session计划从数据库删除  
  * 游离状态(detached)：           持久化转化而来，不存于Session缓存，数据库有对应记录  

示列：
```

Monkey对象                               生命周期            状态  
tx = session.beginTransaction();        开始生命周期        临时状态  
Monkey m1 = new Monkey(“Tom”);  
session.save(m1);                       处于生命周期        转持久态  
Long id = m1.getId();                   处于生命周期       处于持久态  
m1=null; //session缓存会引用它  
Monkey m2 = (Monkey) session.get (Monkey.class, id);  
tx.commit();  
session.close();                       处于生命周期         转游离态  
System.out.println(m2.getName());      处于生命周期        处于游离态  
m2=null;                               结束生命周期       结束生命周期  

```
持久化变游离态方法：
  * a.        Session.close()  session缓存被清空  
  * b.        Session.evict()  从缓存中清除一个持久化对象  
  * c.        Session.clear()  清除缓存中所有持久化对象  

![](/data/dokuwiki/hibernate/pasted/20150907-074252.png)

###  Session接口详细用法 
一般用` saveOrUpdate() `
  * ` save() `  当清理缓存时才执行SQL  
  * save()  返回对象的标识符，而save方法**可以保存任何状态的对象。**  
  * ` persist() `方法无返回值。此外，persist方法**只能保存临时态和持久态的对象**  
  * 
  * ` get() `   方法始终返回对象的实例，查找指定OID不存在，返回null；  
  * ` load() ` 方法除非在持久化上下文中已经存在一个该标识符定位的对象，不然总是先返回对象的代理，指定OID不存在，**抛出ObjectNotFoundException**  

默认所有持久化类都采用` 延迟检索策略 `，及映射<class>元素` lazy属性默认”ture” `  
<class name=”mypack.Monkey” table=”MONKEYS” lazy=”true”>  
**lazy属性为”false”**，load方法才采用立即检索策略  
  * get()  忽略该属性，总是采用立即检索策略  
  * lazy属性为” ture”，load方法采用延迟检索策略，get()采用立即检索  
  * get()  方法访问各属性  
  * load()  方法为了删除，或者建立与别的对象的关系  

**` update() ` 把游离对象转变为持久对象**，计划执行update语句，清缓存时只执行一次  
<classname=”mypack.Monkey” table=”MONKEYS” select-before-update=”true”>  
变化时，才执行update，所以如果不经常变，就ture，默认为false,经常变时默认  
` **saveOrUpdate()  判断游离就UPDATE，不是就SAVE ** ` 
OID为null，基础类型为OID的话就为初始值如long为id，就是0，或者版本控制version为null，有unsaved-value属性值就以它为准或null，这时就save否则update，如果无该id就抛插入异常  
   
session.` merge() `关联游离态则复制到同UID持久态对象中，然后将游离态内容更新到数据库中，计划执行update  
session.merge关联游离态若无同UID的持久态对象，则根据该UID加载持久态对象，然后复制到该持久态，计划执行update,若数据库无该uid对象则创建新对象，复制进去，调用save()转持久态  
session.merge关联临时态则创建对象，复制到该对象，最后调save()转持久态对象  
session.` updat() `**关联游离态转为持久态**，计划执行update，但本来已存该持久态，则抛异常  
   
session.` delete() `删除游离态对象，先是游离对象被session关联，变持久态删除  
删除持久态，计划执行delete语句（清理缓存时，一次一个，不建议再使用）  

![](/data/dokuwiki/hibernate/pasted/20150907-075041.png)

###   级联操作对象图 
` cascade `属性
```

描述  
none                              在保存更新时,忽略其他关联对象,他是cascade默认属性  
save-update                       当通过Session的save(),update()以及saveOrUpdate()方法来保存或更新当前对象时,级联保存所有关联的新建的临时对象,并且级联更新所有关联的游离对象  
delete                            当通过session的delete()方法删除当前对象时,及联删除所有对象  
lock                              把游离对象加入缓存当中，关联对象也加入  
evict                             把持久化对象从缓存中移除，关联对象也移除  
refresh                           刷新当前缓存中对象，级联对象也会刷新  
all                               包含save-update、 delete、 evict、lock及refresh的行为  
delete-orphan                     删除所有和当前对象解除关联关系的对象  
all-delete-orphan                 包含all和delete-orphan 

``` 
###  批量处理数据 

**1.    通过Session来进行批量操作**
```

a)    配置JDBC单次批量处理数目10~50：hibernate.jdbc.batch_size=20  
b)    采用”identity”  
c)    批量操作建议关闭Hibernate二级缓存（SessionFactory应用范围）  
       hibernate.cache.use_second_level_cache=false          进程/集群  
       session.flush();  
       session.clear();  
d)    更新的话用Query q = session.createQuery(selectHQL );  
       ScorllableResults sr =Query.scroll(ScrollMode.FORWARD_ONLY)  
       sr.next()  sr.get(0) 

``` 

**2.      通过StatelessSession来进行批量操作**
```

a)       StatelessSession session =SessionFactory.openStatelessSession();  
b)       无缓存，加载保存更新数据后都为游离态  
c)       不与二级缓存交互  
d)       立即执行SQL  
e)       不自动进行脏检查，变化内存对象属性后需显示调用update()更新  
f)       不会对关联对象进行任何级联操作  
g)       加载两次同UID对象，是具有不同内存地址的对象

```  

**3.      通过HQL来进行批量操作**
```

a)       批量更新session.createQuery(updateHQL ).executeUpdate()  
b)       批量删除session.createQuery(deleteHQL ).executeUpdate()  
例子：String deleteHQL = “delete Monkey m where m.name = :oldName”  
session.createQuery(deleteHQL).setString(“oldName”,”Tom”) .executeUpdate()  
c)       批量插入session.createQuery(insertHQL ).executeUpdate()  
只支持insert into…select…  不支持insert into …values… 

``` 

##  【映射关系】 
本文只包含xml配置方式，JPA注解方式请参考：[Hibernate JPA注解方式处理映射关系](/pages/dokuwiki/Hibernate/Hibernate JPA注解方式处理映射关系)
###  【映射组成关系】 
部分与整体的强关联模式。组件类作为实体类的值类型被嵌入到实体类表列中。
```

<hibernate-mapping >  
  <class name="mypack.People" table="PEOPLE" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
    <property name="name" type="string" column="NAME" />  
    <!--这里component -->
    <component name="hand" class="mypack.HAND">  
           <parent name="people" />  
           <property name="color" type="string" column="COLOR"/>  
    </component>    
  </class>  
</hibernate-mapping>  

```
**说明：**
1.People和Hand是组合关系，在People类中有个成员是Hand类，名字是hand
2.标签<component>和<property>没多大区别 ，对应一个成员
3.标签<component>和<class>其实也没多大区别，<component>里面可以嵌套<class>能嵌套的东西比如嵌套他自己<component>或者<set>什么的
3.标签<component>一点点区别是他里面有个<parent>这你懂的
4.用法就是Hand完全由parent--》People控制，Hand属于Value而不是Entity实体，无OID
###  【映射继承关系】 
 场景有个User类，然后扩展出了Student类和Teacher类，也就是Student类和Teacher类继承了User类 

**方式一：给每个子类造了一个表**
数据库是这样设计的，给每个子类造了一个表，下面是Hibernate项目配置，只配User
```

<class name="User"abstract="true">
 <id name="id" column="USER_ID" type="long">
  <generator class="native"></generator>
 </id>
 <property name="name" column="USER_NAME" type="string"></property>
  <!-- 这里union-subclass-->
 <union-subclass name="Teacher" table="TEACHER">
  <property name="job" column="TEACHER_JOB"/>
 </union-subclass>
 <union-subclass name="Student" table="STUDENT">
  <property name="class" column="STUDENT_CLASS"/>
 </union-subclass>
</class>

```
用法就是存储查询Teacher、Student都可以；如果查全的也行User，数据库发的是` union `语句

**方式二：整个各种子类和他们的父类都在一张表，同时搞个字段` discrimnator ` 区分是哪个类。**
整个各种子类和他们的父类都在一张表，同时搞个字段区分是哪个类。配置文件这么写：
```

 <class name="User"table="USERS">
 <id name="id" column="USER_ID" type="long">
  <generator class="native"></generator>
 </id>
   <!--discrimnator  -->
 <discrimnator column="DISCRIMNATOR" type="String"></discrimnator>
 <property name="name" column="USER_NAME" type="string"></property>
   <!--subclass discrimnator-value-->
 <subclass name="Teacher" discrimnator-value="TEACHER">
  <property name="job" column="TEACHER_JOB"/>
 </subclass>
 <subclass name="Student" discrimnator-value="STUDENT">
  <property name="class" column="STUDENT_CLASS"/>
 </subclass>
</class>

```
父类如果不是抽象类，也可以在<class>表情内部搞个discrimnator-value="某某"，这样父类子类任意搞

**方式三：父类对应一张表，每子类再各自对应一张表，子类表的主键也是外键**
父类对应一张表，每子类再各自对应一张表，**子类表的主键也是外键**，就是说子类id=2,和父类里面有个id=2的组合用，多了个` key `配。配置文件这么写的：
```

 <class name="User"table="USERS">
 <id name="id" column="USER_ID" type="long">
  <generator class="native"></generator>
 </id> 
 <property name="name" column="USER_NAME" type="string"></property>
   <!-- joined-subclass-->
 <joined-subclass name="Teacher" table="TEACHER">
   <!--key -->
  <key column="TEACHER_ID"/>
  <property name="job" column="TEACHER_JOB"/>
 </joined-subclass>
 <joined-subclass name="Student" table="STUDENT">
  <key column="STUDENT_ID"/>
  <property name="class" column="STUDENT_CLASS"/>
 </joined-subclass>
</class>

```

方式四、虽然原则上上述方法是不能混用的，但实际上使用外连接也是可以结合使用的，但是个人认为不太好，除非万不得已，最好不要使用，参考《Hibernate in action》208页。

###  【映射值类型集合】 
Set,Bag,Map,List
**Set**
```

<hibernate-mapping >  
  <class name="mypack.Monkey" table="MONKEYS" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
    <property name="name" type="string" >  
        <column name="NAME" length="15" />  
    </property>  
   <set   name="images"   table="IMAGES"    lazy="true" >  
        <key column="MONKEY_ID" />  
       <element column="FILENAME" type="string"  not-null="true"/>  
   </set>     
  </class> 

</hibernate-mapping>  

```  
说明：
  * 1.**值类型**，就说Image类就行String类型一样当值搞了，一切受包含它的类控制
  * 2.Set是在数据库中**有表**的，并且可延时加载（默认）,数据库有 ID 和 MONKEY_ID
  * 3.elemet，你懂的，实体类可没这个。
  * 4.Set是hibernate里的Persistent类，**Hash的 也就是无序的**

**Bag**

```

<hibernate-mapping >  
  <class name="mypack.Monkey" table="MONKEYS" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
    <property name="name" type="string" >  
        <column name="NAME" length="15" />  
    </property>  
   <idbag   name="images"   table="IMAGES"    lazy="true">  
        <collection-id type="long" column="ID">  
           <generator class="increment"/>  
        </collection-id>  
        <key column="MONKEY_ID" />  
        <element column="FILENAME" type="string"  not-null="true"/>  
   </idbag>     
  </class>  
</hibernate-mapping>

```  
说明：
和上一个唯一的区别是他可以放相同的元素，上面那个是Set你懂的

**List**
```

<list   name="images"   table="IMAGES"    lazy="true">  
     <key column="MONKEY_ID" />  
     <list-index column="POSITION" />  
     <element column="FILENAME" type="string"  not-null="true"/>  
</list> 

```   
说明：
1.它的数据库**没有ID主键**，**多了个POSITIOM的字段，显然他可以排序**；Image类中是没有position属性的。存是按放入的先后顺序来的
2.对了Bag和List 在Monkey中都是包含了一个List的图片集合，但是**BAG无序，List存的时候哪个先存，拿的时候可以get(0)这样取。**

**Map**
```

<map   name="images"   table="IMAGES"    lazy="true">  
     <key column="MONKEY_ID" />  
     <map-key column="IMAGE_NAME" type="string"/>  
     <element column="FILENAME" type="string"  not-null="true"/>  
</map>   

```  
说明：
1.数据库有个ID和MONKEY_ID，组成联合主键
2.处理的时候  得到一个Map images=monkey.getImages()  他有Set keys = images.keySet();显然这个是我们IMAGE_NAME   无序

**对集合进行排序**
这么多无序了，我们怎么对集合排序呢？有2种策略：
1.在**数据库查的时候order-by属性排好序**，这个只有**List不支持**，因为他完全由自己排序
```

示例： <set   name="images"   table="IMAGES"    lazy="true" order-by="lower(FILENAME) desc">  
              <key column="MONKEY_ID" />  
              <element column="FILENAME" type="string"  not-null="true"/>  
       </set>  

```   
 2.也可以**内存排序 sort属性**，其实就是查出来后，用` Comparator `给查出来的集合排了一把
```

示例： <set   name="images"   table="IMAGES"    lazy="true" sort=mypack.selfCpmparator">  
              <key column="MONKEY_ID" />  
              <element column="FILENAME" type="string"  not-null="true"/>  
       </set> 

```    
当然肯定是sort有效果，但愿Hibernate不会去排序查
排序的时候，集合的实现类就可以用TreeSet  TreeMap叻，完全排序完成。如 Monkey类里面有成员 private Map images=new TreeMap();

###  【映射实体关系】 
**一对一**
共享主外键：两个都是主键，子表再加外键
```

Monkey.hbm.xml
<one-to-one name="address"   
    class="mypack.Address"  
    cascade="all"   
 />  
Address.hbm.xml
 <class name="mypack.Address" table="ADDRESSES" >  
    <id name="id" type="long" column="ID">  
      <generator class="foreign">  
        <param name="property">monkey</param>  
      </generator>  
    </id>  
    <one-to-one name="monkey"   
        class="mypack.Monkey"  
       constrained="true"  
    />  
</class>

```  
说明：Address  constrained="true"  generator class="foreign"  
ADDRESSES表ID主键做为外键，两表表共享OID
显然，Monkey 与 Address存在主从关系。

**多对多**
Monkey 和 Teacher是多对多，Learning是中间表
情形一、【组件型式】多对多形式：
```

<hibernate-mapping >  
  <class name="mypack.Monkey" table="MONKEYS" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
    <property name="name" column="NAME" type="string" />    
    <set name="learnings" lazy="true" table="LEARNING" >  
        <key column="MONKEY_ID" />  
        <composite-element class="mypack.Learning" >  
          <parent name="monkey" />  
          <many-to-one name="teacher" class="mypack.Teacher" column="TEACHER_ID" not-null="true"/>  
          <property name="gongfu" column="GONGFU" type="string" not-null="true" />  
         </composite-element>   
    </set>       
  </class>  
</hibernate-mapping> 

``` 
说明：中间表LEARNING映射Learning类以集合组建形式属于Monkey类，然后和Teacher存在多对一的关联
Learning类共享Monkey类OID，操作时候一样是相互保存双方，同通常的多对多保存方式。
<composite>标签  和  组成关系<component>标签需区分
<composite>标签  和  值类型集合<set>标签的区分是 里面不是<element> 

情形二、 【标准】多对多  数据库有中间表，中间表无对应BEAN
```

<hibernate-mapping >  
  <class name="mypack.Monkey" table="MONKEYS" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
    <property name="name" column="NAME" type="string" />        
    <set name="teachers" table="LEARNING"  
        lazy="true"  
        cascade="save-update">  
        <key column="MONKEY_ID" />  
        <many-to-many class="mypack.Teacher" column="TEACHER_ID" />  
    </set>         
  </class>  
</hibernate-mapping> 

``` 
```

<hibernate-mapping >  
  <class name="mypack.Teacher" table="TEACHERS" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
    <property name="name" column="NAME" type="string" />     
    <set name="monkeys" table="LEARNING"  
        lazy="true"  
        inverse="true"  
        cascade="save-update">  
        <key column="TEACHER_ID" />  
        <many-to-many class="mypack.Monkey" column="MONKEY_ID" />  
    </set>   
  </class>  
</hibernate-mapping> 

``` 
说明：标准版得多对多，然后JAVA保存相互关联关系时，**两边都要设置对方，**
注意下inverse，而数据库只发一条修改语句

**一对多 多对一**
中间表做BEAN，中间可做个关系属性
```

<hibernate-mapping >  
  <class name="mypack.Monkey" table="MONKEYS" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
   <property name="name" column="NAME" type="string" />     
   <set name="learnings" lazy="true" inverse="true" cascade="save-update">  
        <key column="MONKEY_ID" />  
        <one-to-many  class="mypack.Learning" />  
   </set>     
  </class>  
</hibernate-mapping>  

```
```

<hibernate-mapping >  
  <class name="mypack.Learning" table="LEARNING" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
    <property name="gongfu" column="GONGFU" type="string" />      
    <many-to-one name="monkey" column="MONKEY_ID" class="mypack.Monkey" not-null="true" />  
    <many-to-one name="teacher" column="TEACHER_ID" class="mypack.Teacher" not-null="true" />   
  </class>  
</hibernate-mapping>  

```
```

<hibernate-mapping >  
  <class name="mypack.Teacher" table="TEACHERS" >  
    <id name="id" type="long" column="ID">  
      <generator class="increment"/>  
    </id>  
    <property name="name" column="NAME" type="string" />    
     <set name="learnings" lazy="true" inverse="true" cascade="save-update">  
        <key column="TEACHER_ID" />  
        <one-to-many  class="mypack.Learning" />  
     </set>   
  </class>  
</hibernate-mapping>  

```
说明：其实就是多对多，变为了 一对多 多对一  ，**注意可以级联保存，**无限级联~  无限级联~

###  lazy的用法(延迟加载) 
lazy是延时(延迟)的意思，如果lazy=true，那么就是说数据库中关联子表的信息在hibernate容器启动的时候不会加载，而是在你真正的访问到字表非标识字段的时候，才会去加载。反之，如果lazy=false的话，就是说，子表的信息会同主表信息同时加载。一般只有完全用到子表信息的时候，才会设置lazy=false
##  【高级篇】并发、Session管理 
数据库事务级别  以及对应 Hibernate事务码
先查  mysql>select @@tx_isolation 
设置  mysql>set global transaction isolation level read committed
  * 1：Read Uncommitted 读未提交数据 一个事务执行中可看到另一个事务未插入和未更新的的记录  
  * 2：Read Committed   读已提交数据 一个事务执行中可看到另一个事务已插入的记录，还能看到别人已更新的  
  * 4：Repeatable Read  可重复读     一个事务执行中可看到另一个事务已插入的记录，但看不到别人已更新的  
  * 8：Serializable     串行化       一个事务等两外一个事务搞完了才进行  

hibernate配置   hibernate.connection.isolation=2
Hibernate加载对象模式
Monkey m = (Monkey )session.get (Monkey.class,1,LockMode.UPGRADE);
  * LockMode.NONE 缓存存在，那么直接拿缓存的（默认）  
  * LockMode.READ 总是去数据库查，如果设置了版本，就检查缓存版本是否一致  
  * LockMode.UPGRADE 去数据库查，检查缓存版本是否一致，数据库支持悲观，发送select...for update  
  * LockMode.UPGRADE_NOWAIT 去数据库查，发送select...for update nowait （如果支持），到点抛异常  
  * LockMode.WRITE hibernate框架内部使用的锁  

说明：LockMode.READ一般这样用session.lock(object,LockMode.READ) 查看版本是否一致，不一致抛异常

###  锁机制 
**①悲观锁**，正如其名，它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度，因此，在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）。

**②乐观锁**（ Optimistic Locking ）
相对悲观锁而言，乐观锁机制采取了更加宽松的加锁机制。悲观锁大多数情况下依靠数据库的锁机制实现，以保证操作最大程度的独占性。但随之而来的就是数据库性能的大量开销，特别是对长事务而言，这样的开销往往无法承受。
而乐观锁机制在一定程度上解决了这个问题。乐观锁，大多是基于数据版本（ Version ）记录机制实现。何谓数据版本？即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个 “version” 字段来实现。读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如果提交的数据版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。
###  管理Session 
                 各种查询方法   commit()  flush()  
FlusMode.AUTO默认   清理        清理    清理  
FlushMode.COMMIT    不清理      清理    清理  
FlushMode.NEVER     不清理      清理    清理  
FlushMode.MANUAL    不清理      不清理  清理  
设置 hibernate.current_session_context_class=thread/jta/managed 线程/JTA/委托
sessionFactory.getCurrentSession()

线程绑定下，如果线程没提交，不管怎么去拿，都还是一样的session，可回滚事务
如果提交了，再拿，会新建一个另外的session

**如果存在要等待的事务**
  * 方式一：一个事务，长时间占数据库连接，和Session内存占用
  * 方式二：多个事务，使用游离态对象传递值，但没保证原子性
  * 补充措施：最后事务才更新，或者提供异常后的补偿代码手动撤销事务
  * 方式三：手动清理缓存缺点：内存中session一直未释放

