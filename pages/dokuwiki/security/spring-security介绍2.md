title: spring-security介绍2 

#  Spring-Security登录流程 
Spring Security命名空间的引入可以简化我们的开发，它涵盖了大部分Spring Security常用的功能。它的设计是基于框架内大范围的依赖的，可以被划分为以下几块。
Ø  Web/Http 安全：这是最复杂的部分。通过建立filter和相关的service bean来实现框架的认证机制。当访问受保护的URL时会将用户引入登录界面或者是错误提示界面。
Ø  业务对象或者方法的安全：控制方法访问权限的。
Ø  AuthenticationManager：处理来自于框架其他部分的认证请求。
Ø  AccessDecisionManager：为Web或方法的安全提供访问决策。会注册一个默认的，但是我们也可以通过普通bean注册的方式使用自定义的AccessDecisionManager。
Ø  AuthenticationProvider：AuthenticationManager是通过它来认证用户的。
Ø  UserDetailsService：跟AuthenticationProvider关系密切，用来获取用户信息的。
我们没有建立上面的登录页面，为什么Spring Security会跳到上面的登录页面呢？这是` 我们设置http的auto-config=”true”时Spring Security自动为我们生成的。 `
 当指定http元素的auto-config=”true”时，就相当于如下内容的简写。
```

   <security:http>
      <security:form-login/>
      <security:http-basic/>
      <security:logout/>
   </security:http>

```
这些元素负责建立表单登录、基本的认证和登出处理。它们都可以通过指定对应的属性来改变它们的行为。

##  form-login元素介绍 
 http元素下的form-login元素是用来定义表单登录信息的。当我们什么属性都不指定的时候Spring Security会为我们生成一个默认的登录页面。如果不想使用默认的登录页面，我们可以指定自己的登录页面。
 
###  使用自定义登录页面 
自定义登录页面是通过` login-page属性 `来指定的。提到login-page我们不得不提另外几个属性。
Ø  username-parameter：表示登录时用户名使用的是哪个参数，默认是“j_username”。
Ø  password-parameter：表示登录时密码使用的是哪个参数，默认是“j_password”。
Ø  **login-processing-url：表示登录时提交的地址，默认是“/j-spring-security-check”。**这个只是Spring Security用来` 标记登录页面使用的提交地址，真正关于登录这个请求是不需要用户自己处理的。 `
所以，我们可以通过如下定义使Spring Security在需要用户登录时跳转到我们自定义的登录页面。
```

   <security:http auto-config="true">
      <security:form-login login-page="/login.jsp"
         login-processing-url="/login.do" username-parameter="username"
         password-parameter="password" />
      <!-- 表示匿名用户可以访问 -->
      <security:intercept-url pattern="/login.jsp" access="IS_AUTHENTICATED_ANONYMOUSLY"/>
      <security:intercept-url pattern="/**" access="ROLE_USER" />
   </security:http>

```
需要注意的是，我们之前配置的是所有的请求都需要ROLE_USER权限，这意味着我们自定义的“/login.jsp”也需要该权限，这样就会形成一个死循环了。解决办法是我们需要给“/login.jsp”放行。通过指定“/login.jsp”的访问权限为“IS_AUTHENTICATED_ANONYMOUSLY”或“ROLE_ANONYMOUS”可以达到这一效果。此外，我们也可以通过指定一个http元素的安全性为none来达到相同的效果。如：
```

   <security:http security="none" pattern="/login.jsp" />
   <security:http auto-config="true">
      <security:form-login login-page="/login.jsp"
         login-processing-url="/login.do" username-parameter="username"
         password-parameter="password" />
      <security:intercept-url pattern="/**" access="ROLE_USER" />
   </security:http>

```
它们两者的区别是**前者将进入Spring Security定义的一系列用于安全控制的filter，而后者不会。当指定一个http元素的security属性为none时，表示其对应pattern的filter链为空**。从3.1开始，Spring Security**允许我们定义多个http元素以满足针对不同的pattern请求使用不同的filter链**。当为指定pattern属性时表示对应的http元素定义将对所有的请求发生作用。
根据上面的配置，我们自定义的登录页面的内容应该是这样子的：
```

 <form action="login.do" method="post">
      <table>
         <tr>
            <td>用户名：</td>
            <td><input type="text" name="username"/></td>
         </tr>
         <tr>
            <td>密码：</td>
            <td><input type="password" name="password"/></td>
         </tr>
         <tr>
            <td colspan="2" align="center">
                <input type="submit" value="登录"/>
                <input type="reset" value="重置"/>
            </td>
         </tr>
      </table>
   </form>

```
 
###  指定登录后的页面 
通过` default-target-url `指定
**默认情况下，我们在登录成功后会返回到原本受限制的页面。**但如果用户是**直接请求**登录页面，登录成功后应该跳转到哪里呢？**默认情况下它会跳转到当前应用的根路径，即欢迎页面**。通过指定form-login元素的default-target-url属性，我们可以让用户在直接登录后跳转到指定的页面。如果想让用户不管是直接请求登录页面，还是通过Spring Security引导过来的，登录之后都跳转到指定的页面，我们可以通过指定form-login元素的` always-use-default-target `属性为true来达到这一效果。
 
通过` authentication-success-handler-ref `指定
authentication-success-handler-ref对应一个` AuthencticationSuccessHandler `实现类的引用。如果指定了authentication-success-handler-ref，登录认证成功后会调用指定AuthenticationSuccessHandler的onAuthenticationSuccess方法。我们需要在该方法体内对认证成功做一个处理，然后返回对应的认证成功页面。使用了authentication-success-handler-ref之后认证成功后的处理就由指定的AuthenticationSuccessHandler来处理，之前的那些default-target-url之类的就都不起作用了。
以下是自定义的一个AuthenticationSuccessHandler的实现类。
```

publicclass AuthenticationSuccessHandlerImpl implements
      AuthenticationSuccessHandler {
 
   publicvoid onAuthenticationSuccess(HttpServletRequest request,
         HttpServletResponse response, Authentication authentication)
         throws IOException, ServletException {
      response.sendRedirect(request.getContextPath());
   }
 
}

```
其对应使用authentication-success-handler-ref属性的配置是这样的：
```

   <security:http auto-config="true">
      <security:form-login login-page="/login.jsp"
         login-processing-url="/login.do" username-parameter="username"
         password-parameter="password"
         authentication-success-handler-ref="authSuccess"/>
      <!-- 表示匿名用户可以访问 -->
      <security:intercept-url pattern="/login.jsp"
         access="IS_AUTHENTICATED_ANONYMOUSLY" />
      <security:intercept-url pattern="/**" access="ROLE_USER" />
   </security:http>
   <!-- 认证成功后的处理类 -->
   <bean id="authSuccess" class="com.xxx.AuthenticationSuccessHandlerImpl"/>

```
 
###  指定登录失败后的页面 
除了可以指定登录认证成功后的页面和对应的AuthenticationSuccessHandler之外，form-login同样允许我们指定认证失败后的页面和对应认证失败后的处理器` AuthenticationFailureHandler。 `
通过` authentication-failure-url `指定
默认情况下登录失败后会返回登录页面，我们也可以通过form-login元素的authentication-failure-url来指定登录失败后的页面。需要注意的是登录失败后的页面跟登录页面一样也是需要配置成在未登录的情况下可以访问，否则登录失败后请求失败页面时又会被Spring Security重定向到登录页面。
```

 <security:http auto-config="true">
      <security:form-login login-page="/login.jsp"
         login-processing-url="/login.do" username-parameter="username"
         password-parameter="password"
         authentication-failure-url="/login_failure.jsp"
         />
      <!-- 表示匿名用户可以访问 -->
      <security:intercept-url pattern="/login*.jsp*"
         access="IS_AUTHENTICATED_ANONYMOUSLY" />
      <security:intercept-url pattern="/**" access="ROLE_USER" />
   </security:http>

```
 
通过` authentication-failure-handler-ref `指定
类似于authentication-success-handler-ref，authentication-failure-handler-ref对应一个用于处理认证失败的` AuthenticationFailureHandler `实现类。指定了该属性，Spring Security在认证失败后会调用指定AuthenticationFailureHandler的onAuthenticationFailure方法对认证失败进行处理，**此时authentication-failure-url属性将不再发生作用。**
