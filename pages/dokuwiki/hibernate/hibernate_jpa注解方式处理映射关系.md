title: hibernate_jpa注解方式处理映射关系 

#  Hibernate JPA注解方式处理映射关系 
本文参考：http://www.cnblogs.com/xiaoluo501395377/p/3374955.html
在hibernate中，通常配置对象关系映射关系有两种，一种是基于xml的方式，另一种是基于annotation的注解方式，熟话说，萝卜青菜，可有所爱，每个人都有自己喜欢的配置方式，我在试了这两种方式以后，发现使用annotation的方式可以更简介，所以这里就简单记录下通过annotation来配置各种映射关系，在hibernate4以后已经将annotation的jar包集成进来了，如果使用hibernate3的版本就需要引入annotation的jar包。

##  JPA基本属性注解 
JPA(Java Persistence API)是Sun官方提出的Java持久化规范。它为Java开发人员提供了一种对象/关系映射工具来管理Java应用中的关系数据
JPA规范要求` 在类路径的META-INF目录下放置persistence.xml `
JPA 中将一个类注解成实体类(entity class)有两种不同的注解方式：**基于属性(property-based)和基于字段**(field-based)的注解
  * 基于字段的注解, 就是直接将注解放置在实体类的字段的前面
  * 基于属性的注解, 就是直接将注解放置在实体类相应的**getter方法前面**(这一点和Spring正好相反),
但是` 同一个实体类中必须并且只能使用其中一种注解方式 `
Entity、Table、Id、GeneratedValue、Basic、Column、Temporal、Transient、Lob、Transient、SecondaryTable、Embeddable、Embedded、EmbeddedId

**(4)GeneratedValue**
@javax.persistence.GeneratedValue(generator="xxx",strategy=GenerationType.AUTO)
strategy:表示主键生成策略,有` AUTO,INDENTITY,SEQUENCE 和 TABLE ` 4种
分别表示让ORM框架自动选择,根据数据库的Identity字段生成,根据数据库表的Sequence字段生成,以有根据一个额外的表生成主键,**默认为AUTO** 
generator:表示主键生成器的名称,这个属性通常和ORM框架相关,例如,Hibernate可以指定uuid等主键生成方式. 
Hibernate UUID
@Id @GeneratedValue(generator="system-uuid")
@GenericGenerator(name="system-uuid",strategy = "uuid")

(5)Basic
@javax.persistence.Basic(fetch=FetchType.LAZY,optional=true)
fetch:抓取策略**,延时加载**与立即加载
optional:指定在生成数据库结构时字段是否允许为 null

(6)Column
@javax.persistence.Column(length=15,nullable=false,columnDefinition="",insertable=true,scale=10,table="",updatable=true)
@Column注解指定字段的详细定义
  * name:字段的名称,默认与属性名称一致 
  * nullable:是否允许为null,默认为true
  * unique:是否唯一,默认为false 
  * length:字段的长度,仅对String类型的字段有效 
  * columnDefinition:表示该字段在数据库中的实际类型。通常ORM框架可以根据属性类型自动判断数据库中字段的类型,但是**对于Date类型仍无法确定**数据库中字段类型究竟是DATE,TIME还是TIMESTAMP,
此外,String的默认映射类型为VARCHAR,如果要将String类型映射到特定数据库的BLOB或TEXT字段类型,该属性非常有用
如:`  @Column(name="BIRTH",nullable="false",columnDefinition="DATE") ` 
  * insertable:默认情况下,JPA持续性提供程序假设所有列始终包含在`  SQL INSERT 语句 `中。如果该列不应包含在这些语句中，请将 insertable 设置为 false 
  * updatable：列始终包含在`  SQL UPDATE 语句 `中。如果该列不应包含在这些语句中，请将 updatable 设置为 false 
  * table:实体的所有持久字段都存储到一个其名称为实体名称的数据库表中,如果该列与 @SecondaryTable表关联
需将 name 设置为相应辅助表名称的String名称

(10)Transient
@javax.persistence.Transient
@Transient表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性

(11)SecondaryTable 
@javax.persistence.SecondaryTable
将一个实体映射到多个数据库表中

(12)@Embeddable
@javax.persistence.Embeddable
嵌套映射,在被嵌套的类中使用Embeddable注解,说明这个就是一个可被嵌套的类,使用 @Embedded

##  @OneToMany级联属性 
@OneToMany(mappedBy="room", cascade={CascadeType.REMOVE})
CascadeType.PRESIST 级联持久化（保存）操作（持久保存拥有方实体时，也会持久保存该实体的所有相关数据。）
# # # # # ===
CascadeType.REMOVE 级联删除操作（删除一个实体时，也会删除该实体的所有相关数据。）
# # # # # ===
CascadeType.MERGE 级联更新（合并）操作（将分离的实体重新合并到活动的持久性上下文时，也会合并该实体的所有相关数据。）
# # # # # ===
CascadeType.REFRESH 级联刷新操作 （只会查询获取操作）
# # # # # ===
CascadeType.ALL 包含以上全部级联操作

Refresh的作用:假如有一个条数据(就有name[值为B]和sex[值为male]两个字段),A用户取出来在进行修改操作(修改name为A),正在A修改的过程中(未提交表单),B用户也对这条数据进行修改操作(修改sex为female),B先将性别修改后提交数据库...接着A用户也提交表单,但是,此时在entityManager中的持久化实体的性别为male,没有更新为B用户修改成的female,所以此时执行一次Refresh操作,就会将该实体更新为数据库中的最新记录,然后再进行提交..做级联的时候就会将关联的实体的也获取最新的然后在更新,前提是要执行Refresh操作,CasCadeType.Refresh才会生效

Merge的作用:你要先去了解持久化实体在entityManager中的几种状态,新建,游离,托管(不是脱管),删除状态,Merge对实体进行操作时,会区分这个实体的状态,假如这个实体处于托管状态,就应该使用merge,否则会报异常..同样,做级联的时候执行merge操作,CasCadeType.Merge也会对关联实体生效
**级联操作很强大，也很危险。**所以，不可以盲目的崇拜和使用级联。应该根据自己的实际业务需求来选择是否需要添加对应的级联操作。


##  映射组成关系 
<note tip>请移步查看[见此段](http://wiki.xby1993.net/doku.php?id=hibernate:hibernate%E6%98%A0%E5%B0%84%E6%8C%81%E4%B9%85%E5%8C%96%E7%B1%BB#映射组件或被嵌入类)</note>

##  一、单对象操作 

```

@Entity　　--->　　如果我们当前这个bean要设置成实体对象，就需要加上Entity这个注解
@Table(name="t_user")　　---->　　设置数据库的表名
public class User
{
    private int id;
    private String username;
    private String password;
    private Date born;
    private Date registerDate;

    @Column(name="register_date")　　--->　　Column中的name属性对应了数据库的该字段名字，里面还有其他属性，例如length，nullable等等
    public Date getRegisterDate()
    {
        return registerDate;
    }
    public void setRegisterDate(Date registerDate)
    {
        this.registerDate = registerDate;
    }

    @Id　　--->　　定义为数据库的主键ID　　(建议不要在属性上引入注解，因为属性是private的，如果引入注解会破坏其封装特性，所以建议在getter方法上加入注解)
    @GeneratedValue　　---->　　ID的生成策略为自动生成　　
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
 　　............
}

```
最后只需要在` hibernate.cfg.xml `文件里面将该实体类加进去即可：
```

<!-- 基于annotation的配置 -->
        <mapping class="com.xiaoluo.bean.User"/>
<!-- 基于hbm.xml配置文件 -->
        <mapping resource="com/xiaoluo/bean/User.hbm.xml"/>

```
这样我们就可以写测试类来进行我们的CRUD操作了。

##  二、一对多的映射(one-to-many) 
这里我们定义了两个实体类，一个是ClassRoom，一个是Student，这两者是一对多的关联关系。
使用JPA的时候，如果A B两个实体间是一对多，多对一的关系，如果不在` @OneToMany里加入mappedBy属性 `会导致自动生成一个多余的中间表。
比如：
```

@Entity
public class A {
	/**由于设置了mappedBy属性，所以这样写会只成生成表A和表B，B中会有一个到表A的外键。但是如果不加mappedBy=”a”， 那么就会再生成一张A_B表。
	*那么 mappedBy="填什么呢"。填的是这个一对多关系的名称。比如这里，A与B存在名为a的一对多关系。
	*/
    @OneToMany(mappedBy="a") //
    public Set<B> bs = new HashSet<B>(0);
}
 
@Entity
public class B {
    @ManyToOne
    public A a;
}

```
一下为撕裂：
ClassRoom类：
```


@Entity
@Table(name="t_classroom")
public class ClassRoom
{
    private int id;
    private String className;
    private Set<Student> students;
    
    public ClassRoom()
    {
        students = new HashSet<Student>();
    }
    
    public void addStudent(Student student)
    {
        students.add(student);
    }

    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public String getClassName()
    {
        return className;
    }

    public void setClassName(String className)
    {
        this.className = className;
    }
/**　OneToMany指定了一对多的关系，mappedBy="room"指定了由多的那一方来维护关联关系，mappedBy指的是多的一方对1的这一方的依赖的属性，(注意：如果没有指定由谁来维护关联关系，则系统会给我们创建一张中间表) */
    @OneToMany(mappedBy="room")　　
  /**如需指定级联，则添加cascade属性，级联共有四个属性，这里指定级联删除。 CascadeType属性有四个值，其中REMOVE属性是实现级联删除，要实现级联删除
 在父栏必需添加CascadeType.REMOVE标注，这是级联删除的关键*/
   @OneToMany(mappedBy="room", cascade={CascadeType.REMOVE})
    @LazyCollection(LazyCollectionOption.EXTRA)　　--->　　LazyCollection属性设置成EXTRA指定了当如果查询数据的个数时候，只会发出一条 count(*)的语句，提高性能
    public Set<Student> getStudents()
    {
        return students;
    }

    public void setStudents(Set<Student> students)
    {
        this.students = students;
    }
    
}

```
Student类：
```

@Entity
@Table(name="t_student")
public class Student
{
    private int id;
    private String name;
    private int age;
    private ClassRoom room;
    
    @ManyToOne(fetch=FetchType.LAZY)　　---> ManyToOne指定了多对一的关系，fetch=FetchType.LAZY属性表示在多的那一方通过延迟加载的方式加载对象(默认不是延迟加载)
    @JoinColumn(name="rid")　　--->　　通过 JoinColumn 的name属性指定了外键的名称 rid　(注意：如果我们不通过JoinColum来指定外键的名称，系统会给我们声明一个名称)
    public ClassRoom getRoom()
    {
        return room;
    }
    public void setRoom(ClassRoom room)
    {
        this.room = room;
    }
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    public int getAge()
    {
        return age;
    }
    public void setAge(int age)
    {
        this.age = age;
    }
    
}

```
##  三、一对一映射(One-to-One) 
一对一关系这里定义了一个Person对象以及一个IDCard对象
Person类：
```

@Entity
@Table(name="t_person")
public class Person
{
    private int id;
    private String name;
    private IDCard card;
    
    @OneToOne(mappedBy="person")　　--->　　指定了OneToOne的关联关系，mappedBy同样指定由对方来进行维护关联关系
    public IDCard getCard()
    {
        return card;
    }
    public void setCard(IDCard card)
    {
        this.card = card;
    }
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    
}

```
IDCard类：

```

@Entity
@Table(name="t_id_card")
public class IDCard
{
    private int id;
    private String no;
    private Person person;
    
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getNo()
    {
        return no;
    }
    public void setNo(String no)
    {
        this.no = no;
    }
    @OneToOne　　--->　　OnetoOne指定了一对一的关联关系，一对一中随便指定一方来维护映射关系，这里选择IDCard来进行维护
    @JoinColumn(name="pid")　　--->　　指定外键的名字 pid
    public Person getPerson()
    {
        return person;
    }
    public void setPerson(Person person)
    {
        this.person = person;
    }
}

```
注意:在判断到底是谁维护关联关系时，**可以通过查看外键，哪个实体类定义了外键，哪个类就负责维护关联关系**。

##  四、Many-to-Many映射(多对多映射关系) 
多对多这里通常有两种处理方式，
  * 一种是通过建立一张中间表，然后由任一一个多的一方来维护关联关系，
  * 另一种就是将多对多拆分成两个一对多的关联关系
  * 
**方式1.通过中间表由任一一个多的一方来维护关联关系**
Teacher类：
```

@Entity
@Table(name="t_teacher")
public class Teacher
{
    private int id;
    private String name;
    private Set<Course> courses;
    
    public Teacher()
    {
        courses = new HashSet<Course>();
    }
    public void addCourse(Course course)
    {
        courses.add(course);
    }
    
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    @ManyToMany(mappedBy="teachers")　　--->　　表示由Course那一方来进行维护
    public Set<Course> getCourses()
    {
        return courses;
    }
    public void setCourses(Set<Course> courses)
    {
        this.courses = courses;
    }
    
}

```
Course类：
```

@Entity
@Table(name="t_course")
public class Course
{
    private int id;
    private String name;
    private Set<Teacher> teachers;
    
    public Course()
    {
        teachers = new HashSet<Teacher>();
    }
    public void addTeacher(Teacher teacher)
    {
        teachers.add(teacher);
    }
    @ManyToMany　　　--->　ManyToMany指定多对多的关联关系
    @JoinTable(name="t_teacher_course", joinColumns={ @JoinColumn(name="cid")}, 
    inverseJoinColumns={ @JoinColumn(name = "tid") })　　--->　　因为多对多之间会通过一张中间表来维护两表直接的关系，所以通过 JoinTable 这个注解来声明，name就是指定了中间表的名字，JoinColumns是一个 @JoinColumn类型的数组，表示的是我这方在对方中的外键名称，我方是Course，所以在对方外键的名称就是 cid，inverseJoinColumns也是一个 @JoinColumn类型的数组，表示的是对方在我这放中的外键名称，对方是Teacher，所以在我方外键的名称就是 tid
    public Set<Teacher> getTeachers()
    {
        return teachers;
    }

    public void setTeachers(Set<Teacher> teachers)
    {
        this.teachers = teachers;
    }

    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

}

```

**2.将Many-to-Many拆分成两个One-to-Many的映射(Admin、Role、AdminRole)**
Admin类：
```

@Entity
@Table(name="t_admin")
public class Admin
{
    private int id;
    private String name;
    private Set<AdminRole> ars;
    public Admin()
    {
        ars = new HashSet<AdminRole>();
    }
    public void add(AdminRole ar)
    {
        ars.add(ar);
    }
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    @OneToMany(mappedBy="admin")　　--->　　OneToMany关联到了AdminRole这个类，由AdminRole这个类来维护多对一的关系，mappedBy="admin"
    @LazyCollection(LazyCollectionOption.EXTRA)　　
    public Set<AdminRole> getArs()
    {
        return ars;
    }
    public void setArs(Set<AdminRole> ars)
    {
        this.ars = ars;
    }
}

```

Role类：
```


@Entity
@Table(name="t_role")
public class Role
{
    private int id;
    private String name;
    private Set<AdminRole> ars;
    public Role()
    {
        ars = new HashSet<AdminRole>();
    }
    public void add(AdminRole ar)
    {
        ars.add(ar);
    }
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    @OneToMany(mappedBy="role")　　--->　　OneToMany指定了由AdminRole这个类来维护多对一的关联关系，mappedBy="role"
    @LazyCollection(LazyCollectionOption.EXTRA)
    public Set<AdminRole> getArs()
    {
        return ars;
    }
    public void setArs(Set<AdminRole> ars)
    {
        this.ars = ars;
    }
}

```
AdminRole类：
```

@Entity
@Table(name="t_admin_role")
public class AdminRole
{
    private int id;
    private String name;
    private Admin admin;
    private Role role;
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    @ManyToOne　　--->　　ManyToOne关联到Admin
    @JoinColumn(name="aid")　　
    public Admin getAdmin()
    {
        return admin;
    }
    public void setAdmin(Admin admin)
    {
        this.admin = admin;
    }
    @ManyToOne　　--->　　
    @JoinColumn(name="rid")
    public Role getRole()
    {
        return role;
    }
    public void setRole(Role role)
    {
        this.role = role;
    }
}

```

小技巧:通过hibernate来进行插入操作的时候，不管是一对多、一对一还是多对多，` **都只需要记住一点，在哪个实体类声明了外键，就由哪个类来维护关系** `，
**在保存数据时，总是先保存的是没有维护关联关系的那一方的数据，后保存维护了关联关系的那一方的数据**，如：

　　　　```

　　　Person p = new Person();
            p.setName("xiaoluo");
            session.save(p);
            
            IDCard card = new IDCard();
            card.setNo("1111111111");
            card.setPerson(p);
            session.save(card);

```
以上就是对hibernate annotation注解方式来配置映射关系的一些总结。
##  JPA复合主键的映射 
一个实体的主键可能同时构成，当且仅当多个字段的值完全相同时，才认为是相同的实体对象。
**复合主键映射时，通常构成将复合主键的多个字段单独抽取出来建一个类作为符合主键类。**
符合主键类必须满足以下几点要求
  * 必须实现` Serializable `接口。
  * 必须有默认的` public无参数的构造方法 `。
  * 必须` 覆盖equals和hashCode方法 `。equals方法用于判断两个对象是否相同，EntityManger通过find方法来查找Entity时，是根据equals的返回值来判断的。

```

@Embeddable
public class AirLinePK implements Serializable {
    @Column(nullable=false,length=3,name="LEAVECITY")
    private String leavecity;
    @Column(nullable=false,length=3,name="ARRIVECITY")
    private String arrivecity;
    public AirLinePK(){}
    public AirLinePK(String leavecity, String arrivecity) {
        this.leavecity = leavecity;
        this.arrivecity = arrivecity;
    }
    @Override
    public int hashCode() {
        int hash = 0;
        hash += (this.leavecity!=null&& this.arrivecity!=null ? (this.leavecity+ "-"+this.arrivecity).hashCode() : 0);
        return hash;
    }
    @Override
    public boolean equals(Object object) {
        if (!(object instanceof AirLinePK)){
            returnfalse;
        }
        AirLinePK other = (AirLinePK)object;
        if (this.leavecity !=other.leavecity && (this.leavecity == null ||!this.leavecity.equalsIgnoreCase(other.leavecity))) return false;
        if (this.arrivecity !=other.arrivecity && (this.arrivecity == null ||!this.arrivecity.equalsIgnoreCase(other.arrivecity))) return false;
        return true;
    }
    //省略getter setter...
}

```
```

@Entity
public class AirLine implements Serializable {
    @EmbeddedId
    private AirLinePK id;
    @Column(length=15)
    private String airLineNum;
    public AirLine(){}
    @Override
    public int hashCode() {
        int hash = 0;
        hash += (this.id != null ?this.id.hashCode() : 0);
        return hash;
    }
    @Override
    public boolean equals(Object object) {
        if (!(object instanceof AirLine)) {
            returnfalse;
        }
        AirLine other = (AirLine)object;
        if (this.id != other.id &&(this.id == null || !this.id.equals(other.id))) return false;
        return true;
    }
    //省略getter setter...
}

```
##  实体继承实体的映射策略 
参考：http://blog.csdn.net/mhmyqn/article/details/37996673
注：这里所说的实体指的是@Entity注解的类
**继承映射**使用` @Inheritance `来注解，它的` strategy属性 `的取值由枚举InheritanceType来定义（包括` SINGLE_TABLE、TABLE_PER_CLASS、JOINED `，分别对应三种继承策略）。@Inheritance注解只能作用于继承结构的超类上。如果不指定继承策略，**默认使用SINGLE_TABLE。**
JPA提供了三种继承映射策略：
1、** 一个类继承结构一个表的策略**。这是继承映射的默认策略。即如果实体类B继承实体类A，实体类C也继承自实体A，那么**只会映射成一个表**，这个表中包括了实体类A、B、C中所有的字段，JPA使用一个叫做“` discriminator列 `”来区分某一行数据是应该映射成哪个实体。注解为` ：@Inheritance(strategy = InheritanceType.SINGLE_TABLE) `
2、** 联合子类策略。**这种情况下父类也被放到一张单独的表中，子类的字段被映射到各自的表中，这些字段包括父类中的字段，并执行一个**join操作**来实例化子类。注解为：` @Inheritance(strategy = InheritanceType.JOINED) `
3、 **每个具体的类一个表的策略**。注解为` ：@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) `
一、一个类继承结构一个表的策略
这种策略中，一个继承结构中的所有类都被映射到一个表中。该表中有一列被当作“discriminator列”，即使用该列来识别某行数据属于某个指定的子类实例。
这种映射策略对实体和涉及类继承结构的**查询的多态系统提供了很好的支持**。但**缺点是要求与子类的指定状态对应的列可以为空。**
```

@Entity  
@Table(name = "EMP")  
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)  
@DiscriminatorColumn(name = "emp_type")  
public class Employee implements Serializable {  
  
    private static final long serialVersionUID = -7674269980281525370L;  
      
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    protected Integer empId;  
      
    @Column  
    protected String name;  
  
    // getter/setter方法  
      
}  

@Entity  
@DiscriminatorValue("FT")  
public class FullTimeEmployee extends Employee {  
  
    private static final long serialVersionUID = 9115429216382631425L;  
  
    @Column  
    private Double salary;  
  
    // getter/setter方法  
      
}  
  
@Entity  
@DiscriminatorValue("PT")  
public class PartTimeEmployee extends Employee {  
  
    private static final long serialVersionUID = -6122347374515830424L;  
  
    @Column(name = "hourly_wage")  
    private Float hourlyWage;  
  
    // getter/setter方法  
  
}  

```
其中，超类的@DiscriminatorColumn注解可以省略，默认的“discriminator列”名为DTYPE，默认类型为STRING。
**@DiscriminatorColumn注解只能使用在超类上**，不能使用到具体的子类上。discriminatorType的值由` DiscriminatorType枚举定义，包括STRING、CHAR、INTEGER。 `如果指定了discriminatorType，那么子类上@ DiscriminatorValue注解的值也应该是相应类型。
**@DiscriminatorValue注解只能使用在具体的实体子类上。**同样@DiscriminatorValue注解也可以省略，默认使用类名作为值。
上面的例子中，只会生成一个表，包含了字段emp_type、empId、name、salary、hourly_wage。当保存FullTimeEmployee时，emp_type的值为“FT”， 当保存PartTimeEmployee时，emp_type的值为“PT”。

二、联合子类策略
这种策略**超类会被映射成一个单独的表**，**每个子类也会映射成一个单独的表**。子类对应的表中只包括自身属性对应的字段，默认情况下使用主键作为超类对应的表的外键。
这种策略对于实体间的多态关系提供了很好的支持。但**缺点是实例化子类实例时需要一个或多个表的关联操作。在深层次的继承结构中，这会导致性能很低**。实例如下：
```

@Entity  
@Table(name = "EMP")  
@Inheritance(strategy = InheritanceType.JOINED)  
public class Employee implements Serializable {  
  
    private static final long serialVersionUID = -7674269980281525370L;  
      
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    protected Integer empId;  
      
    @Column  
    protected String name;  
  
    // getter/setter方法  
      
}   
  
@Entity  
@Table(name = "FT_EMP")  
public class FullTimeEmployee extends Employee {  
  
    private static final long serialVersionUID = 9115429216382631425L;  
  
    @Column  
    private Double salary;  
  
    // getter/setter方法  
      
}  
  
  
@Entity  
@Table(name = "PT_EMP")  
public class PartTimeEmployee extends Employee {  
  
    private static final long serialVersionUID = -6122347374515830424L;  
  
    @Column(name = "hourly_wage")  
    private Float hourlyWage;  
  
    // getter/setter方法  
  

```
这会**映射成三个具体的表**，分别是，Employee对应EMP表，字段包括**empId**、name；FullTimeEmployee对应FT_EMP表，字段包括**empId**、salary；PartTimeEmployee对应PT_EMP表，字段包括**empId**、hourly_wage。其中，` 表FT_EMP和PT_EMP中的empId作为表EMP的外键，同是它也是主键 `。**默认情况下，使用超类的主键作为子类的主键和外键**。当然，可以通过` @PrimaryKeyJoinColumn `注解来自己指定外键的名称，如FullTimeEmployee使用` @PrimaryKeyJoinColumn(name = "FT_EMPID") `注解，那么该子类实体的字段为FT_EMPID、name，FT_EMPID作为表FT_TIME的主键，同时它也是EMP表的外键。
**子类实体每保存一条数据，会在EMP表中插入一条记录**，如FT_EMP表插入一条数据，会先在EMP表中插入name，并生成empId，再在FT_EMP表中插入empId和salary。PT_EMP同理。
不管超类是抽象类还是具体类，都会生成对应的表。

三、每个具体的类一个表的策略
这种映射策略**每个类都会映射成一个单独的表，类的所有属性，包括继承的属性都会映射成表的列。**
这种映射策略的**缺点是：对多态关系的支持有限，当查询涉及到类继承结构时通常需要发起SQL UNION查询。**
```

@Entity  
@Table(name = "EMP")  
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)  
public class Employee implements Serializable {  
  
    private static final long serialVersionUID = -7674269980281525370L;  
      
    @Id  
    @GeneratedValue(strategy = GenerationType.TABLE)  
    protected Integer empId;  
      
    @Column  
    protected String name;  
  
    // getter/setter方法  
      
}  
  
@Entity  
@Table(name = "FT_EMP")  
public class FullTimeEmployee extends Employee {  
  
    private static final long serialVersionUID = 9115429216382631425L;  
  
    @Column  
    private Double salary;  
  
    // getter/setter方法  
      
}  
  
@Entity  
@Table(name = "PT_EMP")  
public class PartTimeEmployee extends Employee {  
  
    private static final long serialVersionUID = -6122347374515830424L;  
  
    @Column(name = "hourly_wage")  
    private Float hourlyWage;  
  
    // getter/setter方法  
  
}  

```
**这会映射成三个具体的表**，分别是，Employee对应EMP表，字段包括empId、name；FullTimeEmployee对应FT_EMP表，字段包括empId、salary；PartTimeEmployee对应PT_EMP表，字段包括empId、hourly_wage。
其中，**表FT_EMP和PT_EMP中的empId和EMP表的empId没有任何关系。**子类实体每保存一条数据，**EMP表中不会插入记录。**
而且主键的生成策略` 不能使用 `GenerationType.AUTO或GenerationType.IDENTITY，**否则会出现异常：**
org.hibernate.MappingException: Cannot use identity column key generation with <union-subclass> mapping for: com.mikan.PartTimeEmployee
因为` TABLE_PER_CLASS策略每个表都是单独的，没有并且各表的主键没有任何关系， `所以不能使用GenerationType.AUTO或GenerationType.IDENTITY主键生成策略，可以使用GenerationType.TABLE。
具体可参考：http://stackoverflow.com/questions/916169/cannot-use-identity-column-key-generation-with-union-subclass-table-per-clas
**如果超类是抽象类，那么不会生成对应的表。如果超类是具体的类，那么会生成对应的表。**
以上实例使用JPA的hibernate实现测试通过。


##  实体类继承映射超类（Mapped Superclasses） 
实体可以继承自一个**超类**，这个超类提供了持久化实体状态（即属性或字段）和映射信息，**但它本身不是一个实体**。通常情况下，这种超类映射的的目的是定义多个实体共有的状态和映射信息。
映射超类和实体不一样，它不能够被查询，所以不能作为参数传递给EntityManager或Query 接口进行操作。映射超类定义的持久化关系必须是单向的。
抽象类或具体的类都可以作为映射超类，使用` @MappedSuperclass注解（或mapped-superclass XML描述符元素）来指定映射超类 `。
**映射超类不会生成单独的表，它的映射信息作用于继承自它的实体类。**
映射超类能够像实体类一样被映射，只是它的映射将作用于继承自它的实体类，因为它本身不存在单独的表。当作用于子类时，继承的映射信息将作用于子类对应的表上。子类可以通过@AttributeOverride和AssociationOverride注解或对应的XML描述符元素来覆盖映射超类的映射信息。
```

@MappedSuperclass  
public class Employee implements Serializable {  
  
    private static final long serialVersionUID = -7674269980281525370L;  
      
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    protected Integer empId;  
      
    @Column  
    protected String name;  
  
    // getter/setter方法  
      
}  
  
@Entity  
@Table(name = "FT_EMP")  
public class FullTimeEmployee extends Employee {  
  
    private static final long serialVersionUID = 9115429216382631425L;  
  
    // 继承映射超类的empId和name属性  
  
    @Column  
    private Double salary;  
  
    // getter/setter方法  
      
}  
  
  
@Entity  
@Table(name = "PT_EMP")  
public class PartTimeEmployee extends Employee {  
  
    private static final long serialVersionUID = -6122347374515830424L;  
  
    // 继承映射超类的empId和name属性  
  
    @Column(name = "hourly_wage")  
    private Float hourlyWage;  
  
    // getter/setter方法  
      
}  

```
其中Employee是映射超类，它包括两个字段（即上面所说的状态）和相应的映射信息，只是这些映射信息都由子类实体FullTimeEmployee和PartTimeEmployee继承，**它本身不会生成对应的表。只会生成FT_EMP和PT_EMP两个表**，**这两个表中的字段包括了子类本身的字段和从映射超类中继承的字段**。即FT_EMP表的字段为：empId、name、salary，而PT_EMP表的字段为：empId、name、hourly_wage。