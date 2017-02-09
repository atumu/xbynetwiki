title: hibernate映射持久化类 

#  Hibernate映射持久化类 
##  数据库主键 
![](/data/dokuwiki/hibernate/pasted/20150906-171342.png)![](/data/dokuwiki/hibernate/pasted/20150906-171359.png)
总结一下：
**标识符**
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

一般开发情况下选择 Hibernate方式选择native,uuid.hex. JPA方式选择AUTO
##  实现命名约定 
略。。。。
##  实体与值类型、同一性 
![](/data/dokuwiki/hibernate/pasted/20150906-173438.png)![](/data/dokuwiki/hibernate/pasted/20150906-173739.png)

##  映射基础属性 
```

    @Column(length = 255, nullable = false)
	private String street;

    @Column(length = 16, nullable = false)
	private String zipcode;

    @Column(length = 255, nullable = false)
	private String city;

```
##  映射组件或被嵌入类 
![](/data/dokuwiki/hibernate/pasted/20150906-171837.png)
这种关联关系被称为聚合，是**整体-部分**的强关联形式。部分的生命周期完全依赖于整体的生命周期。
所以这里应该吧Address映射为值类型。把User映射为实体。**组件Address没有独立的同一性**，是**值类型**，不需要设置主键。其持久化后会被嵌入到User实体的表中。
![](/data/dokuwiki/hibernate/pasted/20150906-172204.png)
```

/**值类型，可嵌入的非实体，被嵌入到实体*/
@Embeddable
public class Address implements Serializable {

    @Column(length = 255, nullable = false)
	private String street;

    @Column(length = 16, nullable = false)
	private String zipcode;

    @Column(length = 255, nullable = false)
	private String city;

	/**
	 * No-arg constructor for JavaBean tools
	 */
	public Address() {}
  。。。。

```
```

/**实体类，含有被嵌入的Address值类型。*/
@Entity(name="UserEntity")//设置实体名称以便Jpql或Hql查询
@Table(name = "USERS")//设置表名
//Serializable
public class User implements Serializable{
  //设置ID和主键生成策略
    @Id @GeneratedValue
    @Column(name = "USER_ID")
    private Long id = null;
  
  //被嵌入的Address值类型
    @Embedded
    private Address homeAddress;
    @Version
    @Column(name = "OBJ_VERSION")
    private int version = 0;

    @Column(name = "USERNAME", length = 16, nullable = false, unique = true)
    //@org.hibernate.annotations.Check( constraints = "regexp_like(USERNAME,'^[[:alpha:]]+$')" )
    private String username; // Unique and immutable

    @Column(name = "`PASSWORD`", length = 12, nullable = false)
    private String password;

    @Column(name = "RANK", nullable = false)
    private int ranking = 0;
。。。。。。。。

```
