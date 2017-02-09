title: spring-security角色继承配置 

#  Spring-Security角色继承配置 
##  角色的继承 
 对于角色继承这种需求也是经常有的，比如要求ROLE_ADMIN将拥有所有的ROLE_USER所具有的权限。当然我们可以给拥有ROLE_ADMIN角色的用户同时授予ROLE_USER角色来达到这一效果或者修改需要ROLE_USER进行访问的资源使用ROLE_ADMIN也可以访问。
Spring Security为我们提供了一种更为简便的办法，那就是` 角色的继承 `，它允许我们的ROLE_ADMIN直接继承ROLE_USER，这样所有ROLE_USER可以访问的资源ROLE_ADMIN也可以访问。
定义角色的继承我们需要在ApplicationContext中定义一个` RoleHierarchy `，然后再把它赋予给一个` RoleHierarchyVoter `，之后再把该RoleHierarchyVoter加入到我们**基于Voter的AccessDecisionManager中**，并指定当前使用的AccessDecisionManager为我们自己定义的那个。以下是一个定义角色继承的完整示例。
```

<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/security
           http://www.springframework.org/schema/security/spring-security.xsd">
     <!-- 方法安全 -->
	<global-method-security pre-post-annotations="enabled" jsr250-annotations="enabled" access-decision-manager-ref="methodAccessDecisionManager" >
	 <expression-handler ref="methodSecurityExpressionHandler"/>
	</global-method-security>
	<authentication-manager> <!-- AuthenticationManager管理AuthenticationProvider管理UserService管理User认证信息获取 -->
		<authentication-provider
        				 user-service-ref="defaultUserAuthenticationService">
			<password-encoder hash="bcrypt" />
		</authentication-provider>
	</authentication-manager>
  	<!--静态资源忽略 -->
	<http security="none" pattern="/resource/**" />
 	<!-- HTTP安全 -->
	<http  use-expressions="true" access-decision-manager-ref="webAccessDecisionManager">   <!-- use-expressions="true"开启SpEL -->
		<expression-handler ref="webSecurityExpressionHandler"/>
		<session-management session-fixation-protection="migrateSession"> <!--会话固定攻击防护，有几种选择：changeSessionId(只支持Servlet3.1以上),migrateSession(默认选项，将创建新的会话并复制所有现有特性),newSession,none -->
			<concurrency-control max-sessions="1" expired-url="/login?maxSessions=1" /> <!--限制用户会话数量，为了启用该特性，记得配置前面所讲的特殊的监听器 -->
		</session-management>
		<!-- <remember-me key="myAppName" token-validity-seconds="2419200" />提供记住我功能，提供一个字段名为_spring_security_remember_me的复选框，如果使用Java配置时字段名为remember-me -->
		<form-login login-page="/login" login-processing-url="/login"
			authentication-failure-url="/login?error=1" default-target-url="/home/"  
			username-parameter="name"
			password-parameter="password"/>
			<logout logout-url="/loginOut" logout-success-url="/index"
				invalidate-session="true" delete-cookies="JSESSIONID" />
			<intercept-url pattern="/index" access="permitAll" />
			<intercept-url pattern="/login" access="permitAll" />
			<intercept-url pattern="/loginOut" access="permitAll" />
			<intercept-url pattern="/captcha/**" access="permitAll" />
			<intercept-url pattern="/home" access="hasAuthority('ROLE_USER')" />
			<intercept-url pattern="/admin/**" access="hasAuthority('ROLE_ADMIN')" />
			<intercept-url pattern="/upload/**" access="hasAuthority('ROLE_ADMIN')" />

	</http>
	 <!-- 角色继承 -->   
   <beans:bean id="roleHierarchy"
      class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl">
      <beans:property name="hierarchy"><!-- 角色继承关系 -->
         <beans:value>
            ROLE_ADMIN > ROLE_USER
         </beans:value>
      </beans:property>
   </beans:bean>
     <beans:bean id="roleHierarchyVoter"
      class="org.springframework.security.access.vote.RoleHierarchyVoter">
      <beans:constructor-arg ref="roleHierarchy" />
   </beans:bean>
 <!-- 用于web的ExpressionHandler -->
    <beans:bean id="webSecurityExpressionHandler" name="webSecurityExpressionHandler"
                class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler">
        <beans:property name="roleHierarchy" ref="roleHierarchy"/>
    </beans:bean>
 
    <!-- 用于method的ExpressionHandler -->
    <beans:bean id="methodSecurityExpressionHandler" class="org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler">
        <beans:property name="roleHierarchy" ref="roleHierarchy"/>
    </beans:bean>
   <!-- 用于web的ExpressionHandler -->   
 <beans:bean id="webAccessDecisionManager"  
    class="org.springframework.security.access.vote.AffirmativeBased">  
    <beans:property name="decisionVoters">  
        <beans:list>  
            <beans:bean class="org.springframework.security.web.access.expression.WebExpressionVoter">
                    <beans:property name="expressionHandler" ref="webSecurityExpressionHandler"/>
                </beans:bean>
                <beans:ref bean="roleHierarchyVoter" />  
                <beans:bean class="org.springframework.security.access.vote.AuthenticatedVoter"/>
        </beans:list>  
    </beans:property>  
</beans:bean> 
 <!-- 用于method的AccessDecisionManager -->
    <beans:bean id="methodAccessDecisionManager" class="org.springframework.security.access.vote.AffirmativeBased">
        <beans:constructor-arg>
            <beans:list>
                <beans:bean class="org.springframework.security.access.vote.RoleHierarchyVoter">
                    <beans:constructor-arg ref="roleHierarchy"/>
                </beans:bean>
                <beans:bean class="org.springframework.security.access.vote.AuthenticatedVoter"/>
                <beans:bean class="org.springframework.security.access.prepost.PreInvocationAuthorizationAdviceVoter">
                    <beans:constructor-arg name="pre">
                        <beans:bean class="org.springframework.security.access.expression.method.ExpressionBasedPreInvocationAdvice">
                            <beans:property name="expressionHandler" ref="methodSecurityExpressionHandler"/>
                        </beans:bean>
                    </beans:constructor-arg>
                </beans:bean>
                <beans:bean class="org.springframework.security.access.annotation.Jsr250Voter"/>
            </beans:list>
        </beans:constructor-arg>
    </beans:bean>
</beans:beans>

```
 在上述配置中我们就定义好了ROLE_ADMIN是继承自ROLE_USER的，这样ROLE_ADMIN将能够访问所有ROLE_USER可以访问的资源。通过RoleHierarchyImpl的hierarchy属性我们可以定义多个角色之间的继承关系，如：
```

 <bean id="roleHierarchy"
      class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl">
      <property name="hierarchy"><!-- 角色继承关系 -->
         <value>
            ROLE_ADMIN > ROLE_USER
            ROLE_A > ROLE_B
            ROLE_B > ROLE_C
            ROLE_C > ROLE_D
         </value>
      </property>
   </bean>

```
在上述配置我们同时定义了ROLE_ADMIN继承了ROLE_USER，ROLE_A继承了ROLE_B，ROLE_B又继承了ROLE_C，ROLE_C又继承了ROLE_D，这样ROLE_A将能访问ROLE_B、ROLE_C和ROLE_D所能访问的所有资源。

参考：http://blog.csdn.net/eddy615/article/details/6726650
http://my.oschina.net/since1986/blog/325157


附录：http://my.oschina.net/since1986/blog/325157这位哥们的配置：
spring security 中启用角色继承、ACL与CAS
```

<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
             xmlns:beans="http://www.springframework.org/schema/beans"
             xmlns:context="http://www.springframework.org/schema/context"
             xmlns:util="http://www.springframework.org/schema/util"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/util
          http://www.springframework.org/schema/util/spring-util.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security.xsd">
 
    <!-- 方法安全 -->
    <global-method-security secured-annotations="enabled" pre-post-annotations="enabled" jsr250-annotations="enabled" access-decision-manager-ref="methodAccessDecisionManager">
        <expression-handler ref="methodSecurityExpressionHandler"/>
        <after-invocation-provider ref="postInvocationAdviceProvider"/>
    </global-method-security>
 
    <!-- HTTP安全 -->
    <http pattern="/proxy.html" security="none" />
    <http entry-point-ref="casEntryPoint" use-expressions="true" access-decision-manager-ref="webAccessDecisionManager">
        <expression-handler ref="webSecurityExpressionHandler"/>
        <intercept-url pattern="/" access="hasRole('ROLE_USER')"/>
        <custom-filter ref="requestSingleLogoutFilter" before="LOGOUT_FILTER"/>
        <custom-filter ref="singleLogoutFilter" before="CAS_FILTER"/>
        <custom-filter ref="casFilter" position="CAS_FILTER"/>
    </http>
 
 
    <!-- ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓授权(Authorization)配置↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
    <beans:bean id="aclAuthorizationStrategy" class="org.springframework.security.acls.domain.AclAuthorizationStrategyImpl">
        <beans:constructor-arg name="auths">
            <beans:list>
                <beans:ref bean="supervisor"/>
                <beans:ref bean="supervisor"/>
                <beans:ref bean="supervisor"/>
            </beans:list>
        </beans:constructor-arg>
    </beans:bean>
 
    <beans:bean id="permissionGrantingStrategy" class="org.springframework.security.acls.domain.DefaultPermissionGrantingStrategy">
        <beans:constructor-arg name="auditLogger">
            <beans:bean class="org.springframework.security.acls.domain.ConsoleAuditLogger"/>
        </beans:constructor-arg>
    </beans:bean>
 
    <beans:bean id="supervisor" class="org.springframework.security.core.authority.SimpleGrantedAuthority">
        <beans:constructor-arg name="role" value="ROLE_SUPERVISOR"/>
    </beans:bean>
 
    <beans:bean id="aclCache" class="org.springframework.security.acls.domain.EhCacheBasedAclCache">
        <beans:constructor-arg name="cache">
            <beans:bean class="org.springframework.cache.ehcache.EhCacheFactoryBean">
                <beans:property name="overflowToDisk" value="true"/>
                <beans:property name="maxElementsInMemory" value="1024"/>
            </beans:bean>
        </beans:constructor-arg>
        <beans:constructor-arg name="permissionGrantingStrategy" ref="permissionGrantingStrategy"/>
        <beans:constructor-arg name="aclAuthorizationStrategy" ref="aclAuthorizationStrategy"/>
    </beans:bean>
 
    <beans:bean id="lookupStrategy" class="org.springframework.security.acls.jdbc.BasicLookupStrategy">
        <beans:constructor-arg name="dataSource" ref="dataSource"/>
        <beans:constructor-arg name="aclCache" ref="aclCache"/>
        <beans:constructor-arg name="aclAuthorizationStrategy" ref="aclAuthorizationStrategy"/>
        <beans:constructor-arg name="grantingStrategy" ref="permissionGrantingStrategy"/>
    </beans:bean>
 
    <beans:bean id="aclService" class="org.springframework.security.acls.jdbc.JdbcMutableAclService">
        <beans:constructor-arg name="dataSource" ref="dataSource"/>
        <beans:constructor-arg name="lookupStrategy" ref="lookupStrategy"/>
        <beans:constructor-arg name="aclCache" ref="aclCache"/>
        <beans:property name="classIdentityQuery" value="select @@identity"/><!-- select @@identity用于获得刚刚插入的自增id select LAST_INSERT_ID() -->
        <beans:property name="sidIdentityQuery" value="select @@identity"/>
    </beans:bean>
 
    <!-- 通过表达式使用ACL -->
    <beans:bean id="aclPermissionEvaluator" class="org.springframework.security.acls.AclPermissionEvaluator">
        <beans:constructor-arg name="aclService" ref="aclService"/>
    </beans:bean>
 
    <!-- 角色继承 -->
    <beans:bean id="roleHierarchy" class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl">
        <beans:property name="hierarchy">
            <beans:value>
                ROLE_DEVELOPER > ROLE_SUPERVISOR
                ROLE_SUPERVISOR > ROLE_ADMIN
                ROLE_ADMIN > ROLE_USER
            </beans:value>
        </beans:property>
    </beans:bean>
 
    <!-- 用于web的ExpressionHandler -->
    <beans:bean id="webSecurityExpressionHandler" name="webSecurityExpressionHandler"
                class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler">
        <beans:property name="roleHierarchy" ref="roleHierarchy"/>
        <beans:property name="permissionEvaluator" ref="aclPermissionEvaluator"/>
    </beans:bean>
 
    <!-- 用于method的ExpressionHandler -->
    <beans:bean id="methodSecurityExpressionHandler" class="org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler">
        <beans:property name="roleHierarchy" ref="roleHierarchy"/>
        <beans:property name="permissionCacheOptimizer">
            <beans:bean class="org.springframework.security.acls.AclPermissionCacheOptimizer">
                <beans:constructor-arg name="aclService" ref="aclService"/>
            </beans:bean>
        </beans:property>
        <beans:property name="permissionEvaluator" ref="aclPermissionEvaluator"/>
    </beans:bean>
 
    <!-- 用于web(taglib以及url)的AccessDecisionManager -->
    <beans:bean id="webAccessDecisionManager" class="org.springframework.security.access.vote.AffirmativeBased">
        <beans:constructor-arg>
            <beans:list>
                <beans:bean class="org.springframework.security.web.access.expression.WebExpressionVoter">
                    <beans:property name="expressionHandler" ref="webSecurityExpressionHandler"/>
                </beans:bean>
                <beans:bean class="org.springframework.security.access.vote.RoleHierarchyVoter">
                    <beans:constructor-arg ref="roleHierarchy"/>
                </beans:bean>
                <beans:bean class="org.springframework.security.access.vote.AuthenticatedVoter"/>
            </beans:list>
        </beans:constructor-arg>
    </beans:bean>
 
    <!-- 用于method的AccessDecisionManager -->
    <beans:bean id="methodAccessDecisionManager" class="org.springframework.security.access.vote.AffirmativeBased">
        <beans:constructor-arg>
            <beans:list>
                <beans:bean class="org.springframework.security.access.vote.RoleHierarchyVoter">
                    <beans:constructor-arg ref="roleHierarchy"/>
                </beans:bean>
                <beans:bean class="org.springframework.security.access.vote.AuthenticatedVoter"/>
                <beans:bean class="org.springframework.security.access.prepost.PreInvocationAuthorizationAdviceVoter">
                    <beans:constructor-arg name="pre">
                        <beans:bean class="org.springframework.security.access.expression.method.ExpressionBasedPreInvocationAdvice">
                            <beans:property name="expressionHandler" ref="methodSecurityExpressionHandler"/>
                        </beans:bean>
                    </beans:constructor-arg>
                </beans:bean>
                <beans:bean class="org.springframework.security.access.annotation.Jsr250Voter"/>
            </beans:list>
        </beans:constructor-arg>
    </beans:bean>
 
    <!-- 后置过滤 -->
    <beans:bean id="postInvocationAdviceProvider" class="org.springframework.security.access.prepost.PostInvocationAdviceProvider">
        <beans:constructor-arg name="postAdvice">
            <beans:bean class="org.springframework.security.access.expression.method.ExpressionBasedPostInvocationAdvice">
                <beans:constructor-arg name="expressionHandler" ref="methodSecurityExpressionHandler"/>
            </beans:bean>
        </beans:constructor-arg>
    </beans:bean>
    <!-- ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑/授权(Authorization)配置↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->
 
    <!-- ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓认证(Authentication)配置↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
    <authentication-manager alias="authManager">
        <authentication-provider ref="casAuthProvider"/>
    </authentication-manager>
 
    <beans:bean id="userService" class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
        <beans:property name="enableGroups" value="true"/>
        <beans:property name="dataSource" ref="dataSource"/>
    </beans:bean>
 
    <!-- This filter handles a Single Logout Request from the CAS Server -->
    <beans:bean id="singleLogoutFilter" class="org.jasig.cas.client.session.SingleSignOutFilter"/>
    <!-- This filter redirects to the CAS Server to signal Single Logout should be performed -->
    <beans:bean id="requestSingleLogoutFilter"
                class="org.springframework.security.web.authentication.logout.LogoutFilter">
        <beans:property name="filterProcessesUrl" value="/j_spring_cas_security_logout"/>
        <beans:constructor-arg name="logoutSuccessUrl" value="https://${cas.server}/logout"/>
        <beans:constructor-arg name="handlers">
            <beans:bean class="org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler"/>
        </beans:constructor-arg>
    </beans:bean>
 
    <beans:bean id="serviceProperties" class="org.springframework.security.cas.ServiceProperties">
        <beans:property name="service" value="https://${cas.service}/j_spring_cas_security_check"/>
    </beans:bean>
    <beans:bean id="casEntryPoint"
                class="org.springframework.security.cas.web.CasAuthenticationEntryPoint">
        <beans:property name="serviceProperties" ref="serviceProperties"/>
        <beans:property name="loginUrl" value="https://${cas.server}/login"/>
    </beans:bean>
    <beans:bean id="casFilter"
                class="org.springframework.security.cas.web.CasAuthenticationFilter">
        <beans:property name="authenticationManager" ref="authManager"/>
        <beans:property name="serviceProperties" ref="serviceProperties"/>
        <beans:property name="authenticationDetailsSource">
            <beans:bean class="org.springframework.security.cas.web.authentication.ServiceAuthenticationDetailsSource"/>
        </beans:property>
        <beans:property name="authenticationFailureHandler">
            <beans:bean class="org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler"/>
        </beans:property>
    </beans:bean>
 
    <!--
        NOTE: In a real application you should not use an in memory implementation. You will also want
              to ensure to clean up expired tickets by calling ProxyGrantingTicketStorage.cleanup()
     -->
    <beans:bean id="casAuthProvider" class="org.springframework.security.cas.authentication.CasAuthenticationProvider">
        <beans:property name="serviceProperties" ref="serviceProperties"/>
        <beans:property name="key" value="${cas.key}"/>
        <beans:property name="authenticationUserDetailsService">
            <beans:bean
                    class="org.springframework.security.core.userdetails.UserDetailsByNameServiceWrapper">
                <beans:constructor-arg ref="userService"/>
            </beans:bean>
        </beans:property>
        <beans:property name="ticketValidator">
            <beans:bean
                    class="org.jasig.cas.client.validation.Cas20ProxyTicketValidator">
                <beans:constructor-arg value="https://${cas.server}"/>
            </beans:bean>
        </beans:property>
    </beans:bean>
 
    <!-- Configuration for the environment can be overriden by system properties -->
    <context:property-placeholder system-properties-mode="OVERRIDE" properties-ref="environment"/>
    <util:properties id="environment">
        <beans:prop key="cas.service">localhost:8444/user</beans:prop>
        <beans:prop key="cas.server">localhost:8443/cas</beans:prop>
        <beans:prop key="cas.key">CAS_KEY_ADMIN.USER</beans:prop>
    </util:properties>
    <!-- ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑/认证(Authentication)配置↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->
</beans:beans>

```