title: apache_shiro_jsp标签 

#  apache Shiro JSP标签 
Shiro提供了JSTL标签用于在JSP/GSP页面进行权限控制，如根据登录用户显示相应的页面按钮。
```

<%@taglib prefix="shiro" uri="http://shiro.apache.org/tags" %> 

``` 
标签库定义在shiro-web.jar包下的META-INF/shiro.tld中定义。
 
guest标签 
```

<shiro:guest>  
欢迎游客访问，<a href="${pageContext.request.contextPath}/login.jsp">登录</a>  
</shiro:guest>  

``` 
用户没有身份验证时显示相应信息，即游客访问信息。
 
user标签 
```

<shiro:user>  
欢迎[<shiro:principal/>]登录，<a href="${pageContext.request.contextPath}/logout">退出</a>  
</shiro:user>  

``` 
用户已经身份验证/记住我登录后显示相应的信息。
  
authenticated标签 
```

<shiro:authenticated>  
    用户[<shiro:principal/>]已身份验证通过  
</shiro:authenticated> 

```  
用户已经身份验证通过，**即Subject.login登录成功，不是记住我登录的。**    
 
notAuthenticated标签
```

<shiro:notAuthenticated>
    未身份验证（包括记住我）
</shiro:notAuthenticated>

``` 
**用户已经身份验证通过，**即没有调用Subject.login进行登录，**包括记住我自动登录的也属于未进行身份验证。** 
 
principal标签 
```

<shiro: principal/>

```
显示用户身份信息，默认调用Subject.getPrincipal()获取，即Primary Principal。 
 
hasRole标签 
```

<shiro:hasRole name="admin">  
    用户[<shiro:principal/>]拥有角色admin<br/>  
</shiro:hasRole> 

```  
如果当前Subject有角色将显示body体内容。
 
hasAnyRoles标签 
```

<shiro:hasAnyRoles name="admin,user">  
    用户[<shiro:principal/>]拥有角色admin或user<br/>  
</shiro:hasAnyRoles>  

``` 
如果当前Subject有任意一个角色（或的关系）将显示body体内容。 
 
lacksRole标签 
```

<shiro:lacksRole name="abc">  
    用户[<shiro:principal/>]没有角色abc<br/>  
</shiro:lacksRole>  

``` 
如果当前Subject没有角色将显示body体内容。 
  
hasPermission标签
```

<shiro:hasPermission name="user:create">  
    用户[<shiro:principal/>]拥有权限user:create<br/>  
</shiro:hasPermission>  

``` 
如果当前Subject有权限将显示body体内容。 
  
lacksPermission标签
```

<shiro:lacksPermission name="org:create">  
    用户[<shiro:principal/>]没有权限org:create<br/>  
</shiro:lacksPermission>  

``` 
如果当前Subject没有权限将显示body体内容。
 