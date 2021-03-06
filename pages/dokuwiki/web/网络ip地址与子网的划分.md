title: 网络ip地址与子网的划分 

#  网络IP地址与子网的划分 
国际规定：把所有的IP地址划分为 A,B,C,D,E。 
　**　A类地址**：范围从0～127，''0是保留的并且表示所有IP地址，而127也是保留
的地址，并且是用于测试环回用的。''因此，A类地址的范围其实是从1～126之间。
如：10.0.0.1，**第一段号码为网络号码，剩下的三段号码为本地计算机的号码**。转换
为2进制来说，**一个A类IP地址由1字节的网络地址和3字节主机地址组成**，网络地址
的最高位必须是“０”，地址范围从0.0.0.1 到126.0.0.0。**可用的A类网络有126个，
每个网络能容纳1亿多个主机（2的24次方的主机数目）。以子网掩码来进行区别：
255.0.0.0。**
　　**B类地址**：范围从128-191，如172.168.1.1，第一和第二段号码为网络号码，剩
下的2段号码为本地计算机的号码。转换为2进制来说，一个B类IP地址由2个字节的
网络地址和2个字节的主机地址组成，网络地址的最高位必须是“10”，地址范
围从128.0.0.0到191.255.255.255。可用的B类网络有16382个，每个网络能容纳6万多
个主机。**以子网掩码来进行区别：255.255.0.0。**
　 **C类地址**：范围从192-223，如192.168.1.1，第一，第二，第三段号码为网络号
码，剩下的最后一段号码为本地计算机的号码。转换为2进制来说，一个C类IP地址
由3字节的网络地址和1字节的主机地址组成，网络地址的最高位必须是“110”。范
围从192.0.0.0到223.255.255.255。C类网络可达209万余个，每个网络能容纳254个主
机。以子网掩码来进行区别：255.255.255.0。
　　**D类地址**：范围从224-239，D类IP地址第一个字节以“１１１０”开始，''它是
一个专门保留的地址。它并不指向特定的网络，目前这一类地址被用在多点广播
（Multicast）中。多点广播地址用来一次寻址一组计算机，它标识共享同一协议的
一组计算机。'' 

 **E类地址**：范围从240-254，以“11110”开始，为将来使用保留。 ''全零
（“0.0.0.0”）地址对应于当前主机。全“1”的IP地址（“255．255．255．255”）是
当前子网的广播地址。''

　在日常网络环境中，基本是都在使用B,C两大类地址，而ADE这3类地址都不打可
能被使用到。

**子网掩码**的简单叙述：子网掩码是一个32位地址，''用于屏蔽IP地址的一部分以
区别网络标识和主机标识，并说明该IP地址是在局域网上，还是在远程网上。''
在这么多网络IP中，国际规定有一部分IP地址是用于我们的局域网使用，也就
是**属于私网IP，不在公网中使用的，它们的范围是**：
  * 10.0.0.0～10.255.255.255 
  * 172.16.0.0～172.31.255.255 
  * 192.168.0.0～192.168.255.255 

有了子网掩码后，IP地址的标识方法如下： 例：` 192.168.1.1 255.255.255.0或者标识成192.168.1.1/24（24表示掩码中“1”的个数） `

接下来我们可以看一下子网划分的例子：
 假如给你一个C类的IP地址段：192.168.0.1-192.168.0.254，其中
192.168.0 这个属于网络号码，而1～254表示这个网段中最大能容纳254台电
脑主机。我们现在要做的就是把这254台主机再次划分一下，将它们区分开来。
　　192.168.0.1-192.168.0.254默认使用的子网掩码为
255.255.255.0，其中的0在2进制中表示，8个0.**因此有8个位置没有被网络号
码给占用，2的8次方就是表示有256个地址，去掉一个头（网络地址）和一个尾
（主机地址），表示有254个电脑主机地址，因此我们想要对这254来划分的话，
就是占用最后8个0中的某几位。**

子网掩码是用来判断任意两台计算机的IP地址是否属于同一子网络。最为简单的理解就是两台计算机各自的IP地址与子网掩码进行AND运算后，如果得出的结果是相同的，则说明这两台计算机是处于同一个子网络上的，可以进行直接的通讯，就这么简单。
请看以下示例：　
运算演示之一：
IP 地址　 192.168.0.1
子网掩码　255.255.255.0　
转化为二进制进行运算：
IP 地址　11010000.10101000.00000000.00000001
子网掩码 11111111.11111111.11111111.00000000
AND运算
11000000.10101000.00000000.00000000
转化为十进制后为：
192.168.0.0

运算演示之二：
IP 地址　 192.168.0.254
子网掩码　255.255.255.0　
转化为二进制进行运算：
IP 地址　11000000.10101000.00000000.11111110
子网掩码 11111111.11111111.11111111.00000000
AND运算
11000000.10101000.00000000.00000000
转化为十进制后为：
192.168.0.0

**公网、内网是两种Internet的接入方式。**

**内网接入方式：**上网的计算机得到的IP地址是**Inetnet上的保留地址**，保留地址有如下3种形式： 
内网的计算机以` NAT（网络地址转换）协议 `，**通过一个公共的网关访问
Internet。**内网的计算机可向Internet上的其他计算机发送连接请求，但
Internet上其他的计算机无法向内网的计算机发送连接请求。

**公网接入方式**：上网的计算机得到的IP地址是**Inetnet上的非保留地址。**''公网
的计算机和Internet上的其他计算机可随意互相访问。''

** NAT（Network Address Translator）是网络地址转换，它实现内网的
IP地址与公网的地址之间的相互转换**，''将大量的内网IP地址转换为一个或少量的
公网IP地址，减少对公网IP地址的占用。''
NAT的最典型应用是：**在一个局域网内，只需要一台计算机连接上Internet，就可以利用NAT共享Internet连接，使局
域网内其他计算机也可以上网。使用NAT协议，局域网内的计算机可以访问
Internet上的计算机，但Internet上的计算机无法访问局域网内的计算机。**

**Windows操作系统的Internet连接共享、sygate、winroute、**
**unix/linux的natd等软件**，` 都是使用NAT协议来共享Internet连接。 ` 所有ISP（Internet服务提供商）**提供的内网Internet接入方式，几乎都是基于NAT协议的**。

##  示例1计算子网掩码容量  
255.255.232.0这个子网掩码可以最多容纳多少台电脑？ 
方法 
第一步：把子网掩码转换为二进制 11111111.1111111.11101000.00000000 
第二步：数数后面有几颗0，一共是有11颗，那就是2^11次方，等于2048 (注意：主机号中全0是保留地址，全1是广播地址，所以它们不算可用主号地址。网络号也是一样的。子网号是可以用全0和全1的)，所以这个子网掩码最多可以容纳2048-2=2046台电脑。

##  示例2计算子网掩码  
一个教室有50台电脑，组成一个对等局域网，子网掩码设多少最合适？ 
思路 
首先，我们从数量上看判断用ABC中的哪类IP，从50台电脑**可知用C类IP最合适**但是C类默认的子网掩码是255.255.255.0，可以容纳254台电脑，显然不太合适，那子网掩码设多少合适呢？ 
方法 
2n(子网掩码转换成二进制后的零的个数)>=50 从这个式子我们可以得出：n=6 
所以我们就可以得出子网掩码的二进制形式：11111111.1111111.11111111.11000000 然后转换成十进形式：255.255.255.192 所以**最合适的子网掩码**为：255.255.255.192
##  示例3计算子网数 
 第一步：确定该IP是属于A，B，C三类中的哪一类。就可知它们的网络号A类前8位，B类前16位，C类前24位。  
第二步：把子网掩码化成2进制看有多少个1，把该进制中1的个数减去第一步所得出的位数，即为子网位数。  
第三步：如果子网位数为n，则从理论是讲可以划分出2n个子网。

##  示例4计算网段标识与主机标识  
问题  要怎么判断两个IP地址是同一网段的呢？ 分析 
要想在同一网段，必需做到网络标识相同，那网络标识怎么算呢？ 各类IP的网络标识取法都是不一样的。 
A类的，只取第一段。B类，只取第一、二段。C类，只取第一、二、三段。 
方法 
只要把IP和子网掩码的每位数AND（与）就可以了。 AND方法：0和1=0 0和0=0 1和1=1 
例题  判断IP：12.196.132.54与56.196.56.165是否在同一网段。（默认子网掩码）  
第一步：这些转换成二进制   IP1：12.196.132.54   00001100.11000100.10000100.00110110   IP2：56.196.56.165   00111000.11000100.00111000.10100101   子网掩码：255.0.0.0       11111111.00000000.00000000.00000000  
第二步：把IP与子网掩码进行AND运算   IP1  AND 子网掩码=00001100.00000000.00000000.00000000   IP2AND 子网掩码=00111000.00000000.00000000.00000000  
第三步：把得到的结果转换成十进制   IP1的网络标识：12.0.0.0   IP2的网络标识：56.0.0.0   所以可知它们不是同一网段的。   

##  计算主机标识  
 
第一步：把子网掩码取反   取反后的子网掩码：00000000.11111111.11111111.11111111  
第二步：把它与IP进行AND运算   IP1  AND 子网掩码=00000000.11000100.10000100.00110110   IP2  AND 子网掩码=00000000.11000100.00111000.10100101  
第三步：把得到的结果转换成十进制   IP1的主机标识：0.196.132.54   IP2的主机标识：0.196.56.165 

##  划分子网  
 示例：IP：192.160.12.50（这可以是网络号）子网掩码：255.255.255.192   
第一步：把IP地址和子网掩码转换成二进制   IP地址：11000000.10100000.00001100.00110010   子网掩码：11111111.11111111.11111111.11000000   
第二步：把IP地址和子网掩码进行AND运算 
因为掩码是255.255.255.192 ，因此它们之间的网段间隔是256-192=64  广播地址：下个子网-1，所以2个子网的广播地址分别是192.160.2.127和192.160.2.191  第一个子网号：11000000.10100000.00001100.00000000（192.160.12.0）  第二个子网号：11000000.10100000.00001100.01000000（192.160.12.64）  第一个广播地址：11000000.10100000.00001100.10111110（192.160.2.127）  第三个子网号：11000000.10100000.00001100.10000000（192.160.12.128）  第二个广播地址：11000000.10100000.00001100.10111111 （192.160.2.191）  第四个子网号：11000000.10100000.00001100.11000000（192.160.12.192）  这个网段可以划分出4个子网，但只有2个可用子网（22-2）：192.160.12.64和192.160.12.128
参考：http://blog.chinaunix.net/uid-20788636-id-1841323.html
http://jingyan.baidu.com/article/4dc408489d6ab1c8d846f17b.html
http://wenku.baidu.com/link?url=1eovRzqLO1F5FfxnY8Cac5cqkA7x8KydRMhQAutcWdZcfWWctq0vNOES7UsdKPZpTVq97dETGC_EOJBI5061F-Wf-wyLe0O1UcVvC-WKwwm