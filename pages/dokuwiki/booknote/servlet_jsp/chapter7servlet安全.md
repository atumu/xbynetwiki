title: chapter7servlet安全 

#  chapter7:servlet安全 
原文：http://jinnianshilongnian.iteye.com/blog/1766303
本质描述了Servlet容器安全机制、接口、部署描述符和基于注解机制用于传达应用安全需求。
尽管质量保障和实现细节可能会有所不同，但servlet容器有满足这些需求的机制和基础设施，共用如下一些特性：
■ 身份认证：表示通信实体彼此证明他们具体身份的行为是被授权访问的。
■ 资源访问控制：表示和资源的交互是受限于集合的用户或为了强制完整性、保密性、或可用性约束的程序。
■ 数据完整性：表示用来证明信息在传输过程中没有被第三方修改。
■ 保密或数据隐私：表示用来保证信息只对以授权访问的用户可用。
##  声明式安全 
声明式安全是指以在应用外部的形式表达应用的安全模型需求，包括**角色、访问控制和认证需求**。部署描述符是web应用中的声明式安全的主要手段。
部署人员映射应用的逻辑安全需求到特定于运行时环境的安全策略的表示。在运行时，servlet容器使用安全策略表示来实施认证和授权。
安全模型适用于web应用的静态内容部分和客户端请求到的应用内的servlet和过滤器。` 安全模型不适用于当servlet使用RequestDispatcher调用静态内容或使用forward或include到的servlet。 `
##  编程式安全 
当仅仅声明式安全是不足以表达应用的安全模型时，编程式安全被用于意识到安全的应用。
编程式安全包括以下**HttpServletRequest**接口的方法：
■ authenticate
■ login
■ logout
■ getRemoteUser
■ isUserInRole
■ getUserPrincipal
login方法允许应用执行用户名和密码收集（作为一种Form-Based Login的替代）。authenticate方法允许应用由容器发起在一个不受约束的请求上下文内的来访者请求认证。
logout方法提供用于应用重置来访者的请求身份。
getRemoteUser方法由容器返回与该请求相关的远程用户（即来访者）的名字。
isUserInRole方法确定是否与该请求相关的远程用户（即来访者）在一个特定的安全角色中。
getUserPrincipal方法确定远程用户（即来访者）的Principal名称并返回一个与远程用户相关的java.security.Principal对象。调用getUserPrincipal返回的Principal的getName方法返回远程用户的名字。这些API允许Servlet基于获得的信息做一些业务逻辑决策。
如果没有用户通过身份认证，getRemoteUser方法返回null，isUserInRole方法总返回false，getUserPrincipal方法总返回null。
```

<security-role-ref>  
  <role-name>FOO</role-name>  
  <role-link>manager</role-link>  
</security-role-ref>   

```
在这种情况下，如果属于“manager”安全角色的用户调用了servlet，则调用isUserInRole("FOO") API的结果是true。
##  编程式访问控制注解 

本章定义的注解和API提供用于配置Servlet容器强制的安全约束。
**@ServletSecurity注解**
@ServletSecurity提供了用于定义访问控制约束的另一种机制，相当于那些通过在便携式部署描述符中声明式或通过ServletRegistration接口的setServletSecurity方法编程式表示。Servlet容器必须支持在实现javax.servlet.Servlet 接口的类（和它的子类）上使用@ServletSecurity注解。
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-175149.png)
**@HttpConstraint**
@HttpConstraint注解用在@ServletSecurity中表示应用到所有HTTP协议方法的安全约束，且HTTP协议方法对应的@HttpMethodConstraint没有出现在@ServletSecurity注解中。
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-175230.png)
**@HttpMethodConstraint**
@HttpMethodConstraint注解用在@ServletSecurity注解中表示在特定HTTP协议消息上的安全约束。
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-175309.png)
##   示例 
```

//用于所有HTTP方法，且无约束
@ServletSecurity  
public class Example1 extends HttpServlet {  
}  
//用于所有HTTP方法，无认证约束，需要加密传输
@ServletSecurity(@HttpConstraint(transportGuarantee = TransportGuarantee.CONFIDENTIAL))  
public class Example2 extends HttpServlet {  
}
//用于所有HTTP方法，所有访问拒绝   
@ServletSecurity(@HttpConstraint(EmptyRoleSemantic.DENY))  
public class Example3 extends HttpServlet {  
}  
//用于所有HTTP方法，认证约束需要成员身份在角色“R1”中
@ServletSecurity(@HttpConstraint(rolesAllowed = "R1"))  
public class Example4 extends HttpServlet {  
} 
//用于除GET和POST之外的所有HTTP方法，无约束；对于GET和POST方法， //认证约束需要成员身份在角色“R1”中；对于POST，需要加密传输
@ServletSecurity((httpMethodConstraints = {  
        @HttpMethodConstraint(value = "GET", rolesAllowed = "R1"),  
        @HttpMethodConstraint(value = "POST", rolesAllowed = "R1",  
        transportGuarantee = TransportGuarantee.CONFIDENTIAL)  
    })  
public class Example5 extends HttpServlet {  
} 
//用于除了GET之外的所有HTTP方法，认证约束需要成员身份在“R1”角色中；对于GET，无约束
@ServletSecurity(value = @HttpConstraint(rolesAllowed = "R1"),  
        httpMethodConstraints = @HttpMethodConstraint("GET"))  
public class Example6 extends HttpServlet {  
}  


```
##  XML声明方式 
```

@ServletSecurity(@HttpConstraint(rolesAllowed = "Role1"))  
  
<security-constraint>  
  <web-resource-collection>  
    <url-pattern>...</url-pattern>  
  </web-resource-collection>  
  <auth-constraint>  
    <role-name>Role1</role-name>  
  </auth-constraint>  
</security-constraint> 

//
@ServletSecurity(value=@HttpConstraint(rolesAllowed = "Role1"),  
httpMethodConstraints = @HttpMethodConstraint(value = "TRACE",  
emptyRoleSemantic = EmptyRoleSemantic.DENY))  
   
<security-constraint>  
  <web-resource-collection>  
    <url-pattern>...</url-pattern>  
    <http-method-omission>TRACE</http-method-omission>  
  </web-resource-collection>  
  <auth-constraint>  
    <role-name>Role1</role-name>  
  </auth-constraint>  
</security-constraint>  
<security-constraint>  
  <web-resource-collection>  
    <url-pattern>...</url-pattern>  
    <http-method>TRACE</http-method>  
  </web-resource-collection>  
  <auth-constraint/>  
</security-constraint> 

//
<user-data-constraint>  
  <transport-guarantee>CONFIDENTIAL</transport-guarantee>  
</user-data-constraint>  
//
<auth-constraint>  
  <security-role-name>Role1</security-role-name>  
</auth-constraint>  
//
<auth-constraint>  
  <security-role-name>Role1</security-role-name>  
</auth-constraint>  
<user-data-constraint>  
  <transport-guarantee>CONFIDENTIAL</transport-guarantee>  
</user-data-constraint>  

<auth-constraint/>  
<user-data-constraint>  
  <transport-guarantee>CONFIDENTIAL</transport-guarantee>  
</user-data-constraint>  

```
##  角色 

安全角色是由应用开发人员或装配人员定义的逻辑用户分组。当部署了应用，由部署人员映射角色到运行时环境的principal或组。
Servlet容器根据principal的安全属性为与进入请求相关的principal实施声明式或编程式安全。
这可能以如下任一方式发生：
1. 部署人员已经映射一个安全角色到运行环境中的一个用户组。调用的principal 所属的用户组取自其安全属性。仅当principal 所属的用户组由部署人员已经映射了安全角色，principal是在安全角色中。
2. 部署人员已经映射安全角色到安全策略域中的principal 名字。在这种情况下，调用的principal的名字取自其安全属性。仅当principal 名字与安全角色已映射到的principal 名字一样时，principal 是在安全角色中。

##  认证 
web客户端可以使用以下机制之一向web服务器认证用户身份：
■ HTTP基本认证（HTTP Basic Authentication）:HTTP基本认证基于用户名和密码.Web服务器请求web客户端认证用户。作为请求的一部分，web服务器传递realm（字符串）给要被认证的用户。Web客户端获取用户的用户名和密码并传给web服务器。Web服务器然后在指定的**realm**认证用户.基本认证是不安全的认证协议。**用户密码以简单的base64编码发送**，且未认证目标服务器。额外的保护可以减少一些担忧：**安全传输机制（HTTPS）**，或者网络层安全（如IPSEC协议或VPN策略）被应用到一些部署场景。
■ HTTP摘要认证（HTTP Digest Authentication）:HTTP摘要认证也是基于用户名和密码认证用户，但不像HTTP基本认证，HTTP摘要认证不在网络上发生用户密码。在HTTP摘要认证中，客户端发送单向散列的密码（和额外的数据）。Servlet容器应支持HTTP_DIGEST身份认证。
■ HTTPS客户端认证（HTTPS Client Authentication）:**使用HTTPS（HTTP over SSL）认证**最终用户是一种强认证机制。**该机制需要客户端拥有Public Key Certificate（PKC）**。目前，PKCs在电子商务应用中是很有用的，也对浏览器中的单点登录很有用。
■ 基于表单的认证（Form Based Authentication）:Web应用部署描述符包含登录表单和错误页面条目。登录界面必须包含用于输入用户名和密码的字段。这些字段必须分别命名为` j_username和j_password `。
当用户试图访问一个受保护的web资源，容器坚持用户的认证。如果用户已经通过认证则具有访问资源的权限，请求的web资源被激活并返回一个引用。如果用户未被认证，发生所有如下步骤：
1. 与安全约束关联的登录界面被发送到客户端，且由容器存储触发认证的URL路径。
2. 用户被要求填写表单，包括用户名和密码字段。
3. 客户端post表单到服务器。
4. 容器尝试使用来自表单的信息认证用户。
5. 如果认证失败，使用forward或redirect返回错误页面，且响应状态码设置为200。
6. 如果授权成功，检查被验证用户的principal，看其是否是以一个授权的角色访问资源。
7. 如果用户被授权，客户端使用存储的URL路径重定向回资源。
如果用户没有被授权，包含失败信息的错误页面发送给用户。
` 基于表单的认证与基本认证具有同样的缺点，因为用户密码以纯文本传输且未认证目标服务器 `。同样，额外的保护可以减少一些担忧：安全传输机制（HTTPS），或者网络层安全（如IPSEC协议或VPN策略）被应用到一些部署场景。
HttpServletRequest接口的login方法提供另一种用于应用控制它的登录界面外观的手段。
```

<form method=”POST” action=”j_security_check”>  
  <input type=”text” name=”j_username”>  
  <input type=”password” name=”j_password”>  
</form> 

```
##  指定安全约束 
**安全约束是一种定义web内容保护的声明式方式**。安全约束关联授权和或在web资源上对HTTP操作的用户数据约束。安全约束，在部署描述符中由**security-constraint**表示，其包含以下元素：
■ web资源集合 (部署描述符中的web-resource-collection)
■ 授权约束 (部署描述符中的auth-constraint)
■ 用户数据约束 (部署描述符中的user-data-constraint)
HTTP操作和网络资源的安全约束应用(即受限的请求)根据一个或多个web资源集合识别。Web资源集合包含以下元素：
■ URL 模式 (部署描述符中的url-pattern)
■ HTTP methods (部署描述符中的http-method或http-method-omission元素)
授权约束规定认证和命名执行受约束请求的被许可的授权角色的要求。用户必须至少是许可执行受约束请求的命名角色中的一个成员。特殊角色名“*”是定义在部署描述符中的所有角色名的一种简写。没有指定角色的授权约束表示在任何情况下不允许访问受约束请求。授权约束包含以下元素：

■ role name (部署描述符中的role-name)
用户数据约束规定了在受保护的传输层连接之上接收受约束的请求的要求。需要保护的强度由传输保障的值定义。INTEGRAL类型的传输保障用于规定内容完整性要求，且传输保障CONFIDENTIAL用于规定保密性要求的。传输保障“NONE”表示当容器通过任何包括不受保护的连接接受到请求时，必须接受此受约束的请求。用户数据约束包括如下元素：

■ transport guarantee (部署描述符中的transport-guarantee)
如果没有授权约束应用到请求，容器必须接受请求，而不要求用户身份认证。如果没有用户数据约束应用到请求，当容器通过任何包括不受保护的连接接收到请求时，必须接受此请求。

###   组合约束 

为了组合约束，HTTP方法可以说是存在于**web-resource-collection**中，仅当没有在集合中指定HTTP方法，或者集合在包含的http-method元素中具体指定了HTTP方法，或者集合包含一个或多个http-method-omission元素，但那些没有指定的HTTP方法。
当url-pattern和HTTP方法以组合方式（即，在web-resource-collection中）出现在多个安全约束中，该约束（在模式和方法上的）是通过合并单个约束定义的。以相同的模式和方法出现的组合约束规则如下所示：

授权约束组合，其明确指定角色或通过“*” 隐式指定角色，可产生单个约束的合并的角色名称作为许可的角色。不包含授权约束的安全约束将与明确指定角色的或隐式指定角色的允许未授权访问的安全约束合并。授权约束的一个特殊情况是其没有指定角色，将与任何其他约束合并并覆盖它们的作用，这导致访问被阻止。

应用到常见的url-pattern和http-method的user-data-constraint组合，可产生合并的单个约束接受的连接类型作为接受的连接类型。不包含user-data-constraint的安全约束，将与其他user-data-constraint合并，使不安全的连接类型是可接受的连接类型。

###  示例 

下面的示例演示了组合约束及它们翻译到的可应用的约束表格。假设部署描述符包含如下安全约束。
```

<security-constraint>   
    <web-resource-collection>  
        <web-resource-name>precluded methods</web-resource-name>  
        <url-pattern>/*</url-pattern>  
        <url-pattern>/acme/wholesale/*</url-pattern>  
        <url-pattern>/acme/retail/*</url-pattern>  
        <http-method-omission>GET</http-method-omission>  
        <http-method-omission>POST</http-method-omission>  
    </web-resource-collection>  
    <auth-constraint/>  
</security-constraint>  
  
<security-constraint>  
    <web-resource-collection>  
        <web-resource-name>wholesale</web-resource-name>  
        <url-pattern>/acme/wholesale/*</url-pattern>  
        <http-method>GET</http-method>  
        <http-method>PUT</http-method>  
    </web-resource-collection>  
    <auth-constraint>  
        <role-name>SALESCLERK</role-name>  
    </auth-constraint>  
</security-constraint>  
<security-constraint>  
    <web-resource-collection>  
        <web-resource-name>wholesale 2</web-resource-name>  
        <url-pattern>/acme/wholesale/*</url-pattern>  
        <http-method>GET</http-method>  
        <http-method>POST</http-method>  
    </web-resource-collection>  
    <auth-constraint>  
        <role-name>CONTRACTOR</role-name>  
    </auth-constraint>  
    <user-data-constraint>  
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>  
    </user-data-constraint>  
</security-constraint>  
<security-constraint>  
    <web-resource-collection>  
        <web-resource-name>retail</web-resource-name>  
        <url-pattern>/acme/retail/*</url-pattern>  
        <http-method>GET</http-method>  
        <http-method>POST</http-method>  
    </web-resource-collection>  
    <auth-constraint>  
        <role-name>CONTRACTOR</role-name>  
        <role-name>HOMEOWNER</role-name>  
    </auth-constraint>  
</security-constraint>  

```
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-181459.png)
###  处理请求 

当servlet容器接收到一个请求，它将使用119页“使用URL路径”描述的规则来选择在请求URI最佳匹配的url-pattern上定义的约束（如果有）。如果没有约束被选择，容器将接受该请求。否则，容器将确定在选择的模式上是否此请求的HTTP方法是受约束的。如果不是，请求将被接受。否则，请求必须满足在urlpattern应用到HTTP方法的约束。请求被接受和分派到相关的servlet，必须满足以下两个规则。

1. 接收到的请求的连接特性必须满足至少一种由约束定义的支持的连接类型。如果该规则不满足，容器将拒绝该请求并重定向到HTTPS端口。（作为一种优化，容器将以拒绝该请求为forbidden 并返回403 (SC_FORBIDDEN)状态码，如果知道该访问将最终将被阻止 (通过没有指定角色的授权约束)）

2. 请求的认证特性必须满足任何由约束定义的认证和角色要求。如果该规则不能满足是因为访问已经被阻止（通过没有指定角色的授权约束），则请求将被拒绝为forbidden 并返回403 (SC_FORBIDDEN)状态码。如果访问是受限于许可的角色且请求还没有被认证，则请求将被拒绝为unauthorized 且401(SC_UNAUTHORIZED)状态码将被返回以导致身份认证。如果访问是受限于许可的角色且请求的认证身份不是这些角色中的成员，则请求将被拒绝为forbidden 且403状态码(SC_FORBIDDEN)将被返回到用户。

##  默认策略 

默认情况下，身份认证并不需要访问资源。当安全约束（如果有）包含的url-pattern是请求URI的最佳匹配，且结合了施加在请求的HTTP方法上的auth-constraint（指定的角色），则身份认证是需要的。同样，一个受保护的传输是不需要的，除非应用到请求的安全约束结合了施加在请求的HTTP方法上的user-data-constraint（有一个受保护的transport-guarantee）。

##  登录和退出 

容器在分派请求到servlet引擎之前建立调用者身份。在整个请求处理过程中或直到应用成功的在请求上调用身份认证、登录或退出，调用者身份保持不变。对于异步请求，调用者身份建立在初始分派时，直到整个请求处理完成或直到应用成功的在请求上调用身份认证、登录或退出，调用者身份保持不变。

在处理请求时登录到一个应用，精确地对应有一个有效的非空的与请求关联的调用者身份，可以通过调用请求的getRemoteUser或getUserPrincipal确定。这些方法的任何一个返回null值表示调用者没有登录到处理请求的应用。

容器可以创建HTTP Session对象用于跟踪登录状态。如果开发人员创建一个session而用户没有进行身份认证，然后容器认证用户，登录后，对开发人员代码可见的session必须是相同的session对象，该session是登录发生之前创建的，以便不丢失session信息。

##  技巧一：禁止直接访问JSP资源 
```

<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">
	<security-constraint>
		<web-resource-collection>
			<web-resource-name>JSP Page</web-resource-name>
			<url-pattern>*.jsp</url-pattern>
		</web-resource-collection>
		<!--禁止直接访问 -->
		<auth-constraint/>
	</security-constraint>

</web-app>

```
##  基本访问认证 
```

<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0"
>
    <!-- restricts access to JSP pages -->
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>JSP pages</web-resource-name>
            <url-pattern>*.jsp</url-pattern>
        </web-resource-collection>
        <!-- must have auth-constraint, otherwise the
            specified web resources will not be restricted -->
        <auth-constraint/>
    </security-constraint>
    
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Servlet1</web-resource-name>
            <url-pattern>/servlet1</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>member</role-name>
            <role-name>manager</role-name>
        </auth-constraint>
    </security-constraint>

    <login-config>
        <auth-method>BASIC</auth-method>
        <realm-name>Members Only</realm-name>
    </login-config>
</web-app>

```
##  摘要访问验证 
```

<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0"
>
    <!-- restricts access to JSP pages -->
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>JSP pages</web-resource-name>
            <url-pattern>*.jsp</url-pattern>
        </web-resource-collection>
        <!-- must have auth-constraint, otherwise the
            specified web resources will not be restricted -->
        <auth-constraint/>
    </security-constraint>

    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Servlet1</web-resource-name>
            <url-pattern>/servlet1</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>member</role-name>
            <role-name>manager</role-name>
        </auth-constraint>
    </security-constraint>
    
    <login-config>
        <auth-method>DIGEST</auth-method>
        <realm-name>Digest authentication</realm-name>
    </login-config>
</web-app>

```
##  基于表单验证 
```

<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0"
>
    <!-- restricts access to JSP pages-->
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>JSP pages</web-resource-name>
            <url-pattern>*.jsp</url-pattern>
        </web-resource-collection>
        <!-- must have auth-constraint, otherwise the
            specified web resources will not be restricted -->
        <auth-constraint/>
    </security-constraint>

    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Servlet1</web-resource-name>
            <url-pattern>/servlet1</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>member</role-name>
            <role-name>manager</role-name>
        </auth-constraint>
    </security-constraint>
    
    <login-config>
        <auth-method>FORM</auth-method>
        <form-login-config>
            <form-login-page>/login.html</form-login-page>
            <form-error-page>/error.html</form-error-page>
        </form-login-config>
    </login-config>
</web-app>

```
