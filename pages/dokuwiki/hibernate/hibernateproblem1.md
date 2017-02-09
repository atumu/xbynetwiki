title: hibernateproblem1 

#  Hibernate SQL Exception java.lang.StringIndexOutOfBoundsException: String index out of range 
```

java.lang.StringIndexOutOfBoundsException: String index out of range: 0  
    at java.lang.String.charAt(String.java:658)  
    at org.hibernate.type.descriptor.java.CharacterTypeDescriptor.wrap(CharacterTypeDescriptor.java:80)  
    at org.hibernate.type.descriptor.java.CharacterTypeDescriptor.wrap(CharacterTypeDescriptor.java:34)  
    at org.hibernate.type.descriptor.sql.VarcharTypeDescriptor$2.doExtract(VarcharTypeDescriptor.java:61)  
    at org.hibernate.type.descriptor.sql.BasicExtractor.extract(BasicExtractor.java:64)  
    at org.hibernate.type.AbstractStandardBasicType.nullSafeGet(AbstractStandardBasicType.java:254)  
    at org.hibernate.type.AbstractStandardBasicType.nullSafeGet(AbstractStandardBasicType.java:250)  
    at org.hibernate.type.AbstractStandardBasicType.nullSafeGet(AbstractStandardBasicType.java:230)  
    at org.hibernate.type.AbstractStandardBasicType.hydrate(AbstractStandardBasicType.java:331)  
    at org.hibernate.persister.entity.AbstractEntityPersister.hydrate(AbstractEntityPersister.java:2283)  
    at org.hibernate.loader.Loader.loadFromResultSet(Loader.java:1527)  
    at org.hibernate.loader.Loader.instanceNotYetLoaded(Loader.java:1455)  
    at org.hibernate.loader.Loader.getRow(Loader.java:1355)  
    at org.hibernate.loader.Loader.getRowFromResultSet(Loader.java:611)  
    at org.hibernate.loader.Loader.doQuery(Loader.java:829)  
    at org.hibernate.loader.Loader.doQueryAndInitializeNonLazyCollections(Loader.java:274)  
    at org.hibernate.loader.Loader.doList(Loader.java:2542)  
    at org.hibernate.loader.Loader.listIgnoreQueryCache(Loader.java:2276)  
    at org.hibernate.loader.Loader.list(Loader.java:2271)  
    at org.hibernate.loader.hql.QueryLoader.list(QueryLoader.java:459)  
    at org.hibernate.hql.ast.QueryTranslatorImpl.list(QueryTranslatorImpl.java:365)  
    at org.hibernate.engine.query.HQLQueryPlan.performList(HQLQueryPlan.java:196)  
    at org.hibernate.impl.SessionImpl.list(SessionImpl.java:1268)  
    at org.hibernate.impl.QueryImpl.list(QueryImpl.java:102) 

``` 
原因是数据库里面有char类型的字段其内容为空格,而非null。在数据类型映射时发生了错误
解决：
```

update mytable set my_column = null where my_column=''

```
参考：http://stackoverflow.com/questions/24380846/hibernate-sql-exception-java-lang-stringindexoutofboundsexception-string-index
http://www.lai18.com/content/669829.html
http://stackoverflow.com/questions/8728362/spring-hibernate-java-lang-stringindexoutofboundsexception-string-index-out-o