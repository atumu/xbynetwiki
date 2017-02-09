title: hibernate入门 

#  Hibernate入门 
基于Hibernate3.2以上版本。
源码下载:http://www.manning.com/bauer2/
主要内容：
  * 用Hibernate与JPA实现Hello world
  * 正向与反向工具
  * Hibernate配置与整合

依赖：
![](/data/dokuwiki/hibernate/pasted/20150817-031832.png)
##  Hibernate实现方式 
**创建领域模型类：**
```

public class Message implements Serilizable {
    private Long id;
    private String text;
    private Message nextMessage;

    public Message() {}

    public Message(String text) {
        this.text = text;
    }

    public Long getId() {
        return id;
    }
    private void setId(Long id) {
        this.id = id;
    }

    public String getText() {
        return text;
    }
    public void setText(String text) {
        this.text = text;
    }

     public Message getNextMessage() {
         return nextMessage;
     }
     public void setNextMessage(Message nextMessage) {
         this.nextMessage = nextMessage;
     }
}


```
**创建映射文件：**
```

<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD//EN"
    "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">

<hibernate-mapping>

    <class
        name="hello.Message"
        table="MESSAGES">

        <id
            name="id"
            column="MESSAGE_ID">
            <generator class="native"/>
        </id>

        <property
            name="text"
            column="MESSAGE_TEXT"/>

        <many-to-one
            name="nextMessage"
            cascade="all"
            column="NEXT_MESSAGE_ID"
            foreign-key="FK_NEXT_MESSAGE"/>

    </class>

</hibernate-mapping>


```
**存储与加载对象**
```

public class HelloWorld {

    public static void main(String[] args) {

        // ############################################################################

        // First unit of work
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = session.beginTransaction();

        Message message = new Message("Hello World");
        session.save(message);

        tx.commit();
        session.close();

        // ############################################################################

        // Second unit of work
        Session secondSession = HibernateUtil.getSessionFactory().openSession();
        Transaction secondTransaction = secondSession.beginTransaction();

        List messages =
            secondSession.createQuery("from Message m order by m.text asc").list();

        System.out.println( messages.size() + " message(s) found:" );

        for ( Iterator iter = messages.iterator(); iter.hasNext(); ) {
            Message loadedMsg = (Message) iter.next();
            System.out.println( loadedMsg.getText() );
        }

        secondTransaction.commit();
        secondSession.close();

        // ############################################################################

        // Third unit of work
        Session thirdSession = HibernateUtil.getSessionFactory().openSession();
        Transaction thirdTransaction = thirdSession.beginTransaction();

        // message.getId() holds the identifier value of the first message
        Message loadedMessage = (Message) thirdSession.get( Message.class, message.getId());
        loadedMessage.setText( "Greetings Earthling" );
        loadedMessage.setNextMessage(
            new Message( "Take me to your leader (please)" )
        );

        thirdTransaction.commit();
        thirdSession.close();

        // ############################################################################

        // Final unit of work (just repeat the query)
        // TODO: You can move this query into the thirdSession before the commit, makes more sense!
        Session fourthSession = HibernateUtil.getSessionFactory().openSession();
        Transaction fourthTransaction = fourthSession.beginTransaction();

        messages =
            fourthSession.createQuery("from Message m order by m.text asc").list();

        System.out.println( messages.size() + " message(s) found:" );

        for ( Iterator iter = messages.iterator(); iter.hasNext(); ) {
            Message loadedMsg = (Message) iter.next();
            System.out.println( loadedMsg.getText() );
        }

        fourthTransaction.commit();
        fourthSession.close();


        // Shutting down the application
        HibernateUtil.shutdown();
    }
}

```
**HibernateUtil工具类：**
```

public class HibernateUtil {

  private static SessionFactory sessionFactory;

  static {
    try {
       sessionFactory = new Configuration().configure().buildSessionFactory();
    } catch (Throwable ex) {
       throw new ExceptionInInitializerError(ex);
    }
  }

  public static SessionFactory getSessionFactory() {
      // Alternatively, we could look up in JNDI here
      return sessionFactory;
  }

  public static void shutdown() {
      // Close caches and connection pools
      getSessionFactory().close();
  }
}

```
**Hibernate.cfg.xml配置文件**
```

<hibernate-configuration>
    <session-factory>

        <property name="hibernate.connection.driver_class">org.hsqldb.jdbcDriver</property>
        <property name="hibernate.connection.url">jdbc:hsqldb:hsql://localhost</property>
        <property name="hibernate.connection.username">sa</property>

        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.timeout">300</property>
        <property name="hibernate.c3p0.max_statements">50</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>

        <!-- SQL to stdout logging
        <property name="show_sql">true</property>
        <property name="format_sql">true</property>
        <property name="use_sql_comments">true</property>
        -->

        <property name="dialect">org.hibernate.dialect.HSQLDialect</property>

        <mapping resource="hello/Message.hbm.xml"/>

    </session-factory>
</hibernate-configuration>

```
###  Hibernate主要接口 
SessionFactory,Session,Transaction,Query
**SessionFactory是线程安全的**，可以被共享的，一般一个应用程序应该只有一个SessionFactory实例。
**Session不是线程安全的**，不可以被共享，属于单个线程。
###  Hibernate配置启动分析 
sessionFactory = new Configuration().configure().buildSessionFactory();
其中new Configuration()会在classpath根目录下搜索hibernate.properties文件，关于数据池的配置可以移动到该文件中。
调用configure()会在classpath根目录下搜索hibernate.cfg.xml文件。如果无法找到则抛出异常。当然可以显式配置路径。 .configure("/your_path")
###  启用日志和统计 
Hibernate会自动搜寻log4j.jar然后启动日志系统。

```

# Direct log messages to stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

# Root logger option
log4j.rootLogger=WARN, stdout

# Hibernate logging options (INFO only shows startup messages)
#log4j.logger.org.hibernate=INFO

# Log JDBC bind parameter runtime arguments
#log4j.logger.org.hibernate.type=DEBUG

```
##  JPA实现方式 
使用Hibernate Annotations,用注解代替Hibernate XML映射文件。这时需要` Hibernate-Annotation.jar `和ejb3-persistence.jar库。
**注解的领域模型**
```

@Entity
@Table(name = "MESSAGES")
public class Message {

    @Id @GeneratedValue
    @Column(name = "MESSAGE_ID")
    private Long id;

    @Column(name = "MESSAGE_TEXT")
    private String text;

    @ManyToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "NEXT_MESSAGE_ID")
    private Message nextMessage;

    Message() {}

    public Message(String text) {
        this.text = text;
    }

    public Long getId() {
        return id;
    }
    private void setId(Long id) {
        this.id = id;
    }

    public String getText() {
        return text;
    }
    public void setText(String text) {
        this.text = text;
    }

     public Message getNextMessage() {
         return nextMessage;
     }
     public void setNextMessage(Message nextMessage) {
         this.nextMessage = nextMessage;
     }
}


```
**JPA配置文件：**
参考：http://docs.jboss.org/hibernate/orm/4.2/manual/en-US/html_single/#d5e747
http://technique-digest.iteye.com/blog/733022
http://technique-digest.iteye.com/blog/733019
` persistence.xml必须位于classpath下的META-INF/目录下，而不是默认的META-INF `
```

<persistence xmlns="http://java.sun.com/xml/ns/persistence"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
        version="2.0">

   <persistence-unit name="helloworld">

       <!-- The provider only needs to be set if you use several JPA providers
       <provider>org.hibernate.ejb.HibernatePersistence</provider>
        -->
        <!-- A managed datasource provided by the application server, in this
            case the microcontainer, see caveatemptor-beans.xml definition.
       
       <jta-data-source>java:/caveatemptorTestingDatasource</jta-data-source>
      -->
       <!-- This is required to be spec compliant, Hibernate however supports
            auto-detection even in JSE.
       <class>hello.Message</class>
        -->
	<!--   厂商专有属性(可选)   -->  
      <properties>
          <!-- Scan for annotated classes and Hibernate mapping XML files -->
          <property name="hibernate.archive.autodetection" value="class, hbm"/>

        
          <property name="hibernate.show_sql" value="true"/>
          <property name="hibernate.format_sql" value="true"/>
          <property name="use_sql_comments" value="true"/>
       

          <property name="hibernate.connection.driver_class"
                    value="com.mysql.jdbc.Driver"/>
          <property name="hibernate.connection.url"
                    value="jdbc:mysql://localhost/mydb"/>
          <property name="hibernate.connection.username"
                    value="root"/>
	  <property name="hibernate.connection.password" value="1234"/>
         <!-- hibernate的c3p0连接池配置（需要jar包：c3p0-0.9.0.4.jar）
	 <property name="hibernate.connection.provider_class" value="org.hibernate.connection.C3P0ConnectionProvider"/> 可以省略。会自动检测。
	--> 
          <property name="hibernate.c3p0.min_size"
                    value="5"/>
          <property name="hibernate.c3p0.max_size"
                    value="20"/>
         <!-- 获得连接的超时时间为3秒,如果超过这个时间,会抛出异常，单位毫秒 -->       
          <property name="hibernate.c3p0.timeout"
                    value="3000"/>
         <!--最大空闲时间,300秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->     
        <property name="c3p0.maxIdleTime" value="300"/>   
          <!-- 最大的PreparedStatement的数量 --> 
          <property name="hibernate.c3p0.max_statements"
                    value="50"/>
         <!-- 每隔3000秒检查连接池里的空闲连接 ，单位是秒。可以解决mysql8小时问题-->   
          <property name="hibernate.c3p0.idle_test_period"
                    value="3000"/>
<!-- 当连接池里面的连接用完的时候，C3P0一下获取的新的连接数 
        <property name="c3p0.acquire_increment" value="3"/>   
-->    
          <property name="hibernate.dialect"
                    value="org.hibernate.dialect.MySQL5InnoDBDialect"/>
<!--Automatically validates or exports schema DDL to the database when the SessionFactory is created. With create-drop, the database schema will be dropped when the SessionFactory is closed explicitly.     e.g. validate | update | create | create-drop
validate 加载hibernate时，验证创建数据库表结构
create 每次加载hibernate，重新创建数据库表结构，这就是导致数据库表数据丢失的原因。
create-drop 加载hibernate时创建，退出是删除表结构
update 加载hibernate自动更新数据库结构,以前的行不会被删除。
以上4个属性对同一配置文件下所用有的映射表都起作用
总结：
1.请慎重使用此参数，没必要就不要随便用。
2.如果发现数据库表丢失，请检查hibernate.hbm2ddl.auto的配置
-->
           <property name="hibernate.hbm2ddl.auto" value="update"/>
      </properties>
   </persistence-unit>

</persistence>

```
**JPA主要接口**
  * Persistence
  * EntityManagerFactory
  * EntityManager
  * Query
  * EntityTransaction
```

public class HelloWorld {

    public static void main(String[] args) {

        // Start EntityManagerFactory
        EntityManagerFactory emf =
                Persistence.createEntityManagerFactory("helloworld");  //参数为persistence-unit的名字

        // First unit of work
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();

        Message message = new Message("Hello World with JPA");
        em.persist(message);

        tx.commit();
        em.close();

        // Second unit of work
        EntityManager newEm = emf.createEntityManager();
        EntityTransaction newTx = newEm.getTransaction();
        newTx.begin();

        List messages =
            newEm.createQuery("select m from Message m order by m.text asc").getResultList();

        System.out.println( messages.size() + " message(s) found:" );

        for (Object m : messages) {
            Message loadedMsg = (Message) m;
            System.out.println(loadedMsg.getText());
        }

        newTx.commit();
        newEm.close();

        // Shutting down the application
        emf.close();
    }
}


```
##  实现领域模型 
1、实现Serializable接口，
2、实现无参构造器
3、setter，getter
4、业务方法
```

public class Category implements Serializable {

    private Long id = null;
    private int version = 0;

    private String name;
    private List<Category> childCategories = new ArrayList<Category>(); // A bag with SQL ORDER BY
    private Category parentCategory;

    private List<Item> items = new ArrayList<Item>();
    private Set<CategorizedItem> categorizedItems = new HashSet<CategorizedItem>();
    private Set<CategorizedItemComponent> categorizedItemComponents = new HashSet<CategorizedItemComponent>();
    private Map<Item,User> itemsAndUser = new HashMap<Item,User>();

    private Date created = new Date();

    /**
     * No-arg constructor for JavaBean tools
     */
    public Category() {}

    /**
     * Full constructor
     */
    public Category(String name,
                    List<Category> childCategories,
                    Category parentCategory,
                    List<Item> items,
                    Set<CategorizedItem> categorizedItems,
                    Set<CategorizedItemComponent> categorizedItemComponents,
                    Map<Item, User> itemsAndUser) {
        this.name = name;
        this.childCategories = childCategories;
        this.parentCategory = parentCategory;
        this.items = items;
        this.categorizedItems = categorizedItems;
        this.categorizedItemComponents = categorizedItemComponents;
        this.itemsAndUser = itemsAndUser;
    }

    /**
     * Simple constructors
     */
    public Category(String name) {
        this.name = name;
    }

    public Category(String name, Category parentCategory) {
        this.name = name;
        this.parentCategory = parentCategory;
    }

    // ********************** Accessor Methods ********************** //

    public Long getId() { return id; }
    public int getVersion() { return version; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public List getChildCategories() { return childCategories; }
    public void addChildCategory(Category childCategory) {
        if (childCategory == null) throw new IllegalArgumentException("Null child category!");
        if (childCategory.getParentCategory() != null)
            childCategory.getParentCategory().getChildCategories().remove(childCategory);
        childCategory.setParentCategory(parentCategory);
        childCategories.add(childCategory);
    }
    public void removeChildCategory(Category childCategory) {
        if (childCategory == null) throw new IllegalArgumentException("Null child category!");
        childCategory.setParentCategory(null);
        childCategories.remove(childCategory);
    }

    public Category getParentCategory() { return parentCategory; }
    private void setParentCategory(Category parentCategory) { this.parentCategory = parentCategory; }

    // Regular many-to-many
    public List<Item> getItems() { return items; }
    public void addItem(Item item) {
        if (item == null) throw new IllegalArgumentException("Null item!");
        items.add(item);
        item.getCategories().add(this);
    }
    public void removeItem(Item item) {
        if (item == null) throw new IllegalArgumentException("Null item!");
        items.remove(item);
        item.getCategories().remove(this);
    }

    // Many-to-many with additional columns on join table, intermediate entity class
    // To create a link, instantiate a CategorizedItem with the right constructor
    // To remove a link, use getCategorizedItems().remove()
    public Set<CategorizedItem> getCategorizedItems() { return categorizedItems; }

    // Many-to-many with additional columns on join table, intermediate component class
    public Set<CategorizedItemComponent> getCategorizedItemComponents() { return categorizedItemComponents; }

    // Many-to-many with additional columns on join table, ternary hash map representation
    public Map<Item, User> getItemsAndUser() { return itemsAndUser; }

    public Date getCreated() { return created; }

```

###  Hibernate XML方式 
```

<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping SYSTEM
    "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd"
[<!ENTITY % globals SYSTEM "classpath://auction/persistence/globals.dtd">%globals;]>
<hibernate-mapping package="auction.model" default-access="field">
<class name="Category" table="CATEGORY">
	<!-- Common id property -->
	<id name="id" type="long" column="CATEGORY_ID">
		<generator class="&idgenerator;"/>
	</id>
	<!-- A versioned entity -->
	<version name="version" column="OBJ_VERSION"/>
	<!-- Name is limited to 255 characters.-->
	<property   name="name"
				type="string">
		<column name="CATEGORY_NAME"
				not-null="true"
				length="255"/>
	</property>

	<!-- Immutable property -->
	<property   name="created"
				column="CREATED"
				type="timestamp"
				update="false"
				not-null="true"/>

    <!-- The non-inverse side of the one-to-many/many-to-one association -->
    <many-to-one name="parentCategory"
                 class="Category"
                 foreign-key="FK_CATEGORY_PARENT_ID">
        <column name="PARENT_CATEGORY_ID"
                not-null="false"/>
    </many-to-one>

    <!-- The inverse side of the one-to-many/many-to-one association, we
         use a bag mapping, iteration order of this collection is by
         category name. -->
    <bag name="childCategories"
            cascade="save-update, merge"
            inverse="true"
            order-by="CATEGORY_NAME asc">
        <key column="PARENT_CATEGORY_ID"/>
        <one-to-many class="Category"/>
    </bag>

    <!-- The pure many-to-many association to Item, a true persistent list with index -->
    <list name="items" table="CATEGORY_ITEM">
        <key column="CATEGORY_ID" foreign-key="FK_CATEGORY_ITEM_CATEGORY_ID"/>
        <list-index column="DISPLAY_POSITION"/>
        <many-to-many class="Item" column="ITEM_ID" foreign-key="FK_CATEGORY_ITEM_ITEM_ID">

            <!-- Dynamic collection filter -->
            <filter name="limitItemsByUserRank"
                    condition=":currentUserRank >=
                               (select u.RANK from USERS u where u.USER_ID = SELLER_ID)"/>

        </many-to-many>
    </list>

    <set    name="categorizedItems"
            cascade="all, delete-orphan"
            inverse="true"
            fetch="subselect">
        <key column="CATEGORY_ID" not-null="true"/>
        <one-to-many class="CategorizedItem"/>
    </set>

    <set name="categorizedItemComponents" table="CATEGORIZED_ITEM_COMPONENT">
        <key column="CATEGORY_ID" foreign-key="FK_CATEGORIZED_ITEM_COMPONENT_CATEGORY_ID"/>
        <composite-element class="CategorizedItemComponent">
            <parent name="category"/>

            <many-to-one name="item"
                         column="ITEM_ID"
                         class="Item"
                         not-null="true"
                         update="false"
                         foreign-key="FK_CATEGORIZED_ITEM_COMPONENT_ITEM_ID"/>

            <property   name="username"
                        type="string"
                        column="ADDED_BY_USER"
                        not-null="true"
                        update="false"/>

            <property   name="dateAdded"
                        column="ADDED_ON"
                        type="timestamp"
                        not-null="true"
                        update="false"/>

        </composite-element>
     </set>

    <!--
        This is an alternative mapping for the many-to-many association
        to Item. We use a Map, with the Item objects as keys, and the User
        who added the Item to the Category as a value. This makes any
        intermediate class unnecessary, but limits what you can represent
        on the many-to-many join table. The date of the addition is no
        longer present.
    -->
    <map name="itemsAndUser" table="CATEGORY_ITEMS_BY_USER">
        <key column="CATEGORY_ID" foreign-key="FK_CATEGORY_ITEMS_BY_USER_CATEGORY_ID"/>
        <map-key-many-to-many column="ITEM_ID" class="Item" foreign-key="FK_CATEGORY_ITEMS_BY_USER_ITEM_ID"/>
        <many-to-many column="ADDED_BY_USER_ID" class="User" foreign-key="FK_CATEGORY_ITEMS_BY_USER_USER_ID"/>
    </map>

</class>

<!--
    There can never be two categories with the same name at
    the same "level", that is, they can't have the same parent
    category. This is protected with a unique constraint.
-->
<database-object>
    <create>
        alter table CATEGORY add constraint UNIQUE_SIBLINGS
            unique (CATEGORY_NAME, PARENT_CATEGORY_ID)
    </create>
    <drop>
        alter table CATEGORY drop constraint UNIQUE_SIBLINGS
    </drop>
</database-object>

</hibernate-mapping>

```
###  JPA注解方式 
```

@Entity
@Table(
   name = "CATEGORY",
   uniqueConstraints =
    @UniqueConstraint(columnNames = {"CATEGORY_NAME", "PARENT_CATEGORY_ID"})
    // If you want a constraint name in the schema, use <database-object> in XML instead
)
public class Category implements Serializable, Comparable {

    @Id @GeneratedValue
    @Column(name = "CATEGORY_ID")
    private Long id = null;

    @Version
    @Column(name = "OBJ_VERSION")
    private int version = 0;

    @Column(name = "CATEGORY_NAME", length = 255, nullable = false)
    private String name;

    @OneToMany(mappedBy = "parentCategory", cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @org.hibernate.annotations.OrderBy(clause = "CATEGORY_NAME asc")
    private List<Category> childCategories = new ArrayList<Category>(); // A bag with SQL ORDER BY

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "PARENT_CATEGORY_ID", nullable = true)
    @org.hibernate.annotations.ForeignKey(name = "FK_CATEGORY_PARENT_ID")
    private Category parentCategory;

    @ManyToMany
    @JoinTable(
        name = "CATEGORY_ITEM",
        joinColumns =        @JoinColumn(name = "CATEGORY_ID"),
        inverseJoinColumns = @JoinColumn(name = "ITEM_ID")
    )
    @org.hibernate.annotations.IndexColumn(name = "DISPLAY_POSITION")
    @org.hibernate.annotations.ForeignKey(
        name        = "FK_CATEGORY_ITEM_CATEGORY_ID",
        inverseName = "FK_CATEGORY_ITEM_ITEM_ID"
    )
    @org.hibernate.annotations.Filter(
        name = "limitItemsByUserRank",
        condition = ":currentUserRank >= (select u.RANK from USERS u where u.USER_ID = SELLER_ID)"
    )
    private List<Item> items = new ArrayList<Item>();

    @OneToMany(cascade = CascadeType.ALL, mappedBy = "category")
    @org.hibernate.annotations.Cascade(value = org.hibernate.annotations.CascadeType.DELETE_ORPHAN)
    @org.hibernate.annotations.Fetch(org.hibernate.annotations.FetchMode.SUBSELECT)
    private Set<CategorizedItem> categorizedItems = new HashSet<CategorizedItem>();

    @org.hibernate.annotations.CollectionOfElements
    @JoinTable(
        name = "CATEGORIZED_ITEM_COMPONENTS",
        joinColumns = @JoinColumn(name = "CATEGORY_ID")
    )
    @org.hibernate.annotations.ForeignKey(name = "FK_CATEGORIZED_ITEM_COMPONENT_CATEGORY_ID")
    private Set<CategorizedItemComponent> categorizedItemComponents = new HashSet<CategorizedItemComponent>();

    @ManyToMany
    @org.hibernate.annotations.MapKeyManyToMany(
        joinColumns = @JoinColumn(name = "ITEM_ID")
    )
    @JoinTable(
        name = "CATEGORY_ITEMS_BY_USER",
        joinColumns = @JoinColumn(name = "CATEGORY_ID"),
        inverseJoinColumns = @JoinColumn(name = "USER_ID")
    )
    @org.hibernate.annotations.ForeignKey(
        name = "FK_CATEGORY_ITEMS_BY_USER_CATEGORY_ID",
        inverseName = "FK_CATEGORY_ITEMS_BY_USER_USER_ID"
        // TODO: Can't declare the foreign key constraint name for ITEM_ID...
    )
    private Map<Item,User> itemsAndUser = new HashMap<Item,User>();

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name="CREATED", nullable = false, updatable = false)
    private Date created = new Date();

    /**
     * No-arg constructor for JavaBean tools
     */
    public Category() {}
。。。。。。。同上

```
