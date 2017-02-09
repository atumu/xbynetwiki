title: httpcore4.1学习 

#  HttpCore4.1学习 
中文:https://svn.apache.org/repos/asf/httpcomponents/httpcore/branches/4.1.x/httpcore-contrib/docs/translated-tutorial/httpcore-tutorial-simplified-chinese.pdf
http://hc.apache.org/httpcomponents-core-ga/tutorial/html/

HttpCore 是一套实现了 HTTP 协议最基础方面的组件
 HttpCore 目标
 实现最基本的 HTTP 传输方面
 良好性能和清晰度&表现力之间的平衡
 小的（预测）内存占用
 自我包含的类库（没有超越 JRE 的额外依赖）
什么是 HttpCore 不能做的
 HttpClient 的替代
 Servlet 容器或 Servlet API 竞争对手的替代
##  HTTP 报文 
结构
HTTP 报文由头部和可选的内容体构成。
HTTP 请求报文的头由请求行和头部字段的集合构成。
HTTP 响应报文的头部由状态行和头部字段的集合构成。所有 HTTP 报文必须包含协议版本。一些 HTTP 报文可选地可以包含内容体。
HttpCore 定义了 HTTP 报文对象模型，它紧跟定义，而且提供对 HTTP 报文元素进行序列化（格式化）和反序列化（解析）的支持。

###  HTTP 请求报文 

HTTP 请求是由客户端向服务器端发送的报文。报文的第一行包含应用于资源的方法，资源的标识符，和使用的协议版本。
```

HttpRequest request = new BasicHttpRequest("GET", "/",
HttpVersion.HTTP_1_1);
System.out.println(request.getRequestLine().getMethod());
System.out.println(request.getRequestLine().getUri());
System.out.println(request.getProtocolVersion());
System.out.println(request.getRequestLine().toString());
GET
/
HTTP/1.1
GET / HTTP/1.1

```
###   HTTP 响应报文 

HTTP 响应是由服务器在收到和解释请求报文之后发回客户端的报文。报文的第一行包含了协议的版本，之后是数字状态码和相关的文本段。
```

HttpResponse response = new
BasicHttpResponse(HttpVersion.HTTP_1_1,HttpStatus.SC_OK, "OK");
System.out.println(response.getProtocolVersion());
System.out.println(response.getStatusLine().getStatusCode());
System.out.println(response.getStatusLine().getReasonPhrase());
System.out.println(response.getStatusLine().toString());
HTTP/1.1
200
OK
HTTP/1.1 200 OK

```
###  HTTP 报文通用的属性和方法 

HTTP 报文可以包含一些描述报文属性的头部信息，比如内容长度，内容类型等。HttpCore提供方法来获取，添加，移除和枚举头部信息。
response.addHeader("Set-Cookie","c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie","c2=b; path=\"/\", c3=c; domain=\"localhost\"");
Header h1 = response.getFirstHeader("Set-Cookie");System.out.println(h1);
Header h2 = response.getLastHeader("Set-Cookie");System.out.println(h2);
Header[] hs = response.getHeaders("Set-Cookie");System.out.println(hs.length);
有一个获得所有给定类型头部信息的有效途径，是使用 **HeaderIterator** 接口。
**HeaderIterator** it = response.headerIterator("Set-Cookie");
while (it.hasNext()) {
System.out.println(it.next());
}
它也提供方便的方法来解析 HTTP 报文到独立头部元素。
**HeaderElementIterator** it = new BasicHeaderElementIterator(
response.headerIterator("Set-Cookie"));
while (it.hasNext()) {
HeaderElement elem = it.nextElement();
System.out.println(elem.getName() + " = " +
elem.getValue());
NameValuePair[] params = elem.getParameters();
for (int i = 0; i < params.length; i++) {
System.out.println(" " + params[i]);
}
}
##   HTTP 实体 
HTTP 报文可以携带和请求或响应相关的内容实体。HTTP 规范定义了两种包含实体的方法：POST 和 PUT。
响应通常期望是包含内容实体的。这个规则也有一些例外，比如对 HEAD 方法的响应，204 没有内容，304 没有修改，205 重置内容响应。
HttpCore 区分三种类型的实体，这是基于它们的内容是在哪里生成的：streamed 流式，self-contained 自我包含式，wrapping 包装式。
下面的实现是由 HttpCore 提供的：
 BasicHttpEntity
 ByteArrayEntity
 StringEntity
 InputStreamEntity
 FileEntity
 EntityTemplate
 HttpEntityWrapper
 **BufferedHttpEntity**
还有一个**UrlEncodedFormEntity**


更多略。。。。
