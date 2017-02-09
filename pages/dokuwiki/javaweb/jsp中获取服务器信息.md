title: jsp中获取服务器信息 

#  JSP中获取服务器信息 
##  主要接口： 

request.getServletContext().getRealPath("/")  获取项目所在服务器的全路径，如：D:\Program Files\apache-tomcat-7.0.25\webapps\TestSytem\
request.getServletPath()    获取客户端请求的路径名，如：/object/delObject
request.getServerName()    获取服务器地址，如：localhost
request.getServerPort()    获取服务器端口，如8080
request.getContextPath()    获取项目名称，如：TestSytem
request.getLocalAddr()    获取本地地址，如：127.0.0.1
request.getLocalName()    获取本地IP映射名，如：localhost
request.getLocalPort()    获取本地端口，如：8090
request.getRealPath("/")    获取项目所在服务器的全路径，如：D:\Program Files\apache-tomcat-7.0.25\webapps\TestSytem\
request.getRemoteAddr()    获取远程主机地址，如：127.0.0.1
request.getRemoteHost()    获取远程主机，如：127.0.0.1
request.getRemotePort()    获取远程客户端端口，如：3623 
request.getRequestedSessionId()    获取会话session的ID，如：823A6BACAC64FB114235CBFE85A46CAA
request.getRequestURI()    获取包含项目名称的请求路径，如：/TestSytem/object/delObject
request.getRequestURL().toString()    获取请求的全路径，如：http://localhost:8090/TestSytem/object/delObject

##  获取部署的应用地址： 

```

El表达式的写法：${pageContext.request.contextPath}
jsp的写法：<%=request.getContextPath()%>

```
或者后台保存base属性传到JSP:
```

String path = request.getContextPath();  //如/hibernate-test
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  //如：http://localhost:8080/hibernate-test/

```
##  pageContext 
pageContext用来取得有关用户或页面的详细信息
     1>取得请求的参数字符串
         ${pageContext.request.queryString}
       或${pageContext["request"]["queryString"]}   <%--名值对的形式,值为对象下同--%>
     2>取得请求URL
         ${pageContext.request.requestURL}
     3>取得Web应用的名称
         ${pageContext.request.contextPath}
     4>取得HTTP请求方式(Post or Get)
         ${pageContext.reuqest.method}
     5>取得使用的协议
         ${pageContext.request.protocol}
     6>取得用户IP地址
         ${pageContext.request.remoteAddr}
     7>判断Session是否为新
         ${pageContext.session.new}
     8>取得SessionID
         ${pageContext.session.id}