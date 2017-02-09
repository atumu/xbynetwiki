title: chapter1_servlet_jsp_session介绍 

#  概述 
Servlet采用3.0版本，JSP 2.2
安装：Servlet-api.jar; jsp-api.jar在Tomcat lib目录下可找到。
源码下载：books.brainysoftware.com/download

##  Tomcat中应用部署方式 
方式一：直接将应用复制到webapps目录下
方法二：编辑conf/server.xml文件.在Host元素下创建一个Context元素
<Host ....>
<Context docBase="path to app" path="/appName" reloadable="true"/>
</Host>
方法三：在conf/catalina/localhost/下创建一个Context为根元素的xml文件，文件名即为应用名。
<Context  docBase="path to app" reloadable="true"/>

#  Servlet 

##  概述 
Maven：
```

<!-- JSTL -->
		<dependency>
			<groupId>org.apache.taglibs</groupId>
			<artifactId>taglibs-standard-spec</artifactId>
			<version>1.2.5</version>
		</dependency>
		<dependency>
			<groupId>org.apache.taglibs</groupId>
			<artifactId>taglibs-standard-impl</artifactId>
			<version>1.2.5</version>
		</dependency>

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>javax.servlet.jsp-api</artifactId>
			<version>2.3.1</version>
			<scope>provided</scope>
		</dependency>

```
四个包：
javax.servlet:
javax.servlet.http:
javax.servlet.annotation:
javax.servlet.descriptor
javax.servlet包中的主要类：Servlet, ` ServletConfig `, ServletRequest, ServletResponse, ` ServletContext `, RequestDispatcher, Filter
javax.servlet包中的:HttpSession,Cookie
注解：@WebServlet, @WebInitParam
//Servlet容器会将会保证Servlet只有一个实例//，这就意味着在某些情况下我们必须考虑线程安全的问题。尽量别使用类级变量，除非他们是只读的，也可以使用java.util.concurrent.atomic包中的成员。
Servlet需要保存传入init方法的ServletConfig对象。

##  Servlet生命周期方法简介 
  * ` init() `只在实例化时被调用一次。后续请求不会调用。可以在该方法中获取` ServletConfig，ServletContext `对象。关于初始化我们可以采用预初始化，那就是在web.xml中添加<load-on-startup>1</load-on-startup>
  * ` service() `每次请求都会调用它。
  * ` destroy() ` 应该总是使用destroy方法清理Servlet持有的资源。

##  编写Servlet应用 
程序目录结构：
```

project/
----WEB-INF
------------classes
------------lib

```
```

@WebServlet(name = "MyServlet", urlPatterns = { "/my" })
public class MyServlet implements Servlet {
	private transient ServletConfig servletConfig;
	@Override
	public void init(ServletConfig servletConfig)
			throws ServletException {
		this.servletConfig = servletConfig;
	}
	@Override
	public ServletConfig getServletConfig() {
		return servletConfig;
	}
	@Override
	public String getServletInfo() {
		return "My Servlet";
	}
	@Override
	public void service(ServletRequest request,
			ServletResponse response) throws ServletException,
			IOException {
		String servletName = servletConfig.getServletName();
		response.setContentType("text/html");
		PrintWriter writer = response.getWriter();
		writer.print("<html><head></head>"
				+ "<body>Hello from " + servletName 
				+ "</body></html>");
	}
	@Override
	public void destroy() {
	}    
}

```
ServletRequest:获取请求参数，及设置保存属性。
ServletResponse:
getWriter获取java.io.PrintWriter进行字符输出，默认情况下PrintWriter采用ISO-8859-1编码。注意输出任何内容前请先通过setContentType()设置输出的Content-Type和编码。
getOutputStream用于传输二进制数据。
ServletConfig:获取初始化参数和ServletContext对象
```

@WebServlet(name = "ServletConfigDemoServlet", 
	urlPatterns = { "/servletConfigDemo" },
	initParams = {
		@WebInitParam(name="admin", value="Harry Taciak"),
		@WebInitParam(name="email", value="admin@example.com")
	}
)

```
ServletContext:代表了一个Web应用的上下文，同时可以用于保存作用于应用全局的序列化属性。
GeneriaServlet:
HttpServlet:

##  部署描述符WEB-INF/web.xml的使用： 
```

<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0"> 
	<servlet>
		<servlet-name>SimpleServlet</servlet-name>
		<servlet-class>app01c.SimpleServlet</servlet-class>
		<load-on-startup>1</load-on-startup> <!--配置自启动，数字越大启动越晚-->
	</servlet>
	<servlet-mapping>
		<servlet-name>SimpleServlet</servlet-name>
		<url-pattern>/simple</url-pattern>
	</servlet-mapping>  
	<servlet>
		<servlet-name>WelcomeServlet</servlet-name>
		<servlet-class>app01c.WelcomeServlet</servlet-class>
		<load-on-startup>20</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>WelcomeServlet</servlet-name>
		<url-pattern>/welcome</url-pattern>
	</servlet-mapping>
</web-app>

```
**我们可以在Servlet中通过` this.getServletName() `来获取配置的<servlet-name>进而进行相应的操作和处理逻辑**。如SpringMVC默认加载配置文件为配置的servlet-name如springmvc.默认配置文件名为springmvc-servlet.xml.
###  配置初始化参数 
**1、上下文初始化参数**
```

<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
         /WEB-INF/abc.xml,classpath:cde.xml 
        </param-value>
</context-param>

```

**2、Servlet初始化参数**
```

<servlet>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring-mvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

```

Servlet代码中可以通过` ServletContext.getInitParameter `来获取这些初始化参数

##  异步请求处理与异步监听 
Servlet3引入的新特性
在早前版本的Servlet规范中，如果Servlet作为控制器调用了一个较耗时的业务方法，那么Servlet必须等到业务方法完全返回之后才生成响应，这使得Servlet对业务方法的调用变成一种阻塞式的调用，从而导致运行效率降低。
异步处理特性可以应用于 Servlet 和过滤器两种组件，**因此默认情况下，Servlet 和过滤器并没有开启异步处理特性**，如果希望使用该特性，则必须按照如下的方式启用：
  * web.xml配置 Servlet 和过滤器的情况，Servlet 3.0 为 <servlet> 和 <filter> 标签增加了 ` <async-supported> ` 子标签，该标签的默认取值为 false，要启用异步处理支持，则将其设为 true 即可。
  * @WebServlet 和 @WebFilter 进行 Servlet 或过滤器配置的情况，这两个注解都提供了 **` asyncSupported 属性 `**，默认该属性的取值为 false，要启用异步处理支持，只需将该属性设置为 true 即可。
Servlet3.0规范引入了异步处理来解决这个问题，**异步处理允许Servlet重新发起一条新线程去调用较耗时的业务方法，这样就可避免等待**。该异步机制是通过**` AsyncContext `**类来处理的，Servlet可通过**ServletRequest**的如下两个方法开启异步调用、创建AsyncContext对象：
  * 1、AsyncContext ` startAsync() `
  * 2、AsyncContext ` startAsync(ServletRequest,ServletResponse) `
**重复调用上面的方法将得到同一个` AyncContext `对象**，AyncContext对象代表异步处理的上下文，它提供了一些工具方法，可以完成` setTimeout，dispatch用于请求，complete,start(Runnable) `启动后台线程、**` addListener(AsyncListener) `**获取request response对象等功能

` **运行机制：** `当调用` startAsync `开启异步请求之后，它返回一个包含了请求对象的` AsyncContext `对象，然后Servlet将从service方法继续执行并返回。不需要对请求作出响应，它所使用请求线程也会被返回到线程池中，**但该请求并没有被关闭**，相反仍然保持打开、未应答的状态。稍后，当某些事件发生时，**容器应该可以从AysncContext中获取到响应对象，并使用它向客户端发送响应数据。**

**前提：设置asyncSupported=true**
''@WebServlet(urlPatterns="/async",asyncSupported=true)
@WebFilter(urlPatterns="/*",asyncSupported=true)''
**异步监听：**
`  asyncContext.addListener(new AsyncListener()); `
```

@WebServlet(name = "AsyncListenerServlet", 
        urlPatterns = { "/asyncListener" }, 
        asyncSupported = true)
public class AsyncListenerServlet extends HttpServlet {
    private static final long serialVersionUID = 62738L;

    @Override
    public void doGet(final HttpServletRequest request, 
            HttpServletResponse response) 
            throws ServletException, IOException {
        final AsyncContext asyncContext = request.startAsync();
        asyncContext.setTimeout(5000);
        
        asyncContext.addListener(new MyAsyncListener());
        asyncContext.start(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                }
                String greeting = "hi from listener";
                System.out.println("wait....");
                request.setAttribute("greeting", greeting);
                asyncContext.dispatch("/test.jsp");
            }
        });
    }    
}

```
```

public class MyAsyncListener implements AsyncListener {

    @Override
    public void onComplete(AsyncEvent asyncEvent) 
            throws IOException {
        System.out.println("onComplete");
    }

    @Override
    public void onError(AsyncEvent asyncEvent) 
            throws IOException {
        System.out.println("onError");
    }

    @Override
    public void onStartAsync(AsyncEvent asyncEvent) 
            throws IOException {
        System.out.println("onStartAsync");
    }

    @Override
    public void onTimeout(AsyncEvent asyncEvent) 
            throws IOException {
        System.out.println("onTimeout");
    }
}


```
##  HttpServletRequest 
**请求参数：三种形式：**
  * 查询参数(URI参数，GET形式附加在URI之后)
  * application/x-www-form-urlencoded(普通表单) 作文请求头的` Content-Type `的值
  * multipart/form-data(表单文件上传) 作文请求头的` Content-Type `的值

**请求参数获取**
  * getParameter返回参数的单个值
  * getParameterValues返回参数值的数组
  * getParameterMap返回所有参数名值对的java.util.Map<String,` String[] `>
  * getParameterNames返回所有参数名字的枚举
` 以上方法与getInputStream或getReader相冲突，因为请求的InputStream只能被读取一次。 `

**确定与请求内容相关的信息**
决定HTTP请求内容的**类型、长度和编码**。
  * ` getContentType `返回请求的MIME类型（如application/json,application/zip,text/plain,application/x-www-form-urlencoded等）
  * ` getContentLength和getContentLengthLong `都返回了请求正文的长度。(字节为单位。如1MB为1024*1024长度)。**后者是Servlet3.1才支持的，用于内容长度超过2GB的请求。**如果是之前的版本，在内容超过2GB的情况下需要调用` getHeader("Content-Length") `,并将返回的String转换为**long**
  * ` getCharacterEncoding `当请求中包含字符类型的内容时，将返回字符编码（如` UTF-8,ISO-8859-1 `）

**获取请求特有的数据，如URL，URI，和头**
请求URL为http://www.xby1993.net/app/login?name=123
  * getRequestURL返回http://www.xby1993.net/app/login
  * getRequestURI返回/app/login
  * getServletPath返回/login
  * getHeader
  * getHeaderNames
  * getIntHeader
  * getDateHeader

获取Session和Cookies
  * getSession
  * getCookies

##  HttpServletResponse 
设置响应头、编写响应正文、重定向请求、设置HTTP状态码、将Cookies返回到客户端等任务。
**设置内容类型和编码格式**（` 必须在获取输出流之前调用 `，否则无效）：
  * setContentType
  * setCharacterEncoding
设置头和其他属性
  * ` setHeader `
  * addheader
  * ` setStatus ` 设置状态码
  * sendError 
  * ` sendRedirect ` 客户端重定向

**编写响应正文**
  * getOutputStream/getWriter

#  Session管理 

HTTP本身是无状态的。(1.1版本)用于保持状态的4种方法：URL重写、隐藏域、cookie、HttpSession

##  Cookie 
domain、path、maxAge属性（必须在cookie被添加到repsonse之前调用setMaxAge，这个调用是必须的）
删除cookie的方法：没有直接删除的方式，可以通过创建一个同名的cookie然后调用setMaxAge(0)之后添加到响应response.add(cookie)即可删除.
```

 Cookie cookie = new Cookie("titleFontWeight", "");
cookie.setMaxAge(0);
response.addCookie(cookie);

```


##  HttpSession 
session是自动创建的。
是服务器端的行为用于跟踪客户的状态，当用户去访问某个站点时，服务器端就会为客户产生一个sessionID,以cookie的方式返回给客户端，当客户的去访问该站点的其他服务时，就会带者当前sessionID一起发出请求，已识别是哪个用户，一个用户就好比一个session对象，互不干扰。
Session的用途：
  * 维持HTTP会话状态
  * 记住用户
  * 启用工作流
  * 其他
**Session重要方法：**
1） getId() --- 获取session的id号，每个id号都是不同的
2） isNew() --- 判断该session是不是新的
**` 3） invalidate() --- 让当前session失效，释放资源。用户注销登录 `**
4） setMaxInactiveInterval(int interval) --- 设置session处于不活动的时间间隔（**以秒为单位**），
超过该时间，session失效，
如果设置的是负数，或0，则不限制session不活动的时间间隔，默认一般是30分钟
5） setAttribute(String name , Object o ) ，getAttribute(String name)
session可以保存会话属性，但是属性是保存在内存中的，**同时属性必须是Serilizable的**，` 同时在分布式环境下需要注意同步问题。 `
每一个session都有一个唯一的id.这个ID发送给浏览器有两种形式：名为JSESSIONID的cookie或者作为一个**JSESSIONID**参数添加到URL后面。
session有效时间默认为30分钟，可以进行设置。
**由于session对象是保存在内存中的，所以使用时需要注意对服务器内存的消耗。**
**运用场景：**
1） 登录
2） 购物车
3）其他

###   Session的有效期 
由于会有越来越多的用户访问服务器，因此Session也会越来越多。为防止内存溢出，服务器会把长时间内没有活跃的Session从内存删除。这个时间就是**Session的超时时间**。如果超过了超时时间没访问过服务器，Session就自动失效了。
Session的超时时间为` maxInactiveInterval `属性，可以通过对应的getMaxInactiveInterval()获取，通过setMaxInactiveInterval(longinterval)修改。
Session的超时时间也可以在web.xml中修改。另外，通过调用Session的` invalidate() `方法可以使Session失效。

Session超时配置：
```

    <session-config>
        <session-timeout>30</session-timeout> <!--超时为30分钟 -->
        <!--  <cookie-config>
     		<http-only>true</http-only>
       	 </cookie-config>  -->
     <!--  <tracking-mode>COOKIE</tracking-mode>  表示容器可以使用哪种技术跟踪会话ID。可以配置多个，可选值 COOKIE, URL , SSL (注意：URL重写可能会带来危险性，因为用户可能会复制并分享带有SessionID的URL)--> 
    </session-config>

```
###  监听会话创建和销毁 
  * HttpSessionListener
  * HttpSessionIdListener
#  JSP 
概述
JSP页面其实也是一个Servlet. HttpJspPage，而它继承了HttpServlet
4个包：
javax.servlet.jsp:
javax.servlet.jsp.tagext
javax.el
javax.servlet.jsp.el
```

<%@page import="java.util.Date"%>
<%@page import="java.text.DateFormat"%>

<html>
<head><title>Today's date</title></head>
<body>
<%
	DateFormat dateFormat = 
			DateFormat.getDateInstance(DateFormat.LONG);
	String s = dateFormat.format(new Date());
	out.println("Today is " + s);
%>
</body>
</html>

```
四类组成部分
<%@ 这是一个指令 %>
<%! 这是一个声明 %>
<% 这是一个脚本 %>
<%= 这是一个表达式 %>
<%-- 这是一个注释 --%>
##  注释： 
两种：jsp注释：<%-- --%>不会发送到浏览器。 html注释<!-- -->发送到客户浏览器。

###  JSP隐式对象： 
![](/data/dokuwiki/booknote/pasted/20150510-103232.png)

属性可以保存在4总范围当中：page,request,session,application

##  指令 
<%@ %>
page, include, taglib, tag, attribute, variable
page指令属性：
* import
* buffer
* autoFlush
* errorPage 如果在JSP执行过程中出现了错误，该属性将告诉容器将请求转发到哪个JSP
* isErrorPage 表示当前JSP是否作为错误页面
* contentType
* pageEncoding:默认ISO-8859-1
* isELIgnored
* session

**包含其他JSP页面**
` include指令：<%@ include file="copy.jsp" %> ` **包含合并操作发生在JSP转换为java代码之前**。所以被包含的jsp页面不必是一个完整页面。` 被包含JSP代码被嵌入当前include位置 `，而不是jsp输出。
这种包含是` 静态行为 `。这与jsp:include是不同的。下面分析：
<jsp:include path="" /> 这是一种动态包含行为，被包含的jsp也会被单独编译成servlet，在运行时，请求会被临时重定向到被包含的JSP，然后将该JSP的结果输出到响应中，然后再将控制权返回给主JSP页面。
千万要注意以上两点的区别。





###  关闭脚本元素： 
<jsp-config>
<jsp-property-group>
<url-pattern>*.jsp</url-pattern>
<scripting-invalid>true</scripting-invalid>
......

##  动作 
<jsp:forward>
<jsp:include>
<jsp:dobody>
<jsp:invoke>
<jsp:param>

<jsp:useBean>
<jsp:setProperty>
<jsp:getProperty>

##  关于<jsp:include page="" />与<%@ include file="" %>的区别: 

前者发生在请求的时候，后者发生在jsp编译为servlet的时候。前者包含的是输出，后者包含的是源代码。

##  错误处理 
声明错误页面：<%@ page isErrorPage="true" %>
```

<%@page isErrorPage="true"%>
<html>
<head><title>Error</title></head>
<body>
An error has occurred. <br/>
Error message: 
<%
	out.println(exception.toString());
%>
</body>
</html>

```
指定错误处理页面：<%@ page errorPage="" %>
```

<%@page errorPage="errorHandler.jsp"%>
Deliberately throw an exception
<%
	Integer.parseInt("Throw me");
%>

```

##  将Servlet请求发送给JSP 
request.getRequestDispatcher("/WEB-INF/index.jsp").forward(request,response);
