title: nginx_tomcat上传大小限制 

#  nginx+tomcat上传大小限制问题解决 
TOMCAT
问题的根源：tomcat默认设置能接收HTTP POST请求的大小最大为2M,如果你的POST请求传递的数据大于2M,就会报错误。
解决的办法：修改tomcat的配置文件C:/MinyooCMS/tomcat/conf/server.xml(或者安装在D盘文件路径是D: /MinyooCMS/tomcat/conf/server.xml),找到里面的<Connector>标签,在该标签中添加"maxPostSize"属性,将该属性值设置成你想要的最大值,单位是字节,或者把这个值设置为 1000M (maxPostSize="1000000000"),(` 注意：网上所传，设置为0会出现问题。 `)即可解决上述问题。


NGINX
利用nginx做了play的前端服务器，应用一切正常，但是管理后台上传文件时，受到了限制，原来是nginx的一个参数惹的祸！ client_max_body_size这个参数限制了上传文件的大小，默认是1M，此参数是在代理设置文件中配置的， 下面是我的proxy.conf 配置信息。  

location / {

proxy_pass        http://fabo;

proxy_redirect          off;

proxy_set_header   Host             $host:80;

proxy_set_header   X-Real-IP        $remote_addr;

proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;

client_max_body_size    1000m;

}

测试一下配置文件/usr/local/nginx/sbin/nginx -t 

重启nginx：kill -HUP `cat /usr/local/nginx/logs/nginx.pid` 

这里我的设置是1000M的上限，通过修改client_max_body_size 设置的大小，重启nginx服务，解决了文件上传问题!

##  附录Tomcat中的Connector配置  
JBoss使用Tomcat作为Web容器，因此在JBoss中对于Web容器的配置也类似于在Tomcat中的配置，主要就是对于 server.xml文件的编辑，在JBoss 5.x中，这个文件位于${JBOSS.HOME}\server\${confifure}\deploy\jbossweb.sar下，其中 configure的值可以是all, default,web，standard, minimal等。下面的代码展示了一个JBoss default配置下的server.xml，由于篇幅原因，将其中的注释都已经去掉了。
```

<Server>  
   <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />  
   <Listener className="org.apache.catalina.core.JasperListener" />  
   <Service name="jboss.web">  
      <Connector protocol="HTTP/1.1" port="8080" address="${jboss.bind.address}"   
               connectionTimeout="20000" redirectPort="8443" compression="on"   
               compressionMinSize="1" compressableMimeType="text/html,text/xml" />  
      <Engine name="jboss.web" defaultHost="localhost">  
         <Realm className="org.jboss.web.tomcat.security.JBossWebRealm"  
            certificatePrincipal="org.jboss.security.auth.certs.SubjectDNMapping"  
            allRolesMode="authOnly"  
            />  
         <Host name="localhost">   
            <Valve className="org.jboss.web.tomcat.service.jca.CachedConnectionValve"  
            cachedConnectionManagerObjectName="jboss.jca:service=CachedConnectionManager"  
            transactionManagerObjectName="jboss:service=TransactionManager" />  
         Host>  
      Engine>  
   Service>  
Server> 

``` 
在上面的配置文件中，Server是根节点，一个Server就代表一个Servlet容器，因此在server.xml中，这个节点只能有一个，在Server节点下，可以存在一个或者多个Service节点。
 
 一个Service节点代表了一个或者多个Connector和一个Engine，而Connector和Engine是在server.xml中两个重 要的配置项，Connector的主要功能是接受、响应用户请求。常用的Connector有HTTP/1.1 Connector和AJP Connector，HTTP/1.1 Connector主要用于处理用户的HTTP请求，需要注意的是虽然它名叫HTTP/1.1 Connector，但是是完全兼容HTTP/1.0协议的。AJP Connector主要使用AJP协议和Web Connector通信，通常用于集群中。
 
 HTTP/1.1 Connector的实例监听在用户配置的端口上，当应用服务器启动时，HTTP/1.1 Connector负责创建若干线程，用于处理用户请求，创建的线程数目取决于用户配置的minThreads值，默认为5，当有更多的用户请求到来 时，HTTP/1.1 Connector将会创建更多的线程用于处理请求，创建线程的最大值由maxThreads定义，默认值为20，当所有的线程都在忙于处理用户请求时， 新到来的请求将会放入HTTP/1.1 Connector创建的Socket队列中，队列的长度由acceptCount属性定义，当等待队列也被占用满了，新来的用户请求将会收到connection refused错误。
 
 所有的Connector提供的配置项（不完全版scheme, isSecure, xpoweredBy, useIPVHosts ）：
allowTrace 如果需要服务器能够处理用户的HAED/TRACE请求，这个值应该设置为true，默认值是false;
emptySessionPath 如果设置为true，所有session,cookie的path将会被设置为/，这种设置通常是在portlet中比较有用，默认值是false；
enableLookups 如果需要在调用request.getRemoteHost()方法时获取到客户端的机器名，则需要配置为true,如果配置为false，将会跳过DNS查询直接返回客户端机器的IP地址，通常为了提高性能，将此值设置为false，默认值是true;
maxPostSize POST方法能够提交的数据的最大大小，如果没有声明或者设置为小于等于0，则表示POST提交的数据大小是不限制的，默认值是2Megabytes.把这个值设置为 1000M (maxPostSize="1000000000"),(` 注意：网上所传，设置为0会出现问题。 `)
protocol 设置处理请求的协议，默认是HTTP/1.1，即org.apache.coyote.http11.Http11Protocol，此外还 支持的协议有：org.apache.coyote.http11.Http11NioProtocol（通过NIO处理用户请求，可以提高系统性能）， org.apache.coyote.http11.HttpAprProtocol。
proxyName/proxyPort 如果Web服务器使用了代理服务器，配置此参数意味着在调用request.getServerName的时候将会获取代理服务器的名称，getServerPort()将会返回proxyPort。
redirectPort 如果Connector的配置是支持非SSL的请求，当一个SSL请求到来时，服务器会自动的将请求重定位到redirectPort。
URIEncoding URI字节转化成String的时候的编码方式，默认为ISO-8859-1，如果页面需要支持中文，一般可以将其设置为UTF-8或者GBK,GB2312。
useBodyEncodingForURI 如果设置为true,则会根据页面的编码决定URI的编码方式，默认是false。

Http/1.1 Connector提供的配置项：
acceptCount 等待队列的长度，默认值是100。
address 如果Tomcat所在的主机有多个IP，这个值声明了用于监听HTTP请求的IP地址。
bufferSize Connector创建的输入流的大小，默认值是2048 bytes，提高这个值可以提升性能，增加内存消耗。
compressableMimeType 使用HTTP压缩的MIME类型，使用逗号分割，默认值是 text/html,text/xml,text/plain。
compression 为了节省带宽，可以将这个值设置为on，从而启用HTTP/1.1 GZIP压缩。off关闭压缩，forces强制使用压缩，默认值是off。
connectionTimeout Connector接受一个连接后等待的时间(milliseconds)，默认值是60000。
executor 在Service节点下，Connector节点前可以配置一个Executor节点用于管理线程，这个属性的值是配置的Executor的名称，如果应用了此属性且executor存在，那么任何其他的关于thread的配置将会被忽略。
keepAliveTimeout 在Connector关闭连接前，Connector为另外一个请求Keep Alive所等待的微妙数，默认值和 connectionTimeout 一样。
maxHttpHeaderSize HTTP请求、响应头信息的最大大小，默认是8192bytes。
maxKeepAliveRequests HTTP/1.0 Keep Alive 和HTTP/1.1 Keep Alive / Pipeline的最大请求数目，如果设置为1，将会禁用掉Keep Alive和Pipeline，如果设置为小于0的数，Keep Alive的最大请求数将没有限制。默认为100。
maxThreads 用于处理用户请求的最大线程数，默认值是20。
noCompressionUserAgents: 设置不使用HTTP GZIP压缩的客户端，使用逗号分隔，在某些浏览器不支持压缩的时候可以使用此属性。
port Connector监听的端口。
restrictedUserAgents 设置不使用Keep Alive的客户端代理名称，使用逗号分割，默认值是空字符串。
server 覆盖HTTP响应的serve头信息，如果不设置的话，默认值是 Apache-Coyote/1.1。一般情况下不需要关注此属性。
socketBuffer Socket输出流缓冲区的大小，默认是9000bytes,如果设置为小于0的值，则表示不使用此缓冲区。
tcpNoDelay 默认值是true，设置为true可以提高系统性能。
threadPriority 请求处理线程的优先级，默认的优先级是NORMAL。