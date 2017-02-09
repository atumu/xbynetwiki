title: apache_shiro分布式集群系统session共享 

#  Apache shiro分布式集群系统下session共享 
[Apache Shiro的基本配置](http://shiro.apache.org/web.html)和构成这里就不详细说明了，其官网有说明文档，这里仅仅说明集群的解决方案
Apache Shiro集群要解决2个问题，**一个是session的共享问题，一个是授权信息的cache共享问题**，官网给的例子是Ehcache的实现，在配置说明上不算很详细，**我这里用nosql（redis）替代了ehcache做了session和cache的存储。**

shiro spring的默认配置（**单机，非集群）**
```

<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">    
    <property name="realm" ref="shiroDbRealm" />    
    <property name="cacheManager" ref="memoryConstrainedCacheManager" />    
</bean>    
    
<!-- 自定义Realm -->    
<bean id="shiroDbRealm" class="com.xxx.security.shiro.custom.ShiroDbRealm">    
    <property name="credentialsMatcher" ref="customCredentialsMather"></property>    
</bean>     
<!-- 用户授权信息Cache（本机内存实现） -->    
<bean id="memoryConstrainedCacheManager" class="org.apache.shiro.cache.MemoryConstrainedCacheManager" />      
    
<!-- 保证实现了Shiro内部lifecycle函数的bean执行 -->    
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>    
    
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">    
    <property name="securityManager" ref="securityManager" />    
    <property name="loginUrl" value="/login" />    
    <property name="successUrl" value="/project" />    
    <property name="filterChainDefinitions">    
        <value>    
        /login = authc    
        /logout = logout                
    </value>    
    </property>    
</bean> 

```
如果要用集群，就需要用本地会话，这里shiro给我准备了一个默认的native session manager，` DefaultWebSessionManager `，所以我们要修改spring配置文件，注入DefaultWebSessionManager
```

<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">    
        <property name="sessionManager" ref="defaultWebSessionManager" />    
        <property name="realm" ref="shiroDbRealm" />    
        <property name="cacheManager" ref="memoryConstrainedCacheManager" />    
    </bean>    
    <bean id="defaultWebSessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">    
        <property name="globalSessionTimeout" value="1200000" />    
    </bean> 

```
我们看DefaultWebSessionManager的源码，发现其父类DefaultSessionManager中有` sessionDAO属性 `，这个属性是**真正实现了session储存的类，这个就是我们自己实现的redis session的储存类。**
如果不自己注入sessionDAO，defaultWebSessionManager会使用MemorySessionDAO做为默认实现类，这个肯定不是我们想要的，所以这就自己动手实现sessionDAO吧。
```

import org.apache.shiro.session.Session;  
import org.apache.shiro.session.UnknownSessionException;  
import org.apache.shiro.session.mgt.eis.AbstractSessionDAO;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
  
public class RedisSessionDAO extends AbstractSessionDAO {  
  
    private static Logger logger = LoggerFactory.getLogger(RedisSessionDAO.class);  
    /** 
     * shiro-redis的session对象前缀 
     */  
    private RedisManager redisManager;  
      
    /** 
     * The Redis key prefix for the sessions  
     */  
    private String keyPrefix = "shiro_redis_session:";  
      
    @Override  
    public void update(Session session) throws UnknownSessionException {  
        this.saveSession(session);  
    }  
      
    /** 
     * save session 
     * @param session 
     * @throws UnknownSessionException 
     */  
    private void saveSession(Session session) throws UnknownSessionException{  
        if(session == null || session.getId() == null){  
            logger.error("session or session id is null");  
            return;  
        }  
          
        byte[] key = getByteKey(session.getId());  
        byte[] value = SerializeUtils.serialize(session);  
        session.setTimeout(redisManager.getExpire()*1000);        
        this.redisManager.set(key, value, redisManager.getExpire());  
    }  
  
    @Override  
    public void delete(Session session) {  
        if(session == null || session.getId() == null){  
            logger.error("session or session id is null");  
            return;  
        }  
        redisManager.del(this.getByteKey(session.getId()));  
  
    }  
  
    //用来统计当前活动的session  
    @Override  
    public Collection<Session> getActiveSessions() {  
        Set<Session> sessions = new HashSet<Session>();  
          
        Set<byte[]> keys = redisManager.keys(this.keyPrefix + "*");  
        if(keys != null && keys.size()>0){  
            for(byte[] key:keys){  
                Session s = (Session)SerializeUtils.deserialize(redisManager.get(key));  
                sessions.add(s);  
            }  
        }  
          
        return sessions;  
    }  
  
    @Override  
    protected Serializable doCreate(Session session) {  
        Serializable sessionId = this.generateSessionId(session);    
        this.assignSessionId(session, sessionId);  
        this.saveSession(session);  
        return sessionId;  
    }  
  
    @Override  
    protected Session doReadSession(Serializable sessionId) {  
        if(sessionId == null){  
            logger.error("session id is null");  
            return null;  
        }  
          
        Session s = (Session)SerializeUtils.deserialize(redisManager.get(this.getByteKey(sessionId)));  
        return s;  
    }  
      
    /** 
     * 获得byte[]型的key 
     * @param key 
     * @return 
     */  
    private byte[] getByteKey(Serializable sessionId){  
        String preKey = this.keyPrefix + sessionId;  
        return preKey.getBytes();  
    }  
  
    public RedisManager getRedisManager() {  
        return redisManager;  
    }  
  
    public void setRedisManager(RedisManager redisManager) {  
        this.redisManager = redisManager;  
          
        /** 
         * 初始化redisManager 
         */  
        this.redisManager.init();  
    }  
  
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
}  

```
我们自定义RedisSessionDAO继承AbstractSessionDAO，实现对session操作的方法，可以用redis、mongoDB等进行实现。
这个是自己redis的` RedisCacheManager存储实现类 `：
```

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
这样RedisSessionDAO我们就完成了，下面继续修改我们spring配置文件：
```

<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jee="http://www.springframework.org/schema/jee"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd   
    http://www.springframework.org/schema/aop   
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd   
    http://www.springframework.org/schema/tx    
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd   
    http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-2.5.xsd  
    http://www.springframework.org/schema/context  
    http://www.springframework.org/schema/context/spring-context-3.0.xsd"  
    default-lazy-init="true">  
  
    <!-- 注解支持 -->  
    <context:annotation-config />  
    <bean id="propertyConfigurer"  
        class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
        <property name="locations">  
            <list>  
                <value>classpath:config/shiro-cas.properties</value>  
            </list>  
        </property>  
    </bean>  
      
      
      
    <jee:local-slsb id="PermissionManager"  
        jndi-name="java:global/itoo-authority-role-ear/itoo-authority-shiro-core-0.0.1-SNAPSHOT/PermissionManager!com.tgb.itoo.authority.service.ShiroBean"  
        business-interface="com.tgb.itoo.authority.service.ShiroBean"></jee:local-slsb>  
  
<!--     <jee:jndi-lookup id="PermissionManager" -->  
  
<!--     jndi-name="ejb:itoo-authority-shiro-ear/itoo-authority-shiro-core-0.0.1-SNAPSHOT/PermissionManager!com.tgb.itoo.authority.service.ShiroBean" -->  
  
<!--     lookup-on-startup="true" cache="true" environment-ref="evn"> -->  
  
<!--     </jee:jndi-lookup> -->  
    <!-- shiro过滤器 start -->  
    <bean id="shiroSecurityFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">  
        <property name="securityManager" ref="securityManager"></property>  
        <property name="loginUrl" value="${loginUrl}"></property>  
        <property name="successUrl" value="${successUrl}"></property>  
        <property name="filters">  
            <map>  
                <entry key="casFilter">  
                    <bean class="org.apache.shiro.cas.CasFilter">  
                        <!--配置验证错误时的失败页面 /main 为系统登录页面 -->  
                        <property name="failureUrl" value="/message.jsp" />  
                    </bean>  
                </entry>  
            </map>  
        </property>  
        <!-- 过滤器链，请求url对应的过滤器 -->  
        <property name="filterChainDefinitions">  
            <value>  
                /message.jsp=anon  
                /logout=logout  
                /shiro-cas=casFilter  
                /** =user  
            </value>  
        </property>  
    </bean>  
    <!-- shiro过滤器 end -->  
  
    <!-- 保证实现shiro内部的生命周期函数bean的执行 -->  
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />  
  
    <!-- 第三：shiro管理中心类 start-李社河 -->  
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
  
  
    <!-- 缓存管理器redis-start-李社河-2015年4月14日 -->  
  
    <!-- sessionManager -->  
    <!-- <bean id="sessionManager" -->  
    <!-- class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager"> -->  
    <!-- <property name="sessionDAO" ref="redisSessionDAO" /> -->  
    <!-- </bean> -->  
  
    <!-- 自定义redisManager-redis -->  
    <bean id="redisCacheManager" class="com.tgb.itoo.authority.cache.RedisCacheManager">  
        <property name="redisManager" ref="redisManager" />  
    </bean>  
    <!-- 自定义cacheManager -->  
    <bean id="redisCache" class="com.tgb.itoo.authority.cache.RedisCache">  
        <constructor-arg ref="redisManager"></constructor-arg>  
    </bean>  
  
    <bean id="redisManager" class="com.tgb.itoo.authority.cache.RedisManager"></bean>  
  
    <!-- 缓存管理器redis-end-李社河-2015年4月14日 -->  
  
  
  
    <!-- session会话存储的实现类 -->  
    <bean id="redisShiroSessionDAO" class="com.tgb.itoo.authority.cache.RedisSessionDAO">  
        <property name="redisManager" ref="redisManager" />  
    </bean>  
  
  
  
    <!-- sessionManager -->  
    <!-- session管理器 -->  
    <bean id="sessionManager"  
        class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">  
        <!-- 设置全局会话超时时间，默认30分钟(1800000) -->  
        <property name="globalSessionTimeout" value="1800000" />  
        <!-- 是否在会话过期后会调用SessionDAO的delete方法删除会话 默认true -->  
        <property name="deleteInvalidSessions" value="true" />  
  
        <!-- 会话验证器调度时间 -->  
        <property name="sessionValidationInterval" value="1800000" />  
  
        <!-- session存储的实现 -->  
        <property name="sessionDAO" ref="redisShiroSessionDAO" />  
        <!-- sessionIdCookie的实现,用于重写覆盖容器默认的JSESSIONID -->  
        <property name="sessionIdCookie" ref="sharesession" />  
        <!-- 定时检查失效的session -->  
        <property name="sessionValidationSchedulerEnabled" value="true" />  
  
    </bean>  
  
  
    <!-- sessionIdCookie的实现,用于重写覆盖容器默认的JSESSIONID -->  
    <bean id="sharesession" class="org.apache.shiro.web.servlet.SimpleCookie">  
        <!-- cookie的name,对应的默认是 JSESSIONID -->  
        <constructor-arg name="name" value="SHAREJSESSIONID" />  
        <!-- jsessionId的path为 / 用于多个系统共享jsessionId -->  
        <property name="path" value="/" />  
        <property name="httpOnly" value="true"/>  
    </bean>  
  
  
  
  
  
  
  
    <!-- 第一：shiro于数据交互的类 ，自己写的类的实现-ShiroRealmBean自己重写的类的实现 -->  
    <bean id="shiroRealm" class="com.tgb.itoo.authority.service.ShiroRealmBean">  
        <property name="defaultRoles" value="user"></property>  
  
        <!-- 注入自己实现的类，授权的过程-PermissionManager是云平台重写的授权的过程，用户Id->角色->资源->查找权限-李社河2015年4月15日 -->  
        <property name="permissionMgr" ref="PermissionManager"></property>  
        <property name="casServerUrlPrefix" value="${casServerUrlPrefix}"></property>  
        <property name="casService" value="${casService}"></property>  
    </bean>  
  
    <bean id="casSubjectFactory" class="org.apache.shiro.cas.CasSubjectFactory" />  
  
    <!-- shiro的自带缓存管理器encahe -->  
    <!-- <bean id="shiroEhcacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager"> -->  
    <!-- <property name="cacheManagerConfigFile" value="classpath:config/ehcache-shiro.xml"   
        /> -->  
    <!-- </bean> -->  
  
    <!-- 开启shiro的注解，需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证 -->  
    <bean  
        class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"  
        depends-on="lifecycleBeanPostProcessor"></bean>  
    <bean  
        class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">  
        <property name="securityManager" ref="securityManager" />  
    </bean>  
</beans>  

```
这样第一个问题，session的共享问题我们就解决好了
参考：http://blog.csdn.net/lishehe/article/details/45223823