title: 分布式中使用_redis_实现_session_共享 

#  分布式中使用 Redis 实现 Session 共享 
参考：http://blog.jobbole.com/91870/
本文着眼于解决负载均衡集群下Session共享问题：
##  Redis安装配置与简单使用 
` redis `是一个**key-value存储系统**。` 和Memcached类似 `，它支持存储的value类型相对更多，` 包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型） `。
这些数据类型都` 支持push/pop、add/remove及取交集并集和差集及更丰富的操作 `，而且**这些操作都是原子性的**。在此基础上，**redis支持各种不同方式的排序**。
与memcached一样，为了保证效率，` 数据都是缓存在内存中 `。区别的是**redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件**，并且在此基础上实现了**master-slave(主从)同步**。

最新版本的redis版本为3.0.3，支持集群功能。我这下载的是window版本的，实际场景都是安装在linux系统下的。下载地址：redis-2.8.19.rar 。更多下载地址：　
官网 ：http://redis.io/download  MSOpenTech：https://github.com/MSOpenTech/redis  dmajkic：https://github.com/dmajkic/redis/downloads
 下载完成之后解压运行redis-server.exe就启动了redis了，启动后会在进程里面看到reids。
![](/data/dokuwiki/javaweb/pasted/20150925-051333.png)

**1.读写分离配置**
redis的读写分离需要修改配置文件，把解压的文件复制了一份。两份文件是一样的，分别命名为MasterRedis-2.8.19（**主redis服务**）,SlaveRedis-2.8.19（**从redis服务**）。
redis默认绑定的是6379端口,我们保持主服务配置不变，修改从服务配置。
![](/data/dokuwiki/javaweb/pasted/20150925-051444.png)
修改从服务绑定端口（修改时可以直接搜索port关键字）
![](/data/dokuwiki/javaweb/pasted/20150925-051459.png)
修改从服务对应的主服务地址（修改时可以直接搜索slaveof关键字）
![](/data/dokuwiki/javaweb/pasted/20150925-051515.png)
配置文件修改完成以后，**分别启动主服务和从服务**
从服务启动以后，主服务会发送一条同步的sync命令，同步从服务器的缓存数据。
###  五种数据类型使用 
服务搭建好以后可以使用.net版本redis操作类库ServiceStack.Redis来操作redis，本文会用到以下三个dll。
初始化RedisClient对象
var client = new RedisClient("120.26.197.185", 6379);
**1.String**
String是最常用的一种数据类型，普通的key/value存储都可以归为此类，value其实不仅是String，也可以是数字：比如想知道什么时候封锁一个IP地址(访问超过几次)。INCRBY命令让这些变得很容易，通过原子递增保持计数。  
```

#region "字符串类型"
client.Set<string>("name", "laowang");
string userName = client.Get<string>("name");
Console.WriteLine(userName);

//访问次数
client.Set<int>("IpAccessCount", 0);
//次数递增
client.Incr("IpAccessCount");
Console.WriteLine(client.Get<int>("IpAccessCount"));
#endregion

```
其他略。。。。

###  连接池的初始化 
```

        private static string[] ReadWriteHosts = System.Configuration.ConfigurationSettings.AppSettings["readWriteHosts"].Split(new char[] { ';' });
        private static string[] ReadOnlyHosts = System.Configuration.ConfigurationSettings.AppSettings["readOnlyHosts"].Split(new char[] { ';' });

        #region -- 连接信息 --
        public static PooledRedisClientManager prcm = CreateManager(ReadWriteHosts, ReadOnlyHosts);

        private static PooledRedisClientManager CreateManager(string[] readWriteHosts, string[] readOnlyHosts)
        {
            // 支持读写分离，均衡负载  
            return new PooledRedisClientManager(readWriteHosts, readOnlyHosts, new RedisClientManagerConfig
            {
                MaxWritePoolSize = 5, // “写”链接池链接数  
                MaxReadPoolSize = 5, // “读”链接池链接数  
                AutoStart = true,
            });
        }

``` 
总结
1.其实php,java等多种语言都能使用redis,在我接触的项目中见到使用**redis做为消息队列和缓存组件**，当然它的功能远不止于此。后面的文章将详细介绍redis的几个使用案例。
2.可以使用redis desktop manager管理工具查看服务器缓存中的数据

##  redis常用配置项为: 

```

daemonize: 是否以后台进程运行，默认为no
pidfile /var/run/redis.pid: pid文件路径
port 6379: 监听端口
bind 127.0.0.1:绑定主机ip
unixsocket /tmp/redis.sock：sock文件路径
timeout 300：超时时间，默认是300s
loglevel verbose：日志等级，可选项有debug:大量的信息，开发和测试有用；verbose：很多极其有用的信息，但是不像debug那么乱；notice：在生产环境中你想用的信息；warning：最关键、最重要的信息才打印。 默认是erbose
logfile stdout：日志记录方式，默认是stdout
syslog-enabled no：日志记录到系统日志中，默认是no
syslog-ident redis：指定系统日志标识
syslog-facility local0：指定系统日志设备，必须是USER或者 LOCAL0~LOCAL7。 默认是local0
databases 16：数据库的数量，默认的数据库是DB 0，你可以使用 SELECT 来选择不同的数据库。dbid的范围是0~(你设置的值-1)

save <seconds> <changes>：RDB在多长时间内，有多少次更新操作，就将数据同步到数据文件。
save 900 1：15min内至少1个key被改变
save 300 10：5min内至少有300个key被改变
save 60 10000：60s内至少有10000个key被改变
rdbcompression yes：存储至本地数据库时是否压缩数据，默认是yes
dbfilename dump.rdb：本地数据库文件名，默认是dump.rdb
dir ./：本地数据库存放路径，默认是./
slaveof <masterip> <masterport>：当本机为从服务时，设置主服务的ip以及端口
masterauth <master-password>：主服务的连接密码
从结点与主结点失去连接、或者正在复制时，从结点对客户端请求的处理方式：
slave-serve-stale-data yes：yes：从结点继续响应客户端的请求，但是数据有可能不准确或者为空 no：除了INFO和SLAVEOF以外，其它的命令都返回“SYNC with master in progress”

requirepass foobared：连接密码foobared
maxclients 128：最大连接数，默认不限制
maxmemory <bytes>：设置最大内存，达到最大内存设置后，redis会先尝试清除已到期或即将到期的key,当此方法处理后，任然到达最大内存设置，将无法再进行写入操作

下面是maxmemory的策略
maxmemory-policy volatile-lru：maxmemory设置策略，默认是volatile-lru
volatile-lru：使用LRU算法，从过期集中移除
allkeys-lru：根据LRU算法移除key
volatile-random：从过期集中随机移动一个
allkeys-random：随机移除一个
volatile-ttl： 根据最近过期时间移除key
noeviction：不移除数据,客户端写操作时返回错误 don’t expire at all, just return an error on write operations

maxmemory-samples 3

appendonly no：是否 在每次更新操作后进行日志记录，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为redis本身同步数据文件是按照上面save条件来进行同步的，所以有的数据会在一段时间内只存在于内存中。默认是no
appendfilename appendonly.aof：更新日志文件名，默认是appendonly.aof
redis支持的三种不同的同步方式:
no: don’t fsync, just let the OS flush the data when it wants. Faster. //等待OS进行数据缓存同步到硬盘
always: fsync after every write to the append only log . Slow, Safest. //每次更新操作后调用fsync()将数据写到磁盘
everysec: fsync only if one second passed since the last fsync. Compromise. //每秒同步一次
appendfsync everysec //更新日志条件，默认是everysec
no-appendfsync-on-rewrite no

slowlog-log-slower-than 10000：设置redis slow log时间，只包括命令执行时间，不包括IO操作时间，比如客户端连接，应答相应时间等等。单位是microseconds(一百万分之一秒)，默认是10000.负值表示禁用slow log,0表示记录所有命令。
slowlog-max-len 1024：slowlog最大长度1024.这会消耗内存，使用SLOWLOG RESET来回收slowlog内存。

vm-enabled no //是否使用虚拟内存，默认是no。在redis2.4版本，强烈不建议使用virtual memory。
vm-swap-file /tmp/redis.swap //虚拟内存文件路径，默认是/tmp/redis.swap，不可多个redis实例共享虚拟内存文件。
vm-max-memory 0 //设置最大vm，默认为0，所有的value存在于磁盘中。
vm-page-size 32 //设置vm的page大小，默认是32
vm-pages 134217728 //设置swap文件中最大memory pages，默认是134217728。swap大小=vm-page-size * vm-pages
vm-max-threads 4 //vm同时运行的最大io线程

指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法：
hash-max-zipmap-entries 512
hash-max-zipmap-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
activerehashing yes //是否重置hash表

include /path/to/other.conf：引用其他配置文件

```
##  实现Session共享方案 
redis实现session共享
建议使用 redis ，不仅仅因为它可以将缓存的内容持久化，还因为它支持的单个对象比较大，而且数据类型丰富， 不只是缓存 session ，还可以做其他用途，一举几得啊。
使用 Redis 服务器来存储Session非常有优势。首先它是一个NOSQL数据，第二它很容易扩展使用。
下面这种安装方式非常清晰明白的引导你把Redis缓存作为一个Session的存储系统。步骤如下：
wget http://download.redis.io/redis-stable.tar.gz 
```

tar xvzf redis-stable.tar.gz 
cd redis-stable 
make
编译完成后,会产生六个文件:
redis-server：这个是redis的服务器
redis-cli：这个是redis的客户端
redis-check-aof：这个是检查AOF文件的工具
redis-check-dump：这个是本地数据检查工具
redis-benchmark：性能基准测试工具，安装完后可以测试一下当前Redis的性能
redis-sentinel：Redis监控工具，集群管理工具
Redis的配置文件是：redis.conf
  
cd RedisDirectory/src
./redis-server --port 6379

```
3. 下载最新的Tomcat 7
**4. 下载最新的Jedis（一个Redis 的Java客户端），Tomcat Redis Session Manager 和 Apache Commons Pool**
**各个组件的下载地址：**
  * Redis：http://redis.io/
  * JRedis: https://github.com/xetorthio/jedis
  * Tomcat Redis Session Manager ：https://github.com/jcoleman/tomcat-redis-session-manager/downloads
  * Apache Commons Pool ：http://commons.apache.org/proper/commons-pool/download_pool.cgi
以 1.2 版为例子，需要用的 jar 包： 
  * tomcat-redis-session-manager-1.2-tomcat-6.jar 
  * jedis-2.1.0.jar 
  * commons-pool-1.6.jar 
**5. 将上面所有的Jar包都拷到Tomcat7安装目录下面的Lib目录下**
6. 在Tomcat 的` conf/context.xml ` 文件里增加如下内容（**` 或者在server.xml的context块中添加 `**）：
不同的是memcached只需要添加一个manager标签，而redis需要增加的内容如下：（注意：` valve标签一定要在manager前面 `。）
` 如果tomcat配置中，将manager放在server.xml中，那么使用maven做热部署时，会发生失败。所以，推荐放在context.xml中。 `
```

<Valve className="com.radiadesign.catalina.session.RedisSessionHandlerValve" />
<Manager className="com.radiadesign.catalina.session.RedisSessionManager"
                   host="localhost" <!-- 可选，默认是"localhost" -->
                   port="6379" <!-- 可选，默认是 "6379" -->
                   database="0" <!-- 可选，默认是 "0" -->
                   maxInactiveInterval="60" <!-- 可选，默认是 "60" （单位：秒）--> />

```
7. 重启Tomcat7，你现你可以看到，Session的内容开始在Redis中创建了。
现在，Tomcat7的Session就保存到Redis中了，而且它也维护着Session的不同方面。



参考：http://my.oschina.net/gccr/blog/321083
http://www.cnblogs.com/interdrp/p/4056525.html