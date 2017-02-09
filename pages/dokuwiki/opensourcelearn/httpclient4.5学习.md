title: httpclient4.5学习 

#  HttpClient4.5学习 
HttpClient版本API变化很大，本系列以4.5为主。
尽管Java.NET包提供了通过HTTP访问资源的基本功能，但它缺少足够的灵活性和其它很多应用程序需要的功能。HttpClient通过提供一个有效的，保持更新的，功能丰富的软件包来实现客户端最新的HTTP标准和建议，来弥补java.net包的在某些技术上的空白。 HttpClient为扩展而设计，同时为基本的HTTP协议提供强大的支持。有一些人会对HttpClient感兴趣，这些人通常是构建 HTTP 客户端应用程序（比如web浏览器，web服务客户端，利用或扩展 HTTP 协议进行来实现的分布式通信系统）的开发人员。
HttpClient 的范围
●基于HttpCore[http://hc.apache.org/httpcomponents-core/index.html]的客户端HTTP通信库
●基于经典（阻塞） I/O
●内容无关
HttpClient 所不能做的
●HttpClient 不是一个浏览器。它是一个客户端的 HTTP 通信实现库。HttpClient 的目标是发送和接收HTTP 报文。HttpClient 不会去处理内容，执行嵌入在 HTML页面中的JavaScript 代码，猜测内容类型，如果没有明确设置，否则不会重新格式化请求/重定向URI，或其它和HTTP通信无关的功能。

API:http://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/
##  执行请求 
HttpClient最基本的功能是执行HTTP方法。使用者被要求提供一个Request对象来执行，HttpClient就会把请求传送给目标服务器并返回一个相对应的response对象，如果执行不成功，将会抛出一个异常。
```

CloseableHttpClient httpclient = HttpClients.createDefault();  
HttpGet httpget = new HttpGet("http://localhost/");  
CloseableHttpResponse response = httpclient.execute(httpget);  
try {  
   <...>  
} finally {  
   response.close();  
}  

```

###  HTTP 请求（Request） 
所有 HTTP 请求都有一个请求起始行，这个起始行由方法名，请求 URI 和 HTTP 协议版本组成。HttpClient很好地支持了HTTP/1.1规范中所有的HTTP方法：GET，HEAD， POST，PUT， DELETE， TRACE 和 OPTIONS。每个方法都有一个特别的类：HttpGet，HttpHead， HttpPost，HttpPut，HttpDelete，HttpTrace和HttpOptions。
URI是统一资源标识符的缩写，用来标识与请求相符合的资源。HTTP 请求URI包含了一个协议名称，主机名，可选端口，资源路径，可选的参数，可选的片段。
HttpClient提供了**URIBuilder**工具类来简化创建、修改请求 URIs。
```

URI uri = new URIBuilder()  
          .setScheme("http")  
          .setHost("www.google.com")  
          .setPath("/search")  
          .setParameter("q", "httpclient")  
          .setParameter("btnG", "Google Search")  
          .setParameter("aq", "f")  
          .setParameter("oq", "")  
          .build();  
HttpGet httpget = new HttpGet(uri);  
System.out.println(httpget.getURI());

```
###  HTTP 响应（Response） 
```

HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1  
                             ,HttpStatus.SC_OK, "OK");  
System.out.println(response.getProtocolVersion());  
System.out.println(response.getStatusLine().getStatusCode());  
System.out.println(response.getStatusLine().getReasonPhrase());  
System.out.println(response.getStatusLine().toString());  

```
###  处理报文首部（Headers） 
一个HTTP报文包含了许多描述报文的首部，比如内容长度，内容类型等。HttpClient提供了一些方法来取出，添加，移除，枚举首部。
```

HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,HttpStatus.SC_OK, "OK");  
response.addHeader("Set-Cookie","c1=a; path=/; domain=localhost");  
response.addHeader("Set-Cookie","c2=b; path=\"/\", c3=c; domain=\"localhost\"");  
Header h1 = response.getFirstHeader("Set-Cookie");  
System.out.println(h1);  
Header h2 = response.getLastHeader("Set-Cookie");  
System.out.println(h2);  
Header[] hs = response.getHeaders("Set-Cookie");  
System.out.println(hs.length);  

```
获得所有指定类型首部最有效的方式是使用**HeaderIterator接口**
```

HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,HttpStatus.SC_OK, "OK");  
response.addHeader("Set-Cookie","c1=a; path=/; domain=localhost");  
response.addHeader("Set-Cookie","c2=b; path=\"/\", c3=c; domain=\"localhost\"");  
HeaderIterator it = response.headerIterator("Set-Cookie");  
while (it.hasNext()) {  
     System.out.println(it.next());  
}  

```
输出：
Set-Cookie: c1=a; path=/; domain=localhost
Set-Cookie: c2=b; path="/", c3=c; domain="localhost"

HttpClient也提供了其他便利的方法吧HTTP报文转化为单个的HTTP元素。
```

HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,HttpStatus.SC_OK, "OK");  
response.addHeader("Set-Cookie","c1=a; path=/; domain=localhost");  
response.addHeader("Set-Cookie","c2=b; path=\"/\", c3=c; domain=\"localhost\"");  
HeaderElementIterator it = new BasicHeaderElementIterator(  
response.headerIterator("Set-Cookie"));  
while (it.hasNext()) {  
     HeaderElement elem = it.nextElement();  
     System.out.println(elem.getName() + " = " + elem.getValue());  
     NameValuePair[] params = elem.getParameters();  
     for (int i = 0; i < params.length; i++) {  
          System.out.println(" " + params[i]);  
     }  
}  

```
输出：
c1 = a
path=/
domain=localhost
c2 = b
path=/
c3 = c
domain=localhost

###  HTTP实体（HTTP Entity） 
**HTTP报文能够携带与请求或相应相关联的内容实体。**实体存在于某些请求、响应中，它门是可选的。使用实体的请求被称为**内含实体请求【entity enclosing requests】**。HTTP规范定义了**两种内含实体请求，POST和PUT**。而**响应总是内含实体**。但有些响应不符合这一规则，比如，对HEAD方法的响应和状态为204 No Content, 304 Not Modified, 205 Reset Content的响应。

依据实体的内容来源，HttpClient区分出三种实体：
  * 流式实体（streamed）：内容来源于一个流，或者在运行中产生【generated on the fly】。特别的，这个类别包括从响应中接收到的实体。流式实体不可重复。
  * 自包含实体（self-contained）：在内存中的内容或者通过独立的连接/其他实体获得的内容。**自包含实体是可重复的。这类实体大部分是HTTP内含实体请求。**
  * 包装实体（wrapping）：从另外一个实体中获取内容。
一个实体能够被重复，意味着它的内容能够被读取多次。它仅可能是自包含实体（像ByteArrayEntity或StringEntity）

由于一个实体能够表现为二进制和字符内容，它支持二进制编码（为了支持后者，即字符内容）。
实体将会在一些情况下被创建：当执行一个含有内容的请求时或者当请求成功，响应体作为结果返回给客户端时。
为了读取实体的内容，可以通过**HttpEntity#getContent()** 方法取出输入流，返回一个Java.io.InputStream，或者提供一个输出流给**HttpEntity#writeTo(OutputStream)** 方法，它将会返回写入给定流的所有内容。
当实体内部含有信息时，使用**HttpEntity#getContentType()**和**HttpEntity#getContentLength()**方法将会读取一些基本的元数据，比如Content-Type和Content-Length这样的首部，**HttpEntity#getContentEncoding()**方法被用来读取这些字符编码。如果对应的首部不存在，则Content-Length的返回值为-1，Content-Type返回值为NULL。如果Content-Type是可用的，一个Header类的对象将会返回。

```

StringEntity myEntity = new StringEntity("important message",  
                          ContentType.create("text/plain", "UTF-8"));  
System.out.println(myEntity.getContentType());  
System.out.println(myEntity.getContentLength());  
System.out.println(EntityUtils.toString(myEntity));  
System.out.println(EntityUtils.toByteArray(myEntity).length);  

```
**确保释放低级别的资源**
为了确保正确的释放系统资源，你必须关掉与实体与实体相关的的内容流，还必须关掉响应本身。
```

CloseableHttpClient httpclient = HttpClients.createDefault();  
HttpGet httpget = new HttpGet("http://localhost/");  
CloseableHttpResponse response = httpclient.execute(httpget);  
try {  
     HttpEntity entity = response.getEntity();  
     if (entity != null) {  
        InputStream instream = entity.getContent();  
        try {  
            // do something useful  
        } finally {  
            instream.close();  
        }  
   }  
} finally {  
     response.close();  
}  

```
**在某些情况下，多次读取实体内容是必要的。**在这种情况下，实体内容必须以一些方式**缓冲**，内存或者硬盘中。
为了达到这个目的，最简单的方法是把原始的实体用**BufferedHttpEntity**类包装起来。这将会使原始实体的内容读入一个in-memory缓冲区。所有方式的实体包装都是代表最原始的那个实体。
```

CloseableHttpResponse response = <...>  
HttpEntity entity = response.getEntity();  
if (entity != null) {  
    entity = new BufferedHttpEntity(entity);  
} 

```

###  例：模拟浏览器GET请求访问新浪网 
问题：模拟浏览器访问新浪网http://www.sina.com.cn/并解析返回结果
一、分析
经过前面的学习，已经能掌握了GET请求并解析返回结果，如下图：
![](/data/dokuwiki/opensourcelearn/pasted/20161008-141730.png)

一个使用HttpClient4.5典型的GET访问步骤为：
1.构建HttpClient
2.构建请求（起始行、首部）
3.使用HttpClient执行请求
4.解析相应（起始行、首部、实体）
另外还包括释放资源HttpClient、实体、响应

二、构建
![](/data/dokuwiki/opensourcelearn/pasted/20161008-141842.png)

1.构建HttpClient
```

CloseableHttpClient client=HttpClients.createDefault(); 

``` 
程序结尾需要关闭
` client.close(); `

2.构建、执行请求
```

//请求起始行--HttpClient会根据信息自动构建  
HttpGet get=new HttpGet("http://www.sina.com.cn/");  
//请求首部--可选的，User-Agent对于一些服务器必选，不加可能不会返回正确结果  
get.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:39.0) Gecko/20100101 Firefox/39.0");  
//执行请求  
CloseableHttpResponse response=client.execute(get);  

```

分析响应
```

//获得起始行  
System.out.println(response.getStatusLine().toString()+"\n");  
//获得首部---当然也可以使用其他方法获取  
Header[] hs=response.getAllHeaders();  
for(Header h:hs){  
    System.out.println(h.getName()+":\t"+h.getValue()+"\n");  
}  
//获取实体  
HttpEntity ety=response.getEntity();  
System.out.println(EntityUtils.toString(ety,"GBK"));//新浪网的编码格式个GBK  
EntityUtils.consume(ety);//释放实体  
          
response.close();//关闭响应  

```


###  生产实体内容 
HttpClient提供了几个类，用来通过HTTP连接高效地传输内容。这些类的实例均与内含实体请求有关，比如POST和PUT，它们能够把实体内容封装进友好的HTTP请求中。
对于基本的数据容器String, byte array, input stream, and file，HttpClient为它们提供了几个对应的类：**StringEntity, ByteArrayEntity, InputStreamEntity, and FileEntity。**
```

File file = new File("somefile.txt");  
FileEntity entity = new FileEntity(file,ContentType.create("text/plain", "UTF-8"));  
HttpPost httppost = new HttpPost("http://localhost/action.do");  
httppost.setEntity(entity);  

```
请注意InputStreamEntity是不可重复的，因为它仅仅能够从数据流中读取一次。一般建议实现一个定制的HttpEntity类来代替使用一般的InputStreamEntity。FileEntity将会是一个好的出发点。

###  HTML表单 
许多应用需要模仿一个登陆HTML表单的过程，比如：为了登陆一个web应用或者提交输入的数据。HttpClient提供了**UrlEncodedFormEntity**类来简化这个过程。
```

List<NameValuePair> formparams = new ArrayList<NameValuePair>();  
formparams.add(new BasicNameValuePair("param1", "value1"));  
formparams.add(new BasicNameValuePair("param2", "value2"));  
UrlEncodedFormEntity entity = new UrlEncodedFormEntity(formparams,  
                                                          Consts.UTF_8);  
HttpPost httppost = new HttpPost("http://localhost/handler.do");  
httppost.setEntity(entity);  

```
UrlEncodedFormEntity实例像上面一样使用URL编码方式来编码参数并生成下面的内容：
param1=value1&param2=value2
###  内容分块 
通常，我们推荐让HttpClient选择基于被传递的HTTP报文属相最合适的传输编码方式。可能地，可以通过设置**HttpEntity#setChunked()**为true来通知HttpClient你要进行分块编码。注意HttpClient将会使用这个标志作为提示。当使用一些不支持分块编码的HTTP版本（比如HTTP/1.0.）时，这个值将会忽略。
【译者：分块编码是是HTTP1.1协议中定义的Web用户向服务器提交数据的一种方法】
```

StringEntity entity = new StringEntity("important message",  
                           ContentType.create("plain/text", Consts.UTF_8));  
entity.setChunked(true);  
HttpPost httppost = new HttpPost("http://localhost/acrtion.do");  
httppost.setEntity(entity); 

```
###  响应处理器 
最简单、最方便的方式来处理响应是使用**ResponseHandler**接口，它包含了一个handleResponse(HttpResponse response)方法。这个方法减轻使用者对于连接管理的担心。
**当你使用ResponseHandler时，无论是请求执行成功亦或出现异常，HttpClient将会自动地确保连接释放回连接管理器中。**
```

CloseableHttpClient httpclient = HttpClients.createDefault();  
HttpGet httpget = new HttpGet("http://localhost/json");  
ResponseHandler<MyJsonObject> rh = new ResponseHandler<MyJsonObject>() {  
    @Override  
    public JsonObject handleResponse(final HttpResponse response) throws IOException {  
        StatusLine statusLine = response.getStatusLine();  
        HttpEntity entity = response.getEntity();  
        if (statusLine.getStatusCode() >= 300) {  
            throw new HttpResponseException(statusLine.getStatusCode(),  
                statusLine.getReasonPhrase());  
        }  
        if (entity == null) {  
             throw new ClientProtocolException("Response contains no content");  
        }  
        Gson gson = new GsonBuilder().create();  
        ContentType contentType = ContentType.getOrDefault(entity);  
        Charset charset = contentType.getCharset();  
        Reader reader = new InputStreamReader(entity.getContent(), charset);  
        return gson.fromJson(reader, MyJsonObject.class);  
    }  
};  
MyJsonObject myjson = client.execute(httpget, rh);  

```

##  HttpClient接口 
HttpClient代表HTTP请求执行的最基本约定。它保留了连接管理，状态管理，认证，重定向等处理细节的个人实现。使用额外的功能来装饰这个接口是非常容易的，比如设置响应体缓存。
下面例子使用内部类演示了自定义持续连接的实现
```

ConnectionKeepAliveStrategy keepAliveStrat = new DefaultConnectionKeepAliveStrategy() {  
@Override  
public long getKeepAliveDuration(HttpResponse response,HttpContext context) {  
       long keepAlive = super.getKeepAliveDuration(response, context);  
       if (keepAlive == -1) {  
        // 如果keep-alive的值内有被服务器明确设置，保持持续连接5秒  
       keepAlive = 5000;  
    }  
    return keepAlive;  
  }  
};  
CloseableHttpClient httpclient = HttpClients.custom().setKeepAliveStrategy(keepAliveStrat).build();  

```
###  HttpClient线程安全 
HttpClient实现是线程安全的。对于不同的请求执行，这个类的相同实例是可复用的。
【译者：也就是说，一组相关的请求由同一个HttpClient实例实现】
HttpClient资源分配
当一个HttpClient实例不再需要时并且它不在连接管理器之内时，必须通过**CloseableHttpClient#close()**方法关闭。

##  Http执行上下文(context) 
最初，HTTP是被设计成无状态的，面向请求-响应的协议。然而，现实世界中的应用程序经常需要通过一些逻辑相关的请求-响应交换来保持状态信息。 为了使应用程序能够维持一个过程状态， **HttpClient允许HTTP请求在一个特定的执行上下文中来执行--称为HTTP上下文。**如果相同的上下文在连续请求之间重用，那么多种逻辑相关的请求可以参与到一个逻辑会话中。HTTP上下文仅仅是任意命名参数值的集合。应用程序可以在请求之前填充上下文属性，也可以在执行完成之后来检查上下文属性。

**HttpContext**能够包含任意的对象，因此在两个不同的线程中共享上下文是不安全的，**建议每个线程都一个它自己执行的上下文。**
在HTTP请求执行的这一过程中， HttpClient添加了下列属性到执行上下文中：
  * `HttpConnection实例代表连接到目标服务器的当前连接。
  * `HttpHost实例代表连接到目标服务器的当前连接。
  * `HttpRoute实例代表了完整的连接路由。
  * `HttpRequest实例代表了当前的HTTP请求。
  * `HttpResponse实例代表当前的HTTP响应。
  * `java.lang.Boolean对象是一个标识，它标志着当前请求是否完整地传输给连接目标。
  * `RequestConfig代表当前请求**配置**。
  * `java.util.List<URI>对象代表一个含有执行请求过程中**所有的重定向地址。**
代表一个逻辑相关的会话中的多个请求序列应该被同一个HttpContext实例执行，以确保请求之间会话上下文和状态信息的自动传输。
下面的例子中：请求配置在最初被初始化，它将在执行上下文中一直保持。共享与同一个会话中所有连续的请求。
```

 CloseableHttpClient httpclient = HttpClients.createDefault();  
 // Create  HTTP context  
HttpClientContext context = HttpClientContext.create();

RequestConfig requestConfig = RequestConfig.custom()  
    .setSocketTimeout(1000)    //超时设置，毫秒为单位
    .setConnectTimeout(1000)  
    .build();  
HttpGet httpget1 = new HttpGet("http://localhost/1");  
httpget1.setConfig(requestConfig);  
CloseableHttpResponse response1 = httpclient.execute(httpget1, context);  
try {  
    HttpEntity entity1 = response1.getEntity();  
} finally {  
    response1.close();  
}  
HttpGet httpget2 = new HttpGet("http://localhost/2");  
CloseableHttpResponse response2 = httpclient.execute(httpget2, context);  
try {  
    HttpEntity entity2 = response2.getEntity();  
} finally {  
    response2.close();  
} 

```
```

public class ClientCustomContext {  
  
    public final static void main(String[] args) throws Exception {  
        CloseableHttpClient httpclient = HttpClients.createDefault();  
        try {  
            // Create a local instance of cookie store  
            CookieStore cookieStore = new BasicCookieStore();  
  
            // Create local HTTP context  
            HttpClientContext localContext = HttpClientContext.create();  
            // Bind custom cookie store to the local context  
            localContext.setCookieStore(cookieStore);  
  
            HttpGet httpget = new HttpGet("http://localhost/");  
            System.out.println("Executing request " + httpget.getRequestLine());  
  
            // Pass local context as a parameter  
            CloseableHttpResponse response = httpclient.execute(httpget, localContext);  
            try {  
                System.out.println("----------------------------------------");  
                System.out.println(response.getStatusLine());  
                List<Cookie> cookies = cookieStore.getCookies();  
                for (int i = 0; i < cookies.size(); i++) {  
                    System.out.println("Local cookie: " + cookies.get(i));  
                }  
                EntityUtils.consume(response.getEntity());  
            } finally {  
                response.close();  
            }  
        } finally {  
            httpclient.close();  
        }  
    }  
  
}  

```

##  HTTP协议拦截器 
协议拦截器将作用于报文的一个特定的首部或一组相关的首部。或者添加一个特定的首部或一组相关的首部到将要发送的报文中。协议拦截器也可以操作报文内含的实体--显而易见的内容解压/压缩就是一个好的例子。
协议拦截器能够通过共享信息来合作--比如处理状态--通过HTTP上下文。协议拦截器使用HTTP上下文为一次请求或几个关联请求储存一个处理状态。
如果这些拦截器具有相具有依赖关系，就必须以一个特定的顺序执行。比如希望他们以某个顺序执行，就必须以相同的序列加到协议进程中。
**协议拦截器必须被实现为线程安全的。**
下面的例子说明了本地上下文在连续请求中保留处理状态的用法。
```

CloseableHttpClient httpclient = HttpClients.custom()
        .addInterceptorLast(new HttpRequestInterceptor() {

            public void process(
                    final HttpRequest request,
                    final HttpContext context) throws HttpException, IOException {
                AtomicInteger count = (AtomicInteger) context.getAttribute("count");
                request.addHeader("Count", Integer.toString(count.getAndIncrement()));
            }

        })
        .build();

AtomicInteger count = new AtomicInteger(1);
HttpClientContext localContext = HttpClientContext.create();
localContext.setAttribute("count", count);

HttpGet httpget = new HttpGet("http://localhost/");
for (int i = 0; i < 10; i++) {
    CloseableHttpResponse response = httpclient.execute(httpget, localContext);
    try {
        HttpEntity entity = response.getEntity();
    } finally {
        response.close();
    }
}


```
##  异常处理 
HttpClient 能够抛出两种类型的异常：
1）Java.io.IOException ：在 I/O 失败时，如socket连接超时或被重置的异常；
2）HttpException：标志 HTTP 请求失败的信号，如违反 HTTP 协议。通常 I/O 错误被认为是非致命的和可以恢复的，而 HTTP 协议错误，则被认为是致命的而且是不能自动恢复的。
###  请求重试处理器（Request retry handler） 
 为了能够使用自定义异常的恢复机制，你必须提供一个**HttpRequestRetryHandler**接口的实现。
请注意你可以使用StandardHttpRequestRetryHandler代替默认使用的，以便处理那些被RFC-2616定义为幂等的并且能够安全的重试的请求方法。方法有：GET, HEAD, PUT, DELETE, OPTIONS, and TRACE。

##  终止请求 
HTTP requests being executed by HttpClient can be aborted at any stage of execution by invoking **HttpUriRequest#abort()** This method is thread-safe and can be called from any thread. 

##  重定向处理 
HttpClient可以自动处理处于HTTP规范下的所有重定向类型。但是不能处理类以POST、PUT重定向请求，它们会被转换成GET请求，而在某些服务器会被拒绝。但是由于HttpClient的可扩展性，你可以自定义redirect strategy 来处理POST的重定向.而org.apache.http.impl.client.LaxRedirectStrategy提供了一个默认实现。
```

LaxRedirectStrategy redirectStrategy = new LaxRedirectStrategy();
CloseableHttpClient httpclient = HttpClients.custom()
        .setRedirectStrategy(redirectStrategy)
        .build();

```
```

package org.apache.http.impl.client;

import org.apache.http.annotation.Immutable;
import org.apache.http.impl.client.DefaultRedirectStrategy;

@Immutable
public class LaxRedirectStrategy extends DefaultRedirectStrategy {
    private static final String[] REDIRECT_METHODS = new String[]{"GET", "POST", "HEAD"};
    public LaxRedirectStrategy() {
    }
    protected boolean isRedirectable(String method) {
        String[] arr$ = REDIRECT_METHODS;
        int len$ = arr$.length;

        for(int i$ = 0; i$ < len$; ++i$) {
            String m = arr$[i$];
            if(m.equalsIgnoreCase(method)) {  //只要是GET、POST、HEAD请求都自动重定向
                return true;
            }
        }

        return false;
    }
}

```
也可以参考stackoverflow上的一个实现:http://stackoverflow.com/questions/3658721/httpclient-4-error-302-how-to-redirect
```

httpclient.setRedirectStrategy(new DefaultRedirectStrategy() {                
        public boolean isRedirected(HttpRequest request, HttpResponse response, HttpContext context)  {
            boolean isRedirect=false;
            try {
                isRedirect = super.isRedirected(request, response, context);
            } catch (ProtocolException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            if (!isRedirect) {
                int responseCode = response.getStatusLine().getStatusCode();
                if (responseCode == 301 || responseCode == 302) {
                    return true;
                }
            }
            return isRedirect;
        }
    });

```
如何获取多次重定向后最终地址：
The final interpreted absolute HTTP location can be built using the original request and the context. 
The utility method **URIUtils#resolve** can be used to build the interpreted absolute URI used to generate the final request.
```

CloseableHttpClient httpclient = HttpClients.createDefault();
HttpClientContext context = HttpClientContext.create();
HttpGet httpget = new HttpGet("http://localhost:8080/");
CloseableHttpResponse response = httpclient.execute(httpget, context);
try {
    HttpHost target = context.getTargetHost();
    List<URI> redirectLocations = context.getRedirectLocations();
    URI location = URIUtils.resolve(httpget.getURI(), target, redirectLocations);
    System.out.println("Final HTTP location: " + location.toASCIIString());
    // Expected to be an absolute URI
} finally {
    response.close();
}

```

本系列参考：
http://blog.csdn.net/u011179993/article/category/5694697
http://hc.apache.org/httpcomponents-client-ga/tutorial/html/
http://www.cnblogs.com/jcli/archive/2012/10/17/2727632.html