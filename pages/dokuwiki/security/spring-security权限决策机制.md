title: spring-security权限决策机制 

#  Spring-Security权限决策机制 
权限
**所有的Authentication实现类都保存了一个GrantedAuthority列表，其表示用户所具有的权限。**
` GrantedAuthority `是通过` AuthenticationManager `设置到` Authentication `对象中的，然后` AccessDecisionManager `将从Authentication中获取用户所具有的GrantedAuthority来鉴定用户是否具有访问对应资源的权限。
GrantedAuthority是一个接口，其中只定义了一个getAuthority()方法，其返回值为String类型。该方法允许AccessDecisionManager获取一个能够精确代表该权限的字符串。通过返回一个字符串，一个GrantedAuthority能够很轻易的被大部分AccessDecisionManager读取。如果一个GrantedAuthority不能够精确的使用一个String来表示，那么其对应的getAuthority()方法调用应当返回一个null，这表示AccessDecisionManager必须对该GrantedAuthority的实现有特定的支持，从而可以获取该GrantedAuthority所代表的权限信息。
Spring Security内置了一个GrantedAuthority的实现，` SimpleGrantedAuthority `。它直接接收一个表示权限信息的字符串，然后getAuthority()方法直接返回该字符串。Spring Security内置的所有AuthenticationProvider都是使用它来封装Authentication对象的。
 
**Spring Security的权限鉴定是由AccessDecisionManager接口负责的。**具体来说是由其中的decide()方法负责，其定义如下。
```

void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) throws AccessDeniedException, InsufficientAuthenticationException;

```
如你所见，该方法接收三个参数，第一个参数是包含当前用户信息的Authentication对象；**第二个参数表示当前正在请求的受保护的对象**，基本上来说是**MethodInvocation（使用AOP）、JoinPoint（使用Aspectj）和FilterInvocation（Web请求）三种类型**；第三个参数表示**与当前正在访问的受保护对象的配置属性，如一个角色列表。**
 
Spring Security的AOP Advice思想
对于使用AOP而言，我们可以使用几种不同类型的advice：before、after、throws和around。其中around advice是非常实用的，通过它我们可以控制是否要执行方法、是否要修改方法的返回值，以及是否要抛出异常。**Spring Security在对方法调用和Web请求时也是使用的around advice的思想。**在方法调用时，可以使用标准的Spring AOP来达到around advice的效果，而在进行Web请求时是通过标准的Filter来达到around advice的效果。
对于大部分人而言都比较喜欢对Service层的方法调用进行权限控制，因为我们的主要业务逻辑都是在Service层进行实现的。**如果你只是想保护Service层的方法，那么使用Spring AOP就可以了。**如果你需要直接保护领域对象，那么你可以考虑使用Aspectj。
你可以选择使用Aspectj或Spring AOP对方法调用进行鉴权，或者选择使用Filter对Web请求进行鉴权。当然，你也可以选择使用这三种方式的任意组合进行鉴权。
**通常的做法是使用Filter对Web请求进行一个比较粗略的鉴权，辅以使用Spring AOP对Service层的方法进行较细粒度的鉴权。**

##  AbstractSecurityInterceptor 
` AbstractSecurityInterceptor `是一个**实现了对受保护对象的访问进行拦截的抽象类**，其中有几个比较重要的方法。beforeInvocation()方法实现了对访问受保护对象的权限校验，内部用到了` AccessDecisionManager `和` AuthenticationManager `；finallyInvocation()方法用于实现受保护对象请求完毕后的一些清理工作，主要是如果在beforeInvocation()中改变了SecurityContext，则在finallyInvocation()中需要将其恢复为原来的SecurityContext，该方法的调用应当包含在子类请求受保护资源时的finally语句块中；afterInvocation()方法实现了对返回结果的处理，在注入了AfterInvocationManager的情况下默认会调用其decide()方法。AbstractSecurityInterceptor只是提供了这几种方法，并且包含了默认实现，具体怎么调用将由子类负责。
**每一种受保护对象都拥有继承自AbstractSecurityInterceptor的拦截器类， MethodSecurityInterceptor将用于调用受保护的方法，而FilterSecurityInterceptor将用于受保护的Web请求**。它们在处理受保护对象的请求时都具有一致的逻辑，具体的逻辑如下。
1、先将正在请求调用的受保护对象传递给beforeInvocation()方法**进行权限鉴定。**
2、权限鉴定失败就直接抛出异常了。
3、鉴定成功将尝试调用受保护对象，调用完成后，不管是成功调用，还是抛出异常，**都将执行finallyInvocation()。**
4、如果在调用受保护对象后没有抛出异常，则调用afterInvocation()。
以下是MethodSecurityInterceptor在进行方法调用的一段核心代码。
```

public Object invoke(MethodInvocation mi) throws Throwable {
        InterceptorStatusToken token = super.beforeInvocation(mi);
 
        Object result;
        try {
            result = mi.proceed();
        } finally {
            super.finallyInvocation(token);
        }
        returnsuper.afterInvocation(token, result);
    }

```
###  ConfigAttribute 
AbstractSecurityInterceptor的beforeInvocation()方法内部在进行鉴权的时候使用的是**注入的AccessDecisionManager的decide()方法进行的**。如前所述，` decide()方法是需要接收一个受保护对象对应的ConfigAttribute集合的。一个ConfigAttribute可能只是一个简单的角色名称，具体将视AccessDecisionManager的实现者而定。 `
AbstractSecurityInterceptor将使用一个` SecurityMetadataSource `对象**来获取与受保护对象关联的ConfigAttribute集合**，具体SecurityMetadataSource将由子类实现提供。

` ConfigAttribute将通过注解的形式定义在受保护的方法上，或者通过access属性定义在受保护的URL上。 `
```

例如我们常见的<intercept-url pattern=”/**” access=”ROLE_USER,ROLE_ADMIN”/>

```
就表示将ConfigAttribute ROLE_USER和ROLE_ADMIN应用在所有的URL请求上。对于默认的AccessDecisionManager的实现，上述配置意味着用户所拥有的权限中只要拥有一个GrantedAuthority与这两个ConfigAttribute中的一个进行匹配则允许进行访问。当然，**严格的来说ConfigAttribute只是一个简单的配置属性而已，具体的解释将由AccessDecisionManager来决定。**

##  调用前的处理 
 Spring Security是通过拦截器来控制受保护对象的访问的，如方法调用和Web请求。在正式访问受保护对象之前，Spring Security将使用AccessDecisionManager来鉴定当前用户是否有访问对应受保护对象的权限。

##  AccessDecisionManager访问决策管理器 
` AccessDecisionManager是由AbstractSecurityInterceptor调用的 `，它**负责鉴定用户是否有访问对应资源（方法或URL）的权限**。AccessDecisionManager是一个接口，其中只定义了三个方法，其定义如下。
```

public interface AccessDecisionManager {
 
    /**
     * 通过传递的参数来决定用户是否有访问对应受保护对象的权限
     *
     * @param authentication 当前正在请求受包含对象的Authentication
     * @param object 受保护对象，其可以是一个MethodInvocation、JoinPoint或FilterInvocation。
     * @param configAttributes 与正在请求的受保护对象相关联的配置属性
     *
     */
    void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
        throws AccessDeniedException, InsufficientAuthenticationException;
 
    /**
     * 表示当前AccessDecisionManager是否支持对应的ConfigAttribute
     */
    boolean supports(ConfigAttribute attribute);
 
    /**
     * 表示当前AccessDecisionManager是否支持对应的受保护对象类型
     */
    boolean supports(Class<?> clazz);
}

```
decide()方法用于决定authentication是否符合受保护对象要求的configAttributes。supports(ConfigAttribute attribute)方法是用来判断AccessDecisionManager是否能够处理对应的ConfigAttribute的。supports(Class<?> clazz)方法用于判断配置的AccessDecisionManager是否支持对应的受保护对象类型。
 
##  基于投票的AccessDecisionManager实现 
Spring Security已经内置了几个基于投票的AccessDecisionManager，当然如果需要你也可以实现自己的AccessDecisionManager。以下是Spring Security官方文档提供的一个图，其展示了与基于投票的AccessDecisionManager实现相关的类。
![](/data/dokuwiki/security/pasted/20160103-224241.png)
使用这种方式，一系列的` AccessDecisionVoter `将会被AccessDecisionManager用来对Authentication是否有权访问受保护对象**进行投票**，然后再根据投票结果来决定是否要抛出` AccessDeniedException `。AccessDecisionVoter是一个接口，其中定义有三个方法，具体结构如下所示。
```

public interface AccessDecisionVoter<S> {
 
    intACCESS_GRANTED = 1;//通过
    intACCESS_ABSTAIN = 0; //弃权
    intACCESS_DENIED = -1;//反对
 
    boolean supports(ConfigAttribute attribute);
 
    boolean supports(Class<?> clazz);
 
    int vote(Authentication authentication, S object, Collection<ConfigAttribute> attributes);
}

```
vote()方法的返回结果会是AccessDecisionVoter中定义的三个常量之一。ACCESS_GRANTED表示同意，ACCESS_DENIED表示返回，ACCESS_ABSTAIN表示弃权。如果一个AccessDecisionVoter不能判定当前Authentication是否拥有访问对应受保护对象的权限，则其vote()方法的返回值应当为弃权ACCESS_ABSTAIN。

Spring Security**内置了三个基于投票的AccessDecisionManager实现类，它们分别是AffirmativeBased、ConsensusBased和UnanimousBased。**
` AffirmativeBased `的逻辑是这样的：
（1）只要有一个AccessDecisionVoter的投票为ACCESS_GRANTED则同意用户进行访问；
（2）如果全部弃权也表示通过；
（3）如果没有一个人投赞成票，但是有人投反对票，则将抛出AccessDeniedException。

` ConsensusBased `的逻辑是这样的：
（1）如果**赞成票多于反对票则表示通过**。
（2）反过来，如果反对票多于赞成票则将抛出AccessDeniedException。
（3）如果赞成票与反对票相同且不等于0，并且属性allowIfEqualGrantedDeniedDecisions的值为true，则表示通过，否则将抛出异常AccessDeniedException。参数allowIfEqualGrantedDeniedDecisions的值默认为true。
（4）如果所有的AccessDecisionVoter都弃权了，则将视参数allowIfAllAbstainDecisions的值而定，如果该值为true则表示通过，否则将抛出异常AccessDeniedException。参数allowIfAllAbstainDecisions的值默认为false。

` UnanimousBased `的逻辑与另外两种实现有点不一样，另外两种会一次性把受保护对象的配置属性全部传递给AccessDecisionVoter进行投票，
而UnanimousBased会一次只传递一个ConfigAttribute给AccessDecisionVoter进行投票。这也就意味着如果我们的AccessDecisionVoter的逻辑是只要传递进来的ConfigAttribute中有一个能够匹配则投赞成票，但是放到UnanimousBased中其投票结果就不一定是赞成了。UnanimousBased的逻辑具体来说是这样的：
（1）如果受保护对象配置的某一个ConfigAttribute被任意的AccessDecisionVoter反对了，则将抛出AccessDeniedException。
（2）如果没有反对票，但是有赞成票，则表示通过。
（3）如果全部弃权了，则将视参数allowIfAllAbstainDecisions的值而定，true则通过，false则抛出AccessDeniedException。
 
###  RoleVoter 
` RoleVoter是Spring Security内置的一个AccessDecisionVoter `，其会将ConfigAttribute简单的看作是一个角色名称，在投票的时如果拥有该角色即投赞成票。` 如果ConfigAttribute是以“ROLE_”开头的，则将使用RoleVoter进行投票。 `当用户拥有的权限中有一个或多个能匹配受保护对象配置的` 以“ROLE_”开头 `的ConfigAttribute时其将投赞成票；如果用户拥有的权限中没有一个能匹配受保护对象配置的` 以“ROLE_”开头 `的ConfigAttribute，则RoleVoter将投反对票；如果受保护对象配置的ConfigAttribute中没有以“ROLE_”开头的，则RoleVoter将弃权。
 
###  AuthenticatedVoter 
**AuthenticatedVoter也是Spring Security内置的一个AccessDecisionVoter实现**。其主要用来**区分匿名用户、通过Remember-Me认证的用户和完全认证的用户。**完全认证的用户是指由系统提供的登录入口进行成功登录认证的用户。
AuthenticatedVoter可以处理的ConfigAttribute有IS_AUTHENTICATED_FULLY、IS_AUTHENTICATED_REMEMBERED和IS_AUTHENTICATED_ANONYMOUSLY。如果ConfigAttribute不在这三者范围之内，则AuthenticatedVoter将弃权。否则将视ConfigAttribute而定，如果ConfigAttribute为IS_AUTHENTICATED_ANONYMOUSLY，则不管用户是匿名的还是已经认证的都将投赞成票；如果是IS_AUTHENTICATED_REMEMBERED则仅当用户是由Remember-Me自动登录，或者是通过登录入口进行登录认证时才会投赞成票，否则将投反对票；而当ConfigAttribute为IS_AUTHENTICATED_FULLY时仅当用户是通过登录入口进行登录的才会投赞成票，否则将投反对票。
AuthenticatedVoter是通过AuthenticationTrustResolver的isAnonymous()方法和isRememberMe()方法来判断SecurityContextHolder持有的Authentication是否为AnonymousAuthenticationToken或RememberMeAuthenticationToken的，即是否为IS_AUTHENTICATED_ANONYMOUSLY和IS_AUTHENTICATED_REMEMBERED。
 
###  自定义Voter 
当然，用户也可以通过实现AccessDecisionVoter来实现自己的投票逻辑。
##  调用后的处理 
AccessDecisionManager是用来在访问受保护的对象之前判断用户是否拥有访问该对象的权限。有的时候我们可能会希望在请求执行完成后对返回值做一些修改，当然，你可以简单的通过AOP来实现这一功能。Spring Security为我们提供了一个` AfterInvocationManager `接口，它允许我们在受保护对象访问完成后**对返回值进行修改或者进行权限鉴定，**看是否需要抛出AccessDeniedException，其将由AbstractSecurityInterceptor的子类进行调用。AfterInvocationManager接口的定义如下。
```

public interface AfterInvocationManager {
  
    Object decide(Authentication authentication, Object object, Collection<ConfigAttribute> attributes,
        Object returnedObject) throws AccessDeniedException;
 
    boolean supports(ConfigAttribute attribute);
 
    boolean supports(Class<?> clazz);
}

```
以下是Spring Security官方文档提供的一个AfterInvocationManager构造实现的图。
![](/data/dokuwiki/security/pasted/20160103-225039.png)

##  角色的继承 
 对于角色继承这种需求也是经常有的，比如要求ROLE_ADMIN将拥有所有的ROLE_USER所具有的权限。当然我们可以给拥有ROLE_ADMIN角色的用户同时授予ROLE_USER角色来达到这一效果或者修改需要ROLE_USER进行访问的资源使用ROLE_ADMIN也可以访问。
Spring Security为我们提供了一种更为简便的办法，那就是` 角色的继承 `，它允许我们的ROLE_ADMIN直接继承ROLE_USER，这样所有ROLE_USER可以访问的资源ROLE_ADMIN也可以访问。
定义角色的继承我们需要在ApplicationContext中定义一个` RoleHierarchy `，然后再把它赋予给一个` RoleHierarchyVoter `，之后再把该RoleHierarchyVoter加入到我们**基于Voter的AccessDecisionManager中**，并指定当前使用的AccessDecisionManager为我们自己定义的那个。以下是一个定义角色继承的完整示例。
```

<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:security="http://www.springframework.org/schema/security"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
          http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security-3.2.xsd">
 
   <!-- 通过access-decision-manager-ref指定将要使用的AccessDecisionManager 
   <security:http access-decision-manager-ref="accessDecisionManager">-->
   <security:http>
      <security:form-login/>
      <security:intercept-url pattern="/admin.jsp" access="ROLE_ADMIN" />
      <security:intercept-url pattern="/**" access="ROLE_USER" />
   </security:http>
 
   <security:authentication-manager alias="authenticationManager">
      <security:authentication-provider
         user-service-ref="userDetailsService"/>
   </security:authentication-manager>
 
   <bean id="userDetailsService"
      class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
      <property name="dataSource" ref="dataSource" />
   </bean>
   <!-- 自己定义AccessDecisionManager对应的bean 
   <bean id="accessDecisionManager" class="org.springframework.security.access.vote.AffirmativeBased">
      <property name="decisionVoters">
         <list>
            <ref local="roleVoter"/>
         </list>
      </property>
   </bean>
 -->
   <bean id="roleVoter"
      class="org.springframework.security.access.vote.RoleHierarchyVoter">
      <constructor-arg ref="roleHierarchy" />
   </bean>
   <bean id="roleHierarchy"
      class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl">
      <property name="hierarchy"><!-- 角色继承关系 -->
         <value>
            ROLE_ADMIN > ROLE_USER
         </value>
      </property>
   </bean>
 
</beans>

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

参考：http://haohaoxuexi.iteye.com/blog/2247057