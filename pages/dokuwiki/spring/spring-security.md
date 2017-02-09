title: spring-security 

#  spring-security 
主要内容：
  * 使用Servlet过滤器保护web应用
  * 基于数据库和LDAP进行认证
  * 透明地对方法调用进行保护

安全性是一个切面。横切关注点。Spring security是一种基于Spring AOP和Servlet过滤器实现的安全框架。
Spring Security能够在两个方面处理身份验证和授权：web请求级别和方法调用级别。
它使用Servlet过滤器保护web请求并限制url级别的访问（这部分使用spring上下文xml配置文件定义）和基于SpringAOP保护方法调用(基于注解)
Spring Security分为8个模块：
![](/data/dokuwiki/spring/pasted/20150807-031754.png)
**Spring Security命名空间**:` xmlns:security="http://www.springframework.org/schema/security" `
##  保护web请求和限制url访问 
###  首先配置代理过滤器： 

```

  <listener>
     <listener-class>org.springframework.security.web.session.HttpSessionEventPublisher</listener-class> <!--为了启用登录并发控制，即同一个用户的登录人数限制。-->
   </listener>
 <filter>
        <filter-name>springSecurityFilterChain</filter-name> <!-- 这个Fileter name：springSecurityFilterChain是有意思的用于查找自动创建的过滤器bean。 -->
        <filter-class>
            org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

```
###  配置最小化的web安全性： 

```

 <http auto-config="true" use-expressions="true">
<intercept-url pattern="/admin/**"   ant风格路径
        access="ROLE_SPITTER"/> 
</http>

```
完整示例：
```

  <http auto-config="true" use-expressions="true">
    <form-login login-page="/login"
                login-processing-url="/static/j_spring_security_check"  
                authentication-failure-url="/login"/>
    <logout logout-url="/static/j_spring_security_logout"/>
    <intercept-url pattern="/favicon.ico" access="permitAll" />
    <intercept-url pattern="/home" access="hasRole('ROLE_SPITTER')"/>
    <intercept-url pattern="/admin/**" 
        access="isAuthenticated() and principal.username=='habuma'"/>
    <intercept-url pattern="/login" requires-channel="https"/>
    <intercept-url pattern="/spitter/form" requires-channel="https"/>

    <remember-me key="spitterKey"
        token-validity-seconds="2419200" />    
        
  </http>

```
这个自动配置会为我们提供一个额外的登录页、HTTP基本认证和退出功能。
但是我们需要自己的登录页和退出。则进行如下配置：
```

<http auto-config="true" use-expressions="true">
    <form-login login-page="/login"
                login-processing-url="/static/j_spring_security_check"     /static/j_spring_security_check固定
                authentication-failure-url="/login"/>
   <logout logout-url="/static/j_spring_security_logout"/>      /static/j_spring_security_logout固定
  ....
</http>

```
**登录页面**的写法也需要规范：` j_username  j_password _spring_security_remember_me   /static/j_spring_security_check `
```

<%@ taglib prefix="s" uri="http://www.springframework.org/tags"%>
<div>
   <s:url var="authUrl" 
          value="/static/j_spring_security_check" /> 
   <form method="post" class="signin" action="${authUrl}">      form action的url必须是/static/j_spring_security_check
   
    <fieldset>
    <table cellspacing="0">
    <tr>
    <th><label for="username_or_email">Username or Email</label></th>
    <td><input id="username_or_email" 
               name="j_username"          				name必须是j_username
               type="text" />  
      </td>
    </tr>
    <tr>
    <th><label for="password">Password</label></th>
      <td><input id="password" 
                 name="j_password" 					 name必须是j_password
                 type="password" /> 
          <small><a href="/account/resend_password">Forgot?</a></small>
      </td>
    </tr>
    <tr>
    <th></th>
    <td><input id="remember_me" 
        name="_spring_security_remember_me"  		name必须是_spring_security_remember_me
        type="checkbox"/> 
        <label for="remember_me" 
               class="inline">Remember me</label></td>
    </tr>
    <tr>
    <th></th>
    <td><input name="commit" type="submit" value="Sign In" /></td>
    </tr>
    </table>
    </fieldset>
   </form>

```
###  拦截请求 
<intercept-url pattern=  access= />
**注意声明顺序，从上之下。**
使用普通模式  ```

<intercept-url pattern="/admin/**" 
        access="ROLE_SPITTER"/>

```
使用SpEL进行安全保护：
```

  <http auto-config="true" use-expressions="true">   use-expressions="true"开启SpEL
    <form-login login-page="/login"
                login-processing-url="/static/j_spring_security_check"  
                authentication-failure-url="/login"/>
    <logout logout-url="/static/j_spring_security_logout"/>
    <intercept-url pattern="/favicon.ico" access="permitAll" /> 
    <intercept-url pattern="/home" access="hasRole('ROLE_SPITTER')"/>
    <intercept-url pattern="/admin/**" 
        access="isAuthenticated() and principal.username=='habuma'"/>
    <intercept-url pattern="/login" requires-channel="https"/>
    <intercept-url pattern="/spitter/form" requires-channel="https"/>
    <remember-me key="spitterKey"
        token-validity-seconds="2419200" />            
  </http>

```
Spring Security的一些安全性相关的SpEL扩展：
![](/data/dokuwiki/spring/pasted/20150807-035319.png)
####  强制请求使用HTTPS 
` <intercept-url pattern="/login" requires-channel="https"/> `
##  保护视图级别元素 
Spring Security提供的几个JSP标签：
![](/data/dokuwiki/spring/pasted/20150807-035544.png)
**访问认证信息细节：**
![](/data/dokuwiki/spring/pasted/20150807-035630.png)
![](/data/dokuwiki/spring/pasted/20150807-035647.png)
**根据权限渲染输出**
![](/data/dokuwiki/spring/pasted/20150807-035734.png)
**如何组织手动输入访问受限url**
![](/data/dokuwiki/spring/pasted/20150807-035825.png)
![](/data/dokuwiki/spring/pasted/20150807-035844.png)
##  认证用户存储方式 
认证策略：
![](/data/dokuwiki/spring/pasted/20150807-035939.png)
###  配置内存用户存储库 
```

<user-service id="userService">
      <user name="habuma" password="letmein" 
                     authorities="ROLE_SPITTER,ROLE_ADMIN"/>
      <user name="twoqubed" password="longhorns" 
                     authorities="ROLE_SPITTER"/>
      <user name="admin" password="admin" 
                     authorities="ROLE_ADMIN"/>
    </user-service>

```
然后在上下文配置文件中配置:
```

  <beans:import resource="spitter-security-inmemory.xml"/>
  <authentication-manager>
    <authentication-provider user-service-ref="userService" />
  </authentication-manager>

```
###  基于数据库进行认证 
```

    <jdbc-user-service id="userService" 
       data-source-ref="dataSource"
       users-by-username-query=
          "select username, password, true from spitter where username=?"  
       authorities-by-username-query=
          "select username,'ROLE_SPITTER' from spitter where username=?" />

```
然后在上下文配置文件中配置:
```

  <beans:import resource="spitter-security-jdbc.xml"/>
  <authentication-manager>
    <authentication-provider user-service-ref="userService" />
  </authentication-manager>

```
![](/data/dokuwiki/spring/pasted/20150807-040605.png)
![](/data/dokuwiki/spring/pasted/20150807-040617.png)
![](/data/dokuwiki/spring/pasted/20150807-040626.png)
###  基于LDAP进行认证 
LDAP能够很好地表示层级的数据结构。
略。。。
###  启用remember-me功能 
记住登录功能。
```

<http auto-config="true" use-expressions="true">
	... 
    <remember-me key="spitterKey"            给私钥一个别名
        token-validity-seconds="2419200" />  有效期，秒为单位  
        ...
  </http>

```
然后表单：` _spring_security_remember_me `
```

<input id="remember_me" 
        name="_spring_security_remember_me" 必须是_spring_security_remember_me
        type="checkbox"/>
        <label for="remember_me" 
               class="inline">Remember me</label></td>

```
![](/data/dokuwiki/spring/pasted/20150807-041008.png)
##  保护方法调用 
**首先开启注解支持:**
''<global-method-security secured-annotations="enabled" /> 针对Spring Security注解@Secured
<global-method-security jsr250-annotations="enabled" />  针对JSR-250标准注解@RolesAllowed''
###  使用@Secured注解保护方法调用 
```

@Secured("ROLE_ADMIN")或者@Secured({"ROLE_ADMIN","ROLE_SPITTER"}) 只需具备其中之一
public void add(){}

```
###  使用JSR-250标准的@RolesAllowed注解 
差不多
###  使用SpEL实现调用前后的安全性 
首先启用这一功能：
` <global-method-security pre-post-annotations="enabled" /> `
![](/data/dokuwiki/spring/pasted/20150807-041716.png)
![](/data/dokuwiki/spring/pasted/20150807-041805.png)
![](/data/dokuwiki/spring/pasted/20150807-041819.png)
略。。。
###  声明方法级别的安全性切点 
![](/data/dokuwiki/spring/pasted/20150807-041958.png)