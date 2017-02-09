title: memcached入门 

#  Memcached入门 
Memcached是免费的，开源的，高性能的，分布式**内存**对象的**缓存系统（键/值字典）**，旨在通过**减轻数据库负载加快动态Web应用**程序的使用。
Memcached是由布拉德·菲茨帕特里克(Brad Fitzpatrick)在2003年为LiveJournal 开发的，现在有很多知名网站都在使用，包括：Netlog, Facebook, Flickr, Wikipedia, Twitter, YouTube等。 
**memcached 主要特点是：**
  * 开源
  * memcached服务器**是一个很大的哈希表**
  * 显著**减少数据库负载**。
  * 非常适合高负载的数据库网站。
  * 在BSD许可下发布
  * 从技术上来说，它是在通过TCP或UDP在服务器和客户端之间来访问。
**不要使用memcached来做什么？**
  * 持久性数据存储
  * 数据库
  * 特殊应用
  * 大对象缓存
  * 容错或高可用性

##  安装与连接 

**在Ubuntu上安装Memcached**
要在Ubuntu上安装Memcached，打开终端，然后输入以下命令：
```

$sudo apt-get update
$sudo apt-get install memcached

```
确认memcached是否安装.要确认memcached安装与否，需要运行下面的命令
```

$ps aux | grep memcached

```
上面的命令将显示Memcached是` 默认端口11211 `上，如运行在其它端口，那么运行以下命令来启动memcached服务器：
```

$memcached -p 11111 -U 11111 -d

```
上面的命令将启动服务器上的TCP端口11111并监听UDP端口11111作为守护进程。可以从一个安装memcached服务器上运行多个实例。

这里需要说明一下**memcached服务的启动参数：**
  * -p 监听的端口
  * ` -l 连接的IP地址, 默认是本机 `
  * -d start 启动memcached服务
  * -d restart 重起memcached服务
  * -d stop|shutdown 关闭正在运行的memcached服务
  * -d install 安装memcached服务
  * -d uninstall 卸载memcached服务
  * -u 以的身份运行 (仅在以root运行的时候有效)
  *`  -m 最大内存使用 `，单位MB。默认64MB
  * -M 内存耗尽时返回错误，而不是删除项
  * -c 最大同时连接数，默认是1024
  * -f 块大小增长因子，默认是1.25-n 最小分配空间，key+value+flags默认是48
  * -h 显示帮助
**Memcached的Java环境设置**
需要下载[spymemcached-2.10.3.jar](http://code.google.com/p/spymemcached/downloads/list)，并设置这个jar到classpath到java程序来使用memcached
要连接到memcache服务器，需要使用telnet命令到主机和端口名称。
```

$telnet HOST PORT

```
在这里，Host 和Port 是memcached服务器运行的IP和端口
下面给出的示例演示如何连接到memcached服务器，并运行一个简单的set和get命令。在这个例子中，假定memcached服务器在主机IP是127.0.0.1，` 端口11211 `上运行
```

$telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
// now store some data and get it from memcached server
set yiibai 0 900 9
memcached
STORED
get yiibai
VALUE yiibai 0 9
memcached
END

```
从Java应用程序连接要从Java程序连接memcached服务器，需要添加memcached的jar包到类路径，如图前面

##  Memcached设置/set数据 
memcached 的 set 命令用于一个新的值,为一个新的或现有的键(key)设置一个值。
memcached set 命令的基本语法如下所示：
set key flags exptime bytes [noreply] 
value 
如下图所示以上关键字的含义：
key 是通过被存储在Memcached的数据并从memcached获取键(key)的名称。
flags 是32位无符号整数，该项目被检索时用的数据(由用户提供)，并沿数据返回服务器存储。
exptime 以秒过期时间，0表示没有延迟，如果exptime大于30天，Memcached将使用它作为UNIX时间戳过期。
bytes 是在数据块中，需要被存储的字节数。基本上，这是一个需要存储在memcached的数据的长度。
noreply (可选) 参数告知服务器不发送回复
value 是一个需要存储的数据。数据需要与上述选项执行命令后，将通过新的一行。
输出
上述命令的输出如下所示：
STORED
STORED 表示成功。
ERROR 以表明有问题，同时保存数据或错误的语法。

示例
```

set yiibai 0 900 9 
memcached 
STORED 
get yiibai 
VALUE yiibai 0 9
memcached
END

``` 
在上面的例子中，我们使用yiibai作为键，memcached在其900秒失效时间并设定值。
使用Java应用程序的数据集
设置memcached服务器的一个键，需要使用memcached 的 set方法。

##  Memcached添加数据 
Memcached的add命令用于为一个值(value)设置为一个新的键(key)。如果键(key)已经存在，那么它输出NOT_STORED。
memcached 的 Add命令的基本语法如下所示：
add key flags exptime bytes [noreply]
value
```

add key 0 900 9
memcached
STORED
get key
VALUE key 0 9
memcached
END

```
在上面的例子中，我们已经使用key作为memcached的键在其900秒失效时间内添加值。
**使用Java应用程序添加数据**
要在memcached服务器中添加数据，需要使用memcached的add方法。
```

import net.spy.memcached.MemcachedClient;
public class MemcachedJava {
   public static void main(String[] args) {
      //Connecting to Memcached server on localhost
      MemcachedClient mcc = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));
      System.out.println("Connection to server sucessfully");
      System.out.println("add status:"+mcc.add("yiibai", 900, "redis").done);
      System.out.println("add status:"+mcc.add("tp", 900, "redis").done);
      //Get value from cache
      System.out.println("Get from Cache tp:"+mcc.get("tp"));
   }
}

```
输出
当上述程序编译和运行，它提供了以下的输出：
Connection to server successfully
add status:false
add status:true
Get from Cache tp:redis

##  Memcached替换/Replace数据 
Memcached的replace 命令用来替换现有键的值。如果该键不存在，那么它输出NOT_STORED
memcached的replace命令的基本语法如下所示：
replace key flags exptime bytes [noreply]
value
使用Java应用程序更换数据
要替换memcached服务器的数据，则需要使用Memcached的replace方法。

##  Memcached追加/append数据 
memcached的append 命令是用来添加一些数据到现有键(key)。数据是存储在键的现有数据之后。
memcached的append命令的基本语法如下所示：
append key flags exptime bytes [noreply]
value
使用Java应用程序追加数据
附加数据到memcached服务器，需要使用memcached的append方法。

**Memcached预先添加/Prepend数据**
Memcached的prepend命令用于添加一些数据到现有的键(key)。数据将存储在键的现有的数据之前。

##  Memcached cas命令 
Memcached 的 cas 命令用于设置数据，如果自上一次获取没有人更新。如果该键不在memcached中，那么它返回NOT_FOUND。
set key flags exptime bytes unique_cas_key [noreply]
value

unique_cas_key 从gets命令的获得唯一键。

STORED 表示成功。
ERROR 以表明有问题，同时保存数据或错误的语法。
EXISTS 以表明自上一次获取起已有人修改了CAS数据。
EXISTS 以表示该键不存在于memcached服务器。

使用Java应用CAS
若要从 memcached 服务器取cas数据，需要使用memcached的gets得到。
```

import net.spy.memcached.MemcachedClient;
public class MemcachedJava {
   public static void main(String[] args) {
      //Connecting to Memcached server on localhost
      MemcachedClient mcc = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));
      System.out.println("Connection to server sucessfully");
      System.out.println("set status:"+mcc.set("yiibai", 900, "memcached").isDone());
      //Get cas token from cache
      long castToken = mcc.gets("yiibai").cas;
      System.out.println("Cas token:"+castToken);
      // now set new data in memcached server
      System.out.println("Now set new data:"+mcc.cas("yiibai", castToken, 900, "redis"));
      System.out.println("Get from Cache:"+mcc.get("yiibai"));
   }
}

```

##  Memcached获取/get数据 
Memcached 的 get 命令用于获取存储在键的值。如果该键在memcached 中不存在，那么它没有返回值。
memcached 的 get 命令的基本语法如下所示：
get key
使用Java应用程序获取数据
要从memcached服务器获取数据，需要使用memcached 的get方法。

##  Memcached删除/Delete数据 
Memcached的delete命令用于删除memcached服务器现有的键。
memcached delete命令的基本语法如下所示：
delete key
如果键成功删除，则返回DELETED，如果key没有找到则返回NOT_FOUND，否则返回ERROR。
使用Java应用程序删除数据
mcc.delete("yiibai").isDone()

##  Memcached清除数据 
Memcached的 flush_all 命令用于删除memcached服务器中的所有数据(键值对)。它接受一个叫做time可选参数，表示这个时间后的所有memcached数据会被清除。
语法
memcached 的 flush_all 命令的基本语法如下所示：
flush_all [time] [noreply]
上面的命令总是返回OK

使用Java应用程序清除数据
要清除memcached服务器的数据，则需要使用memcached的flush 方法。 
mcc.flush().isDone()

参考：http://www.yiibai.com/memcached/
http://blog.csdn.net/wangli61289/article/details/12850927