title: httpclient4.5学习之状态管理 

#  HttpClient4.5学习之状态管理 
HTTP cookie
Cookie 是 HTTP 代理和目标服务器可以交流保持会话的状态信息的令牌或小的数据包。
下面是客户端创建cookie对象的例子：
```

BasicClientCookie cookie = new BasicClientCookie("name", "value");  
// Set effective domain and path attributes  
cookie.setDomain(".mycompany.com");  
cookie.setPath("/");  
// Set attributes exactly as sent by the server  
cookie.setAttribute(ClientCookie.PATH_ATTR, "/");  
cookie.setAttribute(ClientCookie.DOMAIN_ATTR, ".mycompany.com");  

```
**CookieSpec接口**代表一个管理cookie的规范，cookie管理规范是强制的。
·解析Set-Cookie头部规则
·分析验证cookie规则
·用给定的主机名，端口号，和原路径格式化Cookie首部
HttpClient附带了几个CookieSpec的实现：Standard strict，Standard。。。。在新程序中，强烈建议使用Standard或Standard strict策略。
**Cookie策略**能够能够在HTTP客户端设置，如果需要的话，能够在HTTP请求中被重写（覆盖）。
```

RequestConfig globalConfig = RequestConfig.custom()  
    .setCookieSpec(CookieSpecs.DEFAULT)  
    .build();  
CloseableHttpClient httpclient = HttpClients.custom()  
    .setDefaultRequestConfig(globalConfig)  
    .build();  
RequestConfig localConfig = RequestConfig.copy(globalConfig)  
    .setCookieSpec(CookieSpecs.STANDARD_STRICT)  
.build();  
HttpGet httpGet = new HttpGet("/");  
httpGet.setConfig(localConfig);  


```
自定义cookie策略
为了实现自定义的cookie策略你必须创建一个CookieSpec的实现，创建一个CookieSpecProvider的实现，用它来创建和初始化自定义规范的实例、和HttpClient注册工厂。一旦自定义的规范被注册，它将会和标准cookie规范一样被使用。

Cookie 持久化
HttpClient能够使用任何实现了CookieStore接口的物理cookie。默认的CookieStore实现类被称为BasicCookieStore，它的内部是使用java.util.ArrayList的简单实现的。当容器进行垃圾回收时，储存在BasicClientCookie中的对象就会丢失。如果必要的话，使用者可以提供一个复杂的实现。

HTTP状态管理和执行上下文
在一个HTTP执行请求的过程中，HttpClient向执行上下文中添加了下面的状态管理对象
Lookup实例代表了真实的cookie详细记录，这个属性的值设置在本地上下文中，优先于默认的。
CookieSpec实例代表了真实的cookie规范
CookieOrigin实例代表了当前的原服务器的详情
CookieStore实例代表了当前的cookie store，这个属性的值设置在本地上下文中而优先于默认的。