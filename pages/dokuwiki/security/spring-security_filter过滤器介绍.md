title: spring-security_filter过滤器介绍 

#  Spring-Security Filter过滤器介绍 
 Spring Security的底层是通过一系列的Filter来管理的，每个Filter都有其自身的功能，而且各个Filter在功能上还有关联关系，所以**它们的顺序也是非常重要的。**
1.1     Filter顺序
Spring Security已经定义了一些Filter，不管实际应用中你用到了哪些，它们应当保持如下顺序。
（1）ChannelProcessingFilter，如果你访问的channel错了，那首先就会在channel之间进行跳转，如http变为https。
（2）SecurityContextPersistenceFilter，这样的话在一开始进行request的时候就可以在SecurityContextHolder中建立一个SecurityContext，然后在请求结束的时候，任何对SecurityContext的改变都可以被copy到HttpSession。
（3）ConcurrentSessionFilter，因为它需要使用SecurityContextHolder的功能，而且更新对应session的最后更新时间，以及通过SessionRegistry获取当前的SessionInformation以检查当前的session是否已经过期，过期则会调用LogoutHandler。
（4）认证处理机制，如UsernamePasswordAuthenticationFilter，CasAuthenticationFilter，BasicAuthenticationFilter等，以至于SecurityContextHolder可以被更新为包含一个有效的Authentication请求。
（5）SecurityContextHolderAwareRequestFilter，它将会把HttpServletRequest封装成一个继承自HttpServletRequestWrapper的SecurityContextHolderAwareRequestWrapper，同时使用SecurityContext实现了HttpServletRequest中与安全相关的方法。
（6）JaasApiIntegrationFilter，如果SecurityContextHolder中拥有的Authentication是一个JaasAuthenticationToken，那么该Filter将使用包含在JaasAuthenticationToken中的Subject继续执行FilterChain。
（7）RememberMeAuthenticationFilter，如果之前的认证处理机制没有更新SecurityContextHolder，并且用户请求包含了一个Remember-Me对应的cookie，那么一个对应的Authentication将会设给SecurityContextHolder。
（8）AnonymousAuthenticationFilter，如果之前的认证机制都没有更新SecurityContextHolder拥有的Authentication，那么一个AnonymousAuthenticationToken将会设给SecurityContextHolder。
（9）ExceptionTransactionFilter，用于处理在FilterChain范围内抛出的AccessDeniedException和AuthenticationException，并把它们转换为对应的Http错误码返回或者对应的页面。
（10）FilterSecurityInterceptor，保护Web URI，并且在访问被拒绝时抛出异常。
 
1.2     添加Filter到FilterChain
当我们在使用NameSpace时，Spring Security是会自动为我们建立对应的FilterChain以及其中的Filter。但有时我们可能需要添加我们自己的Filter到FilterChain，又或者是因为某些特性需要自己显示的定义Spring Security已经为我们提供好的Filter，然后再把它们添加到FilterChain。使用NameSpace时添加Filter到FilterChain是通过http元素下的custom-filter元素来定义的。定义custom-filter时需要我们通过ref属性指定其对应关联的是哪个Filter，此外还需要通过position、before或者after指定该Filter放置的位置。诚如在上一节《Filter顺序》中所提到的那样，Spring Security对FilterChain中Filter顺序是有严格的规定的。Spring Security对那些内置的Filter都指定了一个别名，同时指定了它们的位置。我们在定义custom-filter的position、before和after时使用的值就是对应着这些别名所处的位置。如position=”CAS_FILTER”就表示将定义的Filter放在CAS_FILTER对应的那个位置，before=”CAS_FILTER”就表示将定义的Filter放在CAS_FILTER之前，after=”CAS_FILTER”就表示将定义的Filter放在CAS_FILTER之后。此外还有两个特殊的位置可以指定，FIRST和LAST，分别对应第一个和最后一个Filter，如你想把定义好的Filter放在最后，则可以使用after=”LAST”。
接下来我们来看一下Spring Security给我们定义好的FilterChain中Filter对应的位置顺序、它们的别名以及将触发自动添加到FilterChain的元素或属性定义。下面的定义是按顺序的。

1.3     DelegatingFilterProxy
可能你会觉得奇怪，我们在web应用中使用Spring Security时只在web.xml文件中定义了如下这样一个Filter，为什么你会说是一系列的Filter呢？
```

   <filter>
      <filter-name>springSecurityFilterChain</filter-name>
     <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
   </filter>
   <filter-mapping>
      <filter-name>springSecurityFilterChain</filter-name>
      <url-pattern>/*</url-pattern>
   </filter-mapping>

```
 而且如果你不在web.xml文件声明要使用的Filter，那么Servlet容器将不会发现它们，它们又怎么发生作用呢？这就是上述配置中DelegatingFilterProxy的作用了。
DelegatingFilterProxy是Spring中定义的一个Filter实现类，其作用是代理真正的Filter实现类，也就是说在调用DelegatingFilterProxy的doFilter()方法时实际上调用的是其代理Filter的doFilter()方法。其代理Filter必须是一个Spring bean对象，所以使用DelegatingFilterProxy的好处就是其代理Filter类可以使用Spring的依赖注入机制方便自由的使用ApplicationContext中的bean。那么DelegatingFilterProxy如何知道其所代理的Filter是哪个呢？这**是通过其自身的一个叫targetBeanName的属性来确定的，通过该名称，DelegatingFilterProxy可以从WebApplicationContext中获取指定的bean作为代理对象**。该属性可以通过在web.xml中定义DelegatingFilterProxy时通过init-param来指定，**如果未指定的话将默认取其在web.xml中声明时定义的名称。**
`  在上述配置中，DelegatingFilterProxy代理的就是名为SpringSecurityFilterChain的Filter。 `
需要注意的是被代理的Filter的初始化方法init()和销毁方法destroy()默认是不会被执行的。通过设置DelegatingFilterProxy的targetFilterLifecycle属性为true，可以使被代理Filter与DelegatingFilterProxy具有同样的生命周期。
 
1.4     FilterChainProxy
Spring Security底层是通过一系列的Filter来工作的，每个Filter都有其各自的功能，而且各个Filter之间还有关联关系，所以它们的组合顺序也是非常重要的。
使用Spring Security时，DelegatingFilterProxy代理的就是一个FilterChainProxy。一个FilterChainProxy中可以包含有多个FilterChain，但是某个请求只会对应一个FilterChain，而一个FilterChain中又可以包含有多个Filter。当我们使用基于Spring Security的NameSpace进行配置时，系统会自动为我们注册一个名为springSecurityFilterChain类型为FilterChainProxy的bean（这也是为什么我们在使用SpringSecurity时需要在web.xml中声明一个name为springSecurityFilterChain类型为DelegatingFilterProxy的Filter了。），而且每一个http元素的定义都将拥有自己的FilterChain，而FilterChain中所拥有的Filter则会根据定义的服务自动增减。所以我们不需要显示的再定义这些Filter对应的bean了，除非你想实现自己的逻辑，又或者你想定义的某个属性NameSpace没有提供对应支持等。
Spring security允许我们在配置文件中配置多个http元素，以针对不同形式的URL使用不同的安全控制。Spring Security将会为每一个http元素创建对应的FilterChain，同时按照它们的声明顺序加入到FilterChainProxy。所以当我们同时定义多个http元素时要确保将更具有特性的URL配置在前。
```

   <security:http pattern="/login*.jsp*" security="none"/>
   <!-- http元素的pattern属性指定当前的http对应的FilterChain将匹配哪些URL，如未指定将匹配所有的请求 -->
   <security:http pattern="/admin/**">
      <security:intercept-url pattern="/**" access="ROLE_ADMIN"/>
   </security:http>
   <security:http>
      <security:intercept-url pattern="/**" access="ROLE_USER"/>
   </security:http>

```
需要注意的是http拥有一个匹配URL的pattern，未指定时表示匹配所有的请求，其下的子元素intercept-url也有一个匹配URL的pattern，该pattern是在http元素对应pattern基础上的，也就是说一个请求必须先满足http对应的pattern才有可能满足其下intercept-url对应的pattern。
 
1.5     Spring Security定义好的核心Filter
通过前面的介绍我们知道Spring Security是通过Filter来工作的，为保证Spring Security的顺利运行，其内部实现了一系列的Filter。这其中有几个是在使用Spring Security的Web应用中必定会用到的。接下来我们来简要的介绍一下FilterSecurityInterceptor、ExceptionTranslationFilter、SecurityContextPersistenceFilter和UsernamePasswordAuthenticationFilter。在我们使用http元素时前三者会自动添加到对应的FilterChain中，当我们使用了form-login元素时UsernamePasswordAuthenticationFilter也会自动添加到FilterChain中。所以我们在利用custom-filter往FilterChain中添加自己定义的这些Filter时需要注意它们的位置。
 
1.5.1   FilterSecurityInterceptor
**FilterSecurityInterceptor是用于保护Http资源的，它需要一个AccessDecisionManager和一个AuthenticationManager的引用。它会从SecurityContextHolder获取Authentication，然后通过SecurityMetadataSource可以得知当前请求是否在请求受保护的资源。**对于请求那些受保护的资源，如果Authentication.isAuthenticated()返回false或者FilterSecurityInterceptor的alwaysReauthenticate属性为true，那么将会使用其引用的**AuthenticationManager再认证**一次，认证之后再使用认证后的Authentication替换SecurityContextHolder中拥有的那个。然后就是**利用AccessDecisionManager进行权限的检查。**
我们在使用基于NameSpace的配置时所配置的intercept-url就会跟FilterChain内部的FilterSecurityInterceptor绑定。如果要自己定义FilterSecurityInterceptor对应的bean，那么该bean定义大致如下所示：
```

   <bean id="filterSecurityInterceptor"
   class="org.springframework.security.web.access.intercept.FilterSecurityInterceptor">
      <property name="authenticationManager" ref="authenticationManager" />
      <property name="accessDecisionManager" ref="accessDecisionManager" />
      <property name="securityMetadataSource">
         <security:filter-security-metadata-source>
            <security:intercept-url pattern="/admin/**" access="ROLE_ADMIN" />
            <security:intercept-url pattern="/**" access="ROLE_USER,ROLE_ADMIN" />
         </security:filter-security-metadata-source>
      </property>
   </bean>

```
filter-security-metadata-source用于配置其securityMetadataSource属性。intercept-url用于配置需要拦截的URL与对应的权限关系。
 
1.5.2   ExceptionTranslationFilter
通过前面的介绍我们知道在Spring Security的Filter链表中ExceptionTranslationFilter就放在FilterSecurityInterceptor的前面。而ExceptionTranslationFilter是捕获来自FilterChain的异常，并对这些异常做处理。ExceptionTranslationFilter能够捕获来自FilterChain所有的异常，但是它只会处理两类异常，AuthenticationException和AccessDeniedException，其它的异常它会继续抛出。如果捕获到的是AuthenticationException，那么将会使用其对应的AuthenticationEntryPoint的commence()处理。如果捕获的异常是一个AccessDeniedException，那么将视当前访问的用户是否已经登录认证做不同的处理，如果未登录，则会使用关联的AuthenticationEntryPoint的commence()方法进行处理，否则将使用关联的AccessDeniedHandler的handle()方法进行处理。
AuthenticationEntryPoint是在用户没有登录时用于引导用户进行登录认证的，在实际应用中应根据具体的认证机制选择对应的AuthenticationEntryPoint。
AccessDeniedHandler用于在用户已经登录了，但是访问了其自身没有权限的资源时做出对应的处理。ExceptionTranslationFilter拥有的AccessDeniedHandler默认是AccessDeniedHandlerImpl，其会返回一个403错误码到客户端。我们可以通过显示的配置AccessDeniedHandlerImpl，同时给其指定一个errorPage使其可以返回对应的错误页面。当然我们也可以实现自己的AccessDeniedHandler。
```

   <bean id="exceptionTranslationFilter"
      class="org.springframework.security.web.access.ExceptionTranslationFilter">
      <property name="authenticationEntryPoint">
         <bean class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
            <property name="loginFormUrl" value="/login.jsp" />
         </bean>
      </property>
      <property name="accessDeniedHandler">
         <bean class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
            <property name="errorPage" value="/access_denied.jsp" />
         </bean>
      </property>
   </bean>

```
在上述配置中我们指定了AccessDeniedHandler为AccessDeniedHandlerImpl，同时为其指定了errorPage，这样发生AccessDeniedException后将转到对应的errorPage上。指定了AuthenticationEntryPoint为使用表单登录的LoginUrlAuthenticationEntryPoint。此外，需要注意的是如果该filter是作为自定义filter加入到由NameSpace自动建立的FilterChain中时需把它放在内置的ExceptionTranslationFilter后面，否则异常都将被内置的ExceptionTranslationFilter所捕获。
```

   <security:http>
      <security:form-login login-page="/login.jsp"
         username-parameter="username" password-parameter="password"
         login-processing-url="/login.do" />
      <!-- 退出登录时删除session对应的cookie -->
      <security:logout delete-cookies="JSESSIONID" />
      <!-- 登录页面应当是不需要认证的 -->
      <security:intercept-url pattern="/login*.jsp*"
         access="IS_AUTHENTICATED_ANONYMOUSLY" />
      <security:intercept-url pattern="/**" access="ROLE_USER" />
      <security:custom-filter ref="exceptionTranslationFilter" after="EXCEPTION_TRANSLATION_FILTER"/>
   </security:http>

```
在捕获到AuthenticationException之后，调用AuthenticationEntryPoint的commence()方法引导用户登录之前，ExceptionTranslationFilter还做了一件事，那就是使用RequestCache将当前HttpServletRequest的信息保存起来，以至于用户成功登录后需要跳转到之前的页面时可以获取到这些信息，然后继续之前的请求，比如用户可能在未登录的情况下发表评论，待用户提交评论的时候就会将包含评论信息的当前请求保存起来，同时引导用户进行登录认证，待用户成功登录后再利用原来的request包含的信息继续之前的请求，即继续提交评论，所以待用户登录成功后我们通常看到的是用户成功提交了评论之后的页面。Spring Security默认使用的RequestCache是HttpSessionRequestCache，其会将HttpServletRequest相关信息封装为一个SavedRequest保存在HttpSession中。
 
1.5.3   SecurityContextPersistenceFilter
**SecurityContextPersistenceFilter会在请求开始时从配置好的SecurityContextRepository中获取SecurityContext，然后把它设置给SecurityContextHolder。在请求完成后将SecurityContextHolder持有的SecurityContext再保存到配置好的SecurityContextRepository，同时清除SecurityContextHolder所持有的SecurityContext。**在使用NameSpace时，Spring  Security默认会给SecurityContextPersistenceFilter的SecurityContextRepository设置一个HttpSessionSecurityContextRepository，其会将SecurityContext保存在HttpSession中。此外HttpSessionSecurityContextRepository有一个很重要的属性allowSessionCreation，默认为true。这样需要把SecurityContext保存在session中时，如果不存在session，可以自动创建一个。也可以把它设置为false，这样在请求结束后如果没有可用的session就不会保存SecurityContext到session了。SecurityContextRepository还有一个空实现，NullSecurityContextRepository，如果在请求完成后不想保存SecurityContext也可以使用它。
这里再补充说明一点为什么SecurityContextPersistenceFilter在请求完成后需要清除SecurityContextHolder的SecurityContext。SecurityContextHolder在设置和保存SecurityContext都是使用的静态方法，具体操作是由其所持有的SecurityContextHolderStrategy完成的。默认使用的是基于线程变量的实现，即SecurityContext是存放在ThreadLocal里面的，这样各个独立的请求都将拥有自己的SecurityContext。在请求完成后清除SecurityContextHolder中的SucurityContext就是清除ThreadLocal，Servlet容器一般都有自己的线程池，这可以避免Servlet容器下一次分发线程时线程中还包含SecurityContext变量，从而引起不必要的错误。
下面是一个SecurityContextPersistenceFilter的简单配置。
```

   <bean id="securityContextPersistenceFilter"
   class="org.springframework.security.web.context.SecurityContextPersistenceFilter">
      <property name='securityContextRepository'>
         <bean
         class='org.springframework.security.web.context.HttpSessionSecurityContextRepository'>
            <property name='allowSessionCreation' value='false' />
         </bean>
      </property>
   </bean>

```
 
1.5.4   UsernamePasswordAuthenticationFilter
UsernamePasswordAuthenticationFilter用于处理来自表单提交的认证。该表单必须提供对应的用户名和密码，对应的参数名默认为j_username和j_password。如果不想使用默认的参数名，可以通过UsernamePasswordAuthenticationFilter的usernameParameter和passwordParameter进行指定。表单的提交路径默认是“j_spring_security_check”，也可以通过UsernamePasswordAuthenticationFilter的filterProcessesUrl进行指定。通过属性postOnly可以指定只允许登录表单进行post请求，默认是true。其内部还有登录成功或失败后进行处理的AuthenticationSuccessHandler和AuthenticationFailureHandler，这些都可以根据需求做相关改变。此外，它还需要一个AuthenticationManager的引用进行认证，这个是没有默认配置的。
```

   <bean id="authenticationFilter"
   class="org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter">
      <property name="authenticationManager" ref="authenticationManager" />
      <property name="usernameParameter" value="username"/>
      <property name="passwordParameter" value="password"/>
      <property name="filterProcessesUrl" value="/login.do" />
   </bean>

```
 如果要在http元素定义中使用上述AuthenticationFilter定义，那么完整的配置应该类似于如下这样子。
```

<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:security="http://www.springframework.org/schema/security"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
          http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security-3.1.xsd">
   <!-- entry-point-ref指定登录入口 -->
   <security:http entry-point-ref="authEntryPoint">
      <security:logout delete-cookies="JSESSIONID" />
      <security:intercept-url pattern="/login*.jsp*"
         access="IS_AUTHENTICATED_ANONYMOUSLY" />
      <security:intercept-url pattern="/**" access="ROLE_USER" />
      <!-- 添加自己定义的AuthenticationFilter到FilterChain的FORM_LOGIN_FILTER位置 -->
      <security:custom-filter ref="authenticationFilter" position="FORM_LOGIN_FILTER"/>
   </security:http>
   <!-- AuthenticationEntryPoint，引导用户进行登录 -->
   <bean id="authEntryPoint" class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
      <property name="loginFormUrl" value="/login.jsp"/>
   </bean>
   <!-- 认证过滤器 -->
   <bean id="authenticationFilter"
   class="org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter">
      <property name="authenticationManager" ref="authenticationManager" />
      <property name="usernameParameter" value="username"/>
      <property name="passwordParameter" value="password"/>
      <property name="filterProcessesUrl" value="/login.do" />
   </bean>
  
   <security:authentication-manager alias="authenticationManager">
      <security:authentication-provider
         user-service-ref="userDetailsService">
         <security:password-encoder hash="md5"
            base64="true">
            <security:salt-source user-property="username" />
         </security:password-encoder>
      </security:authentication-provider>
   </security:authentication-manager>
 
   <bean id="userDetailsService"
      class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
      <property name="dataSource" ref="dataSource" />
   </bean>
 
</beans>

```
原文：http://haohaoxuexi.iteye.com/blog/2161648