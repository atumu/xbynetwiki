title: memcached与spring_cache整合 

#  memcached与spring提供的cache接口整合 
使用Xmemcached客户端：
https://code.google.com/p/xmemcached/wiki/User_Guide_zh
gitHub:https://github.com/killme2008/xmemcached
```

<dependency>
    <groupId>com.googlecode.xmemcached</groupId>
    <artifactId>xmemcached</artifactId>
    <version>2.0.0</version>
</dependency>

```
spring 从3.x就提供了cache接口，spring默认实现的缓存是ehcache，spring的cache接口：
```

public interface Cache {
	String getName();
	Object getNativeCache();
	ValueWrapper get(Object key);
	<T> T get(Object key, Class<T> type);
	void put(Object key, Object value);
	void evict(Object key);
	void clear();
	interface ValueWrapper {
		Object get();
	}
}

```
从spring的cache接口，我们可以看出**spring的cache模型就是在内存中画一片内存区域出来，为这盘内存指定名称，在这盘内存区域中以key-value的方式存放数据**，画个图来说明cache的存取方式，用户和部门的缓存，名称分别为DepartCache和UserCache
![](/data/dokuwiki/javaweb/pasted/20151215-154540.png)
##  实现cacheManager 
1. 要使用spring的cache，首先要实现cacheManager， 可以参考spring对ehcache的实现，整合memcached的代码如下：
```

public class MemcachedCacheManager extends AbstractTransactionSupportingCacheManager {

	private ConcurrentMap<String, Cache> cacheMap = new ConcurrentHashMap<String, Cache>();
	private Map<String, Integer> expireMap = new HashMap<String, Integer>();   //缓存的时间
	private MemcachedClient memcachedClient;   //xmemcached的客户端
	public MemcachedCacheManager() {
	}
	@Override
	protected Collection<? extends Cache> loadCaches() {
		Collection<Cache> values = cacheMap.values();
		return values;
	}
	@Override
	public Cache getCache(String name) {
		Cache cache = cacheMap.get(name);
		if (cache == null) {
			Integer expire = expireMap.get(name);
			if (expire == null) {
				expire = 0;
				expireMap.put(name, expire);
			}
			cache = new MemcachedCache(name, expire.intValue(), memcachedClient);
			cacheMap.put(name, cache);
		}
		return cache;
	}

	public void setMemcachedClient(MemcachedClient memcachedClient) {
		this.memcachedClient = memcachedClient;
	}

	public void setConfigMap(Map<String, Integer> configMap) {
		this.expireMap = configMap;
	}

}

```
##  Cache的实现 
2. 接着看下MemcachedCache的实现，主要实现spring的cache接口
```

public class MemcachedCache implements Cache {
	private final String name;
	private final MemCache memCache;

	public MemcachedCache(String name, int expire, MemcachedClient memcachedClient) {
		this.name = name;
		this.memCache = new MemCache(name, expire, memcachedClient);
	}

	@Override
	public void clear() {
		memCache.clear();
	}

	@Override
	public void evict(Object key) {
		memCache.delete(key.toString());
	}

	@Override
	public ValueWrapper get(Object key) {
		ValueWrapper wrapper = null;
		Object value = memCache.get(key.toString());
		if (value != null) {
			wrapper = new SimpleValueWrapper(value);
		}
		return wrapper;
	}

	@Override
	public String getName() {
		return this.name;
	}

	@Override
	public MemCache getNativeCache() {
		return this.memCache;
	}

	@Override
	public void put(Object key, Object value) {
		memCache.put(key.toString(), value);
	}

	@Override
	@SuppressWarnings("unchecked")
	public <T> T get(Object key, Class<T> type) {
		Object cacheValue = this.memCache.get(key.toString());
		Object value = (cacheValue != null ? cacheValue : null);
		if (type != null && !type.isInstance(value)) {
			throw new IllegalStateException("Cached value is not of required type [" + type.getName() + "]: " + value);
		}
		return (T) value;
	}

}

```
3. **spring提供的这套缓存api其实就是底层缓存框架与应用程序之间的中间层转换**， 这里还有一个类就是**MemCache，这个类主要是对应memcached缓存的模型**，spring的缓存只是提供了一套api，具体的实现要根据底层使用的什么缓存框架
![](/data/dokuwiki/javaweb/pasted/20151215-160148.png)

MemCache.java
```

public class MemCache {
	private static Logger log = LoggerFactory.getLogger(MemCache.class);

	private Set<String> keySet = new HashSet<String>();
	private final String name;
	private final int expire;
	private final MemcachedClient memcachedClient;

	public MemCache(String name, int expire, MemcachedClient memcachedClient) {
		this.name = name;
		this.expire = expire;
		this.memcachedClient = memcachedClient;
	}

	public Object get(String key) {
		Object value = null;
		try {
			key = this.getKey(key);
			value = memcachedClient.get(key);
		} catch (TimeoutException e) {
			log.warn("获取 Memcached 缓存超时", e);
		} catch (InterruptedException e) {
			log.warn("获取 Memcached 缓存被中断", e);
		} catch (MemcachedException e) {
			log.warn("获取 Memcached 缓存错误", e);
		}
		return value;
	}

	public void put(String key, Object value) {
		if (value == null)
			return;

		try {
			key = this.getKey(key);
			memcachedClient.setWithNoReply(key, expire, value);
			keySet.add(key);
		} catch (InterruptedException e) {
			log.warn("更新 Memcached 缓存被中断", e);
		} catch (MemcachedException e) {
			log.warn("更新 Memcached 缓存错误", e);
		}
	}

	public void clear() {
		for (String key : keySet) {
			try {
				memcachedClient.deleteWithNoReply(this.getKey(key));
			} catch (InterruptedException e) {
				log.warn("删除 Memcached 缓存被中断", e);
			} catch (MemcachedException e) {
				log.warn("删除 Memcached 缓存错误", e);
			}
		}
	}

	public void delete(String key) {
		try {
			key = this.getKey(key);
			memcachedClient.deleteWithNoReply(key);
		} catch (InterruptedException e) {
			log.warn("删除 Memcached 缓存被中断", e);
		} catch (MemcachedException e) {
			log.warn("删除 Memcached 缓存错误", e);
		}
	}

	private String getKey(String key) {
		return name + "_" + key;
	}
}

```
##  4. 在spring的配置文件中使用MemcachedCacheManager 

```

<cache:annotation-driven cache-manager="cacheManager" proxy-target-class="true" />    <!-- 开启缓存 -->
<bean id="memcachedClientBuilder" class="net.rubyeye.xmemcached.XMemcachedClientBuilder">   <!-- 配置memcached的缓存服务器 -->
	<constructor-arg>
		<list>
			<bean class="java.net.InetSocketAddress">
				<constructor-arg value="localhost" />
				<constructor-arg value="11211" />
			</bean>
		</list>
	</constructor-arg>
</bean>
<bean id="memcachedClient" factory-bean="memcachedClientBuilder" factory-method="build" destroy-method="shutdown" />
<bean id="cacheManager" class="com.hqhop.framework.common.cache.memcached.MemcachedCacheManager">
	<property name="memcachedClient" ref="memcachedClient" />
	<!-- 配置缓存时间 -->
	<property name="configMap">  
            <map>  
                <entry key="typeList" value="3600" />   <!-- key缓存对象名称   value缓存过期时间 -->
            </map>  
        </property>  
</bean>

```
5. 到此为止，spring和memcached整合就完成了，spring的缓存还提供了很多的注解 @Cachable，@cachePut.....，这里不多说了。

#  通过simple-spring-memcached简化与Spring集成 
github:https://github.com/ragnor/simple-spring-memcached
wiki:http://code.google.com/p/simple-spring-memcached/wiki/Getting_Started
simple-spring-memcached组件通过与spring框架整合，让memcached的调用变得更加简单。

```

<dependency>
  <groupId>com.google.code.simple-spring-memcached</groupId>
  <artifactId>spring-cache</artifactId>
  <version>3.6.0</version>
</dependency>
<dependency>
  <groupId>com.google.code.simple-spring-memcached</groupId>
  <artifactId>xmemcached-provider</artifactId>
  <version>3.6.0</version>
</dependency>   
        <dependency>  
            <groupId>com.googlecode.xmemcached</groupId>  
            <artifactId>xmemcached</artifactId>  
            <version>2.0.0</version>  
        </dependency>  

```
```

<!-- 启用缓存注解功能，这个是必须的，否则注解不会生效，另外，该注解一定要声明在spring主配置文件中才会生效 -->  
    <cache:annotation-driven cache-manager="cacheManager" />  
       <!-- AOP 你懂的 -->
  <aop:aspectj-autoproxy />
     <bean name="cacheManager" class="com.google.code.ssm.spring.SSMCacheManager">
    <property name="caches">
      <set>
        <bean class="com.google.code.ssm.spring.SSMCache">
          <constructor-arg name="cache" index="0" ref="defaultCache" />
          <!-- 5 minutes -->
          <constructor-arg name="expiration" index="1" value="300" />
          <!-- @CacheEvict(..., "allEntries" = true) won't work because allowClear is false, 
           so we won't flush accidentally all entries from memcached instance -->
          <constructor-arg name="allowClear" index="2" value="false" />
        </bean>
      </set>
    </property>
  </bean>

  <bean name="defaultCache" class="com.google.code.ssm.CacheFactory">
    <property name="cacheName" value="default" />
    <property name="cacheClientFactory">
      <bean name="cacheClientFactory" class="com.google.code.ssm.providers.xmemcached.MemcacheClientFactoryImpl" />
    </property>
   
    <property name="addressProvider">
      <bean class="com.google.code.ssm.config.DefaultAddressProvider">
        <property name="address" value="127.0.0.1:11211" /> <!--value可以指定多个节点，以逗号分隔 -->
      </bean>
    </property>
       <!--分布式策略 一致性hash ~ -->
    <property name="configuration">
      <bean class="com.google.code.ssm.providers.CacheConfiguration">
        <property name="consistentHashing" value="true" />
        <!-- spring can produce keys that contain unacceptable characters -->
        <property name="useBinaryProtocol" value="true" />
      </bean>
    </property>
  </bean>
 <!-- 这玩意儿在3.2 后，文档可以指定顺序 以及 拦截器 前后执行 - -！暂时没用过，加上不报错
  <bean class="com.google.code.ssm.Settings">
    <property name="order" value="500" />
  </bean> -->

```
```

// cache name: default, expiration: 600s instead of default 300s
@Cacheable(value = "default#600", key = "#id")
public User getByPk(int id) {
  // ...
  return result;
}

```
参考：http://www.tuicool.com/m/articles/ENB73a
http://hanqunfeng.iteye.com/blog/2175075
http://www.tuicool.com/articles/R3YFjqi