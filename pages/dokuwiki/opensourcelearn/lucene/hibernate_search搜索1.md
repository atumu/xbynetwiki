title: hibernate_search搜索1 

#  Hibernate Search搜索入门 
Hibernate Search是在apache Lucene的基础上建立的主要用于Hibernate的持久化模型的全文检索工具。像Lucene这样的检索引擎能够给我们的项目在进行检索的时候带来非常高的效率，但是它们在基本对象的检索时会有一些问题，比如不能实现检索内容跟实体的转换，Hibernate Search正是在这样的情况下发展起来的，**基于对象的检索引擎**，能够很方便的将检索出来的内容转换为具体的实体对象。此外Hibernate Search能够根据需要进行同步或异步的索引更新。
**Hibernate Search的作用是对数据库中的数据进行检索的。它是hibernate对著名的全文检索系统Lucene的一个集成方案**，作用在于对数据表中某些内容庞大的字段（如声明为text的字段）建立全文索引，这样通过hibernate search就可以对这些字段进行全文检索后获得相应的POJO，**从而加快了对内容庞大字段进行模糊搜索的速度（sql语句中like匹配）。**
**Hibernate search可以无缝得整合Hibernate和Lucene、JPA，帮助你快速实现功能强大的全文检索。**
一个Lucene文档包含了文档ID和希望索引的域数据。通常的做法是` 使用实体的主键作为Lucene文档的ID。 `
由于Hibernate Search提供了更为简单的接口，但我们使用的是Hibernate作为持久化框架的时候，我们没有理由拒绝使用它。
` Hibernate和Hibernate Search将在使用JPA添加和更新实体时，自动在Lucene中索引他们。并在使用JPA搜索持久化的实体时从Lucene中搜索他们。 `
官网：http://hibernate.org/search/
```

<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-search-orm</artifactId>
			<version>4.5.3.Final</version>
		</dependency>
<!-- Optional: to use JPA 2.1 -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${org.hibernate.version}</version>
		</dependency>


```

##  使用Hibernate Search索引注解标注实体 
参考：http://hibernate.org/search/documentation/getting-started/
如果是直接在persistence.xml 文件里面加入：
```

<property name="hibernate.search.default.directory_provider" value="filesystem"/>
<property name="hibernate.search.default.indexBase" value="e:/index"/>

```
如果是在Spring中配置
```

<!-- JPA实体工厂配置 -->  
    <bean id="emf"  
        class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">  
        <property name="dataSource" ref="dataSource" />  
        <!-- 扫描实体路径 -->  
        <property name="packagesToScan" value="net.xby1993.springmvc.entity" />  
        <property name="jpaVendorAdapter">  
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">   
                <property name="database" value="MYSQL" />
    			<property name="showSql" value="true"/>
    			<property name="generateDdl" value="false"/>
    			<property name="databasePlatform" value="org.hibernate.dialect.MySQL5InnoDBDialect" />
            </bean>  
        </property> 
        <property name="jpaProperties">
			<props>
				<!-- 命名规则 My_NAME->MyName java驼峰变db下划线命名-->
				<prop key="hibernate.ejb.naming_strategy">org.hibernate.cfg.ImprovedNamingStrategy</prop>
				<prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate4.SpringSessionContext</prop>
                          
                           	<!--Hibernate Search 配置 -->
                         	 <prop key="hibernate.search.default.directory_provider">filesystem</prop> <!-- 指定索引存储方式-->
				<prop key="hibernate.search.default.indexBase">e:/index</prop> <!-- 指定索引存储目录-->
			</props>
		</property> 
        
    </bean>  

```
```

import org.hibernate.search.annotations.Field;

import javax.persistence.Basic;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "UserPrincipal")
public class User
{
    private long id;
    private String username;

    @Id
    @Column(name = "UserId")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public long getId()
    {
        return this.id;
    }

    public void setId(long id)
    {
        this.id = id;
    }

    @Field   //这是一个Hibernate Search注解，表明这个属性应该进行全文索引，并可用于搜索。
    public String getUsername()
    {
        return this.username;
    }

    public void setUsername(String username)
    {
        this.username = username;
    }
}


```
```

import org.hibernate.search.annotations.Boost;
import org.hibernate.search.annotations.DocumentId;
import org.hibernate.search.annotations.Field;
import org.hibernate.search.annotations.Indexed;
import org.hibernate.search.annotations.IndexedEmbedded;
@Entity
@Table(name = "Post")
@Indexed //表明该实体适用于全文索引。Hibernate Search将自动为该实体创建或更新文档。无论何时添加或者保存它
@Analyzer(impl = IKAnalyzer.class) //指定分词器实现
public class ForumPost
{
    private long id;
    private User user;
    private String title;
    private String body;
    private String keywords;

    @Id
    @DocumentId  //指定哪个属性为文档ID
    @Column(name = "PostId")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public long getId()
    {
        return this.id;
    }

    public void setId(long id)
    {
        this.id = id;
    }

    @ManyToOne(fetch = FetchType.EAGER, optional = false)
    @JoinColumn(name = "UserId")
    @IndexedEmbedded  //说明该属性本身就是一个含有索引字段的实体，应该作为文档的一部分进行索引。
    public User getUser()
    {
        return this.user;
    }

    public void setUser(User user)
    {
        this.user = user;
    }

    @Field(store=Store.NO)  /*可索引，但不存储*/ 
    public String getTitle()
    {
        return this.title;
    }

    public void setTitle(String title)
    {
        this.title = title;
    }

    @Lob
    @Field(index=Index.YES, analyze=Analyze.YES, store=Store.YES) 
  /**
@Field ：索引字段，该注解默认属性值为
store=Store.NO：是否将数据存储在索引中，经实验无论store=Store.NO还是store=Store.YES都不会影响最终的搜索。如果store=Store.NO值是通过数据库中获取，如果store=Store.YES值是直接从索引文档中获取。
index=Index.YES：是否索引
analyze=Analyze.YES：是否分词
标注了注解后的实体在保存和更新的时候，会自动生成或修改索引。
  */
    public String getBody()
    {
        return this.body;
    }

    public void setBody(String body)
    {
        this.body = body;
    }

    @Field(analyze=Analyze.NO,boost = @Boost(2.0F)) //索引域，通过boost得到一个额外的加分。
    public String getKeywords()
    {
        return this.keywords;
    }

    public void setKeywords(String keywords)
    {
        this.keywords = keywords;
    }
}


```
使用luke工具，查看索引数据
**Luke是一款显示Lucene索引数据、修改Lucene索引数据和进行模拟搜索的开源工具**
###  域映射选项(Field Mapping Options) 
我们已经知道@Field注解用来让某个域可以被全文搜索到。 实际上，在添加该注解后，Hibernate Search会将一些有意义的默认设置添加到相应的Lucene索引中。
这些被添加的默认设置通常都可以通过` @Field `的属性进行设置，下面对这些属性进行简要介绍：
  * analyze：告诉Lucene是否直接保存该域的数据或者将该域进行某种处理(Analysis, Parsing, Processing, etc)后再进行保存。可以设置成Analyze.NO或者Analyze.YES。
  * index：告诉Lucene当前域是否需要被索引，默认值是Index.YES。也许这个属性会显得有些奇怪，既然已经被@Field标注了，那为什么要设置是否需要被索引呢？其实，在一些高级查询中这种情况是存在的
  * indexNullAs：声明如何处理域值为null的情况。**默认情况下，null值会被忽略并且也不会被添加到索引中**。但是，我们也可以将null值索引成某些特殊的值。在“高级映射”中会进行介绍。
  * name：用来给该域在Lucene索引中赋予一个名字，默认使用的就是该域的名字进行设置。
  * norms：用来决定是否存储用在索引时提升(Index-time Boosting)的信息。默认设置为Norms.YES。在“高级映射”中会进行说明。
  * store：通常情况下，域会以一种优化后的形式被保存到索引中。这样做虽然可以提升搜索性能，但是在取得搜索结果时，该域的原始数据可能也会被丢失。**默认设置是Store.NO，也就意味着原始数据也许会被丢失。当设置成Store.YES或者Store.COMPRESS后，在获取搜索结果时可以直接从Lucene索引中取得，而不需要再访问一次数据库。**在“高级查询”中会进行说明。

###  映射数值类型的域 
Hibernate Search提供了一个注解` @NumericField `用来处理Integer，Long，Float和Double等数值类型：
```

@Column
@Field
@NumericField
private float price;

```

##  创建SpringDataJPA仓库 
```

public interface SearchableRepository<T>
{
    Page<SearchResult<T>> search(String query, Pageable pageable);
}

```
```

public interface ForumPostRepository extends CrudRepository<ForumPost, Long>,
        SearchableRepository<ForumPost>
{
}

```
```

import com.wrox.site.entities.ForumPost;
import org.apache.lucene.search.Query;
import org.hibernate.search.jpa.FullTextEntityManager;
import org.hibernate.search.jpa.FullTextQuery;
import org.hibernate.search.jpa.Search;
import org.hibernate.search.query.dsl.QueryBuilder;
import org.springframework.beans.FatalBeanException;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.Pageable;
import org.springframework.orm.jpa.EntityManagerProxy;
import javax.annotation.PostConstruct;
public class ForumPostRepositoryImpl implements SearchableRepository<ForumPost>
{
    @PersistenceContext EntityManager entityManager;
    EntityManagerProxy entityManagerProxy; //使用它才能得到实际使用的Hibernate Search提供的EntityManager。
    @Override
    public Page<SearchResult<ForumPost>> search(String query,
                                                Pageable pageable)
    {
        FullTextEntityManager manager = this.getFullTextEntityManager(); 

        QueryBuilder builder = manager.getSearchFactory().buildQueryBuilder() //从实体类创建一个QueryBuilder
                .forEntity(ForumPost.class).get();

        Query lucene = builder.keyword()   
                .onFields("title", "body", "keywords", "user.username")
                .matching(query)
                .createQuery();

        FullTextQuery q = manager.createFullTextQuery(lucene, ForumPost.class); //创建全文搜索查询
        q.setProjection(FullTextQuery.THIS, FullTextQuery.SCORE);//第一个参数说明将该实体作为对象数组的第一个实体返回,第二个参数说明将相关性得分作为第二个元素返回。

        long total = q.getResultSize(); //得到总大小

        q.setFirstResult(pageable.getOffset())
                .setMaxResults(pageable.getPageSize()); //设置分页信息

        @SuppressWarnings("unchecked") List<Object[]> results = q.getResultList(); //获取分页结果
        List<SearchResult<ForumPost>> list = new ArrayList<>();
        results.forEach(o -> list.add(  
                new SearchResult<>((ForumPost)o[0], (Float)o[1]) //o[0]表示得到的实体对象，o[1]表示相关性得分。对应于FullTextQuery.THIS, FullTextQuery.SCORE
        ));

        return new PageImpl<>(list, pageable, total);
    }

    private FullTextEntityManager getFullTextEntityManager()
    {
        return Search.getFullTextEntityManager(
                this.entityManagerProxy.getTargetEntityManager() //获取实际的全文搜索的EntityManager
        );
    }

    @PostConstruct  //Bean生命周期注解方法
    public void initialize()
    {
        if(!(this.entityManager instanceof EntityManagerProxy))
            throw new FatalBeanException("Entity manager " + this.entityManager +
                    " was not a proxy");

        this.entityManagerProxy = (EntityManagerProxy)this.entityManager; //强制转换。
    }
}


```
**注意：Lucene不参与JPA事务**，所以在调用getResultSize和getResultList之间，结果的总数可能会发生变化。


##  高亮显示结果 
```

public PageModel<Article> searchArticle(int pageNum, int pageSize, String keyword) {
    FullTextSession fts = Search.getFullTextSession(sessionFactory.getCurrentSession());
    QueryBuilder qb = fts.getSearchFactory().buildQueryBuilder().forEntity(Article.class).get();
    Query luceneQuery = qb.keyword().onFields("title", "content", "description").matching(keyword).createQuery();
    FullTextQuery query = fts.createFullTextQuery(luceneQuery, Article.class);
    query.setFirstResult((pageNum - 1) * pageSize);
    query.setMaxResults(pageSize);
    List<Article> data = query.list();
    //封装分页数据
    PageModel<Article> model = new PageModel<>(pageNum, pageSize, data.size());
    //将数据高亮
    model.setData(hightLight(luceneQuery, data, "title", "content", "description"));
    return model;
}
/**
* 高亮显示文章
* 
* @param query {@link org.apache.lucene.search.Query}
* @param data 未高亮的数据
* @param fields 需要高亮的字段
* @return 高亮数据
*/
public static List<Article> hightLight(Query query, List<Article> data, String... fields) {
    List<Article> result = new ArrayList<Article>();
    Formatter formatter = new SimpleHTMLFormatter("<b style=\"color:red\">", "</b>"); //设定高亮格式
    QueryScorer queryScorer = new QueryScorer(query); 
    Highlighter highlighter = new Highlighter(formatter, queryScorer); //获取高亮器对象
    // 使用IK中文分词
    Analyzer analyzer = new IKAnalyzer();
    for (Article a : data) {
    // 构建新的对象进行返回，避免页面错乱（我的页面有错乱）
    Article article = new Article();
    for (String fieldName : fields) {
        // 获得字段值，并给新的文章对象赋值
        Object fieldValue = ReflectionUtils
            .invokeMethod(BeanUtils.getPropertyDescriptor(Article.class, fieldName).getReadMethod(),a); //获取字段值
        ReflectionUtils.invokeMethod(BeanUtils.getPropertyDescriptor(Article.class, fieldName).getWriteMethod(), 
            article, fieldValue); //将属性值写入新对象
        String hightLightFieldValue = null;
        try {
            hightLightFieldValue = highlighter.getBestFragment(analyzer, fieldName, String.valueOf(fieldValue)); //格式化高亮
        } catch (Exception e) {
            throw new RuntimeException("高亮显示关键字失败", e);
        }
        // 如果高亮成功则重新赋值
        if (hightLightFieldValue != null) {
            ReflectionUtils.invokeMethod(BeanUtils.getPropertyDescriptor(Article.class, fieldName).getWriteMethod(),
                article,hightLightFieldValue);
        }
        }
        // 赋值ID
        ReflectionUtils.invokeMethod(BeanUtils.getPropertyDescriptor(Article.class, "id").getWriteMethod(),
            article, a.getId());
        result.add(article);
    }
    return result;
}

```
参考：http://hibernate.org/search/documentation/getting-started/
http://my.oschina.net/harmel/blog/491159
http://blog.csdn.net/dm_vincent/article/details/40649203
http://my.oschina.net/weiwei/blog/490073
http://simonlei.iteye.com/blog/577093