createAt:2016-12-31 12:06:15
author:xbynet
modifyAt:2016-12-31 12:06:15
location:Python核心编程3e学习笔记之网络编程
title:Python核心编程3e学习笔记之网络编程

# 套接字(Socket):通信端点
套接字是计算机网络数据结构，体现了通信端点的概念。在任何类型的通信开始之前，网络应用程序必须创建套接字。
有两种类型的套接字：基于文件的(UNIX Socket文件)和面向网络的。这类型也称为**地址家族(Address Family, AF)**

* 基于文件的(UNIX Socket文件)：AF_LOCAL(**AF_UNIX**)
* 面向网络的:**AF_INET** ,AF_INET6

此外Python还引入了AF_NETLINK用于支持用户级别和内核级别代码之间的IPC(进程间通信),以及AF_TIPC用于支持计算机集群中的机器相互通信，而无需使用基于IP的寻址方式。

## 面向连接的套接字(TCP Stream Socket)和无连接的套接字(UDP Datagram Socket):
面向连接的套接字(TCP Stream Socket):通信之前必须先建立一个连接。这种类型的通信也称为**虚拟电路**或**流式套接字**。
面向连接的通信提供序列化、可靠、不重复的数据交付，而没有记录边界。这意味着每条消息可以拆分成多个片段，并且每条消息片段都确保能够到达目的地，然后将它们组合在一起，最后将完整消息传递给正在等待的应用程序。
实现这种类型的主要协议是传输控制协议(TCP).类型为**SOCK_STREAM**.

无连接的套接字：数据报类型的套接字，通信之前并不需要建立连接。数据传输过程中并无法确保它的顺序性、可靠性、重复性。记录边界。消息是以整体发送的。由于数据报开销较低，所以也有它的适用场景：实时性要求高而数据重要性相对较低。
实现这种类型的主要协议是用户数据报协议(UDP),类型为**SOCK_DGRAM**

# Python中的网络编程
相关模块

* socket 低级网络编程模块
* asyncore/asynchar 最新3.6版本已被废弃
* select 在一个单线程的网络服务器应用管理中管理多个套接字连接
* SocketServer 高级网络编程模块。

开源库

* Twisted
* Concurrence

## 低级的socket模块
使用socket.socket(socket_family,socket_type,protocol=0)来创建套接字。比如
```
socket.socket(socket.AF_INET,socket.SOCK_STREAM)
socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
```
上述返回套接字对象，下面我们介绍**套接字对象的内置方法和属性**：
服务器套接字方法
s.bind()
s.listen()
s.accept()
客户端套接字方法
s.connect()/s.connect_ex()
普通套接字方法
s.recv()/s.recv_into()  tcp消息接收
s.send()/s.sendall()   tcp消息发送
s.recvfrom()/s.recvfrom()_into  udp消息接收
s.sendto() udp消息发送
s.getpeername() 连接到tcp的远程地址
s.getsocketname() 当前套接字地址
s.shutdown()
s.close()
面向阻塞的套接字方法
s.setblocking()
s.settimeout()
s.gettimeout()
面向文件的套接字方法
s.fileno()
s.makefile()
数据属性
s.family
s.type
s.proto


-----


**socket模块属性**：
数据属性
AF_UNIX、AF_INET、AF_INET6、AF_NETLINK、AF_TIPC :套接字地址家族
SOCK_STREAM、SOCK_DGRAM :套接字类型(TCP=流、UDP=数据报)
has_ipv6 是否支持ipv6
异常
error、herror、gaierror、timeout
函数
socket()
socketpair()
create_connection()
fromfd()
ssl()
getaddrinfo()
getnameinfo()
gethostname()
gethostbyname() 将主机名映射到ip地址
gethostbyaddr() 将一个ip地址映射到dns信息.
setdefaulttimeout()
其它。
-----


### 简单的TCP服务器与客户端
```
#!/usr/bin/env python

from socket import *
from time import ctime

HOST = ''
PORT = 21567
BUFSIZ = 1024
ADDR = (HOST, PORT)

tcpSerSock = socket(AF_INET, SOCK_STREAM)
tcpSerSock.bind(ADDR)
tcpSerSock.listen(5)

while True:
    print('waiting for connection...')
    tcpCliSock, addr = tcpSerSock.accept()
    print('...connected from:', addr)

    while True:
        data = tcpCliSock.recv(BUFSIZ)
        if not data:
            break
        #tcpCliSock.send('[%s] %s' % (bytes(ctime(), 'utf-8'), data))
        tcpCliSock.send(bytes('[%s] %s' % (ctime(), data.decode('utf-8')), 'utf-8'))

    tcpCliSock.close()
tcpSerSock.close()

```
```
#!/usr/bin/env python

from socket import *

HOST = '127.0.0.1'
PORT = 21567
BUFSIZ = 1024
ADDR = (HOST, PORT)

tcpCliSock = socket(AF_INET, SOCK_STREAM)
tcpCliSock.connect(ADDR)

while True:
    data = input('> ')
    if not data:
        break
    tcpCliSock.send(bytes(data, 'utf-8'))
    data = tcpCliSock.recv(BUFSIZ)
    if not data:
        break
    print(data.decode('utf-8'))

tcpCliSock.close()

```

### 简单的UDP服务器和客户端
```
from socket import *
from time import ctime

HOST = ''
PORT = 21567
BUFSIZ = 1024
ADDR = (HOST, PORT)

udpSerSock = socket(AF_INET, SOCK_DGRAM)
udpSerSock.bind(ADDR)

while True:
    print 'waiting for message...'
    data, addr = udpSerSock.recvfrom(BUFSIZ)
    udpSerSock.sendto('[%s] %s' % (ctime(), data), addr)
    print '...received from and returned to:', addr

udpSerSock.close()
```
```
#!/usr/bin/env python

from socket import *

HOST = '127.0.0.1'
PORT = 21567
BUFSIZ = 1024
ADDR = (HOST, PORT)

udpCliSock = socket(AF_INET, SOCK_DGRAM)

while True:
    data = raw_input('> ')
    if not data:
        break
    udpCliSock.sendto(data, ADDR)
    data, ADDR = udpCliSock.recvfrom(BUFSIZ)
    if not data:
        break
    print data

udpCliSock.close()

```

## SocketServer高级模块
SocketServer模块类

* BaseServer
* TCPServer/UDPServer
* UnixStreamServer/UnixDatagramServer
* ForkingMixIn/ThreadingMixIn
* ForkingTCPServer/ForkingUDPServer
* ThreadingTCPServer/ThreadingUDPServer
* BaseRequestHandler
* **StreamRequestHandler/DatagramRequestHandler**

### 创建简单的TCP服务器和客户端
```
import SocketServer
from time import ctime

HOST = ''
PORT = 21567
ADDR = (HOST, PORT)

class MyRequestHandler(SocketServer.StreamRequestHandler):
    def handle(self):
        print '...connected from:', self.client_address
        self.wfile.write('[%s] %s\n' % (
            ctime(), self.rfile.readline().strip())
        )

tcpSerSock = SocketServer.TCPServer(ADDR, MyRequestHandler)
print 'waiting for connection...'
tcpSerSock.serve_forever()
```
```
from socket import *

HOST = '127.0.0.1'
PORT = 21567
BUFSIZ = 1024
ADDR = (HOST, PORT)

while True:
    tcpCliSock = socket(AF_INET, SOCK_STREAM)
    tcpCliSock.connect(ADDR)
    data = raw_input('> ')
    if not data:
        break
    tcpCliSock.send(data+'\n')
    data = tcpCliSock.recv(BUFSIZ)
    if not data:
        break
    print data
    tcpCliSock.close()
```

## Twisted框架介绍
Twisted是一个完整的事件驱动的网络框架，利用它可以开发完整的**异步**网络应用程序和协议。它提供大量的支持来建立完整的系统，包括网络协议、线程、安全性、身份验证、聊天/IM、DBM、RDBMS数据库集成、Web、电子邮件、命令行参数、GUI集成工具包等。

### 创建Twisted Reactor TCP服务器和客户端
```
from twisted.internet import protocol, reactor
from time import ctime

PORT = 21567

class TSServProtocol(protocol.Protocol):
    def connectionMade(self):
        clnt = self.clnt = self.transport.getPeer().host
        print '...connected from:', clnt

    def dataReceived(self, data):
        self.transport.write('[%s] %s' % (ctime(), data))

factory = protocol.Factory()
factory.protocol = TSServProtocol
print 'waiting for connection...'
reactor.listenTCP(PORT, factory)
reactor.run()

```
```
from twisted.internet import protocol, reactor

HOST = '127.0.0.1'
PORT = 21567

class TSClntProtocol(protocol.Protocol):
    def sendData(self):
        data = raw_input('> ')
        if data:
            self.transport.write(data)
        else:
            self.transport.loseConnection()

    def connectionMade(self):
        self.sendData()

    def dataReceived(self, data):
        print data
        self.sendData()

class TSClntFactory(protocol.ClientFactory):
    protocol = TSClntProtocol
    clientConnectionLost = clientConnectionFailed = \
        lambda self, connector, reason: reactor.stop()

reactor.connectTCP(HOST, PORT, TSClntFactory())
reactor.run()
```