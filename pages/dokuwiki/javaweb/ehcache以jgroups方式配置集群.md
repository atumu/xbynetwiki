title: ehcache以jgroups方式配置集群 

#  ehcache以JGroups方式配置集群 
由于 EhCache 是进程中的缓存系统，一旦将应用部署在集群环境中，每一个节点维护各自的缓存数据，当某个节点对缓存数据进行更新，这些更新的数据无法在其它节点中共享，这不仅会降低节点运行的效率，而且会导致数据不同步的情况发生。例如某个网站采用 A、B 两个节点作为集群部署，当 A 节点的缓存更新后，而 B 节点缓存尚未更新就可能出现用户在浏览页面的时候，一会是更新后的数据，一会是尚未更新的数据，尽管我们也可以通过 Session Sticky 技术来将用户锁定在某个节点上，但对于一些交互性比较强或者是非 Web 方式的系统来说，Session Sticky 显然不太适合。所以就需要用到 **EhCache 的集群解决方案。**
EhCache 从 1.7 版本开始，支持五种集群方案，分别是：
  * Terracotta
  * RMI
  * JMS
  * JGroups
  * EhCache Server
```

<dependency>
    <groupId>org.jgroups</groupId>
    <artifactId>jgroups</artifactId>
    <version>3.6.6.Final</version>
</dependency>
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache-jgroupsreplication</artifactId>
    <version>1.7</version>
</dependency>
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.1</version>
</dependency>

```
##  JGroups 集群模式 
在服务器开发中，经常要同步几台数据库服务器的数据，同步数据要好几种，比如说在linux中，经常用rsync来同步数据或者写几个 Servlet来同步服务器数据。
JGroups是一个可靠的群组通讯Java工具包。它基于IP组播(IP multicast)，但在可靠性，组成员管理上对它作了扩展。
**JGroups 适合使用场合---服务器集群cluster、多服务器通讯、服务器replication(复制)等，分布式cache缓存。**

JGroups的可靠性体现在：
1，对所有接收者的消息的无丢失传输（通过丢失消息的重发）
2，大消息的分割传输和重组
3，消息的顺序发送和接收
4，原子性：消息要么被所有接收者接收，要么全不

JavaGroups的成员关系管理体现在：
1，可以知道组内有哪些成员
2，成员的加入，离开，掉线等的通知

JavaGroups的主要功能特征：
- 组的创建与删除。组成员能在LAN或WAN环境内互相发送消息
- 组的成员加入或离开
- 组成员的检测和通知：加入，离开，掉线
- 检测与移除已掉线的成员
- 消息的组播 (member-to-group或point-to-multipoint)
- 消息的点对点发送 (member-to-member或point-to-point)
- 支持UDP (IP Multicast), TCP, JMS等传输协议
- 免费开放源代码
JGroups是一个可靠的组间通讯工具，进程可以加入一个通讯组，给组内所有的成员或单独的成员发送消息，同样，也可以从组中的成员处接收消息。 系统会记录组的每一个成员，在新成员加入或是现有的成员离开或是崩溃时，会通知组内的其他成员，这样我们就不必自己去管理这些事情，要想加入一个组，并与 组内其他的成员交互，**必须建立一个Channel连接到组**，**同一个组内的所有成员使用相同的组名称，这样才能进行通信。**

EhCache 从 1.5. 版本开始增加了 JGroups 的分布式集群模式。与 RMI 方式相比较， 
JGroups 提供了一个非常灵活的协议栈、可靠的单播和**多播消息传输**，主要的缺点是配置复杂以及一些协议栈对第三方包的依赖。
JGroups 也提供了基于 TCP 的单播 ( Unicast ) 和**基于 UDP 的多播 ( Multicast )** ，对应 RMI 的手工配置和` 自动发现 `。使用单播方式需要指定其它节点的主机地址和端口，
下面是两个节点，并使用了**单播方式的配置**：
```

<cacheManagerPeerProviderFactory
    class="net.sf.ehcache.distribution.jgroups.JGroupsCacheManagerPeerProviderFactory"
    properties="connect=TCP(start_port=7800):
        TCPPING(initial_hosts=host1[7800],host2[7800];port_range=10;timeout=3000;
        num_initial_members=3;up_thread=true;down_thread=true):
        VERIFY_SUSPECT(timeout=1500;down_thread=false;up_thread=false):
        pbcast.NAKACK(down_thread=true;up_thread=true;gc_lag=100;
	retransmit_timeout=3000):
        pbcast.GMS(join_timeout=5000;join_retry_timeout=2000;shun=false;
        print_local_addr=false;down_thread=true;up_thread=true)"
propertySeparator="::" />

```
          
` **使用多播方式配置如下：** `
```

<cacheManagerPeerProviderFactory
    class="net.sf.ehcache.distribution.jgroups.JGroupsCacheManagerPeerProviderFactory"
    properties="connect=UDP(mcast_addr=231.12.21.132;mcast_port=45566;):PING:
    MERGE2:FD_SOCK:VERIFY_SUSPECT:pbcast.NAKACK:UNICAST:pbcast.STABLE:FRAG:pbcast.GMS"
    propertySeparator="::"
/>

```
从上面的配置来看，JGroups 的配置要比 RMI 复杂得多，但也提供更多的微调参数，有助于提升缓存数据复制的性能。详细的 JGroups 配置参数的具体意义可参考 JGroups 的配置手册。

配置jgroups udp 集群.
` **JGroups 方式对应缓存节点的配置信息如下：** `
```

<cache name="sampleCache2"
    maxElementsInMemory="10"
    eternal="false"
    timeToIdleSeconds="100"
    timeToLiveSeconds="100"
    overflowToDisk="false">
    <cacheEventListenerFactory
        class="net.sf.ehcache.distribution.jgroups.JGroupsCacheReplicatorFactory"
        properties="replicateAsynchronously=true, replicatePuts=true,
        replicateUpdates=true, replicateUpdatesViaCopy=false, replicateRemovals=true" />
</cache>

```
cacheEventListenerFactory的配置参数: 
replicatePuts=true| false - whether new elements placed in a cache are replicated to others.Defaults to true.
新元素的添加是否会被复制到其他机器上.
replicateUpdates=true| false - whether new elements which override an element already existing withthe same key are replicated. Defaults to true.
元素的更新是否会被复制到其他机器上.
replicateRemovals=true- whether element removals are replicated. Defaults to true.
元素的移除是否会被更新到到其他机器上.
replicateAsynchronously=true| false - whether replications are asyncrhonous (true) or synchronous (false).Defaults to true.
复制操作是异步还是同步的.默认是异步的.
replicateUpdatesViaCopy=true| false - whether the new elements are copied to other caches (true), orwhether a remove message is sent. Defaults to true.
元素的复制消息或者删除消息是否会被同步.默认是true.
asynchronousReplicationIntervalMillisdefault 1000ms Time between updates when replication is asynchroneous
asynchronousReplicationIntervalMillis所有的更新操作是异步的时候,会在1秒内同步完成.

**使用组播方式的注意事项**
使用 JGroups 需要引入 **JGroups 的 Jar 包**以及 EhCache 对 JGroups 的封装包 **ehcache-jgroupsreplication-xxx.jar** 。
在一些启用了 IPv6 的电脑中，经常启动的时候报如下错误信息：
java.lang.RuntimeException: the type of the stack (IPv6) and the user supplied addresses (IPv4) don't match: /231.12.21.132.
**解决的办法是增加 JVM 参数：-Djava.net.preferIPv4Stack=true。如果是 Tomcat 服务器，可在 catalina.bat 或者 catalina.sh 中增加如下环境变量即可：
 SET CATALINA_OPTS=-Djava.net.preferIPv4Stack=true**
经过实际测试发现，集群方式下的缓存数据都可以在 1 秒钟之内完成到其节点的复制。

云服务器上，端口的权限默认没有开放，**你可以用jgroups自带测试程序看看是否能通讯，**先在一台服务器上启动McastReceiverTest
java org.jgroups.tests.McastReceiverTest -mcast_addr 235.6.6.6 -port 45588
然后去另外一个服务器上启动McastSenderTest
java org.jgroups.tests.McastSenderTest -mcast_addr 235.6.6.6 -port 45588
如果在sender端发消息在receiver端能收到，说明是能够通讯的

##  EhCache Server 
与前面介绍的两种集群方案不同的是， **EhCache Server 是一个独立的缓存服务器，其内部使用 EhCache 做为缓存系统**，可利用前面提到的两种方式进行内部集群。**对外提供编程语言无关的基于 HTTP 的 RESTful 或者是 SOAP 的数据缓存操作接口。**
下面是 EhCache Server 提供的对缓存数据进行操作的方法：
**OPTIONS /{cache}}
获取某个缓存的可用操作的信息。**
HEAD /{cache}/{element}
获取缓存中某个元素的 HTTP 头信息，例如：
curl --head  http://localhost:8080/ehcache/rest/sampleCache2/2
EhCache Server 返回的信息如下：
HTTP/1.1 200 OK 
X-Powered-By: Servlet/2.5 
Server: GlassFish/v3 
Last-Modified: Sun, 27 Jul 2008 08:08:49 GMT 
ETag: "1217146129490"
Content-Type: text/plain; charset=iso-8859-1 
Content-Length: 157 
Date: Sun, 27 Jul 2008 08:17:09 GMT
**GET /{cache}/{element}
读取缓存中某个数据的值。
PUT /{cache}/{element}
写缓存。**
**由于这些操作都是基于 HTTP 协议的，因此你可以在任何一种编程语言中使用它，例如 Perl、PHP 和 Ruby 等等。**
下图是 EhCache Server 在应用中的架构：
![](/data/dokuwiki/javaweb/pasted/20151216-230256.png)
EhCache Server 同时也提供强大的安全机制、监控功能。在数据存储方面，最大的 Ehcache 单实例在内存中可以缓存 20GB。最大的磁盘可以缓存 100GB。通过将节点整合在一起，这样缓存数据就可以跨越节点，以此获得更大的容量。将缓存 20GB 的 50 个节点整合在一起就是 1TB 了。

总结
以上我们介绍了三种 EhCache 的集群方案，除了第三种跨编程语言的方案外，EhCache 的集群对应用程序的代码编写都是透明的，程序人员无需考虑缓存数据是如何复制到其它节点上。既保持了代码的轻量级，同时又支持庞大的数据集群。EhCache 可谓是深入人心。
2009 年年中，Terracotta 宣布收购 EhCache 产品。Terracotta 公司的产品 Terracotta 是一个 JVM 级的开源群集框架，提供 HTTP Session 复制、分布式缓存、POJO 群集、跨越集群的 JVM 来实现分布式应用程序协调。最近 EhCache 主要的改进都集中在跟 Terracotta 框架的集成上，这是一个真正意义上的企业级缓存解决方案。
参考：
http://www.ehcache.org/documentation/2.8/replication/jgroups-replicated-caching
http://www.oschina.net/question/1258821_173625
https://www.ibm.com/developerworks/cn/java/j-lo-ehcache/
http://blog.csdn.net/w329636271/article/details/46664063