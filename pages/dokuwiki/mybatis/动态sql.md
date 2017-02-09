title: 动态sql 

#  mybatis:动态SQL 

MyBatis 的强大特性之一便是它的动态 SQL 能力。动态 SQL 元素和使用 JSTL 或其他相似的基于 XML 的文本处理器相似。在 MyBatis 之前的版本中,有很多的元素需要来了解。MyBatis 3 大大提升了它们,现在用不到原先一半的元素就能工作了。MyBatis 采用功能强大的基于 OGNL 的表达式来消除其他元素。
  * if
  * choose (when, otherwise)
  * trim (where, set)
  * foreach

#  if 

动态 SQL 通常要做的事情是有条件地包含 where 子句的一部分。
```

<select id="findActiveBlogWithTitleLike"  
     resultType="Blog">  
  SELECT * FROM BLOG   
  WHERE state = ‘ACTIVE’   
  <if test="title != null">  
    AND title like #{title}  
  </if>  
</select>  

```
就这个例子而言，细心的读者会发现其中的参数值是可以包含一些**掩码或通配符的**

#  choose, when, otherwise 

MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句
```

<select id="findActiveBlogLike"  
     resultType="Blog">  
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’  
  <choose>  
    <when test="title != null">  
      AND title like #{title}  
    </when>  
    <when test="author != null and author.name != null">  
      AND author_name like #{author.name}  
    </when>  
    <otherwise>  
      AND featured = 1  
    </otherwise>  
  </choose>  
</select>  

```
#  trim, where, set 

前面几个例子已经合宜地解决了一个臭名昭著的动态 SQL 问题。现在考虑回到“if”示例，这次我们将“ACTIVE = 1”也设置成动态的条件，看看会发生什么。
```

<select id="findActiveBlogLike"  
     resultType="Blog">  
  SELECT * FROM BLOG   
  WHERE   
  <if test="state != null">  
    state = #{state}  
  </if>   
  <if test="title != null">  
    AND title like #{title}  
  </if>  
  <if test="author != null and author.name != null">  
    AND author_name like #{author.name}  
  </if>  
</select>  

```
如果这些条件没有一个能匹配上将会怎样？最终这条 SQL 会变成这样：
SELECT * FROM BLOG  
WHERE  
这会导致查询失败。如果仅仅第二个条件匹配又会怎样？这条 SQL 最终会是这样:
SELECT * FROM BLOG  
WHERE   
AND title like ‘someTitle’  
MyBatis 有一个简单的处理，这在90%的情况下都会有用。而在不能使用的地方，你可以自定义处理方式来令其正常工作。一处简单的修改就能得到想要的效果：
```

<select id="findActiveBlogLike"  
     resultType="Blog">  
  SELECT * FROM BLOG   
  <where>   
    <if test="state != null">  
         state = #{state}  
    </if>   
    <if test="title != null">  
        AND title like #{title}  
    </if>  
    <if test="author != null and author.name != null">  
        AND author_name like #{author.name}  
    </if>  
  </where>  
</select>  

```
where 元素知道只有在一个以上的if条件有值的情况下才去插入“WHERE”子句。而且，若最后的内容是“AND”或“OR”开头的，where 元素也知道如何将他们去除。
我们还是可以通过自定义 trim 元素来定制我们想要的功能。比如，和 where 元素等价的自定义 trim 元素为：
<trim prefix="WHERE" prefixOverrides="AND |OR ">  
  ...   
</trim>  
类似的用于动态更新语句的解决方案叫做 set。set 元素可以被用于动态包含需要更新的列，而舍去其他的。比如：
```

<update id="updateAuthorIfNecessary">  
  update Author  
    <set>  
      <if test="username != null">username=#{username},</if>  
      <if test="password != null">password=#{password},</if>  
      <if test="email != null">email=#{email},</if>  
      <if test="bio != null">bio=#{bio}</if>  
    </set>  
  where id=#{id}  
</update>  

```
这里，set 元素会动态前置 SET 关键字，同时也会消除无关的逗号，因为用了条件语句之后很可能就会在生成的赋值语句的后面留下这些逗号。
若你对等价的自定义 trim 元素的样子感兴趣，那这就应该是它的真面目：
```

<trim prefix="SET" suffixOverrides=",">  
  ...  
</trim>  

```
注意这里我们忽略的是后缀中的值，而又一次附加了前缀中的值

#  foreach 

动态 SQL 的另外一个常用的必要操作是需要对一个集合进行遍历，通常是在构建 IN 条件语句的时候
```

<select id="selectPostIn" resultType="domain.blog.Post">  
  SELECT *  
  FROM POST P  
  WHERE ID in  
  <foreach item="item" index="index" collection="list"  
      open="(" separator="," close=")">  
        #{item}  
  </foreach>  
</select> 

```
**你可以将一个 List 实例或者数组作为参数对象传给 MyBatis，当你这么做的时候，MyBatis 会自动将它包装在一个 Map 中并以名称为键。List 实例将会以“list”作为键，而数组实例的键将是“array”。**

#  bind 

bind 元素可以从 OGNL 表达式中创建一个变量并将其绑定到上下文。比如：
```

<select id="selectBlogsLike" resultType="Blog">  
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />  
  SELECT * FROM BLOG  
  WHERE title LIKE #{pattern}  
</select>  

```
#  Multi-db vendor support 

一个配置了“_databaseId”变量的 databaseIdProvider 对于动态代码来说是可用的，这样就可以根据不同的数据库厂商构建特定的语句。比如下面的例子：
```

<insert id="insert">  
  <selectKey keyProperty="id" resultType="int" order="BEFORE">  
    <if test="_databaseId == 'oracle'">  
      select seq_users.nextval from dual  
    </if>  
    <if test="_databaseId == 'db2'">  
      select nextval for seq_users from sysibm.sysdummy1"  
    </if>  
  </selectKey>  
  insert into users values (#{id}, #{name})  
</insert>  

```
#  Pluggable Scripting Languages For Dynamic SQL 

MyBatis 从 3.2 开始支持可插拔的脚本语言，因此你可以在插入一种语言的驱动（language driver）之后来写基于这种语言的动态 SQL 查询。