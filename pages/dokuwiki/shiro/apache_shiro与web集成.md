title: apache_shiro与web集成 

#  apache Shiro与Web集成 
Shiro提供了与Web集成的支持，其通过一个` ShiroFilter `入口来**拦截需要安全控制的URL**，然后进行相应的控制，ShiroFilter类似于如Strut2/SpringMVC这种web框架的前端控制器，其是安全控制的入口点，其负责读取配置（如ini配置文件），然后判断URL是否需要登录/权限等工作。

**准备环境**
```

<dependency>  
    <groupId>javax.servlet</groupId>  
    <artifactId>javax.servlet-api</artifactId>  
    <version>3.0.1</version>  
    <scope>provided</scope>  
</dependency> 

<dependency>  
    <groupId>org.apache.shiro</groupId>  
    <artifactId>shiro-web</artifactId>  
    <version>1.2.2</version>  
</dependency>   

```  
其他依赖请参考源码的pom.xml。

**7.2 ShiroFilter入口**
2、Shiro 1.2及以后版本的配置方式
从Shiro 1.2开始引入了Environment/WebEnvironment的概念，即由它们的实现提供相应的SecurityManager及其相应的依赖。**ShiroFilter会自动找到Environment然后获取相应的依赖。**
```

<listener>  
   <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>  
</listener> 

```  
通过EnvironmentLoaderListener来创建相应的WebEnvironment，并自动绑定到ServletContext，默认使用IniWebEnvironment实现。
 
可以通过如下配置修改默认实现及其加载的配置文件位置：
```

<context-param>  
   <param-name>shiroEnvironmentClass</param-name>  
   <param-value>org.apache.shiro.web.env.IniWebEnvironment</param-value>  
</context-param>  
    <context-param>  
        <param-name>shiroConfigLocations</param-name>  
        <param-value>classpath:shiro.ini</param-value>  
    </context-param>  

``` 
**shiroConfigLocations默认是“/WEB-INF/shiro.ini”，**IniWebEnvironment默认是先从/WEB-INF/shiro.ini加载，如果没有就默认加载classpath:shiro.ini。
 
##  3、与Spring集成 
```

<filter>  
    <filter-name>shiroFilter</filter-name>  
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
    <init-param>  
        <param-name>targetFilterLifecycle</param-name>  
        <param-value>true</param-value>  
    </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>shiroFilter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>  

``` 
DelegatingFilterProxy作用是自动到spring容器查找名字为shiroFilter（filter-name）的bean并把所有Filter的操作委托给它。然后将ShiroFilter配置到spring容器即可：
```

<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">  
<property name="securityManager" ref="securityManager"/>  
<!—忽略其他，详见与Spring集成部分 -->  
</bean>  

``` 
最后不要忘了使用org.springframework.web.context.ContextLoaderListener加载这个spring配置文件即可。

##  Web INI配置 
ini配置部分和之前的相比将多出对url部分的配置。     
```

[main]  
#默认是/login.jsp  
authc.loginUrl=/login  
roles.unauthorizedUrl=/unauthorized  
perms.unauthorizedUrl=/unauthorized  
[users]  
zhang=123,admin  
wang=123  
[roles]  
admin=user:*,menu:*  
[urls]  
/login=anon  
/unauthorized=anon  
/static/**=anon  
/authenticated=authc  
/role=authc,roles[admin]  
/permission=authc,perms["user:create"]  

``` 
其中最重要的就是[urls]部分的配置，其格式是： “url=拦截器[参数]，拦截器[参数]”；即如果当前请求的url匹配[urls]部分的某个url模式，将会执行其配置的拦截器。比如anon拦截器表示匿名访问（即不需要登录即可访问）；authc拦截器表示需要身份认证通过后才能访问；roles[admin]拦截器表示需要有admin角色授权才能访问；而perms["user:create"]拦截器表示需要有“user:create”权限才能访问。
 
url模式使用Ant风格模式
```

Ant路径通配符支持?、*、**，注意通配符匹配不包括目录分隔符“/”：
?：匹配一个字符，如”/admin?”将匹配/admin1，但不匹配/admin或/admin2；
*：匹配零个或多个字符串，如/admin*将匹配/admin、/admin123，但不匹配/admin/1；
**：匹配路径中的零个或多个路径，如/admin/**将匹配/admin/a或/admin/a/b。

```
 
**url模式匹配顺序**
url模式匹配顺序是按照在配置中的声明顺序匹配，即从头开始使用第一个匹配的url模式对应的拦截器链。如：
拦截器将在下一节详细介绍。接着我们来看看身份验证、授权及退出在web中如何实现。
 
###  1、身份验证（登录） 
1.1、首先配置需要身份验证的url  
```

/authenticated=authc  
/role=authc,roles[admin]  
/permission=authc,perms["user:create"]  

``` 
**即访问这些地址时会首先判断用户有没有登录，如果没有登录默会跳转到登录页面，默认是/login.jsp，**可以通过在[main]部分通过如下配置修改： 
```

authc.loginUrl=/login 

```
登录Servlet（com.github.zhangkaitao.shiro.chapter7.web.servlet.LoginServlet）
```

@WebServlet(name = "loginServlet", urlPatterns = "/login")  
public class LoginServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        req.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(req, resp);  
    }  
    @Override  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        String error = null;  
        String username = req.getParameter("username");  
        String password = req.getParameter("password");  
        Subject subject = SecurityUtils.getSubject();  
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);  
        try {  
            subject.login(token);  
        } catch (UnknownAccountException e) {  
            error = "用户名/密码错误";  
        } catch (IncorrectCredentialsException e) {  
            error = "用户名/密码错误";  
        } catch (AuthenticationException e) {  
            //其他错误，比如锁定，如果想单独处理请单独catch处理  
            error = "其他错误：" + e.getMessage();  
        }  
        if(error != null) {//出错了，返回登录页面  
            req.setAttribute("error", error);  
            req.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(req, resp);  
        } else {//登录成功  
            req.getRequestDispatcher("/WEB-INF/jsp/loginSuccess.jsp").forward(req, resp);  
        }  
    }  
}  

``` 
1、doGet请求时展示登录页面；
2、doPost时进行登录，登录时收集username/password参数，然后提交给Subject进行登录。如果有错误再返回到登录页面；否则跳转到登录成功页面（此处应该返回到访问登录页面之前的那个页面，或者没有上一个页面时访问主页）。
3、JSP页面请参考源码。
 
1.3、测试
首先输入http://localhost:8080/chapter7/login进行登录，登录成功后接着可以访问http://localhost:8080/chapter7/authenticated来显示当前登录的用户： 
${subject.principal}身份验证已通过。  
 
##  Shiro内置了登录（身份验证）的实现： 
基于表单的和基于Basic的验证，其通过拦截器实现。
基于表单的拦截器身份验证
基于表单的拦截器身份验证和【1】类似，但是更简单，因为其已经实现了大部分登录逻辑；我们只需要指定：登录地址/登录失败后错误信息存哪/成功的地址即可。
  
###  3.1、shiro-formfilterlogin.ini  
```

[main]  
authc.loginUrl=/formfilterlogin  
authc.usernameParam=username  
authc.passwordParam=password  
authc.successUrl=/  
authc.failureKeyAttribute=shiroLoginFailure  
  
[urls]  
/role=authc,roles[admin]  

``` 
1、` authc `是org.apache.shiro.web.filter.authc.**FormAuthenticationFilter类型的实例**，其用于**实现基于表单的身份验证**；
  * 通过loginUrl指定当身份验证时的登录表单；
  * usernameParam指定登录表单提交的用户名参数名；passwordParam指定登录表单提交的密码参数名；
  * successUrl指定登录成功后重定向的默认地址（默认是“/”）（如果有上一个地址会自动重定向带该地址）；
  * failureKeyAttribute指定登录失败时的request属性key（默认shiroLoginFailure）；**这样可以在登录表单得到该错误key显示相应的错误消息；**

3.2、web.xml
把shiroConfigLocations改为shiro- formfilterlogin.ini即可。
 
3.3、登录Servlet 
```

@WebServlet(name = "formFilterLoginServlet", urlPatterns = "/formfilterlogin")  
public class FormFilterLoginServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        doPost(req, resp);  
    }  
    @Override  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)  
     throws ServletException, IOException {  
        String errorClassName = (String)req.getAttribute("shiroLoginFailure");  
        if(UnknownAccountException.class.getName().equals(errorClassName)) {  
            req.setAttribute("error", "用户名/密码错误");  
        } else if(IncorrectCredentialsException.class.getName().equals(errorClassName)) {  
            req.setAttribute("error", "用户名/密码错误");  
        } else if(errorClassName != null) {  
            req.setAttribute("error", "未知错误：" + errorClassName);  
        }  
        req.getRequestDispatcher("/WEB-INF/jsp/formfilterlogin.jsp").forward(req, resp);  
    }  
} 

``` 
**在登录Servlet中通过shiroLoginFailure得到authc登录失败时的异常类型名，然后根据此异常名来决定显示什么错误消息**。
 
4、测试
**输入http://localhost:8080/chapter7/role，会跳转到“/formfilterlogin”登录表单，
提交表单如果authc拦截器登录成功后，会直接重定向会之前的地址“/role”；
假设我们直接访问“/formfilterlogin”的话登录成功将直接到默认的successUrl。**

##  授权（角色/权限验证） 
4.1、shiro.ini   
```

[main]  
roles.unauthorizedUrl=/unauthorized  
perms.unauthorizedUrl=/unauthorized  
 [urls]  
/role=authc,roles[admin]  
/permission=authc,perms["user:create"]  

``` 
通过` unauthorizedUrl `属性指定如果**授权失败时重定向到的地址**。roles是org.apache.shiro.web.filter.authz.**RolesAuthorizationFilter类型的实例**，通过参数指定访问时需要的角色，如“[admin]”，如果有多个使用“，”分割，且验证时是hasAllRole验证，即且的关系。Perms是org.apache.shiro.web.filter.authz.**PermissionsAuthorizationFilter类型的实例**，和roles类似，只是验证权限字符串。
 
4.2、web.xml
把shiroConfigLocations改为shiro.ini即可。
 
4.3、RoleServlet/PermissionServlet  
```

@WebServlet(name = "permissionServlet", urlPatterns = "/permission")  
public class PermissionServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        Subject subject = SecurityUtils.getSubject();  
        subject.checkPermission("user:create");  
        req.getRequestDispatcher("/WEB-INF/jsp/hasPermission.jsp").forward(req, resp);  
    }  
} 

``` 
```

@WebServlet(name = "roleServlet", urlPatterns = "/role")  
public class RoleServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        Subject subject = SecurityUtils.getSubject();  
        subject.checkRole("admin");  
        req.getRequestDispatcher("/WEB-INF/jsp/hasRole.jsp").forward(req, resp);  
    }  
}  

``` 
 
4.4、测试
首先访问http://localhost:8080/chapter7/login，使用帐号“zhang/123”进行登录，再访问/role或/permission时会跳转到成功页面（因为其授权成功了）；如果使用帐号“wang/123”登录成功后访问这两个地址会跳转到“/unauthorized”即没有授权页面。

 
##  5、退出 
5.1、shiro.ini 
```

[urls]  
/logout=anon  

``` 
**指定/logout使用anon拦截器即可，**即不需要登录即可访问。
5.2、LogoutServlet
```

@WebServlet(name = "logoutServlet", urlPatterns = "/logout")  
public class LogoutServlet extends HttpServlet {  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        SecurityUtils.getSubject().logout();  
        req.getRequestDispatcher("/WEB-INF/jsp/logoutSuccess.jsp").forward(req, resp);  
    }  
}  

``` 
直接调用Subject.logout即可，退出成功后转发/重定向到相应页面即可。
 
5.3、测试
首先访问http://localhost:8080/chapter7/login，使用帐号“zhang/123”进行登录，登录成功后访问/logout即可退出。
Shiro也提供了logout拦截器用于退出，其是org.apache.shiro.web.filter.authc**.LogoutFilter类型的实例**，我们可以在shiro.ini配置文件中通过如下配置完成退出：
```

[main]  
logout.redirectUrl=/login  
  
[urls]  
/logout2=logout

```   
通过logout.redirectUrl指定退出后重定向的地址；通过/logout2=logout指定退出url是/logout2。这样当我们登录成功后然后访问/logout2即可退出。

