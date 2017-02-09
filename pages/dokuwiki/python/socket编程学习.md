title: socket编程学习 

#  Socket编程原理与学习 
参考：http://python.jobbole.com/86733/
https://en.wikipedia.org/wiki/Berkeley_sockets#Socket_API_functions
demo:https://github.com/jiacai2050/socket.py

Socket 在英文中的含义为“（连接两个物品的）凹槽”.在计算机科学中，socket 通常是指一个连接的两个端点，这里的连接可以是同一机器上的，像 **unix domain socket** ，也可以是不同机器上的，像 **network socket** 。
**Socket API 是由操作系统提供的一个编程接口**，让应用程序可以控制使用 socket 技术。Unix 哲学中有一条 **一切皆为文件** ，所以 socket 和 file 的 API 使用很类似：可以进行 **read 、 write 、 open 、 close** 等操作。

现在的**网络系统是分层**的，理论上有 OSI模型 ，工业界有 TCP/IP协议簇 。其对比如下：
以太网(Ethernet)
![](/data/dokuwiki/python/pasted/20161107-092124.png)
每层上都有其相应的协议，**socket API 不属于TCP/IP协议簇，只是操作系统提供的一个用于网络编程的接口**，工作在应用层与传输层之间：
![](/data/dokuwiki/python/pasted/20161107-092209.png)

我们平常浏览网站所使用的http协议，收发邮件用的smtp与imap，都是基于 socket API 构建的。
**一个 socket，包含两个必要组成部分：**
  * 地址，由 ip 与 端口组成，像 192.168.0.1:80 。
  * 协议，socket 所是用的传输协议，目前有三种： TCP 、 UDP 、 raw IP 。
**地址与协议可以确定一个socket**；一台机器上，只允许存在一个同样的socket。TCP 端口 53 的 socket 与 UDP 端口 53 的 socket 是两个不同的 socket。

**根据 socket 传输数据方式的不同**（使用协议不同），可以分为以下三种：
  * Stream sockets ，也称为“面向连接”的 socket，使用 TCP 协议。实际通信前需要进行连接，传输的数据没有特定的结构，所以高层协议需要自己去界定数据的分隔符，但其优势是数据是可靠的。
  * Datagram sockets ，也称为“无连接”的 socket，使用 UDP 协议。实际通信前不需要连接，一个优势时 UDP 的数据包自身是可分割的（self-delimiting），也就是说每个数据包就标示了数据的开始与结束，其劣势是数据不可靠。
  * Raw sockets ，通常用在路由器或其他网络设备中，这种 socket **不经过TCP/IP协议簇中的传输层（transport layer）**，直接由网络层（Internet layer）通向应用层（Application layer），所以这时的数据包就不会包含 tcp 或 udp 头信息。
![](/data/dokuwiki/python/pasted/20161107-094651.png)

##  Python socket API 
Python 里面用** (ip, port) 的元组**来表示 socket 的地址属性，用** AF_* 来表示协议类型**。
数据通信有两组动词可供选择： **send/recv 或 read/write** 。 read/write 方式也是 Java 采用的方式，这里不会对这种方式进行过多的解释，但是需要注意的是：
read/write 操作的具有 buffer 的“文件”，所以在进行读写后需要调用 **flush** 方法去真正发送或读取数据，否则数据会一直停留在缓冲区内。
###  TCP socket 
TCP socket 由于在通向前需要建立连接。具体如下：
![](/data/dokuwiki/python/pasted/20161107-095611.png)
每个API 的具体含义这里不在赘述，可以查看 [手册](https://en.wikipedia.org/wiki/Berkeley_sockets) ，这里给出 Python 语言的实现的 echo server。
```

# echo_server.py
# coding=utf8
import socket
 
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 设置 SO_REUSEADDR 后,可以立即使用 TIME_WAIT 状态的 socket
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sock.bind(('', 5500))
sock.listen(5)
 
defhandler(client_sock, addr):
    print('new client from %s:%s' % addr)
    msg = client_sock.recv(1024)
    client_sock.send(msg)
    client_sock.close()
    print('client[%s:%s] socket closed' % addr)
 
if __name__ == '__main__':
    while 1:
        client_sock, addr = sock.accept()
        handler(client_sock, addr)

```
```

# echo_client.py
# coding=utf8
importsocket
 
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('', 5500))
sock.send('hello socket world')
printsock.recv(1024)

```
上面简单的echo server 代码中有一点需要注意的是：
###  socket()函数三个参数 
**domain**,指定协议簇. For example:
  * AF_INET for network protocol IPv4 (IPv4-only) or
  * AF_INET6 for IPv6 (and in some cases, backwards compatible with IPv4).
  * AF_UNIX for local socket (using a file).
**type**, one of:
  * SOCK_STREAM (reliable stream-oriented service or Stream Sockets)，即TCP
  * SOCK_DGRAM (datagram service or Datagram Sockets)，即UDP
  * SOCK_SEQPACKET (reliable sequenced packet service), or
  * SOCK_RAW (raw protocols atop the network layer).
**protocol**，可选，指定实际的传输层协议（transport protocol）. The most common are IPPROTO_TCP, IPPROTO_SCTP, IPPROTO_UDP, IPPROTO_DCCP. These protocols are specified in file netinet/in.h. The value 0 may be used to select a default protocol from the selected domain and type.

server 端的 socket 设置了 SO_REUSEADDR 为1，目的是可以立即使用处于 TIME_WAIT 状态的socket，那么 TIME_WAIT 又是什么意思呢？后面在讲解 tcp 状态变更图时再做详细介绍。
###  setsockopt()参数说明 
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740476(v=vs.85).aspx
http://www.cppblog.com/killsound/archive/2009/01/16/72138.html
**socket.setsockopt(level, option, value)**
level:(级别)： 指定选项代码的类型。
  * SOL_SOCKET: 基本套接口
  * IPPROTO_IP: IPv4套接口
  * IPPROTO_IPV6: IPv6套接口
  * IPPROTO_TCP: TCP套接口
option： 选项名称
value:选项值
下面分别说明不同level情况下常用的选项名称：
level = SOL_SOCKET
Value	Type	Description
SO_BROADCAST BOOL 允许套接口传送广播信息。
SO_KEEPALIVE	BOOL	
SO_RCVBUF	int	为接收确定缓冲区大小。
SO_REUSEADDR	BOOL	允许套接口和一个已在使用中的地址捆绑。For more information, see bind.
SO_RCVTIMEO	DWORD	Sets the timeout, in milliseconds, for blocking receive calls.
SO_SNDBUF	int	Specifies the total per-socket buffer space reserved for sends.
SO_SNDTIMEO	DWORD	The timeout, in milliseconds, for blocking send calls.

level = IPPROTO_TCP
Value	Type	Description
TCP_NODELAY	BOOL	禁止发送合并的Nagle算法。

返回值：若无错误发生，setsockopt()返回0。否则的话，返回SOCKET_ERROR错误，应用程序可通过WSAGetLastError()获取相应错误代码。


###  UDP socket 
![](/data/dokuwiki/python/pasted/20161107-100508.png)
UDP socket server 端代码在进行 bind 后，无需调用 listen 方法。

```

# udp_echo_server.py
# coding=utf8
importsocket
 
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
# 设置 SO_REUSEADDR 后,可以立即使用 TIME_WAIT 状态的 socket
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sock.bind(('', 5500))
# 没有调用 listen
 
if __name__ == '__main__':
    while 1:
        data, addr = sock.recvfrom(1024)
 
        print('new client from %s:%s' % addr)
        sock.sendto(data, addr)
 
# udp_echo_client.py
# coding=utf8
importsocket
 
udp_server_addr = ('', 5500)
 
if __name__ == '__main__':
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    data_to_sent = 'hello udp socket'
    try:
        sent = sock.sendto(data_to_sent, udp_server_addr)
        data, server = sock.recvfrom(1024)
        print('receive data:[%s] from %s:%s' % ((data,) + server))
    finally:
        sock.close()

```
###  常见陷阱-忽略返回值 
本文中的 echo server 示例因为篇幅限制，也忽略了返回值。**网络通信是个非常复杂的问题，通常无法保障通信双方的网络状态，很有可能在发送/接收数据时失败或部分失败。所以有必要对发送/接收函数的返回值进行检查。**本文中的 tcp echo client 发送数据时，正确写法应该如下：
```

total_send = 0
content_length = len(data_to_sent)
while total_send < content_length:
    sent = sock.send(data_to_sent[total_send:])
    if sent == 0:
        raise RuntimeError("socket connection broken")
    total_send += total_send + sent

```
send/recv 操作的是网络缓冲区的数据，它们不必处理传入的所有数据。
一般来说，当网络缓冲区填满时， send函数 就返回了；当网络缓冲区被清空时， recv 函数 就返回。
**当 recv 函数返回0时，意味着对端已经关闭。**
可以通过下面的方式设置缓冲区大小。
```

s.setsockopt(socket.SOL_SOCKET, socket.SO_SNDBUF, buffer_size)

```

###  TCP 的状态机 
![](/data/dokuwiki/python/pasted/20161107-194649.png)
在前面echo server 的示例中，提到了TIME_WAIT状态，为了正式介绍其概念，需要了解下 TCP 从生成到结束的状态机器。
连接的打开与关闭都有被动（passive）与主动（active）两种，主动关闭时，涉及到的状态转移最多，包括FIN_WAIT_1、FIN_WAIT_2、CLOSING、TIME_WAIT。
此外，由于 TCP 是可靠的传输协议，所以每次发送一个数据包后，都需要得到对方的确认（ACK）.

##  实战HTTP UA 
http 协议是如今万维网的基石，可以通过 socket API 来简单模拟一个浏览器（UA）是如何解析 HTTP 协议数据的。
```

#coding=utf8
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
baidu_ip = socket.gethostbyname('baidu.com')
sock.connect((baidu_ip, 80))
print('connected to %s' % baidu_ip)
 
req_msg = [
    'GET / HTTP/1.1',
    'User-Agent: curl/7.37.1',
    'Host: baidu.com',
    'Accept: */*',
]
delimiter = '\r\n'
 
sock.send(delimiter.join(req_msg))
sock.send(delimiter)
sock.send(delimiter)
 
print('%sreceived%s' % ('-'*20, '-'*20))
http_response = sock.recv(4096)
print(http_response)

```
http_response 是通过直接调用 recv(4096) 得到的，万一真正的返回大于这个值怎么办？我们前面知道了 TCP 协议是面向流的，它本身并不关心消息的内容，需要应用程序自己去界定消息的边界，对于应用层的 HTTP 协议来说，有几种情况，最简单的一种时通过解析返回值头部的 Content-Length 属性，这样就知道 body 的大小了，对于 HTTP 1.1版本，支持 Transfer-Encoding: chunked 传输。 **TCP 协议本身无法区分消息体。**

##  Unix_domain_socket 
UDS 用于同一机器上不同进程通信的一种机制，其API适用与 network socket 很类似。只是其连接地址为本地文件而已。
代码示例参考： uds_server.py 、 uds_client.py

##  ping 
ping 命令作为检测网络联通性最常用的工具，其适用的传输协议既不是TCP，也不是 UDP，而是 ICMP，利用 raw sockets。
[ping.py](https://github.com/jiacai2050/socket.py/blob/master/in_action/ping.py)
