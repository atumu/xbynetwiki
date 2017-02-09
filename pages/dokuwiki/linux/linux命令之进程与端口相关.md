title: linux命令之进程与端口相关 

#  Linux命令之进程与端口、网络相关 
##  ps 
在Linux下**ps命令**是用于查看系统上运行的进程的最基本的命令之一。它提供了当前进程的同时，如用户ID，CPU使用率，内存使用率，命令名称等,它不显示实时数据，如top或htop命令的详细信息。
语法注意：
**ps命令带有2种不一样的风格,分别是BSD和UNIX。**新用户经常会混淆和错误地解释这两种风格。所以要弄清楚他们，继续操作之前这里是一些基本的信息。
` 注意："ps aux"和"ps -aux"不相同。例如"-u"用来显示该用户的进程。但是"u"则是显示详细的信息。 `

**BSD风格:在BSD风格的语法选项前不带连字符。**
```

ps aux

``` 
**UNIX/LINUX的风格：在linux风格的语法选项前面有一个破折号如常。…**
```

ps -ef

``` 
混合使用两种Linux系统上的语法风格是好事儿。例如“ps ax -f”。但在这篇文章中，我们将主要集中在UNIX风格的语法。

----
**linux上进程有5种状态:** 
1. 运行(正在运行或在运行队列中等待) 
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号) 
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生) 
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放) 
5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行) 

**ps工具标识进程的5种状态码:** 
D 不可中断 uninterruptible sleep (usually IO) 
R 运行 runnable (on run queue) 
S 中断 sleeping 
T 停止 traced or stopped 
Z 僵死 a defunct (”zombie”) process 
----
ps 为我们提供了进程的一次性的查看，它所提供的查看结果并不动态连续的；如果想对进程时间监控，应该用 top 工具。
kill 命令用于杀死进程。

**1、显示所有进程：**
下面的命令将列出所有的进程：
```

$ ps ax 
$ ps -ef

``` 
加上管道输出给less，来滚动显示

"u"或者"-f"参数来显示所有进程的详细信息
```

$ ps aux 
$ ps -ef -f

``` 

----
**2、根据用户显示进程：**
由进程的所属用户使用“-u”选项后跟用户名来显示。多个用户名可以提供以逗号分隔。
```

$ ps -f -u www-data
UID        PID  PPID  C STIME TTY          TIME CMD 
www-data  1329  1328  0 09:32 ?        00:00:00 nginx: worker process 
www-data  1330  1328  0 09:32 ?        00:00:00 nginx: worker process 
www-data  1332  1328  0 09:32 ?        00:00:00 nginx: worker process 

``` 
3、通过名字和进程ID显示进程：
通过名字或命令搜索进程，使用“-C”选项后面加搜索词。
```

$ ps -C apache2

```

----

**4、根据CPU或者内存进行排序：**
系管理员经常希望找出那些消耗大量内存或CPU的进程。排序选项将基于特定的字段或参数让进程列表进行排序。
` “–sort”选项 `由逗号分隔的多个字段可以用指定。此外，该字段可以带有前缀“-”或“”符号，表示降序或升序分别排序。通过进程列表进行排序有很多参数，你可以检查手册页的完整列表。
```

$ ps aux --sort=-pcpu,+pmem 
显示前5个消耗了大部分的CPU进程。
$ ps aux --sort=-pcpu | head -5

```

----

**8、改变要显示的列：**
ps命令可以配置为只显示选中的列表。为了显示完整列表可以查看手册。
下面的命令只显示PID，用户名，CPU，内存和命令的列。
```

$ ps -e -o pid,uname,pcpu,pmem,comm 
可以重命名列标签，相当的灵活。
$ ps -e -o pid,uname=USERNAME,pcpu=CPU_USAGE,pmem,comm

```

----
**10、把ps命令变成一个实时查看器：**
像往常一样，watch命令可以用来实时捕捉ps显示进程。简单的例子如下:
```

$ watch -n 1 'ps -e -o pid,uname,cmd,pmem,pcpu --sort=-pmem,-pcpu | head -15'

```
输出将被刷新，每1秒刷新统计数据。不过不要以为这是类似上面。
你会注意到在相比情况下top/htop命令的输出变化会更加频繁。


参考:http://os.51cto.com/art/201312/421280.htm
http://www.cnblogs.com/peida/archive/2012/12/19/2824418.html
http://blog.jobbole.com/83610/

##  netstat 
netstat命令用于显示与IP、TCP、UDP和ICMP协议相关的统计数据，**一般用于检验本机各端口的网络连接情况。**netstat是在内核中访问网络及相关信息的程序，它能提供TCP连接，TCP和UDP监听，进程内存管理的相关报告。
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
命令参数：
-a或–all 显示所有连线中的Socket。
-A<网络类型>或–<网络类型> 列出该网络类型连线中的相关地址。
**-c或–continuous 持续列出网络状态**。
-C或–cache 显示路由器配置的快取信息。
-e或–extend 显示网络其他相关信息。
-F或–fib 显示FIB。
-g或–groups 显示多重广播功能群组组员名单。
-h或–help 在线帮助。
-i或–interfaces 显示网络界面信息表单。
**-l或–listening 显示监控中的服务器的Socket。**
-M或–masquerade 显示伪装的网络连线。
**-n或–numeric 直接使用IP地址，而不通过域名服务器。**
-N或–netlink或–symbolic 显示网络硬件外围设备的符号连接名称。
-o或–timers 显示计时器。
**-p或–programs 显示正在使用Socket的程序识别码和程序名称。**
**-r或–route 显示Routing Table。**
-s或–statistice 显示网络工作信息统计表。
**-t或–tcp 显示TCP传输协议的连线状况。**
**-u或–udp 显示UDP传输协议的连线状况。**
-v或–verbose 显示指令执行过程。
-V或–version 显示版本信息。
-w或–raw 显示RAW传输协议的连线状况。
-x或–unix 此参数的效果和指定”-A unix”参数相同。
–ip或–inet 此参数的效果和指定”-A inet”参数相同。

----

状态说明：
  * LISTEN：侦听来自远方的TCP端口的连接请求
  * SYN-SENT：再发送连接请求后等待匹配的连接请求（如果有大量这样的状态包，检查是否中招了）
  * SYN-RECEIVED：再收到和发送一个连接请求后等待对方对连接请求的确认（如有大量此状态，估计被flood攻击了）
  * ESTABLISHED：代表一个打开的连接
  * FIN-WAIT-1：等待远程TCP连接中断请求，或先前的连接中断请求的确认
  * FIN-WAIT-2：从远程TCP等待连接中断请求
  * CLOSE-WAIT：等待从本地用户发来的连接中断请求
  * CLOSING：等待远程TCP对连接中断的确认
  * LAST-ACK：等待原来的发向远程TCP的连接中断请求的确认（不是什么好东西，此项出现，检查是否被攻击）
  * TIME-WAIT：等待足够的时间以确保远程TCP接收到连接中断请求的确认
  * CLOSED：没有任何连接状态

----
列出所有端口
netstat -a
列出所有 tcp 端口 netstat -at,  列出所有 udp 端口 netstat -au
显示网卡列表
netstat -i
显示关于路由表的信息
netstat -r
列出所有 tcp 端口
netstat -at
找出程序运行的端口
netstat -ap | grep ssh
在 netstat 输出中显示 PID 和进程名称
netstat -pt
找出运行在指定端口的进程
netstat -anpt | grep ':16064'
持续输出 netstat 信息
netstat -c

一般情况下，直接用**netstat -tlnp**

参考:http://www.cnblogs.com/peida/archive/2013/03/08/2949194.html
http://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html

##  ping 
ping命令是常用的网络命令，它通常用来测试与目标主机的连通性.
具体参考：http://www.cnblogs.com/peida/archive/2013/03/06/2945407.html
##  ifconfig命令 
许多windows非常熟悉ipconfig命令行工具，它被用来获取网络接口配置信息并对此进行修改。Linux系统拥有一个类似的工具，也就是ifconfig(interfaces config)。
1．命令格式：
ifconfig [网络设备] [参数]
2．命令功能：
ifconfig 命令用来查看和配置网络设备。当网络环境发生改变时可通过此命令对网络进行相应的配置。
3．命令参数：
up 启动指定网络设备/网卡。
down 关闭指定网络设备/网卡。该参数可以有效地阻止通过指定接口的IP信息流，如果想永久地关闭一个接口，我们还需要从核心路由表中将该接口的路由信息全部删除。
arp 设置指定网卡是否支持ARP协议。
-promisc 设置是否支持网卡的promiscuous模式，如果选择此参数，网卡将接收网络中发给它所有的数据包
-allmulti 设置是否支持多播模式，如果选择此参数，网卡将接收网络中所有的多播数据包
-a 显示全部接口信息
-s 显示摘要信息（类似于 netstat -i）
add 给指定网卡配置IPv6地址
del 删除指定网卡的IPv6地址
<硬件地址> 配置网卡最大的传输单元
mtu<字节数> 设置网卡的最大传输单元 (bytes)
netmask<子网掩码> 设置网卡的子网掩码。掩码可以是有前缀0x的32位十六进制数，也可以是用点分开的4个十进制数。如果不打算将网络分成子网，可以不管这一选项；如果要使用子网，那么请记住，网络中每一个系统必须有相同子网掩码。
tunel 建立隧道
dstaddr 设定一个远端地址，建立点对点通信
-broadcast<地址> 为指定网卡设置广播协议
-pointtopoint<地址> 为网卡设置点对点通讯协议
multicast 为网卡设置组播标志
address 为网卡设置IPv4地址
txqueuelen<长度> 为网卡设置传输列队的长度

----
显示网络设备信息（激活状态的）
ifconfig
启动关闭指定网卡
ifconfig eth0 up
ifconfig eth0 down

----

###  配置IP地址 
[root@localhost ~]# ifconfig eth0 192.168.120.56 
[root@localhost ~]# ifconfig eth0 192.168.120.56 netmask 255.255.255.0 
[root@localhost ~]# ifconfig eth0 192.168.120.56 netmask 255.255.255.0 broadcast 192.168.120.255

----
启用和关闭ARP协议
ifconfig eth0 arp
ifconfig eth0 -arp

具体参考http://www.cnblogs.com/peida/archive/2013/02/27/2934525.html

