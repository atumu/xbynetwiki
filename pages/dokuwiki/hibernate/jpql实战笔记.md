title: jpql实战笔记 

#  JPQL实战笔记 
##  JPQL与SpringData JPA使用的问题 
` 注意：使用native sql 不能够享受Spring Data JPA的Pageable与Sortable接口。 `
此次采用in与子查询 ` in(select t.id.newsId from TNewsSortRelEntity t where t.id.dictNewsSort=?1) ` 
```

  /**
     * 根据新闻类别关联分页查询
     * @param dictNewsSort
     * @return
     */
    @Query(value = "select m from TNewsEntity m where m.newsId in(select t.id.newsId from TNewsSortRelEntity t where t.id.dictNewsSort=?1) and m.isPublish = 'Y' order by m.publishDate desc")
// 这是错误的   @Query(value = "select m from TNewsEntity m join TNewsSortRelEntity t  where m.newsId=t.id.newsId and t.id.dictNewsSort=?1 and m.isPublish = 'Y' order by m.publishDate desc")
    public Page<TNewsEntity> findPageByDictNewsSort(String dictNewsSortPage,Pageable pageable);

```
```

@Entity
@Table(name = "T_NEWS")
public class TNewsEntity extends CommonEntity {
    private static final long serialVersionUID = 1L;

    public static final String IS_PUBLISH_YES = "Y";

    @Id
    @GeneratedValue(generator = "paymentableGenerator")
    @GenericGenerator(name = "paymentableGenerator", strategy = "uuid")
    @Column(name = "NEWS_ID")
    private String newsId;

    @JSONField(serialize = false)
    @OneToMany(mappedBy = "TNewsEntity", fetch=FetchType.EAGER)
    private List<TNewsSortRelEntity> newsSortList;
 
    @Transient
    private List<TNewsImgEntity> newsImgs;//新闻图片列表

    @Transient
    private String relArtistIds;// 逗号间隔的与新闻关联的艺术家id字符串
    @Transient
    private String relArtistNames;// 逗号间隔的与新闻关联的艺术家姓名字符串
   
    @Transient
    private String relDictSortCodes;// 逗号间隔的与新闻类型code字符串

```
```

@Entity
@Table(name="T_NEWS_SORT_REL")
public class TNewsSortRelEntity extends com.css.sdc.framework.entity.BaseEntity implements Serializable {
	private static final long serialVersionUID = 1L;

	@EmbeddedId
	private TNewsSortRelEntityPK id;
	
	@JSONField(serialize = false)
	@ManyToOne()
    @JoinColumn(name="NEWS_ID",insertable = false, updatable = false)
    private TNewsEntity TNewsEntity;

```
```

@Embeddable
public class TNewsSortRelEntityPK implements Serializable {
    
	private static final long serialVersionUID = 1L;

	@Column(name="NEWS_ID", insertable=false, updatable=false)
	private String newsId;

	@Column(name="DICT_NEWS_SORT")
	private String dictNewsSort;


```
##  JPQL语法与SQL的区别 
SQL:比如select * from student,teacher where ...
但是JPQL不能使用类似的语法，**比如select u from student u, teacher t where ...这样` 会报错 `。**
**在JPQL中要想达到类似效果有两种方式**
  * 1、使用关联查询Join。但是关联查询只有在两个实体有关联关系时才可以使用。否则报错。
  * 2、使用子查询


##  JPQL关联查询 
` 注意join关联查询只有在两个实体有关联关系时才可以使用。如果两个实体没有任何关联关系就会报错。 `一定在注意这一点。尤其在使用SpringData JPA的@Query注解的时候。
例如
```

public class A{
	
}
public class B{
	private C c
	....
}
public class C{
	
}

```
A与B不存在关联关系，所以不能采用Join. B与C存在关联关系。可以采用Join.

从一关联到多的查询和从多关联到一的查询来简单说说关联查询。
　　实体Team：球队。
　　实体Player：球员。
　　球队和球员是一对多的关系。
```

/**
 * 球队
 * @author Luxh
 */
@Entity
@Table(name="team")
public class Team{
 
    @Id
    @GeneratedValue
    private Long id;
     
    /**球队名称*/
    @Column(name="name",length=32)
    private String name;
     
    /**拥有的球员*/
    @OneToMany(mappedBy="team",cascade=CascadeType.ALL,fetch=FetchType.LAZY)
    private Set<Player> players = new HashSet<Player>();
 
    //以下省略了getter/setter方法 
 
    //......
}

```
```

/**
 * 球员
 * @author Luxh
 */
@Entity
@Table(name="player")
public class Player{
     
    @Id
    @GeneratedValue
    private Long id;
     
    /**球员姓名*/
    @Column(name="name")
    private String name;
     
    /**所属球队*/
    @ManyToOne(cascade={CascadeType.MERGE,CascadeType.REFRESH})
    @JoinColumn(name="team_id")
    private Team team;
     
    //以下省略了getter/setter方法
 
        //......
}

```
###  1、从One的一方关联到Many的一方： 
查找出球员所属的球队，可以使用以下语句：
```

SELECT DISTINCT t FROM Team t JOIN t.players p where p.name LIKE :name

```
　　或者使用以下语句：
```

SELECT DISTINCT t FROM Team t,IN(t.players) p WHERE p.name LIKE :name

```
　　上面两条语句是等价的，产生的SQL语句如下：
```

select
    distinct team0_.id as id0_,
    team0_.name as name0_
from
    team team0_
inner join
    player players1_
        on team0_.id=players1_.team_id
where
    players1_.name like ?

```
从SQL语句中可以看到team inner join 到player。inner`  join要求右边的表达式必须有返回值。 `
**不能使用以下语句：**
SELECT DISTINCT t FROM Team t  WHERE t.players.name LIKE :name
` 不能使用t.players.name这样的方式从集合中取值，要使用join或者in才行。 `
###  2、从Many的一方关联到One的一方： 
查找出某个球队下的所有球员，可以使用以下查询语句：
SELECT p FROM Player p JOIN p.team t WHERE t.id = :id
　　或者使用以下语句：
SELECT p FROM Player p, IN(p.team) t WHERE t.id = :id
　　这两条查询语句是等价的，产生的SQL语句如下：(产生了两条SQL)
```

Hibernate:
    select
        player0_.id as id1_,
        player0_.name as name1_,
        player0_.team_id as team3_1_
    from
        player player0_
    inner join
        team team1_
            on player0_.team_id=team1_.id
    where
        team1_.id=?
Hibernate:
    select
        team0_.id as id2_0_,
        team0_.name as name2_0_
    from
        team team0_
    where
        team0_.id=?

```
从Many关联到One的查询，还可以使用以下的查询语句：
```

SELECT p FROM Player p WHERE p.team.id = :id

```
　　这条语句产生的SQL如下：(产生了两条SQL)
```

Hibernate:
    select
        player0_.id as id1_,
        player0_.name as name1_,
        player0_.team_id as team3_1_
    from
        player player0_
    where
        player0_.team_id=?
Hibernate:
    select
        team0_.id as id0_0_,
        team0_.name as name0_0_
    from
        team team0

```
　　
以上从Many到One的关联查询都产生了两条SQL，还可以使用join fetch只产生一条SQL语句。查询语句如下：
```

SELECT p FROM Player p JOIN FETCH p.team t WHERE t.id = :id

```
　　这条查询语句产生的SQL如下：
```

Hibernate:
    select
        player0_.id as id1_0_,
        team1_.id as id2_1_,
        player0_.name as name1_0_,
        player0_.team_id as team3_1_0_,
        team1_.name as name2_1_
    from
        player player0_
    inner join
        team team1_
            on player0_.team_id=team1_.id
    where
        team1_.id=?

```
###  从Many的一方关联到Many的一方： 
```

@Entity
public class Exchange implements Serializable {
    @Id(name="EXCHANGE_ID")
    private long exchangeId;

    @Column
    private String exchangeShortName;
}
@Entity
public class Contract implements Serializable { 
        @Id
        private long contractId;

        @Column
        private String contractName;

        @ManyToOne
        @JoinColumn(name="EXCHANGE_ID")
        private Exchange exchange;

        @ManyToMany
        @JoinTable(name="CONTRACT_COMBO",
                        joinColumns = { @JoinColumn(name="CONTRACT_ID") },
                        inverseJoinColumns = {@JoinColumn(name="COMBO_ID")})
        private Set<Combo> combos;

        @Column(name = "ACTIVE_FLAG")
        private String activeFlag;
}

@Entity
public class Combo implements Serializable {

        @Id
        @Column(name="COMBO_ID")
        private Integer id;

        @ManyToMany
        @JoinTable(name="CONTRACT_COMBO",
                        joinColumns = { @JoinColumn(name="COMBO_ID") },
                        inverseJoinColumns = {@JoinColumn(name="CONTRACT_ID")})
        private Set<Contract> legs;

        @OneToMany(mappedBy = "combo")
        private Set<Trade> trades;    
}

@Entity
public class Trade implements Serializable {
        @Id
        @Column(name="TRADE_ID")
        private long tradeId;

        @Column(name="REFERENCE")
        private String reference;

        @ManyToOne
        @JoinColumn(name="COMBO_ID")
        private Combo combo;
}

```
```

select distinct t 
  from Trade t
  join t.combo c
  join c.legs l
  join l.exchange e
 where e.exchangeShortName = 'whatever'

```


##  JPQL多实体/多表连接查询 
```

public class Test implements Serializable {
    //无参数构造方法
    public Test() {
        super();
    }

    //编号
    private String CId;
    public String getCId(){
        return this.CId;
    }
    public void setCId(String CId){
        this.CId=CId;
   }
    //姓名
    private String CName;
    public String getCName(){
        return this.CName;
    }
    public void setCName(String CName){
        this.CName=CName;
    }

    //编号
    private String DId;
    public String getDId(){

        return this.DId;

    }
    public void setDId(String DId){

        this.DId=DId;
   }

    //姓名
    private String DName;
    public String getDName(){

        return this.DName;

    }
    public void setDName(String DName){

        this.DName=DName;

    }

    //含参数构造方法
    public Test(String cId,String cName,String dId,String dName){
        super();
        this.CId=cId;
        this.CName=cName;
        this.DId=dId;
        this.DName=dName;

    }

}

```
```

@Entity
@Table(name = "test1")
public class Test1 implements Serializable {
    @Id
    @GenericGenerator(name = "generator", strategy = "uuid.hex")
    @GeneratedValue(generator = "generator")
    @Column(name = "C_Id") 
    private String CId;

    @Column(name = "C_Name") 
    private String CName;

    @Column(name = "C_Pwd") 
    private String CPwd;
    @Column(name = "C_Sex") 
    private String CSex;

    @Column(name = "C_Ago") 
    private String CAgo;
    public String getCId(){
        return this.CId;
    }

    public void setCId(String CId){
        this.CId=CId;
    }

    public String getCName(){
        return this.CName;
    }

    public void setCName(String CName){
        this.CName=CName;
    }

    public String getCPwd(){
        return this.CPwd;
    }

    public void setCPwd(String CPwd){
        this.CPwd=CPwd;
    }

    public String getCSex(){
        return this.CSex;
    }

    public void setCSex(String CSex){
        this.CSex=CSex;
    }

    public String getCAgo(){
       return this.CAgo;
    }

    public void setCAgo(String CAgo){
        this.CAgo=CAgo;
    }

}

```
```

@Entity
@Table(name = "test2")
public class Test2 implements Serializable {
    @Id
    @GenericGenerator(name = "generator", strategy = "uuid.hex")
    @GeneratedValue(generator = "generator")
    @Column(name = "D_Id") 
    private String DId;
    @Column(name = "D_Name") 
    private String DName;
    @Column(name = "D_Pwd") 
    private String DPwd;
    @Column(name = "D_Age") 
    private Integer DAge;
    @Column(name = "D_Addr") 
    private String DAddr;
    @Column(name = "D_Sex") 
    private String DSex;

    public String getDId(){
        return this.DId;
    }
    public void setDId(String DId){
        this.DId=DId;
    }

    public String getDName(){
        return this.DName;
    }

    public void setDName(String DName){
        this.DName=DName;
    }

    public String getDPwd(){
        return this.DPwd;
    }

    public void setDPwd(String DPwd){
        this.DPwd=DPwd;
    }

    public Integer getDAge(){
        return this.DAge;
    }

    public void setDAge(Integer DAge){
        this.DAge=DAge;
    }

    public String getDAddr(){
        return this.DAddr;
    }

    public void setDAddr(String DAddr){
        this.DAddr=DAddr;
    }

    public String getDSex(){
        return this.DSex;
    }

    public void setDSex(String DSex){
        this.DSex=DSex;
    }
}

```
```

public class MyJunitTest {
    @org.junit.Test
    public void testDDL(){
        //插入数据
        HibernateTest.doInHibernateSession(new InHibernateSession(){
            @Override
            public Object doInHibernateSession(Session session) {
                Test1 test1 = new Test1();
                test1.setCAgo("CAgo");
                test1.setCName("CName");
                test1.setCPwd("CPwd");
                test1.setCSex("CSex");
                Test2 test2 = new Test2();
                test2.setDAddr("DAddr");
                test2.setDAge(123);
                test2.setDName("DName");
                test2.setDPwd("DPwd");
                test2.setDSex("DSex");
                session.save(test1);
                session.save(test2);
                return null;
            }
        });
        //查询
         List<Test> all = (List<Test>) HibernateTest.doInHibernateSession(new InHibernateSession(){
            public Object doInHibernateSession(Session session) {
                String sql="select new y.model.Test(c.CId,c.CName,d.DId,d.DName) from Test1 c,Test2 d where c.CId=d.DId";
                Query query = session.createQuery(sql);
                return query.list();
            }
        });

         for(Test test : all){
             System.out.println(test);
         }
    }


    public static class HibernateTest {
        public static Object doInHibernateSession(InHibernateSession doInHibernateSession){
            Session session = HibernateSessionFactory.getSession();
            Transaction transaction = session.beginTransaction();
            try{
                Object object =  doInHibernateSession.doInHibernateSession(session);
                transaction.commit();
                return object;
            }catch(RuntimeException e){
                try{
                    System.err.println("rollback");
                    transaction.rollback();
                }catch(RuntimeException ex){
                    System.err.println("回滚失败！");
                }
                throw e;
            }finally{
                //HibernateSessionFactory.closeSession();
            }
        }
    }

    public interface InHibernateSession {
        public Object doInHibernateSession(Session session);
    }
}

```
参考：http://www.cnblogs.com/luxh/archive/2012/06/02/2531750.html
http://macrabbit.iteye.com/blog/855384
http://stackoverflow.com/questions/3124087/jpa-many-to-many-query-help-needed
http://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.at-query
http://www.csdn123.com/html/blogs/20130428/7458.htm