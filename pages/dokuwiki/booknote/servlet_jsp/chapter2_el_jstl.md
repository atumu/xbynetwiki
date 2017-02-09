title: chapter2_el_jstl 



#  Chapter2 EL&JSTL 
Created Saturday 09 May 2015

##  EL 
JSP2.0引入EL特性
语法
${expression}
[]和.运算符和访问JavaBean：${obj[propertyName]}、${obj.properName}
Maven安装：
```

<dependency>
  <groupId>javax.el</groupId>
  <artifactId>javax.el-api</artifactId>
  <version>3.0.0</version>
</dependency>

```
不过3.0需要Java8支持。所以目前我们采用2.x版本：
```

<dependency>
  <groupId>javax.el</groupId>
  <artifactId>javax.el-api</artifactId>
  <version>2.2.5</version>
</dependency>

```
从JSP2.0和JSTL1.1之后，EL规范从JSTL中移除到JSP规范中。` EL表达式可以运行在JSP的任何部位，而不再局限于JSTL标签内。 `
目前EL已经变成了JUEL即java统一表达式语言，目的是为了统一JSP2.1和JSF1.2
伴随着Java8和JavaEE7，发布了EL3.0支持lambda表达式，流式API访问集合等。

###  EL隐式对象 
![](/data/dokuwiki/booknote/pasted/20150510-103515.png)
以及requestScope、pageScope
**` 其中的pageContext包含所有的JSP隐式对象： `**
**注意到请求属性与请求参数的不同**，访问请求参数需通过${pageContext.request.remoteAddr},${pageContext.request.session.id}
而访问请求属性则可以通过${requestScope["myAttr"]}
![](/data/dokuwiki/booknote/pasted/20150510-103532.png)
![](/data/dokuwiki/booknote/pasted/20150510-103542.png)
访问铭文jsessionid的cookie：
${cookie.jseesionid.value}

###  empty运算符:可用于检查字符串、map、集合、数组是否为空 
${empty x}如果为null或空，则返回true

<html>
<head>
<title>Employee</title>
</head>
<body>
accept-language: ${header['accept-language']}
<br/>
session id: ${pageContext.session.id}
<br/>
employee: ${requestScope.employee.name}, ${employee.address.city}
<br/>
capital: ${capitals["Canada"]}
</body>
</html>
###  EL函数 
常用的几个如下：
${fn:contains(String,String)}
${fn:escapeXml(String)}
${fn:join(String[],String)}
${length(Object)}
${fn:toLowerCase(String)}和${fn:toUpperCase(String)}
${fn:trim(String)}

##  JSTL 
用于解决诸如迭代集合，条件测试与选择、数字与时间格式化等。

###  安装： 
从__jstl.java.net__上下载两个jar: jstl api和jstl实现。添加到lib
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
通用动作指令

###  out、set、remove 
![](/data/dokuwiki/booknote/pasted/20150510-103722.png)

###  条件指令 
if、choose、when、otherwise
![](/data/dokuwiki/booknote/pasted/20150510-103757.png)

###  forEach指令：执行重复、迭代集合、迭代map 
![](/data/dokuwiki/booknote/pasted/20150510-103832.png)
![](/data/dokuwiki/booknote/pasted/20150510-103849.png)

###  forTokens指令 
![](/data/dokuwiki/booknote/pasted/20150510-103908.png)
##  格式化动作指令fmt:formatNumber、formatDate、timeZone、parseNumber、parseDate 
![](/data/dokuwiki/booknote/pasted/20150510-103935.png)
![](/data/dokuwiki/booknote/pasted/20150510-103959.png)

##  函数fn 
首先，我们要在页面的最上方引用：<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
下面是JSTL中自带的方法列表以及其描述 ：
fn:contains(string, substring)
假如参数string中包含参数substring，返回true
例如：<c:if test="${fn:contains(name, searchString)}">
fn:containsIgnoreCase(string, substring)
假如参数string中包含参数substring（忽略大小写），返回true
例如：<c:if test="${fn:containsIgnoreCase(name, searchString)}">
fn:endsWith(string, suffix)
假如参数 string 以参数suffix结尾，返回true
例如：<c:if test="${fn:endsWith(filename, ".txt")}">
fn:escapeXml(string)
将有非凡意义的XML (和HTML)转换为对应的XML character entity code，并返回
例如： <字符应该转为&lt; ${fn:escapeXml(param:info)}
fn:indexOf(string, substring)
返回参数substring在参数string中第一次出现的位置
${fn:indexOf(name, "-")}
fn:join(array, separator)
将一个给定的数组array用给定的间隔符separator串在一起，组成一个新的字符串并返回。
${fn:join(array, ";")}
fn:length(item)
返回参数item中包含元素的数量。参数Item类型是数组、collection或者String。假如是String类型,返回值是String中的字符数。
${fn:length(shoppingCart.products)}
fn:replace(string, before, after)
返回一个String对象。用参数after字符串替换参数string中所有出现参数before字符串的地方，并返回替换后的结果
${fn:replace(text, "-", "&#149;")}
fn:split(string, separator)
返回一个数组，以参数separator 为分割符分割参数string，分割后的每一部分就是数组的一个元素
${fn:split(customerNames, ";")}
 fn:startsWith(string, prefix)
假如参数string以参数prefix开头，返回true
<c:if test="${fn:startsWith(product.id, "100-")}">
fn:substring(string, begin, end)
返回参数string部分字符串, 从参数begin开始到参数end位置，包括end位置的字符
${fn:substring(zip, 6, -1)}
fn:substringAfter(string, substring)
返回参数substring在参数string中后面的那一部分字符串
${fn:substringAfter(zip, "-")}
fn:substringBefore(string, substring)
返回参数substring在参数string中前面的那一部分字符串
${fn:substringBefore(zip, "-")}
fn:toLowerCase(string)
将参数string所有的字符变为小写，并将其返回
${fn.toLowerCase(product.name)}
fn:toUpperCase(string)
将参数string所有的字符变为大写，并将其返回
${fn.UpperCase(product.name)}
fn:trim(string)
去除参数string 首尾的空格，并将其返回
${fn.trim(name)}
