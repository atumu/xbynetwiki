title: jsp与el内置对象 

#  JSP与EL内置对象 
参考：http://blog.csdn.net/zhangguoliang521/article/details/8189493
http://blog.csdn.net/zhai56565/article/details/9981339
##  内置对象表格 
JSP九个内置对象:
![](/data/dokuwiki/javaweb/pasted/20150913-161713.png)
EL 11个内置对象:
![](/data/dokuwiki/javaweb/pasted/20150913-161831.png)

##  EL隐式对象说明 
与` 作用范围有关 `的EL隐含对象包含有：` pageScope、requestScope、sessionScope和applicationScope `
– 它们可以读取使用JSP内置对象pageContext、request、session以及application的**setAttribute()方法所设定的对象的值**-----**即getAttribute(String name)，` 却不能取得其他相关信息 `。**
– 例如，要取得session中储存的一个username属性的值，可以利用下面的方法：session.getAttribute("username")
– 在EL中则使用下面的方法：${sessionScope.username}


<note important>再次强调这4个隐含对象` pageScope、requestScope、sessionScope和applicationScope `只能用来取得指定范围内的` 属性值(setAttribute()方法所设定的) `，而**不能取得其他相关信息。**
如需获取其他信息，如服务器相关信息，请用PageContext.request,PageContext.session等。</note>



与` 输入有关 `的隐含对象有两个，即` param和paramValues `
• 例如，要取得用户的请求参数时在EL中则可以使用param和paramValues两者来取得数据：
– ${param.name}
– ${paramValues.name[]},这种方法可以用数组直接获取，不用循环了。这时可同时使用[]运算符号来读取哪个元素，例如${paramValues.week[0]}
**2. 其他隐式对象**
**• cookie**
用来取得使用者的cookie值，例如在cookie中设定了username属性值，可以使用` ${cookie.username.value} `来取得属性值。

**• header和headerValues**
读取请求的头数据，使用header或headerValues内置对象，例如${header[“User-Agent”]}，headerValues则用来取得所有的头信息，等价于调用request.getHeaders()方法。

**• initParam**
initParam用来读取设置在web.xml中的参数值。例如${initParam.repeat}，等价于：(String)application.getInitParameter(“repeat”); 或
servletContext.getInitParameter(“repeat”);

**` • pageContext `**
**pageContext用于取得其他有关用户要求或页面的详细信息，这是requestScope等无法做到的**
  * ${pageContext.request.queryString}取得请求的参数字符串
  * ${pageContext.request.requestURL} 取得请求的URL，不包括参数字符串
  * ${pageContext.request.contextPath} 服务的web application 的名称
  * ${pageContext.request.method} 取得HTTP 的方法(GET、POST)
  * ${pageContext.request.protocol} 取得使用的协议(HTTP/1.1、HTTP/1.0)
  * ${pageContext.request.remoteUser} 取得用户名称
  * ${pageContext.request.remoteAddr} 取得用户的IP 地址
  * ${pageContext.session.new} 判断session 是否为新的
  * ${pageContext.session.id} 取得session 的ID
  * ${pageContext.servletContext.serverInfo} 取得主机端的服务信息