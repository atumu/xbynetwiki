title: python学习笔记34 

#  Python学习笔记之TCP/UDP网络编程 
` 网络通信是两台计算机上的两个进程之间的通信。 `比如，浏览器进程和新浪服务器上的某个Web服务进程在通信，而QQ进程是和腾讯的某个服务器上的某个进程在通信。
用Python进行网络编程，就是在Python程序本身这个进程内，连接别的服务器进程的通信端口进行通信。
**IP协议**负责把数据从一台计算机通过网络发送到另一台计算机。数据被分割成一小块一小块，然后通过IP包发送出去。由于互联网链路复杂，两台计算机之间经常有多条线路，因此，路由器就负责决定如何把一个IP包转发出去。IP包的特点是按块发送，途径多个路由，但不保证能到达，也不保证顺序到达。
**TCP协议**则是建立在IP协议之上的。TCP协议负责在两台计算机之间建立可靠连接，保证数据包按顺序到达。TCP协议会通过握手建立连接，然后，对每个IP包编号，确保对方按顺序收到，如果包丢掉了，就自动重发。
许多常用的更高级的协议都是建立在TCP协议基础上的，比如用于浏览器的HTTP协议、发送邮件的SMTP协议等。
一个IP包除了包含要传输的数据外，还包含源IP地址和目标IP地址，源端口和目标端口。
**端口有什么作用？**在两台计算机通信时，只发IP地址是不够的，因为同一台计算机上跑着多个网络程序。一个IP包来了之后，到底是交给浏览器还是QQ，就需要端口号来区分。每个网络程序都向操作系统申请唯一的端口号，这样，两个进程在两台计算机之间建立网络连接就需要各自的IP地址和各自的端口号。
一个进程也可能同时与多个计算机建立链接，因此它会申请很多端口。
了解了TCP/IP协议的基本概念，IP地址和端口的概念，我们就可以开始进行网络编程了。

##  TCP编程 
**Socket是网络编程的一个抽象概念。通常我们用一个Socket表示“打开了一个网络链接”**，而打开一个Socket需要知道**目标计算机的IP地址和端口号**，再指定协议类型即可。

###  客户端 
大多数连接都是可靠的TCP连接。**创建TCP连接时，主动发起连接的叫客户端，被动响应连接的叫服务器。**
举个例子，当我们在浏览器中访问新浪时，我们自己的计算机就是客户端，浏览器会主动向新浪的服务器发起连接。如果一切顺利，新浪的服务器接受了我们的连接，一个TCP连接就建立起来的，后面的通信就是发送网页内容了。
所以，我们要创建一个基于TCP连接的Socket，可以这样做：
```

# 导入socket库:
import socket
# 创建一个socket:
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 建立连接:
s.connect(('www.sina.com.cn', 80))

```
**创建Socket时，AF_INET指定使用IPv4协议，如果要用更先进的IPv6，就指定为AF_INET6。SOCK_STREAM指定使用面向流的TCP协议**，这样，一个Socket对象就创建成功，但是还没有建立连接。
客户端要主动发起TCP连接，必须知道服务器的IP地址和端口号。
我们连接新浪服务器的代码如下：
s.connect(('www.sina.com.cn', 80))
**注意参数是一个tuple，包含地址和端口号。**
建立TCP连接后，我们就可以向新浪服务器发送请求，要求返回首页的内容：
```

# 发送数据:
s.send(b'GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')

```
**TCP连接创建的是双向通道，双方都可以同时给对方发数据。**但是谁先发谁后发，怎么协调，要根据具体的协议来决定。例如，HTTP协议规定客户端必须先发请求给服务器，服务器收到后才发数据给客户端。
发送的文本格式必须符合HTTP标准，如果格式没问题，接下来就可以接收新浪服务器返回的数据了：
```

# 接收数据:
buffer = []
while True:
    # 每次最多接收1k字节:
    d = s.recv(1024)
    if d:
        buffer.append(d)
    else:
        break
data = b''.join(buffer)

```
接收数据时，调用recv(max)方法，一次最多接收指定的字节数，因此，在一个while循环中反复接收，直到recv()返回空数据，表示接收完毕，退出循环。
当我们接收完数据后，调用close()方法关闭Socket，这样，一次完整的网络通信就结束了：
```

# 关闭连接:
s.close()

```
接收到的数据包括HTTP头和网页本身，我们只需要把HTTP头和网页分离一下，把HTTP头打印出来，网页内容保存到文件：
```

header, html = data.split(b'\r\n\r\n', 1)
print(header.decode('utf-8'))
# 把接收的数据写入文件:
with open('sina.html', 'wb') as f:
    f.write(html)

```
现在，只需要在浏览器中打开这个sina.html文件，就可以看到新浪的首页了。
###  服务器 
和客户端编程相比，服务器编程就要复杂一些。
**服务器进程首先要绑定一个端口并监听来自其他客户端的连接。**如果某个客户端连接过来了，服务器就与该客户端建立Socket连接，随后的通信就靠这个Socket连接了。
所以，服务器会打开固定端口（比如80）监听，**每来一个客户端连接，就创建该Socket连接。由于服务器会有大量来自客户端的连接，所以，服务器要能够区分一个Socket连接是和哪个客户端绑定的。**
**一个Socket依赖4项：服务器地址、服务器端口、客户端地址、客户端端口来唯一确定一个Socket。**
但是服务器还需要同时响应多个客户端的请求，所以，**每个连接都需要一个新的进程或者新的线程来处理**，否则，服务器一次就只能服务一个客户端了。
我们来编写一个简单的服务器程序，它接收客户端连接，把客户端发过来的字符串加上Hello再发回去。
首先，创建一个基于IPv4和TCP协议的Socket：
```

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

```
然后，我们要绑定监听的地址和端口。服务器可能有多块网卡，可以绑定到某一块网卡的IP地址上，也可以用0.0.0.0绑定到所有的网络地址，还可以用127.0.0.1绑定到本机地址。127.0.0.1是一个特殊的IP地址，表示本机地址，如果绑定到这个地址，客户端必须同时在本机运行才能连接，也就是说，外部的计算机无法连接进来。
端口号需要预先指定。因为我们写的这个服务不是标准服务，所以用9999这个端口号。请注意，小于1024的端口号必须要有管理员权限才能绑定：
```

# 监听端口:
s.bind(('127.0.0.1', 9999))

```
紧接着，调用listen()方法开始监听端口，传入的参数指定等待连接的最大数量：
```

s.listen(5)
print('Waiting for connection...')

```
接下来，服务器程序通过一个永久循环来接受来自客户端的连接，accept()会等待并返回一个客户端的连接:
```

while True:
    # 接受一个新连接:
    sock, addr = s.accept()
    # 创建新线程来处理TCP连接:
    t = threading.Thread(target=tcplink, args=(sock, addr))
    t.start()

```
**每个连接都必须创建新线程（或进程）来处理**，否则，单线程在处理连接的过程中，无法接受其他客户端的连接：
```

def tcplink(sock, addr):
    print('Accept new connection from %s:%s...' % addr)
    sock.send(b'Welcome!')
    while True:
        data = sock.recv(1024)
        time.sleep(1)
        if not data or data.decode('utf-8') == 'exit':
            break
        sock.send(('Hello, %s!' % data.decode('utf-8')).encode('utf-8'))
    sock.close()
    print('Connection from %s:%s closed.' % addr)

```
连接建立后，服务器首先发一条欢迎消息，然后等待客户端数据，并加上Hello再发送给客户端。如果客户端发送了exit字符串，就直接关闭连接。
要测试这个服务器程序，我们还需要编写一个客户端程序：
```

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 建立连接:
s.connect(('127.0.0.1', 9999))
# 接收欢迎消息:
print(s.recv(1024).decode('utf-8'))
for data in [b'Michael', b'Tracy', b'Sarah']:
    # 发送数据:
    s.send(data)
    print(s.recv(1024).decode('utf-8'))
s.send(b'exit')
s.close()

```
我们需要打开两个命令行窗口，一个运行服务器程序，另一个运行客户端程序，就可以看到效果了：

小结
用TCP协议进行Socket编程在Python中十分简单，对于客户端，要主动连接服务器的IP和指定端口，对于服务器，要首先监听指定端口，然后，对每一个新的连接，创建一个线程或进程来处理。通常，服务器程序会无限运行下去。
同一个端口，被一个Socket绑定了以后，就不能被别的Socket绑定了。

##  UDP 
**TCP是建立可靠连接，并且通信双方都可以以流的形式发送数据。相对TCP，UDP则是面向无连接的协议。**
**使用UDP协议时，不需要建立连接，只需要知道对方的IP地址和端口号，就可以直接发数据包。但是，能不能到达就不知道了。**
**虽然用UDP传输数据不可靠，但它的优点是和TCP比，速度快，**对于不要求可靠到达的数据，就可以使用UDP协议。
我们来看看如何通过UDP协议传输数据。和TCP类似，使用UDP的通信双方也分为客户端和服务器。服务器首先需要绑定端口：
```

s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
# 绑定端口:
s.bind(('127.0.0.1', 9999))

```
创建Socket时，SOCK_DGRAM指定了这个Socket的类型是UDP。绑定端口和TCP一样，但是不需要调用listen()方法，而是直接接收来自任何客户端的数据：
```

print('Bind UDP on 9999...')
while True:
    # 接收数据:
    data, addr = s.recvfrom(1024)
    print('Received from %s:%s.' % addr)
    s.sendto(b'Hello, %s!' % data, addr)

```
` recvfrom()方法 `返回数据和客户端的地址与端口，这样，服务器收到数据后，直接调用` sendto() `就可以把数据用UDP发给客户端。
注意这里省掉了多线程，因为这个例子很简单。
客户端使用UDP时，首先仍然创建基于UDP的Socket，然后，不需要调用connect()，直接通过sendto()给服务器发数据：
```

s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
for data in [b'Michael', b'Tracy', b'Sarah']:
    # 发送数据:
    s.sendto(data, ('127.0.0.1', 9999))
    # 接收数据:
    print(s.recv(1024).decode('utf-8'))
s.close()

```
从服务器接收数据仍然调用recv()方法。
仍然用两个命令行分别启动服务器和客户端测试.