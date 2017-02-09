title: apache_shiro_会话管理 

#  apache Shiro 会话管理 
**Shiro提供了完整的企业级会话管理功能，不依赖于底层容器（如web容器tomcat）**，不管JavaSE还是JavaEE环境都可以使用，
提供了会话管理、会话事件监听、会话存储/持久化、**容器无关的集群**、失效/过期支持、对Web的透明支持、**SSO单点登录的支持**等特性。即**直接使用Shiro的会话管理可以直接替换如Web容器的会话管理。**
会话
所谓**会话，即用户访问应用时保持的连接关系，在多次交互中应用能够识别出当前访问的用户是谁，且可以在多次交互中保存一些数据**。如访问一些网站时登录成功后，网站可以记住用户，且在退出之前都可以识别当前用户是谁。
 
Shiro的会话支持不仅可以在普通的JavaSE应用中使用，也可以在JavaEE应用中使用，如web应用。且使用方式是一致的。 
```

login("classpath:shiro.ini", "zhang", "123");  
Subject subject = SecurityUtils.getSubject();  
Session session = subject.getSession();  

``` 
登录成功后使用**Subject.getSession()即可获取会话**；其等价于Subject.getSession(true)，即如果当前没有创建Session对象会创建一个；另外Subject.getSession(false)，如果当前没有创建Session则返回null（不过默认情况下如果启用会话存储功能的话在创建Subject时会主动创建一个Session）。

session.getId();  
获取当前会话的唯一标识。
  
session.getHost();  
获取当前Subject的主机地址，该地址是通过HostAuthenticationToken.getHost()提供的。 
 
session.getTimeout();  
session.setTimeout(毫秒);   
获取/设置当前Session的过期时间；如果不设置默认是会话管理器的全局过期时间。

session.getStartTimestamp();  
session.getLastAccessTime();  
获取会话的启动时间及最后访问时间；如果是JavaSE应用需要自己定期调用session.touch()去更新最后访问时间；如果是Web应用，每次进入ShiroFilter都会自动调用session.touch()来更新最后访问时间。    
 
session.touch();  
session.stop();   
更新会话最后访问时间及销毁会话；当Subject.logout()时会自动调用stop方法来销毁会话。如果在web中，调用javax.servlet.http.HttpSession. invalidate()也会自动调用Shiro Session.stop方法进行销毁Shiro的会话。 
 
session.setAttribute("key", "123");  
Assert.assertEquals("123", session.getAttribute("key"));  
session.removeAttribute("key");  
设置/获取/删除会话属性；在整个会话范围内都可以对这些属性进行操作。 
**Shiro提供的会话**可以用于JavaSE/JavaEE环境，**不依赖于任何底层容器，可以独立使用，是完整的会话模块。**

##  会话管理器 
会话管理器管理着应用中所有Subject的会话的创建、维护、删除、失效、验证等工作。是Shiro的核心组件，顶层组件SecurityManager直接继承了SessionManager，且提供了SessionsSecurityManager实现直接把会话管理委托给相应的SessionManager，DefaultSecurityManager及DefaultWebSecurityManager默认SecurityManager都继承了SessionsSecurityManager。
 
SecurityManager提供了如下接口：
Session start(SessionContext context); //启动会话  
Session getSession(SessionKey key) throws SessionException; //根据会话Key获取会话   
 
另外用于Web环境的WebSessionManager又提供了如下接口：
```

boolean isServletContainerSessions();//是否使用Servlet容器的会话 

``` 
Shiro还提供了ValidatingSessionManager用于验资并过期会话： 
```

void validateSessions();//验证所有会话是否过期 

```
![](/data/dokuwiki/shiro/pasted/20151029-222255.png)

Shiro提供了三个默认实现：
  * DefaultSessionManager：DefaultSecurityManager使用的默认实现，**用于JavaSE环境**；
  * ServletContainerSessionManager：DefaultWebSecurityManager使用的默认实现，**用于Web环境，其直接使用Servlet容器的会话；**
  * DefaultWebSessionManager：**用于Web环境的实现**，可以替代ServletContainerSessionManager，**自己维护着会话，直接废弃了Servlet容器的会话管理。**
替换SecurityManager默认的SessionManager可以在ini中配置（shiro.ini）：
```

[main]  
sessionManager=org.apache.shiro.session.mgt.DefaultSessionManager  
securityManager.sessionManager=$sessionManager  

``` 
 
 
Web环境下的ini配置(shiro-web.ini)：
```

[main]  
sessionManager=org.apache.shiro.web.session.mgt.ServletContainerSessionManager  
securityManager.sessionManager=$sessionManager 

``` 

另外可以设置会话的全局过期时间（毫秒为单位），默认30分钟：
```

sessionManager.globalSessionTimeout=1800000

```   
默认情况下globalSessionTimeout将应用给所有Session。可以单独设置每个Session的timeout属性来为每个Session设置其超时时间。
 
另外如果使用ServletContainerSessionManager进行会话管理，Session的超时依赖于底层Servlet容器的超时时间，可以在web.xml中配置其会话的超时时间（分钟为单位）： 
```

<session-config>  
  <session-timeout>30</session-timeout>  
</session-config>

```  
**在Servlet容器中，默认使用JSESSIONID Cookie维护会话，且会话默认是跟容器绑定的；**在某些情况下可能需要使用自己的会话机制，此时我们可以使用` DefaultWebSessionManager `来维护会话：
```

sessionIdCookie=org.apache.shiro.web.servlet.SimpleCookie  
sessionManager=org.apache.shiro.web.session.mgt.DefaultWebSessionManager  
sessionIdCookie.name=sid  
#sessionIdCookie.domain=sishuok.com  
#sessionIdCookie.path=  
sessionIdCookie.maxAge=1800  
sessionIdCookie.httpOnly=true  
sessionManager.sessionIdCookie=$sessionIdCookie  
sessionManager.sessionIdCookieEnabled=true  
securityManager.sessionManager=$sessionManager  

``` 
sessionIdCookie是sessionManager创建会话Cookie的模板：
sessionIdCookie.name：设置Cookie名字，默认为JSESSIONID；
sessionIdCookie.domain：设置Cookie的域名，默认空，即当前访问的域名；
sessionIdCookie.path：设置Cookie的路径，默认空，即存储在域名根下；
sessionIdCookie.maxAge：设置Cookie的过期时间，秒为单位，默认-1表示关闭浏览器时过期Cookie；
sessionIdCookie.httpOnly：如果设置为true，则客户端不会暴露给客户端脚本代码，使用HttpOnly cookie有助于减少某些类型的跨站点脚本攻击；此特性需要实现了Servlet 2.5 MR6及以上版本的规范的Servlet容器支持；
sessionManager.sessionIdCookieEnabled：是否启用/禁用Session Id Cookie，默认是启用的；如果禁用后将不会设置Session Id Cookie，即默认使用了Servlet容器的JSESSIONID，且通过URL重写（URL中的“;JSESSIONID=id”部分）保存Session Id。
另外我们可以如“sessionManager. sessionIdCookie.name=sid”这种方式操作Cookie模板。

##  会话监听器 
会话监听器用于监听会话创建、过期及停止事件： 
```

public class MySessionListener1 implements SessionListener {  
    @Override  
    public void onStart(Session session) {//会话创建时触发  
        System.out.println("会话创建：" + session.getId());  
    }  
    @Override  
    public void onExpiration(Session session) {//会话过期时触发  
        System.out.println("会话过期：" + session.getId());  
    }  
    @Override  
    public void onStop(Session session) {//退出/会话过期时触发  
        System.out.println("会话停止：" + session.getId());  
    }    
} 

``` 
如果只想监听某一个事件，可以继承` SessionListenerAdapter `实现：
 
在shiro-web.ini配置文件中可以进行如下配置设置会话监听器：
```

sessionListener1=com.github.zhangkaitao.shiro.chapter10.web.listener.MySessionListener1  
sessionListener2=com.github.zhangkaitao.shiro.chapter10.web.listener.MySessionListener2  
sessionManager.sessionListeners=$sessionListener1,$sessionListener2 

```
##  会话存储/持久化  
Shiro提供` SessionDAO `用于会话的CRUD，即DAO（Data Access Object）模式实现：
```

//如DefaultSessionManager在创建完session后会调用该方法；如保存到关系数据库/文件系统/NoSQL数据库；即可以实现会话的持久化；返回会话ID；主要此处返回的ID.equals(session.getId())；  
Serializable create(Session session);  
//根据会话ID获取会话  
Session readSession(Serializable sessionId) throws UnknownSessionException;  
//更新会话；如更新会话最后访问时间/停止会话/设置超时时间/设置移除属性等会调用  
void update(Session session) throws UnknownSessionException;  
//删除会话；当会话过期/会话停止（如用户退出时）会调用  
void delete(Session session);  
//获取当前所有活跃用户，如果用户量多此方法影响性能  
Collection<Session> getActiveSessions();  

```
![](/data/dokuwiki/shiro/pasted/20151029-222752.png)

可以通过如下配置设置SessionDAO：
```

sessionDAO=org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO  
sessionManager.sessionDAO=$sessionDAO  

``` 
Shiro提供了使用**Ehcache进行会话存储，**Ehcache可以配合TerraCotta实现容器无关的分布式集群。
 
首先在pom.xml里添加如下依赖：
```

<dependency>  
    <groupId>org.apache.shiro</groupId>  
    <artifactId>shiro-ehcache</artifactId>  
    <version>1.2.2</version>  
</dependency> 

```  
 
接着配置shiro-web.ini文件：    
```

sessionDAO=org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO  
sessionDAO. activeSessionsCacheName=shiro-activeSessionCache  
sessionManager.sessionDAO=$sessionDAO  
cacheManager = org.apache.shiro.cache.ehcache.EhCacheManager  
cacheManager.cacheManagerConfigFile=classpath:ehcache.xml  
securityManager.cacheManager = $cacheManager  

``` 
sessionDAO. activeSessionsCacheName：设置Session缓存名字，默认就是shiro-activeSessionCache；
cacheManager：缓存管理器，用于管理缓存的，此处使用Ehcache实现；
cacheManager.cacheManagerConfigFile：设置ehcache缓存的配置文件；
securityManager.cacheManager：设置SecurityManager的cacheManager，会自动设置实现了CacheManagerAware接口的相应对象，如SessionDAO的cacheManager；
 
然后配置ehcache.xml：
```

<cache name="shiro-activeSessionCache"  
       maxEntriesLocalHeap="10000"  
       overflowToDisk="false"  
       eternal="false"  
       diskPersistent="false"  
       timeToLiveSeconds="0"  
       timeToIdleSeconds="0"  
       statistics="true"/>

```   
Cache的名字为shiro-activeSessionCache，即设置的sessionDAO的activeSessionsCacheName属性值。

另外可以通过如下ini配置设置会话ID生成器：
sessionIdGenerator=org.apache.shiro.session.mgt.eis.JavaUuidSessionIdGenerator  
sessionDAO.sessionIdGenerator=$sessionIdGenerator   
用于生成会话ID，默认就是JavaUuidSessionIdGenerator，使用java.util.UUID生成。

