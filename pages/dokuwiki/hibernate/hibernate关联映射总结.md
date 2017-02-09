title: hibernate关联映射总结 

#  Hibernate关联映射总结 
（1）ManyToOne（多对一）**单向**：**不产生中间表，但可以用@Joincolumn（name="  "）来指定生成外键的名字，外键在多的一方表中产生！**
（2）OneToMany（一对多）**单向**：**会产生中间表，此时可以用@onetoMany @Joincolumn（name=" "）避免产生中间表**，并且指定了外键的名字（别看@joincolumn在一中写着，但它存在在多的那个表中）
（3）OneToMany ,ManyToOne 双向（两个注解一起用的）：如果不在@OneToMany中加mappedy属性就会产生中间表，此时通常在@ManyToOne的注解下再添上注解@Joincolumn(name=" ")来指定外键的名字(说明：多的一方为关系维护端，关系维护端负责外键记录的更新，关系被维护端没有权利更新外键记录)！（@OneToMany(mappedBy="一对多中，多中一的属性")出现mapby为被维护端|||默认为延迟加载)

使用@JoinColumn标记是，需要注意一下几点： 
a、@JoinColumn与@Column标记一样，是用来注释表中的字段的，它的属性和@Column有很多相似之处，不在详述。 
b、但是它们的区别在于：@JoinColumn注释是保存表与表之间关系的字段 
c、如果不设置name，默认为name = 关联表的名称+"-"+关联表主键的字段名，在上面实例中，默认为“address_id” 
e、默认情况下，关联的实体的主键一般是用来做外键的，但如果此时不想用主键作为外键，则需要设置referencedColumnName属性.


##  一、一对多（@OneToMany） 
**1、` 单向 `一对多模型**
假设通过一个客户实体可以获得多个地址信息。
对于一对多的实体关系而言，表结构有两种设计策略，**分别是外键关联和表关联。**
(1) 映射策略---外键关联
在数据库中表customer和表结构address定义，如下：
```

create table customer (
  id int(20) not null auto_increment,
  name varchar(100),
  primary key(id)
)
 
create table address (
  id int(20) not null auto_increment,
  province varchar(50),
  city varchar(50),
  postcode varchar(50),
  detail varchar(50),
  customer_id int(20),
  primary key (id)
)

```
注意此时**外键定义在多的一方，也就是address表中。**
 此时，表customer映射为实体CustomerEO，代码如下：
```

@Entity
@Table(name="customer")
public class CustomerEO implements java.io.Serializable {
  @OneToMany(cascade={ CascadeType.ALL })
  @JoinColumn(name="customer_id")
  private Collection<AddressEO> addresses = new ArrayList<AddressEO>();
 ...
}

```
```

@Target({METHOD, FIELD}) @Retention(RUNTIME)
public @interface OneToMany {
  Class targetEntity() default void.class;
  CascadeType[] cascade() default {};
  FetchType fetch() default LAZY;
  String mappedBy() default "";
}

```
使用时要注意一下几点问题： 
a、targetEntity属性表示默认关联的实体类型。如果集合类中指定了具体类型了，不需要使用targetEntity.否则要指定targetEntity=AddressEO.class。 
b、` mappedBy属性用于标记当实体之间是双向时使用。 ` 

**(2) 映射策略---表关联** 
在上面address表中去掉customer_id字段，在增加一个表ref_customer_address，如下： 
--客户地址关系表
```

create table ref_customer_address (
  customer_id int(20) not null,
  address_id int(20) not null unique
)

```
此时表customer映射为CustomerEO实体，代码如下：
```

@Entity
@Table(name = "customer")
public class CustomerEO implements java.io.Serializable {
  ...
  @OneToMany(cascade = { CascadeType.ALL })
  @JoinTable(name="ref_customer_address",
           joinColumns={ @JoinColumn(name="customer_id",referencedColumnName="id")},
           inverseJoinColumns={@JoinColumn(name="address_id",referencedColumnName="id")})
  private Collection<AddressEO> addresses = new ArrayList<AddressEO>();
  ...
}

```
表关联@JoinTable，定义如下：```


@Target({METHOD,FIELD}) 
public @interface JoinTable {
  String name() default "";
  String catalog() default "";
  String schema() default "";
  JoinColumn[] joinColumns() default {};
  JoinColumn[] inverseJoinColumns() default {};
  UniqueConstraint[] uniqueConstraints default {};
}


```
其中：
a、该标记和@Table相似，用于标注用于关联的表。
b、name属性为连接两张表的表名。默认的表名为：“表名1”+“-”+“表名2”，上面例子默认的表名为customer_address。
c、joinColumns属性表示，在保存关系中的表中，所保存关联的外键字段。
d、inverseJoinColumns属性与joinColumns属性类似，不过它保存的是保存关系的另一个外键字段。

(3) 默认关联
在数据库底层为两张表添加约束，如下：
```

create table customer_address (
  customer_id int(20) not null,
  address_id int(20) not null unique
)
alter table customer_address add constraint fk_ref_customer foreign key (customer_id) references customer (id);
alter table customer_address add constraint fk_ref_address foreign key (address_id) references address (id);

```
这样，在CustomerEO中只需要在标注@OneToMany即可！

##  二、多对一@ManyToOne 

1、` 单向 `多对一模型。
(1) 外键关联
配置AddressEO实体如下：
```

@Entity
@Table(name="address")
public class AddressEO implements java.io.Serializable {
     
  @ManyToOne(cascade = { CascadeType.ALL })
  @JoinColumn(name="customer_id")
  private CustomerEO customer;
     
  // ...
}

```
@ManyToOne定义如下：
```

@Target({METHOD,FIELD}) @Retention(RUNTIME)
public @interface ManyToOne {
  Class targetEntity() default void.class;
  CascadeType[] cascade() default {};
  FetchType fatch() default EAGER;
  boolean optional() default true;
}

```

(2) 默认关联
数据库脚本定义的相关字段的约束，创建外键后，直接使用@ManyToOne

##  三、高级一对多和多对一映射 
即双向关联模型，确定了双向关联后，多的一方AddressEO不变使用@ManyToOne,而CustomerEO实体修改为：
```

@Entity
@Table(name="customer")
public class CustomerEO {
     
  @OneToMany(mappedBy="customer")
  private Collection<AddressEO> addresses = new ArrayList<AddressEO>();
     
  // ...
}

```
其中，@OneToMany标记中的mappedBy属性的值为AddressEO实体中所引用的CustomerEO实体的属性名。

四、多对多（@ManyToMany）
和一对多类型，不在赘述。@ManyToMany标记的定义如下：
```

@Target({METHOD, FIELD}) @Retention(RUNTIME)
public @interface ManyToMany {
  Class targetEntity() default void.class;
  CascadeType[] cascade() default {};
  FetchType fecth() default LAZY;
  String mappedBy() default "";
}

```

五、最后，谈谈关于集合类的选择
在映射关系中可以使用的集合类有Collection、Set、List和Map，下面看下如何选择。
1、定义时使用接口，初始化使用具体的类。
如Collection可以初始化为ArrayList或HashSet；
Set可以初始化为HashSet；
List可以初始化为ArrayList；
Map可以初始化为HashMap.
2、集合类的选择
Collection类是Set和List的父类，在未确定使用Set或List时可使用；
Set集合中对象不能重复，并且是无序的;
List集合中的对象可以有重复，并且可以有排序；
Map集合是带有key和value值的集合。

##  一对多双向关联 
例如order与orderitem：
在order中：
```

/*  
     * @OneToMany: 指明Order 与OrderItem关联关系为一对多关系 
     *  
     * mappedBy: 定义类之间的双向关系。如果类之间是单向关系，不需要提供定义，如果类和类之间形成双向关系，我们就需要使用这个属性进行定义， 
     * 否则可能引起数据一致性的问题。 
     *  
     * cascade: CascadeType[]类型。该属性定义类和类之间的级联关系。定义的级联关系将被容器视为对当前类对象及其关联类对象采取相同的操作， 
     * 而且这种关系是递归调用的。举个例子：Order 和OrderItem有级联关系，那么删除Order 时将同时删除它所对应的OrderItem对象。 
     * 而如果OrderItem还和其他的对象之间有级联关系，那么这样的操作会一直递归执行下去。cascade的值只能从CascadeType.PERSIST（级联新建）、 
     * CascadeType.REMOVE（级联删除）、CascadeType.REFRESH（级联刷新）、CascadeType.MERGE（级联更新）中选择一个或多个。 
     * 还有一个选择是使用CascadeType.ALL，表示选择全部四项。 
     *  
     * fatch: 可选择项包括：FetchType.EAGER 和FetchType.LAZY。前者表示关系类(本例是OrderItem类)在主类(本例是Order类)加载的时候 
     * 同时加载;后者表示关系类在被访问时才加载,默认值是FetchType. LAZY。 
     *  
     */  
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)  
    @OrderBy(value = "id ASC")//注释指明加载OrderItem时按id的升序排序  
    public Set<OrderItem> getOrderItems() {  
        return orderItems;  
    }  
  
    public void setOrderItems(Set<OrderItem> orderItems) {  
        this.orderItems = orderItems;  
    }  
      
    //@Temporal注释用来指定java.util.Date 或java.util.Calendar 属性与数据库类型date,time 或timestamp 中的那一种类型进行映射  
    @Temporal(value = TemporalType.TIMESTAMP)  
    public Date getCreatedate() {  
        return createdate;  
    }

```  
在orderItem中：
```

/* 
     * @ManyToOne指明OrderItem和Order之间为多对一关系，多个OrderItem实例关联的都是同一个Order对象。 
     * 其中的属性和@OneToMany基本一样，但@ManyToOne注释的fetch属性默认值是FetchType.EAGER。 
     *  
     * optional 属性是定义该关联类对是否必须存在，值为false时，关联类双方都必须存在，如果关系被维护端不存在，查询的结果为null。 
     * 值为true 时, 关系被维护端可以不存在，查询的结果仍然会返回关系维护端，在关系维护端中指向关系被维护端的属性为null。 
     * optional 属性的默认值是true。举个例：某项订单(Order)中没有订单项(OrderItem)，如果optional 属性设置为false， 
     * 获取该项订单（Order）时，得到的结果为null，如果optional 属性设置为true，仍然可以获取该项订单，但订单中指向订单项的属性为null。 
     * 实际上在解释Order 与OrderItem的关系成SQL时，optional 属性指定了他们的联接关系optional=false联接关系为inner join,  
     * optional=true联接关系为left join。 
     *  
     * @JoinColumn:指明了被维护端（OrderItem）的外键字段为order_id，它和维护端的主键(orderid)连接,unique= true 指明order_id列的值不可重复。 
     */  
    @ManyToOne(cascade = CascadeType.REFRESH, optional = false)  
    @JoinColumn(name = "order_id")  
    public Order getOrder() {  
        return order;  
    }  
  
    public void setOrder(Order order) {  
        this.order = order;  
    } 

``` 
注意：在ManyToOne中的 @JoinColumn(name = "order_id")，这里的name = "order_id"是orderitem的外键，与order表的主键关联，如果OrderItem类当中有个属性叫oder_id,那么就会报：
should be mapped with insert="false" updatable=false
这是要在 @JoinColumn(name = "order_id",referencedColumnName="orderid")加上insertable=false updatable=false以避免字段重复映射


参考：http://blog.csdn.net/conjimmy/article/details/46139081
http://my.oschina.net/liangbo/blog/92301