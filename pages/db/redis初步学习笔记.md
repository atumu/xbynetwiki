modifyAt:2016-12-11 14:20:54
location:db/redis初步学习笔记
title:redis初步学习笔记
author:xbynet
createAt:2016-12-11 14:20:54

Redis中文官方网站：http://www.redis.cn/documentation.html
本文主要参考：http://www.jb51.net/article/56448.htm
http://www.redis.net.cn/tutorial/3501.html

**Redis**是一个开源（BSD许可），内存存储的数据结构服务器，**可用作数据库，高速缓存和消息队列代理**。它支持**字符串、哈希表、列表、集合、有序集合，位图，hyperloglogs**等数据类型。内置**复制**、Lua脚本、**LRU**收回、**事务**以及不同级别**磁盘持久化**功能，同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动分区。

Redis程序文件说明:
./redis-benchmark //用于进行redis性能测试的工具
./redis-check-dump //用于修复出问题的dump.rdb文件
./redis-cli //redis的客户端
./redis-server //redis的服务端
./redis-check-aof //用于修复出问题的AOF文件
./redis-sentinel //用于集群管理

默认情况下，redis-server会以**非daemon**的方式来运行，且默认服务端口为**6379**。
【Redis 连接】
Redis 连接命令主要是用于连接 redis 服务。
以下实例演示了客户端如何通过密码验证连接到 redis 服务，并检测服务是否在运行：
redis 127.0.0.1:6379> AUTH "password"
OK
redis 127.0.0.1:6379> PING
PONG
**下表列出了 redis 连接的基本命令：**
1	AUTH password 验证密码是否正确
2	ECHO message 打印字符串
3	PING 查看服务是否运行
4	QUIT 关闭当前连接
5	SELECT index 切换到指定的数据库

# redis数据结构 
redis是一种高级的key:value存储系统.
Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。
关于key.在一个项目中，key最好使用统一的命名模式，例如user:10000:passwd。

【redis数据结构 – string】
```
set mystr "hello world!" //设置字符串类型
get mystr //读取字符串类型
```
string是redis最基本的类型,一个key对应一个value,一个键最大能存储512MB。
string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
另外，我们还可以通过字符串类型进行`数值自增自减操作`（在遇到数值操作时，redis会将字符串类型转换成数值。）：
```
127.0.0.1:6379> set mynum "2"
OK
127.0.0.1:6379> get mynum
"2"
127.0.0.1:6379> incr mynum
(integer) 3
127.0.0.1:6379> get mynum
"3"
```
由于INCR等指令本身就具有`原子操作的特性`，所以我们完全可以利用redis的**INCR、INCRBY、DECR、DECRBY**等指令来实现**原子计数**的效果。

【redis数据结构 – list】
redis的另一个重要的数据结构叫做lists。
首先要明确一点，redis中的lists在底层实现上并不是数组，而是链表。
lists的常用操作包括LPUSH、RPUSH、LRANGE等。我们可以用LPUSH在lists的左侧插入一个新元素，用RPUSH在lists的右侧插入一个新元素，用**LRANGE命令从lists中指定一个范围来提取元素**。
```
/新建一个list叫做mylist，并在列表头部插入元素"1"
127.0.0.1:6379> lpush mylist "1" 
//返回当前mylist中的元素个数
(integer) 1 
//在mylist右侧插入元素"2"
127.0.0.1:6379> rpush mylist "2" 
(integer) 2
//在mylist左侧插入元素"0"
127.0.0.1:6379> lpush mylist "0" 
(integer) 3
//列出mylist中从编号0到编号1的元素
127.0.0.1:6379> lrange mylist 0 1 
1) "0"
2) "1"
//列出mylist中从编号0到倒数第一个元素,即所有元素
127.0.0.1:6379> lrange mylist 0 -1 
1) "0"
2) "1"
3) "2"
```

lists的应用相当广泛，随便举几个例子：
1.我们可以**利用lists来实现一个消息队列**，而且可以确保先后顺序。
2.利用**LRANGE还可以很方便的实现分页**的功能。
3.在博客系统中，每片博文的评论也可以存入一个单独的list中。

【redis数据结构 – 集合】
redis的集合，是一种无序的集合，集合中的元素没有先后顺序，**没有重复**。
集合相关的操作也很丰富，如添加新元素、删除已有元素、取交集、取并集、取差集等。我们来看例子：
```
//向集合myset中加入一个新元素"one"
127.0.0.1:6379> sadd myset "one" 
(integer) 1
127.0.0.1:6379> sadd myset "two"
(integer) 1
//列出集合myset中的所有元素
127.0.0.1:6379> smembers myset 
1) "one"
2) "two"
//判断元素1是否在集合myset中，返回1表示存在
127.0.0.1:6379> sismember myset "one" 
(integer) 1
//判断元素3是否在集合myset中，返回0表示不存在
127.0.0.1:6379> sismember myset "three" 
(integer) 0
127.0.0.1:6379> smembers yourset
1) "1"
2) "2"
//对两个集合求并集
127.0.0.1:6379> sunion myset yourset 
1) "1"
2) "one"
3) "2"
4) "two"
```
对于集合的使用，也有一些常见的方式，比如，QQ有一个社交功能叫做“**好友标签**”，大家可以给你的好友贴标签，比如“大美女”、“土豪”、“欧巴”等等，这时就可以使用redis的集合来实现，把每一个用户的标签都存储在一个集合之中。

【redis数据结构 – 有序集合】
有序集合（sorted sets)，叫做**zset**：每个元素都关联一个分数（score），这便是排序的依据。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的**成员**是唯一的,但**分数**(score)却可以重复。
```
//新增一个有序集合myzset，并加入一个元素baidu.com，给它赋予的序号是1：
127.0.0.1:6379> zadd myzset 1 baidu.com 
(integer) 1
//向myzset中新增一个元素360.com，赋予它的序号是3
127.0.0.1:6379> zadd myzset 3 360.com 
(integer) 1
//向myzset中新增一个元素google.com，赋予它的序号是2
127.0.0.1:6379> zadd myzset 2 google.com 
(integer) 1
//列出myzset的所有元素，同时列出其序号，可以看出myzset已经是有序的了。
127.0.0.1:6379> zrange myzset 0 -1 with scores 
1) "baidu.com"
2) "1"
3) "google.com"
4) "2"
5) "360.com"
6) "3"
//只列出myzset的元素
127.0.0.1:6379> zrange myzset 0 -1 
1) "baidu.com"
2) "google.com"
3) "360.com"
```

【redis数据结构 – 哈希】
Redis hash 是一个键值对集合。
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。比如一个用户要存储其全名、姓氏、年龄等等，就很适合使用哈希。
```
//建立哈希，并赋值HMSET key value(值是一系列空格分隔的 key value key value ...)
127.0.0.1:6379> HMSET user:001 username antirez password P1pp0 age 34 
OK
//列出哈希的内容
127.0.0.1:6379> HGETALL user:001 
1) "username"
2) "antirez"
3) "password"
4) "P1pp0"
5) "age"
6) "34"
//更改哈希中的某一个值
127.0.0.1:6379> HSET user:001 password 12345 
(integer) 0
//再次列出哈希的内容
127.0.0.1:6379> HGETALL user:001 
1) "username"
2) "antirez"
3) "password"
4) "12345"
5) "age"
6) "34"
```

# redis持久化RDB和AOF
http://www.slideshare.net/eugef/redis-persistence-in-practice-1
https://www.inovex.de/blog/redis-backup/

redis提供了两种持久化的方式，分别是**RDB（Redis DataBase）和AOF（Append Only File）**。
RDB，简而言之，就是在不同的时间点，将redis存储的数据生成快照并存储到磁盘等介质上；
AOF，是将redis执行过的所有写指令记录下来，在下次redis重新启动时，只要把这些写指令从前到后再重复执行一遍，就可以实现数据恢复了。
**其实RDB和AOF两种方式也可以同时使用**，在这种情况下，如果redis重启的话，则会优先采用AOF方式来进行数据恢复，这是因为AOF方式的数据恢复完整度更高。
如果你没有数据持久化的需求，也完全可以关闭RDB和AOF方式，这样的话，redis将变成一个纯内存数据库。
## redis持久化 – RDB
RDB方式，是将redis某一时刻的数据持久化到磁盘中，是一种**快照式的持久化方法**。
redis在进行数据持久化的过程中，会先将数据写入到一个临时文件中，待持久化过程都结束了，才会用这个临时文件替换上次持久化好的文件。正是这种特性，让我们可以随时来进行备份，因为快照文件总是完整可用的。
对于RDB方式，redis会单独创建（fork）一个子进程来进行持久化，而主进程是不会进行任何IO操作的，这样就确保了redis极高的性能。**但是会导致内存使用临时达到2倍+**
**如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。**
虽然RDB有不少优点，但它的缺点也是不容忽视的。如果你对数据的完整性非常敏感，那么RDB方式就不太适合你，因为即使你每5分钟都持久化一次，当redis故障时，仍然会有近5分钟的数据丢失。所以，redis还提供了另一种持久化方式，那就是AOF。
## redis持久化 – AOF
AOF，英文是Append Only File，即只允许追加不允许改写的文件。
如前面介绍的，AOF方式是将执行过的写指令记录下来，在数据恢复时按照从前到后的顺序再将指令都执行一遍，就这么简单。
我们通过配置redis.conf中的appendonly yes就可以打开AOF功能。如果有写操作（如SET等），redis就会被追加到AOF文件的末尾。
**默认的AOF持久化策略是每秒钟fsync一次**（fsync是指把缓存中的写指令记录到磁盘中），因为在这种情况下，redis仍然可以保持很好的处理性能，即使redis故障，也只会丢失最近1秒钟的数据。
如果在追加日志时，恰好遇到磁盘空间满、inode满或断电等情况导致日志写入不完整，也没有关系，redis提供了`redis-check-aof`工具，可以用来进行日志修复。
因为采用了追加方式，如果不做任何处理的话，AOF文件会变得越来越大，为此，redis提供了**AOF文件重写（rewrite）机制**，即当AOF文件的大小超过所设定的阈值时，redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。举个例子或许更形象，假如我们调用了100次INCR指令，在AOF文件中就要存储100条指令，但这明显是很低效的，完全可以把这100条指令合并成一条SET指令，这就是重写机制的原理。
在进行AOF重写时，仍然是采用先写临时文件，全部完成后再替换的流程，所以断电、磁盘满等问题都不会影响AOF文件的可用性，这点大家可以放心。
虽然优点多多，但AOF方式也同样存在缺陷，比如在同样数据规模的情况下，AOF文件要比RDB文件的体积大。而且，**AOF方式的恢复速度也要慢于RDB方式。**
如果你直接执行BGREWRITEAOF命令，那么redis会生成一个全新的AOF文件，其中便包括了可以恢复现有数据的最少的命令集。
如果运气比较差，AOF文件出现了被写坏的情况，也不必过分担忧，redis并不会贸然加载这个有问题的AOF文件，而是报错退出。**这时可以通过以下步骤来修复出错的文件：**
1.备份被写坏的AOF文件
2.运行redis-check-aof –fix进行修复
3.用diff -u来看下两个文件的差异，确认问题点
4.重启redis，加载修复后的AOF文件

【聊聊redis持久化 – AOF重写】
AOF重写的内部运行原理，我们有必要了解一下。
在重写即将开始之际，redis会创建（fork）一个“重写子进程”，这个子进程会首先读取现有的AOF文件，并将其包含的指令进行分析压缩并写入到一个临时文件中。
与此同时，主工作进程会将新接收到的写指令一边累积到内存缓冲区中，一边继续写入到原有的AOF文件中，这样做是保证原有的AOF文件的可用性，避免在重写过程中出现意外。
当“重写子进程”完成重写工作后，它会给父进程发一个信号，父进程收到信号后就会将内存中缓存的写指令追加到新AOF文件中。
当追加结束后，redis就会用新AOF文件来代替旧AOF文件，之后再有新的写指令，就都会追加到新的AOF文件中了。

【聊聊redis持久化 – 如何选择RDB和AOF】
对于我们应该选择RDB还是AOF，官方的建议是两个同时使用。这样可以提供更可靠的持久化方案。

# 主从 – 用法
像MySQL一样，redis是支持主从同步的，而且也支持一主多从以及多级从结构。
主从结构，一是为了纯粹的冗余备份，二是为了提升读性能，比如很消耗性能的SORT就可以由从服务器来承担。
redis的主从同步是异步进行的，这意味着主从同步不会影响主逻辑，也不会降低redis的处理性能。
主从架构中，可以考虑关闭主服务器的数据持久化功能，只让从服务器进行持久化，这样可以提高主服务器的处理性能。
在主从架构中，从服务器通常被设置为只读模式，这样可以避免从服务器的数据被误修改。

# redis的事务处理
先和大家介绍四个redis指令，即`MULTI、EXEC、DISCARD、WATCH`。这四个指令构成了redis事务处理的基础。
1.MULTI用来组装一个事务；
2.EXEC用来执行一个事务；
3.DISCARD用来取消一个事务；
4.WATCH用来监视一些key，一旦这些key在事务执行之前被改变，则取消事务的执行。
```
redis> MULTI //标记事务开始
OK
redis> INCR user_id //多条命令按顺序入队
QUEUED
redis> INCR user_id
QUEUED
redis> INCR user_id
QUEUED
redis> PING
QUEUED
redis> EXEC //执行
1) (integer) 1
2) (integer) 2
3) (integer) 3
4) PONG
```
**“WATCH”，这是一个很好用的指令，它可以帮我们实现类似于“乐观锁”的效果，即CAS（check and set）。**
**WATCH本身的作用是“监视key是否被改动过”，而且支持同时监视多个key，只要还没真正触发事务，WATCH都会尽职尽责的监视，一旦发现某个key被修改了，在执行EXEC时就会返回nil，表示事务无法触发。**
```
127.0.0.1:6379> set age 23
OK
127.0.0.1:6379> watch age //开始监视age
OK
127.0.0.1:6379> set age 24 //在EXEC之前，age的值被修改了
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 25
QUEUED
127.0.0.1:6379> get age
QUEUED
127.0.0.1:6379> exec //触发EXEC
(nil) //事务无法被执行
```

# Redis 发布订阅
http://www.redis.net.cn/tutorial/3514.html
Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。
Redis 客户端可以订阅任意数量的频道。
以下实例演示了发布订阅是如何工作的。在我们实例中我们创建了订阅频道名为 redisChat:
```
redis 127.0.0.1:6379> SUBSCRIBE redisChat
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
```
现在，我们先重新开启个 redis 客户端，然后在同一个频道 redisChat 发布两次消息，订阅者就能接收到消息。
```
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"
(integer) 1
redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by w3cschool.cc"
(integer) 1
```

订阅者的客户端会显示如下消息
```
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by w3cschool.cc"
```

**下表列出了 redis 发布订阅常用命令：**
1	PSUBSCRIBE pattern [pattern ...] 订阅一个或多个符合给定模式的频道。
2	PUBSUB subcommand [argument [argument ...]] 查看订阅与发布系统状态。
3	PUBLISH channel message 将信息发送到指定的频道。
4	PUNSUBSCRIBE [pattern [pattern ...]] 退订所有给定模式的频道。
5	SUBSCRIBE channel [channel ...] 订阅给定的一个或多个频道的信息。
6	UNSUBSCRIBE [channel [channel ...]] 指退订给定的频道。

# redis配置
redis配置文件被分成了几大块区域，它们分别是：
1.通用（general）
2.快照（snapshotting）
3.复制（replication）
4.安全（security）
5.限制（limits)
6.追加模式（append only mode)
7.LUA脚本（lua scripting)
8.慢日志（slow log)
9.事件通知（event notification）

常用配置：
**默认情况下，redis并不是以daemon形式来运行的。**通过daemonize配置项可以控制redis的运行形式，如果改为yes，那么redis就会以daemon形式运行：
daemonize no
redis允许你通过bind配置项来**指定要绑定的IP**，比如：
bind 192.168.1.2 10.8.4.2
redis的默认**服务端口**是6379。如果端口设置为0的话，redis便不会监听端口了。
port 6379
当一个redis-client一直没有请求发向server端，那么server端有权主动关闭这个连接，可以通过timeout来设置“空闲超时时限”，0表示永不关闭。
timeout 0

TCP连接保活策略，可以通过tcp-keepalive配置项来进行设置，单位为秒，假如设置为60秒，则server端会每60秒向连接空闲的客户端发起一次ACK请求，以检查客户端是否已经挂掉，对于无响应的客户端则会关闭其连接。所以关闭一个连接最长需要120秒的时间。如果设置为0，则不会进行保活检测。
tcp-keepalive 0

redis支持通过loglevel配置项设置日志等级，共分四级，即**debug、verbose、notice、warning**。
loglevel notice
redis也支持通过logfile配置项来设置**日志文件**的生成位置。如果设置为空字符串，则redis会将日志输出到标准输出。假如你在daemon情况下将日志设置为输出到标准输出，则日志会被写到/dev/null中。
logfile ""
对于redis来说，可以**设置其数据库的总数量，假如你希望一个redis包含16个数据库**，那么设置如下：
databases 16
这16个数据库的编号将是0到15。默认的数据库是编号为**0**的数据库。用户可以**使用select < DBid>来选择相应的数据库。**

## redis配置 – 快照
快照，主要涉及的是redis的RDB持久化相关的配置。
我们可以用如下的指令来让数据保存到磁盘上，即控制RDB快照功能：
save < seconds> < changes>
save 900 1 //表示每15分钟且至少有1个key改变，就触发一次持久化
save 300 10 //表示每5分钟且至少有10个key改变，就触发一次持久化
save 60 10000 //表示每60秒至少有10000个key改变，就触发一次持久化

我们还可以**设置快照文件的名称**，默认是这样配置的：
dbfilename dump.rdb
最后，你还可以设置这个**快照文件存放的路径**。比如默认设置就是当前文件夹：
dir ./


## redis配置 – 主从复制
redis提供了主从同步功能。
通过slaveof配置项可以控制某一个redis作为另一个redis的从服务器，通过指定IP和端口来定位到主redis的位置。一般情况下，我们会建议用户为从redis设置一个不同频率的快照持久化的周期，或者为从redis配置一个不同的服务端口等等。
slaveof < masterip> < masterport>
如果主redis设置了验证密码的话（使用requirepass来设置），则在从redis的配置中要使用masterauth来设置校验密码，否则的话，主redis会拒绝从redis的访问请求。
masterauth < master-password>
当从redis失去了与主redis的连接，或者主从同步正在进行中时，redis该如何处理外部发来的访问请求呢？这里，从redis可以有两种选择：
第一种选择：如果slave-serve-stale-data设置为yes（默认），则从redis仍会继续响应客户端的读写请求。
第二种选择：如果slave-serve-stale-data设置为no，则从redis会对客户端的请求返回“SYNC with master in progress”，当然也有例外，当客户端发来INFO请求和SLAVEOF请求，从redis还是会进行处理。
自从redis2.6版本之后，默认从redis为只读。
slave-read-only yes

只读的从redis并不适合直接暴露给不可信的客户端。为了尽量降低风险，可以使用rename-command指令来将一些可能有破坏力的命令重命名，避免外部直接调用。比如：
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52

我们可以给众多的从redis设置优先级，在主redis持续工作不正常的情况，优先级高的从redis将会升级为主redis。而编号越小，优先级越高。比如一个主redis有三个从redis，优先级编号分别为10、100、25，那么编号为10的从redis将会被首先选中升级为主redis。当优先级被设置为0时，这个从redis将永远也不会被选中。默认的优先级为100。
slave-priority 100

## redis配置 – 安全
我们可以要求redis客户端在向redis-server发送请求之前，先进行密码验证。当你的redis-server处于一个不太可信的网络环境中时，相信你会用上这个功能。由于redis性能非常高，所以每秒钟可以完成多达15万次的密码尝试，所以你最好设置一个足够复杂的密码，否则很容易被黑客破解。
requirepass zhimakaimen
redis允许我们对redis指令进行更名，比如将一些比较危险的命令改个名字，避免被误执行。比如可以把CONFIG命令改成一个很复杂的名字，这样可以避免外部的调用。

## redis配置 -限制
我们可以设置redis同时可以与多少个客户端进行连接。默认情况下为10000个客户端。
maxclients 10000
我们甚至可以设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。
maxmemory < bytes>
需要注意的一点是，如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。
对于内存移除规则来说，redis提供了多达6种的移除规则。他们是：
1.volatile-lru：使用LRU算法移除过期集合中的key
2.allkeys-lru：使用LRU算法移除key
3.volatile-random：在过期集合中移除随机的key
4.allkeys-random：移除随机的key
5.volatile-ttl：移除那些TTL值最小的key，即那些最近才过期的key。
6.noeviction：不进行移除。针对写操作，只是返回错误信息。
maxmemory-policy volatile-lru

## redis配置 – 追加模式
默认情况下，redis会异步的将数据持久化到磁盘。这种模式在大部分应用程序中已被验证是很有效的，但是在一些问题发生时，比如断电，则这种机制可能会导致数分钟的写请求丢失。
如博文上半部分中介绍的，追加文件（Append Only File）是一种更好的保持数据一致性的方式。即使当服务器断电时，也仅会有1秒钟的写请求丢失，当redis进程出现问题且操作系统运行正常时，甚至只会丢失一条写请求。
我们建议大家，AOF机制和RDB机制可以同时使用，不会有任何冲突。
appendonly no
我们还可以设置aof文件的名称：
appendfilename "appendonly.aof"
fsync()调用，用来告诉操作系统立即将缓存的指令写入磁盘。一些操作系统会“立即”进行，而另外一些操作系统则会“尽快”进行。
redis支持三种不同的模式：
1.no：不调用fsync()。而是让操作系统自行决定sync的时间。这种模式下，redis的性能会最快。
2.always：在每次写请求后都调用fsync()。这种模式下，redis会相对较慢，但数据最安全。
3.everysec：每秒钟调用一次fsync()。这是性能和安全的折衷。
appendfsync everysec
我们允许redis自动重写aof。当aof增长到一定规模时，redis会隐式调用BGREWRITEAOF来重写log文件，以缩减文件体积。
redis是这样工作的：redis会记录上次重写时的aof大小。假如redis自启动至今还没有进行过重写，那么启动时aof文件的大小会被作为基准值。这个基准值会和当前的aof大小进行比较。如果当前aof大小超出所设置的增长比例，则会触发重写。另外，你还需要设置一个最小大小，是为了防止在aof很小时就触发重写。
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Redis 数据备份与恢复
Redis SAVE 命令用于创建当前数据库的备份。
redis Save 命令基本语法如下：
redis 127.0.0.1:6379> SAVE 
OK
该命令将在 redis 安装目录中创建dump.rdb文件。

恢复数据
如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"
以上命令 CONFIG GET dir 输出的 redis 安装目录为 /usr/local/redis/bin。

**Bgsave**
创建 redis 备份文件也可以使用命令 BGSAVE，该命令在**后台执行**。
127.0.0.1:6379> BGSAVE
Background saving started


# 管道（Pipelining）
# key的过期时间
http://www.redis.cn/commands/expire.html
设置key的过期时间，超过时间后，将会自动删除该key。
```
redis> SET mykey "Hello"
OK
redis> EXPIRE mykey 10
(integer) 1
redis> TTL mykey
(integer) 10
redis> SET mykey "Hello World"
OK
redis> TTL mykey
(integer) -1
```

案例: Navigation session
想象一下，你有一个网络服务器，你对用户最近访问的N个网页感兴趣，每一个相邻的页面设置超时时间为60秒。在概念上你为这些网页添加Navigation session，如果你的用户，可能包含有趣的信息，他或她正在寻找什么样的产品，你可以推荐相关产品。
你可以使用下面的策略模型，使用这种模式：每次用户浏览网页调用下面的命令：
```
MULTI
RPUSH pagewviews.user:<userid> http://.....
EXPIRE pagewviews.user:<userid> 60
EXEC
```
如果用户60秒没有操作，这个key将会被删除，不到60秒的话，后续网页将会被继续记录。
Keys的过期时间
通常Redis keys创建时没有设置相关过期时间。他们会一直存在，除非使用显示的命令移除，例如，使用DEL命令。
key的过期时间和永久有效性可以通过`EXPIRE和PERSIST`命令（或者其他相关命令）来进行更新或者删除过期时间。
http://www.redis.cn/commands/expire.html 未完待补充