title: scapy 

#  数据包处理Scapy 
文档：http://www.secdev.org/projects/scapy/doc/
中文：https://github.com/Larryxi/Scapy_zh-cn
github:https://github.com/secdev/scapy
当前版本：2.3.3
但是很遗憾，Scapy2.x版本不会支持Python3，怎么办？
github上有一个Scapy3k版本：https://github.com/phaethon/scapy，文档：https://phaethon.github.io/scapy/api/installation.html
安装：pip install scapy-python3 对于windows，还需要安装winpcap这个二进制程序
与2.x的区别：
use bytes() instead of str() when converting packet to bytes. Also, most arguments expect bytes value instead of str value except the ones, which are naturally suited for human input (e.g. domain name).

Scapy的是一个强大的交互式数据包处理程序（使用Python编写）。
它能够伪造或者解码大量的网络协议数据包，能够发送、捕捉、匹配请求和回复包等等。它可以很容易地处理一些典型操作，比如**端口扫描**，tracerouting，探测，单元测试，**攻击或网络发现**（可替代hping，NMAP，arpspoof，ARP-SK，arping，tcpdump，tethereal，P0F等）。最重要的他还有很多更优秀的特性——发送无效数据帧、注入修改的802.11数据帧、在WEP上解码加密通道（VOIP）、**ARP缓存攻击**（VLAN）等，这也是其他工具无法处理完成的。
Scapy是一个可以让用户发送、侦听和解析并伪装网络报文的Python程序。这些功能可以用于制作侦测、扫描和攻击网络的工具。

测试：
```

$ sudo scapy
Welcome to Scapy (2.2.0)  
>>> IP()  
<IP  |>  
>>> target="www.baidu.com"  
>>> ip=IP(dst=target)  
>>> ip  
<IP  dst=Net('www.baidu.com') |>  
>>> [p for p in ip]  
[<IP  dst=180.97.33.107 |>]  
>>> 

```
TCP路由跟踪测试
```

from scapy.all import *  
ans,unans=sr(IP(dst="www.google.com",ttl=(2,25),id=RandShort())/TCP(flags=0x2))  
for snd,rcv in ans:  
    print snd.ttl,rcv.src,isinstance(rcv.payload,TCP)

```  
一、数据包
包(Packet)是TCP/IP协议通信传输中的数据单位，一般也称“数据包”。其主要**由“目的IP地址”、“源IP地址”、“净载数据”等部分构成**，包括包头和包体，包头是固定长度，包体的长度不定，各字段长度固定，双方的请求数据包和应答数据包的包头结构是一致的，不同的是包体的定义。 数据包的结构与我们平常写信非常类似，目的IP地址是说明这个数据包是要发给谁的，相当于收信人地址；源IP地址是说明这个数据包是发自哪里的，相当于发信人地址；而净载数据相当于信件的内容。包沿着不同的路径在一个或多个网络中传输，并且在目的地重新组合。

二、常见的几个关键字
**ICMP:Internet Control Message Protocol（Internet控制报文协议）的缩写**。它是TCP/IP协议族的一个子协议，用于在IP主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。这些控制消息虽然并不传输用户数据，但是对于用户数据的传递起着重要的作用。

DST:目的地址
SRC:源地址

TTL：(Time To Live ) 生存时间,指定数据包被路由器丢弃之前**允许通过的网段数量**。TTL是IP协议包中的一个值，它告诉网络，数据包在网络中的时间是否太长而应被丢弃。有很多原因使包在一定时间内不能被传递到目的地。解决方法就是在一段时间后丢弃这个包，然后给发送者一个报文，由发送者决定是否要重发。TTL的初值通常是系统缺省值，是包头中的8位的域。TTL的最初设想是确定一个时间范围，超过此时间就把包丢弃。由于每个路由器都至少要把TTL域减一，TTL通常表示包在被丢弃前最多能经过的路由器个数。当记数到0时，路由器决定丢弃该包，并发送一个ICMP报文给最初的发送者。

三、scapy中常用的几个辅助函数
1、ls():作用也是list show，可以显示所有支持的数据包对象。ls()可以不带参数，也可以带参数，参数可是任何一个具体的包。
2、lsc()列出所有函数。
3、hide_defaults()方法，用来删除一些用户提供的那些和default value相同的项目
4、display():display()方法可以简单查看当前packet的各个参数的取值情况。类似的还有：
summary()   显示一个关于每个数据包的摘要列表
show()  显示首选表示（通常用nsummary()）
5、sprintf:输出某一层某个参数的取值，如果不存在就输出？？,具体的format是：&lt;nowiki&gt;%[[mt][r],][layer[:nb].]field%,&lt;/nowiki&gt;参数的具体信息请参看《Security Power Tools》146页

例如
```

ans,unans=sr(IP(dst="192.168.8.1")/TCP(dport=[21,22,23]))
ans.show()
ans.summary()
ans.summary( lambda s,r: r.sprintf("%TCP.sport% \t %TCP.flags%") )
ans.summary(lfilter = lambda s,r: r.sprintf("%TCP.flags%") == "SA",prn=lambda s,r:r.sprintf("%TCP.sport% is open"))

```
四、创建包
scapy的包创建是按照**网络接口层，互联网层，传输层，应用层四层参考模型**来完成，各个层都有各自的创建函数，比如IP（），TCP（），UDP()等等，不同层之间通过“/”来连接。

###  简单的发送包-ping 
1、send()在第三层发送数据包，但没有接收功能。如：
```

>>> send(IP(dst="www.baidu.com",ttl=1)/ICMP())

```
 这里相当于ping了下百度，ttl=1

2、sendp()，在第二层发送数据包，同样没有接收功能。如：
```

>>> sendp(Ether()/IP(dst="www.baidu.com",ttl=1)/ICMP())
WARNING: Mac address to reach destination not found. Using broadcast.
.
Sent 1 packets.
>>> sendp(Ether()/IP(dst="127.0.0.1",ttl=1)/ICMP())
.
Sent 1 packets.

```
3、sr()，在第三层发送数据包，有接收功能。如：
```

>>> p=sr(IP(dst="www.baidu.com",ttl=1)/ICMP())
Begin emission:
..Finished to send 1 packets.
.*
Received 4 packets, got 1 answers, remaining 0 packets
>>> p
(<Results: TCP:0 UDP:0 ICMP:1 Other:0>, <Unanswered: TCP:0 UDP:0 ICMP:0 Other:0>)
>>> p[0]
<Results: TCP:0 UDP:0 ICMP:1 Other:0>
>>> p[0].show()
0000 IP / ICMP 27.214.222.160 > 61.135.169.105 echo-request 0 ==> IP / ICMP 27.214.220.1 > 27.214.222.160 time-exceeded ttl-zero-during-transit / IPerror / ICMPerror

```
4、sr1(),在第三层发送数据包，有接收功能，但只接收第一个包。以上面的发送四个包为例：
```

>>> q=sr1(IP(dst="www.baidu.com",ttl=(1,4))/ICMP())
Begin emission:
Finished to send 4 packets.
.*.*.*.*
Received 8 packets, got 4 answers, remaining 0 packets
>>> q
<IP  version=4L ihl=5L tos=0xc0 len=56 id=4773 flags= frag=0L ttl=255 proto=icmp chksum=0xb611 src=27.214.220.1 dst=27.214.222.160 options=[] |<ICMP  type=time-exceeded code=ttl-zero-during-transit chksum=0xf4ff unused=0 |<IPerror  version=4L ihl=5L tos=0x0 len=28 id=1 flags= frag=0L ttl=1 proto=icmp chksum=0xd879 src=27.214.222.160 dst=61.135.169.105 options=[] |<ICMPerror  type=echo-request code=0 chksum=0xf7ff id=0x0 seq=0x0 |>>>>
>>> q.show()
###[ IP ]###
  version= 4L
  ihl= 5L
  tos= 0xc0
  len= 56
  id= 4773
  flags= 
  frag= 0L
  ttl= 255
  proto= icmp
  chksum= 0xb611
  src= 27.214.220.1
  dst= 27.214.222.160
  \options\
###[ ICMP ]###
     type= time-exceeded
     code= ttl-zero-during-transit
     chksum= 0xf4ff
     unused= 0
###[ IP in ICMP ]###
        version= 4L
        ihl= 5L
        tos= 0x0
        len= 28
        id= 1
        flags= 
        frag= 0L
        ttl= 1
        proto= icmp
        chksum= 0xd879
        src= 27.214.222.160
        dst= 61.135.169.105
        \options\
###[ ICMP in ICMP ]###
           type= echo-request
           code= 0
           chksum= 0xf7ff
           id= 0x0
           seq= 0x0

```
5、srloop()，在第三层工作。
p=srloop(IP(dst="www.baidu.com",ttl=1)/ICMP())  ，将会不停的ping百度
6、srp()、srp1()、srploop()与上面3、4、5相同，只是工作在第二层。
二、SYN扫描
SYN扫描：也叫“半开式扫描”（half-open scanning），因为它没有完成一个完整的TCP连接。这种方法向目标端口发送一个SYN分组（packet），如果目标端口返回SYN/ACK，那么可以肯定该端口处于检听状态；否则，返回的是RST/ACK。
```

>>> sr1(IP(dst="61.135.169.105")/TCP(dport=80,flags="S"))
如果要扫描多个端口，可以使用以下语句，如扫描百度的80－83端口：
>>>sr(IP(dst="www.baidu.com")/TCP(dport=(80,83),flags="S"))
如要扫描21，80，3389等端口：
>>>sr(IP(dst="www.baidu.com")/TCP(dport=[21,80,3389],flags="S"))

```
三、TCP traceroute
traceroute:用来追踪出发点到目的地所经过的路径,通过Traceroute我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径。当然每次数据包由某一同样的出发点（source）到达某一同样的目的地(destination)走的路径可能会不一样，但基本上来说大部分时候所走的路由是相同的。
```

>>> ans,unans=sr(IP(dst="www.baidu.com",ttl=(4,25),id=RandShort())/TCP(flags=0x2))
>>> for snd,rcv in ans:
...     print snd.ttl,rcv.src,isinstance(rcv.payload,TCP)

```

http://www.cnblogs.com/xiaowuyi/p/3337189.html
##  端口扫描初步 
http://www.tuicool.com/articles/mEvEf2B
http://www.freebuf.com/sectool/94507.html
常见的端口扫描类型有：
1. TCP 连接扫描
2. TCP SYN 扫描(也称为半开放扫描或stealth扫描)
3. TCP 圣诞树(Xmas Tree)扫描
4. TCP FIN 扫描
5. TCP 空扫描(Null)
6. TCP ACK 扫描
7. TCP 窗口扫描
8. UDP 扫描

###  TCP 连接扫描
客户端与服务器建立 TCP 连接要进行一次**三次握手**，如果进行了一次成功的三次握手，则说明端口开放。
![](/data/dokuwiki/python/pasted/20161107-002422.png)
客户端想要连接服务器80端口时，会先发送一个带有 SYN 标识和端口号的 TCP 数据包给服务器(本例中为80端口)。如果端口是开放的，则服务器会接受这个连接并返回一个带有 SYN 和 ACK 标识的数据包给客户端。随后客户端会返回带有 ACK 和 RST 标识的数据包，此时客户端与服务器建立了连接。如果完成一次三次握手，那么服务器上对应的端口肯定就是开放的。
![](/data/dokuwiki/python/pasted/20161107-002506.png)
当客户端发送一个带有 SYN 标识和端口号的 TCP 数据包给服务器后，如果服务器端返回一个带 RST 标识的数据包，则说明端口处于关闭状态。
代码：
```

#! /usr/bin/python
import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "10.0.0.1"
src_port = RandShort()
dst_port=80

tcp_connect_scan_resp = sr1(IP(dst=dst_ip)/TCP(sport=src_port,dport=dst_port,flags="S"),timeout=10)
if(str(type(tcp_connect_scan_resp))==""):
    print "Closed"
elif(tcp_connect_scan_resp.haslayer(TCP)):
    if(tcp_connect_scan_resp.getlayer(TCP).flags == 0x12):
        send_rst = sr(IP(dst=dst_ip)/TCP(sport=src_port,dport=dst_port,flags="AR"),timeout=10)
        print "Open"
elif (tcp_connect_scan_resp.getlayer(TCP).flags == 0x14):
    print "Closed"

```
###  TCP SYN 扫描 
![](/data/dokuwiki/python/pasted/20161107-002629.png)
这个技术同 TCP 连接扫描非常相似。同样是客户端向服务器发送一个带有 SYN 标识和端口号的数据包，如果目标端口开发，则会返回带有 SYN 和 ACK 标识的 TCP 数据包。但是，这时客户端不会发送 RST+ACK 而是一个只带有 RST 标识的数据包。**这种技术主要用于躲避防火墙的检测。**
如果目标端口处于关闭状态，那么同之前一样，服务器会返回一个 RST 数据包。
```

#! /usr/bin/python
import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "10.0.0.1"
src_port = RandShort()
dst_port=80

stealth_scan_resp = sr1(IP(dst=dst_ip)/TCP(sport=src_port,dport=dst_port,flags="S"),timeout=10)
if(str(type(stealth_scan_resp))==""):
    print "Filtered"
elif(stealth_scan_resp.haslayer(TCP)):
    if(stealth_scan_resp.getlayer(TCP).flags == 0x12):
        send_rst = sr(IP(dst=dst_ip)/TCP(sport=src_port,dport=dst_port,flags="R"),timeout=10)
        print "Open"
    elif (stealth_scan_resp.getlayer(TCP).flags == 0x14):
        print "Closed"
elif(stealth_scan_resp.haslayer(ICMP)):
    if(int(stealth_scan_resp.getlayer(ICMP).type)==3 and int(stealth_scan_resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
        print "Filtered"

```
###  TCP 圣诞树(Xmas Tree)扫描 
![](/data/dokuwiki/python/pasted/20161107-002750.png)
在圣诞树扫描中，客户端会向服务器发送带有 PSH,FIN,URG 标识和端口号的数据包给服务器。如果目标端口是开放的，那么不会有任何来自服务器的回应。
如果服务器返回了一个带有 RST 标识的 TCP 数据包，那么说明端口处于关闭状态。
![](/data/dokuwiki/python/pasted/20161107-002841.png)
但如果服务器返回了一个 ICMP 数据包，其中包含 ICMP 目标不可达错误类型3以及 ICMP 状态码为1，2，3，9，10或13，则说明目标端口被过滤了无法确定是否处于开放状态。
```

#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "10.0.0.1"
src_port = RandShort()
dst_port=80

xmas_scan_resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="FPU"),timeout=10)
if (str(type(xmas_scan_resp))==""):
    print "Open|Filtered"
elif(xmas_scan_resp.haslayer(TCP)):
    if(xmas_scan_resp.getlayer(TCP).flags == 0x14):
        print "Closed"
elif(xmas_scan_resp.haslayer(ICMP)):
    if(int(xmas_scan_resp.getlayer(ICMP).type)==3 and int(xmas_scan_resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
        print "Filtered"

```
###  TCP FIN扫描 
![](/data/dokuwiki/python/pasted/20161107-002920.png)
FIN 扫描会向服务器发送带有 FIN 标识和端口号的 TCP 数据包。如果没有服务器端回应则说明端口开放。
如果服务器返回一个 RST 数据包，则说明目标端口是关闭的。
如果服务器返回了一个 ICMP 数据包，其中包含 ICMP 目标不可达错误类型3以及 ICMP 代码为1，2，3，9，10或13，则说明目标端口被过滤了无法确定端口状态。
```

#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "10.0.0.1"
src_port = RandShort()
dst_port=80

fin_scan_resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="F"),timeout=10)
if (str(type(fin_scan_resp))==""):
    print "Open|Filtered"
elif(fin_scan_resp.haslayer(TCP)):
    if(fin_scan_resp.getlayer(TCP).flags == 0x14):
        print "Closed"
elif(fin_scan_resp.haslayer(ICMP)):
    if(int(fin_scan_resp.getlayer(ICMP).type)==3 and int(fin_scan_resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
        print "Filtered"

```
###  TCP 空扫描(Null) 
![](/data/dokuwiki/python/pasted/20161107-003004.png)
在空扫描中，客户端发出的 TCP 数据包仅仅只会包含端口号而不会有其他任何的标识信息。如果目标端口是开放的则不会回复任何信息。
如果服务器返回了一个 RST 数据包，则说明目标端口是关闭的。
如果返回 ICMP 错误类型3且代码为1，2，3，9，10或13的数据包，则说明端口被服务器过滤了。
```

#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "10.0.0.1"
src_port = RandShort()
dst_port=80

null_scan_resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags=""),timeout=10)
if (str(type(null_scan_resp))==""):
    print "Open|Filtered"
elif(null_scan_resp.haslayer(TCP)):
    if(null_scan_resp.getlayer(TCP).flags == 0x14):
        print "Closed"
elif(null_scan_resp.haslayer(ICMP)):
    if(int(null_scan_resp.getlayer(ICMP).type)==3 and int(null_scan_resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
        print "Filtered"

```
###  TCP ACK扫描 
**ACK 扫描不是用于发现端口开启或关闭状态的，而是用于发现服务器上是否存在有状态防火墙的。它的结果只能说明端口是否被过滤。**再次强调，ACK 扫描不能发现端口是否处于开启或关闭状态。
![](/data/dokuwiki/python/pasted/20161107-003115.png)
客户端会发送一个带有 ACK 标识和端口号的数据包给服务器。如果服务器返回一个带有 RST 标识的 TCP 数据包，则说明端口没有被过滤，不存在状态防火墙。
如果目标服务器没有任何回应或者返回ICMP 错误类型3且代码为1，2，3，9，10或13的数据包，则说明端口被过滤且存在状态防火墙。
```

#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "10.0.0.1"
src_port = RandShort()
dst_port=80

ack_flag_scan_resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="A"),timeout=10)
if (str(type(ack_flag_scan_resp))==""):
    print "Stateful firewall presentn(Filtered)"
elif(ack_flag_scan_resp.haslayer(TCP)):
    if(ack_flag_scan_resp.getlayer(TCP).flags == 0x4):
        print "No firewalln(Unfiltered)"
elif(ack_flag_scan_resp.haslayer(ICMP)):
    if(int(ack_flag_scan_resp.getlayer(ICMP).type)==3 and int(ack_flag_scan_resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
        print "Stateful firewall presentn(Filtered)"

```
###  TCP窗口扫描 
![](/data/dokuwiki/python/pasted/20161107-003154.png)
TCP 窗口扫描的流程同 ACK 扫描类似，同样是客户端向服务器发送一个带有 ACK 标识和端口号的 TCP 数据包，但是这种扫描能够用于发现目标服务器端口的状态。在 ACK 扫描中返回 RST 表明没有被过滤，但在窗口扫描中，当收到返回的 RST 数据包后，它会检查窗口大小的值。如果窗口大小的值是个非零值，则说明目标端口是开放的。
如果返回的 RST 数据包中的窗口大小为0，则说明目标端口是关闭的。
```

#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "10.0.0.1"
src_port = RandShort()
dst_port=80

window_scan_resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="A"),timeout=10)
if (str(type(window_scan_resp))==""):
    print "No response"
elif(window_scan_resp.haslayer(TCP)):
    if(window_scan_resp.getlayer(TCP).window == 0):
        print "Closed"
    elif(window_scan_resp.getlayer(TCP).window > 0):
        print "Open"

```
###  UDP扫描 
TCP 是面向连接的协议，而UDP则是无连接的协议。
面向连接的协议会先在客户端和服务器之间建立通信信道，然后才会开始传输数据。如果客户端和服务器之间没有建立通信信道，则不会有任何产生任何通信数据。
无连接的协议则不会事先建立客户端和服务器之间的通信信道，只要客户端到服务器存在可用信道，就会假设目标是可达的然后向对方发送数据。
![](/data/dokuwiki/python/pasted/20161107-003244.png)
客户端会向服务器发送一个带有端口号的 UDP 数据包。如果服务器回复了 UDP 数据包，则目标端口是开放的。
如果服务器返回了一个 ICMP 目标不可达的错误和代码3，则意味着目标端口处于关闭状态。
如果服务器返回一个 ICMP 错误类型3且代码为1，2，3，9，10或13的数据包，则说明目标端口被服务器过滤了。
但如果服务器没有任何相应客户端的 UDP 请求，则可以断定目标端口可能是开放或被过滤的，无法判断端口的最终状态。
```

#! /usr/bin/python

import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import *

dst_ip = "10.0.0.1"
src_port = RandShort()
dst_port=53
dst_timeout=10

def udp_scan(dst_ip,dst_port,dst_timeout):
    udp_scan_resp = sr1(IP(dst=dst_ip)/UDP(dport=dst_port),timeout=dst_timeout)
    if (str(type(udp_scan_resp))==""):
        retrans = []
        for count in range(0,3):
            retrans.append(sr1(IP(dst=dst_ip)/UDP(dport=dst_port),timeout=dst_timeout))
        for item in retrans:
            if (str(type(item))!=""):
                udp_scan(dst_ip,dst_port,dst_timeout)
        return "Open|Filtered"
    elif (udp_scan_resp.haslayer(UDP)):
        return "Open"
    elif(udp_scan_resp.haslayer(ICMP)):
        if(int(udp_scan_resp.getlayer(ICMP).type)==3 and int(udp_scan_resp.getlayer(ICMP).code)==3):
            return "Closed"
        elif(int(udp_scan_resp.getlayer(ICMP).type)==3 and int(udp_scan_resp.getlayer(ICMP).code) in [1,2,9,10,13]):
            return "Filtered"

print udp_scan(dst_ip,dst_port,dst_timeout)

```
下面解释下上述代码中的一些函数和变量：
  * RandShort()：产生随机数 
  * type()：获取数据类型 
  * sport：源端口号
  * dport：目标端口号
  * timeout：等待相应的时间
  * haslayer()：查找指定层：TCP或UDP或ICMP
  * getlayer()：获取指定层：TCP或UDP或ICMP

` 以上源码地址：https://github.com/interference-security/Multiport `
###  简单扫描示例 
构建数据包a
a=Ether()/IP()/TCP()/””
/:各个层之间的分隔
():内可以存放这一层的参数设置和数据
常用方法函数
hexdump(a):包的内容十六进制显示
str(pkt) 组装包
hexdump(pkt) 十六进制转储
ls(pkt) 字段值的列表
pkt.summary() 总结
pkt.show() 数据包
pkt.show2() 一样显示但组装包(例如校验和计算)
pkt.sprintf() 格式字符串填满包的字段值
summary() 显示一个列表,每个包的摘要
nsummary() 包的数量
filter() 用lambda函数返回一个过滤后的包列表
hexraw() 返回原始十六进制数据
make table() 用lambda函数返回一个表格
send():第三层发送数据包,自动处理物理层和数据链路层
sendp():第二层发送,可以选择接口和链路层协议
sr():发送包并接受回答
sr1():只返回一个被发送的包

例子
DNS查询包: sr1(IP(dst=”DNS服务器地址”)/UDP()/DNS(rd=1,qd=DNSQR(qname=”查询域名”)))
SYN扫描: sr1(IP(dst=”目的IP”)/TCP(dport=目的端口,flags=”S”))

TCP FLAG
TCP FLAG 标记基于标记的TCP包匹配经常被用于过滤试图打开新连接的TCP数据包。
TCP标记和他们的意义如下所列:
F : FIN – 结束; 结束会话
S : SYN – 同步; 表示开始会话请求
R : RST – 复位;中断一个连接
P : PUSH – 推送; 数据包立即发送
A : ACK – 应答
U : URG – 紧急
E : ECE – 显式拥塞提醒回应
W : CWR – 拥塞窗口减少
常用参数
inter: 设置两个数据包之间等待的时间间隔。如果有些数据包丢失了，或者设置时间间隔不足以满足要求，你可以重新发送所有无应答数据包。
retry:设置重复发送次数。 如果retry设置为3，scapy会对无应答的数据包重复发送三次。如果retry设为-3，scapy则会一直发送无应答的数据包。
timeout: 设置在最后一个数据包发出去之后的等待时间。

TCP-SYN扫描初步
TCP-SYN扫描可以明确可靠地区分 open (开放的)， closed (关闭的)，和 filtered (被过滤的) 状态。又被称为半开放扫描， 因为它不打开一个完全的TCP连接。它发送一个SYN报文， 就像您真的要打开一个连接，然后等待响应。 SYN/ACK表示端口在监听 (开放)，而 RST (复位)表示没有监听者。如果数次重发后仍没响应， 该端口就被标记为被过滤。如果收到ICMP不可到达错误 (类型3，代码1，2，3，9，10，或者13)，该端口也被标记为被过滤。
即三种可能的标志位/数据包状况：
接收到数据包，flags=”SA”：端口开放。
接收到数据包，flags=”RA”：端口未被监听。
没有接收到数据包：端口关闭或被防火墙禁止。


参考：http://blog.csdn.net/wang_walfred/article/details/40072751
http://www.tuicool.com/articles/uYVfimF