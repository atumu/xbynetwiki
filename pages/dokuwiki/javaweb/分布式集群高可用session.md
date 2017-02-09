title: 分布式集群高可用session 

#  分布式集群系统下的高可用session解决方案 
目前，为了使web能适应大规模的访问,需要实现应用的集群部署. 而` 实现集群部署首先要解决session的统一，即需要实现session的共享机制。 `
目前，在集群系统下实现session统一的有如下几种方案：
(1) 应用服务器间的session复制共享（如tomcat session共享）
(2) 基于cache DB缓存的session共享

##  应用服务器间的session复制共享 
session复制共享，主要是指集群环境下，多台应用服务器之间同步session，使session保持一致，对外透明。 如果其中一台服务器发生故障，根据负载均衡的原理，web服务器（apache/nginx）会遍历寻找可用节点，分发请求，由于session已同步， 故能保证用户的session信息不会丢失。
 
**此方案的不足之处：**
技术复杂,必须在同一种中间件之间完成(如:tomcat-tomcat之间).
session复制带来的性能损失会快速增加.特别是当session中保存了较大的对象,而且对象变化较快时, 性能下降更加显著. 这种特性使得web应用的水平扩展受到了限制。
Session内容序列化（serialize），会消耗系统性能。
Session内容通过广播同步给成员，会造成网络流量瓶颈，即便是内网瓶颈。  

##   基于cache DB缓存的session共享 
即使用cacheDB存取session信息，应用服务器接受新请求将session信息保存在cache DB中，当应用服务器发生故障时，web服务器（apache/nginx）会遍历寻找可用节点，分发请求，**当应用服务器发现session不在本机内存 时，则去cache DB中查找，如果找到则复制到本机，这样实现session共享和高可用。**

###  Memcached Session共享方案 
目前有` 开源 `的msm用于解决tomcat之间的session共享：Memcached_Session_Manager（MSM）
http://code.google.com/p/memcached-session-manager/
一个高可用的Tomcat session共享解决方案，除了可以从本机内存快速读取Session信息(仅针对黏性Session)外，同时可使用memcached存取Session，以实现高可用。
特性
  * 支持Tomcat6、Tomcat7支持黏性、非黏性Session
  * 无单一故障点
  * 可处理tomcat故障转移
  * 可处理memcached故障转移
  * 插件式session序列化
  * 允许异步保存session，以提升响应速度
  * 只有当session有修改时，才会将session写回memcached
  * JMX管理&监控

该方案的不足之处：
  * memcache支持的数据结构比较单一
  * memcache的内存必须足够大，否则会出现用户session从Cache中被清除
  * 需要定期的刷新缓存
  * 服务器故障时，存在于内存的memcache数据将会丢失
  * 为了解决基于memcache中存在的不足，故提出了下面的一种解决方案：

###  基于redis缓存的session共享 
结合上面的 MSM 思想，由 redis负责 session 数据的存储，而我们自己实现的 session manager 将负责 session 生命周期的管理。
一般的系统架构:
![](/data/dokuwiki/javaweb/pasted/20151031-104657.png)
**此架构存在着当redis master故障时, 虽然可以有一到多个备用slave，但是redis不会主动的进行master切换，这时session服务中断。**
为了做到redis的高可用，引入了` zookper或者haproxy或者keepalived `来解决**redis master slave的切换问题**。即：
![](/data/dokuwiki/javaweb/pasted/20151031-104754.png)
此体系结构中, redis master出现故障时,** 通过haproxy设置redis slave为临时master, redis master重新恢复后,**
再切换回去.** 此方案中, redis-master 与redis-slave 是双向同步的, 解决目前redis单点问题. 这样保证了session信息**
在redis中的高可用。
实现此方案：
nginx        1   192.168.1.102
tomcat1    1  
tomcat2    1
redis-master   1 
redis-slave      1
slave1     1
slave2     1
haproxy vip  1
参考：http://blog.csdn.net/lishehe/article/details/45223477