title: java_net_io 

#  Java网络操作与IO 
参考：http://ifeve.com/java-network/ http://blog.csdn.net/xiazdong/article/details/6713691
Java提供了非常易用的网络API，调用这些API我们可以很方便的通过建立TCP/IP或UDP套接字，在网络之间进行相互通信，其中TCP要比UDP更加常用
Java中Socket类和ServerSocket类

##  Socket 
当我们想要在Java中使用TCP/IP通过网络连接到服务器时，就需要创建java.net.Socket对象并连接到服务器。假如希望使用Java NIO，也可以创建Java NIO中的SocketChannel对象。
**socket发送数据**
```

Socket socket = new Socket("jenkov.com", 80);
OutputStream out = socket.getOutputStream(); 
out.write("some data".getBytes());
out.flush();
out.close(); 
socket.close();

```
但是想要通过网络将数据发送到服务器端，一定不要忘记调` 用flush() `方法
**Socket读取数据**
```

Socket socket = new Socket("jenkov.com", 80);
InputStream in = socket.getInputStream(); 
int data = in.read();
//... read more data... 
in.close();
socket.close();

```
从Socket的输入流中读取数据并不能读取文件那样，一直调用read()方法直到返回-1为止**，因为对Socket而言，只有当服务端关闭连接时，Socket的输入流才会返回-1，**而是事实上服务器并不会不停地关闭连接。假设我们想要通过一个连接发送多个请求，那么在这种情况下关闭连接就显得非常愚蠢。
` **因此，从Socket的输入流中读取数据时我们必须要知道需要读取的字节数，这可以通过让服务器在数据中告知发送了多少字节来实现，也可以采用在数据末尾设置特殊字符标记的方式连实现。** `
当使用完Socket后我们必须将Socket关闭，断开与服务器之间的连接。关闭Socket只需要调用Socket.close()方法即可，
##  ServerSocket 
用java.net.ServerSocket实现java服务通过TCP/IP监听客户端连接，你也可以用Java NIO 来代替java网络标准API，这时候需要用到 ServerSocketChannel。
```

ServerSocket serverSocket = new ServerSocket(9000);
 boolean isStopped = false;
while(!isStopped){   
Socket clientSocket = serverSocket.accept();    
  //do something with clientSocket
}

```
”接受”请求的线程通常都会把Socket的请求连接放入一个工作线程池中，然后再和客户端连接。更多关于多线程服务端设计的文档请参考 java多线程服务
socket.close();
serverSocket.close();
##  UDP 
略。。。请参考http://tutorials.jenkov.com/java-networking/udp-datagram-sockets.html
##  InetAddress 
InetAddress address = InetAddress.getByName("jenkov.com");
InetAddress address = InetAddress.getLocalHost();
##  URL + URLConnection 
在java.net包中包含两个有趣的类：URL类和URLConnection类。这两个类可以用来创建客户端到web服务器（HTTP服务器）的连接。
```

URL url = new URL("http://jenkov.com");
URLConnection urlConnection = url.openConnection();
Map<String, List<String>> map = urlConnection.getHeaderFields();  
        for (Map.Entry<String, List<String>> entry : map.entrySet()) {  
            String key = entry.getKey();  
            List<String> value = entry.getValue();  
            System.out.println(key + ":" + value);  
        }     
InputStream input = urlConnection.getInputStream();
int data = input.read();
while(data != -1){
System.out.print((char) data);
data = input.read();
}
input.close();

```
**HTTP GET和POST**
默认情况下URLConnection发送一个HTTP GET请求到web服务器。如果你想发送一个HTTP POST请求，要调用` URLConnection.setDoOutput(true) `方法，如下：
```

URL url = new URL("http://jenkov.com");
URLConnection urlConnection = url.openConnection();
urlConnection.setDoOutput(true);
OutputStream output = urlConnection.getOutputStream();

```
你可以使用这个OutputStream向相应的HTTP请求中写任何数据，但你要记得将其转换成URL编码` URLEncoder,URLDecoder `（译者注：具体名字是：application/x-www-form-urlencoded MIME 格式编码）。
当你写完数据的时候要记得关闭OutputStream。
##  JarURLConnection 
```

String urlString = "http://butterfly.jenkov.com/"
                 + "container/download/"
                 + "jenkov-butterfly-container-2.9.9-beta.jar";

URL jarUrl = new URL(urlString);
JarURLConnection connection = new JarURLConnection(jarUrl);

Manifest manifest = connection.getManifest();


JarFile jarFile = connection.getJarFile();

//do something with Jar file...

... 

```
