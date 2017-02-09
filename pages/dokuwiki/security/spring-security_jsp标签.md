title: spring-security_jsp标签 

#  Spring-Security JSP标签 
 Spring Security也有对Jsp标签的支持的标签库。其中一共定义了**三个标签：authorize、authentication和accesscontrollist。**其中authentication标签是用来代表当前Authentication对象的，我们可以利用它来展示当前Authentication对象的相关信息。另外两个标签是用于权限控制的，可以利用它们来包裹需要保护的内容，通常是超链接和按钮。
如果需要使用Spring Security的标签库，那么首先我们应当将对应的jar包spring-security-taglibs-xxx.jar放入WEB-INF/lib下；其次我们需要在页面上引入**Spring Security的标签库。**
```

<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>

```
接下来就可以在页面上自由的使用Spring Security的标签库提供的标签了。
1.1     authorize
**authorize是用来判断普通权限的，通过判断用户是否具有对应的权限而控制其所包含内容的显示**，其可以指定如下属性。
1、access
access属性需要使用表达式来判断权限，当表达式的返回结果为true时表示拥有对应的权限。
```

<sec:authorize access="hasRole('admin')">
      <a href="admin.jsp">admin page</a>
   </sec:authorize>

```
需要注意的是因为access属性是使用表达式的，所以我们必须确保ApplicationContext中存在一个WebSecurityExpressionHandler，最简单的办法就是直接使用NameSpace，通过设置http元素的use-expressions="true"让NameSpace自动为我们创建一个WebSecurityExpressionHandler。
 
2、ifAllGranted、ifAnyGranted和ifNotGranted
这三个属性的用法类似，它们都接收以逗号分隔的权限列表，且不能使用表达式。ifAllGranted表示需要包含所有的权限，ifAnyGranted表示只需要包含其中的任意一个即可，ifNotGranted表示不能包含指定的任意一个权限。
```

 <!-- 需要拥有所有的权限 -->
   <sec:authorize ifAllGranted="ROLE_ADMIN">
      <a href="admin.jsp">admin</a>
   </sec:authorize>
   <!-- 只需拥有其中任意一个权限 -->
   <sec:authorize ifAnyGranted="ROLE_USER,ROLE_ADMIN">hello</sec:authorize>
   <!-- 不允许拥有指定的任意权限 -->
   <sec:authorize ifNotGranted="ROLE_ADMIN">
      <a href="user.jsp">user</a>
   </sec:authorize>

```
 3、url
 url表示如果用户拥有访问指定url的权限即表示可以显示authorize标签包含的内容。
```

   <!-- 拥有访问指定url的权限才显示其中包含的内容 -->
   <sec:authorize url="/admin.jsp">
      <a href="admin.jsp">admin</a>
   </sec:authorize>

```
4、method
method属性是配合url属性一起使用的，表示用户应当具有指定url指定method访问的权限，method的默认值为GET，可选值为http请求的7种方法。
```

 <!-- 拥有访问指定url的权限才显示其中包含的内容 -->
   <sec:authorize url="/admin.jsp">
      <a href="admin.jsp">admin</a>
   </sec:authorize>
       限制访问方法是通过http元素下的intercept-url元素的method属性来指定的，如：
   <security:intercept-url pattern="/admin.jsp" access="ROLE_ADMIN" method="POST"/> 

``` 
 
5、var
 用于指定将权限鉴定的结果存放在pageContext的哪个属性中。该属性的主要作用是对于在同一页面的多个地方具有相同权限鉴定时，**我们只需要定义一次，然后将鉴定结果以var指定的属性名存放在pageContext中，其它地方可以直接使用之前的鉴定结果。**
```

   <sec:authorize access="isFullyAuthenticated()" var="isFullyAuthenticated">
   </sec:authorize>
   上述权限的鉴定结果是：${isFullyAuthenticated }<br/>
   <%if((Boolean)pageContext.getAttribute("isFullyAuthenticated")) {%>
      只有通过登录界面进行登录的用户才能看到2。
   <%}%>

```
 
各属性对应的优先级
既然我们可以通过属性access、url、ifAllGranted、ifAnyGranted等来指定应当具有的权限，那么当同时指定多个属性时，它们的作用效果是什么样的呢？authorize标签进行权限鉴定的属性根据优先级的不同可以分为三类，access为一类；url为一类；ifAllGranted、ifAnyGranted和ifNotGranted为一类。这三类将同时只有一类产生效果。它们的优先级如下：
 1、access具有最高的优先级，如果指定了access属性，那么将以access属性指定的表达式来鉴定当前用户是否有权限。不管结果如何，此时其它属性都将被忽略。
2、如果没有指定access属性，那么url属性将具有最高优先级，此时将直接通过url属性和method属性（默认为GET）来鉴定当前用户是否有权限。不管结果如何，此时都将忽略ifAllGranted、ifAnyGranted和ifNotGranted属性。
3、如果access和url都没有指定，那么将使用第三类属性来鉴定当前用户的权限。当第三类里面同时指定了多个属性时，它们将都发生效果，即必须指定的三类权限都满足才认为是有对应的权限。如ifAllGranted要求有ROLE_USER的权限，同时ifNotGranted要求不能有ROLE_ADMIN的权限，则结果是它们的并集，即只有拥有ROLE_USER权限，同时不拥有ROLE_ADMIN权限的用户才被允许获取指定的内容。
 
1.2     authentication
**authentication标签用来代表当前Authentication对象，主要用于获取当前Authentication的相关信息。**authentication标签的主要属性是property属性，我们可以通过它来获取当前Authentication对象的相关信息。如通常我们的Authentication对象中存放的principle是一个UserDetails对象，所以我们可以通过如下的方式来获取当前用户的用户名。
<sec:authentication property="principal.username"/>
当然，我们也可以直接通过Authentication的name属性来获取其用户名。
<sec:authentication property="name"/>
 property属性只允许指定Authentication所拥有的属性，可以进行属性的级联获取，如“principle.username”，不允许直接通过方法进行调用。
除了property属性之外，authentication还可以指定的属性有：var、scope和htmlScape。
var属性
var属性用于指定一个属性名，这样当获取到了authentication的相关信息后会将其以var指定的属性名进行存放，默认是存放在pageConext中。可以通过scope属性进行指定。此外，当指定了var属性后，authentication标签不会将获取到的信息在页面上进行展示，如需展示用户应该通过var指定的属性进行展示，或去掉var属性。
```

   <!-- 将获取到的用户名以属性名username存放在session中 -->
   <sec:authentication property="principal.username" scope="session" var="username"/>
   ${username }

```
 
scope属性
与var属性一起使用，用于指定存放获取的结果的属性名的作用范围，默认我pageContext。Jsp中拥有的作用范围都进行进行指定。
 
htmlScape属性
表示是否需要将html进行转义。默认为true。
 
1.3     accesscontrollist
accesscontrollist标签是用于鉴定ACL权限的。其一共定义了三个属性：hasPermission、domainObject和var，其中前两个是必须指定的。hasPermission属性用于指定以逗号分隔的权限列表；domainObject用于指定对应的域对象；而var则是用以将鉴定的结果以指定的属性名存入pageContext中，以供同一页面的其它地方使用。需要注意的是使用accesscontrollist标签时ApplicationContext中必须存在一个PermissionEvaluator bean，因为accesscontrollist标签就是通过PermissionEvaluator来鉴定对应的权限的。如果我们正在使用Spring Security的ACL模块，那么PermissionEvaluator通常就对应着AclPermissionEvaluator。此外，如果domainObject属性指定的domainObject为null则默认认为是有权限的，否则如果当前Authentication对象为null则默认认为是没有权限的。
```

   <sec:accesscontrollist hasPermission="1,2" domainObject="${someTargetDomainObject }" >
      如果当前Authentication对指定的domainObject拥有指定的hasPermission则将可以看到这部分内容。
   </sec:accesscontrollist>

```
   
参考：http://haohaoxuexi.iteye.com/blog/2263097