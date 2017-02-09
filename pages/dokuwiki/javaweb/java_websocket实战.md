title: java_websocket实战 

#  java websocket实战 
可以参考：http://www.hackbase.com/article-1068-1.html
##  简单聊天室 
参考：http://blog.csdn.net/hzw2312/article/details/41252159
java jar包：tomcat-coyote.jar、tomcat-juli.jar、websocket-api.jar(maven  provider环境即可。tomcat自带)

```

/** 
 * WebSocket 消息推送服务类 
 */  
@ServerEndpoint(value = "/websocket/chat")  
public class ChatAnnotation {  
  
    private static final Log log = LogFactory.getLog(ChatAnnotation.class);  
  
    private static final String GUEST_PREFIX = "Guest";  
    private static final AtomicInteger connectionIds = new AtomicInteger(0);  
    private static final Map<String,Object> connections = new ConcurrentHashMap<String,Object>();  
  
    private final String nickname;  
    private Session session;  
  
    public ChatAnnotation() {  
        nickname = GUEST_PREFIX + connectionIds.getAndIncrement();  
    }  
  
  
    @OnOpen  
    public void start(Session session) {  
        this.session = session;  
        connections.put(nickname, this);   
        String message = String.format("* %s %s", nickname, "has joined.");  
        broadcast(message);  
    }  
  
  
    @OnClose  
    public void end() {  
        connections.remove(this);  
        String message = String.format("* %s %s",  
                nickname, "has disconnected.");  
        broadcast(message);  
    }  
  
  
    /** 
     * 消息发送触发方法 
     * @param message 
     */  
    @OnMessage  
    public void incoming(String message) {  
        // Never trust the client  
        String filteredMessage = String.format("%s: %s",  
                nickname, HTMLFilter.filter(message.toString()));  
        broadcast(filteredMessage);  
    }  
  
    @OnError  
    public void onError(Throwable t) throws Throwable {  
        log.error("Chat Error: " + t.toString(), t);  
    }  
  
    /** 
     * 消息发送方法 
     * @param msg 
     */  
    private static void broadcast(String msg) {  
        if(msg.indexOf("Guest0")!=-1){  
            sendUser(msg);  
        } else{  
            sendAll(msg);  
        }  
    }   
      
    /** 
     * 向所有用户发送 
     * @param msg 
     */  
    public static void sendAll(String msg){  
        for (String key : connections.keySet()) {  
            ChatAnnotation client = null ;  
            try {  
                client = (ChatAnnotation) connections.get(key);  
                synchronized (client) {  
                    client.session.getBasicRemote().sendText(msg);  
                }  
            } catch (IOException e) {   
                log.debug("Chat Error: Failed to send message to client", e);  
                connections.remove(client);  
                try {  
                    client.session.close();  
                } catch (IOException e1) {  
                    // Ignore  
                }  
                String message = String.format("* %s %s",  
                        client.nickname, "has been disconnected.");  
                broadcast(message);  
            }  
        }  
    }  
      
    /** 
     * 向指定用户发送消息  
     * @param msg 
     */  
    public static void sendUser(String msg){  
        ChatAnnotation c = (ChatAnnotation)connections.get("Guest0");  
        try {  
            c.session.getBasicRemote().sendText(msg);  
        } catch (IOException e) {  
            log.debug("Chat Error: Failed to send message to client", e);  
            connections.remove(c);  
            try {  
                c.session.close();  
            } catch (IOException e1) {  
                // Ignore  
            }  
            String message = String.format("* %s %s",  
                    c.nickname, "has been disconnected.");  
            broadcast(message);    
        }   
    }  
}  

```
```

/** 
 * HTML 工具类  
 */  
public final class HTMLFilter {  
    public static String filter(String message) {  
        if (message == null)  
            return (null);  
        char content[] = new char[message.length()];  
        message.getChars(0, message.length(), content, 0);  
        StringBuilder result = new StringBuilder(content.length + 50);  
        for (int i = 0; i < content.length; i++) {  
            switch (content[i]) {  
            case '<':  
                result.append("<");  
                break;  
            case '>':  
                result.append(">");  
                break;  
            case '&':  
                result.append("&");  
                break;  
            case '"':  
                result.append(""");  
                break;  
            default:  
                result.append(content[i]);  
            }  
        }  
        return (result.toString());  
    }  
}  

```
```

<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>  
<%  
String path = request.getContextPath();  
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  
%>  
  
<?xml version="1.0" encoding="UTF-8"?>  
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">  
<head>  
    <title>测试</title>  
    <style type="text/css">  
        input#chat {  
            width: 410px  
        }  
  
        #console-container {  
            width: 400px;  
        }  
  
        #console {  
            border: 1px solid #CCCCCC;  
            border-right-color: #999999;  
            border-bottom-color: #999999;  
            height: 170px;  
            overflow-y: scroll;  
            padding: 5px;  
            width: 100%;  
        }  
  
        #console p {  
            padding: 0;  
            margin: 0;  
        }  
 </style>  
    <script type="text/javascript">  
  
        var Chat = {};  
  
        Chat.socket = null;  
  
        Chat.connect = (function(host) {  
            if ('WebSocket' in window) {  
                Chat.socket = new WebSocket(host);  
            } else if ('MozWebSocket' in window) {  
                Chat.socket = new MozWebSocket(host);  
            } else {  
                Console.log('Error: WebSocket is not supported by this browser.');  
                return;  
            }  
  
            Chat.socket.onopen = function () {  
                Console.log('Info: WebSocket connection opened.');  
                document.getElementById('chat').onkeydown = function(event) {  
                    if (event.keyCode == 13) {  
                        Chat.sendMessage();  
                    }  
                };  
            };  
  
            Chat.socket.onclose = function () {  
                document.getElementById('chat').onkeydown = null;  
                Console.log('Info: WebSocket closed.');  
            };  
  
            Chat.socket.onmessage = function (message) {  
                Console.log(message.data);  
            };  
        });  
  
        Chat.initialize = function() {  
            if (window.location.protocol == 'http:') {  
                Chat.connect('ws://' + window.location.host + '/socket2/websocket/chat');  
            } else {  
                Chat.connect('wss://' + window.location.host + '/socket2/websocket/chat');  
            }  
        };  
  
        Chat.sendMessage = (function() {  
            var message = document.getElementById('chat').value;  
            if (message != '') {  
                Chat.socket.send(message);  
                document.getElementById('chat').value = '';  
            }  
        });  
  
        var Console = {};  
  
        Console.log = (function(message) {  
            var console = document.getElementById('console');  
            var p = document.createElement('p');  
            p.style.wordWrap = 'break-word';  
            p.innerHTML = message;  
            console.appendChild(p);  
            while (console.childNodes.length > 25) {   
                console.removeChild(console.firstChild);  
            }  
            console.scrollTop = console.scrollHeight;  
        });  
  
        Chat.initialize();  
  
  
        document.addEventListener("DOMContentLoaded", function() {  
            // Remove elements with "noscript" class - <noscript> is not allowed in XHTML  
            var noscripts = document.getElementsByClassName("noscript");  
            for (var i = 0; i < noscripts.length; i++) {  
                noscripts[i].parentNode.removeChild(noscripts[i]);  
            }  
        }, false);  
  
   </script>  
</head>  
<body>  
<div class="noscript"><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websockets rely on Javascript being enabled. Please enable  
    Javascript and reload this page!</h2></div>  
<div>  
    <p>  
        <input type="text" placeholder="请输入内容" id="chat" />  
    </p>  
    <div id="console-container">  
        <div id="console"/>  
    </div>  
</div>  
</body>  
</html>  

```
可指定发送给某个用户，也可全部发送，详情见ChatAnnotation类的broadcast方法。
程序发布时记得删除tomcat-coyote.jar、tomcat-juli.jar、websocket-api.jar这三个jar包在启动Tomcat。
程序截图，Guest0用户发送信息的信息，在后台进行了判断只发送给自己：
![](/data/dokuwiki/javaweb/pasted/20151124-100416.png)![](/data/dokuwiki/javaweb/pasted/20151124-100421.png)![](/data/dokuwiki/javaweb/pasted/20151124-100428.png)