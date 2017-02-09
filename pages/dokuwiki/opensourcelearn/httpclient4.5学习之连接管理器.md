title: httpclient4.5学习之连接管理器 

#  HttpClient4.5学习之连接管理器 
**连接持久化**
两个主机之间建立连接的过程是很复杂的，包括了两个终端之间许多数据包的交换，会消耗了大量时间。对于很小的HTTP报文传输，上层连接的握手也是必须的【译者：上层连接指的是TCP/IP连接】。如果已有的连接能够重复使用，来执行多个请求，将会加大你程序的数据吞吐量。
保持持续连接的能力通常被称为连接持久化。HttpClient完全地支持连接持久化。
**HTTP连接路由**
HttpClient 能够直接建立连接到目标主机，或者通过路由，但这会涉及多个中间连接----也被称为”一跳”。 HttpClient区分连接路由plain, tunneled 和layered。连接到目标主机的隧道使用多个中间代理，被称为**代理链**。
**路由计算**
HttpRoutePlanner 是一个接口，它代表计算到基于执行上下文到给定目标完整路由策略。 HttpClient 装载了 **两 个 默 认 的 HttpRoutePlanner 实 现 。**
  * ProxySelectorRoutePlanner基于 Java.NET.ProxySelector 。默认情况下 ，它会从系统属性中或从正在运行浏览器中选JVM的代理设置 。
  * DefaultHttpRoutePlanner 实现既不使用任何 Java 系统属性，也不使用系统或浏览器的代理设置。它总是通过默认的相同代理来计算路由。

##  HTTP连接管理器 
HTTP 连接是复杂的，有状态的，非线程安全的对象，它需要恰当的管理以便正确地执行。
**ClientConnectionManager**接口就是代表。HTTP 连接管理器的目的是作为工厂创建新的 HTTP 连接，管理持久连接的生命周期和同步访问持久连接（确保同一时间仅有一个线程可以访问一个连接）。内部的 HTTP 连接管理器使用 OperatedClientConnection 实例，这个实例为真正的连接扮演了一个代理，来管理连接状态和控制I/O操作的执行。
如果一个管理的连接被释放或者被使用者明确地关闭，潜在的连接就会从它的代理分离，退回到管理器中。
这里有一个从连接管理器中获取连接的示例：
```

HttpClientContext context = HttpClientContext.create();  
HttpClientConnectionManager connMrg = new BasicHttpClientConnectionManager();  
HttpRoute route = new HttpRoute(new HttpHost("localhost", 80));  
// Request new connection. This can be a long process  
ConnectionRequest connRequest = connMrg.requestConnection(route, null);  
// Wait for connection up to 10 sec  
HttpClientConnection conn = connRequest.get(10, TimeUnit.SECONDS);  
try {  
    // If not open  
    if (!conn.isOpen()) {  
      // establish connection based on its route info  
       connMrg.connect(conn, route, 1000, context);  
       // and mark it as route complete  
        connMrg.routeComplete(conn, route, context);  
    }  
// Do useful things with the connection.  
} finally {  
    connMrg.releaseConnection(conn, null, 1, TimeUnit.MINUTES);  
}

```
如果必要的话，调用ConnectionRequest#cancel()能够尽早地结束连接。这将会使锁定在ConnectionRequest#get()中的线程解锁。

简单的连接管理器
**BasicHttpClientConnectionManager**是一个简单的连接管理器，它保持一次只有一个连接。尽管这个方法是线程安全的，它也应该一次使用一个线程。BasicHttpClientConnectionManager将会城尝试在同一个路由中随后的请求使用同一个连接.如果持续的连接不匹配连接的请求，他将会关闭现有连接并为给定的路由重新打开。如果连接已经被分配，将会抛出java.lang.IllegalStateException异常，连接管理器应该在EJB容器中实现。

连接池管理器Pooling connection manager
**PoolingHttpClientConnectionManager**是一个管理客户端连接更复杂的实现。它也为执行多线程的连接请求提供服务。对于每个基本的路由，连接都是池管理的。对于路由的请求，管理器在池中有可用的持久性连接，将被从池中取出连接服务，而不是创建一个新的连接。

下面例子说明了怎样设置连接池参数是最合适的：
```

PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();  
// Increase max total connection to 200  
cm.setMaxTotal(200);  
// Increase default max connection per route to 20  
cm.setDefaultMaxPerRoute(20);  
// Increase max connections for localhost:80 to 50  
HttpHost localhost = new HttpHost("locahost", 80);  
cm.setMaxPerRoute(new HttpRoute(localhost), 50);  
CloseableHttpClient httpClient = HttpClients.custom()setConnectionManager(cm).build();  

```

##  关闭连接管理器 
当一个HttpClient实例不再被需要时，而且即将超出使用范围，重要是的，关闭连接管理器来保证由管理器保持活动的所有连接被关闭，并且由连接分配的系统资源被释放。
CloseableHttpClient httpClient = <...>
**httpClient.close();**
##  请求执行的多线程 
当配备连接池管理器时，比如 PoolingClientConnectionManager， HttpClient 可以被用使用多线程来同时执行多个请求。
PoolingClientConnectionManager将会基于它的配置来分配连接。如果对于给定路由的所有连接都被使用了，那么连接的请求将会阻塞，直到一个连接被释放回连接池。 它
可以通过设置'http.conn-manager.timeout'为一个正数来保证连接管理器不会在连接请求执行时无限期的被阻塞。 如果连接请求不能在给定的时间内被响应，将会抛ConnectionPoolTimeoutException 异常。
一个HttpClient实例是线程安全的，并且能够在多个执行线程直接共享，**强烈建议每个线程保持一个它自己专有的HttpContext实例。**
##  连接回收策略Connection eviction policy 
一个典型阻塞 I/O 的模型的主要缺点是网络socket仅当 I/O 操作阻塞时才可以响应 I/O
HttpClient 通过测试连接是否是连接是过时的来尝试去减轻这个问题。在服务器端关闭的连接已经不再有效了，优先使用之前使用执行 HTTP 请求的连接。过时的连接检查也并不是100%可靠。唯一可行的而不涉及每个空闲连接的socket模型线程的解决方案是：是使用专用的监控线程来收回因为长时间不活动而被认 为 是 过 期 的 连 接 。 监 控 线 程 可 以 周 期 地 调 用ClientConnectionManager#closeExpiredConnections()方法来关闭所有过期 的 连 接 并且从 连 接 池 中 收 回 关 闭 的 连 接 。 它 也 可 以 选 择 性 调 用ClientConnectionManager#closeIdleConnections()方法来关闭所有已经空闲超过给定时间周期的连接。
```

public static class IdleConnectionMonitorThread extends Thread {  
  private final HttpClientConnectionManager connMgr;  
  private volatile boolean shutdown;  
  public IdleConnectionMonitorThread(HttpClientConnectionManager connMgr) {  
    super();  
    this.connMgr = connMgr;  
  }  
  @Override  
  public void run() {  
    try {  
      while (!shutdown) {  
        synchronized (this) {  
        wait(5000);  
        // Close expired connections  
        connMgr.closeExpiredConnections();  
        // Optionally, close connections  
        // that have been idle longer than 30 sec  
        connMgr.closeIdleConnections(30, TimeUnit.SECONDS);  
      }  
     }  
    } catch (InterruptedException ex) {  
     // terminate  
    }  
  }  
  public void shutdown() {  
    shutdown = true;  
    synchronized (this) {  
      notifyAll();  
    }  
  }  
}  

```
##  连接保持策略 
HTTP 规范没有详细说明一个持久连接可能或应该保持活动多长时间。一些 HTTP 服务器使用非标准的首部Keep-Alive来告诉客户端它们想在服务器端保持连接活动的周期秒数。如果这个信息存在， HttClient 就会使用它。如果响应中首部Keep-Alive 在不存在， HttpClient会假定无限期地保持连接。然而许多 HTTP 服务器的普遍配置是在特定非
活动周期之后丢掉持久连接来保护系统资源，往往这是不通知客户端的。默认的策略是过于乐观的，你必须提供一个自定义的keep-alive策略
```

ConnectionKeepAliveStrategy myStrategy = new ConnectionKeepAliveStrategy() {  
    public long getKeepAliveDuration(HttpResponse response, HttpContext context) {  
        // Honor 'keep-alive' header  
        HeaderElementIterator it = new BasicHeaderElementIterator(  
            response.headerIterator(HTTP.CONN_KEEP_ALIVE));  
        while (it.hasNext()) {  
            HeaderElement he = it.nextElement();  
            String param = he.getName();  
            String value = he.getValue();  
            if (value != null && param.equalsIgnoreCase("timeout")) {  
                try {  
                    return Long.parseLong(value) * 1000;  
                } catch(NumberFormatException ignore) {  
                }  
            }  
        }  
        HttpHost target = (HttpHost) context.getAttribute(  
                    HttpClientContext.HTTP_TARGET_HOST);  
        if ("www.naughty-server.com".equalsIgnoreCase(target.getHostName())) {  
            // Keep alive for 5 seconds only  
            return 5 * 1000;  
        } else {  
            // otherwise keep alive for 30 seconds  
            return 30 * 1000;  
        }  
    }  
};  
CloseableHttpClient client = HttpClients.custom().setKeepAliveStrategy(myStrategy).build();  

```
##  连接socket工厂 
HTTP连接内部使用java.net.Socket 来通过网线处理传输数据。可是，他们依靠**ConnectionSocketFactory**接口来创建、初始化和连接socket。HttpClient为使用者提供了在程序运行时明确的socket初始化代码。**PlainConnectionSocketFactory**是创建，初始化普通的socket（非加密socket）的工厂。
```

HttpClientContext clientContext = HttpClientContext.create();  
PlainConnectionSocketFactory sf = PlainConnectionSocketFactory.getSocketFactory();  
Socket socket = sf.createSocket(clientContext);  
int timeout = 1000; //ms  
HttpHost target = new HttpHost("localhost");  
InetSocketAddress remoteAddress =   
      new InetSocketAddress(netAddress.getByAddress(new byte[] {127,0,0,1}), 80);  
sf.connectSocket(timeout, socket, target, remoteAddress, null, clientContext);  

```
安全套接字分层**LayeredConnectionSocketFactory**是ConnectionSocketFactory接口的拓展。套接字分层主要通过代理来创建安全的socket。
HttpClient装载实现了 SSL/TLS 分层的 **SSLSocketFactory**。 请注意 HttpClient 不使用任何自定义加密功能。它完全依赖于标准的Java Cryptography（JCE）和Secure Sockets（JSEE）扩展
###  与连接管理器整合 
自定义的连接socket工厂与一个被称为HTTP或HTTPS的特定协议模式有关，它被用来创建自定义连接管理器。
```

ConnectionSocketFactory plainsf = <...>  
LayeredConnectionSocketFactory sslsf = <...>  
Registry<ConnectionSocketFactory> r = RegistryBuilder.<ConnectionSocketFactory>create()  
    .register("http", plainsf).register("https", sslsf).build();  
HttpClientConnectionManager cm = new PoolingHttpClientConnectionManager(r);  
HttpClients.custom().setConnectionManager(cm).build();  

```

###  定制SSL/TLS 
HttpClient 使用 **SSLSocketFactory**来创建 SSL连接。SSLSocketFactory允许高度定制。它可以使用** javax.net.ssl.SSLContext** 的实例作为参数， 并使用它来创建能够自定义配置的 SSL连接。
```

KeyStore myTrustStore = <...>  
SSLContext sslContext = SSLContexts.custom().loadTrustMaterial(myTrustStore).build();  
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext);

``` 
SSLSocketFactory的定制意味着对SSL/TLS协议概念有一定的熟悉，详细说明它超出
了本文档的范围。请参考Java™Secure Socket Extension(JSSE) 参考指南[http://java.sun.com/j2se/1.5.0/docs/guide/security/jsse/JSSERefGuide.html]，从而获得javax.net.ssl.SSLContext的详细描述和相关工具。
###  主机名核实 
为了信任证明并且客户端的表现在SSL/TLS级别上，**HttpClient核实是否目标主机名匹配储存在X.509服务器上的证书**。
**javax.net.ssl.HostnameVerifier**接口代表了主机名验证的策略。 HttpClient有两个
javax.net.ssl.HostnameVerifier的实现。重点：主机名验证不应该和SSL信任验证混淆。
·**DefaultHostnameVerifier**：遵从RFC 2818规范，被HttpClient默认实现。主机名必须根据规定的证书匹配被几个可选择名字，如果没有合适的名字，将会给出最具体的证书CN 。一个通配符可能会出现在CN中和任何可选择的地方。
·**NoopHostnameVerifier**：主机名验证器根本就关掉了主机名验证。他接受任何合法的SSL会话，并且匹配目标主机
每次HttpClient默认使用DefaultHostnameVerifier实现。**如果必要的话，你可以制定一个不同的主机名认证器实现。**
```

SSLContext sslContext = SSLContexts.createSystemDefault();  
SSLConnectionSocketFactory sslsf =   
       new SSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE); 

```
在HttpClient 4.4 版本，使用被Mozilla Foundation维护的公共的后缀列表，确保SSL证书中的通配符不能够被滥用来在顶级域名下启用多个域名。HttpClient在发布时复制了一份列表。
最新的版本列表能够在https://publicsuffix.org/list/ [https://publicsuffix.org/list/
effective_tld_names.dat]找到。强烈建议下载一份列表，防止在每天使用时访问原始位置。
```

PublicSuffixMatcher publicSuffixMatcher = PublicSuffixMatcherLoader.load(  
    PublicSuffixMatcher.class.getResource("my-copy-effective_tld_names.dat"));  
DefaultHostnameVerifier hostnameVerifier = new   
                             DefaultHostnameVerifier (publicSuffixMatcher) 

```
  
##  HttpClient代理配置 
尽管HttpClient了解复杂的路由模式和路由链，但它只支持简单的重定向或者一跳代理连接。
告诉HttpClient通过代理连接目标服务器最简单的方式是设置默认的代理参数
```

HttpHost proxy = new HttpHost("someproxy", 8080);
DefaultProxyRoutePlanner routePlanner = new DefaultProxyRoutePlanner(proxy);
CloseableHttpClient httpclient = HttpClients.custom() .setRoutePlanner(routePlanner) .build();

```
你也可以实现一个自定义的RoutePlanner，来实现一个在完整控制HTTP路由计算的处理器。
```

HttpRoutePlanner routePlanner = new HttpRoutePlanner() {  
  public HttpRoute determineRoute( HttpHost target, HttpRequest request,  
                                 HttpContext context) throws HttpException {  
    return new HttpRoute(target, null, new HttpHost("someproxy", 8080),  
          "https".equalsIgnoreCase(target.getSchemeName()));  
  }  
};  
CloseableHttpClient httpclient = HttpClients.custom() .setRoutePlanner(routePlanner).build();  

```
