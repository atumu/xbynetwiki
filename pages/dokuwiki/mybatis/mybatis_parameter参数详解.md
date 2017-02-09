title: mybatis_parameter参数详解 

#  mybatis parameter参数详解 
SqlSession 实例在 MyBatis 中是非常强大的一个类。在这里你会发现 所有执行语句的方法,提交或回滚事务,还有获取映射器实例。
语句执行方法
这些方法被用来执行定义在 SQL 映射的 XML 文件中的 SELECT,INSERT,UPDA E T 和 DELETE 语句。
它们都会自行解释,每一句都使用**语句的 ID 属性**和**参数对象**,` 参数可以 是原生类型(自动装箱或包装类) ,JavaBean,POJO 或 Map。 `
  * <T> T selectOne(String statement, Object parameter)
  * <E> List<E> selectList(String statement, Object parameter)
  * <K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey)
  * int insert(String statement, Object parameter)
  * int update(String statement, Object parameter)
  * int delete(String statement, Object parameter)
**selectOne 和 selectList 的不同仅仅是 selectOne 必须返回一个对象。** 如果多余一个, 或者 没有返回 (或返回了 null) 那么就会抛出异常。  
**如果你不知道需要多少对象, 使用 selectList。**

如果你想检查一个对象是否存在,那么最好返回统计数(0 或 1) 。因为并不是所有语句都需 要参数,这些方法都是有不同重载版本的,它们可以不需要参数对象。
  * <T> T selectOne(String statement)
  * <E> List<E> selectList(String statement)
  * <K,V> Map<K,V> selectMap(String statement, String mapKey)
  * int insert(String statement)
  * int update(String statement)
  * int delete(String statement)
最后,还有查询方法的三个高级版本,它们允许你**限制返回行数的范围**,或者**提供自定 义结果控制逻辑**,这通常用于大量的数据集合。
  * <E> List<E> selectList (String statement, Object parameter, RowBounds rowBounds)
  * <K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey, RowBounds rowbounds)
  * void select (String statement, Object parameter, ResultHandler<T> handler)
  * void select (String statement, Object parameter, RowBounds rowBounds, ResultHandler<T> handler)
**RowBounds 参数会告诉 MyBatis 略过指定数量的记录,还有限制返回结果的数量。** RowBounds 类有一个构造方法来接收 offset 和 limit,否则是不可改变的。
```

int offset = 100;
int limit = 25;
RowBounds rowBounds = new RowBounds(offset, limit);

```
不同的驱动会实现这方面的不同级别的效率。对于最佳的表现,使用结果集类型的 SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE(或句话说:不是 FORWARD_ONLY)。
**ResultHandler 参数允许你按你喜欢的方式处理每一行。** 
它的接口很简单。
```

package org.apache.ibatis.session;
public interface ResultHandler<T> {
  void handleResult(ResultContext<? extends T> context);
}

```
ResultContext 参数给你访问结果对象本身的方法, 大量结果对象被创建, 你可以使用布 尔返回值的 stop()方法来停止 MyBatis 加载更多的结果。


##  具体说明 
**1.传入简单类型**
```

public User get(Long id) {    
    return (User) getSqlSession().selectOne("com.liulanghan.get" , id);  
}

```  
MAPPER :
```

<select id="findUserListByIdList" parameterType="java.lang.Long" resultType="User">  
        select * from user where  id = #{id};  
</select> 

``` 
  
**2. 传入List**
```

public List<Area> findUserListByIdList(List<Long> idList) {  
        return getSqlSession().selectList("com.liulanghan.findUserListByIdList", idList);  
    } 

``` 
MAPPER :
```

   <select id="findUserListByIdList" parameterType="java.util.ArrayList" resultType="User">  
    select * from user user  
    <where>  
        user.ID in (  
        <foreach item="guard" index="index" collection="list"  
            separator=","> #{guard} </foreach>  
        )  
    </where>  
</select>

```   
` **单独传入list时** `，foreach中的collection` **必须是list** `，不管变量的具体名称是什么。比如这里变量名为idList，collection却是是list。 
 
**3. 传入数组**
```

public List<Area> findUserListByIdList(int[] ids) {  
        return getSqlSession().selectList("com.liulanghan.findUserListByIdList", ids);  
    } 

``` 
 
 MAPPER :
```

<select id="findUserListByIdList" parameterType="java.util.HashList" resultType="User">  
    select * from user user  
    <where>  
        user.ID in (  
        <foreach item="guard" index="index" collection="array"  
            separator=","> #{guard} </foreach>  
        )  
    </where>  
</select>  

```   
 ` 单独传入数组时 `，foreach中的collection` 必须是array `，不不管变量的具体名称是什么。比如这里变量名为ids，
 collection却是是array
 
**4.  传入map**
```

public boolean exists(Map<String, Object> map){  
        Object count = getSqlSession().selectOne("com.liulanghan.exists", map);  
        int totalCount = Integer.parseInt(count.toString());  
        return totalCount > 0 ? true : false;  
    }

```  
  
 MAPPER :
```

<select id="exists" parameterType="java.util.HashMap" resultType="java.lang.Integer">  
        SELECT COUNT(*) FROM USER user  
        <where>  
            <if test="code != null">   
                and user.CODE = #{code}   
            </if>  
            <if test="id != null">   
                and user.ID = #{id}   
            </if>  
            <if test="idList !=null ">  
                and user.ID in (  
                <foreach item="guard" index="index" collection="idList"  
                    separator=","> #{guard} </foreach>  
                )  
            </if>  
        </where>  
    </select>

```  
`  MAP中有list或array时 `，foreach中的collection` 必须是具体list或array的变量名 `。比如这里MAP含有一个名为idList的list，所以MAP中用idList取值，这点和单独传list或array时不太一样。
 
 
**5. 传入JAVA对象**
```

public boolean findUserListByDTO(UserDTO userDTO){  
        Object count = getSqlSession().selectOne("com.liulanghan.exists", userDTO);  
        int totalCount = Integer.parseInt(count.toString());  
        return totalCount > 0 ? true : false;  
    }  
 

``` 
 MAPPER :
```

<select id="findUserListByDTO" parameterType="UserDTO" resultType="java.lang.Integer">  
        SELECT COUNT(*) FROM USER user  
        <where>  
            <if test="code != null">   
                and user.CODE = #{code}   
            </if>  
            <if test="id != null">   
                and user.ID = #{id}   
            </if>  
            <if test="idList !=null ">  
                and user.ID in (  
                <foreach item="guard" index="index" collection="idList"  
                    separator=","> #{guard} </foreach>  
                )  
            </if>  
        </where>  
    </select>

```  
` JAVA对象中有list或array时 `，foreach中的collection` 必须是具体list或array的变量名 `。比如这里UserDTO含有一个名为idList的list，所以UserDTO中用idList取值，这点和单独传list或array时不太一样。
 
6.取值
由上面可以看出，取值的时候用的是#{}。它具体的意思是告诉MyBatis创建一个预处理语句参数。
使用JDBC，这样的一个参数在SQL中会由一个“?”来标识，并被传递到一个新的预处理语句中，就像这样：
/ / Similar JDBC code, NOT MyBatis…  
String selectPerson = “SELECT * FROM PERSON WHERE ID=?”;  
PreparedStatement ps = conn.prepareStatement(selectPerson);  
ps.setInt(1,id);  
  
可以看到这个写法比较简单，MyBatis为我们做了很多默认的事情，具体的写法应该如下：
#{property,javaType=int,jdbcType=NUMERIC,typeHandler=MyTypeHandler,mode=OUT,resultMap=User}  
 property:属性名，即代码传入的变量名。
 javaType：该字段在JAVA中的类型，比如int。
 jdbcType：该字段在JDBC中的类型，比如NUMERIC。
 typeHandler:类型处理器
 mode：参数类型为IN，OUT或INOUT参数
 resultMap：结果。
 
** 还好,MyBatis比较体谅我们，一般我们只需写一个属性名即可**，如#{id},其他的如javaType和typeHandlerMybatis
 会自动帮我们填好。` 可是这样有时也会出问题，比如出现CLOB字段时。 `
 由于JAVA代码中的String类型对应的默认typeHandler为StringTypeHandler,**当用String类型处理时，如果String长度超过一定长度，就会报如下错误**：
 setString can only process strings of less than 32766 chararacters
 **解决办法是指定该属性的typeHandler，如下：
 #{message,typeHandler=org.apache.ibatis.type.ClobTypeHandler}**
 我们也可以自定义typeHandler来处理需要的数据，具体这里详述。
 JDBC类型是仅仅需要对插入，更新和删除操作**可能为空的列进行处理**。这是JDBC的需要，而不是MyBatis的。一般不需要配置
 mode、resultMap一般不需要，在写存储过程时会用到，这里不详述。

http://www.mybatis.org/mybatis-3/zh/java-api.html
http://zhuyuehua.iteye.com/blog/1717525
http://blog.csdn.net/isea533/article/details/44002219