title: spring3数据库访问与事务 

#  Spring3数据库访问与事务 
**主要步骤：配置数据源bean->用所配置的数据源bean配置数据管理器工厂(hibernate为SessionFactor,JPA为EntityManagerFactory)->用所配置的管理器工厂bean配置事务管理器transactionManager
-->定义事务与事务属性（在XML中定义事务通知和切面或者基于注解配置事务@Transactional）.
注意XML中的几个配置： **
```

<context:component-scan base-package="com.habuma.spitter.persistence" />
<bean class=
 "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"
/> 
<bean class=
  "org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>
<tx:annotation-driven tansaction-manager="your txManger bean id"/>

```

#  数据访问 
主要内容：
  * 定义Spring对数据访问的支持；
  * 配置数据库资源
  * 使用Spring的JDBC模板
  * Spring与Hibernate和JPA集成使用
Spring的目标之一就是能够使开发人员能够遵守“面向接口编程的原则”
DAO(Data access Object)
![](/data/dokuwiki/spring/pasted/20150805-105345.png)
Spring提供了一套数据访问异常体系。它与具体的持久化实现无关。
##  数据访问模板化 
Spring将数据访问过程中固定的和可变动的部分明确划分为两个不同的类：
**模板（template)和回调（callback)**
模板管理过程中固定的部分，回调处理自定义的数据访问代码。
![](/data/dokuwiki/spring/pasted/20150805-105755.png)
![](/data/dokuwiki/spring/pasted/20150805-105808.png)
使用DAO支持类：
![](/data/dokuwiki/spring/pasted/20150805-105833.png)
![](/data/dokuwiki/spring/pasted/20150805-105843.png)
##  配置数据源 
三种方式：
![](/data/dokuwiki/spring/pasted/20150805-105911.png)
###  使用JNDI数据源： 
![](/data/dokuwiki/spring/pasted/20150805-105938.png)
![](/data/dokuwiki/spring/pasted/20150805-105950.png)![](/data/dokuwiki/spring/pasted/20150805-105956.png)
###  使用数据源连接池 
以apache DBCP为例：
![](/data/dokuwiki/spring/pasted/20150805-110035.png)
###  基于JDBC驱动的数据源 
![](/data/dokuwiki/spring/pasted/20150805-110123.png)
##  使用JDBC模板 
Spring的JDBC框架承担了资源管理与异常处理的工作，从而简化了JDBC代码
` SimpleJdbcTemplate: `
```

  <bean id="jdbcTemplate"
     class="org.springframework.jdbc.core.simple.SimpleJdbcTemplate">
     <constructor-arg ref="dataSource" />
  </bean>
  <bean id="spitterDao" 
      class="com.habuma.spitter.persistence.SimpleJdbcTemplateSpitterDao">
    <property name="jdbcTemplate" ref="jdbcTemplate" />
  </bean>

```
```

  private static final String SQL_INSERT_SPITTER = "insert into spitter (username, password, fullname, email, update_by_email) values (?, ?, ?, ?, ?)";

  private static final String SQL_UPDATE_SPITTER = "update spitter set username = ?, password = ?, fullname = ?, set email=?"
          + "where id = ?";
  public Spitter getSpitterById(long id) {
    return jdbcTemplate.queryForObject(//<co id="co_query"/>
            SQL_SELECT_SPITTER_BY_ID,
        new ParameterizedRowMapper<Spitter>() {
          public Spitter mapRow(ResultSet rs, int rowNum) 
              throws SQLException {
            Spitter spitter = new Spitter();//<co id="co_map"/>
            spitter.setId(rs.getLong(1));
            spitter.setUsername(rs.getString(2));
            spitter.setPassword(rs.getString(3));
            spitter.setFullName(rs.getString(4));
            return spitter;
          }
        }, 
        id //<co id="co_bind"/>
        );
  }
  //<end id="java_getSpitterById" />

  //<start id="java_addSpitter" /> 
  public void addSpitter(Spitter spitter) {
    jdbcTemplate.update(SQL_INSERT_SPITTER,//<co id="co_updateSpitter"/>
            spitter.getUsername(), 
            spitter.getPassword(),
            spitter.getFullName(),
            spitter.getEmail(),
            spitter.isUpdateByEmail());
    spitter.setId(queryForIdentity());
  }
  //<end id="java_addSpitter" />

  public void saveSpitter(Spitter spitter) {
    jdbcTemplate.update(SQL_UPDATE_SPITTER,
            spitter.getUsername(), 
            spitter.getPassword(),
            spitter.getFullName(), 
            spitter.getEmail(),
            spitter.getId());
  }

```
使用` SimpleJdbcDaoSupport `
```

  public void addSpitter(Spitter spitter) {
    getSimpleJdbcTemplate().update(
        SQL_INSERT_SPITTER,
        new Object[] { spitter.getUsername(), spitter.getPassword(),
            spitter.getFullName() });
    spitter.setId(queryForIdentity());
  }

```

` public class JdbcSpitterDao extends SimpleJdbcDaoSupport implements SpitterDao ` 
![](/data/dokuwiki/spring/pasted/20150805-111247.png)

##  在Spring中集成Hibernate 
![](/data/dokuwiki/spring/pasted/20150805-111405.png)
![](/data/dokuwiki/spring/pasted/20150805-111413.png)
**配置Hibernate的SessionFactory:**
![](/data/dokuwiki/spring/pasted/20150805-111457.png)
![](/data/dokuwiki/spring/pasted/20150805-111512.png)
![](/data/dokuwiki/spring/pasted/20150805-111551.png)
具体配置:
xmlns:context="http://www.springframework.org/schema/context"
```

 <context:component-scan  可以检索@Repository注解，将其声明为bean
     base-package="com.habuma.spitter.persistence" />

<bean class=  使用Spring的异常体系
  "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

 <bean id="sessionFactory"
  class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  <property name="packagesToScan" 
      value="com.habuma.spitter.domain" />  
  <property name="hibernateProperties">
   <props>
    <prop key="dialect">org.hibernate.dialect.HSQLDialect</prop>
   </props>
  </property>
 </bean>

```
```

@Repository
public class HibernateSpitterDao implements SpitterDao {
  private SessionFactory sessionFactory;

  @Autowired
  public HibernateSpitterDao(SessionFactory sessionFactory) {//<co id="co_injectSessionFactory"/>
    this.sessionFactory = sessionFactory;
  }
   /** 
     * 进行持久化的方法需要使用@Transactional进行事务管理 
     */  
    @Transactional(readOnly = false, rollbackFor = RuntimeException.class)  
    public void addSpitter(Spitter spitter){  
        this.currentSession().save(spitter);  
    }  

```
##  Spring与JPA 
![](/data/dokuwiki/spring/pasted/20150805-112352.png)
###  配置EntityManagerFactory 
两种类型的实体管理器：
![](/data/dokuwiki/spring/pasted/20150805-112455.png)
![](/data/dokuwiki/spring/pasted/20150805-112518.png)
**使用应用程序管理类型的JPA:**
![](/data/dokuwiki/spring/pasted/20150805-112619.png)
META-INF/persistence.xml：
![](/data/dokuwiki/spring/pasted/20150805-112753.png)
![](/data/dokuwiki/spring/pasted/20150805-112803.png)
**使用容器管理类型的JPA:**
![](/data/dokuwiki/spring/pasted/20150805-112847.png)
![](/data/dokuwiki/spring/pasted/20150805-112923.png)
![](/data/dokuwiki/spring/pasted/20150805-112944.png)
**从JNDI获取实体管理器工厂：**
![](/data/dokuwiki/spring/pasted/20150805-113044.png)
###  编写基于JPA的POJO: 
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

  @PersistenceContext
  private EntityManager em; //<co id="co_injectEM"/>

```
` @PersistenceContext注解表明需要将一个EntityManager实例注入到em上，为了在Spring中实现EntityManager注入，我们需要在Spring应用上下文中配置一个 `
```

<bean class=
  "org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>

```
` @Transactional注解表明这个DAO中的持久化方法是在事务上下文中执行的。 `
**所以一般的配置文件形如：**
```

<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
 xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
  
  <context:component-scan base-package="com.habuma.spitter.persistence" />
  
  <bean class=
 "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"
/>

<bean class=
  "org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>
  
  <!--<start id="bean_jpa_emf"/>--> 
  <bean id="emf" class=
      "org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
  </bean>
  <bean id="jpaVendorAdapter" 
      class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
    <property name="database" value="HSQL" />
    <property name="showSql" value="true"/>
    <property name="generateDdl" value="false"/>
    <property name="databasePlatform" 
              value="org.hibernate.dialect.HSQLDialect" />
  </bean>


  <bean id="transactionManager"
      class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="emf" />
  </bean>

</beans>

```
#  Spring事务管理 
在软件开发领域，全有全无的操作被称为事务（transaction）.
<wrap em>事务的4个特性：ACID:
原子性Atomic, 一致性Consistent, 隔离性Isolated, 持久性Durable</wrap>
![](/data/dokuwiki/spring/pasted/20150806-023546.png)
**Spring基于AOP的声明式事务编程**。
Spring提供的事务管理器分类：
![](/data/dokuwiki/spring/pasted/20150806-024359.png)
![](/data/dokuwiki/spring/pasted/20150806-024434.png)
##  JDBC事务 
![](/data/dokuwiki/spring/pasted/20150806-024516.png)
##  Hibernate事务 
![](/data/dokuwiki/spring/pasted/20150806-024556.png)
![](/data/dokuwiki/spring/pasted/20150806-024616.png)
##  JPA事务 
![](/data/dokuwiki/spring/pasted/20150806-024708.png)
##  声明式事务 
###  定义事务属性： 
事务属性常量都` 是org.springframework.transaction.TransactionDefinition `接口中的常量。
![](/data/dokuwiki/spring/pasted/20150806-024919.png)
传播规则：
![](/data/dokuwiki/spring/pasted/20150806-024939.png)
隔离级别：
![](/data/dokuwiki/spring/pasted/20150806-025030.png)
![](/data/dokuwiki/spring/pasted/20150806-025040.png)
![](/data/dokuwiki/spring/pasted/20150806-025058.png)

###  在XML中定义事务 
![](/data/dokuwiki/spring/pasted/20150806-025428.png)
![](/data/dokuwiki/spring/pasted/20150806-025654.png)
![](/data/dokuwiki/spring/pasted/20150806-025832.png)
###  定义注解驱动的事务 
xmlns:tx="http://www.springframework.org/schema/tx"
` <tx:annotation-driven /> `查找所有被注解为` @Transactional `的bean,并为其添加事务通知（通过注解的value）:
如果事务管理器不是默认的名称 transactionManager则需要：
` <tx:annotation-driven  transaction-manager="txManager"/> `
```

@Service("spitterService")
@Transactional(propagation=Propagation.REQUIRED)
public class SpitterServiceImpl implements SpitterService {

  public void saveSpittle(Spittle spittle) {
    spittle.setWhen(new Date());
    spitterDao.saveSpittle(spittle);
  }

  @Transactional(propagation=Propagation.SUPPORTS, readOnly=true)
  public List<Spittle> getRecentSpittles(int count) {
    List<Spittle> recentSpittles = 
        spitterDao.getRecentSpittle();
    
    reverse(recentSpittles);
    
    return recentSpittles.subList(0, 
            min(49, recentSpittles.size()));
  }

```
![](/data/dokuwiki/spring/pasted/20150806-030424.png)
