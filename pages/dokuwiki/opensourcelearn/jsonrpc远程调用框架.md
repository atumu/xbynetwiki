title: jsonrpc远程调用框架 

#  JSON-RPC轻量级远程调用协议介绍  
##  概述： 
协议规范官网：http://json-rpc.org/
json-rpc是基于json的跨语言远程调用协议，比xml-rpc、webservice等基于文本的协议传输数据格小；相对hessian、java-rpc等二进制协议便于调试、实现、扩展，是非常优秀的一种远程调用协议。
目前主流语言都已有json-rpc的实现框架，
java语言中较好的json-rpc实现框架有jsonrpc4j、jpoxy、json-rpc。三者之中**jsonrpc4j**既可独立使用，又可与spring无缝集合，比较适合于基于spring的项目开发。 
##  JSON-RPC协议描述 
json-rpc协议非常简单，发起远程调用时向服务端传输数据格式如下：
   { "method": "sayHello", "params": ["Hello JSON-RPC"], "id": 1}
参数说明：
method： 调用的方法名
params： 方法传入的参数，若无参数则传入 []
id ： 调用标识符，用于标示一次远程调用过程
服务器其收到调用请求，处理方法调用，将方法效用结果效应给调用方；返回数据格式：
```

 {   
"result":"Hello JSON-RPC",         
"error": null,       
  "id": 1
 }   

```
参数说明:
result: 方法返回值，若无返回值，则返回null。若调用错误，返回null。
error ：调用时错误，无错误返回null。
id : 调用标识符，与调用方传入的标识符一致。

以上就是json-rpc协议规范，非常简单，小巧，便于各种语言实现。

#  Json-RPC的使用示例 
采用**jsonrpc4j**进行讲解
##  服务器端Java调用示例 
jsonrpc4j服务器端java示例：
```

public class HelloWorldServlet extends HttpServlet {
    private static final long serialVersionUID = 3638336826344504848L;
    private JsonRpcServer rpcService = null;
    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        rpcService = new JsonRpcServer(new HelloWorldService(), HelloWorldService.class);
    }
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        rpcService.handle(req, resp);    
    }
}

```
##  jsonrpc4j的Java客户端调用示例： 
```

JsonRpcHttpClient client = new JsonRpcHttpClient(
 new URL("http://127.0.0.1:8080/index.json"));
 Map<String,String> headers = new HashMap<String,String>();
headers.put("name", "剑白");
client.setHeaders(headers);
 String properties = client.invoke("getSystemProperties", null, String.class);
 System.out.println(properties);

```
##  基于jsonrpcjs的JavaScript客户端调用示例： 
```

var rpc = new jsonrpc.JsonRpc('http://127.0.0.1:8080/index.json');
rpc.call('getSystemProperties', function(result){
alert(result);
});

```
##  直接GET请求进行调用 

无需任何客户端，只需手工拼接参数进行远程调用，请求URL如下：
http://127.0.0.1:8080/index.json?method=getSystemProperties&id=3325235235235&params=JTViJTVk
参数说明:
method : 方法名
params ：调用参数，json的数组格式[], 将参数需先进行url编码，再进行base64编码
id : 调用标识符，任意值。
#  JSON-RPC总结 
json-rpc是一种非常轻量级的跨语言远程调用协议，实现及使用简单。仅需几十行代码，即可实现一个远程调用的客户端，方便语言扩展客户端的实现。服务器端有php、java、python、ruby、.net等语言实现，是非常不错的及轻量级的远程调用协议。

#  JSON-RPC2.0规范介绍 
JSON-RPC2.0规范由JSON-RPC工作组（json-rpc@googlegroups.com）维护，发布于2010-03-26(基于2009-05-24的版本)， 最近的更新于2013-01-04。
整体来说，2.0版本的JSON-RPC规范改动的很小，大的改动大概有3点：

  * 参数可以用数组或命名参数
  * 批量请求的细节明确化了
  * 错误处理的机制标准化了

与1.0版本的兼容性

  * 建议2.0规范的实现兼容1.0协议，但是不强制要求，如果不能兼容，建议给出友好提示。
  * 请求和响应报文加了个参数表示协议的版本号：jsonrpc，它必须是“2.0”。
  * method的修改：以rpc开头方法名表示rpc内部的方法和扩展，其他地方必须不能使用。
  * 请求参数可以使用数组[参数1，参数2，，，]，也可以使用命名参数{key:value}。
  * 请求参数为空时params可省略。
  * id一般不应该为null，是数值的话不应该是小数。
  * 请求里没有id时，被当做通知。（1.0时这里是id为null。）
  * 请求参数必须精确匹配，包括大小写。
  * 应答必须包含result或error，但是两个成员都必须不能同时包含。

##  批量请求 
```

请求
 [
        {"jsonrpc": "2.0", "method": "sum", "params": [1,2,4], "id": "1"},
        {"jsonrpc": "2.0", "method": "notify_hello", "params": [7]},
        {"jsonrpc": "2.0", "method": "subtract", "params": [42,23], "id": "2"},
        {"jsonrpc": "2.0", "method": "foo.get", "params": {"name": "myself"}, "id": "5"},
        {"jsonrpc": "2.0", "method": "get_data", "id": "9"} 
    ]
服务端响应
[
        {"jsonrpc": "2.0", "result": 19, "id": "2"},
        {"jsonrpc": "2.0", "error": {"code": -32601, "message": "Method not found"}, "id": "5"},
        {"jsonrpc": "2.0", "result": ["hello", 5], "id": "9"}
    ]

```
规范定义了所有的请求应该并发执行，并且返回不保证顺序，客户端自己使用id去匹配对应的请求和响应。而且对于请求的**处理中只要有一个出错，则返回一个统一的错误信息（就是不区分哪一条失败，全部都算失败了）**。 这个设计看起来是针对事务考虑的，但是在一般的使用场景里应该会比较麻烦。
##  错误对象 
**改进的error机制是，error变成了一个明确定义的对象**。包括三个属性：
  * code：数值，见下一节错误代码。
  * message：字符串格式的错误信息。
  * data：可选的，服务器端定义的一个数值或是对象，来附加额外的信息。
比原来的粗放型错误机制好多了。
###  错误代码 
![](/data/dokuwiki/opensourcelearn/pasted/20150513-094026.png)

#  jsonrpc4j介绍与使用 
概述：
jsonrpc4j：JSON-RPC for Java 
这个项目能够帮助开发人员利用Java编程语言轻松实现JSON-RPC远程调用。
**jsonrpc4j使用` Jackson `类库实现Java对象与JSON对象之间的相互转换**。jsonrpc4j包含一个JSON-RPC服务器，支持Stream与HTTP（GET与POST），同时还提供一个支持Stream的JSON-RPC客户端。此外还提供一个HTTP客户端、Spring Service Provider和Spring Service Consumer。
目前已实现JSON-RPC2.0规范。
` jsonrpc4j支持Android开发 `
项目地址：https://github.com/briandilley/jsonrpc4j
##  安装 

依赖：
服务端依赖：（必须）
jackson-core-2.0.2.jar
jackson-annotations-2.0.2.jar
jackson-databind-2.0.2.jar
jsonrpc4j-1.1.jar
portlet-api-2.0.jar
客户端依赖（必须）：
jackson-core-2.0.2.jar
jackson-annotations-2.0.2.jar
jackson-databind-2.0.2.jar
jsonrpc4j-1.1.jar
可选（本人暂且未知是否必须）:
commons-codec.jar
commons-logging.jar
httpcore-nio-4.2.1.jar
httpcore-4.2.1.jar


maven方式安装：
```

   <dependency>
        <groupId>com.github.briandilley.jsonrpc4j</groupId>
        <artifactId>jsonrpc4j</artifactId>
        <version>1.1</version>
    </dependency>

```
核心类:
JsonRpcHttpClient继承JsonRpcClient  
JsonRpcServer 
##  使用 
###  客户端： 

```

JsonRpcHttpClient client = new JsonRpcHttpClient(
    new URL("http://example.com/UserService.json"));
//注意这里的参数User指的是createUser方法的返回值
User user = client.invoke("createUser", new Object[] { "bob", "the builder" }, User.class);

```
你也可以使用动态代理方式：
```

JsonRpcHttpClient client = new JsonRpcHttpClient(
    new URL("http://example.com/UserService.json"));

UserService userService = ProxyUtil.createClientProxy(
    getClass().getClassLoader(),
    UserService.class,
    client);

User user = userService.createUser("bob", "the builder");

```
###  服务端： 

```

class UserServiceServlet
    extends HttpServlet {

    private UserService userService;
    private JsonRpcServer jsonRpcServer;

    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        jsonRpcServer.handle(req, resp);
    }

    public void init(ServletConfig config) {
        this.userService = ...;
        this.jsonRpcServer = new JsonRpcServer(this.userService, UserService.class);
    }

}

```
###  组合多个rpc服务到一个地方： 

```

UserverService userService = ...;
ContentService contentService = ...;
BlackJackService blackJackService = ...;

Object compositeService = ProxyUtil.createCompositeServiceProxy(
    this.getClass().getClassLoader(),
    new Object[] { userService, contentService, blackJackService},
    new Class<?>[] { UserService.class, ContentService.class, BlackJackService.class},
    true);

// now compositeService can be used as any of the above service, ie:
User user = ((UserverService)compositService).createUser(...);
Content content =  ((ContentService)compositService).getContent(...);
Hand hand = ((BlackJackService)compositService).dealHand(...);

JsonRpcServer jsonRpcServer = new JsonRpcServer(compositeService);

```
##  与Spring集成 
首先创建调用服务接口：
```

package com.mycompany;
public interface UserService {
    User createUser(String userName, String firstName, String password);
    User createUser(String userName, String password);
    User findUserByUserName(String userName);
    int getUserCount();
}

```
然后实现这个服务接口：
```

package com.mycompany;
public class UserServiceImpl
    implements UserService {
    public User createUser(String userName, String firstName, String password) {
        User user = new User();
        user.setUserName(userName);
        user.setFirstName(firstName);
        user.setPassword(password);
        database.saveUser(user)
        return user;
    }

    public User createUser(String userName, String password) {
        return this.createUser(userName, null, password);
    }

    public User findUserByUserName(String userName) {
        return database.findUserByUserName(userName);
    }

    public int getUserCount() {
        return database.getUserCount();
    }
}

```
##  配置Service到Spring 

```

<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <bean id="userService" class="com.mycompany.UserServiceImpl">
    </bean>

    <bean name="/UserService.json" class="com.googlecode.jsonrpc4j.spring.JsonServiceExporter">
        <property name="service" ref="userService"/>
        <property name="serviceInterface" value="com.mycompany.UserService"/>
    </bean>
    <bean  class="com.googlecode.jsonrpc4j.spring.JsonProxyFactoryBean">
        <property name="serviceUrl" value="http://example.com/UserService.json"/>
        <property name="serviceInterface" value="com.mycompany.UserService"/>
    </bean>

```
现在你的服务就被配置到了可通过url访问 /UserService.json。Type conversion of JSON->Java and Java->JSON will happen for you automatically

##  错误消息注解： 

@JsonRpcErrors
```

package com.mycompany;
public interface UserService {
    @JsonRpcErrors({
        @JsonRpcError(exception=UserExistsException.class,
            code=-5678, message="User already exists", data="The Data"),
        @JsonRpcError(exception=Throwable.class,code=-187)
    })
    User createUser(@JsonRpcParamName("theUserName") String userName, @JsonRpcParamName("thePassword") String password);
}

```
##  配置Auto Discovery With Annotations 
```

@JsonRpcService("/path/to/MyService")
interface MyService {
... service methods ...
}

<bean class="com.googlecode.jsonrpc4j.spring.AutoJsonRpcServiceExporter"/>
<bean class="com.mycompany.MyServiceImpl" />
<bean class="com.googlecode.jsonrpc4j.spring.AutoJsonRpcClientProxyCreator">
    <property name="baseUrl" value="http://hostname/api/" />
    <property name="scanPackage" value="com.mycompany.services" />
  </bean>
  

```
使用注解定制方法签名
```

@JsonRpcService("/jsonrpc")
public interface LibraryService {
    @JsonRpcMethod("VideoLibrary.GetTVShows")
    List<TVShow> fetchTVShows(@JsonRpcParam("properties") final List<String> properties);
}
{"jsonrpc":"2.0", "method": "VideoLibrary.GetTVShows", "params": { "properties": ["title"] }, "id":1}

```
#  请求参数与方法签名的对应示例 
```

json request:

{"jsonrpc":"2.0", "id":"10", "method":"aMethod", "params":["Test"]}

java methods:

public void aMethod(String param1);
public void aMethod(String param1, String param2);
public void aMethod(String param1, String param2, int param3);

```
```

json request:

{"jsonrpc":"2.0", "id":"10", "method":"addFriend", "params":[{"username":"example", "firstName":"John"}]}

java methods:

public void addFriend(UserObject userObject);
public void addFriend(UserObjectEx userObjectEx);

```
参考：
http://gubaojian.blog.163.com/blog/static/1661799082012101439591/
http://blog.csdn.net/KimmKing/article/details/43412381
http://blog.csdn.net/kimmking/article/details/43410253
http://blog.csdn.net/mhmyqn/article/details/39718097