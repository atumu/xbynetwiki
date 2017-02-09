title: ehcache_memcache_redis 

#  Ehcache、memcache、redis缓存介绍 
在开发中大型Java软件项目时，很多Java架构师都会遇到数据库读写瓶颈，如果你在系统架构时并没有将缓存策略考虑进去，或者并没有选择更优的缓存策略，那么到时候重构起来将会是一个噩梦。
使用缓存的原则就是：尽量用低开销的计算代替高开销的计算。
##  1、Ehcache 
1、Ehcache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider。
主要的特性有：
1. 快速.
2. 简单.
3. **多种缓存策略** 提供LRU、LFU、FIFO淘汰算法
4. **缓存数据有两级：内存和磁盘**，因此无需担心容量问题
5. 缓存数据会在虚拟机重启的过程中写入磁盘
6. 可以通过RMI、可插入API等方式进行分布式缓存
7. 具有缓存和缓存管理器的侦听接口
8. 支持多缓存管理器实例，以及一个实例的多个缓存区域
9. 提供Hibernate的缓存实现
10. 等等
官方网站：http://ehcache.org/
 够简单
开发者提供的接口非常简单明了，从Ehcache的搭建到运用运行仅仅需要的是你宝贵的几分钟。其实很多开发者都不知道自己用在用Ehcache，Ehcache被广泛的运用于其他的开源项目
比如：hibernate

够袖珍
关于这点的特性，官方给了一个很可爱的名字small foot print ，一般Ehcache的发布版本不会到2M，V 2.2.3  才 668KB。

够轻量
核心程序仅仅依赖slf4j这一个包，没有之一！

好扩展
**Ehcache提供了对大数据的内存和硬盘的存储**，最近版本允许多实例、保存对象高灵活性、提供LRU、LFU、FIFO淘汰算法，基础属性支持热配置、支持的插件多

监听器
**缓存管理器监听器 （CacheManagerListener）和 缓存监听器（CacheEvenListener**）,做一些统计或数据一致性广播挺好用的

如何使用？
够简单就是Ehcache的一大特色，自然用起来just so easy!

贴一段基本使用代码
```

CacheManager manager = CacheManager.newInstance("src/config/ehcache.xml");
Ehcache cache = new Cache("testCache", 5000, false, false, 5, 2);
cacheManager.addCache(cache);

```
代码中有个ehcache.xml文件，现在来介绍一下这个文件中的一些属性
       name:缓存名称。
       maxElementsInMemory：缓存最大个数。
       eternal:对象是否永久有效，一但设置了，timeout将不起作用。
       timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
       timeToLiveSeconds：设置对象在失效前允许存活时间,最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时 间无穷大。
       overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。
       diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
       maxElementsOnDisk：硬盘最大缓存个数。
       diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
       diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
       memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU。你可以设置为 FIFO或是LFU。
       clearOnFlush：内存数量最大时是否清除。
       
##  2、memcache 
**memcache 是一种高性能、分布式对象缓存系统**，最初设计于缓解动态网站数据库加载数据的延迟性，你可以把它想象成一个大的内存HashTable，就是一个key-value键值缓存。
**Memcached 是一种集中式 Cache**，支持分布式横向扩展。通过在内存里维护一个统一的巨大的hash表，Memcached能够用来存储各种格式的数据，包括图像、视频、文件以及数据库检索的结果等。
**Memcached服务端本身是单实例的，只是在客户端实现过程中可以根据存储的主键作分区存储**，而这个区就是Memcached服务端的一个或者多个实例，如果将客户端也囊括到Memcached中，那么可以部分概念上说是集中式的。其实集中式的构架，无非两种情况：1.` 节点均衡的网状（JBoss Tree Cache），利用JGroup的多播通信机制来同步数据 `。2.Master-Slaves模式（分布式文件系统），由Master来管理Slave，如何选择Slave，如何迁移数据，都是由Master来完成，但是Master本身也存在单点问题。

**Memcached内存分配机制**：首先要说明的是Memcached**支持最大的存储对象为1M。key的长度小于250字符**; 像vmware, xen这类虚拟化技术并不适合运行memcached; **Memcached未提供任何安全策略，仅仅通过telnet就可以访问到memcached**, Memcached Java客户端代码的人就会了解其实客户端的事情很简单，就是要有一套高性能的 Socket通信框架以及对 Memcached 的私有协议实现的接口。

在基于分布式缓存的应用中，要确保每个缓存中的数据完全的一致是不可能的，总会存在这样那样的问题，即使像memcached，也因为没有commit机制，可能出现一个node上先放入cache，而最后transaction回滚，但其他的cache node已经为其他用户提供了这个数据。

首先memcached是独立的服务器组件,独立于应用系统,从客户端保存和读取对象到memcached是必须通过网络传输,因为网络传输都是二进制的数据,所以所有的对象都必须经过序列化,否则无法存储到memcahced的服务器端。

1.依赖
memcache C语言所编写，依赖于最近版本的GCC和libevent。GCC是它的编译器，同事基于libevent做socket io。在安装memcache时保证你的系统同事具备有这两个环境。
2.多线程支持
memcache支持多个cpu同时工作，在memcache安装文件下有个叫threads.txt中特别说明，By default, memcached is compiled as a single-threaded application.
**` 默认是单线程编译安装 `**，如果你需要多线程则需要修改./configure --enable-threads，为了支持多核系统，前提是你的系统必须具有多线程工作模式。开启多线程工作的线程数默认是4，如果线程数超过cpu数容易发生操作死锁的概率。结合自己业务模式选择才能做到物尽其用。

3.高性能
通过libevent完成socket 的通讯，理论上性能的瓶颈落在网卡上。
简单安装：
```

1.分别把memcached和libevent下载回来，放到 /tmp 目录下：
# cd /tmp
# wget http://www.danga.com/memcached/dist/memcached-1.2.0.tar.gz
# wget http://www.monkey.org/~provos/libevent-1.2.tar.gz
2.先安装libevent：
# tar zxvf libevent-1.2.tar.gz
# cd libevent-1.2
# ./configure -prefix=/usr
# make （如果遇到提示gcc 没有安装则先安装gcc)
# make install

```

4.安装memcached，同时需要安装中指定libevent的安装位置：
# cd /tmp
# tar zxvf memcached-1.2.0.tar.gz
# cd memcached-1.2.0
# ./configure -with-libevent=/usr
# make
# make install
如果中间出现报错，请仔细检查错误信息，按照错误信息来配置或者增加相应的库或者路径。
安装完成后会把memcached放到 /usr/local/bin/memcached ，

**启动Memcached服务：**
1.启动Memcache的服务器端：
# /usr/local/bin/memcached -d -m 8096 -u root -l 192.168.77.105 -p 12000 -c 256 -P /tmp/memcached.pid
-d选项是启动一个守护进程，
-m是分配给Memcache使用的内存数量，单位是MB，我这里是8096MB，
-u是运行Memcache的用户，我这里是root，
-l是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.77.105.默认是本机
-p是设置Memcache监听的端口，我这里设置了12000，最好是1024以上的端口，
-c选项是最大运行的并发连接数，默认是1024，我这里设置了256，按照你服务器的负载量来设定，
-P是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid，
-d start 启动memcached服务
-d restart 重起memcached服务
-d stop|shutdown 关闭正在运行的memcached服务

windows下:
 -d install 安装memcached服务
-d uninstall 卸载memcached服务

** memcache 的连接**
telnet  ip   port 
注意连接之前需要再memcache服务端把memcache的防火墙规则加上
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT 
重新加载防火墙规则
service iptables restart
OK ,现在应该就可以连上memcache了
在客户端输入stats 查看memcache的状态信息
pid              memcache服务器的进程ID
uptime      服务器已经运行的秒数
time           服务器当前的unix时间戳
version     memcache版本
pointer_size         当前操作系统的指针大小（32位系统一般是32bit）
rusage_user          进程的累计用户时间
rusage_system    进程的累计系统时间
curr_items            服务器当前存储的items数量
total_items           从服务器启动以后存储的items总数量
bytes                       当前服务器存储items占用的字节数
curr_connections        当前打开着的连接数
total_connections        从服务器启动以后曾经打开过的连接数
connection_structures          服务器分配的连接构造数
cmd_get get命令          （获取）总请求次数
cmd_set set命令          （保存）总请求次数
get_hits          总命中次数
get_misses        总未命中次数
evictions     为获取空闲内存而删除的items数（分配给memcache的空间用满后需要删除旧的items来得到空间分配给新的items）
bytes_read    读取字节数（请求字节数）
bytes_written     总发送字节数（结果字节数）
limit_maxbytes     分配给memcache的内存大小（字节）
threads         当前线程数

###  memcache集群配置 
由于Memcached服务器与服务器之间没有任何通讯，并且不进行任何数据复制备份，所以当任何服务器节点出现故障时，会出现单点故障，如果需要实现HA，则需要通过另外的方式来解决。
通过Magent缓存代理，防止单点现象，缓存代理也可以做备份，通过客户端连接到缓存代理服务器，缓存代理服务器连接缓存连接服务器，缓存代理服务器可以连接多台Memcached机器可以将每台Memcached机器进行数据同步。如果其中一台缓存服务器down机，系统依然可以继续工作，如果其中一台Memcached机器down掉，数据不会丢失并且可以保证数据的完整性。具体可以参考：http://code.google.com/p/memagent/
memcache集群的实现
memcached尽管是“分布式”缓存服务器，但服务器端并没有分布式功能。各个memcached不会互相通信以共享信息。那么，怎样进行分布式呢？这完全取决于客户端的实现。

##  redis 
redis是在memcache之后编写的，大家经常把这两者做比较，如果说它是个key-value store 的话但是**它具有丰富的数据类型**，我想暂时把它叫做缓存数据流中心，就像现在物流中心那样，order、package、store、classification、distribute、end。。
先说说reidis的特性
1. 支持持久化
redis的本地持久化支持两种方式：RDB和AOF。RDB 在redis.conf配置文件里配置持久化触发器，AOF指的是redis没增加一条记录都会保存到持久化文件中（保存的是这条记录的生成命令），如果不是用redis做DB用的话还会不要开AOF ，数据太庞大了，重启恢复的时候是一个巨大的工程！

2.丰富的数据类型
redis 支持 String 、Lists、sets、sorted sets、hashes 多种数据类型,新浪微博会使用redis做nosql主要也是它具有这些类型，时间排序、职能排序、我的微博、发给我的这些功能List 和 sorted set
的强大操作功能息息相关

3.高性能
这点跟memcache很想象，内存操作的级别是毫秒级的比硬盘操作秒级操作自然高效不少，较少了磁头寻道、数据读取、页面交换这些高开销的操作！这也是NOSQL冒出来的原因吧，应该是高性能
是基于RDBMS的衍生产品，虽然RDBMS也具有缓存结构，但是始终在app层面不是我们想要的那么操控的。

4.replication
**redis提供主从复制方案**，跟mysql一样增量复制而且复制的实现都很相似，这个复制跟AOF有点类似复制的是新增记录命令，主库新增记录将新增脚本发送给从库，从库根据脚本生成记录，这个过程非常快，就看网络了，**一般主从都是在同一个局域网**，所以可以说redis的主从近似及时同步，**同事它还支持一主多从，动态添加从库，从库数量没有限制。 主从库搭建，我觉得还是采用网状模式**，如果使用链式（master-slave-slave-slave-slave·····）如果第一个slave出现宕机重启，首先从master 接收 数据恢复脚本，这个是阻塞的，如果主库数据几TB的情况恢复过程得花上一段时间，在这个过程中其他的slave就无法和主库同步了。


##  Memcache与Ehcache的对比与适用范围 
本地缓存与远程缓存
根据缓存和应用的耦合程度将其划分为Local Cache和Remote Cache。

Local Cache是指包含在应用之中的缓存组件，如Ehcache, Oscache.
Remote Cache指和应用解耦，在应用之外的缓存组件，如Memcached

**Local Cache最大的优点是应用和Cache在同一进程内部，请求缓存非常快速，完全不需要网络开销，**所以单应用，不需要集群，或者集群时Cache node不需要互相通知的情况下使用比较合适。
缺点：多个应用程序无法直接共享缓存，应用集群的情况下这个问题就更加明显（好像是废话= =）

Cache的类别：
本地缓存：从最简单的Map到Ehcache单机版都属于一类。
分布式缓存：分布在不同JVM的Cache可以互相同步与备份，如JBossCache和Oracle那个天价的产品。
集中式缓存：最著名的代表是Memcache，Terracotta其实也属于透明的集中式架构。
建议Ehcache单机使用，因为Terracotta收购后在分布式缓存中必然侧重于TC，JGroup等广播通知方式已停止发展。
 
Memcached是一种集中式Cache，支持分布式横向扩展。
集中式架构：
1.节点均衡的网状（JBoss Tree Cache），**利用JGroup的多广播通信机制来同步数据。**
2.Maste-Slaves模式（分布式文件系统），由Master来管理slave，如何选择slave，如何迁移数据，都是由Master来完成，但是Master本身也存在单点问题。
应用点：
小对象的缓存（用户的token，权限信息，资源信息），小的静态资源缓存，SQL结果的缓存。
**` 应对高并发访问的应用，本地缓存采用EHCache,外部共享缓存采用Memcached,两者集成结合使用 `**

##  自定义cache接口实现与缓存框架解耦 
由于缓存系统有很多，如memcached，OSCache，Ehcache，JbossCache等，其中 OSCache，Ehcache，JbossCache是用java实现的开源的缓存框架，一个大型项目功能模块多，通常都集成了多个缓存系统。每套缓存框架都有各自的优缺点，随着项目的不断发展，更换缓存系统也是很有可能的，所以跟缓存系统的解耦是非常有必要的。

跟缓存进行解耦只需两个工作：
1. 自定义缓存接口
2. 实现各个缓存框架的adapter（适配器模式）。

下面给出一个示意图：
![](/data/dokuwiki/javaweb/pasted/20151214-173258.png)

引用缓存策略
使用spring AOP 抽出一个**专门缓存切面**，切面主要做两件事：
一、拦截类中定义的切入点；
二、进入切入点后，先从相应缓存中取key对应的value,如果取到值，就不执行切入点对应的方法；如果取不到值，则继续执行原来方法。
 key：包名类名方法名参数
value:该方法的返回值

memcache+ehcache结合示例项目：https://github.com/yangjiandong/pm

##  Java两级缓存框架 J2Cache 
J2Cache 是 OSChina 目前正在使用的两级缓存框架。第一级缓存使用 Ehcache，第二级缓存使用 Redis 。由于大量的缓存读取会导致 L2 的网络成为整个系统的瓶颈，因此 L1 的目标是降低对 L2 的读取次数。该缓存框架主要用于集群环境中。单机也可使用，用于避免应用重启导致的 Ehcache 缓存数据丢失。
J2Cache 使用 JGroups 进行组播通讯。
J2Cache 介绍 PPT：http://www.oschina.net/doc/652
Maven:
```

<dependency>
  <groupId>net.oschina.j2cache</groupId>
  <artifactId>j2cache-core</artifactId>
  <version>1.3.0</version>
</dependency>

```
示例代码：
```

CacheChannel cache = J2Cache.getChannel();
cache.set("cache1","key1","OSChina.net");
cache.get("cache1","key1");

```
测试方法：
安装 Redis
修改 core/Java/j2cache.properties  配置使用已安装的 Redis 服务器
执行 build.sh 进行项目编译
运行多个 runtest.sh
直接在 runtest 输入多个命令进行测试
依赖项目：
Ehcache
Redis
JGroups
视频介绍：http://v.youku.com/v_show/id_XNzAzMTY5MjUy.html

参考：http://m.blog.csdn.net/blog/jationxiaozi/8509732
http://blog.163.com/luowei505050@126/blog/static/11990720620128682726877/
http://m.oschina.net/p/j2cache