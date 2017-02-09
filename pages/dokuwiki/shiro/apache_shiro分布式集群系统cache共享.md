title: apache_shiro分布式集群系统cache共享 

#  Apache shiro分布式集群系统下cache共享 
非集群，单机配置如下：
```

<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager" depends-on="userRepository,roleRepository">    
    <property name="sessionManager" ref="defaultWebSessionManager" />    
    <property name="realm" ref="shiroDbRealm" />    
    <property name="cacheManager" ref="memoryConstrainedCacheManager" />    
</bean>    
<bean id="memoryConstrainedCacheManager" class="org.apache.shiro.cache.MemoryConstrainedCacheManager" />

```
这里cacheManager我们注入了shiro自定的本机内存实现的**cacheManager类**，**当然，这肯定不满足我们集群的需要，所以我们要自己实现cacheManager类，这里我还是用了redis作为cache的存储，先创建RedisCacheManager实现类**

```

import java.util.concurrent.ConcurrentHashMap;  
import java.util.concurrent.ConcurrentMap;  
  
import org.apache.shiro.cache.Cache;  
import org.apache.shiro.cache.CacheException;  
import org.apache.shiro.cache.CacheManager;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
  
public class RedisCacheManager implements CacheManager{  
  
    private static final Logger logger = LoggerFactory  
            .getLogger(RedisCacheManager.class);  
  
    // fast lookup by name map  
    private final ConcurrentMap<String, Cache> caches = new ConcurrentHashMap<String, Cache>();  
  
    private RedisManager redisManager;  
  
    /** 
     * The Redis key prefix for caches  
     */  
    private String keyPrefix = "shiro_redis_cache:";  
      
    /** 
     * Returns the Redis session keys 
     * prefix. 
     * @return The prefix 
     */  
    public String getKeyPrefix() {  
        return keyPrefix;  
    }  
  
    /** 
     * Sets the Redis sessions key  
     * prefix. 
     * @param keyPrefix The prefix 
     */  
    public void setKeyPrefix(String keyPrefix) {  
        this.keyPrefix = keyPrefix;  
    }  
      
    @Override  
    public <K, V> Cache<K, V> getCache(String name) throws CacheException {  
        logger.debug("获取名称为: " + name + " 的RedisCache实例");  
          
        Cache c = caches.get(name);  
          
        if (c == null) {  
  
            // initialize the Redis manager instance  
            redisManager.init();  
              
            // create a new cache instance  
            c = new RedisCache<K, V>(redisManager, keyPrefix);  
              
            // add it to the cache collection  
            caches.put(name, c);  
        }  
        return c;  
    }  
  
    public RedisManager getRedisManager() {  
        return redisManager;  
    }  
  
    public void setRedisManager(RedisManager redisManager) {  
        this.redisManager = redisManager;  
    }  
}  

```

这个是自己` relm `的AuthorizingRealm中的方法
```

protected AuthorizationInfo getAuthorizationInfo(PrincipalCollection principals) {       
            if (principals == null) {    
                return null;    
            }       
            AuthorizationInfo info = null;    
            if (log.isTraceEnabled()) {    
                log.trace("Retrieving AuthorizationInfo for principals [" + principals + "]");    
            }    
            Cache<Object, AuthorizationInfo> cache = getAvailableAuthorizationCache();    
            if (cache != null) {    
                if (log.isTraceEnabled()) {    
                    log.trace("Attempting to retrieve the AuthorizationInfo from cache.");    
                }    
                Object key = getAuthorizationCacheKey(principals);    
                info = cache.get(key);    
                if (log.isTraceEnabled()) {    
                    if (info == null) {    
                        log.trace("No AuthorizationInfo found in cache for principals [" + principals + "]");    
                    } else {    
                        log.trace("AuthorizationInfo found in cache for principals [" + principals + "]");    
                    }    
                }    
            }    
            if (info == null) {    
                // Call template method if the info was not found in a cache     
                info = doGetAuthorizationInfo(principals);    //注意这一行
                // If the info is not null and the cache has been created, then cache the authorization info.     
                if (info != null && cache != null) {    
                    if (log.isTraceEnabled()) {    
                        log.trace("Caching authorization info for principals: [" + principals + "].");    
                    }    
                    Object key = getAuthorizationCacheKey(principals);    
                    cache.put(key, info);    //注意这一行
                }    
            }    
            return info;    
        }

``` 
我们自定义relm要实现一个叫做doGetAuthorizationInfo()的方法，它的作用是查询授权信息，我们注意加粗的2行，发现其实cache的put方法其实才是cache存储的核心类，其实现都在cache中，
所以**我们需要实现自己的cache，创建RedisCache类**
```

import org.apache.shiro.cache.Cache;  
import org.apache.shiro.cache.CacheException;  
import org.apache.shiro.util.CollectionUtils;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
  
public class RedisCache<K,V> implements Cache<K,V> {  
  
    private Logger logger = LoggerFactory.getLogger(this.getClass());  
      
    /** 
     * The wrapped Jedis instance. 
     */  
    private RedisManager cache;  
      
    /** 
     * The Redis key prefix for the sessions  
     */  
    private String keyPrefix = "shiro_redis_session:";  
      
    /** 
     * Returns the Redis session keys 
     * prefix. 
     * @return The prefix 
     */  
    public String getKeyPrefix() {  
        return keyPrefix;  
    }  
  
    /** 
     * Sets the Redis sessions key  
     * prefix. 
     * @param keyPrefix The prefix 
     */  
    public void setKeyPrefix(String keyPrefix) {  
        this.keyPrefix = keyPrefix;  
    }  
      
    /** 
     * 通过一个JedisManager实例构造RedisCache 
     */  
    public RedisCache(RedisManager cache){  
         if (cache == null) {  
             throw new IllegalArgumentException("Cache argument cannot be null.");  
         }  
         this.cache = cache;  
    }  
      
    /** 
     * Constructs a cache instance with the specified 
     * Redis manager and using a custom key prefix. 
     * @param cache The cache manager instance 
     * @param prefix The Redis key prefix 
     */  
    public RedisCache(RedisManager cache,   
                String prefix){  
           
        this( cache );  
          
        // set the prefix  
        this.keyPrefix = prefix;  
    }  
      
    /** 
     * 获得byte[]型的key 
     * @param key 
     * @return 
     */  
    private byte[] getByteKey(K key){  
        if(key instanceof String){  
            String preKey = this.keyPrefix + key;  
            return preKey.getBytes();  
        }else{  
            return SerializeUtils.serialize(key);  
        }  
    }  
      
    @Override  
    public V get(K key) throws CacheException {  
        logger.debug("根据key从Redis中获取对象 key [" + key + "]");  
        try {  
            if (key == null) {  
                return null;  
            }else{  
                byte[] rawValue = cache.get(getByteKey(key));  
                @SuppressWarnings("unchecked")  
                V value = (V)SerializeUtils.deserialize(rawValue);  
                return value;  
            }  
        } catch (Throwable t) {  
            throw new CacheException(t);  
        }  
  
    }  
  
    @Override  
    public V put(K key, V value) throws CacheException {  
        logger.debug("根据key从存储 key [" + key + "]");  
         try {  
                cache.set(getByteKey(key), SerializeUtils.serialize(value));  
                return value;  
            } catch (Throwable t) {  
                throw new CacheException(t);  
            }  
    }  
  
    @Override  
    public V remove(K key) throws CacheException {  
        logger.debug("从redis中删除 key [" + key + "]");  
        try {  
            V previous = get(key);  
            cache.del(getByteKey(key));  
            return previous;  
        } catch (Throwable t) {  
            throw new CacheException(t);  
        }  
    }  
  
    @Override  
    public void clear() throws CacheException {  
        logger.debug("从redis中删除所有元素");  
        try {  
            cache.flushDB();  
        } catch (Throwable t) {  
            throw new CacheException(t);  
        }  
    }  
  
    @Override  
    public int size() {  
        try {  
            Long longSize = new Long(cache.dbSize());  
            return longSize.intValue();  
        } catch (Throwable t) {  
            throw new CacheException(t);  
        }  
    }  
  
    @SuppressWarnings("unchecked")  
    @Override  
    public Set<K> keys() {  
        try {  
            Set<byte[]> keys = cache.keys(this.keyPrefix + "*");  
            if (CollectionUtils.isEmpty(keys)) {  
                return Collections.emptySet();  
            }else{  
                Set<K> newKeys = new HashSet<K>();  
                for(byte[] key:keys){  
                    newKeys.add((K)key);  
                }  
                return newKeys;  
            }  
        } catch (Throwable t) {  
            throw new CacheException(t);  
        }  
    }  
  
    @Override  
    public Collection<V> values() {  
        try {  
            Set<byte[]> keys = cache.keys(this.keyPrefix + "*");  
            if (!CollectionUtils.isEmpty(keys)) {  
                List<V> values = new ArrayList<V>(keys.size());  
                for (byte[] key : keys) {  
                    @SuppressWarnings("unchecked")  
                    V value = get((K)key);  
                    if (value != null) {  
                        values.add(value);  
                    }  
                }  
                return Collections.unmodifiableList(values);  
            } else {  
                return Collections.emptyList();  
            }  
        } catch (Throwable t) {  
            throw new CacheException(t);  
        }  
    }  
}  

```
最后修改spring配置文件
```

    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">  
        <property name="realm" ref="shiroRealm"></property>  
        <property name="sessionMode" value="http"></property>  
        <property name="subjectFactory" ref="casSubjectFactory"></property>  
        <!-- ehcahe缓存shiro自带 -->  
        <!-- <property name="cacheManager" ref="shiroEhcacheManager"></property> -->  
  
        <!-- redis缓存 -->  
        <property name="cacheManager" ref="redisCacheManager" />  
  
        <!-- sessionManager -->  
        <property name="sessionManager" ref="sessionManager"></property>  
    </bean>

```
这样就完成了整个shiro集群的配置
参考：http://blog.csdn.net/lishehe/article/details/45224181