title: java_websocket通信 

#  java websocket通信 
作为下一代的 Web 标准，HTML5 拥有许多引人注目的新特性，如 Canvas、本地存储、多媒体编程接口、WebSocket 等等。这其中**有“Web 的 TCP ”之称的 WebSocket** 格外吸引开发人员的注意。
WebSocket 的出现使得浏览器提供对 Socket 的支持成为可能，**从而在浏览器和服务器之间提供了一个基于 TCP 连接的双向通道。**
Web 开发人员可以非常方便地**使用 WebSocket 构建实时 web 应用**，开发人员的手中从此又多了一柄神兵利器。

##  实时 Web 应用的窘境 
Web 应用的信息交互过程通常是客户端通过浏览器发出一个请求，服务器端接收和审核完请求后进行处理并返回结果给客户端，然后客户端浏览器将信息呈现出来，这种机制对于信息变化不是特别频繁的应用尚能相安无事，**但是对于那些实时要求比较高的应用来说**，比如说在线游戏、在线证券、设备监控、新闻在线播报、RSS 订阅推送等等，当客户端浏览器准备呈现这些信息的时候，这些信息在服务器端可能已经过时了。
所以**保持客户端和服务器端的信息同步是实时 Web 应用的关键要素**，对 Web 开发人员来说也是一个难题。
**在 WebSocket 规范出来之前，开发人员想实现这些实时的 Web 应用，不得不采用一些折衷的方案，其中最常用的就是轮询 (Polling) 和 Comet 技术**，而 Comet 技术实际上是轮询技术的改进，又可细分为两种实现方式，一种是长轮询机制，一种称为流技术。下面我们简单介绍一下这几种技术：
**轮询：**
这是最早的一种实现实时 Web 应用的方案。**客户端以一定的时间间隔向服务端发出请求，以频繁请求的方式来保持客户端和服务器端的同步。**这种同步方案的最大问题是，当客户端以固定频率向服务器发起请求的时候，服务器端的数据可能并没有更新，**这样会带来很多无谓的网络传输，所以这是一种非常低效的实时方案。**
**长轮询：**
长轮询是对定时轮询的改进和提高，**目地是为了降低无效的网络传输。当服务器端没有数据更新的时候，连接会保持一段时间周期直到数据或状态改变或者时间过期，**通过这种机制来减少无效的客户端和服务器间的交互。当然，如果服务端的数据变更非常频繁的话，这种机制和定时轮询比较起来没有本质上的性能的提高。
**流：**
流技术方案通常就是**在客户端的页面使用一个隐藏的窗口向服务端发出一个长连接的请求**。服务器端接到这个请求后作出回应并不断更新连接状态以保证客户端和服务器端的连接不过期。通过这种机制可以将服务器端的信息源源不断地推向客户端。这种机制在用户体验上有一点问题，需要针对不同的浏览器设计不同的方案来改进用户体验，同时**这种机制在并发比较大的情况下，对服务器端的资源是一个极大的考验。**
综合这几种方案，您会发现这些目前我们所使用的所谓的实时技术**并不是真正的实时技术，它们只是在用 Ajax 方式来模拟实时的效果**，在每次客户端和服务器端交互的时候都是一次 HTTP 的请求和应答的过程，而每一次的 HTTP 请求和应答都带有完整的 HTTP 头信息，这就增加了每次传输的数据量，而且这些方案中客户端和服务器端的编程实现都比较复杂，在实际的应用中，为了模拟比较真实的实时效果，开发人员往往需要构造两个 HTTP 连接来模拟客户端和服务器之间的双向通讯，一个连接用来处理客户端到服务器端的数据传输，一个连接用来处理服务器端到客户端的数据传输，这不可避免地增加了编程实现的复杂度，也增加了服务器端的负载，制约了应用系统的扩展性。

##  WebSocket规范 
**HTML5 WebSocket** 设计出来的目的就是要取代轮询和 Comet 技术，使客户端浏览器具备像 C/S 架构下桌面系统的**实时通讯能力**。 **浏览器通过 JavaScript 向服务器发出建立 WebSocket 连接的请求，连接建立以后，客户端和服务器端就可以通过`  TCP ` 连接直接交换数据。**因为 WebSocket 连接本质上就是一个 TCP 连接，所以在数据传输的稳定性和数据传输量的大小方面，和轮询以及 Comet 技术比较，具有很大的**性能优势**。

**WebSocket 协议本质上是一个基于 TCP 的协议。**为了建立一个 WebSocket 连接，客户端浏览器首先要向服务器发起一个 HTTP 请求，这个请求和通常的 HTTP 请求不同，包含了一些附加头信息，
其中附加头信息”` Upgrade: WebSocket `”表明这是一个**申请协议升级**的 HTTP 请求，服务器端解析这些附加的头信息然后产生应答信息返回给客户端，客户端和服务器端的 WebSocket 连接就建立起来了，双方就可以通过这个连接通道自由的传递信息，并且这个连接会持续存在直到客户端或者服务器端的某一方主动的关闭连接。
下面我们来详细介绍一下 WebSocket 规范
一个典型的 WebSocket 发起请求和得到响应的例子看起来如下：
清单 1. WebSocket 握手协议
客户端到服务端： 
```

GET /demo HTTP/1.1 
Host: example.com 
Connection: Upgrade 
Sec-WebSocket-Key2: 12998 5 Y3 1 .P00 
Upgrade: WebSocket 
Sec-WebSocket-Key1: 4@1 46546xW%0l 1 5 
Origin: http://example.com 
[8-byte security key]

``` 

服务端到客户端：
```

HTTP/1.1 101 WebSocket Protocol Handshake 
Upgrade: WebSocket 
Connection: Upgrade 
WebSocket-Origin: http://example.com 
WebSocket-Location: ws://example.com/demo 
[16-byte hash response]

```
这些请求和通常的 HTTP 请求很相似，**但是其中有些内容是和 WebSocket 协议密切相关的**。我们需要简单介绍一下这些请求和应答信息，”` Upgrade:WebSocket `”表示这是一个特殊的 HTTP 请求，请求的目的就是要将客户端和服务器端的通讯协议**从 HTTP 协议升级到 WebSocket 协议**。从客户端到服务器端请求的信息里包含有”` Sec-WebSocket-Key1 `”、“Sec-WebSocket-Key2”和”[8-byte securitykey]”这样的头信息。这是客户端浏览器需要向服务器端提供的**握手信息**，服务器端解析这些头信息，并在握手的过程中依据这些信息生成一个 16 位的安全密钥并返回给客户端，以表明服务器端获取了客户端的请求，同意创建 WebSocket 连接。**一旦连接建立，客户端和服务器端就可以抛开HTTP协议而通过这个websocket通道双向传输数据了。**
在实际的开发过程中，为了使用 WebSocket 接口构建 Web 应用，我们首先需要构建一个实现了 WebSocket 规范的服务器，服务器端的实现不受平台和开发语言的限制，只需要遵从 WebSocket 规范即可，目前已经出现了一些比较成熟的 WebSocket 服务器端实现，比如：
  * Tomcat7 — 一个 Java 实现的 WebSocket Server
  * mod_pywebsocket — 一个 Python 实现的 WebSocket Server
  * Netty —一个 Java 实现的网络框架其中包括了对 WebSocket 的支持


##  WebSocket远远不止这些 
WebSocker协议的用途几乎是无限的。
除了以上所说的可以应用于B/S下,也可以应用于C/S下，还可应用于` **应用程序集群节点之间的通信** `
##  WebSocket JavaScript 接口 
上一节介绍了 WebSocket 规范，其中主要介绍了 **WebSocket 的握手协议**。握手协议通常是我们在构建 WebSocket 服务器端的实现和提供浏览器的 WebSocket 支持时需要考虑的问题，
而针对 Web 开发人员的 WebSocket JavaScript 客户端接口是非常简单的，以下是 WebSocket JavaScript 接口的定义：
 ```

[Constructor(in DOMString url, in optional DOMString protocol)] 
 interface WebSocket { 
   readonly attribute DOMString URL; 
        // ready state 
   const unsigned short CONNECTING = 0; 
   const unsigned short OPEN = 1; 
   const unsigned short CLOSING=2;
   const unsigned short CLOSED = 3; 
   readonly attribute unsigned short readyState; //CONNECTING，OPEN，CLOSING,CLOSED
   readonly attribute unsigned long bufferedAmount; //表示之前的send调用还有多少数据需要发送到服务器。
   //networking 
   attribute Function onopen; 
   attribute Function onmessage; 
   attribute Function onclose; 
   boolean send(in DOMString data);  //接受一个字符串或Blob或ArrayBuffer或ArrayBufferView作为唯一的参数。
   void close();  //可以传可选的一个数字表示关闭代码，默认1000，一个理由reason参数。
 }; 
 WebSocket implements EventTarget;

```
  * **其中 URL 属性代表 WebSocket 服务器的网络地址，协议通常是”ws”或"wss"**,
  * send 方法就是发送数据到服务器端，
  * close 方法就是关闭连接。
  * 还有一些很重要的事件：onopen，onmessage，onerror 以及 onclose。我们借用 Nettuts 网站上的一张图来形象的展示一下 WebSocket 接口：

![](/data/dokuwiki/javaweb/pasted/20151124-001551.png)
下面是一段简单的 JavaScript 代码展示了怎样建立 WebSocket 连接和获取数据：
```

 var  wsServer = 'ws://localhost:8888/Demo'; 
 var  websocket = new WebSocket(wsServer); 
 websocket.onopen = function (evt) { onOpen(evt) }; 
 websocket.onclose = function (evt) { onClose(evt) }; 
 websocket.onmessage = function (evt) { onMessage(evt) }; 
 websocket.onerror = function (evt) { onError(evt) }; 
 function onOpen(evt) { 
 console.log("Connected to WebSocket server."); 
 } 
 function onClose(evt) { 
 console.log("Disconnected"); 
 } 
 function onMessage(evt) { 
 console.log('Retrieved data from server: ' + evt.data); //event中只有一个data对象
 } 
 function onError(evt) { 
 console.log('Error occured: ' + evt.data); //event中除了data对象外还有wasClean布尔，code,reason属性等。
 }

```

**浏览器支持**
下面是主流浏览器对 HTML5 WebSocket 的支持情况：
浏览器	支持情况
Chrome	Supported in version 4+
Firefox	Supported in version 4+
Internet Explorer	Supported in version 10+
Opera	Supported in version 10+
Safari	Supported in version 5+

WebSocket 将会成为未来开发实时 Web 应用的生力军应该是毫无悬念的了，作为 Web 开发人员，关注 HTML5，关注 WebSocket 也应该提上日程了，否则我们在新一轮的软件革新的浪潮中只能做壁上观了。

##  Java EE HTML5 WebSocket 示例 
本教程需要以下环境：
Ubuntu 12.04
JDK 1.7.0.21
tomcat 7.0.43以上版本
注: Java EE 7中才引入了WebSocket。

Maven:
```

<dependency>
            <groupId>javax.websocket</groupId>
            <artifactId>javax.websocket-api</artifactId>
            <version>1.1</version>
            <scope>provided</scope>
        </dependency>

```

客户端API与服务端API
服务端API依赖于客户端API。
客户端API基于ContainerProvider,WebSocketContainer,RemoteEndpoint,Session接口构建。
WebSocket提供了对所有WebSocket客户端特性的访问，而ContainerProvider提供一个静态的getWebSocketContainer方法，用于获取底层的WebSocket客户端实现。
WebSocket的Endpoint有三个方法onOpen,onClose,onError，onMessage而@ClientEndpoint可以有标注了@OnOpen,@OnClose,@OnError的方法，一个或多个@OnMessage方法。
  * @OnOpen参数：Session，EndpointConfig
  * @OnClose参数：Session,CloseReason
  * @OnError参数：Session,Throwable
  * @OnMessage参数：Session,一个字符串/Reader/ByteBuffer/InputStream/PongMessage

服务端API依赖于客户端API。
  * ServerContainer继承了WebSockerContainer
  * ServerEndpointConfig
  * @ServerEndpoint
在Servlet环境中，可以调用` ServletContext.getAttribute("javax.websocket.server.ServerContainer") `可以获得ServerContainer实例.
不过，几乎在所有的用例中你都不需要获取ServerContainer,相反，只需要使用@ServerEndpoint标注服务器终端类即可。WebSocket实现可以扫描类的注解，并自动选择和注册服务器终端。
` @ServerEndpoint("/game/{type}") `  必须以/开头。必须指定value。可以包含路径参数

**其实服务器和客户端的区别只是在握手的时候，在握手完成，连接建立之后，服务器和客户端都将变成对等的端点。此时就没有服务器和客户端的概念了，因为大家都是端点。**
###  WebSocket服务器端 
让我们定义一个 Java EE websocket服务器端：
WebSocketTest.java
```

import java.io.IOException;
 
import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;
 
@ServerEndpoint("/websocket")
public class WebSocketTest {
 
  @OnMessage
  public void onMessage(String message, Session session)
    throws IOException, InterruptedException {
   
    // Print the client message for testing purposes
    System.out.println("Received: " + message);
   
    // Send the first message to the client
    session.getBasicRemote().sendText("This is the first server message");
   
    // Send 3 messages to the client every 5 seconds
    int sentMessages = 0;
    while(sentMessages < 3){
      Thread.sleep(5000);
      session.getBasicRemote().
        sendText("This is an intermediate server message. Count: "
          + sentMessages);
      sentMessages++;
    }
   
    // Send a final message to the client
    session.getBasicRemote().sendText("This is the last server message");
  }
   
  @OnOpen
  public void onOpen() {
    System.out.println("Client connected");
  }
 
  @OnClose
  public void onClose() {
    System.out.println("Connection closed");
  }
}

```
你可能已经注意到我们从 **javax.websocket**包中引入了一些类。
` @ServerEndpoint ` 注解是一个类层次的注解，它的功能主要是**将目前的类定义成一个websocket服务器端**。注解的值将被用于监听用户连接的终端访问URL地址。
onOpen 和 onClose 方法分别被` @OnOpen和@OnClose ` 所注解。这两个注解的作用不言自明：他们定义了当一个新用户连接和断开的时候所调用的方法。
onMessage 方法被` @OnMessage `所注解。这个注解定义了当服务器接收到客户端发送的消息时所调用的方法。注意：这个方法可能包含一个` javax.websocket.Session `可选参数（在我们的例子里就是session参数）。如果有这个参数，容器将会把当前发送消息客户端的连接Session注入进去。
本例中我们仅仅是将客户端消息内容打印出来，然后首先我们将发送一条开始消息，之后间隔5秒向客户端发送1条测试消息，共发送3次，最后向客户端发送最后一条结束消息。
###  客户端 
现在我们要来写websocket测试应用的客户端：
page.html
```

<!DOCTYPE html>
<html>
<head>
<title>Testing websockets</title>
</head>
<body>
  <div>
    <input type="submit" value="Start" onclick="start()" />
  </div>
  <div id="messages"></div>
  <script type="text/javascript">
    var webSocket =
      new WebSocket(getRootUri()+'/byteslounge/websocket');
    function getRootUri() {
                return "ws://" + (document.location.hostname == "" ? "localhost" : document.location.hostname) + ":" +
                        (document.location.port == "" ? "8080" : document.location.port);
            }
    webSocket.onerror = function(event) {
      onError(event)
    };
 
    webSocket.onopen = function(event) {
      onOpen(event)
    };
 
    webSocket.onmessage = function(event) {
      onMessage(event)
    };
 
    function onMessage(event) {
      document.getElementById('messages').innerHTML
        += '<br />' + event.data;
    }
 
    function onOpen(event) {
      document.getElementById('messages').innerHTML
        = 'Connection established';
    }
 
    function onError(event) {
      alert(event.data);
    }
 
    function start() {
      webSocket.send('hello');
      return false;
    }
  </script>
</body>
</html>

```
这是一个简单的页面，包含有JavaScript代码，这些代码**创建了一个websocket连接到websocket服务器端。**
  * onOpen 我们创建一个连接到服务器的连接时将会调用此方法。
  * onError 当客户端-服务器通信发生错误时将会调用此方法。
  * onMessage 当从服务器接收到一个消息时将会调用此方法。在我们的例子中，我们只是将从服务器获得的消息添加到DOM。
我们连接到websocket 服务器端，使用构造函数 new WebSocket() 而且传之以端点URL：
```

ws://localhost:8080/byteslounge/websocket

```

###  WebSockets 握手 
客户端和服务器端TCP连接建立在HTTP协议握手发生之后。通过HTTP流量调试，很容易观察到握手。客户端一创建一个 WebSocket实例，就会出现如下请求和服务器端响应： 
注意: 我们只录入了WebSocket握手所用到的HTTP头。
```

请求:
GET /byteslounge/websocket HTTP/1.1 
Connection: Upgrade 
Upgrade: websocket 
Sec-WebSocket-Key: wVlUJ/tu9g6EBZEh51iDvQ==
响应:
HTTP/1.1 101 Web Socket Protocol Handshake 
Upgrade: websocket 
Sec-WebSocket-Accept: 2TNh+0h5gTX019lci6mnvS66PSY=

```
注意：进行连接需要将通过Upgrade and Upgrade将协议升级到支持websocket HTTP头的Websocket协议。服务器响应表明请求被接受，协议将转换到WebSocket协议（HTTP状态码101）:
HTTP/1.1 101 Web Socket Protocol Handshake

##  传输更复杂的对象与编码解码器 
参考：http://ju.outofmemory.cn/entry/117332
**编解码器**
前面的例子中WebSocket通信的消息类型默认为String。接下来的例子演示如何**使用` Encoder和Decoder `传输更复杂的数据。**
**Websocket使用Decoder将文本消息转换成Java对象，然后传给@OnMessage方法处理; 而当对象写入到session中时，Websocket将使用Encoder将Java对象转换成文本，再发送给客户端。**
更常用的， 我们**使用XML 或者 JSON 来传送数据，所以将会会将Java对象与XML/JSON数据相互转换.**
下图描绘了客户端和服务器使用encoder/decoder标准通信过程。
![](/data/dokuwiki/javaweb/pasted/20151124-101513.png)
声明Encoder/Decoder也是相当的简单: 你只需在@ServerEndpoint注解中增加encoder/decoder设置：
```

import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;
 
@ServerEndpoint(value = "/hello",
        decoders = {
            MessageDecoder.class,},
        encoders = {
            MessageEncoder.class
        })
public class HelloWorldEndpoint {
 
    @OnMessage
    public Person hello(Person person, Session session) {
        if (person.getName().equals("john")) {
            person.setName("Mr. John");
        }
        try {
            session.getBasicRemote().sendObject(person);
            System.out.println("sent ");
        } catch (Exception ex) {
            Logger.getLogger(HelloWorldEndpoint.class.getName()).log(Level.SEVERE, null, ex);
        }
        return person;
 
    }
 
    @OnOpen
    public void myOnOpen(Session session) {
    }
 
}

```
正像你看到的，** OnMessage方法使用Java Object person作为参数**， 我们为名字增加个尊称再返回给客户端。通过` session.getBasicRemote().sendObject(Object obj) `返回数据.
**容器负责使用你指定的Decoder将接收到的XML消息转为Java对象：**
```

import javax.websocket.Decoder;
import javax.websocket.EndpointConfig;
import javax.xml.bind.*;
 
  
public class MessageDecoder implements Decoder.Text<Person> {
 
    @Override
    public Person decode(String s) {
        System.out.println("Incoming XML " + s);
        Person person = null;
        JAXBContext jaxbContext;
        try {
            jaxbContext = JAXBContext.newInstance(Person.class);
 
            Unmarshaller unmarshaller = jaxbContext.createUnmarshaller();
 
            StringReader reader = new StringReader(s);
            person = (Person) unmarshaller.unmarshal(reader);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return person;
    }
 
    @Override
    public boolean willDecode(String s) {
          
        return (s != null);
    }
 
    @Override
    public void init(EndpointConfig endpointConfig) {
        // do nothing.
    }
 
    @Override
    public void destroy() {
        // do nothing.
    }
}

```
这里我们使用JAXB做转换。我们只要实现一个泛型接口` Decoder.Text 或者 Decoder.Binary `， **根据你传输的数据是文本还是二进制选择一个.**
所以数据由Decoder解码， 传给Endpoint (这里的 HelloWorldEndpoint)，** 在返回给client之前， 它还会被下面的Encoder转换成XML:**
```

import javax.websocket.EncodeException;
import javax.websocket.Encoder;
import javax.websocket.EndpointConfig;
import javax.xml.bind.JAXBContext;
import javax.xml.bind.Marshaller;
  
public class MessageEncoder implements Encoder.Text<Person> {
 
    @Override
    public String encode(Person object) throws EncodeException {
 
        JAXBContext jaxbContext = null;
        StringWriter st = null;
        try {
            jaxbContext = JAXBContext.newInstance(Person.class);
 
            Marshaller marshaller = jaxbContext.createMarshaller();
            st = new StringWriter();
            marshaller.marshal(object, st);
            System.out.println("OutGoing XML " + st.toString());
 
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return st.toString();
    }
 
    @Override
    public void init(EndpointConfig endpointConfig) {
        // do nothing.
    }
 
    @Override
    public void destroy() {
        // do nothing.
    }
}

```
```

<html>
    <head>
        <meta http-equiv="content-type" content="text/html; charset=ISO-8859-1">
    </head>
 
    <body>
        <meta charset="utf-8">
        <title>HelloWorld Web sockets</title>
        <script language="javascript" type="text/javascript">
            var wsUri = getRootUri() + "/websocket-hello/hello";
 
            function getRootUri() {
                return "ws://" + (document.location.hostname == "" ? "localhost" : document.location.hostname) + ":" +
                        (document.location.port == "" ? "8080" : document.location.port);
            }
 
            function init() {
                output = document.getElementById("output");
            }
 
            function send_message() {
 
                websocket = new WebSocket(wsUri);
                websocket.onopen = function(evt) {
                    onOpen(evt)
                };
                websocket.onmessage = function(evt) {
                    onMessage(evt)
                };
                websocket.onerror = function(evt) {
                    onError(evt)
                };
 
            }
 
            function onOpen(evt) {
                writeToScreen("Connected to Endpoint!");
                doSend(textID.value);
 
            }
 
            function onMessage(evt) {
                writeToScreen("Message Received: " + evt.data);
            }
 
            function onError(evt) {
                writeToScreen('<span style="color: red;">ERROR:</span> ' + evt.data);
            }
 
            function doSend(message) {
                writeToScreen("Message Sent: " + message);
                websocket.send(message);
            }
 
            function writeToScreen(message) {
                alert(message);
                
            }
 
            window.addEventListener("load", init, false);
 
        </script>
 
        <h1 style="text-align: center;">Hello World WebSocket Client</h2>
 
        <br>
 
        <div style="text-align: center;">
            <form action="">
                <input onclick="send_message()" value="Send" type="button">
                <textarea id="textID" rows="4" cols="50" name="message" >
                </textarea>
            </form>
        </div>
        <div id="output"></div>
</body>
</html>

```
在文本框中输入下面的XML进行测试。

```

<person>
  <name>john</name>
  <surname>smith</surname>
</person>

```
WebSocket是HTML5开始提供的一种在单个 TCP 连接上进行全双工通讯的协议，可以用来创建快速的更大规模的健壮的高性能实时的web应用程序。使用WebSocket连接， web应用程序可以执行实时的交互， 而不是以前的poll方式。
在WebSocket API中，浏览器和服务器**只需要做一个握手的动作**，然后，浏览器和服务器之间就**形成了一条快速通道。两者之间就直接可以数据互相传送**。
###  实战实例 
```

@ServerEndpoint("/ticTacToe/{gameId}/{username}")
public class TicTacToeServer
{
    private static Map<Long, Game> games = new Hashtable<>();
    private static ObjectMapper mapper = new ObjectMapper();

    @OnOpen
    public void onOpen(Session session, @PathParam("gameId") long gameId,
                       @PathParam("username") String username)
    {
        try
        {
            TicTacToeGame ticTacToeGame = TicTacToeGame.getActiveGame(gameId);
            if(ticTacToeGame != null)
            {
                session.close(new CloseReason(
                        CloseReason.CloseCodes.UNEXPECTED_CONDITION,
                        "This game has already started."
                ));
            }

            List<String> actions = session.getRequestParameterMap().get("action");
            if(actions != null && actions.size() == 1)
            {
                String action = actions.get(0);
                if("start".equalsIgnoreCase(action))
                {
                    Game game = new Game();
                    game.gameId = gameId;
                    game.player1 = session;
                    TicTacToeServer.games.put(gameId, game);
                }
                else if("join".equalsIgnoreCase(action))
                {
                    Game game = TicTacToeServer.games.get(gameId);
                    game.player2 = session;
                    game.ticTacToeGame = TicTacToeGame.startGame(gameId, username);
                    this.sendJsonMessage(game.player1, game,
                            new GameStartedMessage(game.ticTacToeGame));
                    this.sendJsonMessage(game.player2, game,
                            new GameStartedMessage(game.ticTacToeGame));
                }
            }
        }
        catch(IOException e)
        {
            e.printStackTrace();
            try
            {
                session.close(new CloseReason(
                        CloseReason.CloseCodes.UNEXPECTED_CONDITION, e.toString()
                ));
            }
            catch(IOException ignore) { }
        }
    }

    @OnMessage
    public void onMessage(Session session, String message,
                          @PathParam("gameId") long gameId)
    {
        Game game = TicTacToeServer.games.get(gameId);
        boolean isPlayer1 = session == game.player1;

        try
        {
            Move move = TicTacToeServer.mapper.readValue(message, Move.class);
            game.ticTacToeGame.move(
                    isPlayer1 ? TicTacToeGame.Player.PLAYER1 :
                            TicTacToeGame.Player.PLAYER2,
                    move.getRow(),
                    move.getColumn()
            );
            this.sendJsonMessage((isPlayer1 ? game.player2 : game.player1), game,
                    new OpponentMadeMoveMessage(move));
            if(game.ticTacToeGame.isOver())
            {
                if(game.ticTacToeGame.isDraw())
                {
                    this.sendJsonMessage(game.player1, game,
                            new GameIsDrawMessage());
                    this.sendJsonMessage(game.player2, game,
                            new GameIsDrawMessage());
                }
                else
                {
                    boolean wasPlayer1 = game.ticTacToeGame.getWinner() ==
                            TicTacToeGame.Player.PLAYER1;
                    this.sendJsonMessage(game.player1, game,
                            new GameOverMessage(wasPlayer1));
                    this.sendJsonMessage(game.player2, game,
                            new GameOverMessage(!wasPlayer1));
                }
                game.player1.close();
                game.player2.close();
            }
        }
        catch(IOException e)
        {
            this.handleException(e, game);
        }
    }

    @OnClose
    public void onClose(Session session, @PathParam("gameId") long gameId)
    {
        Game game = TicTacToeServer.games.get(gameId);
        if(game == null)
            return;
        boolean isPlayer1 = session == game.player1;
        if(game.ticTacToeGame == null)
        {
            TicTacToeGame.removeQueuedGame(game.gameId);
        }
        else if(!game.ticTacToeGame.isOver())
        {
            game.ticTacToeGame.forfeit(isPlayer1 ? TicTacToeGame.Player.PLAYER1 :
                    TicTacToeGame.Player.PLAYER2);
            Session opponent = (isPlayer1 ? game.player2 : game.player1);
            this.sendJsonMessage(opponent, game, new GameForfeitedMessage());
            try
            {
                opponent.close();
            }
            catch(IOException e)
            {
                e.printStackTrace();
            }
        }
    }

    private void sendJsonMessage(Session session, Game game, Message message)
    {
        try
        {
            session.getBasicRemote()
                   .sendText(TicTacToeServer.mapper.writeValueAsString(message));
        }
        catch(IOException e)
        {
            this.handleException(e, game);
        }
    }

    private void handleException(Throwable t, Game game)
    {
        t.printStackTrace();
        String message = t.toString();
        try
        {
            game.player1.close(new CloseReason(
                    CloseReason.CloseCodes.UNEXPECTED_CONDITION, message
            ));
        }
        catch(IOException ignore) { }
        try
        {
            game.player2.close(new CloseReason(
                    CloseReason.CloseCodes.UNEXPECTED_CONDITION, message
            ));
        }
        catch(IOException ignore) { }
    }


```
```

 <script type="text/javascript" language="javascript">
            var move;
            $(document).ready(function() {
                var modalError = $("#modalError");
                var modalErrorBody = $("#modalErrorBody");
                var modalWaiting = $("#modalWaiting");
                var modalWaitingBody = $("#modalWaitingBody");
                var modalGameOver = $("#modalGameOver");
                var modalGameOverBody = $("#modalGameOverBody");
                var opponent = $("#opponent");
                var status = $("#status");
                var opponentUsername;
                var username = '<c:out value="${username}" />';
                var myTurn = false;

                $('.game-cell').addClass('span1');

                if(!("WebSocket" in window))
                {
                    modalErrorBody.text('WebSockets are not supported in this ' +
                            'browser. Try Internet Explorer 10 or the latest ' +
                            'versions of Mozilla Firefox or Google Chrome.');
                    modalError.modal('show');
                    return;
                }

                modalWaitingBody.text('Connecting to the server.');
                modalWaiting.modal({ keyboard: false, show: true });

                var server;
                try {
                    server = new WebSocket('ws://' + window.location.host +
                            '<c:url value="/ticTacToe/${gameId}/${username}">
                                <c:param name="action" value="${action}" />
                            </c:url>');
                } catch(error) {
                    modalWaiting.modal('hide');
                    modalErrorBody.text(error);
                    modalError.modal('show');
                    return;
                }

                server.onopen = function(event) {
                    modalWaitingBody
                            .text('Waiting on your opponent to join the game.');
                    modalWaiting.modal({ keyboard: false, show: true });
                };

                window.onbeforeunload = function() {
                    server.close();
                };

                server.onclose = function(event) {
                    if(!event.wasClean || event.code != 1000) {
                        toggleTurn(false, 'Game over due to error!');
                        modalWaiting.modal('hide');
                        modalErrorBody.text('Code ' + event.code + ': ' +
                                event.reason);
                        modalError.modal('show');
                    }
                };


                server.onmessage = function(event) {
                    var message = JSON.parse(event.data);
                    if(message.action == 'gameStarted') {
                        if(message.game.player1 == username)
                            opponentUsername = message.game.player2;
                        else
                            opponentUsername = message.game.player1;
                        opponent.text(opponentUsername);
                        toggleTurn(message.game.nextMoveBy == username);
                        modalWaiting.modal('hide');
                    } else if(message.action == 'opponentMadeMove') {
                        $('#r' + message.move.row + 'c' + message.move.column)
                                .unbind('click')
                                .removeClass('game-cell-selectable')
                                .addClass('game-cell-opponent game-cell-taken');
                        toggleTurn(true);
                    } else if(message.action == 'gameOver') {
                        toggleTurn(false, 'Game Over!');
                        if(message.winner) {
                            modalGameOverBody.text('Congratulations, you won!');
                        } else {
                            modalGameOverBody.text('User "' + opponentUsername +
                                    '" won the game.');
                        }
                        modalGameOver.modal('show');
                    } else if(message.action == 'gameIsDraw') {
                        toggleTurn(false, 'The game is a draw. ' +
                                'There is no winner.');
                        modalGameOverBody.text('The game ended in a draw. ' +
                                'Nobody wins!');
                        modalGameOver.modal('show');
                    } else if(message.action == 'gameForfeited') {
                        toggleTurn(false, 'Your opponent forfeited!');
                        modalGameOverBody.text('User "' + opponentUsername +
                                '" forfeited the game. You win!');
                        modalGameOver.modal('show');
                    }
                };

                var toggleTurn = function(isMyTurn, message) {
                    myTurn = isMyTurn;
                    if(myTurn) {
                        status.text(message || 'It\'s your move!');
                        $('.game-cell:not(.game-cell-taken)')
                                .addClass('game-cell-selectable');
                    } else {
                        status.text(message ||'Waiting on your opponent to move.');
                        $('.game-cell-selectable')
                                .removeClass('game-cell-selectable');
                    }
                };

                move = function(row, column) {
                    if(!myTurn) {
                        modalErrorBody.text('It is not your turn yet!');
                        modalError.modal('show');
                        return;
                    }
                    if(server != null) {
                        server.send(JSON.stringify({ row: row, column: column }));
                        $('#r' + row + 'c' + column).unbind('click')
                                .removeClass('game-cell-selectable')
                                .addClass('game-cell-player game-cell-taken');
                        toggleTurn(false);
                    } else {
                        modalErrorBody.text('Not connected to came server.');
                        modalError.modal('show');
                    }
                };
            });
        </script>

```
参考：http://www.ibm.com/developerworks/cn/web/1112_huangxa_websocket/
http://www.oschina.net/translate/java-ee-html5-websocket-example
http://ju.outofmemory.cn/entry/117332