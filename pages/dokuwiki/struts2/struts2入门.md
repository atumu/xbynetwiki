title: struts2入门 

#  struts2入门 
安装：
```

 <!-- struts2依赖包 -->  
    <dependency>  
        <groupId>org.apache.struts</groupId>  
        <artifactId>struts2-core</artifactId>  
        <version>2.3.14</version>  
    </dependency> 

```
WEB-INF/web.xml配置
```

<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0"
>

  <display-name>Archetype Created Web Application</display-name>
  
  <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
        <init-param>
            <param-name>actionPackages</param-name>
            <param-value>com.mycompany.myapp.actions</param-value>
        </init-param>
    </filter>
 
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>


```
WEB-INF/classes/struts.xml配置
```

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
    "http://struts.apache.org/dtds/struts-2.0.dtd">

<struts>
    <constant name="struts.enable.DynamicMethodInvocation" value="false" />
    <constant name="struts.devMode" value="true" />
    <package name="app03a" extends="struts-default">

        <default-action-ref name="CatchAll"/>
        <global-results>
            <result name="error">/jsp/Error.jsp</result>
        </global-results>
        <global-exception-mappings>
            <exception-mapping exception="java.lang.Exception" result="error"/>
        </global-exception-mappings>
        
        <action name="Product_input">
            <result>/jsp/ProductForm.jsp</result>
            <result name="login">/jsp/Login.jsp</result>
        </action>

        <action name="Product_save" class="app03a.Product">
            <result type="dispatcher">/jsp/ProductDetails.jsp</result>
            <result name="input">/jsp/ProductForm.jsp</result>
        </action>


        <action name="User_input">
            <result>
                <param name="location">/jsp/Login.jsp</param>
            </result>
        </action>
        <action name="User_login" class="app03a.User" method="login">
            <result name="success">/jsp/Menu.jsp</result>
            <result name="input">/jsp/Login.jsp</result>
        </action>
        <action name="User_logout" class="app03a.User" method="logout">
            <result name="success">/jsp/Login.jsp</result>
        </action>
        
        
        <action name="CatchAll">
        	<result type="httpheader">
        	    <param name="status">404</param>
        	</result>
        </action>
        <action name="RedirectTest" class="app03a.TestUser">
            <result type="redirect-action">
                <param name="actionName">User_input</param>
            </result>    
        </action>

    </package>

    <!--
    <package name="wildcardMappingTest" namespace="/wild" extends="struts-default">
        <action name="Book_add" class="app03a.Book" method="add">
            <result>/jsp/Book.jsp</result>
        </action>
        <action name="Book_edit" class="app03a.Book" method="edit">
            <result>/jsp/Book.jsp</result>
        </action>
        <action name="Book_delete" class="app03a.Book" method="delete">
            <result>/jsp/Book.jsp</result>
        </action>
        
        <action name="Author_add" class="app03a.Author" method="add">
            <result>/jsp/Author.jsp</result>
        </action>
        <action name="Author_edit" class="app03a.Author" method="edit">
            <result>/jsp/Author.jsp</result>
        </action>
        <action name="Author_delete" class="app03a.Author" method="delete">
            <result>/jsp/Author.jsp</result>
        </action>
    </package>
    -->
    <!--
    <package name="wildcardMappingTest" namespace="/wild" extends="struts-default">
        <action name="*_*" class="app03a.{1}" method="{2}">
            <result>/jsp/{1}.jsp</result>
        </action>
    </package>
    -->

    <package name="dynamicMethodInvocation" namespace="/dmi" extends="struts-default">
        <action name="Book" class="app03a.Book">
            <result>/jsp/Book.jsp</result>
        </action>
    </package> 

</struts>


```
JSP：
```

<%@ page language="java" import="java.util.*"
	contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags"%>

<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">

<title>My JSP 'pagePerson.jsp' starting page</title>

<meta http-equiv="pragma" content="no-cache">
<meta http-equiv="cache-control" content="no-cache">
<meta http-equiv="expires" content="0">
<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
<meta http-equiv="description" content="This is my page">

<script type="text/javascript">
	function validate() {
		var page = document.getElementsByName("page")[0].value;

		if (page > <s:property value="#request.pageBean.totalPage"/>) {
			alert("你输入的页数大于最大页数，页面将跳转到首页！");

			window.document.location.href = "person";

			return false;
		}

		return true;
	}
</script>

</head>

<body>

	<h1>
		<font color="blue">分页查询</font>
	</h1>
	<hr>

	<table border="1" align="center" bordercolor="yellow" width="50%">

		<tr>
			<th>序号</th>
			<th>姓名</th>
			<th>年龄</th>
		</tr>


		<s:iterator value="#request.pageBean.list" id="person">

			<tr>
				<th><s:property value="#person.uid" /></th>
				<th><s:property value="#person.name" /></th>
				<th><s:property value="#person.age" /></th>
			</tr>

		</s:iterator>

	</table>

	<center>

		<font size="5">共<font color="red"><s:property
					value="#request.pageBean.totalPage" /></font>页
		</font>&nbsp;&nbsp; <font size="5">共<font color="red"><s:property
					value="#request.pageBean.allRows" /></font>条记录
		</font><br>
		<br>

		<s:if test="#request.pageBean.currentPage == 1">
            首页&nbsp;&nbsp;&nbsp;上一页
        </s:if>

		<s:else>
			<a href="person.action">首页</a>
            &nbsp;&nbsp;&nbsp;
            <a
				href="person.action?page=<s:property value="#request.pageBean.currentPage - 1"/>">上一页</a>
		</s:else>

		<s:if
			test="#request.pageBean.currentPage != #request.pageBean.totalPage">
			<a
				href="person.action?page=<s:property value="#request.pageBean.currentPage + 1"/>">下一页</a>
            &nbsp;&nbsp;&nbsp;
            <a
				href="person.action?page=<s:property value="#request.pageBean.totalPage"/>">尾页</a>
		</s:if>

		<s:else>
            下一页&nbsp;&nbsp;&nbsp;尾页
        </s:else>

	</center>
	<br>

	<center>

		<form action="person" onsubmit="return validate();">
			<font size="4">跳转至</font> <input type="text" size="2" name="page">页
			<input type="submit" value="跳转">
		</form>

	</center>

</body>
</html>

```