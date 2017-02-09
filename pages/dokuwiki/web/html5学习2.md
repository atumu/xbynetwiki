title: html5学习2 

#  HTML5学习之SSE(服务器推送事件) 
注意:目前不支持IE
**HTML5 服务器发送事件（server-sent event）允许网页获得来自服务器的更新。**
Server-Sent 事件 - 单向消息传递
Server-Sent 事件指的是网页自动获取来自服务器的更新。

以前也可能做到这一点，前提是网页不得不询问是否有可用的更新。通过服务器发送事件，更新能够自动到达。
例子：Facebook/Twitter 更新、估价更新、新的博文、赛事结果等。
浏览器支持
**所有主流浏览器均支持服务器发送事件，除了 Internet Explorer。**

##  接收 Server-Sent 事件通知 
` EventSource ` 对象用于接收服务器发送事件通知：
实例
```

var source=new EventSource("demo_sse.php");
source.onmessage=function(event)
  {
  document.getElementById("result").innerHTML+=event.data + "<br />";
  };

```
例子解释：
创建一个新的 EventSource 对象，然后规定发送更新的页面的 URL（本例中是 "demo_sse.php"）
每接收到一次更新，就会发生`  onmessage 事件 ` 当 onmessage 事件发生时，把已接收的数据推入 id 为 "result" 的元素中
##  检测 Server-Sent 事件支持 
在上面的 TIY 实例中，我们编写了一段额外的代码来检测服务器发送事件的浏览器支持情况：
```

if(typeof(EventSource)!=="undefined")
  {
  // Yes! Server-sent events support!
  // Some code.....
  }
else
  {
  // Sorry! No server-sent events support..
  }

```
##  服务器端代码实例 
为了让上面的例子可以运行，您还需要能够发送数据更新的服务器（比如 PHP 和 ASP）。
服务器端事件流的语法是非常简单的。` 把 "Content-Type" 报头设置为 "text/event-stream"。 `现在，您可以开始发送事件流了。
PHP 代码 (demo_sse.php)：
```

<?php
header('Content-Type: text/event-stream');
header('Cache-Control: no-cache');

$time = date('r');
echo "data: The server time is: {$time}\n\n";
flush();
?>

```
代码解释：
  * 把报头 "Content-Type" 设置为 "text/event-stream"
  * 规定不对页面进行缓存
  * 输出发送日期（输出数据始终以 "data: " 开头，然后\n\n结尾）
  * 向网页刷新flush输出数据
##  EventSource 对象 
在上面的例子中，我们使用 onmessage 事件来获取消息。不过还可以使用其他事件：
  * onopen	当通往服务器的连接被打开
  * onmessage	当接收到消息
  * onerror	当错误发生
##  服务器发送数据格式 
只存在一个事件时。
格式为data: bbbbbb\n\n
存在多个事件时，
格式为
event: 事件名\n
data: 数据\n\n

##  HTML5SSE结合JAVA示列 
###  当只有一个事件时 
```

<!DOCTYPE HTML>
<html>
<body>
    Time: <span id="foo"></span>
     
    <br><br>
    <button onclick="start()">Start</button>
 
    <script type="text/javascript">
    function start() {
 
        var eventSource = new EventSource("HelloServlet");
         
        eventSource.onmessage = function(event) {
         
            document.getElementById('foo').innerHTML = event.data;
         
        };
         
    }
    </script>
< /body>
< /html>

```
服务器返回数据格式必须是如下:
data: This is some data
 
data: a quick brown fox
 
data: jumps over a lazy dog.
...
```

public class TestServlet extends HttpServlet {
     
 
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
     
        //content type must be set to text/event-stream
        response.setContentType("text/event-stream");    //注意text/event-stream
 
        //encoding must be set to UTF-8
        response.setCharacterEncoding("UTF-8");//注意
 
        PrintWriter writer = response.getWriter();
 
        for(int i=0; i<10; i++) {
 
            writer.write("data: "+ System.currentTimeMillis() +"\n\n");//注意数据格式
 	     writer.flush();//注意flush
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        writer.close();
    }
}

```
###  当存在多个事件时 
```

var eventSource = new EventSource("HelloServlet");
eventSource.addEventListener('up_vote', function(event) {
        document.getElementById('up').innerHTML = event.data;
    }, false);
 
eventSource.addEventListener('down_vote', function(event) {
     
        document.getElementById('down').innerHTML = event.data;
         
    }, false);

```
服务器返回数据格式必须是如下:
event: up_vote
data: 10
 
event: down_vote
data: 5
 
event: up_vote
data: 12
 
event: down_vote
data: 9
...
```

public class HelloServlet extends HttpServlet {
 
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
 
        response.setContentType("text/event-stream");
        response.setCharacterEncoding("UTF-8");
 
        PrintWriter writer = response.getWriter();
        int upVote = 0;
        int downVote = 0;
        for (int i = 0; i < 20; i++) {
 
            upVote = upVote + (int) (Math.random() * 10);
            downVote = downVote + (int) (Math.random() * 10);
 
            writer.write("event:up_vote\n");
            writer.write("data: " + upVote + "\n\n");
 
            writer.write("event:down_vote\n");
            writer.write("data: " + downVote + "\n\n");
 
            writer.flush();
             
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        writer.close();
    }
 
}

```
参考：http://www.w3school.com.cn/html5/html_5_serversentevents.asp
http://viralpatel.net/blogs/html5-server-sent-events-java-servlets-example/