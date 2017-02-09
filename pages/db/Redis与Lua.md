author:xbynet
title:Redis与Lua
modifyAt:2016-12-26 22:39:22
location:db/Redis与Lua
createAt:2016-12-22 17:46:22

# 基本命令
Redis 脚本使用 Lua 解释器来执行脚本。 Reids 2.6 版本通过内嵌支持 Lua 环境。执行脚本的常用命令为 EVAL。

```
EVAL script numkeys key [key ...] arg [arg ...]
EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

1	EVAL script numkeys key [key ...] arg [arg ...] 执行 Lua 脚本。
2	EVALSHA sha1 numkeys key [key ...] arg [arg ...] 执行 Lua 脚本。
3	SCRIPT EXISTS script [script ...] 查看指定的脚本是否已经被保存在缓存当中。
4	SCRIPT FLUSH 从脚本缓存中移除所有脚本。
5	SCRIPT KILL 杀死当前正在运行的 Lua 脚本。
6	SCRIPT LOAD script 将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。

**Redis Eval** 命令使用 Lua 解释器执行脚本。

EVAL script numkeys key [key ...] arg [arg ...]
参数说明
script： 参数是一段 Lua 5.1 脚本程序。脚本不必(也不应该)定义为一个 Lua 函数。
numkeys： 用于指定键名参数的个数。
key [key ...]： 从 EVAL 的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，用 1 为基址的形式访问( KEYS[1] ， KEYS[2] ，以此类推)。
arg [arg ...]： 附加参数，在 Lua 中通过全局变量 ARGV 数组访问，访问的形式和 KEYS 变量类似( ARGV[1] 、 ARGV[2] ，诸如此类)。

**Redis Evalsha** 命令根据给定的 sha1 校验码，执行缓存在服务器中的脚本。
EVALSHA sha1 numkeys key [key ...] arg [arg ...] 

```
redis 127.0.0.1:6379> SCRIPT LOAD "return 'hello moto'"
"232fd51614574cf0867b83d384a5e898cfd24e5a"
 
redis 127.0.0.1:6379> EVALSHA "232fd51614574cf0867b83d384a5e898cfd24e5a" 0
"hello moto"

```

**Redis Script Exists** 命令用于校验指定的脚本是否已经被保存在缓存当中。
SCRIPT EXISTS script [script ...] 

```
redis 127.0.0.1:6379> SCRIPT LOAD "return 'hello moto'"    # 载入一个脚本
"232fd51614574cf0867b83d384a5e898cfd24e5a"
 
redis 127.0.0.1:6379> SCRIPT EXISTS 232fd51614574cf0867b83d384a5e898cfd24e5a
1) (integer) 1
 
redis 127.0.0.1:6379> SCRIPT FLUSH     # 清空缓存
OK
 
redis 127.0.0.1:6379> SCRIPT EXISTS 232fd51614574cf0867b83d384a5e898cfd24e5a
1) (integer) 0
```

**SCRIPT FLUSH **从脚本缓存中移除所有脚本。
**SCRIPT KILL **杀死当前正在运行的 Lua 脚本。
**SCRIPT LOAD script** 将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。

# 详细说明
这是从一个Lua脚本中使用两个不同的Lua函数来调用Redis的命令的例子：

```
redis.call()
redis.pcall()
```
redis.call() 与 redis.pcall()很类似, 他们唯一的区别是当redis命令执行结果返回错误时， redis.call()将返回给调用者一个错误，而redis.pcall()会将捕获的错误以Lua表的形式返回
redis.call() 和 redis.pcall() 两个函数的参数可以是任意的 Redis 命令：
```
> eval "return redis.call('set','foo','bar')" 0
OK
```
需要注意的是，上面这段脚本的确实现了将键 foo 的值设为 bar 的目的，`但是，它违反了 EVAL 命令的语义，因为脚本里使用的所有键都应该由 KEYS 数组来传递`，就像这样：
```
> eval "return redis.call('set',KEYS[1],'bar')" 1 foo
OK
```
要求使用正确的形式来传递键(key)是有原因的，**因为不仅仅是 EVAL 这个命令，所有的 Redis 命令，在执行之前都会被分析，籍此来确定命令会对哪些键进行操作。
因此，对于 EVAL 命令来说，必须使用正确的形式来传递键，才能确保分析工作正确地执行。 **

## Lua 数据类型和 Redis 数据类型之间转换
当 Lua 通过 call() 或 pcall() 函数执行 Redis 命令的时候，命令的返回值会被转换成 Lua 数据结构。 同样地，当 Lua 脚本在 Redis 内置的解释器里运行时，Lua 脚本的返回值也会被转换成 Redis 协议(protocol)，然后由 EVAL 将值返回给客户端。
下面两点需要重点注意：
lua中整数和浮点数之间没有什么区别。因此，我们始终将Lua的数字转换成整数的回复，这样将舍去小数部分。`如果你想从Lua返回一个浮点数，你应该将它作为一个字符串`
有两个辅助函数从Lua返回Redis的类型。

* redis.error_reply(error_string) returns an error reply. This function simply returns the single field table with the err field set to the specified string for you.
* redis.status_reply(status_string) returns a status reply. This function simply returns the single field table with the ok field set to the specified string for you.
```
return {err="My Error"}
return redis.error_reply("My Error")
```
## 脚本的原子性
Redis 使用单个 Lua 解释器去运行所有脚本，并且， Redis 也保证脚本会以原子性(atomic)的方式执行： 当某个脚本正在运行的时候，不会有其他脚本或 Redis 命令被执行。 这和使用 MULTI / EXEC 包围的事务很类似。 在其他别的客户端看来，脚本的效果(effect)要么是不可见的(not visible)，要么就是已完成的(already completed)。

## 脚本缓存和 EVALSHA
EVAL 命令要求你在每次执行脚本的时候都发送一次脚本主体(script body)。**Redis 有一个内部的脚本缓存机制，因此它不会每次都重新编译脚本**。
 EVALSHA 命令，它的作用和 EVAL 一样，都用于对脚本求值，但它接受的第一个参数不是脚本，而是脚本的 SHA1 校验和(sum)。
 客户端库的底层实现可以一直乐观地使用 EVALSHA 来代替 EVAL ，并期望着要使用的脚本已经保存在服务器上了，只有当 NOSCRIPT 错误发生时，才使用 EVAL 命令重新发送脚本，这样就可以最大限度地节省带宽。
 刷新脚本缓存的唯一办法是显式地调用 SCRIPT FLUSH 命令，这个命令会清空运行过的所有脚本的缓存。通常只有在云计算环境中，才会执行这个命令。
 
## Redis对lua脚本做出的限制
 
* 不能访问系统时间或者其他内部状态
* Redis 会返回一个错误，阻止这样的脚本运行： 这些脚本在执行随机命令之后(比如 RANDOMKEY 、 SRANDMEMBER 或 TIME 等)，还会执行可以修改数据集的 Redis 命令。如果脚本只是执行只读操作，那么就没有这一限制。
* 每当从 Lua 脚本中调用那些返回无序元素的命令时，执行命令所得的数据在返回给 Lua 之前会先执行一个静默(slient)的字典序排序(lexicographical sorting)。举个例子，因为 Redis 的 Set 保存的是无序的元素，所以在 Redis 命令行客户端中直接执行 SMEMBERS ，返回的元素是无序的，但是，假如在脚本中执行 redis.call(“smembers”, KEYS[1]) ，那么返回的总是排过序的元素。
* 对 Lua 的伪随机数生成函数 math.random 和 math.randomseed 进行修改，使得每次在运行新脚本的时候，总是拥有同样的 seed 值。这意味着，每次运行脚本时，只要不使用 math.randomseed ，那么 math.random 产生的随机数序列总是相同的。
* 全局变量保护，为了防止不必要的数据泄漏进 Lua 环境， `Redis 脚本不允许创建全局变量`。如果一个脚本需要在多次执行之间维持某种状态，它应该使用 Redis key 来进行状态保存。避免引入全局变量的一个诀窍是：将脚本中用到的所有变量都使用 `local` 关键字定义为局部变量。

## 可用库
Redis Lua解释器可用加载以下Lua库：
base
table
string
math
debug 
struct  一个Lua装箱/拆箱的库
cjson 为Lua提供极快的JSON处理
cmsgpack为Lua提供了简单、快速的MessagePack操纵
bitop 为Lua的位运算模块增加了按位操作数。
redis.sha1hex function. 对字符串执行SHA1算法
每一个Redis实例都拥有以上的所有类库，以确保您使用脚本的环境都是一样的。
struct, CJSON 和 cmsgpack 都是外部库, 所有其他库都是标准。

```
redis 127.0.0.1:6379> eval 'return cjson.encode({["foo"]= "bar"})' 0
"{\"foo\":\"bar\"}"
redis 127.0.0.1:6379> eval 'return cjson.decode(ARGV[1])["foo"]' 0 "{\"foo\":\"bar\"}"
"bar"

127.0.0.1:6379> eval 'return cmsgpack.pack({"foo", "bar", "baz"})' 0
"\x93\xa3foo\xa3bar\xa3baz"
127.0.0.1:6379> eval 'return cmsgpack.unpack(ARGV[1])' 0 "\x93\xa3foo\xa3bar\xa3baz"
1) "foo"
2) "bar"
3) "baz"
```

## 使用脚本记录Redis 日志
在 Lua 脚本中，可以通过调用 redis.log 函数来写 Redis 日志(log)：

```
redis.log(loglevel,message)
```
其中， message 参数是一个字符串，而 loglevel 参数可以是以下任意一个值：
* redis.LOG_DEBUG
* redis.LOG_VERBOSE
* redis.LOG_NOTICE
* redis.LOG_WARNING
上面的这些等级(level)和标准 Redis 日志的等级相对应。
只有那些和当前 Redis 实例所设置的日志等级相同或更高级的日志才会被散发。
以下是一个日志示例：
```
redis.log(redis.LOG_WARNING, "Something is wrong with this script.")
执行上面的函数会产生这样的信息：
[32343] 22 Mar 15:21:39 # Something is wrong with this script.
```

# 沙箱(sandbox)和最大执行时间
**脚本应该仅仅用于传递参数和对 Redis 数据进行处理**，它不应该尝试去访问外部系统(比如文件系统)，或者执行任何系统调用。
除此之外，**脚本还有一个最大执行时间限制，它的默认值是 5 秒钟**，一般正常运作的脚本通常可以在几分之几毫秒之内完成，花不了那么多时间，这个限制主要是为了防止因编程错误而造成的无限循环而设置的。
最大执行时间的长短由` lua-time-limit` 选项来控制(以毫秒为单位)，可以通过编辑 redis.conf 文件或者使用 CONFIG GET 和 CONFIG SET 命令来修改它。

当一个脚本达到最大执行时间的时候，它并不会自动被 Redis 结束，因为 Redis 必须保证脚本执行的原子性，而中途停止脚本的运行意味着可能会留下未处理完的数据在数据集(data set)里面。
因此，当脚本运行的时间超过最大执行时间后，以下动作会被执行：
Redis 记录一个脚本正在超时运行
Redis 开始重新接受其他客户端的命令请求，但是只有 SCRIPT KILL 和 SHUTDOWN NOSAVE 两个命令会被处理，对于其他命令请求， Redis 服务器只是简单地返回 BUSY 错误。
可以使用 `SCRIPT KILL` 命令将一个仅执行只读命令的脚本杀死，因为只读命令并不修改数据，因此杀死这个脚本并不破坏数据的完整性
如果脚本已经执行过写命令，那么唯一允许执行的操作就是 `SHUTDOWN NOSAVE` ，它通过停止服务器来阻止当前数据集写入磁盘

# pipeline上下文(context)中的 EVALSHA
一旦在pipeline中因为 EVALSHA 命令而发生** NOSCRIPT 错误**，那么这个pipeline就再也没有办法重新执行了，否则的话，命令的执行顺序就会被打乱。
为了防止出现以上所说的问题，客户端库实现应该实施以下的其中一项措施：
* 总是在pipeline中使用 EVAL 命令
* 检查pipeline中要用到的所有命令，找到其中的 EVAL 命令，并使用 `SCRIPT EXISTS` 命令检查要用到的脚本是不是全都已经保存在缓存里面了。如果所需的全部脚本都可以在缓存里找到，那么就可以放心地将所有 EVAL 命令改成 EVALSHA 命令，否则的话，就要在pipeline的顶端(top)将缺少的脚本用 `SCRIPT LOAD` 命令加上去。


# 案例1-实现访问频率限制: 
实现访问者 $ip 在一定的时间 $time 内只能访问 $limit 次.
非脚本实现
```
private boolean accessLimit(String ip, int limit, int time, Jedis jedis) {
    boolean result = true;

    String key = "rate.limit:" + ip;
    if (jedis.exists(key)) {
        long afterValue = jedis.incr(key);
        if (afterValue > limit) {
            result = false;
        }
    } else {
        Transaction transaction = jedis.multi();
        transaction.incr(key);
        transaction.expire(key, time);
        transaction.exec();
    }
    return result;
}
```
以上代码有两点缺陷

* 可能会出现竞态条件: 解决方法是用 WATCH 监控 rate.limit:$IP 的变动, 但较为麻烦;
* 以上代码在不使用 pipeline 的情况下最多需要向Redis请求5条指令, 传输过多.

Lua脚本实现
Redis 允许将 Lua 脚本传到 Redis 服务器中执行, 脚本内可以调用大部分 Redis 命令, 且 Redis 保证脚本的 原子性 :
首先需要准备Lua代码: script.lua

```
local key = "rate.limit:" .. KEYS[1]
local limit = tonumber(ARGV[1])
local expire_time = ARGV[2]

local is_exists = redis.call("EXISTS", key)
if is_exists == 1 then
    if redis.call("INCR", key) > limit then
        return 0
    else
        return 1
    end
else
    redis.call("SET", key, 1)
    redis.call("EXPIRE", key, expire_time)
    return 1
end
```

Java
```
private boolean accessLimit(String ip, int limit, int timeout, Jedis connection) throws IOException {
    List<String> keys = Collections.singletonList(ip);
    List<String> argv = Arrays.asList(String.valueOf(limit), String.valueOf(timeout));

    return 1 == (long) connection.eval(loadScriptString("script.lua"), keys, argv);
}

// 加载Lua代码
private String loadScriptString(String fileName) throws IOException {
    Reader reader = new InputStreamReader(Client.class.getClassLoader().getResourceAsStream(fileName));
    return CharStreams.toString(reader);
}
```

Lua 嵌入 Redis 优势:

* 减少网络开销: 不使用 Lua 的代码需要向 Redis 发送多次请求, 而脚本只需一次即可, 减少网络传输;
* 原子操作: Redis 将整个脚本作为一个原子执行, 无需担心并发, 也就无需事务;
* 复用: 脚本会永久保存 Redis 中, 其他客户端可继续使用.

# 案例2-使用Lua脚本重新构建带有过期时间的分布式锁.
案例来源: < Redis实战 > 第6、11章, 构建步骤:

* 锁申请 
* 首先尝试加锁: 
* 成功则为锁设定过期时间; 返回;
* 失败检测锁是否添加了过期时间;
* wait.
* 锁释放 
* 检查当前线程是否真的持有了该锁:
* 持有: 则释放; 返回成功;
* 失败: 返回失败.

非Lua实现
```
def acquire_lock_with_timeout(
    conn, lockname, acquire_timeout=10, lock_timeout=10):
    # 128 位随机标识符。
    identifier = str(uuid.uuid4())                     
    lockname = 'lock:' + lockname
    # 确保传给 EXPIRE 的都是整数。
    lock_timeout = int(math.ceil(lock_timeout))        
    
    end = time.time() + acquire_timeout
    while time.time() < end:
        # 获取锁并设置过期时间。
        if conn.setnx(lockname, identifier):           
            conn.expire(lockname, lock_timeout)        
            return identifier
        # 检查过期时间，并在有需要时对其进行更新。
        elif not conn.ttl(lockname):                   
            conn.expire(lockname, lock_timeout)        
    
        time.sleep(.001)
    
    return False

def release_lock(conn, lockname, identifier):
    pipe = conn.pipeline(True)
    lockname = 'lock:' + lockname
    
    while True:
        try:
            pipe.watch(lockname)                  #A
            if pipe.get(lockname) == identifier:  #A
                pipe.multi()                      #B
                pipe.delete(lockname)             #B
                pipe.execute()                    #B
                return True                       #B
    
            pipe.unwatch()
            break
    
        except redis.exceptions.WatchError:       #C
            pass                                  #C
    
    return False  
```

Lua脚本实现

```
import math
from redis import StrictRedis

redis_client=StrictRedis()
def register_all_script():
    global lock_lua
    global release_lock_lua
    lock_lua=redis_client.register_script("""
        -- 检测锁是否已经存在。（再次提醒，Lua 表格的索引是从 1 开始的。）
        if redis.call('exists', KEYS[1]) == 0 then             
            -- 使用给定的过期时间以及标识符去设置键。
            return redis.call('setex', KEYS[1], unpack(ARGV))  
        end
        """) 
    release_lock_lua=redis_client.register_script("""
        -- 检查锁是否匹配。
        if redis.call('get', KEYS[1]) == ARGV[1] then              
            -- 删除锁并确保脚本总是返回真值。
            return redis.call('del', KEYS[1]) or true              
        end
        """)

register_all_script()

def acquire_lock_with_timeout(
    conn, lockname, acquire_timeout=10, lock_timeout=10):
    identifier = str(uuid.uuid4())                      
    lockname = 'lock:' + lockname
    lock_timeout = int(math.ceil(lock_timeout))      
    
    acquired = False
    end = time.time() + acquire_timeout
    while time.time() < end and not acquired:
        # 执行实际的锁获取操作，通过检查确保 Lua 调用已经执行成功。
        acquired = lock_lua( keys=[lockname],args=[lock_timeout, identifier],
            client=conn ).decode('utf-8') == 'OK'  
    
        time.sleep(.001 * (not acquired))
    
    return acquired and identifier

def release_lock(conn, lockname, identifier):
    lockname = 'lock:' + lockname
    # 调用负责释放锁的 Lua 函数。
    return release_lock_lua( keys=[lockname], args=[identifier],client=conn) 

```
参考：http://www.redis.cn/commands/eval.html
http://www.redis.net.cn/tutorial/3516.html
http://www.oschina.net/translate/intro-to-lua-for-redis-programmers
http://www.tuicool.com/articles/bUjYv2i