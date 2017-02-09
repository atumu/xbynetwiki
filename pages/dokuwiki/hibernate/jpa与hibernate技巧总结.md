title: jpa与hibernate技巧总结 

#  JPA与Hibernate技巧总结 
##  让JPA的Query返回Map对象 
在JPA 2.0 中我们可以使用` entityManager.createNativeQuery() `来执行原生的SQL语句。 
但当我们查询结果没有对应实体类时，` query.getResultList()返回的是一个List<Object[]> `。也就是说**每行的数据被作为一个对象数组返回**。 
可惜的是JPA的API中并没有提供这样的设置。其实很多JPA的底层实现都是支持返回Map对象的。例如：
EclipseLink的query.setHint(QueryHints.RESULT_TYPE, ResultType.Map); 
` Hibernate的.setResultTransformer(Transformers.ALIAS_TO_ENTITY_MAP); `
所以，如果我们想要返回Map并且确定底层用的是某一种JPA的实现时我们可以退而求其次， 牺牲跨实现的特性来满足我们的需求：
```

public void testNativeQuery(){  
    Query query = entityManager.createNativeQuery("select id, name, age from t_user");  
    query.unwrap(SQLQuery.class).setResultTransformer(Transformers.ALIAS_TO_ENTITY_MAP);  
    List rows = query.getResultList();  
    for (Object obj : rows) {  
        Map row = (Map) obj;  
        System.out.println("id = " + row.get("ID"));  
        System.out.println("name = " + row.get("NAME"));  
        System.out.println("age = " + row.get("AGE"));  
    }  
}  

```
这里需要注意的是， 用Map肯定要比用Object数组来的效率低。所以你要看性能下降是否在可接受范围内。再就是在我的Hibernate 4.2.x的环境下，无论你原生SQL中写的是大写字母还是小写字母，返回的字段名都是大写的。当然你可以通过自定义ResultTransformer的形式对字段名进行一定的处理， 甚至是返回自己需要的POJO。
参考：http://blog.csdn.net/a9529lty/article/details/21597615

##  JPA结合Hibernate UUID主键生成策略 
```

    @Id
    @GeneratedValue(generator = "system-uuid")
    @GenericGenerator(name = "system-uuid", strategy = "uuid")
    @Column(name = "NEWS_ID")
    private String newsId;

```
@GeneratedValue(generator = 可以随便写代表主键字符串生成器的名字，只要能对应上( @GenericGenerator的name)。
  * javax.persistence.GeneratedValue;
  * org.hibernate.annotations.GenericGenerator;


参考：http://bbs.csdn.net/topics/240015295