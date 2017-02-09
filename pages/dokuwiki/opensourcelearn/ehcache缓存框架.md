title: ehcache缓存框架 

#  ehcache缓存框架 

Ehcache 是现在最流行的纯Java开源缓存框架，配置简单、结构清晰、功能强大。
EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider。
官网：http://ehcache.org/
#  一、特性一览 
1、快速轻量
过去几年，诸多测试表明Ehcache是最快的Java缓存之一。
Ehcache的线程机制是为大型高并发系统设计的。
很多用户都不知道他们正在用Ehcache，因为不需要什么特别的配置。
API易于使用，这就很容易部署上线和运行。
很小的jar包，Ehcache 2.2.3才668kb。
` 最小的依赖：唯一的依赖就是SLF4J了。 `

2、伸缩性
缓存在内存和磁盘存储可以伸缩到数G，Ehcache为大数据存储做过优化。
大内存的情况下，所有进程可以支持数百G的吞吐。
为高并发和大型多CPU服务器做优化。
线程安全和性能总是一对矛盾，Ehcache的线程机制设计采用了Doug Lea的想法来获得较高的性能。

3、灵活性
Ehcache 1.2具备对象API接口和可序列化API接口。
不能序列化的对象可以使用除磁盘存储外Ehcache的所有功能。
除了元素的返回方法以外，API都是统一的。只有这两个方法不一致：getObjectValue和getKeyValue。这就使得缓存对象、序列化对象来获取新的特性这个过程很简单。
支持基于Cache和基于Element的过期策略，每个Cache的存活时间都是可以设置和控制的。
提供了LRU、LFU和FIFO缓存淘汰算法，Ehcache 1.2引入了最少使用和先进先出缓存淘汰算法，构成了完整的缓存淘汰算法。
` 提供内存和磁盘存储 `，Ehcache和大多数缓存解决方案一样，提供高性能的内存和磁盘存储。
动态、运行时缓存配置，存活时间、空闲时间、内存和磁盘存放缓存的最大数目都是可以在运行时修改的。

4、标准支持
Ehcache提供了对JSR107 JCACHE API最完整的实现。因为JCACHE在发布以前，Ehcache的实现（如net.sf.jsr107cache）已经发布了。
实现JCACHE API有利于到未来其他缓存解决方案的可移植性。
Ehcache的维护者Greg Luck，正是JSR107的专家委员会委员。

5、可扩展性
监听器可以插件化。Ehcache 1.2提供了CacheManagerEventListener和CacheEventListener接口，实现可以插件化，并且可以在ehcache.xml里配置。
节点发现，冗余器和监听器都可以插件化。
分布式缓存，从Ehcache 1.2开始引入，包含了一些权衡的选项。Ehcache的团队相信没有什么是万能的配置。
实现者可以使用内建的机制或者完全自己实现，因为有完整的插件开发指南。
缓存的可扩展性可以插件化。创建你自己的缓存扩展，它可以持有一个缓存的引用，并且绑定在缓存的生命周期内。
缓存加载器可以插件化。创建你自己的缓存加载器，可以使用一些异步方法来加载数据到缓存里面。
缓存异常处理器可以插件化。创建一个异常处理器，在异常发生的时候，可以执行某些特定操作。

6、应用持久化
在VM重启后，持久化到磁盘的存储可以复原数据。
Ehcache是第一个引入缓存数据持久化存储的开源Java缓存框架。缓存的数据可以在机器重启后从磁盘上重新获得。
根据需要将缓存刷到磁盘。将缓存条目刷到磁盘的操作可以通过cache.flush()方法来执行，这大大方便了Ehcache的使用。

7、监听器
缓存管理器监听器。允许注册实现了CacheManagerEventListener接口的监听器：
notifyCacheAdded()
notifyCacheRemoved()
缓存事件监听器。允许注册实现了CacheEventListener接口的监听器，它提供了许多对缓存事件发生后的处理机制：
notifyElementRemoved/Put/Updated/Expired 

8、开启JMX
Ehcache的JMX功能是默认开启的，你可以监控和管理如下的MBean：
CacheManager、Cache、CacheConfiguration、CacheStatistics 

9、分布式缓存
从Ehcache 1.2开始，支持高性能的分布式缓存，兼具灵活性和扩展性。
分布式缓存的选项包括：
通过Terracotta的缓存集群：设定和使用Terracotta模式的Ehcache缓存。缓存发现是自动完成的，并且有很多选项可以用来调试缓存行为和性能。
使用RMI、JGroups或者JMS来冗余缓存数据：节点可以通过多播或发现者手动配置。状态更新可以通过RMI连接来异步或者同步完成。
Custom：一个综合的插件机制，支持发现和复制的能力。
可用的缓存复制选项。支持的通过RMI、JGroups或JMS进行的异步或同步的缓存复制。
可靠的分发：使用TCP的内建分发机制。
节点发现：节点可以手动配置或者使用多播自动发现，并且可以自动添加和移除节点。对于多播阻塞的情况下，手动配置可以很好地控制。
分布式缓存可以任意时间加入或者离开集群。缓存可以配置在初始化的时候执行引导程序员。
BootstrapCacheLoaderFactory抽象工厂，实现了BootstrapCacheLoader接口（RMI实现）。
缓存服务端。Ehcache提供了一个Cache Server，一个war包，为绝大多数web容器或者是独立的服务器提供支持。
缓存服务端有两组API：面向资源的RESTful，还有就是SOAP。客户端没有实现语言的限制。
RESTful缓存服务器：Ehcached的实现严格遵循RESTful面向资源的架构风格。
SOAP缓存服务端：Ehcache RESTFul Web Services API暴露了单例的CacheManager，他能在ehcache.xml或者IoC容器里面配置。
标准服务端包含了内嵌的Glassfish web容器。它被打成了war包，可以任意部署到支持Servlet 2.5的web容器内。Glassfish V2/3、Tomcat 6和Jetty 6都已经经过了测试。

10、搜索
标准分布式搜索使用了流式查询接口的方式，请参阅文档。

11、Java EE和应用缓存
为普通缓存场景和模式提供高质量的实现。
阻塞缓存：它的机制避免了复制进程并发操作的问题。
SelfPopulatingCache在缓存一些开销昂贵操作时显得特别有用，它是一种针对读优化的缓存。它不需要调用者知道缓存元素怎样被返回，也支持在不阻塞读的情况下刷新缓存条目。
CachingFilter：一个抽象、可扩展的cache filter。
SimplePageCachingFilter：用于缓存基于request URI和Query String的页面。它可以根据HTTP request header的值来选择采用或者不采用gzip压缩方式将页面发到浏览器端。你可以用它来缓存整个Servlet页面，无论你采用的是JSP、velocity，或者其他的页面渲染技术。
SimplePageFragmentCachingFilter：缓存页面片段，基于request URI和Query String。在JSP中使用jsp:include标签包含。
已经使用Orion和Tomcat测试过，兼容Servlet 2.3、Servlet 2.4规范。
Cacheable命令：这是一种老的命令行模式，支持异步行为、容错。
兼容Hibernate，兼容Google App Engine。
基于JTA的事务支持，支持事务资源管理，二阶段提交和回滚，以及本地事务。

#  二、Ehcache的加载模块列表 
他们都是独立的库，每个都为Ehcache添加新的功能
  * ehcache-core：API，标准缓存引擎，RMI复制和Hibernate支持
  * ehcache：分布式Ehcache，包括Ehcache的核心和Terracotta的库
  * ehcache-monitor：企业级监控和管理
  * ehcache-web：为Java Servlet Container提供缓存、gzip压缩支持的filters
  * ehcache-jcache：JSR107 JCACHE的实现
  * ehcache-jgroupsreplication：使用JGroup的复制
  * ehcache-jmsreplication：使用JMS的复制
  * ehcache-openjpa：OpenJPA插件
  * ehcache-server：war内部署或者单独部署的RESTful cache server
  * ehcache-unlockedreadsview：允许Terracotta cache的无锁读
  * ehcache-debugger：记录RMI分布式调用事件
  * Ehcache for Ruby：Jruby and Rails支持

![](/data/dokuwiki/opensourcelearn/pasted/20150512-150040.png)

#  三、核心定义： 
  * ` cache manager `：缓存管理器，以前是只允许单例的，不过现在也可以多实例了
  * ` cache `：缓存管理器内可以放置若干cache，存放数据的实质，所有cache都实现了Ehcache接口
  * ` element `：单条缓存数据的组成单位
  * system of record（SOR）：可以取到真实数据的组件，可以是真正的业务逻辑、外部接口调用、存放真实数据的数据库等等，缓存就是从SOR中读取或者写入到SOR中去的。
##  简单使用： 
安装：
` 依赖： SLF4J `
maven方式：
```

<dependency>
 <groupId>net.sf.ehcache</groupId>
 <artifactId>ehcache</artifactId>
 <version>2.10.0</version>
 <type>pom</type>
</dependency>


```
大概步骤为： 
第一步：生成CacheManager对象 (4种方式)
第二步：生成Cache对象 
第三步：向Cache对象里添加由key,value组成的键值对的Element元素 
首先介绍CacheManager类。它主要负责读取配置文件，默认读取CLASSPATH下的ehcache.xml，根据配置文件创建并管理Cache对象。
` 注意：对于缓存的对象都是必须可序列化的 `
```

CacheManager manager = CacheManager.newInstance("src/config/ehcache.xml");  
manager.addCache("testCache");  
Cache test = singletonManager.getCache("testCache");  
test.put(new Element("key1", "value1"));  
manager.shutdown();  

```
##  配置文件 
配置文件ehcache.xml中命名为demoCache的缓存配置：
```

<cache name="demoCache"
maxElementsInMemory="10000"
eternal="false"
overflowToDisk="true"
timeToIdleSeconds="300"
timeToLiveSeconds="600"
memoryStoreEvictionPolicy="LFU" />

```
###  各配置参数的含义： 

  * maxElementsInMemory：缓存中允许创建的最大对象数
  * eternal：缓存中对象是否为永久的，如果是，超时设置将被忽略，对象从不过期。
  * timeToIdleSeconds：缓存数据的钝化时间，也就是在一个元素消亡之前，两次访问时间的最大时间间隔值，这只能在元素不是永久驻留时有效，如果该值是 0 就意味着元素可以停顿无穷长的时间。
  * timeToLiveSeconds：缓存数据的生存时间，也就是一个元素从构建到消亡的最大时间间隔值，这只能在元素不是永久驻留时有效，如果该值是0就意味着元素可以停顿无穷长的时间。
  * overflowToDisk：内存不足时，是否启用磁盘缓存。
  * memoryStoreEvictionPolicy：缓存满了之后的淘汰算法。LRU和FIFO算法这里就不做介绍。LFU算法直接淘汰使用比较少的对象，在内存保留的都是一些经常访问的对象。对于大部分网站项目，该算法比较适用。
 如果应用需要配置多个不同命名并采用不同参数的Cache，可以相应修改配置文件，增加需要的Cache配置即可。
##  代码方式配置： 

```

Cache testCache = new Cache(  
  new CacheConfiguration("testCache", maxElements)  
    .memoryStoreEvictionPolicy(MemoryStoreEvictionPolicy.LFU)  
    .overflowToDisk(true)  
    .eternal(false)  
    .timeToLiveSeconds(60)  
    .timeToIdleSeconds(30)  
    .diskPersistent(false)  
    .diskExpiryThreadIntervalSeconds(0));  

```
事务示例：
```

Ehcache cache = cacheManager.getEhcache("xaCache");  
transactionManager.begin();  
try {  
    Element e = cache.get(key);  
    Object result = complexService.doStuff(element.getValue());  
    cache.put(new Element(key, result));  
    complexService.doMoreStuff(result);  
    transactionManager.commit();  
} catch (Exception e) {  
    transactionManager.rollback();  
}  


```
#  利用Spring APO整合EHCache 
首先，在CLASSPATH下面放置ehcache.xml配置文件。在Spring的配置文件中先添加如下cacheManager配置：
```

<bean
 id="cacheManager"
class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
</bean>
配置demoCache：
<bean
 id="demoCache" class="org.springframework.cache.ehcache.EhCacheFactoryBean">
<property
 name="cacheManager" ref="cacheManager" />
<property
 name="cacheName">
<value>demoCache</value>
</property>
</bean>

```
接 下来，写一个实现org.aopalliance.intercept.MethodInterceptor接口的拦截器类。有了拦截器就可以有选择性的 配置想要缓存的 bean 方法。如果被调用的方法配置为可缓存，拦截器将为该方法生成 cache key 并检查该方法返回的结果是否已缓存。如果已缓存，就返回缓存的结果，否则再次执行被拦截的方法，并缓存结果供下次调用。具体代码如下：
```

  public class MethodCacheInterceptor
implements MethodInterceptor,
InitializingBean
 {
private Cache
 cache;
   
public void setCache(Cache
 cache) {
this.cache
 = cache;
}
   
public void afterPropertiesSet()
throws Exception
 {
Assert.notNull(cache,
"A
 cache is required. Use setCache(Cache) to provide one.");
}
   
public Object
 invoke(MethodInvocation invocation) throws Throwable
 {
String
 targetName = invocation.getThis().getClass().getName();
String
 methodName = invocation.getMethod().getName();
Object[]
 arguments = invocation.getArguments();
Object
 result;
String
 cacheKey = getCacheKey(targetName, methodName, arguments);
Element
 element = null;
synchronized (this){
element
 = cache.get(cacheKey);
if (element
 == null)
 {
//调用实际的方法
result
 = invocation.proceed();
element
 = new Element(cacheKey,
 (Serializable) result);
cache.put(element);
}
}
return element.getValue();
}
   
private String
 getCacheKey(String targetName, String methodName,
Object[]
 arguments) {
StringBuffer
 sb = new StringBuffer();
sb.append(targetName).append(".").append(methodName);
if ((arguments
 != null)
 && (arguments.length != 0))
 {
for (int i
 = 0;
 i < arguments.length; i++) {
sb.append(".").append(arguments[i]);
}
}
return sb.toString();
}
}

```
synchronized (this)这段代码实现了同步功能。为什么一定要同步？Cache对象本身的get和put操作是同步的。如果我们缓存的数据来自数据库查询，在没有这 段同步代码时，当key不存在或者key对应的对象已经过期时，在多线程并发访问的情况下，许多线程都会重新执行该方法，由于对数据库进行重新查询代价是 比较昂贵的，而在瞬间大量的并发查询，会对数据库服务器造成非常大的压力。所以这里的同步代码是很重要的。
接下来，继续完成拦截器和Bean的配置：
```

<bean id="methodCacheInterceptor" class="com.xiebing.utils.interceptor.MethodCacheInterceptor">
<property name="cache">
<ref local="demoCache" />
</property>
</bean>
<bean id="methodCachePointCut" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
<property name="advice">
<ref local="methodCacheInterceptor" />
</property>
<property name="patterns">
<list>
<value>.*myMethod</value>
</list>
</property>
</bean>

<bean id="myServiceBean"
class="com.xiebing.ehcache.spring.MyServiceBean">
</bean>
<bean id="myService" class="org.springframework.aop.framework.ProxyFactoryBean">
<property name="target">
<ref local="myServiceBean" />
</property>
<property name="interceptorNames">
<list>
<value>methodCachePointCut</value>
</list>
</property>
</bean>

```
其 中myServiceBean是实现了业务逻辑的Bean，里面的方法myMethod()的返回结果需要被缓存。这样每次对myServiceBean 的myMethod()方法进行调用,都会首先从缓存中查找,其次才会查询数据库。使用AOP的方式极大地提高了系统的灵活性，通过修改配置文件就可以实 现对方法结果的缓存，所有的对Cache的操作都封装在了拦截器的实现中。

##  CachingFilter功能 
使用Spring的AOP进行整合，可以灵活的对方法的的返回结果对象进行缓存。CachingFilter功能可以对HTTP响应的内容进行缓存。这种方式缓存数据的粒度比较粗，例如缓存整张页面。它的优点是使用简单、效率高，缺点是不够灵活，可重用程度不高。
EHCache 使用SimplePageCachingFilter类实现Filter缓存。该类继承自CachingFilter，有默认产生cache key的calculateKey()方法，该方法使用HTTP请求的URI和查询条件来组成key。也可以自己实现一个Filter，同样继承 CachingFilter类,然后覆写calculateKey()方法，生成自定义的key。
在笔者参与的项目中很多页面都使用AJAX，为 保证JS请求的数据不被浏览器缓存，每次请求都会带有一个随机数参数i。如果使用SimplePageCachingFilter，那么每次生成的key 都不一样，缓存就没有意义了。这种情况下，我们就会覆写calculateKey()方法。
要使用SimplePageCachingFilter，首先在配置文件ehcache.xml中，增加下面的配置：
```

<cache name="SimplePageCachingFilter" maxElementsInMemory="10000" eternal="false"
overflowToDisk="false" timeToIdleSeconds="300" timeToLiveSeconds="600"
memoryStoreEvictionPolicy="LFU" />
其中name属性必须为SimplePageCachingFilter，修改web.xml文件，增加一个Filter的配置：
<filter>
<filter-name>SimplePageCachingFilter</filter-name>
<filter-class>net.sf.ehcache.constructs.web.filter.SimplePageCachingFilter</filter-class>
</filter>
<filter-mapping>
<filter-name>SimplePageCachingFilter</filter-name>
<url-pattern>/test.jsp</url-pattern>
</filter-mapping>

```

下面我们写一个简单的test.jsp文件进行测试，缓存后的页面每次刷新，在600秒内显示的时间都不会发生变化的。代码如下：
```

<%
out.println(new Date());
%>

```
CachingFilter输出的数据会根据浏览器发送的Accept-Encoding头信息进行Gzip压缩。经过笔者测试，Gzip压缩后的数据量是原来的1/4，速度是原来的4-5倍，所以缓存加上压缩，效果非常明显。
在使用Gzip压缩时，需注意两个问题：
1. Filter在进行Gzip压缩时，采用系统默认编码，对于使用GBK编码的中文网页来说，需要将操作系统的语言设置为：zh_CN.GBK，否则会出现乱码的问题。
2. 默 认情况下CachingFilter会根据浏览器发送的请求头部所包含的Accept-Encoding参数值来判断是否进行Gzip压缩。虽然 IE6/7浏览器是支持Gzip压缩的，但是在发送请求的时候却不带该参数。为了对IE6/7也能进行Gzip压缩，可以通过继承 CachingFilter，实现自己的Filter，然后在具体的实现中覆写方法acceptsGzipEncoding。

具体实现参考：
```

protected boolean acceptsGzipEncoding(HttpServletRequest
 request) {
final boolean ie6
 = headerContains(request, "User-Agent",
"MSIE
 6.0");
final boolean ie7
 = headerContains(request, "User-Agent",
"MSIE
 7.0");
return acceptsEncoding(request,
"gzip")
 || ie6 || ie7;
}

```
#  EHCache在Hibernate中的使用 

EHCache可以作为Hibernate的二级缓存使用。在hibernate.cfg.xml中需增加如下设置：
```

<prop key="hibernate.cache.provider_class">
org.hibernate.cache.EhCacheProvider
</prop>


```
然后在Hibernate映射文件的每个需要Cache的Domain中，加入类似如下格式信息：
<cache usage="read-write|nonstrict-read-write|read-only" />
比如：
```

<cache usage="read-write" />
最后在配置文件ehcache.xml中增加一段cache的配置，其中name为该domain的类名。
<cache name="domain.class.name"
maxElementsInMemory="10000"
eternal="false"
timeToIdleSeconds="300"
timeToLiveSeconds="600"
overflowToDisk="false"
/>

```
#  EHCache的监控 

对 于Cache的使用，除了功能，在实际的系统运营过程中，我们会比较关注每个Cache对象占用的内存大小和Cache的命中率。有了这些数据，我们就可 以对Cache的配置参数和系统的配置参数进行优化，使系统的性能达到最优。EHCache提供了方便的API供我们调用以获取监控数据，其中主要的方法 有：
```

//得到缓存中的对象数
cache.getSize();
//得到缓存对象占用内存的大小
cache.getMemoryStoreSize();
//得到缓存读取的命中次数
cache.getStatistics().getCacheHits()
//得到缓存读取的错失次数
cache.getStatistics().getCacheMisses()

```
#  分布式缓存 

EHCache从1.2版本开始支持分布式缓存。分布式缓存主要解决集群环境中不同的服务器间的数据的同步问题。具体的配置如下：
在配置文件ehcache.xml中加入
```

<cacheManagerPeerProviderFactory
class="net.sf.ehcache.distribution.RMICacheManagerPeerProviderFactory"
properties="peerDiscovery=automatic,
 multicastGroupAddress=230.0.0.1, multicastGroupPort=4446"/>
<cacheManagerPeerListenerFactory
class="net.sf.ehcache.distribution.RMICacheManagerPeerListenerFactory"/>

```
另外，需要在每个cache属性中加入
```

<cacheEventListenerFactory class="net.sf.ehcache.distribution.RMICacheReplicatorFactory"/>
例如：
<cache name="demoCache"
maxElementsInMemory="10000"
eternal="true"
overflowToDisk="true">
<cacheEventListenerFactory class="net.sf.ehcache.distribution.RMICacheReplicatorFactory"/>
</cache>

总结：
EHCache 是一个非常优秀的基于Java的Cache实现。它简单、易用，而且功能齐全，并且非常容易与Spring、Hibernate等流行的开源框架进行整 合。通过使用EHCache可以减少网站项目中数据库服务器的访问压力，提高网站的访问速度，改善用户的体验。

```

![](/data/dokuwiki/opensourcelearn/ehcache学习文档.zip|附上encache学习文档：)
参考：
http://raychase.iteye.com/blog/1545906
http://m.blog.csdn.net/blog/yhc13429826359/39051719
http://www.open-open.com/lib/view/open1322892011546.html