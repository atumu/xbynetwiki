title: mapper_xml映射 

##  mybatis:mapper xml映射

Mapper XML文件
MyBatis 的真正强大在于它的映射语句，也是它的魔力所在。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。MyBatis 就是针对 SQL 构建的，并且比普通的方法做的更好。

SQL 映射文件有很少的几个顶级元素（按照它们应该被定义的顺序）：

  - cache – 给定命名空间的缓存配置。
  - cache-ref – 其他命名空间缓存配置的引用。
  - resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
  -  parameterMap – 已废弃！老式风格的参数映射。内联参数是首选,这个元素可能在将来被移除，这里不会记录。
  - sql – 可被其他语句引用的可重用语句块。
  - insert – 映射插入语句
  - update – 映射更新语句
  - delete – 映射删除语句
  - select – 映射查询语句
##  cache缓存 
MyBatis 包含一个非常强大的查询缓存特性,它可以非常方便地配置和定制。默认情况下是没有开启缓存的,除了局部的 session 缓存。要开启二级缓存,你需要在你的 SQL 映射文件中添加一行:
<blockquote><cache/></blockquote>  
这个简单语句的效果如下:
  - 映射语句文件中的所有 select 语句将会被缓存。
  - 映射语句文件中的所有 insert,update 和 delete 语句会刷新缓存。
  - 缓存会使用 Least Recently Used(LRU,最近最少使用的)算法来收回。
  - 根据时间表(比如 no Flush Interval,没有刷新间隔), 缓存不会以任何时间顺序 来刷新。
  - 缓存会存储列表集合或对象(无论查询方法返回什么)的 1024 个引用。
  - 缓存会被视为是 read/write(可读/可写)的缓存,意味着对象检索不是共享的,而 且可以安全地被调用者修改,而不干扰其他调用者或线程所做的潜在修改。
所有的这些属性都可以通过缓存元素的属性来修改。比如
```

<cache  
  eviction="FIFO"  
  flushInterval="60000"  
  size="512"  
  readOnly="true"/>  

```
这个更高级的配置创建了一个 FIFO 缓存,并每隔 60 秒刷新,存数结果对象或列表的 512 个引用,而且返回的对象被认为是只读的,因此在不同线程中的调用者之间修改它们会 导致冲突。

可用的收回策略有:

LRU – 最近最少使用的:移除最长时间不被使用的对象。
FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。
默认的是 LRU。

flushInterval(刷新间隔)可以被设置为任意的正整数,而且它们代表一个合理的毫秒 形式的时间段。默认情况是不设置,也就是没有刷新间隔,缓存仅仅调用语句时刷新。

size(引用数目)可以被设置为任意正整数,要记住你缓存的对象数目和你运行环境的 可用内存资源数目。默认值是 1024。

readOnly(只读)属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓 存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存 会返回缓存对象的拷贝(通过序列化) 。这会慢一些,但是安全,因此默认是 false。
###  自定义缓存 

<cache type="com.domain.something.MyCustomCache"/>  
type 属 性指 定的 类必 须实现 org.mybatis.cache.Cache 接口。这个接口是 MyBatis 框架中很多复杂的接口之一,但是简单 给定它做什么就行。

###  缓存引用 

<cache-ref namespace="com.someone.application.data.SomeMapper"/> 
##  自动映射 

在简单的场景下，MyBatis可以替你自动映射查询结果。 如果遇到复杂的场景，你需要构建一个result map。 但是你也可以混合使用这两种策略。
当自动映射查询结果时，MyBatis会获取sql返回的列名并在java类中查找相同名字的属性（忽略大小写）。 这意味着如果Mybatis发现了ID列和id属性，Mybatis会将ID的值赋给id。

**通常数据库列使用大写单词命名，单词间用下划线分隔；而java属性一般遵循驼峰命名法。**` 为了在这两种命名方式之间启用自动映射，需要将 mapUnderscoreToCamelCase设置为true。 `

在接下来的例子中， id 和 userName列将被自动映射， hashed_password 列将根据配置映射。
```

<select id="selectUsers" resultMap="userResultMap">  
  select  
    user_id             as "id",  
    user_name           as "userName",  
    hashed_password  
  from some_table  
  where id = #{id}  
</select>  
<resultMap id="userResultMap" type="User">  
  <result property="password" column="hashed_password"/>  
</resultMap>  

```
三种自动映射模式
NONE
PARTIAL 默认
FULL

##  select 

简单查询的 select 元素是非常简单的。比如：
```

<select id="selectPerson" parameterType="int" resultType="hashmap">  
  SELECT * FROM PERSON WHERE ID = #{id}  
</select>  

```
这个语句被称作 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值。

**注意参数符号：#{id}  这就告诉 MyBatis 创建一个预处理语句参数，通过 JDBC，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中,**就如下面这样
```

String selectPerson = "SELECT * FROM PERSON WHERE ID=?";  
PreparedStatement ps = conn.prepareStatement(selectPerson);  
ps.setInt(1,id);  

```
select 元素有很多属性允许你配置，来决定每条语句的作用细节。
```

<select  
  id="selectPerson"  
  parameterType="int"  
  resultType="hashmap"  
  resultMap="personResultMap"  
  flushCache="false"  
  useCache="true"  
  timeout="10000"  
  fetchSize="256"  
  statementType="PREPARED"  
  resultSetType="FORWARD_ONLY">  

```
![](/data/dokuwiki/pasted/20150511-025917.png)

##  insert, update 和 delete 

数据变更语句 insert，update 和 delete 的实现非常接近：
```

<insert  
  id="insertAuthor"  
  parameterType="domain.blog.Author"  
  flushCache="true"  
  statementType="PREPARED"  
  keyProperty=""  
  keyColumn=""  
  useGeneratedKeys=""  
  timeout="20">  
  
<update  
  id="insertAuthor"  
  parameterType="domain.blog.Author"  
  flushCache="true"  
  statementType="PREPARED"  
  timeout="20">  
  
<delete  
  id="insertAuthor"  
  parameterType="domain.blog.Author"  
  flushCache="true"  
  statementType="PREPARED"  
  timeout="20">  

```
![](/data/dokuwiki/pasted/20150511-030029.png)
下面就是 insert，update 和 delete 语句的示例：
```

<insert id="insertAuthor">  
  insert into Author (id,username,password,email,bio)  
  values (#{id},#{username},#{password},#{email},#{bio})  
</insert>  
  
<update id="updateAuthor">  
  update Author set  
    username = #{username},  
    password = #{password},  
    email = #{email},  
    bio = #{bio}  
  where id = #{id}  
</update>  
  
<delete id="deleteAuthor">  
  delete from Author where id = #{id}  
</delete>  

```
插入语句的配置规则更加丰富，` 在插入语句里面有一些额外的属性和子元素用来处理主键的生成，而且有多种生成方式 `。
首先，**如果你的数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server）**，那么你可以` 设置 useGeneratedKeys=”true” `，然后再把`  keyProperty ` 设置到目标属性上就OK了。例如，如果上面的 Author 表已经对 id 使用了自动生成的列类型，那么语句可以修改为:
```

<insert id="insertAuthor" useGeneratedKeys="true"  
    keyProperty="id">  
  insert into Author (username,password,email,bio)  
  values (#{username},#{password},#{email},#{bio})  
</insert>  

```
对于不支持自动生成类型的数据库或可能不支持自动生成主键 JDBC 驱动来说，MyBatis 有另外一种方法来生成主键。
这里有一个简单（甚至很傻）的示例，它可以生成一个随机 ID（你最好不要这么做，但这里展示了 MyBatis 处理问题的灵活性及其所关心的广度）：
```

<insert id="insertAuthor">  
  <selectKey keyProperty="id" resultType="int" order="BEFORE">  
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1  
  </selectKey>  
  insert into Author  
    (id, username, password, email,bio, favourite_section)  
  values  
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})  
</insert>  

```
在上面的示例中，selectKey 元素将会首先运行，Author 的 id 会被设置，然后插入语句会被调用。这给你了一个和数据库中来处理自动生成的主键类似的行为，避免了使 Java 代码变得复杂。
**selectKey 元素描述如下：**
```

<selectKey  
  keyProperty="id"  
  resultType="int"  
  order="BEFORE"  
  statementType="PREPARED">  

```

order	这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。

##  sql 

这个元素可以被用来定义可重用的 SQL 代码段，可以包含在其他语句中。比如：
```

<sql id="userColumns"> id,username,password </sql>  
这个 SQL 片段可以被包含在其他语句中，例如：
[html] view plaincopy
<select id="selectUsers" resultType="map">  
  select <include refid="userColumns"/>  
  from some_table  
  where id = #{id}  
</select>  

```

##  参数（Parameters） 

```

<select id="selectUsers" resultType="User">  
  select id, username, password  
  from users  
  where id = #{id}  
</select>  

```
上面的这个示例说明了一个非常简单的命名参数映射。参数类型被设置为 int，这样这个参数就可以被设置成任何内容。原生的类型或简单数据类型（比如整型和字符串）因为没有相关属性，它会完全用参数值来替代。然而，如果传入一个复杂的对象，行为就会有一点不同了。比
```

<insert id="insertUser" parameterType="User">  
  insert into users (id, username, password)  
  values (#{id}, #{username}, #{password})  
</insert>  

```
如果 User 类型的参数对象传递到了语句中，id、username 和 password 属性将会被查找，然后将它们的值传入预处理语句的参数中。
这点对于向语句中传参是比较好的而且又简单，不过参数映射的功能远不止于此。
首先，像 MyBatis 的其他部分一样，参数也可以指定一个特殊的数据类型。
#{property,javaType=int,jdbcType=NUMERIC}  
` 注意：如果 null 被当作值来传递，对于所有可能为空的列，JDBC Type 是需要的。 `
为了以后定制类型处理方式，你也可以指定一个特殊的类型处理器类（或别名），比如：
#{age,javaType=int,jdbcType=NUMERIC,typeHandler=MyTypeHandler}  
对于数值类型，还有一个小数保留位数的设置，来确定小数点后保留的位数。
#{height,javaType=double,jdbcType=NUMERIC,numericScale=2}  
尽管所有这些强大的选项很多时候你只简单指定属性名，其他的事情 MyBatis 会自己去推断，` 最多你需要为可能为空的列名指定 jdbcType。 `
#{firstName}  
#{middleInitial,jdbcType=VARCHAR}  
#{lastName}  

##  Result Maps 

resultMap 元素是 MyBatis 中最重要最强大的元素。它就是让你远离 90%的需要从结果 集中取出数据的 JDBC 代码的那个东西,
你的应 用程序将会使用 JavaBeans 或 POJOs(Plain Old Java Objects,普通 Java 对象)来作为领域 模型。MyBatis 对两者都支持。看看下面这个 JavaBean:
```

package com.someapp.model;  
public class User {  
  private int id;  
  private String username;  
  private String hashedPassword;  
    
  public int getId() {  
    return id;  
  }  
  public void setId(int id) {  
    this.id = id;  
  }  
  public String getUsername() {  
    return username;  
  }  
  public void setUsername(String username) {  
    this.username = username;  
  }  
  public String getHashedPassword() {  
    return hashedPassword;  
  }  
  public void setHashedPassword(String hashedPassword) {  
    this.hashedPassword = hashedPassword;  
  }  
} 

<select id="selectUsers" resultType="com.someapp.model.User">  
  select id, username, hashedPassword  
  from some_table  
  where id = #{id}  
</select>  

```
###  如果列名没有精确匹配, 
你可以在列名上使用 select 字句的别名(一个 基本的 SQL 特性)来匹配标签。比如:
```

<select id="selectUsers" resultType="User">  
  select  
    user_id             as "id",  
    user_name           as "userName",  
    hashed_password     as "hashedPassword"  
  from some_table  
  where id = #{id}  
</select>  

```
###  解决列名不匹配的另外一种方式 

```

<resultMap id="userResultMap" type="User">  
  <id property="id" column="user_id" />  
  <result property="username" column="username"/>  
  <result property="password" column="password"/>  
</resultMap>  
<select id="selectUsers" resultMap="userResultMap">  
  select user_id, user_name, hashed_password  
  from some_table  
  where id = #{id}  
</select>  

```

##  高级结果映射 
```

<!-- Very Complex Statement -->  
<select id="selectBlogDetails" resultMap="detailedBlogResultMap">  
  select  
       B.id as blog_id,  
       B.title as blog_title,  
       B.author_id as blog_author_id,  
       A.id as author_id,  
       A.username as author_username,  
       A.password as author_password,  
       A.email as author_email,  
       A.bio as author_bio,  
       A.favourite_section as author_favourite_section,  
       P.id as post_id,  
       P.blog_id as post_blog_id,  
       P.author_id as post_author_id,  
       P.created_on as post_created_on,  
       P.section as post_section,  
       P.subject as post_subject,  
       P.draft as draft,  
       P.body as post_body,  
       C.id as comment_id,  
       C.post_id as comment_post_id,  
       C.name as comment_name,  
       C.comment as comment_text,  
       T.id as tag_id,  
       T.name as tag_name  
  from Blog B  
       left outer join Author A on B.author_id = A.id  
       left outer join Post P on B.id = P.blog_id  
       left outer join Comment C on P.id = C.post_id  
       left outer join Post_Tag PT on PT.post_id = P.id  
       left outer join Tag T on PT.tag_id = T.id  
  where B.id = #{id}  
</select> 

```
你可能想把它映射到一个智能的对象模型,包含一个作者写的博客,有很多的博文,每 篇博文有零条或多条的评论和标签。 下面是一个完整的复杂结果映射例子 (假设作者, 博客, 博文, 评论和标签都是类型的别名) 我们来看看, 。 但是不用紧张, 我们会一步一步来说明。 当天最初它看起来令人生畏,但实际上非常简单。
```

<!-- Very Complex Result Map -->  
<resultMap id="detailedBlogResultMap" type="Blog">  
  <constructor>  
    <idArg column="blog_id" javaType="int"/>  
  </constructor>  
  <result property="title" column="blog_title"/>  
  <association property="author" javaType="Author">  
    <id property="id" column="author_id"/>  
    <result property="username" column="author_username"/>  
    <result property="password" column="author_password"/>  
    <result property="email" column="author_email"/>  
    <result property="bio" column="author_bio"/>  
    <result property="favouriteSection" column="author_favourite_section"/>  
  </association>  
  <collection property="posts" ofType="Post">  
    <id property="id" column="post_id"/>  
    <result property="subject" column="post_subject"/>  
    <association property="author" javaType="Author"/>  
    <collection property="comments" ofType="Comment">  
      <id property="id" column="comment_id"/>  
    </collection>  
    <collection property="tags" ofType="Tag" >  
      <id property="id" column="tag_id"/>  
    </collection>  
    <discriminator javaType="int" column="draft">  
      <case value="1" resultType="DraftPost"/>  
    </discriminator>  
  </collection>  
</resultMap> 

```

resultMap 元素有很多子元素和一个值得讨论的结构。 下面是 resultMap 元素的概念视图
<html>
<ul style="padding:0px; margin:0px 0px 10px 25px">
<li><span style="padding:1px 3px">constructor</span>&nbsp;- 类在实例化时,用来注入结果到构造方法中
<ul style="padding:0px; margin:0px 0px 0px 25px">
<li><span style="font-family:Monaco,Andale Mono,Courier New,monospace"><span style="font-size:0.8em; padding:1px 3px; background-color:rgb(254,233,204)">idArg</span></span>&nbsp;- ID 参数;标记结果作为 ID 可以帮助提高整体效能</li><li><span style="font-family:Monaco,Andale Mono,Courier New,monospace"><span style="font-size:0.8em; padding:1px 3px; background-color:rgb(254,233,204)">arg</span></span>&nbsp;- 注入到构造方法的一个普通结果</li></ul>
</li><li><span style="font-family:Monaco,Andale Mono,Courier New,monospace"><span style="font-size:0.8em; padding:1px 3px; background-color:rgb(254,233,204)">id</span></span>&nbsp;– 一个 ID 结果;标记结果作为 ID 可以帮助提高整体效能</li><li><span style="font-family:Monaco,Andale Mono,Courier New,monospace"><span style="font-size:0.8em; padding:1px 3px; background-color:rgb(254,233,204)">result</span></span>&nbsp;– 注入到字段或 JavaBean 属性的普通结果</li><li><span style="font-family:Monaco,Andale Mono,Courier New,monospace"><span style="font-size:0.8em; padding:1px 3px; background-color:rgb(254,233,204)">association</span></span>&nbsp;– 一个复杂的类型关联;许多结果将包成这种类型
<ul style="padding:0px; margin:0px 0px 0px 25px">
<li>嵌入结果映射 – 结果映射自身的关联,或者参考一个</li></ul>
</li><li><span style="font-family:Monaco,Andale Mono,Courier New,monospace"><span style="font-size:0.8em; padding:1px 3px; background-color:rgb(254,233,204)">collection</span></span>&nbsp;– 复杂类型的集
<ul style="padding:0px; margin:0px 0px 0px 25px">
<li>嵌入结果映射 – 结果映射自身的集,或者参考一个</li></ul>
</li><li><span style="font-family:Monaco,Andale Mono,Courier New,monospace"><span style="font-size:0.8em; line-height:20px; padding:1px 3px; background-color:rgb(254,233,204)">discriminator</span></span>&nbsp;– 使用结果&#20540;来决定使用哪个结果映射
<ul style="padding:0px; margin:0px 0px 0px 25px">
<li><span style="font-family:Monaco,Andale Mono,Courier New,monospace"><span style="font-size:0.8em; line-height:20px; padding:1px 3px; background-color:rgb(254,233,204)">case</span></span>&nbsp;– 基于某些&#20540;的结果映射
<ul style="padding:0px; margin:0px 0px 0px 25px">
<li>嵌入结果映射 – 这种情形结果也映射它本身,因此可以包含很多相 同的元素,或者它可以参照一个外部的结果映射。</li></ul>
</li></ul>
</li></ul>
</html>
最佳实践提示:你确定你实现想要的行为的最好选择是编写单元 测试。
##  支持的 JDBC 类型 

为了未来的参考,MyBatis 通过包含的 jdbcType 枚举型,支持下面的 JDBC 类型。

##  构造方法 

```

<constructor>  
   <idArg column="id" javaType="int"/>  
   <arg column="username" javaType="String"/>  
</constructor>  
看看下面这个构造方法:  
public class User {  
   //...  
   public User(int id, String username) {  
     //...  
  }  
//...  
}  

```
##  关联association 
```

<association property="author" column="blog_author_id" javaType="Author">  
  <id property="id" column="author_id"/>  
  <result property="username" column="author_username"/>  
</association>  

```
关联元素处理“有一个”类型的关系。比如,在我们的示例中,一个博客有一个用户。 关联映射就工作于这种结果之上。关联中不同的是你需要告诉 MyBatis 如何加载关联。MyBatis 在这方面会有两种不同的 方式:

嵌套查询:通过执行另外一个 SQL 映射语句来返回预期的复杂类型。
嵌套结果:使用嵌套结果映射来处理重复的联合结果的子集。首先,然让我们来查看这个元素的属性。所有的你都会看到,它和普通的只由 select 和resultMap 属性的结果映射不同。

###  嵌套查询 

```

<resultMap id="blogResult" type="Blog">  
  <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>  
</resultMap>  
  
<select id="selectBlog" resultMap="blogResult">  
  SELECT * FROM BLOG WHERE ID = #{id}  
</select>  
  
<select id="selectAuthor" resultType="Author">  
  SELECT * FROM AUTHOR WHERE ID = #{id}  
</select>  

```
这种方式很简单, ` 但是对于大型数据集合和列表将不会表现很好。 问题就是我们熟知的 “N+1 查询问题”。 `概括地讲,N+1 查询问题可以是这样引起的:
你执行了一个单独的 SQL 语句来获取结果列表(就是“+1”)。
对返回的每条记录,你执行了一个查询语句来为每个加载细节(就是“N”)。

这个问题会导致成百上千的 SQL 语句被执行。这通常不是期望的。

###  嵌套结果 


```

<select id="selectBlog" resultMap="blogResult">  
  select  
    B.id            as blog_id,  
    B.title         as blog_title,  
    B.author_id     as blog_author_id,  
    A.id            as author_id,  
    A.username      as author_username,  
    A.password      as author_password,  
    A.email         as author_email,  
    A.bio           as author_bio  
  from Blog B left outer join Author A on B.author_id = A.id  
  where B.id = #{id}  
</select>  

<resultMap id="blogResult" type="Blog">  
  <id property="id" column="blog_id" />  
  <result property="title" column="blog_title"/>  
  <association property="author" column="blog_author_id" javaType="Author" resultMap="authorResult"/>  
</resultMap>  
  
<resultMap id="authorResult" type="Author">  
  <id property="id" column="author_id"/>  
  <result property="username" column="author_username"/>  
  <result property="password" column="author_password"/>  
  <result property="email" column="author_email"/>  
  <result property="bio" column="author_bio"/>  
</resultMap>  

```
还可以通过添加列前缀**columnPrefix**重用resultMap:看下面这个多个了co-author:
```

<select id="selectBlog" resultMap="blogResult">  
  select  
    B.id            as blog_id,  
    B.title         as blog_title,  
    A.id            as author_id,  
    A.username      as author_username,  
    A.password      as author_password,  
    A.email         as author_email,  
    A.bio           as author_bio,  
    CA.id           as co_author_id,  
    CA.username     as co_author_username,  
    CA.password     as co_author_password,  
    CA.email        as co_author_email,  
    CA.bio          as co_author_bio  
  from Blog B  
  left outer join Author A on B.author_id = A.id  
  left outer join Author CA on B.co_author_id = CA.id  
  where B.id = #{id}  
</select>  
而我们只需要一个针对author的resultMap:
<resultMap id="authorResult" type="Author">  
  <id property="id" column="author_id"/>  
  <result property="username" column="author_username"/>  
  <result property="password" column="author_password"/>  
  <result property="email" column="author_email"/>  
  <result property="bio" column="author_bio"/>  
</resultMap>  

```
通过` columnPrefix `重用这个resultMap:
```

<resultMap id="blogResult" type="Blog">  
  <id property="id" column="blog_id" />  
  <result property="title" column="blog_title"/>  
  <association property="author"  
    resultMap="authorResult" />  
  <association property="coAuthor"  
    resultMap="authorResult"  
    columnPrefix="co_" />  
</resultMap>  

```

##  集合collection-处理多个关联 
```

<collection property="posts" ofType="domain.blog.Post">  
  <id property="id" column="post_id"/>  
  <result property="subject" column="post_subject"/>  
  <result property="body" column="post_body"/>  
</collection>  

```
我们来继续上面的示例,一个博客只有一个作者。但是博客有很多文章。在博客类中, 这可以由下面这样的写法来表示:
private List<Post> posts;

###  集合的嵌套查询 

```

<resultMap id="blogResult" type="Blog">  
  <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>  
</resultMap>  
  
<select id="selectBlog" resultMap="blogResult">  
  SELECT * FROM BLOG WHERE ID = #{id}  
</select>  
  
<select id="selectPostsForBlog" resultType="Blog">  
  SELECT * FROM POST WHERE BLOG_ID = #{id}  
</select>  

```
这里你应该注意很多东西,但大部分代码和上面的关联元素是非常相似的。首先,你应 该注意我们使用的是集合元素。然后要注意那个新的“ofType”属性。这个属性用来区分 JavaBean(或字段)属性类型和集合包含的类型来说是很重要的。
读作: “在 Post 类型的 ArrayList 中的 posts 的集合。”javaType 属性是不需要的,因为 MyBatis 在很多情况下会为你算出来。所以你可以缩短 写法:

###  集合的嵌套结果 

```

<select id="selectBlog" resultMap="blogResult">  
  select  
  B.id as blog_id,  
  B.title as blog_title,  
  B.author_id as blog_author_id,  
  P.id as post_id,  
  P.subject as post_subject,  
  P.body as post_body,  
  from Blog B  
  left outer join Post P on B.id = P.blog_id  
  where B.id = #{id}  
</select>  

<resultMap id="blogResult" type="Blog">  
  <id property="id" column="blog_id" />  
  <result property="title" column="blog_title"/>  
  <collection property="posts" ofType="Post">  
    <id property="id" column="post_id"/>  
    <result property="subject" column="post_subject"/>  
    <result property="body" column="post_body"/>  
  </collection>  
</resultMap>  

```
同样, **如果你引用更长的形式允许你的结果映射的更多cloumnPrefix重用**, 你可以使用下面这个替代 的映射:
```

<resultMap id="blogResult" type="Blog">  
  <id property="id" column="blog_id" />  
  <result property="title" column="blog_title"/>  
  <collection property="posts" ofType="Post" resultMap="blogPostResult" columnPrefix="post_"/>  
</resultMap>  
  
<resultMap id="blogPostResult" type="Post">  
  <id property="id" column="id"/>  
  <result property="subject" column="subject"/>  
  <result property="body" column="body"/>  
</resultMap>  

```
注意：你的应用在找到最佳方法前要一直进行的单元测试和性 能测试

##  鉴别器descriminator 

```

<discriminator javaType="int" column="draft">  
  <case value="1" resultType="DraftPost"/>  
</discriminator>  

```
鉴别器非常容易理 解,因为它的表现很像 Java 语言中的 switch 语句。
定义鉴别器指定了 column 和 javaType 属性。 列是 MyBatis 查找比较值的地方。 JavaType 是需要被用来保证等价测试的合适类型(尽管字符串在很多情形下都会有用)。比如
```

<resultMap id="vehicleResult" type="Vehicle">  
  <id property="id" column="id" />  
  <result property="vin" column="vin"/>  
  <result property="year" column="year"/>  
  <result property="make" column="make"/>  
  <result property="model" column="model"/>  
  <result property="color" column="color"/>  
  <discriminator javaType="int" column="vehicle_type">  
    <case value="1" resultMap="carResult"/>  
    <case value="2" resultMap="truckResult"/>  
    <case value="3" resultMap="vanResult"/>  
    <case value="4" resultMap="suvResult"/>  
  </discriminator>  
</resultMap>  
或者直接这样
<discriminator javaType="int" column="vehicle_type">  
  <case value="1" resultType="carResult">  
    <result property="doorCount" column="door_count" />  
  </case>  
  <case value="2" resultType="truckResult">  
    <result property="boxSize" column="box_size" />  
    <result property="extendedCab" column="extended_cab" />  
  </case>  
</discriminator>  

```