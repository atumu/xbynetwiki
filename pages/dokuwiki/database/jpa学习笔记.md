title: jpa学习笔记 

#  JPA学习笔记 
参考：http://www.blogjava.net/luyongfa/archive/2012/11/01/390572.html
http://blog.csdn.net/aspnetandjava/article/details/7034779
还可以参考：
http://blog.csdn.net/chjttony/article/details/6086298
http://blog.csdn.net/chjttony/article/details/6086305
http://wenku.baidu.com/link?url=73244p-ZanDZEHG0KIvYVTB1deZbdj9ZgHut8Rb8V7fhHnQhf2sza55rhHTjM-9CXzkfeQ_FvRDETdH8Bm5ny_Uk1ZbHFk9KBK7jSwUp5B_
##  一、JPA基础 

**1.1 JPA基础**
JPA： java persistence api 支持XML、JDK5.0注解俩种元数据的形式,是SUN公司引入的JPA ORM规范

元数据：对象和表之间的映射关系

实体： entity，需要使用Javax.persistence.Entity注解或xml映射，需要无参构造函数，类和相关字段不能使用final关键字
 游离状态实体以值方式进行传递，需要serializable

JPA是一套规范、有很多框架支持(如Hibernate3.2以上、Toplink,一般用Hibernate就行 oracle可以用toplink)
  
 JPQL
 1、与数据库无关的，基于实体的查询语言
 2、操作的是抽象持久化模型
 3、JPQL是一种强类型语言，一个JPQL语句中每个表达式都有类型 
 4、EJBQL的扩展
 5、支持projection(可以查询某个实体的字段而不需要查询整个实体)、批量操作(update、delete)、子查询、join、group by having(group by聚合后 having 聚合函数 比较 条件)
**1.2JPA开发过程**
  *  JPA配置文件声明持久化单元 --> 配置文件persistence.xml
  *  编写带标注的实体类
  *  编写Dao类
xml配置
 
 事务类型分为：RESOURCE_LOCAL
本地事务、JTA(java事务API)
 
###   注解 

 ```

JPA实体类的定义以及一些注释：
定义实体类：@Entity @Table(name = "students")
定义时间类型：@Temporal(TemporalType.DATE)
定义主键生成器：@Id @GeneratedValue(strategy = GenerationType.AUTO)
定义普通属性：@Column(name = "student_name", nullable = false, length = 20, unique = true)
定义枚举类型的映射： @Enumerated(EnumType.STRING) Column(name="sex",length=4,nullable=false)
定义不是持久化属性：@Transient
定义大数据类型注解：@log
@Basic是懒加载注解，只有去调用此属性的时候数据库才会把数据转载到内存中去

主键生成策略
@GeneratedValue(strategy=GenerationType.XXX)
GenerationType.IDENTITY，自增字段，SqlServer支持，Oracle不支持；
GenerationType.AUTO，JPA自动选择合适的策略，是默认选项；
GenerationType.SEQUENCE，通过序列产生主键，通过@SequenceGenerator注解指定序列名；MySql不支持；
GenerationType.TABLE，通过表产生主键，框架借助由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植；不同的JPA实现商生成的表名是不同的，如OpenJPA生成openjpa_sequence_table表，Hibernate生成一个hibernate_sequences表，而TopLink则生成sequence表；这些表都具有一个序列名和对应值两个字段，如SEQ_NAME和SEQ_COUNT；
 
时间字段标识
@Temporal(TemporalType.XXX)，来标识；
TemporalType.DATE：等于java.sql.Date；
TemporalType.TIME：等于java.sql.Time；
TemporalType.TIMESTAMP：等于java.sql.Timestamp；

```
**管理实体**
  *   Persistence
  *   EntityManagerFactory
  *   EntityManager  

** 实体类写法：**
1、必须有无参的构造函数
2、没有final类型的变量或方法
3、字段不可以是public类型的，只能通过get、set方法读写
`  Persistence.createEntityManagerFactory('persistence.xml中配置的persistence unit').createEntityManager()获取EntityManager `
###  1.3 实体的生命周期及实体管理器常用方法  
 
 ```

 EntityManager声明周期    Java对象   实体管理器    数据库
   1、 新实体(new)     存在    不存在     不存在    
   2、持久化实体(managed)   存在    存在     存在
   3、分离的实体(detached)   不存在    不存在     存在
   4、删除的实体(removed)   存在    存在     不存在

   常用方法
   1、persist(Object)       持久化
   2、remove(Object)       删除对象
   3、find(Class entityClass,Object key)  根据主键查询
   4、flush()         实体与底层同步，执行sql
   5、createQuery()       创建JPQL查询对象
   5、createNativeQuery()      根据普通SQL查询
   5、createNamedQuery()      命名查询@NamedQuerie标注 
   5、merge(Object)       将一个detached的实体持久化到EntityManager中
   5、close()         关闭管理器

———————————————————————————————————————
javax.persistence.Query   
int executeUpdate()   执行更新、删除、添加
Object getSingleResult() 执行查询(返回一条记录) 
List getResultList()  执行查询(返回结果链表)
Query setParameter(int position,object value) 给Query对象设置参数
Query setMaxResults(int maxResult)    给Query对象设置返回数
Query setFirstResult(int firstResult)  给Query对象设置返回偏移

```
##  2、实现JPA的增删改查功能，JQL的查询功能（基于面向对象的查询）： 

2.1、获得数据库的链接：
//取得实体管理工厂
EntityManagerFactory managerFactory=Persistence.createEntityManagerFactory("sample")
//创建实体管理对象
EntityManager em=managerFactory.createEntityManager();
2.2、保存数据：
//创建实体管理工厂
managerFactory=Persistence.createEntityManagerFactory("sample");
//通过实体工厂创建实体管理器
em=managerFactory.createEntityManager();
//打开事物
em.getTransaction().begin();
//保存数据
em.persist(new Student("小明1",20,new Date()));
em.getTransaction().commit();
em.close();
2.3、获取数据：

2.3.1、find获取数据方法：
//创建实体管理工厂
managerFactory=Persistence.createEntityManagerFactory("sample");
//通过实体工厂创建实体管理器
em=managerFactory.createEntityManager();
//Student指定要查找的实体Bean,1：表示实体Bean的标识符
student=em.find(Student.class,i);
em.close();
managerFactory.close();

2.3.2、getReferences获取数据：
Student student=null;
//创建实体管理工厂
managerFactory=Persistence.createEntityManagerFactory("sample");
//通过实体工厂创建实体管理器
em=managerFactory.createEntityManager();
//使用getReference()方法返回的是代理对象，只有在调用数据的时候才会发生装载行为
student=em.getReference(Student.class,1);

****************JQL查询*************


String jql="select o from Student o where o.name=?1";

Query query=em.createQuery(jql);
query.setParameter(1, "小明");
List<Student> students=query.getResultList();
Student student1=(Student)query.getSingleResult();
em.close();
managerFactory.close();
************************************
2.4更新数据：
//创建实体管理工厂
managerFactory=Persistence.createEntityManagerFactory("sample");
//通过实体工厂创建实体管理器
em=managerFactory.createEntityManager();
//打开事物
em.getTransaction().begin();

student.setName("小花");
student.setGender(Gender.女);
em.merge(student);
em.getTransaction().commit();
****************JQL查询*************
String jql="update Student o set o.name=:name,o.age=:age where o.id=:id";
Query query=em.createQuery(jql);
query.setParameter("name", "张三");
query.setParameter("age",20);
query.setParameter("id", 2);
int i=query.executeUpdate();
System.out.println("数据库受影响行数："+i);

em.close();
managerFactory.close();
************************************
2.5、删除数据：
//创建实体管理工厂
managerFactory=Persistence.createEntityManagerFactory("sample");
//通过实体工厂创建实体管理器
em=managerFactory.createEntityManager();
//打开事物
em.getTransaction().begin();
student=em.find(Student.class,2);
em.remove(student);
****************JQL查询*************
String jql="delete from Student o  where o.id=:id";

Query query=em.createQuery(jql);

query.setParameter("id", 2);

int i=query.executeUpdate();

System.out.println("数据库受影响行数："+i);

em.getTransaction().commit();

em.close();
managerFactory.close();
************************************
##  复合主键 
1.1、先定义一个复合主键类，其要求：` **必须继承接口Serializeable、使用@Embeddable注解该类、必须有一个public的无参构造方法、重载hashcode(),equals()方法** `
```

@Embeddable

public class CarPK implements Serializable{

private String startSite;

private String endSite;

public CarPK(){}

public CarPK(String startSite,String endSite)

{

this.startSite=startSite;

this.endSite=endSite;

}

public String getStartSite() {

return startSite;

}

public void setEndSite(String endSite) {

this.endSite = endSite;

}

@Override

public int hashCode() {

final int prime = 31;

int result = 1;

result = prime * result + ((endSite == null) ? 0 : endSite.hashCode());

result = prime * result + ((startSite == null) ? 0 : startSite.hashCode());

return result;

}

@Override

public boolean equals(Object obj) {

if (this == obj)

return true;

if (obj == null)

return false;

if (getClass() != obj.getClass())

return false;

CarPK other = (CarPK) obj;

if (endSite == null) {

if (other.endSite != null)

return false;

} else if (!endSite.equals(other.endSite))

return false;

if (startSite == null) {

if (other.startSite != null)

return false;

} else if (!startSite.equals(other.startSite))

return false;

return true;

}


public void setStartSite(String startSite) {

this.startSite = startSite;

}

public String getEndSite() {

return endSite;

}

```

1.2、如何使用该复合主键类呢？
```

@Entity

public class CarLine {

private CarPK id;

private String lineName;

private float price=0f;

public CarLine()

{

}

public CarLine(CarPK id)

{

this.id=id;

}

public CarLine(String startSite,String endSite,String lineName,float price)

{

this.id=new CarPK(startSite,endSite);

this.lineName=lineName;

this.price=price;

}

@EmbeddedId

public CarPK getId() {

return id;

}

public void setId(CarPK id) {

this.id = id;

}

@Column(length=20)

public String getLineName() {

return lineName;

}

public void setLineName(String lineName) {

this.lineName = lineName;

}

@Column(nullable=false)

public float getPrice() {

return price;

}

public void setPrice(float price) {

this.price = price;

}

}

```

<fc #ff0000>
主意事项：在定义一个实体bean时最好继承Serialable接口，那么为什么要实现此接口呢？
答：首先JPA(Hibernate)框架使用的是反射技术，反射里面就存在输入、输出流的东西所以实体类需要序列化，序列化的好处就是方便读写和网络传输这个在某些事时候是非常有必要的为了考虑到以后的扩展性.一般都实现序列化.</fc>
##  二、环境搭建 

2.1 添加JPA支持
1、准备JPA用到的jar包(JPA支持包)
antlr-2.7.6.jar
cglib-2.1.3.jar
classes12.jar
commons-collections-3.1.jar
dom4j-1.6.1.jar
ehcache-1.2.3.jar
ejb3-persistence.jar
hibernate3.jar
hibernate-annotations.jar
hibernate-cglib-repack-2.1_3.jar
hibernate-commons-annotations.jar
hibernate-entitymanager.jar
javassist-3.4.GA.jar
jta-1.1.jar
log4j-1.2.15.jar
persistence-api-1.0.jar
slf4j-api-1.5.2.jar
slf4j-log4j12.jar

2.2 添加配置文件
 1、项目中SRC目录下添加META-INF目录(与Web项目下META-INF同名)
 2、在新添加的META-INF中添加配置文件persistences.xml
###  persistence.xml配置信息 

```

<?xml version="1.0" encoding="UTF-8"?>
<persistence version="1.0" xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_1_0.xsd">
 <!-- name="持久化单元命名"  transaction-type="本地事务/JTA" -->
 <persistence-unit name="JPA" transaction-type="RESOURCE_LOCAL">
   <properties>
   <!-- 参数：数据库驱动名、地址、用户、密码、方言、显示执行SQL语句 -->
   <property name="hibernate.connection.driver_class" value=""/>
   <property name="hibernate.connection.driver_class" value="org.gjt.mm.mysql.Driver"/>
   <property name="hibernate.connection.url" value="jdbc:mysql://127.0.0.1:3306/JPA"/>
   <property name="hibernate.connection.username" value="root"/>
   <property name="hibernate.connection.password" value="123456"/>
   <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>
   <property name="hibernate.show_sql" value="true"/>
   
   <!-- 其他设置 -->
   <property name="minPoolSize" value="5"/>
   <property name="initialPoolSize" value="10"/>
   <property name="idleConnectionTestPeriod" value="120"/>
   <property name="acquireIncrement" value="10"/>
   <property name="checkoutTimeout" value="3600"/>
   <property name="numHelperThreads" value="4"/>
   <property name="maxStatements" value="400"/>
   <property name="maxStatementsPerConnection" value="20"/>
   <property name="maxIdleTime" value="180"/>
   <property name="acquireRetryAttempts" value="30"/>
   <property name="maxPoolSize" value="200"/>
  
  </properties>
 </persistence-unit>
</persistence>


```
  <!-- 供应商 -->
  <provider>org.hibernate.ejb.HibernatePersistence</provider>
  * persistence.xml配置信息 (Hibernate)
  * 数据库连接信息查询
  * 主要配置信息：
  * 事务类型：本地事务、JTA事务
  * JPA供应商
  * 数据库驱动、URL、User、Password
 3、在SRC目录下添加log4j.properties文件(显示数据库操作信息的)
```

实体注解
@Entity
@Table(name="Person")
public class Person {
 @Id
 @Column(name="pid")
 private Integer id;
 @Column(name="pname")
 private String name;
 public Integer getId() {
  return id;
 }
 public void setId(Integer id) {
  this.id = id;
 }
 public String getName() {
  return name;
 }
 public void setName(String name) {
  this.name = name;
 }
 
}

```
##  常用注解 

3.1 批注完全参考
@Entity要将 Java 类指定为 JPA 实体，请使用批注
详细信息
 
3.2 ID相关的
复合主键需要
1、 实现序列话
2、 重写hascode、equal方法
3、 有构造方法
@Embeddable 复合主键设置可以被引用
@EmbeddedId 引用独立复合主键ID
3.3主键生成策略
使用Hibernate的主键生成策略生成字符串主键
@Id
@GenericGenerator(name="generator",strategy="uuid")
@GeneratedValue(generator="generator")
@Column(name="id")

使用Hibernate的主键生成策略与其他类共享主键
@Id
 @GenericGenerator(name = "generator", 
             strategy = "foreign", 
              parameters = { 
              @Parameter(name = "property", value = "person") 
            })
  @GeneratedValue(generator = "generator")
@Column(name="cid")
3.4字段、添加字段、添加表关联
@Column 持久化字段
 可添加、可更新、可为空
 长度、表名、字段名、unique是否唯一

@JoinColumn
 name 列名
referencedColumnName指向对象的列名
unique约束唯一
@JoinColumns多个连接的列
@JoinColumns({
@JoinColumn(name="ADDR_ID", referencedColumnName="ID"),
@JoinColumn(name="ADDR_ZIP", referencedColumnName="ZIP")
    })
@JoinTable
@JoinTable(
name="EJB_PROJ_EMP",
joinColumns=@JoinColumn(name="EMP_ID", referencedColumnName="ID"),
inverseJoinColumns=@JoinColumn(name="PROJ_ID", referencedColumnName="ID")
)

3.5映射相关
@OneToOne
@ManyToOne
@ManyToMany
cascade级联、CURD
fetch一次性全部读取相关对象，还是lazy加载
optional 关联对象是否允许为空
targetEntity关联对象


3.6其他
 排序
@OrderBy("lastname ASC", "seniority DESC")

共享主键
@PrimaryKeyJoinColumn

标注为非持久话对象
@Transient
时间类型
@Temporal（TemporalType.…………）
枚举类型
@Enumerated(EnumType.…………)

@Lob //声明属性对应的是一个大文件数据字段。
@Basic(fetch = FetchType.LAZY) //设置为延迟加载，当我们在数据库中取这条记录的时候，不会去取

##  JPA映射 

4.1一对一映射
4.1.1共享主键映射
 1、一端提供主键、一端共享主键
  设置生成策略generator，主键值gereratedValue(generator=””)
  uuid字符串ID
  foreign引用别人ID作为自己的主键(需要设置引用对象参数)
 2、oneToOne
  targetEntity 关联的目标对象，是类名.class形式
fetch   抓去策略，有关联的一起抓去、还是lazy加载
cascade   级联
mappedBy  本对象被映射为*(在PrimaryKeyJoinColumn的另一端)
 3、PrimaryKeyJoinColumn设置在一端即可
  name    自身字段
  referenceColumnName指向对象的字段
 注意：
只要一个PrimaryKeyJoinColumn，另一端的oneToOne 设置mappedBy
都只有一个
注意：
1、向共享主键的对象设置提供主键的对象，然后持久化共享主键对象
 2、需要设置级联
3、共享主键端维护关系、提供主键端被维护使用mappedBy

 
Person提供主键
 @Id
    @GenericGenerator(name = "generator", strategy = "uuid")
    @GeneratedValue(generator = "generator")
 @Column(name="pid")
 private String id;
 
 @Column(name="pname",length=2)
 private String name;

 @OneToOne(mappedBy="person",fetch=FetchType.EAGER,targetEntity=Idcard.class)
 private Idcard idcard;

 Idcard共享主键
 @Id
 @GenericGenerator(name = "generator", 
            strategy = "foreign", 
            parameters = { 
              @Parameter(name = "property", value = "person") 
            })
    @GeneratedValue(generator = "generator")
 @Column(name="cid")
 private String id;
 @Column(name="cno")
 private String no;
 @OneToOne(targetEntity=Person.class,fetch=FetchType.EAGER,cascade=CascadeType.ALL)
 @PrimaryKeyJoinColumn(name="id",referencedColumnName="id")
 private Person person;

 @Id
 @GenericGenerator(name = "generator", 
            strategy = "foreign", 
            parameters = { 
              @Parameter(name = "property", value = "person") 
            })
            @GeneratedValue(generator = "generator")
 @Column(name="cid")
 private String id;
 @Column(name="cno")
 private String no;
 @OneToOne(mappedBy="idcard",fetch=FetchType.EAGER,targetEntity=Person.class)
 private Person person;


使用共享主键关联
Person p = new Person();
   p.setName("ader");
   Idcard idcard = new Idcard();
   idcard.setNo("321321");
   idcard.setPerson(p);

 

 


4.1.2关联外键映射
 关系的维护端
@OneToOne(级联)
 @JoinColumn(name="本表中关联字段",referencedColumnName="指向字段")
 被维护端
@OneToOne(mappedBy=””)

关系维护端添加一个字段作为外键指向被维护段。被维护端声明mappedBy
@Entity
 @Table(name="Test_Trousers")
 public class Trousers {
    @Id
    public Integer id;
    @OneToOne
    @JoinColumn(name = "zip_id")
    public TrousersZip zip;
 }

 @Entity
 @Table(name="Test_TrousersZip")
 public class TrousersZip {
    @Id
    public Integer id;
    @OneToOne(mappedBy = "zip")
    public Trousers trousers;
 }
4.1.3添加表关联
 添加关联表的一一关联
 @OneToOne
 @JoinTable(name ="关联表名称",
  joinColumns = @JoinColumn(name="关联本表的字段"),
  inverseJoinColumns = @JoinColumn(name="要关联的表的字段")
 )
 @Entity
 @Table(name="Test_People")
 public class People {
    @Id
    public Integer id;
    @OneToOne
    @JoinTable(name ="TestPeoplePassports",
      joinColumns = @JoinColumn(name="perple_fk"),
      inverseJoinColumns = @JoinColumn(name="passport_fk")
    )
    public Passport passport;
 }

 @Entity
 @Table(name="Test_Passport")
 public class Passport {
    @Id
    public Integer id;
    @OneToOne(mappedBy = "passport")
    public People people;
 }

 

4.2一对多关联
  
  我们维护的多是他们的关系 而不是实体所以建议使用添加关系表
  在多的一端：添加字段、或者添加表 设置级联添加
  在一端： mappedBy
  用法：new一个一的一端对象  set到多的对象中，在持久化多的一端

 4.2.1添加字段的一对多、多对一关联
 manyToOne  多对一
 oneToMany    用一个set存放对象
添加字段关联，一般是在多的一端维护关系，设置级联。一的一端mappedBy


 @Id
 @GenericGenerator(name="t1",strategy="uuid")
 @GeneratedValue(generator="t1")
 @Column(name="cid")
 private String cid;
 @Column(name="cname")
 private String cname;
 @OneToMany(fetch=FetchType.EAGER,mappedBy="cls")
 private Set<Student> studentSet;


 @Id
 @GenericGenerator(name="t2",strategy="uuid")
 @GeneratedValue(generator="t2")
 @Column(name="id")
 private String id;
 @Column(name="name")
 private String name;
 @ManyToOne(fetch=FetchType.EAGER,cascade=CascadeType.PERSIST,targetEntity=Clas.class)
 @JoinColumn(name="classid",referencedColumnName="cid")
 private Clas cls;


4.2.2添加表的一对多、多对一关联
 多的一端
@Id
 @GenericGenerator(name="generator",strategy="uuid")
 @GeneratedValue(generator="generator")
 @Column(name="id")
 private String id;
 @Column(name="name")
 private String name;
 @ManyToOne(targetEntity=C.class,cascade = CascadeType.PERSIST)
 @JoinTable(name="csrel",
   joinColumns=@JoinColumn(name="sid",referencedColumnName="id"),
   inverseJoinColumns=@JoinColumn(name="cid",referencedColumnName="cid")
 )
 private C c;
 一的一端
 @Id
 @GenericGenerator(name="generator",strategy="uuid")
 @GeneratedValue(generator="generator")
 @Column(name="cid")
 private String id;
 @Column(name="cname")
 private String name;
 
 @OneToMany(targetEntity=S.class,mappedBy="c")
 private Set<S> sset;


4.3多对多关联
添加表关联

假设Teacher 和 Student是多对多的关系，具体元数据声明如下:
pubic class Teacher{        
@ManyToMany(targetEntity = Student.class, cascade = CascadeType.PERSIST)        
@JoinTable(table = @Table(name = "M2M_TEACHER_STUDENT"),        
joinColumns = @JoinColumn(name = "TEACHER_ID", referencedColumnName = "ID"),  
inverseJoinColumns = @JoinColumn(name = "STUDENT_ID", referencedColumnName = "ID"))        
public List<Student> getStudents() {
return students;
}                                      
}

public class Student{        
@ManyToMany(targetEntity = Teacher.class, mappedBy = "students")        
public List<Teacher> getTeachers() {               
return teachers;        }
}
************************************************
Student student = em.find(Student.class, 1);
Teacher teacher = em.getReference(Teacher.class, 1);
student.removeTeacher(teacher);
在一个事务中
************************************************


4.4继承映射
继承关系映射在同一张表中

@Entity
@Table(name = "customer")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "flag", discriminatorType = DiscriminatorType.STRING)
public class Customer{
}

@Entity
@DiscriminatorValue(value = "A")
public class CustomerA extends Customer{
}

@Entity
@DiscriminatorValue(value = "B")
public class CustomerB extends Customer{
}

SINGLE_TABLE：父子类都保存到同一个表中，通过字段值进行区分。
JOINED：父子类相同的部分保存在同一个表中，不同的部分分开存放，通过表连 接获取完整数据；

TABLE_PER_CLASS：每一个类对应自己的表，一般不推荐采用这种方式。
--------------------------------------------------------------
##  数据库连接信息查询 

1、Hibernate JDBC属性 
属性名  用途 
hibernate.connection.driver_class jdbc驱动类
hibernate.connection.url jdbc URL
hibernate.connection.username 数据库用户
hibernate.connection.password 数据库用户密码
hibernate.dialect 数据库方言

2、驱动包
Db2：db2java.jar(JDBC直连)
：db2jcc.jar(Hibernate要用到此驱动jar文件和上面的驱动jar文件)
sybase：jconn3d.jar
MSSQL：msbase.jar+mssqlserver.jar+msutil.jar
MySQL：mysql-connector-java-3.1.12-bin.jar
Oracle10g：ojdbc14.jar

3、连接字符串(可以先添加驱动包然后到包里找Driver.class)
a、MSSQL
驱动：com.microsoft.jdbc.sqlserver.SQLServerDriver
地址：jdbc:microsoft:sqlserver:/ /127.0.0.1:1433;DatabaseName=数据库

b、Oracle10g
驱动：oracle.jdbc.driver.OracleDriver
地址：jdbc:oracle:thin:@127.0.0.1:1521:全局标识符

c、MySQL
驱动：org.gjt.mm.mysql.Driver
地址：jdbc:mysql:/ /127.0.0.1:3306/数据库

d、Access
驱动：sun.jdbc.odbc.JdbcOdbcDriver
地址：jdbc:odbc:Driver={MicroSoft Access Driver (*.mdb)};DBQ=c:\\demodb.mdb

e、DB2
驱动：COM.ibm.db2.jdbc.net.DB2Driver
地址：jdbc:db2:/ /127.0.0.1:6789/demodb
4、Hibernate SQL方言 (hibernate.dialect) 
RDBMS 方言 
DB2 org.hibernate.dialect.DB2Dialect
DB2 AS/400 org.hibernate.dialect.DB2400Dialect
DB2 OS390 org.hibernate.dialect.DB2390Dialect
PostgreSQL org.hibernate.dialect.PostgreSQLDialect
MySQL org.hibernate.dialect.MySQLDialect
MySQL with InnoDB org.hibernate.dialect.MySQLInnoDBDialect
MySQL with MyISAM org.hibernate.dialect.MySQLMyISAMDialect
Oracle (any version) org.hibernate.dialect.OracleDialect
Oracle 9i/10g org.hibernate.dialect.Oracle9Dialect
Sybase org.hibernate.dialect.SybaseDialect
Sybase Anywhere org.hibernate.dialect.SybaseAnywhereDialect
Microsoft SQL Server org.hibernate.dialect.SQLServerDialect
SAP DB org.hibernate.dialect.SAPDBDialect
Informix org.hibernate.dialect.InformixDialect
HypersonicSQL org.hibernate.dialect.HSQLDialect
Ingres org.hibernate.dialect.IngresDialect
Progress org.hibernate.dialect.ProgressDialect
Mckoi SQL org.hibernate.dialect.MckoiDialect
Interbase org.hibernate.dialect.InterbaseDialect
Pointbase org.hibernate.dialect.PointbaseDialect
FrontBase org.hibernate.dialect.FrontbaseDialect
Firebird org.hibernate.dialect.FirebirdDialect
