title: spring_cache使用 

#  Spring Cache使用 
缓存简介
缓存，工作机制是：先从缓存中读取数据，如果没有再从慢速设备上读取实际数据（数据也会存入缓存）；
缓存什么：那些**经常读取且不经常修改的数据/那些昂贵（CPU/IO）的且对于相同的请求有相同的计算结果的数据**。如CPU--L1/L2--内存--磁盘就是一个典型的例子，CPU需要数据时先从L1/L2中读取，如果没有到内存中找，如果还没有会到磁盘上找。还有如用过Maven的朋友都应该知道，我们找依赖的时候，先从本机仓库找，再从本地服务器仓库找，最后到远程仓库服务器找；还有如京东的物流为什么那么快？他们在各个地都有分仓库，如果该仓库有货物那么送货的速度是非常快的。
 
缓存命中率
即从缓存中读取数据的次数 与 总读取次数的比率，命中率越高越好：
命中率 = 从缓存中读取次数 / (总读取次数[从缓存中读取次数 + 从慢速设备上读取的次数])
Miss率 = 没有从缓存中读取的次数 / (总读取次数[从缓存中读取次数 + 从慢速设备上读取的次数])
 
这是一个非常重要的监控指标，如果做缓存一定要健康这个指标来看缓存是否工作良好；
 
缓存策略Eviction policy
移除策略，即如果缓存满了，从缓存中移除数据的策略；常见的有LFU、LRU、FIFO：
FIFO（First In First Out）：先进先出算法，即先放入缓存的先被移除；
LRU（Least Recently Used）：最久未使用算法，使用时间距离现在最久的那个被移除；
LFU（Least Frequently Used）：最近最少使用算法，一定时间段内使用次数（频率）最少的那个被移除；

TTL（Time To Live ）
存活期，即从缓存中创建时间点开始直到它到期的一个时间段（不管在这个时间段内有没有访问都将过期）
TTI（Time To Idle）
空闲期，即一个数据多久没被访问将从缓存中移除的时间。
 
 
在Java中，我们一般对调用方法进行缓存控制，比如我调用"findUserById(Long id)"，那么我应该在调用这个方法之前先从缓存中查找有没有，如果没有再掉该方法如从数据库加载用户，然后添加到缓存中，下次调用时将会从缓存中获取到数据。

**自Spring 3.1起，提供了类似于@Transactional注解事务的注解Cache支持，且提供了Cache抽象；在此之前一般通过AOP实现；使用Spring Cache的好处：**
  * ` 提供基本的Cache抽象，方便切换各种底层Cache； `
  * 通过注解Cache可以实现类似于事务一样，缓存逻辑透明的应用到我们的业务代码上，且只需要更少的代码就可以完成；
  * 提供事务回滚时也自动回滚缓存；
  * 支持比较复杂的缓存逻辑；
 
对于Spring Cache抽象，主要从以下几个方面学习：
  * Cache API及默认提供的实现
  * Cache注解
  * 实现复杂的Cache逻辑
 ```

 <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>4.2.3.RELEASE</version>
        </dependency>
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
Cache API及默认提供的实现
Spring提供的核心Cache接口： 
```

package org.springframework.cache;  
  
public interface Cache {  
    String getName();      //缓存实例的名字，可以把数据写到多个缓存 ，一般在配置文件中配好缓存名字，在这里使用。 
    Object getNativeCache(); //得到底层使用的缓存，如Ehcache  
    ValueWrapper get(Object key); //根据key得到一个ValueWrapper，然后调用其get方法获取值  
    <T> T get(Object key, Class<T> type);//根据key，和value的类型直接获取value  
    void put(Object key, Object value);//往缓存放数据  
    void evict(Object key);//从缓存中移除key对应的缓存  
    void clear(); //清空缓存  
  
    interface ValueWrapper { //缓存值的Wrapper  
        Object get(); //得到真实的value  
        }  
}  

```
**默认提供了如下实现：**
  * ConcurrentMapCache：使用java.util.concurrent.ConcurrentHashMap实现的Cache；
  * GuavaCache：对Guava com.google.common.cache.Cache进行的Wrapper，需要Google Guava 12.0或更高版本，@since spring 4；
  * EhCacheCache：使用Ehcache实现
  * JCacheCache：对javax.cache.Cache进行的wrapper，@since spring 3.2；spring4将此类更新到JCache 0.11版本；
另外，**因为我们在应用中并不是使用一个Cache，而是多个，因此Spring还提供了` CacheManager `抽象，用于缓存的管理：**
```

package org.springframework.cache;  
import java.util.Collection;  
public interface CacheManager {  
    Cache getCache(String name); //根据Cache名字获取Cache   
    Collection<String> getCacheNames(); //得到所有Cache的名字  
} 

```
默认提供的实现： 
  * ConcurrentMapCacheManager/ConcurrentMapCacheFactoryBean：管理ConcurrentMapCache；
  * GuavaCacheManager；
  * EhCacheCacheManager/EhCacheManagerFactoryBean；
  * JCacheCacheManager/JCacheManagerFactoryBean；
另外还提供了` CompositeCacheManager `用于**组合CacheManager**，即可以**从多个CacheManager中轮询得到相应的Cache**，如
```

<bean id="cacheManager" class="org.springframework.cache.support.CompositeCacheManager">  
    <property name="cacheManagers">  
        <list>  
            <ref bean="ehcacheManager"/>  
            <ref bean="jcacheManager"/>  
        </list>  
    </property>  
    <property name="fallbackToNoOpCache" value="true"/>  
</bean>  
<cache:annotation-driven cache-manager="cacheManager" proxy-target-class="true"/> 

```

当我们调用cacheManager.getCache(cacheName) 时，会先从第一个cacheManager中查找有没有cacheName的cache，如果没有接着查找第二个，如果最后找不到，因为fallbackToNoOpCache=true，那么将返回一个NOP的Cache否则返回null。
除了GuavaCacheManager之外，其他Cache都支持Spring事务的，即如果事务回滚了，Cache的数据也会移除掉。
Spring不进行Cache的缓存策略的维护，这些都是由底层Cache自己实现，Spring只是提供了一个Wrapper，提供一套对外一致的API。

示例：https://github.com/zhangkaitao/spring4-showcase/tree/master/spring-cache
```

<bean id="ehcacheManager" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">  
    <property name="configLocation" value="classpath:ehcache.xml"/>  
</bean>  
  
<bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager">  
    <property name="cacheManager" ref="ehcacheManager"/>  
    <property name="transactionAware" value="true"/>  
</bean> 

```
spring提供` EhCacheManagerFactoryBean `来简化ehcache cacheManager的创建，这样注入configLocation，会自动根据路径从classpath下找，比编码方式简单多了，然后就可以从spring容器获取cacheManager进行操作了。此处的` transactionAware表示是否事务环绕的，如果true，则如果事务回滚，缓存也回滚，默认false。 `


ehcache.xml配置：
```

<?xml version="1.0" encoding="UTF-8"?>
<ehcache name="es">

    <diskStore path="java.io.tmpdir"/>

    <defaultCache
            maxEntriesLocalHeap="1000"
            eternal="false"
            timeToIdleSeconds="3600"
            timeToLiveSeconds="3600"
            overflowToDisk="false">
    </defaultCache>

    <cache name="user"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="600"
           timeToLiveSeconds="600"
           overflowToDisk="false"
           statistics="true">
    </cache>
</ehcache>

```
##  Cache注解 
```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:cache="http://www.springframework.org/schema/cache"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
     http://www.springframework.org/schema/cache
     http://www.springframework.org/schema/cache/spring-cache.xsd">
 
</beans>

```
启用Cache注解。XML风格的（spring-cache.xml）：
```

<cache:annotation-driven cache-manager="cacheManager" proxy-target-class="true"/> 

``` 
###   <cache:annotation-driven/> 
 <cache:annotation-driven/>有一个cache-manager属性用来指定当前所使用的CacheManager对应的bean的名称，默认是cacheManager，所以当我们的CacheManager的id为cacheManager时我们可以不指定该参数，否则就需要我们指定了。
<cache:annotation-driven/>还可以指定一个
**mode属性**，` 可选值有proxy和aspectj `。**` 默认是使用proxy。当mode为proxy时，只有缓存方法在外部被调用的时候Spring Cache才会发生作用，这也就意味着如果一个缓存方法在其声明对象内部被调用时Spring Cache是不会发生作用的。 `**
而mode为aspectj时就不会有这种问题。
另外使用proxy时，只有public方法上的@Cacheable等标注才会起作用，如果需要非public方法上的方法也可以使用Spring Cache时把mode设置为aspectj。
此外，<cache:annotation-driven/>还可以指定一个proxy-target-class属性，表示是否要代理class，默认为false。我们前面提到的@Cacheable、@cacheEvict等也可以标注在接口上，这对于基于接口的代理来说是没有什么问题的，**但是需要注意的是当我们设置proxy-target-class为true或者mode为aspectj时，是直接基于class进行操作的，定义在接口上的@Cacheable等Cache注解不会被识别到**，那对应的Spring Cache也不会起作用了。
 需要注意的是<cache:annotation-driven/>**只会去寻找定义在同一个ApplicationContext下的@Cacheable等缓存注解。**

另外还可以指定一个 key-generator，即默认的key生成策略，` 从spring4开始默认的keyGenerator是SimpleKeyGenerator `；后边讨论；

###  @Cacheable 

应用到读取数据的方法上，即可缓存的方法，如查找方法：先从缓存中读取，` 如果没有再调用方法获取数据 `，然后把数据添加到缓存中：
**` 需要注意的是当一个支持缓存的方法在对象内部被调用时(即通过this.方法名调用)是不会触发缓存功能的。 `**
@Cacheable可以标记在一个方法上，也可以标记在一个类上。当标记在一个方法上时表示该方法是支持缓存的，当标记在一个类上时则表示该类所有的方法都是支持缓存的。对于一个支持缓存的方法，Spring会在其被调用后将其返回值缓存起来，以保证下次利用同样的参数来执行该方法时可以直接从缓存中获取结果，**而不需要再次执行该方法。**
@Cacheable可以指定三个属性，value、key和condition。 
` value属性是必须指定的，其表示当前方法的返回值是会被缓存在哪个Cache上的， `对应Cache的名称。其可以是一个Cache也可以是多个Cache，当需要指定多个Cache时其是一个数组。
```

   @Cacheable("cache1")//Cache是发生在cache1上的
   public User find(Integer id) {
      returnnull;
   }
 
   @Cacheable({"cache1", "cache2"})//Cache是发生在cache1和cache2上的
   public User find(Integer id) {
      returnnull;
   }

```

```

@Cacheable(value = "user", key = "#id")  
 public User findById(final Long id) {  
     System.out.println("cache miss, invoke find by id, id:" + id);  
     for (User user : users) {  
         if (user.getId().equals(id)) {  
             return user;  
         }  
     }  
     return null;  
 }

```  

###  @CachePut 
` 每次都会执行方法 `，并将结果存入指定的缓存中
在支持Spring Cache的环境下，对于使用@Cacheable标注的方法，Spring在每次执行前都会检查Cache中是否存在相同key的缓存元素，如果存在就不再执行该方法，而是直接从缓存中获取结果进行返回，否则才会执行并将返回结果存入指定的缓存中。@CachePut也可以声明一个方法支持缓存功能。**与@Cacheable不同的是使用@CachePut标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。**
@CachePut也可以标注在类上和方法上。使用@CachePut时我们可以指定的属性跟@Cacheable是一样的。

应用到写数据的方法上，如新增/修改方法，调用方法时会自动把相应的数据放入缓存： 
```

@CachePut(value = "user", key = "#user.id")   //使用方法参数时我们可以直接使用“#参数名”或者“#p参数index（如#p0）”。
public User save(User user) {  
    users.add(user);  
    return user;  
} 

``` 
即调用该方法时，会把user.id作为key，返回值作为value放入缓存；
@CachePut注解：
```

public @interface CachePut {  
    String[] value();              //缓存实例的名字，可以把数据写到多个缓存 ，一般在ehcache.xml中配好缓存名字，在这里使用。 
    String key() default "";       //缓存key，如果不指定将使用默认的KeyGenerator生成，后边介绍  
    String condition() default ""; //满足缓存条件的数据才会放入缓存，condition在调用方法之前和之后都会判断  
    String unless() default "";    //用于否决缓存更新的，不像condition，该表达只在方法执行之后判断，此时可以拿到返回值result进行判断了  
}

```  
 
###  @CacheEvict 

即应用到移除数据的方法上，如删除方法，调用方法时会从缓存中移除相应的数据：
```

@CacheEvict(value = "user", key = "#user.id") //移除指定key的数据  
public User delete(User user) {  
    users.remove(user);  
    return user;  
}  
@CacheEvict(value = "user", allEntries = true) //移除所有数据  
public void deleteAll() {  
    users.clear();  
} 

``` 
###  @CacheEvict注解： 
@CacheEvict是用来标注在需要清除缓存元素的方法或类上的。当标记在一个类上时表示其中所有的方法的执行都会触发缓存的清除操作。
@CacheEvict可以指定的属性有value、key、condition、allEntries和beforeInvocation。
其中value、key和condition的语义与@Cacheable对应的属性类似。即value表示清除操作是发生在哪些Cache上的（对应Cache的名称）；key表示需要清除的是哪个key，如未指定则会使用默认策略生成的key；condition表示清除操作发生的条件。
下面我们来介绍一下新出现的两个属性allEntries和beforeInvocation。
allEntries属性
**allEntries是boolean类型，表示是否需要清除缓存中的所有元素**。默认为false，表示不需要。当指定了allEntries为true时，Spring Cache将忽略指定的key。有的时候我们需要Cache一下清除所有的元素，这比一个一个清除元素更有效率。
```

   @CacheEvict(value="users", allEntries=true)
   public void delete(Integer id) {
      System.out.println("delete user by id: " + id);
   }

```
 beforeInvocation属性
**清除操作默认是在对应方法成功执行之后触发的**，即方法如果因为抛出异常而未能成功返回时也不会触发清除操作。**使用beforeInvocation可以改变触发清除操作的时间，当我们指定该属性值为true时，Spring会在调用该方法之前清除缓存中的指定元素。**
```

   @CacheEvict(value="users", beforeInvocation=true)
   public void delete(Integer id) {
      System.out.println("delete user by id: " + id);
   }

```
其实除了使用@CacheEvict清除缓存元素外，当我们使用Ehcache作为实现时，我们也可以配置Ehcache自身的驱除策略，其是通过Ehcache的配置文件来指定的。
```

public @interface CacheEvict {  
    String[] value();                        //请参考@CachePut  
    String key() default "";                 //请参考@CachePut  
    String condition() default "";           //请参考@CachePut  
    boolean allEntries() default false;      //是否移除所有数据  
    boolean beforeInvocation() default false;//是调用方法之前移除/还是调用之后移除  


``` 

运行流程
1、首先执行@CacheEvict（如果beforeInvocation=true且condition 通过），如果allEntries=true，则清空所有  
2、接着收集@Cacheable（如果condition 通过，且key对应的数据不在缓存），放入cachePutRequests（也就是说如果cachePutRequests为空，则数据在缓存中）  
3、如果cachePutRequests为空且没有@CachePut操作，那么将查找@Cacheable的缓存，否则result=缓存数据（也就是说只要当没有cache put请求时才会查找缓存）  
4、如果没有找到缓存，那么调用实际的API，把结果放入result  
5、如果有@CachePut操作(如果condition 通过)，那么放入cachePutRequests  
6、执行cachePutRequests，将数据写入缓存（unless为空或者unless解析结果为false）；  
7、执行@CacheEvict（如果beforeInvocation=false 且 condition 通过），如果allEntries=true，则清空所有  
流程中需要注意的就是2/3/4步：
如果有@CachePut操作，即使有@Cacheable也不会从缓存中读取；问题很明显，如果要混合多个注解使用，不能组合使用@CachePut和@Cacheable；官方说应该避免这样使用（解释是如果带条件的注解相互排除的场景）；不过个人感觉还是不要考虑这个好，让用户来决定如何使用，否则一会介绍的场景不能满足。

###  使用key属性自定义key 
键的生成策略
键的生成策略有两种，一种是默认策略，一种是自定义策略。
3.1默认策略
默认的key生成策略是通过KeyGenerator生成的，**其默认策略如下**：
n  如果方法没有参数，则使用0作为key。
n  如果只有一个参数的话则使用该参数作为key。
n  如果参数多余一个的话则使用所有参数的hashCode作为key。
如果我们需要指定自己的默认策略的话，那么我们可以实现自己的` KeyGenerator `，然后指定我们的Spring Cache使用的KeyGenerator为我们自己定义的KeyGenerator。
```

   <bean id="userKeyGenerator" class="com.xxx.cache.UserKeyGenerator"/>

```
```

@Cacheable(cacheNames="books", keyGenerator="userKeyGenerator")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)

```

key属性是用来指定Spring缓存方法的**返回结果时对应的key的。**
该属性` 支持SpringEL表达式（指的是不需要#{}的，比如获取一个字符串应该是这样"'str'"而不应该是这样"#{'str'}"） `。
当我们没有指定该属性时，Spring将使用**默认策略**生成key。我们这里先来看看自定义策略，至于默认策略会在后文单独介绍。
**自定义策略**是指我们可以通过Spring的EL表达式来指定我们的key。这里的EL表达式可以使用方法参数及它们对应的属性。` 使用方法参数时我们可以直接使用“#参数名”或者“#p参数index” `。下面是几个使用参数作为key的示例。
```

   @Cacheable(value="users", key="#id")
   public User find(Integer id) {
      returnnull;
   }
 
   @Cacheable(value="users", key="#p0")
   public User find(Integer id) {
      returnnull;
   }
 
   @Cacheable(value="users", key="#user.id")
   public User find(User user) {
      returnnull;
   }
 
   @Cacheable(value="users", key="#p0.id")
   public User find(User user) {
      returnnull;
   }


``` 
除了上述使用方法参数作为key之外，Spring还为我们提供了一个root对象可以用来生成key。通过该root对象我们可以获取到以下信息。
![](/data/dokuwiki/javaweb/pasted/20151215-145209.png)
当我们要使用root对象的属性作为key时我们也可以将“#root”省略，因为Spring默认使用的就是root对象的属性。如：
```

   @Cacheable(value={"users", "xxx"}, key="caches[1].name")
   public User find(User user) {
      returnnull;
   }

```
Spring Cache提供了一些供我们使用的SpEL上下文数据
![](/data/dokuwiki/javaweb/pasted/20151214-180932.png)


###  condition属性指定发生的条件 
有的时候我们可能并不希望缓存一个方法所有的返回结果。通过condition属性可以实现这一功能。condition属性默认为空，表示将缓存所有的调用情形。
其值是通过SpringEL表达式来指定的，当为true时表示进行缓存处理；当为false时表示不进行缓存处理，即每次调用该方法时该方法都会执行一次。如下示例表示只有当user的id为偶数时才会进行缓存。
```

   @Cacheable(value={"users"}, key="#user.id", condition="#user.id%2==0")
   public User find(User user) {
      System.out.println("find user by user " + user);
      return user;
   }

```
##  @CacheConfig指定全局Cache配置 
Spring 4.1之前需要每个方法上都指定： 
Spring 4.1时可以直接在类级别使用@CacheConfig指定： 
```

@Service  
@CacheConfig(cacheNames = {"user", "user2"})  
public class UserService {  
  
    Set<User> users = new HashSet<User>();  
  
    @CachePut(key = "#user.id")  
    public User save(User user) {  
        users.add(user);  
        return user;  
    }  
  
    @CachePut(key = "#user.id")  
    public User update(User user) {  
        users.remove(user);  
        users.add(user);  
        return user;  
    }  
  
    @CacheEvict(key = "#user.id")  
    public User delete(User user) {  
        users.remove(user);  
        return user;  
    }  
  
    @CacheEvict(allEntries = true)  
    public void deleteAll() {  
        users.clear();  
    }  
  
    @Cacheable(key = "#id")  
    public User findById(final Long id) {  
        System.out.println("cache miss, invoke find by id, id:" + id);  
        for (User user : users) {  
            if (user.getId().equals(id)) {  
                return user;  
            }  
        }  
        return null;  
    }  
}  

```

##  @Caching 
@Caching注解可以让我们在一个方法或者类上同时指定多个Spring Cache相关的注解。其拥有三个属性：cacheable、put和evict，分别用于指定@Cacheable、@CachePut和@CacheEvict。
```

   @Caching(cacheable = @Cacheable("users"), evict = { @CacheEvict("cache2"),
         @CacheEvict(value = "cache3", allEntries = true) })
   public User find(Integer id) {
      returnnull;
   }

```

##  CacheManager与Cache实例配置说明 
参考：http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#cache
###  JDK ConcurrentMap-based Cache 
```

<!-- simple cache manager -->
<bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
    <property name="caches">
        <set>
            <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="default"/>
            <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="books"/>
        </set>
    </property>
</bean>

```
###  EhCache-based Cache 
```

<bean id="cacheManager"
      class="org.springframework.cache.ehcache.EhCacheCacheManager" p:cache-manager-ref="ehcache"/>

<!-- EhCache library setup -->
<bean id="ehcache"
      class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean" p:config-location="ehcache.xml"/>

```
配置与JDK的那个配置有点不同，请细看。
` 主要是我们这里没有显示配置Cache Bean，因为这里我们会从ehcache.xml配置文件读取 `
###  Guava Cache 
```

<bean id="cacheManager" class="org.springframework.cache.guava.GuavaCacheManager">
    <property name="caches">
        <set>
            <value>default</value>
            <value>books</value>
        </set>
    </property>
</bean>

```
###  组合cache manager 
```

<bean id="cacheManager" class="org.springframework.cache.support.CompositeCacheManager">
    <property name="cacheManagers">
        <list>
            <ref bean="jdkCache"/>
            <ref bean="gemfireCache"/>
        </list>
    </property>
    <property name="fallbackToNoOpCache" value="true"/>
</bean>

```
###  使用自定义注解 
Spring允许我们在配置可缓存的方法时使用自定义的注解，前提是自定义的注解上必须使用对应的注解进行标注。如我们有如下这么一个使用@Cacheable进行标注的自定义注解。
```

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Cacheable(value="users")
public @interface MyCacheable {
 
}
       那么在我们需要缓存的方法上使用@MyCacheable进行标注也可以达到同样的效果。
   @MyCacheable
   public User findById(Integer id) {
      System.out.println("find user by id: " + id);
      User user = new User();
      user.setId(id);
      user.setName("Name" + id);
      return user;
   }

```

##  SpringAOP配置缓存切面方式 
```

<!-- the service we want to make cacheable -->
<bean id="bookService" class="x.y.service.DefaultBookService"/>

<!-- cache definitions -->
<cache:advice id="cacheAdvice" cache-manager="cacheManager">
    <cache:caching cache="books">
        <cache:cacheable method="findBook" key="#isbn"/>
        <cache:cache-evict method="loadBooks" all-entries="true"/>
    </cache:caching>
</cache:advice>

<!-- apply the cacheable behavior to all BookService interfaces -->
<aop:config>
    <aop:advisor advice-ref="cacheAdvice" pointcut="execution(* x.y.BookService.*(..))"/>
</aop:config>

<!-- cache manager definition omitted -->

```


##  注意和限制 
基于 proxy 的 spring aop 带来的内部调用问题
上面介绍过 spring cache 的原理，即它是基于动态生成的 proxy 代理机制来对方法的调用进行切面，这里关键点是对象的引用问题，如果对象的方法是内部调用（即 this 引用）而不是外部引用，则会导致 proxy 失效，那么我们的切面就失效，也就是说上面定义的各种注释包括 @Cacheable、@CachePut 和 @CacheEvict 都会失效，我们来演示一下。
清单 28. AccountService.java
```

 public Account getAccountByName2(String userName) { 
   return this.getAccountByName(userName); 
 } 

 @Cacheable(value="accountCache")// 使用了一个缓存名叫 accountCache 
 public Account getAccountByName(String userName) { 
   // 方法内部实现不考虑缓存逻辑，直接实现业务
   return getFromDB(userName); 
 }

```
上面我们定义了一个新的方法 getAccountByName2，其自身调用了 getAccountByName 方法，这个时候，发生的是内部调用（this），所以没有走 proxy，导致 spring cache 失效
```

清单 29. Main.java
 public static void main(String[] args) { 
   ApplicationContext context = new ClassPathXmlApplicationContext( 
      "spring-cache-anno.xml");// 加载 spring 配置文件
  
   AccountService s = (AccountService) context.getBean("accountServiceBean"); 
  
   s.getAccountByName2("someone"); 
   s.getAccountByName2("someone"); 
   s.getAccountByName2("someone"); 
 }
清单 30. 运行结果
 real querying db...someone 
 real querying db...someone 
 real querying db...someone

```
可见，结果是每次都查询数据库，缓存没起作用。` 要避免这个问题，就是要避免对缓存方法的内部调用，或者避免使用基于 proxy 的 AOP 模式，可以使用基于 aspectJ 的 AOP 模式来解决这个问题。 `
**非 public 方法问题**
和内部调用问题类似，非 public 方法如果想实现基于注释的缓存**，必须采用基于 AspectJ 的 AOP 机制**，这里限于篇幅不再细述。
参考：http://haohaoxuexi.iteye.com/blog/2123030
http://jinnianshilongnian.iteye.com/blog/2001040
http://jinnianshilongnian.iteye.com/blog/2105367
http://www.ibm.com/developerworks/cn/opensource/os-cn-spring-cache/
http://doc.okbase.net/yunzhu/archive/101034.html
