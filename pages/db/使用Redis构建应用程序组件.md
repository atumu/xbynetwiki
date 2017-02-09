modifyAt:2016-12-24 22:26:59
location:db/使用Redis构建应用程序组件
createAt:2016-12-22 23:33:58
author:xbynet
title:使用Redis构建应用程序组件(自动补全、分布式锁、技术信号量、任务队列、消息拉取与群组聊天、文件分发)

# 分布式锁
使用锁来代替watch命令以提升性能。
一般来说，在对数据进行加锁时，程序首先需要通过获取(acquire)锁来得到对数据进行排他性访问的能力，然后才对数据进行一系列操作，最后还要将锁释放(release)给其他程序。对于能够被多个线程访问的**共享内存数据结**构来说，这种先**获取锁，然后执行操作，最后释放锁**的动作非常常见。
Redis使用**WATCH命令**来代替对数据进行加锁，因为WATCH只会在数据被其他客户端抢先修改了的情况下通知执行了这个命令的客户端，而不会阻止其他客户端对数据进行修改，所以这个命令被称为**乐观锁**(optimistic locking)
之所以不选择直接使用操作系统级别的锁，编程语言级别的锁等，而是选择使用Redis来构建锁，这其中的一个原因是和范围有关:为了对Redis存储的数据进行排他性访问，客户端需要访问一个锁，这个锁必须在所有客户端都可见的范围之内，而这个范围就是Redis本身。
注：如果使用Lua脚本，那么会极大得减少对WATCH命令和锁的依赖。

## 简易锁
锁的定义
```
import uuid
import time
import logging as log
import redis

def acquire_lock(conn,lockname,timeout=10): #这个timeout指的是获取锁的最大等待时长，超过时间就说明获取失败
    identifier=str(uuid.uuid4())#128位随机标识符
    end=time.time()+timeout
    while time.time()<end:
        # try to get lock
        if conn.setnx('lock:'+lockname,identifier):
            return identifier
        time.sleep(.001)
    return False

def release_lock(conn,lockname,identifier):
    key='lock:'+lockname
    while True:
        try:
            pipe.watch(key)
            if pipe.get(key)==identifier:
                pipe.multi()
                pipe.delete(key)
                pipe.execute()
                return True
            
            pipe.unwatch()
            break
        except redis.exceptions.WatchError as e:
            log.warning('有其他客户端修改了锁{}'.format(key))
    # 已经失去了锁
    return False

def release_lock2(conn,lockname,identifier):
    key='lock:'+lockname
    status=False
    def release(pipe):
        nonlocal status
        if pipe.get(key)==identifier:
            pipe.multi()
            pipe.delete(key)
            pipe.execute()
            status=True
            return True
        status=False
    conn.transaction(release,key)
    return status

```
锁的应用
```
def purchase_item_with_lock(conn, buyerid, itemid, sellerid):
    buyer = "users:%s" % buyerid
    seller = "users:%s" % sellerid
    item = "%s.%s" % (itemid, sellerid)
    inventory = "inventory:%s" % buyerid

    # 尝试获取锁。
    locked = acquire_lock(conn, 'market:')   
    if not locked:
        return False

    pipe = conn.pipeline(True)
    try:
        # 检查物品是否已经售出，以及买家是否有足够的金钱来购买物品。
        pipe.zscore("market:", item)        
        pipe.hget(buyer, 'funds')            
        price, funds = pipe.execute()         
        if price is None or price > funds:   
            return None                     

        # 将买家支付的货款转移给卖家，并将售出的物品转移给买家。
        pipe.hincrby(seller, 'funds', int(price)) 
        pipe.hincrby(buyer, 'funds', int(-price)) 
        pipe.sadd(inventory, itemid)            
        pipe.zrem("market:", item)               
        pipe.execute()                           
        return True
    finally:
        # 释放锁。
        release_lock(conn, 'market:', locked)   
```

## 超时锁
超时锁存在生存时间限制，确保锁不会被一直占用而导致死锁问题等。
```
def acquire_lock_with_timeout(
    conn, lockname, acquire_timeout=10, lock_timeout=10):
    # 128位随机标识符。
    identifier = str(uuid.uuid4())                   
    lockname = 'lock:' + lockname
    # 确保传给EXPIRE的都是整数。
    lock_timeout = int(math.ceil(lock_timeout))     

    end = time.time() + acquire_timeout
    while time.time() < end:
        # 获取锁并设置过期时间。
        if conn.set(lockname, identifier,ex=lock_timeout,nx=True):        
            return identifier
            
        time.sleep(.001)

    return False
```

建议超时锁和watch一起使用。

# 任务队列

目前有很多专门的任务队列软件：如ActiveMQ，RabbitMQ等等。
两种不同类型的任务队列：第一种队列会根据任务被插入队列的顺序来尽快执行任务；第二种队列则具有安排任务在未来某个特定时间执行的能力。

## 先进先出队列(FIFO)
几种不同的队列FIFO，LIFO，priority队列。
使用一个列表来实现FIFO队列

简单的通过队列发送邮件的例子：
```
def send_sold_email_via_queue(conn, seller, item, price, buyer):
    # 准备好待发送邮件。
    data = {
        'seller_id': seller,                 
        'item_id': item,                      
        'price': price,                         
        'buyer_id': buyer,                      
        'time': time.time()                    
    }
    # 将待发送邮件推入到队列里面。
    conn.rpush('queue:email', json.dumps(data)) 

def process_sold_email_queue(conn):
    while not QUIT:
        # 尝试获取一封待发送邮件。
        packed = conn.blpop(['queue:email'], 30)                  
        # 队列里面暂时还没有待发送邮件，重试。
        if not packed:                                          
            continue

        # 从JSON对象中解码出邮件信息。
        to_send = json.loads(packed[1])                       
        try:
            # 使用预先编写好的邮件发送函数来发送邮件。
            fetch_data_and_send_sold_email(to_send)            
        except EmailSendError as err:
            log_error("Failed to send sold email", err, to_send)
        else:
            log_success("Sent sold email", to_send)
```
如果任务不止是发送邮件呢？当有多个不同类型的任务呢？
### 多个可执行任务

```
def worker_watch_queue(conn, queue, callbacks):
    while not QUIT:
        # 尝试从队列里面取出一项待执行任务。
        packed = conn.blpop([queue], 30)                   
        # 队列为空，没有任务需要执行；重试。
        if not packed:                                     
            continue                                      

        # 解码任务信息。
        name, args = json.loads(packed[1])                
        # 没有找到任务指定的回调函数，用日志记录错误并重试。
        if name not in callbacks:                         
            log_error("Unknown callback %s"%name)        
            continue                                      
        # 执行任务。
        callbacks[name](*args)
```

任务优先级处理，使用多个队列来实现。
```
def worker_watch_queues(conn, queues, callbacks):   # 实现优先级特性要修改的第一行代码。queues是队列key列表.如['hight','midum','low']
    while not QUIT:
        packed = conn.blpop(queues, 30)             # 实现优先级特性要修改的第二行代码。blpop支持从多个listkey中弹出元素
        if not packed:
            continue

        name, args = json.loads(packed[1])
        if name not in callbacks:
            log_error("Unknown callback %s"%name)
            continue
        callbacks[name](*args)
```

## 延时任务
通过将zset(分值为执行时间)和列表(等待列表)结合来实现。
使用一个进程来循环检查zset是否有可执行任务，有则将其从zset中移除并添加到等待队列中以供执行。
zset存储的成员是一个json串，该json串包含4个值：唯一标识符，处理任务的队列名，回调函数名，回调函数参数。
```
def execute_later(conn, queue, name, args, delay=0):#delay=0表示立即执行
    # 创建唯一标识符。
    identifier = str(uuid.uuid4())                        
    # 准备好需要入队的任务。
    item = json.dumps([identifier, queue, name, args])  
    if delay > 0:
        # 延迟执行这个任务。
        conn.zadd('delayed:', item, time.time() + delay) 
    else:
        # 立即执行这个任务。
        conn.rpush('queue:' + queue, item)                 
    # 返回标识符。
    return identifier 
```

```
def poll_queue(conn):
    while not QUIT:
        # 获取队列中的第一个任务。
        item = conn.zrange('delayed:', 0, 0, withscores=True)   
        # 队列没有包含任何任务，或者任务的执行时间未到。
        if not item or item[0][1] > time.time():               
            #由于有序集合不会像列表那样的阻塞弹出机制，所以休眠时长有待优化从而减少网络请求，可以设置为一段时间都没有发现可执行任务时，自动延长休眠时长。或者是根据下一个任务的执行时间来决定休眠的时长。
            time.sleep(.01)                                    
            continue                                            

        # 解码要被执行的任务，弄清楚它应该被推入到哪个任务队列里面。
        item = item[0][0]                                      
        identifier, queue, function, args = json.loads(item)   

        # 为了对任务进行移动，尝试获取锁。
        locked = acquire_lock(conn, identifier)                
        # 获取锁失败，跳过后续步骤并重试。
        if not locked:                                         
            continue                                          

        # 将任务推入到适当的任务队列里面。
        if conn.zrem('delayed:', item):                       
            conn.rpush('queue:' + queue, item)                 

        # 释放锁。
        release_lock(conn, identifier, locked)      
```

# 消息推送与拉取
两个或多个客户端在互相发送和接受消息的时候，通常会使用以下两种方法来传递消息。第一种为消息推送，也就是由发送者来确保所有接受者已经成功接收到消息。Redis内置了用于消息推送的PUBLISH和SUBCRIBE命令(这两个命令的缺点是必须要求客户端一直处于在线状态才能接收消息，断线可能导致客户端丢失消息)。第二种为消息拉取，这种方式要求客户端自己去获取存储的消息。
由于Redis内置用于消息推送的PUBLISH和SUBCRIBE命令(这两个命令的缺点是必须要求客户端一直处于在线状态才能接收消息，断线可能导致客户端丢失消息)的缺点，所以我们需要寻求自己的解决方案。
## 单一接收者消息的发送与订阅替代
思路：由于是单一接收者，所以每条消息只会被发送至一个客户端。为每个客户端使用一个列表结构，发送者会把消息放到接收者的列表里，客户端从各自列表拉取最新消息。

## 多接收者消息的发送与订阅替代
例如实现类似于聊天群组的功能。
思路：每个新创建的群组都会有一些初始用户，各个用户都可以加入或离开群组。群组使用有序集合(成员为用户名，分值是用户在群组内接收到的最大消息ID)来记录加入群组的用户。用户也会使用有序集合(成员为群组ID，分值是用户在群组内接收到的最大消息ID)来记录自己参加的所有群组。
1、创建群组聊天会话
群组聊天产生的内容会以消息为成员，消息ID为分值的形式存储在有序集合里面。创建群组时，程序首先会对一个全局计数器(ids:chat:)执行自增操作，以此来获取一个新的群组ID。之后，程序会把初始用户添加到一个有序集合('chat:' + chat_id)里，并将分值设为0，另外还会把新群组的ID添加到记录用户已参加群组的有序集合('seen:' + rec)里。最后程序会将一条初始化消息放置到群组有序集合里，以此来向参加聊天的用户发送初始化消息。
```
def create_chat(conn, sender, recipients, message, chat_id=None):
    # 获得新的群组ID。
    chat_id = chat_id or str(conn.incr('ids:chat:'))     

    # 创建一个由用户和分值组成的字典，字典里面的信息将被添加到有序集合里面。
    recipients.append(sender)                           
    recipientsd = dict((r, 0) for r in recipients)       

    pipeline = conn.pipeline(True)
    # 将所有参与群聊的用户添加到有序集合里面。
    pipeline.zadd('chat:' + chat_id, **recipientsd)      
    # 初始化已读有序集合。
    for rec in recipients:                                
        pipeline.zadd('seen:' + rec, chat_id, 0)          
    pipeline.execute()

    # 发送消息。
    return send_message(conn, chat_id, sender, message)  
```

2、发送消息
为了向群组发送消息，程序需要创建一个新的消息ID，并将想要发送的消息添加到群组消息有序集合('msgs:' + chat_id)
```
def send_message(conn, chat_id, sender, message):
    identifier = acquire_lock(conn, 'chat:' + chat_id) 
    if not identifier:
        raise Exception("Couldn't get the lock")
    try:
        # 筹备待发送的消息。
        mid = conn.incr('ids:' + chat_id) 
        ts = time.time()                                 
        packed = json.dumps({                            
            'id': mid,                                   
            'ts': ts,                                   
            'sender': sender,                           
            'message': message,                         
        })                                              

        # 将消息发送至群组。
        conn.zadd('msgs:' + chat_id, packed, mid)     
    finally:
        release_lock(conn, 'chat:' + chat_id, identifier)
    return chat_id
```
一般来说，当程序使用一个来自redis的值去构建另一个将要被添加到redis里面的值时，就需要使用锁或者由watch、multi、exec组成的事务来消除竞争条件。这里使用锁，是因为锁的性能比事务好。

3、获取消息
为了获取用户的所有未读消息，程序需要对记录用户数据的有序集合('seen:' + recipient)执行ZRANGE命令，以此来获取群组ID以及已读消息ID，然后根据这两个ID，对用户参加的所有群组的消息有序集合执行ZRANGEBYSCORE命令，以此来取得用户在各个群组内的未读消息( 'msgs:' + chat_id)。在取得聊天消息之后，程序根据消息ID对已读有序集合(('seen:' + recipient))以及群组有序集合('chat:' + chat_id)里面的用户记录进行更新。最后程序会查找并清除那些已经被所有人接收了的群组消息。
```
def fetch_pending_messages(conn, recipient):
    # 获取最后接收到的消息的ID。
    seen = conn.zrange('seen:' + recipient, 0, -1, withscores=True) 

    pipeline = conn.pipeline(True)

    # 获取所有未读消息。
    for chat_id, seen_id in seen:                              
        pipeline.zrangebyscore(                              
            'msgs:' + chat_id, seen_id+1, 'inf')               
    # 这些数据将被返回给函数调用者。
    chat_info = zip(seen, pipeline.execute())                 

    for i, ((chat_id, seen_id), messages) in enumerate(chat_info):
        if not messages:
            continue
        messages[:] = map(json.loads, messages)
        # 使用最新收到的消息来更新群组有序集合。
        seen_id = messages[-1]['id']                         
        conn.zadd('chat:' + chat_id, recipient, seen_id)       

        # 找出那些所有人都已经阅读过的消息。
        min_id = conn.zrange(                                
            'chat:' + chat_id, 0, 0, withscores=True)          

        # 更新已读消息有序集合。
        pipeline.zadd('seen:' + recipient, chat_id, seen_id)   
        if min_id:
            # 清除那些已经被所有人阅读过的消息。
            pipeline.zremrangebyscore(                        
                'msgs:' + chat_id, 0, min_id[0][1])             
        chat_info[i] = (chat_id, messages)
    pipeline.execute()

    return chat_info
```

4、加入群组和离开群组
```
def join_chat(conn, chat_id, user):
    # 取得最新群组消息的ID。
    message_id = int(conn.get('ids:' + chat_id))            

    pipeline = conn.pipeline(True)
    # 将用户添加到群组成员列表里面。
    pipeline.zadd('chat:' + chat_id, user, message_id)         
    # 将群组添加到用户的已读列表里面。
    pipeline.zadd('seen:' + user, chat_id, message_id)        
    pipeline.execute()
```

```
def leave_chat(conn, chat_id, user):
    pipeline = conn.pipeline(True)
    # 从群组里面移除给定的用户。
    pipeline.zrem('chat:' + chat_id, user)                     
    pipeline.zrem('seen:' + user, chat_id)                     
    # 查看群组剩余成员的数量。
    pipeline.zcard('chat:' + chat_id)                          

    if not pipeline.execute()[-1]:
        # 删除群组。
        pipeline.delete('msgs:' + chat_id)                    
        pipeline.delete('ids:' + chat_id)                     
        pipeline.execute()
    else:
        # 查找那些已经被所有成员阅读过的消息。
        oldest = conn.zrange(                                  
            'chat:' + chat_id, 0, 0, withscores=True)          
        # 删除那些已经被所有成员阅读过的消息。
        conn.zremrangebyscore('msgs:' + chat_id, 0, oldest[0][1])
```

