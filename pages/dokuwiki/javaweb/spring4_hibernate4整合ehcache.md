title: spring4_hibernate4整合ehcache 

#  Spring4+Hibernate4整合EHCACHE注解配置 
使用ehcache来提高系统的性能，现在用的非常多，在hibernate当中作为二级缓存的实现产品，可以提高查询性能。
```

<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-ehcache</artifactId>
    <version>4.2.21.Final</version>
</dependency>
 <dependency>
      <groupId>net.sf.ehcache</groupId>
      <artifactId>ehcache</artifactId>
      <version>2.10.1</version>
    </dependency>

```  
在项目的src下面添加ehcache的配置文件ehcache.xml
```

<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
	<!--
    指定一个目录：当 EHCache 把数据写到硬盘上时, 将把数据写到这个目录下.
    -->
    <diskStore path="d:\\tempDirectory"/>

    <!--
    Mandatory Default Cache configuration. These settings will be applied to caches
    created programmtically using CacheManager.add(String cacheName)
设置缓存的默认数据过期策略 
    -->
    <defaultCache
	     maxElementsInMemory="10000" ＜!-- 缓存最大数目 --＞ 
	     eternal="false"＜!-- 缓存是否持久 --＞ 
	     timeToIdleSeconds="120" ＜!-- 当缓存闲置n秒后销毁 --＞ 
	     timeToLiveSeconds="120" ＜!-- 当缓存存活n秒后销毁--＞ 
	     overflowToDisk="true"  ＜!-- 是否保存到磁盘，当系统当机时--＞  
	     maxElementsOnDisk="10000000" 
	     diskPersistent="false"
	     diskExpiryThreadIntervalSeconds="120"
	     memoryStoreEvictionPolicy="LRU"
     />
    
    <cache name="org.hibernate.cache.spi.UpdateTimestampsCache"
		   maxElementsInMemory="5000" 
	       eternal="true" 
	       overflowToDisk="true" />
	<cache name="org.hibernate.cache.internal.StandardQueryCache"
	       maxElementsInMemory="10000" 
	       eternal="false" 
	       timeToLiveSeconds="120"
	       overflowToDisk="true" />	
	
	<!--
	java文件注解查找cache方法名的策略：如果不指定java文件注解中的region="ehcache.xml中的name的属性值", 
	则使用name名为com.lysoft.bean.user.User的cache(即类的全路径名称), 如果不存在与类名匹配的cache名称, 则用 defaultCache
	如果User包含set集合, 则需要另行指定其cache
	例如User包含citySet集合, 则也需要
	添加配置到ehcache.xml中
设定具体的命名缓存的数据过期策略。每个命名缓存代表一个缓存区域
         缓存区域(region)：一个具有名称的缓存块，可以给每一个缓存块设置不同的缓存策略。
         如果没有设置任何的缓存区域，则所有被缓存的对象，都将使用默认的缓存策略。即：<defaultCache.../>
         Hibernate 在不同的缓存区域保存不同的类/集合。
            对于类而言，区域的名称是类名。如:com.atguigu.domain.Customer
            对于集合而言，区域的名称是类名加属性名。如com.atguigu.domain.Customer.orders
	-->    
    <cache name="javaClassName" maxElementsInMemory="2000" eternal="false" 
	       timeToIdleSeconds="120" timeToLiveSeconds="120"
	       overflowToDisk="true" />  
	    
</ehcache>

```
在spring 集成hibernate 的配置文件中，添加如下配置
说明一下：如果不设置“**查询缓存**”，那么hibernate只会缓存使用load()方法获得的单个持久化对象，如果想缓存使用 findall()、list()、Iterator()、createCriteria()、createQuery()等方法获得的数据结果集的话， 就需要在配置文件中设置` hibernate.cache.use_query_cache true ` 才行
```

<!-- 开启查询缓存 -->
<prop key="hibernate.cache.use_query_cache">true</prop>
<!-- 开启二级缓存 -->
<prop key="hibernate.cache.use_second_level_cache">true</prop>
<!-- 高速缓存提供程序 --> 
<!-- 由于spring也使用了Ehcache, 保证双方都使用同一个缓存管理器 -->
<prop key="hibernate.cache.region.factory_class">
     org.hibernate.cache.ehcache.SingletonEhCacheRegionFactory
</prop>
<!-- cacheManager, 指定ehcache.xml的位置  Spring也使用ehcache, 所以也需要在spring配置文件中添加ehcache的配置--> 
	<bean id="cacheManagerEhcache" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
		<property name="configLocation">
        	<value>classpath:ehcache.xml</value>
        </property>
        <!-- 由于hibernate也使用了Ehcache, 保证双方都使用同一个缓存管理器 -->
        <property name="shared" value="true"/>
    </bean>

```

在类中定义:这样才能知道哪些实体类对象需要缓存
```

@Entity  
@Table(name = "t_user")  
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE, region="javaClassName")  
public class User implements Serializable {  

}

```
参考：http://www.cnphp6.com/archives/63475
http://my.oschina.net/u/1453975/blog/340463