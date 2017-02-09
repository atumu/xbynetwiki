title: chapter4监听器与过滤器 

#  chapter4监听器与过滤器 
##  监听器 
###  监听器接口 

Servlet事件java.util.Event主要有3类：ServletContext、HttpSession与ServletRequest。下面具体讲解这3类事件的监听器实现。
**Servlet 3.0新增javax.servlet.AyncListener**
1．对Servlet上下文进行监听
可以监听ServletContext对象的创建和删除以及属性的添加、删除和修改等操作。该监听器需要使用到如下两个接口类：
   ● ServletContextAttributeListener：监听对ServletContext属性的操作，如增加、删除、修改操作。
   ● ServletContextListener：监听ServletContext。
当创建ServletContext时，激发contextInitialized (ServletContextEvent sce)方法；
当销毁ServletContext时，激发contextDestroyed(ServletContext- Event sce)方法。
2．监听Http会话
可以监听Http会话活动情况、Http会话中属性设置情况，也可以监听Http会话的active、paasivate情况等。
该监听器需要使用到如下多个接口类：
   ● HttpSessionListener：监听HttpSession的操作。
当创建一个Session时，激发session Created (SessionEvent se)方法；
当销毁一个Session时，激发sessionDestroyed (HttpSessionEvent se)    方法。
   ● HttpSessionActivationListener：用于监听Http会话active、passivate情况。
   ● HttpSessionAttributeListener：监听HttpSession中属性的操作。
当在Session增加一个属性时，激发attributeAdded(HttpSessionBindingEvent se) 方法；
当在Session删除一个属性时，激发attributeRemoved(HttpSessionBindingEvent se)方法；
在Session属性被重新设置时，激发attributeReplaced(HttpSessionBindingEvent se) 方法。

3．对客户端请求进行监听
对客户端的请求进行监听是在Servlet 2.4规范中新添加的一项技术，使用的接口类如下：
   ● ServletRequestListener接口类。
   ● ServletRequestAttrubuteListener接口类。
   
4、` javax.servlet.AyncListener ` 用于异步操作监听。
###  监听器注册 
两种方式：
` @WebListener `注解：
web.xml配置:` <listener><listener-calss>net.xby1993.MyListener</listener-calss></listener> `

###  servlet监听器实现在线人数统计 
```

import javax.servlet.http.HttpSessionEvent;
import javax.servlet.http.HttpSessionListener;
public class CounterListener implements HttpSessionListener {
	private static long onlineNumber = 0;
	public static long getOnlineNumber() {
		return onlineNumber;
	}
	public void sessionCreated(HttpSessionEvent se) {
		onlineNumber++;
	}
	public void sessionDestroyed(HttpSessionEvent se) {
		onlineNumber--;
	}
}


当前应用中一共有<%=CounterListener.getOnlineNumber()%>人在线<br>

```
不过上段代码需要注意线程同步问题。
##  过滤器Filter 
###  一、Servlet过滤器的概念： 
过滤器javax.servlet.Filter是可以拦截访问资源的请求、响应的应用组件。过滤器可以检测和修改请求和响应，他们甚至可以拒绝、重定向或转发请求。
Servlet过滤器是在Java Servlet规范2.3中定义的，它能够对Servlet容器的请求和响应对象进行检查和修改。　　　
Servlet过滤器本身并不产生请求和响应对象，它只能提供过滤作用。Servlet过期能够在Servlet被调用之前检查Request对象，修改Request Header和Request内容；在Servlet被调用之后检查Response对象，修改Response Header和Response内容。
Servlet过期负责过滤的Web组件可以是Servlet、JSP或者HTML文件。

**Servlet过滤器的特点：**
A．Servlet过滤器可以检查和修改ServletRequest和ServletResponse对象
B．Servlet过滤器可以被指定和特定的URL关联，只有当客户请求访问该URL时，才会触发过滤器
C．Servlet过滤器可以被串联在一起，形成**链式管道**效应，协同修改请求和响应对象
**Servlet过滤器的作用：**
A．查询请求并作出相应的行动。
B．阻塞请求-响应对，使其不能进一步传递。
C．修改请求的头部和数据。用户可以提供自定义的请求。
D．修改响应的头部和数据。用户可以通过提供定制的响应版本实现。
E．与外部资源进行交互。

**常见过滤器举例：**
  * 日志过滤器
  * 验证授权过滤器
  * Session超时过滤器
  * 压缩和加密过滤器
  * 错误处理过滤器 ：**可以将请求处理包含在try-catch块中**，用于捕捉和记录所有错误。**请求结果被转发至一个通用的错误页面**，该页面不包含任何诊断或者敏感信息。防止被黑客利用等。

###  Servlet过滤器的适用场合： 
A．认证过滤
B．登录和审核过滤
C．图像转换过滤 
D．数据压缩过滤 
E．加密过滤 
F．令牌过滤 
G．资源访问触发事件过滤 
H．XSL/T过滤 
I．Mime-type过滤
###  Filter API的构成： 
**Filter、FilterConfig、FilterChain**
所有的Servlet过滤器类都必须实现**javax.servlet.Filter**接口。这个接口含有3个过滤器类必须实现的方法：
**A.init(FilterConfig)：**
这是Servlet过滤器的初始化方法，Servlet容器创建Servlet过滤器实例后将调用这个方法。在这个方法中可以读取web.xml文件中Servlet过滤器的初始化参数
**B.doFilter(ServletRequest,ServletResponse,FilterChain)：**
这个方法完成实际的过滤操作，当客户请求访问于过滤器关联的URL时，Servlet容器将先调用过滤器的doFilter方法。**FilterChain参数用于访问后续过滤器(链式)**
**C.destroy()：**
Servlet容器在销毁过滤器实例前调用该方法，这个方法中可以释放Servlet过滤器占用的资源

<note important>注意：Servlet容器只会创建一个Filter实例。所以对于多线程问题需要谨慎处理</note>
###  Servlet过滤器的创建步骤： 
A．实现javax.servlet.Filter接口
B．实现init方法，读取过滤器的初始化函数
C．实现doFilter方法，完成对请求或过滤的响应
D．**调用FilterChain接口对象的doFilter方法**，向后续的过滤器传递请求或响应
E．销毁过滤器
###  Servlet过滤器的发布： 
Servlet3可以使用@WebFilter注解：（配合@WebInitParam可指定初始化参数）
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-123625.png)
web.xml方式：
` 注意：Filter的定义顺序很重要 `
A．发布Servlet过滤器时，必须在**web.xml**文件中加入**<filter>元素和<filter-mapping>**元素。
B．**<filter>**元素用来定义一个过滤器：
属性                   含义
filter-name    指定过滤器的名字
filter-class    指定过滤器的类名
init-param    为过滤器实例提供初始化参数，可以有多个
C．**<filter-mapping>**元素用于将过滤器和URL关联：
属性                     含义
filter-name    指定过滤器的名字
url-pattern    指定和过滤器关联的URL，为”/*”表示所有URL
dispatcher     指定过滤器所拦截的资源被 Servlet 容器调用的方式，可以是**REQUEST,INCLUDE,FORWARD和ERROR**之一，默认**REQUEST**。用户可以设置多个<dispatcher> 子元素用来指定 Filter 对资源的多种调用方式进行拦截。

**映射到不同的请求派发器类型**
  * 普通请求
  * 转发请求
  * 包含请求
  * 错误资源请求
  * 异步请求

```

<filter>
        <filter-name>normalFilter</filter-name>
        <filter-class>com.wrox.AnyRequestFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>
 <filter-mapping>
        <filter-name>normalFilter</filter-name>
        <url-pattern>/*</url-pattern>
         <url-pattern>/foo</url-pattern>
          <servlet-name>myServlet</servlet-name>
        <dispatcher>REQUEST</dispatcher>
         <dispatcher>FORWARD</dispatcher>
          <dispatcher>ASYNC</dispatcher>
    </filter-mapping>

```
dispatcher标签的类型有**REQUEST,FORWARD,INCLUDE,ERROR,ASYNC**

###  一个实例 

首先来看一下web.xml的配置：
```

<!-- 请求url日志记录过滤器 -->    
    <filter>    
        <filter-name>logfilter</filter-name>    
        <filter-class>com.weijia.filterservlet.LogFilter</filter-class>    
    </filter>    
    <filter-mapping>    
        <filter-name>logfilter</filter-name>    
        <url-pattern>/*</url-pattern>    
    </filter-mapping>  
      
<!-- 编码过滤器 -->    
    <filter>    
        <filter-name>setCharacterEncoding</filter-name>    
        <filter-class>com.weijia.filterservlet.EncodingFilter</filter-class>    
        <init-param>    
            <param-name>encoding</param-name>    
            <param-value>utf-8</param-value>    
        </init-param>    
    </filter>    
    <filter-mapping>    
        <filter-name>setCharacterEncoding</filter-name>    
        <url-pattern>/*</url-pattern>    
    </filter-mapping>   

```
编码过滤器：
```

public class EncodingFilter implements Filter {    
    private String encoding;    
    private HashMap<String,String> params = new HashMap<String,String>();    
    // 项目结束时就已经进行销毁    
    public void destroy() {    
        System.out.println("end do the encoding filter!");    
        params=null;    
        encoding=null;    
    }    
    public void doFilter(ServletRequest req, ServletResponse resp,FilterChain chain) throws IOException, ServletException {    
        System.out.println("before encoding " + encoding + " filter！");    
        req.setCharacterEncoding(encoding);    
        chain.doFilter(req, resp);          
        System.out.println("after encoding " + encoding + " filter！");    
        System.err.println("----------------------------------------");    
    }    
     
    // 项目启动时就已经进行读取    
    public void init(FilterConfig config) throws ServletException {    
        System.out.println("begin do the encoding filter!");    
        encoding = config.getInitParameter("encoding");    
        for (Enumeration<?> e = config.getInitParameterNames(); e.hasMoreElements();) {    
            String name = (String) e.nextElement();    
            String value = config.getInitParameter(name);    
            params.put(name, value);    
        }    
    }    
 }    

```
日志过滤器：
```

public class LogFilter implements Filter {    
      
    public FilterConfig config;    
     
    public void destroy() {    
        this.config = null;    
        System.out.println("end do the logging filter!");  
    }    
     
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {    
        System.out.println("before the log filter!");    
        // 将请求转换成HttpServletRequest 请求    
        HttpServletRequest hreq = (HttpServletRequest) req;    
        // 记录日志    
        System.out.println("Log Filter已经截获到用户的请求的地址:"+hreq.getServletPath() );    
        try {    
            // Filter 只是链式处理，请求依然转发到目的地址。    
            chain.doFilter(req, res);    
        } catch (Exception e) {    
            e.printStackTrace();    
        }    
        System.out.println("after the log filter!");    
    }    
     
    public void init(FilterConfig config) throws ServletException {    
        System.out.println("begin do the log filter!");    
        this.config = config;    
    }    
     
 }   

```
测试Servlet:
```

public class FilterServlet extends HttpServlet {  
  
    private static final long serialVersionUID = 1L;  
  
    public void doGet(HttpServletRequest request, HttpServletResponse response)  
            throws ServletException, IOException {  
        response.setDateHeader("expires", -1);  
    }  
  
    public void doPost(HttpServletRequest request, HttpServletResponse response)  
            throws ServletException, IOException {  
    }  
  
} 

```
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-122440.png)
###  Filter的典型应用1：使浏览器不缓存页面的过滤器 
有 3 个 HTTP 响应头字段都可以禁止浏览器缓存当前页面，它们在 Servlet 中的示例代码如下：
''response.setDateHeader("Expires",-1);
response.setHeader("Cache-Control","no-cache");
response.setHeader("Pragma","no-cache");''
并不是所有的浏览器都能完全支持上面的三个响应头，因此最好是同时使用上面的三个响应头 
###  典型应用2 ：字符编码的过滤器 
```

public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException, ServletException  
{  
    arg0.setCharacterEncoding(“UTF-8”);   
  
    arg2.doFilter(arg0, arg1);   
} 

```
###  范例3：图片保护过滤器： 
```

@WebFilter(filterName = "ImageProtetorFilter", urlPatterns = {
        "*.png", "*.jpg", "*.gif" })
public class ImageProtectorFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig)
            throws ServletException {
    }

    @Override
    public void destroy() {
    }

    @Override
    public void doFilter(ServletRequest request,
            ServletResponse response, FilterChain filterChain)
            throws IOException, ServletException {
        System.out.println("ImageProtectorFilter");
        HttpServletRequest httpServletRequest = 
                (HttpServletRequest) request;
        String referrer = httpServletRequest.getHeader("referer");
        System.out.println("referrer:" + referrer);
        if (referrer != null) {
            filterChain.doFilter(request, response);
        } else {
            throw new ServletException("Image not available");
        }
    }
}

```
###  范例4：下载计数过滤器 
由于Filter只有一个实例，只会初始化一次。` 在同时收到多个请求时，我们的下载计数会涉及到线程安全性问题 `。
` 我们可以通过采取同步的方法解决，但是我们应该还有更好的方式，那就是利用**Queue和Executor**解决这个线程安全性问题。 `
所有请求都在一个线程池Executor的队列中放置一项任务。放置任务比较快，因为这是一个异步操作，你不需要等待任务完成。
Executor每次从队列中取出一个任务。` 由于我们的Executor只使用一个线程 `，从而消除了线程安全性问题带来的影响。
```

@WebFilter(filterName = "DownloadCounterFilter",
        urlPatterns = { "/*" })
public class DownloadCounterFilter implements Filter {
	//只使用一个线程的线程池作为队列
    ExecutorService executorService = Executors
            .newSingleThreadExecutor();
    Properties downloadLog;
    File logFile;

    @Override
    public void init(FilterConfig filterConfig)
            throws ServletException {
        System.out.println("DownloadCounterFilter");
        String appPath = filterConfig.getServletContext()
                .getRealPath("/");
        logFile = new File(appPath, "downloadLog.txt");
        if (!logFile.exists()) {
            try {
                logFile.createNewFile();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        downloadLog = new Properties();
        try {
            downloadLog.load(new FileReader(logFile));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void destroy() {
      //关闭线程池
        executorService.shutdown();
    }

    @Override
    public void doFilter(ServletRequest request,
            ServletResponse response, FilterChain filterChain)
            throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;

        final String uri = httpServletRequest.getRequestURI();
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                String property = downloadLog.getProperty(uri);
                if (property == null) {
                    downloadLog.setProperty(uri, "1");
                } else {
                    int count = 0;
                    try {
                        count = Integer.parseInt(property);
                    } catch (NumberFormatException e) {
                        // silent
                    }
                    count++;
                    downloadLog.setProperty(uri,
                            Integer.toString(count));
                }
                try {
                    downloadLog
                            .store(new FileWriter(logFile), "");
                } catch (IOException e) {
                }
            }
        });
        filterChain.doFilter(request, response);
    }
}

```


###  使用过滤器处理异步请求 
```

<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <display-name>Filter Async Application</display-name>

    <filter>
        <filter-name>normalFilter</filter-name>
        <filter-class>com.wrox.AnyRequestFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>

    <filter-mapping>
        <filter-name>normalFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
    </filter-mapping>

    <filter>
        <filter-name>forwardFilter</filter-name>
        <filter-class>com.wrox.AnyRequestFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>

    <filter-mapping>
        <filter-name>forwardFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>FORWARD</dispatcher>
    </filter-mapping>

    <filter>
        <filter-name>asyncFilter</filter-name>
        <filter-class>com.wrox.AnyRequestFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>

    <filter-mapping>
        <filter-name>asyncFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>ASYNC</dispatcher>
    </filter-mapping>

</web-app>


```
```

public class AnyRequestFilter implements Filter
{
    private String name;

    @Override
    public void init(FilterConfig config)
    {
        this.name = config.getFilterName();
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException
    {
        System.out.println("Entering " + this.name + ".doFilter().");
        chain.doFilter(
                new HttpServletRequestWrapper((HttpServletRequest)request),
                new HttpServletResponseWrapper((HttpServletResponse)response)
        );
        if(request.isAsyncSupported() && request.isAsyncStarted())
        {
            AsyncContext context = request.getAsyncContext();
            System.out.println("Leaving " + this.name + ".doFilter(), async " +
                    "context holds wrapped request/response = " +
                    !context.hasOriginalRequestAndResponse());
        }
        else
            System.out.println("Leaving " + this.name + ".doFilter().");
    }

    @Override
    public void destroy() { }
}


```
```

@WebServlet(name = "asyncServlet", urlPatterns = "/async", asyncSupported = true)
public class AsyncServlet extends HttpServlet
{
    private static volatile int ID = 1;

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
    {
        final int id;
        synchronized(AsyncServlet.class)
        {
            id = ID++;
        }
        long timeout = request.getParameter("timeout") == null ?
                10_000L : Long.parseLong(request.getParameter("timeout"));

        System.out.println("Entering AsyncServlet.doGet(). Request ID = " + id +
                ", isAsyncStarted = " + request.isAsyncStarted());

        final AsyncContext context = request.getParameter("unwrap") != null ?
                request.startAsync() : request.startAsync(request, response);
        context.setTimeout(timeout);

        System.out.println("Starting asynchronous thread. Request ID = " + id +
                ".");

        AsyncThread thread = new AsyncThread(id, context);
        context.start(thread::doWork);

        System.out.println("Leaving AsyncServlet.doGet(). Request ID = " + id +
                ", isAsyncStarted = " + request.isAsyncStarted());
    }

    private static class AsyncThread
    {
        private final int id;
        private final AsyncContext context;

        public AsyncThread(int id, AsyncContext context)
        {
            this.id = id;
            this.context = context;
        }

        public void doWork()
        {
            System.out.println("Asynchronous thread started. Request ID = " +
                    this.id + ".");

            try {
                Thread.sleep(5_000L);
            } catch (Exception e) {
                e.printStackTrace();
            }

            HttpServletRequest request =
                    (HttpServletRequest)this.context.getRequest();
            System.out.println("Done sleeping. Request ID = " + this.id +
                    ", URL = " + request.getRequestURL() + ".");

            this.context.dispatch("/WEB-INF/jsp/view/async.jsp");

            System.out.println("Asynchronous thread completed. Request ID = " +
                    this.id + ".");
        }
    }
}


```
由于当service方法返回时FilterChain的doChain方法也将返回。此时过滤器其实无法拦截响应。因此如果想使过滤器可以拦截响应，那么请映射到ASYNC派发器的过滤器将拦截调用**AsyncContext的dispatch方法。**
如果使用AsyncContext直接处理响应对象，代码将在所有过滤器的范围之外执行。

##  使用过滤器压缩响应内容完整项目示例 
```

package com.wrox;

import javax.servlet.FilterRegistration;
import javax.servlet.ServletContext;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;

@WebListener
public class Configurator implements ServletContextListener
{
    @Override
    public void contextInitialized(ServletContextEvent event)
    {
        ServletContext context = event.getServletContext();

        FilterRegistration.Dynamic registration =
                context.addFilter("requestLogFilter", new RequestLogFilter());
        registration.addMappingForUrlPatterns(null, false, "/*");

        registration = context.addFilter("compressionFilter",
                new CompressionFilter());
        registration.setAsyncSupported(true);
        registration.addMappingForUrlPatterns(null, false, "/*");
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) { }
}


```
```

public class RequestLogFilter implements Filter
{
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException
    {
        Instant time = Instant.now();
        StopWatch timer = new StopWatch();
        try
        {
            timer.start();
            chain.doFilter(request, response);
        }
        finally
        {
            timer.stop();
            HttpServletRequest in = (HttpServletRequest)request;
            HttpServletResponse out = (HttpServletResponse)response;
            String length = out.getHeader("Content-Length");
            if(length == null || length.length() == 0)
                length = "-";
            System.out.println(in.getRemoteAddr() + " - - [" + time + "]" +
                    " \"" + in.getMethod() + " " + in.getRequestURI() + " " +
                    in.getProtocol() + "\" " + out.getStatus() + " " + length +
                    " " + timer);
        }
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException { }

    @Override
    public void destroy() { }
}

```

```

public class CompressionFilter implements Filter
{
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException
    {
        if(((HttpServletRequest)request).getHeader("Accept-Encoding")
                .contains("gzip"))
        {
            System.out.println("Encoding requested.");
            ((HttpServletResponse)response).setHeader("Content-Encoding", "gzip");
            ResponseWrapper wrapper =
                    new ResponseWrapper((HttpServletResponse)response);
            try
            {
                chain.doFilter(request, wrapper);
            }
            finally
            {
                try {
                    wrapper.finish();
                } catch(Exception e) {
                    e.printStackTrace();
                }
            }
        }
        else
        {
            System.out.println("Encoding not requested.");
            chain.doFilter(request, response);
        }
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException { }

    @Override
    public void destroy() { }

    private static class ResponseWrapper extends HttpServletResponseWrapper
    {
        private GZIPServletOutputStream outputStream;
        private PrintWriter writer;

        public ResponseWrapper(HttpServletResponse request)
        {
            super(request);
        }

        @Override
        public synchronized ServletOutputStream getOutputStream()
                throws IOException
        {
            if(this.writer != null)
                throw new IllegalStateException("getWriter() already called.");
            if(this.outputStream == null)
                this.outputStream =
                        new GZIPServletOutputStream(super.getOutputStream());
            return this.outputStream;
        }

        @Override
        public synchronized PrintWriter getWriter() throws IOException
        {
            if(this.writer == null && this.outputStream != null)
                throw new IllegalStateException(
                        "getOutputStream() already called.");
            if(this.writer == null)
            {
                this.outputStream =
                        new GZIPServletOutputStream(super.getOutputStream());
                this.writer = new PrintWriter(new OutputStreamWriter(
                        this.outputStream, this.getCharacterEncoding()
                ));
            }
            return this.writer;
        }

        @Override
        public void flushBuffer() throws IOException
        {
            if(this.writer != null)
                this.writer.flush();
            else if(this.outputStream != null)
                this.outputStream.flush();
            super.flushBuffer();
        }

        @Override
        public void setContentLength(int length) { }

        @Override
        public void setContentLengthLong(long length) { }

        @Override
        public void setHeader(String name, String value)
        {
            if(!"content-length".equalsIgnoreCase(name))
                super.setHeader(name, value);
        }

        @Override
        public void addHeader(String name, String value)
        {
            if(!"content-length".equalsIgnoreCase(name))
                super.setHeader(name, value);
        }

        @Override
        public void setIntHeader(String name, int value)
        {
            if(!"content-length".equalsIgnoreCase(name))
                super.setIntHeader(name, value);
        }

        @Override
        public void addIntHeader(String name, int value)
        {
            if(!"content-length".equalsIgnoreCase(name))
                super.setIntHeader(name, value);
        }

        public void finish() throws IOException
        {
            if(this.writer != null)
                this.writer.close();
            else if(this.outputStream != null)
                this.outputStream.finish();
        }
    }

    private static class GZIPServletOutputStream extends ServletOutputStream
    {
        private final ServletOutputStream servletOutputStream;
        private final GZIPOutputStream gzipStream;

        public GZIPServletOutputStream(ServletOutputStream servletOutputStream)
                throws IOException
        {
            this.servletOutputStream = servletOutputStream;
            this.gzipStream = new GZIPOutputStream(servletOutputStream);
        }

        @Override
        public boolean isReady()
        {
            return this.servletOutputStream.isReady();
        }

        @Override
        public void setWriteListener(WriteListener writeListener)
        {
            this.servletOutputStream.setWriteListener(writeListener);
        }

        @Override
        public void write(int b) throws IOException
        {
            this.gzipStream.write(b);
        }

        @Override
        public void close() throws IOException
        {
            this.gzipStream.close();
        }

        @Override
        public void flush() throws IOException
        {
            this.gzipStream.flush();
        }

        public void finish() throws IOException
        {
            this.gzipStream.finish();
        }
    }
}

```
```

@WebServlet(name = "compressedServlet", urlPatterns = "/servlet")
public class CompressedServlet extends HttpServlet
{
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
    {
        response.setContentType("text/plain");
        response.setCharacterEncoding("UTF-8");
        response.getOutputStream()
                .println("This Servlet response may be compressed.");
    }
}


```
```

<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <display-name>Compression Filter Application</display-name>

    <jsp-config>
        <jsp-property-group>
            <url-pattern>*.jsp</url-pattern>
            <url-pattern>*.jspf</url-pattern>
            <page-encoding>UTF-8</page-encoding>
            <scripting-invalid>true</scripting-invalid>
            <include-prelude>/WEB-INF/jsp/base.jspf</include-prelude>
            <trim-directive-whitespaces>true</trim-directive-whitespaces>
            <default-content-type>text/html</default-content-type>
        </jsp-property-group>
    </jsp-config>

    <session-config>
        <session-timeout>30</session-timeout>
        <cookie-config>
            <http-only>true</http-only>
        </cookie-config>
        <tracking-mode>COOKIE</tracking-mode>
    </session-config>

    <distributable />

</web-app>


```
参考：
http://blog.csdn.net/gaogaoshan/article/details/7423132
http://www.cnblogs.com/hxsyl/p/3443047.html
http://blog.csdn.net/jiangwei0910410003/article/details/23372847
http://www.cnblogs.com/xdp-gacl/p/3948353.html