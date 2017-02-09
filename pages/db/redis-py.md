author:xbynet
title:Redis-py
modifyAt:2016-12-26 22:52:57
location:db/redis-py
createAt:2016-12-12 03:10:37

官网：https://github.com/andymccurdy/redis-py
当前版本：2.10.5
注：这不是完整翻译，只提取了关键信息。省略了部分内容，如lua脚本支持。

    pip install redis
    pip install hiredis（解析器，可选。windows下好像不行。）

    >>> import redis
    >>> r = redis.StrictRedis(host='localhost', port=6379, db=0)
    >>> r.set('foo', 'bar')
    True
    >>> r.get('foo')
    'bar'

**redis-py采取两种client class实现redis命令：**
**其一、StrictRedis类尽量坚持官方语法**，但是以下除外：

 - SELECT: 没有实现，应该是线程安全的原因。
 - DEL: 由于del是python语法关键字，所用delete来代替。
 - CONFIG GET|SET: 分开用 config_get or config_set来代替
 - MULTI/EXEC: 事务作为Pipeline类的其中一部分的实现。Pipeline默认保证了MULTI,EXEC声明。但是你可以指定transaction=False来禁用这一行为。
 - SUBSCRIBE/LISTEN:PubSub作为一个独立的类来实现发布订阅机制。
 - SCAN/SSCAN/HSCAN/ZSCAN:每个命令都对应一个等价的迭代器方法scan_iter/sscan_iter/hscan_iter/zscan_iter methods for this behavior.

**其二、Redis类是StrictRedis的子类，提供redis-py版本向后的兼容性。**

关于StrictRedis与Redis的区别：(**官方推荐使用StrictRedis.**)
以下几个方法在StrictRedis和Redis类中的参数顺序不同。
**LREM**: Order of 'num' and 'value' arguments reversed such that 'num' can provide a default value of zero.
在Redis类中是这样的：
lrem(self, name, value, num=0)
在StrictRedis类中是这样的：
lrem(self, name, count, value)

**ZADD**: Redis specifies the 'score' argument before 'value'. These were swapped accidentally when being implemented and not discovered until after people were already using it. The Redis class expects *args in the form of: name1, score1, name2, score2, ...
在Redis类中是这样的：
redis.zadd('my-key', 'name1', 1.1, 'name2', 2.2, name3=3.3, name4=4.4)
在StrictRedis中是这样的：
redis.zadd('my-key', 1.1, 'name1', 2.2, 'name2', name3=3.3, name4=4.4)

**SETEX**: Order of 'time' and 'value' arguments reversed.
在Redis类中是这样的：
setex(self, name, value, time)
而在StrictRedis中是这样的：
setex(self, name, time, value)

# 连接池
**`默认情况下，当你实例化Redis或StrictRedis时，会自动在内部创建一个连接池`**，
但是你也可以显式指定连接池(记住：通常你没有必要这么做。)

    >>> pool = redis.ConnectionPool(host='localhost', port=6379, db=0)
    >>> r = redis.Redis(connection_pool=pool)

Connections：redis-py提供两种类型的连接：基于TCP端口的，基于Unix socket文件的（需要redis服务器开启配置）。

    >>> r = redis.Redis(unix_socket_path='/tmp/redis.sock')

如果你需要，自定义连接类，需要告知连接池。

    >>> pool = redis.ConnectionPool(connection_class=YourConnectionClass,
                                    your_arg='...', ...)

**释放连接回到连接池(这是对于pipeline而言)**：可以使用Redis类的`reset()`方法，或者使用`with`上下文管理语法。

**解析器：**
解析器控制如何解析Redis-server的响应内容，redis-py提供两种方式的解析器类支持：**PythonParser和HiredisParser**(需要单独安装)。它优先选用HiredisParser,如果不存在，则选用PythonParser. Hiredis是redis核心团队开发的一个高性能c库，能够提高10x的解析速度。

**响应回调：**
The client class使用一系列的callbacks来完成响应到对应python类型的映射。这些响应回调，定义在 Redis client class中的`RESPONSE_CALLBACKS`字典中。你可以使用`set_response_callback` 方法来添加自定义回调类。这个方法接受两个参数：一个命令名字，一个回调类。回调类接受至少一个参数：响应内容，关键字参数作为命令调用时的参数。

# 线程安全性：
Redis client instances是线程安全的。由于线程安全原因，不提供select实现，因为它会导致数据库的切换。
在不同线程间传递PubSub or Pipeline对象也是不安全的。

# Pipelines 
Pipelines是Redis类的一个子类，支持缓存多个命令，然后作为单个请求发送。通过减少TCP请求次数来达到提供性能的目的。

    >>> r = redis.Redis(...)
    >>> r.set('bing', 'baz')
    >>> # Use the pipeline() method to create a pipeline instance
    >>> pipe = r.pipeline()
    >>> # The following SET commands are buffered
    >>> pipe.set('foo', 'bar')
    >>> pipe.get('bing')
    >>> # the EXECUTE call sends all buffered commands to the server, returning
    >>> # a list of responses, one for each command.
    >>> pipe.execute()
    [True, 'baz']

Pipelines的实现采用**流式API**，故而你可以采用以下链式调用的方式：

    >>> pipe.set('foo', 'bar').sadd('faz', 'baz').incr('auto_number').execute()
    [True, True, 6]

**Pipelines默认以原子性(事务)的形式执行所有缓存的命令**,你也可以禁用这一行为：

    >>> pipe = r.pipeline(transaction=False)

`WATCH命令`提供了在事务之前检测一个或多个key值的变化。一旦在事务执行之前，某个值发生了变化，那么事务将被取消然后抛出`WatchError` 异常。
利用watch我们可以实现client-side incr命令：

    >>> with r.pipeline() as pipe:
    ...     while 1:
    ...         try:
    ...             # put a WATCH on the key that holds our sequence value
    ...             pipe.watch('OUR-SEQUENCE-KEY')
    ...             # after WATCHing, the pipeline is put into immediate execution
    ...             # mode until we tell it to start buffering commands again.
    ...             # this allows us to get the current value of our sequence
    ...             current_value = pipe.get('OUR-SEQUENCE-KEY')
    ...             next_value = int(current_value) + 1
    ...             # now we can put the pipeline back into buffered mode with MULTI
    ...             pipe.multi()
    ...             pipe.set('OUR-SEQUENCE-KEY', next_value)
    ...             # and finally, execute the pipeline (the set command)
    ...             pipe.execute()
    ...             # if a WatchError wasn't raised during execution, everything
    ...             # we just did happened atomically.
    ...             break
    ...        except WatchError:
    ...             # another client must have changed 'OUR-SEQUENCE-KEY' between
    ...             # the time we started WATCHing it and the pipeline's execution.
    ...             # our best bet is to just retry.
    ...             continue

不过你可以使用`transaction方法`来简化这一操作：它包含handling and retrying watch errors的样板代码。第一参数为callable(这个callable只能接受一个Pipeline参数),及多个需要被WATCH的keys

    >>> def client_side_incr(pipe):
    ...     current_value = pipe.get('OUR-SEQUENCE-KEY')
    ...     next_value = int(current_value) + 1
    ...     pipe.multi()
    ...     pipe.set('OUR-SEQUENCE-KEY', next_value)
    >>>
    >>> r.transaction(client_side_incr, 'OUR-SEQUENCE-KEY')
    [True]


# Publish / Subscribe
`PubSub`对象subscribes to channels and listens for new messages。

    >>> r = redis.StrictRedis(...)
    >>> p = r.pubsub()
    
    >>> p.subscribe('my-first-channel', 'my-second-channel', ...)
    >>> p.psubscribe('my-*', ...)
    
    >>> p.get_message()
    {'pattern': None, 'type': 'subscribe', 'channel': 'my-second-channel', 'data': 1L}
    >>> p.get_message()
    {'pattern': None, 'type': 'subscribe', 'channel': 'my-first-channel', 'data': 2L}
    >>> p.get_message()
    {'pattern': None, 'type': 'psubscribe', 'channel': 'my-*', 'data': 3L}

**通过PubSub获取消息时返回的是一个字典，字典key有如下几个：**
**type**:其中一个， 'subscribe', 'unsubscribe', 'psubscribe', 'punsubscribe', 'message', 'pmessage'
**channel:** The channel [un]subscribed to or the channel a message was published to
**pattern:** The pattern that matched a published message's channel. Will be None in all cases except for 'pmessage' types.
**data:** The message data. With [un]subscribe messages, this value will be the number of channels and patterns the connection is currently subscribed to. With [p]message messages, this value will be the actual published message.
现在来发布消息：

    # the publish method returns the number matching channel and pattern
    # subscriptions. 'my-first-channel' matches both the 'my-first-channel'
    # subscription and the 'my-*' pattern subscription, so this message will
    # be delivered to 2 channels/patterns
    >>> r.publish('my-first-channel', 'some data')
    2
    >>> p.get_message()
    {'channel': 'my-first-channel', 'data': 'some data', 'pattern': None, 'type': 'message'}
    >>> p.get_message()
    {'channel': 'my-first-channel', 'data': 'some data', 'pattern': 'my-*', 'type': 'pmessage'}

**取消订阅：**如果没有传递任何参数，那么这个PubSub订阅的所有的channels or patterns都会被取消。

    >>> p.unsubscribe()
    >>> p.punsubscribe('my-*')
    >>> p.get_message()
    {'channel': 'my-second-channel', 'data': 2L, 'pattern': None, 'type': 'unsubscribe'}
    >>> p.get_message()
    {'channel': 'my-first-channel', 'data': 1L, 'pattern': None, 'type': 'unsubscribe'}
    >>> p.get_message()
    {'channel': 'my-*', 'data': 0L, 'pattern': None, 'type': 'punsubscribe'}
    
## 回调的方式处理发布的消息
**redis-py还允许你通过回调的方式处理发布的消息。**
Message handlers接受一个参数，the message,是一个字典对象。just like the examples above. 
以回调形式订阅：subscribe接受关键字参数，键为channels or patterns,值为回调函数。

    >>> def my_handler(message):
    ...     print 'MY HANDLER: ', message['data']
    >>> p.subscribe(**{'my-channel': my_handler})

`在你注册了回调处理的情况下， get_message()会返回None`。

默认情况下除了发布消息之外，还会传递 subscribe/unsubscribe成功的确认消息，如果你不想接收它们：`ignore_subscribe_messages=True`

    >>> p = r.pubsub(ignore_subscribe_messages=True)
    >>> p.subscribe('my-channel')
    >>> p.get_message()  # hides the subscribe message and returns None
    >>> r.publish('my-channel')
    1
    >>> p.get_message()
    {'channel': 'my-channel', 'data': 'my data', 'pattern': None, 'type': 'message'}

## 三种读取消息的方式
第一种：无限循环通过PubSub对象的get_message()读取消息

    >>> while True:
    >>>     message = p.get_message()
    >>>     if message:
    >>>         # do something with the message
    >>>     time.sleep(0.001)  # be nice to the system :)

第二种，通过阻塞方法`listen()`来读取：p.listen()返回一个generator，阻塞直到有消息可获取。

    >>> for message in p.listen():
    ...     # do something with the message

第三种，开启一个`事件循环线程pubsub.run_in_thread()`方法 creates a new thread and starts the event loop. 并返回线程对象。
但是需要注意的是：`如果你没有注册消息处理函数，那么调用run_in_thread()将会抛出异常redis.exceptions.PubSubError`

    >>> p.subscribe(**{'my-channel': my_handler})
    >>> thread = p.run_in_thread(sleep_time=0.001)
    # the event loop is now running in the background processing messages
    # when it's time to shut it down...
    >>> thread.stop()

## 关于字符编码：
默认情况下，publish的消息会被编码，当你获取消息时得到的是编码后的字节，如果你需要它自动解码，创建Redis client实例时需要指定`decode_responses=True`,(译者注：不建议使用该选项，因为当存在pickle序列化的值时，client.get(key)时会出现解码失败的错误UnicodeDecodeError)

## 关闭释放资源：
PubSub.close() method to shutdown the connection.

    >>> p = r.pubsub()
    >>> ...
    >>> p.close()

# LUA脚本支持：
redis-py支持EVAL, EVALSHA, and SCRIPT命令。同时redis-py提供一个Script 对象来统一进行操作。

```
>>> r = redis.StrictRedis()
>>> lua = """
... local value = redis.call('GET', KEYS[1])
... value = tonumber(value)
... return value * ARGV[1]"""
>>> multiply = r.register_script(lua) 
```
上面代码中的multiply就是一个Script 对象实例。Script 对象实例是一个callable，可以接受以下可选参数来调用：

* **keys**: A list of key names that the script will access. This becomes the `KEYS` list in LUA.
* **args**: A list of argument values. This becomes the `ARGV` list in LUA.
* **client**: A redis-py Client or Pipeline instance that will invoke the script. If client isn't specified, the client that intiially created the Script instance (the one that register_script was invoked from) will be used.

```
>>> r.set('foo', 2)
>>> multiply(keys=['foo'], args=[5])
10
```
你可以指定client，甚至是来自不同redis server的client。

```
>>> r2 = redis.StrictRedis('redis2.example.com')
>>> r2.set('foo', 3)
>>> multiply(keys=['foo'], args=[5], client=r2)
15
```

Script对象可以保证 LUA script被加载到redis脚本缓存。
下面使用pipeline作为client

```
>>> pipe = r.pipeline()
>>> pipe.set('foo', 5)
>>> multiply(keys=['foo'], args=[5], client=pipe)
>>> pipe.execute() #注意这里需要调用execute()
[True, 25]
```

# Sentinel support与节点发现：
Redis Sentinel用于发现Redis节点。**请确保至少一个Sentinel daemon 进程在运行。**
你可以使用Sentinel connection to discover the master and slaves network addresses:

    >>> from redis.sentinel import Sentinel
    >>> sentinel = Sentinel([('localhost', 26379)], socket_timeout=0.1)
    >>> sentinel.discover_master('mymaster')
    ('127.0.0.1', 6379)
    >>> sentinel.discover_slaves('mymaster')
    [('127.0.0.1', 6380)]
    
    >>> master = sentinel.master_for('mymaster', socket_timeout=0.1)
    >>> slave = sentinel.slave_for('mymaster', socket_timeout=0.1)
    >>> master.set('foo', 'bar')
    >>> slave.get('foo')
    'bar'

上面的master and slave对象就是一个普通的`StrictRedis`对象实例。**如果Sentinel配置了连接池的话，它们还会使用这个连接池。**
可能抛出的异常：`MasterNotFoundError ，SlaveNotFoundError` 它们都是`ConnectionError`的子类

# Scan Iterators
Redis 2.8之后有了*SCAN命令。redis-py also exposes the following methods that return Python iterators for convenience: scan_iter, hscan_iter, sscan_iter and zscan_iter.

    >>> for key, value in (('A', '1'), ('B', '2'), ('C', '3')):
    ...     r.set(key, value)
    >>> for key in r.scan_iter():
    ...     print key, r.get(key)
    A 1
    B 2
    C 3