title: jstl标签 

#  EL运算符与JSTL标签 
参考：http://www.cnblogs.com/lihuiyy/archive/2012/02/24/2366806.html
http://blog.csdn.net/h396071018/article/details/6663412
http://www.cnblogs.com/lihuiyy/archive/2012/02/15/2352643.html
采用Apache taglib实现https://www.apache.org/dist/tomcat/taglibs/taglibs-standard-1.2.5/README_bin.txt
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <%@ taglib prefix="x" uri="http://java.sun.com/jsp/jstl/xml" %>
    <%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
    <%@ taglib prefix="sql" uri="http://java.sun.com/jsp/jstl/sql" %>
    <%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
    
##  JSTL 核心标签库 
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
JSTL 核心标签库标签共有13个，功能上分为4类：
1.表达式控制标签：out、set、remove、catch
2.流程控制标签：if、choose、when、otherwise
3.循环标签：forEach、forTokens
4.URL操作标签：import、url、redirect

1. ` <c:out> ` 用来显示数据对象（字符串、表达式）的内容或结果
<th><c:out value="${person.uid}" escapeXml="true" default="默认值"/></th>

2. ` <c:set> ` 用于将变量存取于 JSP 范围中或 JavaBean 属性中。
```

<%@ page language="java" import="java.util.*" pageEncoding="gb2312"%>
<%@page contentType="text/html; charset=utf-8" %>

<jsp:useBean id="person" class="lihui.Person"></jsp:useBean>

<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title>JSTL测试</title>
  </head>
  
  <body>
  <c:set value="张三" var="name1" scope="session"></c:set>
  <c:set var="name2" scope="session">李四</c:set>
  <c:set value="赵五" target="${person}" property="name"></c:set>
  <c:set target="${person}" property="age">19</c:set>
  <li>从session中得到的值：${sessionScope.name1}</li>
  <li>从session中得到的值：${sessionScope.name2}</li>
  <li>从Bean中获取对象person的name值：<c:out value="${person.name}"></c:out></li>
  <li>从Bean中获取对象person的age值：<c:out value="${person.age}"></c:out></li>
  </body>
</html>

```
![](/data/dokuwiki/javaweb/pasted/20150913-163801.png)

3.` <c:remove> ` 主要用来从指定的 jsp 范围内移除指定的变量。使用类似，下面只给出语法：
<c:remove var="变量名" [scope="page|request|session|application"]></c:remove>

4.` <c:catch> ` 用来处理 JSP 页面中产生的异常，并存储异常信息
```

<c:catch var="name1">
	容易产生异常的代码
</c:catch>

```
如果抛异常，则异常信息保存在变量 name1 中。

5.` <c:if> `
<c:if test="条件1" var="name" [scope="page|request|session|application"]></c:remove>
```

  <body>
  <c:set value="赵五" target="${person}" property="name"></c:set>
  <c:set target="${person}" property="age">19</c:set>
  <c:if test="${person.name == '赵武'}" var="name1"></c:if>
  <c:out value="name1的值：${name1}"></c:out><br/>
  <c:if test="${person.name == '赵五'}" var="name2"></c:if>
  <c:out value="name2的值：${name2}"></c:out>
  </body>

```

` 6. <c:choose> <c:when> <c:otherwise> ` 三个标签通常嵌套使用，第一个标签在最外层，最后一个标签在嵌套中只能使用一次
```

<c:set var="score">85</c:set>
    <c:choose>
    <c:when test="${score>=90}">
    你的成绩为优秀！
    </c:when>
    <c:when test="${score>=70&&score<90}">
    您的成绩为良好!
    </c:when>
    <c:when test="${score>60&&score<70}">
    您的成绩为及格
    </c:when>
    <c:otherwise>
    对不起，您没有通过考试！
    </c:otherwise>
    </c:choose>

```

7.` <c:forEach> `
语法：<c:forEach var="name" items="Collection" varStatus="statusName" begin="begin" end="end" step="step"></c:forEach>
该标签根据循环条件遍历集合 Collection 中的元素。 var 用于存储从集合中取出的元素；items 指定要遍历的集合；varStatus 用于存放集合中元素的信息。**varStatus 一共有4种状态属性**，下面例子中说明：
```

<%@ page contentType="text/html;charset=GBK" %>
<%@page import="java.util.List"%>
<%@page import="java.util.ArrayList"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>JSTL: -- forEach标签实例</title>
</head>
<body>
<h4><c:out value="forEach实例"/></h4>
<hr>
    <% 
        List a=new ArrayList();
        a.add("贝贝");
        a.add("晶晶");
        a.add("欢欢");
        a.add("莹莹");
        a.add("妮妮");
    request.setAttribute("a",a);
%>
    <B><c:out value="不指定begin和end的迭代：" /></B><br>
    <c:forEach var="fuwa" items="${a}">
    &nbsp;<c:out value="${fuwa}"/><br>
    </c:forEach>
    <B><c:out value="指定begin和end的迭代：" /></B><br>
    <c:forEach var="fuwa" items="${a}" begin="1" end="3" step="2">
    &nbsp;<c:out value="${fuwa}" /><br>
    </c:forEach>
    <B><c:out value="输出整个迭代的信息：" /></B><br>
    <c:forEach var="fuwa" items="${a}" begin="3" end="4" step="1" varStatus="s">
    &nbsp;<c:out value="${fuwa}" />的四种属性：<br>
    &nbsp;&nbsp;所在位置，即索引：<c:out value="${s.index}" /><br>
    &nbsp;&nbsp;总共已迭代的次数：<c:out value="${s.count}" /><br>
    &nbsp;&nbsp;是否为第一个位置：<c:out value="${s.first}" /><br>
    &nbsp;&nbsp;是否为最后一个位置：<c:out value="${s.last}" /><br>
    </c:forEach>
</body>
</html>

```
![](/data/dokuwiki/javaweb/pasted/20150913-164215.png)

8.` <c:forTokens> ` 用于浏览字符串，并根据指定的字符串截取字符串
语法：<c:forTokens items="stringOfTokens" delims="delimiters" [var="name" begin="begin" end="end" step="len" varStatus="statusName"]></c:forTokens>
```

<c:forTokens items="北、京、欢、迎、您" delims="、" var="c1">
13             <c:out value="${c1}"></c:out>
14         </c:forTokens>

```

**9.URL 操作标签**
（1）` <c:import> ` **把其他静态或动态文件包含到 JSP 页面**。与<jsp:include>的区别是后者只能包含同一个web应用中的文件，前者可以包含其他web应用中的文件，甚至是网络上的资源。
```

语法：<c:import url="url" [context="context"] [value="value"] [scope="..."] [charEncoding="encoding"]></c:import>
      <c:import url="url"  varReader="name" [context="context"][charEncoding="encoding"]></c:import>
    
<c:catch var="error1">
             <c:import url="http://www.baidu.com" />
         </c:catch>
<c:catch>
             <c:import url="a1.txt" charEncoding="gbk" />
         </c:catch>

```
URL路径有个绝对路径和相对路径。相对路径：<c:import url="a.txt"/>那么，a.txt必须与当前文件放在同一个文件目录下。如果以"/"开头，表示存放在应用程序的根目录下，如Tomcat应用程序的根目录文件夹为 webapps。导入该文件夹下的 b.txt 的编写方式： <c:import url="/b.txt">。
如果要访问webapps管理文件夹中的其他Web应用，就要用**context属性**。例如访问demoProj下的index.jsp，则：<c:import url="/index.jsp" context="/demoProj"/>

（2）` <c:redirect> ` 该标签用来实现请求的**重定向**。例如，对用户输入的用户名和密码进行验证，不成功则重定向到登录页面。或者实现Web应用不同模块之间的衔接
```

语法：<c:redirect url="url" [context="context"]/>
  或：<c:redirect url="url" [context="context"]>
            <c:param name="name1" value="value1">
       </c:redirect>

1 <%@ page contentType="text/html;charset=GBK"%>
2 <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
3 <c:redirect url="http://127.0.0.1:8080">
4     <c:param name="uname">lihui</c:param>
5     <c:param name="password">11111</c:param>
6 </c:redirect>

```

（3）` <c:url> ` 用于**动态生成一个 String 类型的URL**，可以同上个标签共同使用，也可以使用HTML的<a>标签实验超链接。
```

语法：<c:url value="value" [var="name"] [scope="..."] [context="context"]>
            <c:param name="name1" value="value1">
       </c:url>
或：<c:url value="value" [var="name"] [scope="..."] [context="context"]/>

 8 <c:url value="http://127.0.0.1:8080" var="url" scope="session">
 9 </c:url>
10 <a href="${url}">Tomcat首页</a>

```

##  EL运算符 
![](/data/dokuwiki/javaweb/pasted/20150913-165257.png)
```

${(pageScope.sampleValue + 12) /3==4} <br>

```   
EL表达式的内置对象
（1）与范围有关的内置对象
pageScope、requestScope、sessionScope、applicationScope
（2）与输入有关的内置对象
param 和 paramValues 用来获取表单中提交的信息。前者返回 String 类型数据，后者返回 String[] 类型的数据。如 ${paramValues.name}。
（3）其他隐含对象
Cookie
header  如：${header["UserAgent"]} 获取浏览器的版本信息
headerValues 
initParam  如：${initParam.DBDriver} 获取web.xml中配置的相关参数
` pageContex `t 如：${pageContext.request.remoteAddr} 获取用户的IP地址

` **特别注意：** `
通过获得的参数进行比较判断时，要这样比较：${param.name1 == param.name2}
` empty 运算符 `用于判断值是否为 null 或 空 。 ${empty name} , ${empty null}