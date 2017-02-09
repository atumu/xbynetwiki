author:xbynet
title:使用Redis构建简单的社交网站
modifyAt:2016-12-25 14:43:41
location:db/使用Redis构建简单的社交网站
createAt:2016-12-25 14:35:49

本文将考虑创建一个类似twitter那样的社交网站。一般社交网站会有一些基本东西：主页时间线(home timeline)、正在关注列表(following list)、被关注列表(follower list)、发布删除状态消息、关注取消关注某人等、开放API。

# 用户和状态
## 用户信息
使用散列来存储用户信息，这些信息包括用户显示名，用户登录名，用户关注人数，用户被关注人数，用户发布状态消息数，用户的注册日期等。
首先定义锁
```
def acquire_lock_with_timeout(
    conn, lockname, acquire_timeout=10, lock_timeout=10):
    identifier = str(uuid.uuid4())                      #A
    lockname = 'lock:' + lockname
    lock_timeout = int(math.ceil(lock_timeout))         #D

    end = time.time() + acquire_timeout
    while time.time() < end:
        if conn.setnx(lockname, identifier):            #B
            conn.expire(lockname, lock_timeout)         #B
            return identifier
        elif not conn.ttl(lockname):                    #C
            conn.expire(lockname, lock_timeout)         #C

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

    return False                                  #D

```

**创建用户帐号**(不考虑在这里存储用户密码等敏感信息)：
这里涉及两个散列:'users:'，'user:%s'%id；一个全局用户ID计数键'user:id:'；一个锁'user:' + llogin
```
def create_user(conn, login, name):
    llogin = login.lower()
    # 使用第 6 章定义的加锁函数尝试对小写的用户名进行加锁。
    lock = acquire_lock_with_timeout(conn, 'user:' + llogin, 1) 
    # 如果加锁不成功，那么说明给定的用户名已经被其他用户占用了。
    if not lock:                           
        return None                        

    # 程序使用了一个散列来储存小写的用户名以及用户 ID 之间的映射，
    # 如果给定的用户名已经被映射到了某个用户 ID ，
    # 那么程序就不会再将这个用户名分配给其他人。
    if conn.hget('users:', llogin):        
        release_lock(conn, 'user:' + llogin, lock) 
        return None                     

    # 每个用户都有一个独一无二的 ID ，
    # 这个 ID 是通过对计数器执行自增操作产生的。
    id = conn.incr('user:id:')              
    pipeline = conn.pipeline(True)
    # 在散列里面将小写的用户名映射至用户 ID 。
    pipeline.hset('users:', llogin, id)    
    # 将用户信息添加到用户对应的散列里面。
    pipeline.hmset('user:%s'%id, {          
        'login': login,                     
        'id': id,                           
        'name': name,                       
        'followers': 0,                     
        'following': 0,                     
        'posts': 0,                         
        'signup': time.time(),              
    })
    pipeline.execute()
    # 释放之前对用户名加的锁。
    release_lock(conn, 'user:' + llogin, lock)  
    # 返回用户 ID 。
    return id 
```

## 状态消息
使用一个散列来存储状态消息：消息内容，消息id，消息发布者id和用户名，其他等
涉及一个全局消息计数ID键：'status:id:'，一个消息散列'status:%s'%id，一个用户信息散列'user:%s'%uid
```
def create_status(conn, uid, message, **data):
    pipeline = conn.pipeline(True)
    # 根据用户 ID 获取用户的用户名。
    pipeline.hget('user:%s'%uid, 'login')   
    # 为这条状态消息创建一个新的 ID 。
    pipeline.incr('status:id:')            
    login, id = pipeline.execute()

    # 在发布状态消息之前，先检查用户的账号是否存在。
    if not login:                           
        return None                         

    # 准备并设置状态消息的各项信息。
    data.update({
        'message': message,                
        'posted': time.time(),            
        'id': id,                          
        'uid': uid,                        
        'login': login,                   
    })
    pipeline.hmset('status:%s'%id, data)   
    # 更新用户的已发送状态消息数量。
    pipeline.hincrby('user:%s'%uid, 'posts')
    pipeline.execute()
    # 返回新创建的状态消息的 ID 。
    return id  
```

# 主页时间线
用户在登录的情况下，首先看到的是他们自己的主页时间线，这个时间线由用户以及用户正在关注的人所发布的状态消息组成。我们希望能够尽快地获取展示一个页面所需的全部数据，因此我们决定使用**有序集合**来实现主页时间线，这个有序集合的成员为消息ID，分值为发布时间撮。

```
# 函数接受三个可选参数，
# 它们分别用于指定函数要获取哪条时间线、要获取多少页时间线、以及每页要有多少条状态消息。
def get_status_messages(conn, uid, timeline='home:', page=1, count=30):
    # 获取时间线上面最新的状态消息的 ID 。
    statuses = conn.zrevrange(                                  
        '%s%s'%(timeline, uid), (page-1)*count, page*count-1)   

    pipeline = conn.pipeline(True)
    # 获取状态消息本身。
    for id in statuses:                                         
        pipeline.hgetall('status:%s'%id)                        

    # 使用过滤器移除那些已经被删除了的状态消息。
    return filter(None, pipeline.execute())
```

除了主页时间线，还应该有个人时间线：区别是前者包含其他人的状态消息，后者仅包含个人状态消息。
要获取个人时间线只是传参不同即可。

# 关注者列表和正在关注列表
使用**有序集合**来存储关注者列表和正在关注列表。有序集合成员为用户ID，分值为关注或被关注时的时间撮。

```
HOME_TIMELINE_SIZE = 1000
def follow_user(conn, uid, other_uid):
    # 把正在关注有序集合以及关注者有序集合的键名缓存起来。
    fkey1 = 'following:%s'%uid          
    fkey2 = 'followers:%s'%other_uid    

    # 如果 uid 指定的用户已经关注了 other_uid 指定的用户，那么函数直接返回。
    if conn.zscore(fkey1, other_uid):   
        return None                    

    now = time.time()

    pipeline = conn.pipeline(True)
    # 将两个用户的 ID 分别添加到相应的正在关注有序集合以及关注者有序集合里面。
    pipeline.zadd(fkey1, other_uid, now)    
    pipeline.zadd(fkey2, uid, now)          
    # 从被关注用户的个人时间线里面获取 HOME_TIMELINE_SIZE 条最新的状态消息。
    pipeline.zrevrange('profile:%s'%other_uid,      
        0, HOME_TIMELINE_SIZE-1, withscores=True)   
    following, followers, status_and_score = pipeline.execute()[-3:]

    # 修改两个用户的散列，更新他们各自的正在关注数量以及关注者数量。
    pipeline.hincrby('user:%s'%uid, 'following', int(following))        
    pipeline.hincrby('user:%s'%other_uid, 'followers', int(followers))  
    if status_and_score:
        # 对执行关注操作的用户的定制时间线进行更新，并保留时间线上面的最新 1000 条状态消息。
        pipeline.zadd('home:%s'%uid, **dict(status_and_score))  
    pipeline.zremrangebyrank('home:%s'%uid, 0, -HOME_TIMELINE_SIZE-1)

    pipeline.execute()
    # 返回 True 表示关注操作已经成功执行。
    return True         
```

```
def unfollow_user(conn, uid, other_uid):
    # 把正在关注有序集合以及关注者有序集合的键名缓存起来。
    fkey1 = 'following:%s'%uid          
    fkey2 = 'followers:%s'%other_uid    

    # 如果 uid 指定的用户并未关注 other_uid 指定的用户，那么函数直接返回。
    if not conn.zscore(fkey1, other_uid):   
        return None                         

    pipeline = conn.pipeline(True)
    # 从正在关注有序集合以及关注者有序集合里面移除双方的用户 ID 。
    pipeline.zrem(fkey1, other_uid)                
    pipeline.zrem(fkey2, uid)                      
    # 获取被取消关注的用户最近发布的 HOME_TIMELINE_SIZE 条状态消息。
    pipeline.zrevrange('profile:%s'%other_uid,     
        0, HOME_TIMELINE_SIZE-1)                   
    following, followers, statuses = pipeline.execute()[-3:]

    # 对用户信息散列里面的正在关注数量以及关注者数量进行更新。
    pipeline.hincrby('user:%s'%uid, 'following', int(following))        
    pipeline.hincrby('user:%s'%other_uid, 'followers', int(followers))  
    if statuses:
        # 对执行取消关注操作的用户的定制时间线进行更新，
        # 移除被取消关注的用户发布的所有状态消息。
        pipeline.zrem('home:%s'%uid, *statuses)                

    pipeline.execute()
    # 返回 True 表示取消关注操作执行成功。
    return True      
```

# 发布和删除状态消息
```
def post_status(conn, uid, message, **data):
    # 使用之前介绍过的函数来创建一条新的状态消息。
    id = create_status(conn, uid, message, **data)  
    # 如果创建状态消息失败，那么直接返回。
    if not id:              
        return None         

    # 获取消息的发布时间。
    posted = conn.hget('status:%s'%id, 'posted')    
    # 如果程序未能顺利地获取消息的发布时间，那么直接返回。
    if not posted:                                  
        return None                                 

    post = {str(id): float(posted)}
    # 将状态消息添加到用户的个人时间线里面。
    conn.zadd('profile:%s'%uid, **post)            

    # 将状态消息推送给用户的关注者。
    syndicate_status(conn, uid, post)     
    return id
```

```
# 函数每次被调用时，最多只会将状态消息发送给一千个关注者。
POSTS_PER_PASS = 1000          
def syndicate_status(conn, uid, post, start=0):
    # 以上次被更新的最后一个关注者为起点，获取接下来的一千个关注者。
    followers = conn.zrangebyscore('followers:%s'%uid, start, 'inf',
        start=0, num=POSTS_PER_PASS, withscores=True)  

    pipeline = conn.pipeline(False)
    # 在遍历关注者的同时，
    # 对 start 变量的值进行更新，
    # 这个变量可以在有需要的时候传递给下一个 syndicate_status() 调用。
    for follower, start in followers:                   
        # 将状态消息添加到所有被获取的关注者的定制时间线里面，
        # 并在有需要的时候对关注者的定制时间线进行修剪，
        # 防止它超过限定的最大长度。
        pipeline.zadd('home:%s'%follower, **post)        
        pipeline.zremrangebyrank(                        
            'home:%s'%follower, 0, -HOME_TIMELINE_SIZE-1)
    pipeline.execute()

    # 如果需要更新的关注者数量超过一千人，
    # 那么在延迟任务里面继续执行剩余的更新操作。
    if len(followers) >= POSTS_PER_PASS:                    
        execute_later(conn, 'default', 'syndicate_status',  
            [conn, uid, post, start])
```

```
def execute_later(conn, queue, name, args):
    # this is just for testing purposes
    assert conn is args[0]
    t = threading.Thread(target=globals()[name], args=tuple(args))
    t.setDaemon(1)
    t.start()
```

```
def delete_status(conn, uid, status_id):
    key = 'status:%s'%status_id
    # 对指定的状态消息进行加锁，防止两个程序同时删除同一条状态消息的情况出现。
    lock = acquire_lock_with_timeout(conn, key, 1)  
    # 如果加锁失败，那么直接返回。
    if not lock:                
        return None             

    # 如果 uid 指定的用户并非状态消息的发布人，那么函数直接返回。
    if conn.hget(key, 'uid') != str(uid):   
        release_lock(conn, key, lock)       
        return None                        

    pipeline = conn.pipeline(True)
    # 删除指定的状态消息。
    pipeline.delete(key)                            
    # 从用户的个人时间线里面移除指定的状态消息 ID 。
    pipeline.zrem('profile:%s'%uid, status_id)      
    # 从用户的定制时间线里面移除指定的状态消息 ID 。
    pipeline.zrem('home:%s'%uid, status_id)        
    # 对储存着用户信息的散列进行更新，减少已发布状态消息的数量。
    pipeline.hincrby('user:%s'%uid, 'posts', -1)   
    pipeline.execute()

    release_lock(conn, key, lock)
    return True
```

# 开放流式API
绝大多数web服务器在执行操作的时候都会假定程序会一次性地将全部回复返回给请求，然后这并不适用于流式API。可用的解决方案：WebSockets、SPDY等，但是浏览器兼容目前并不是很全(尤其是IE)。我们可以采用**分块(chunked)传输编码(('Transfer-Encoding', 'chunked')**，这样我们就可以使用HTTP服务器生成并发送**增量式流式数据**。

## 实现简单的HTTP流服务器
```
# 创建一个名为 StreamingAPIServer 的类。
class StreamingAPIServer(               
    # 这个类是一个 HTTP 服务器，
    # 并且它具有为每个请求创建一个新线程的能力。
    SocketServer.ThreadingMixIn,        
    BaseHTTPServer.HTTPServer):         

    # 让线程服务器内部组件在主服务器线程死亡（die）之后，
    # 关闭所有客户端请求线程。
    daemon_threads = True              

# 创建一个名为 StreamingAPIRequestHandler 的类。
class StreamingAPIRequestHandler(              
    # 这个新创建的类可以用于处理 HTTP 请求。
    BaseHTTPServer.BaseHTTPRequestHandler):    

    # 创建一个名为 do_GET() 的方法，用于处理服务器接收到的 GET 请求。
    def do_GET(self):                                      
        # 调用辅助函数，获取客户端标识符。
        parse_identifier(self)                             
        # 如果这个 GET 请求访问的不是 sample 流或者 firehose 流，
        # 那么返回“404 页面未找到”错误。
        if self.path != '/statuses/sample.json':           
            return self.send_error(404)                    

        # 如果一切顺利，那么调用辅助函数，执行实际的过滤工作。
        process_filters(self)                              

    # 创建一个名为 do_POST() 的方法，用于处理服务器接收到的 POST 请求。
    def do_POST(self):                                     
        # 调用辅助函数，获取客户端标识符。
        parse_identifier(self)                             
        # 如果这个 POST 请求访问的不是用户过滤器、关键字过滤器或者位置过滤器，
        # 那么返回“404 页面未找到”错误。
        if self.path != '/statuses/filter.json':           
            return self.send_error(404)                    

        # 如果一切顺利，那么调用辅助函数，执行实际的过滤工作。
        process_filters(self)  
```

启动服务器
```
if __name__ == '__main__':
    server=StreamingAPIServer(('localhost',8080),StreamingAPIRequestHandler)
    print('Starting server,use <Ctrl-C> to stop')
    server.serve_forever()
```

```
def parse_identifier(handler):
    # 将标识符和查询参数设置为预留值。
    handler.identifier = None       
    handler.query = {}              
    # 如果请求里面包含了查询参数，那么处理这些参数。
    if '?' in handler.path:        
        # 取出路径里面包含查询参数的部分，并对路径进行更新。
        handler.path, _, query = handler.path.partition('?')    
        # 通过语法分析得出查询参数。
        handler.query = urlparse.parse_qs(query)               
        # 获取名为 identifier 的查询参数列表。
        identifier = handler.query.get('identifier') or [None]  
        # 使用第一个传入的标识符。
        handler.identifier = identifier[0]                      

# 把需要传入参数的过滤器都放到一个列表里面。
FILTERS = ('track', 'filter', 'location')                  
def process_filters(handler):
    id = handler.identifier
    # 如果客户端没有提供标识符，那么返回一个错误。
    if not id:                                              
        return handler.send_error(401, "identifier missing")

    # 获取客户端指定的方法，
    # 结果应该是 sample （随机消息）或者 filter （过滤器）这两种的其中一种。
    method = handler.path.rsplit('/')[-1].split('.')[0]    
    name = None
    args = None
    # 如果客户端指定的是过滤器方法，那么程序需要获取相应的过滤参数。
    if method == 'filter':                                 
        # 对 POST 请求进行语法分析，从而获知过滤器的类型以及参数。
        data = cgi.FieldStorage(                               
            fp=handler.rfile,                                   
            headers=handler.headers,                            
            environ={'REQUEST_METHOD':'POST',                   
                     'CONTENT_TYPE':handler.headers['Content-Type'],
        })

        # 找到客户端在请求中指定的过滤器。
        for name in data:                              
            if name in FILTERS:                         
                args = data.getfirst(name).lower().split(',')   
                break                                   

        # 如果客户端没有指定任何过滤器，那么返回一个错误。
        if not args:                                           
            return handler.send_error(401, "no filter provided")
    else:
        # 如果客户端指定的是随机消息请求，那么将查询参数用作 args 变量的值。
        args = handler.query                                

    # 最后，向客户端返回一个回复，
    # 告知客户端，服务器接下来将向它发送流回复。
    handler.send_response(200)                              
    handler.send_header('Transfer-Encoding', 'chunked')     
    handler.end_headers()

    # 使用 Python 列表来做引用传递（pass-by-reference）变量的占位符，
    # 用户可以通过这个变量来让内容过滤器停止接收消息。
    quit = [False]                                         
    # 对过滤结果进行迭代。
    for item in filter_content(id, method, name, args, quit):   
        try:
            #  使用分块传输编码向客户端发送经过预编码后（pre-encoded）的回复。
            handler.wfile.write('%X\r\n%s\r\n'%(len(item), item))   
        # 如果发送操作引发了错误，那么让订阅者停止订阅并关闭自身。
        except socket.error:                                   
            quit[0] = True                                      
    if not quit[0]:
        # 如果服务器与客户端的连接并未断开，
        # 那么向客户端发送表示“分块到此结束”的消息。
        handler.wfile.write('0\r\n\r\n')                        

```

```
def create_status(conn, uid, message, **data):
    pipeline = conn.pipeline(True)
    pipeline.hget('user:%s'%uid, 'login')
    pipeline.incr('status:id:')
    login, id = pipeline.execute()

    if not login:
        return None

    data.update({
        'message': message,
        'posted': time.time(),
        'id': id,
        'uid': uid,
        'login': login,
    })
    pipeline.hmset('status:%s'%id, data)
    pipeline.hincrby('user:%s'%uid, 'posts')
    # 新添加的这一行代码用于向流过滤器发送消息。
    pipeline.publish('streaming:status:', json.dumps(data)) 
    pipeline.execute()
    return id

_delete_status = delete_status

def delete_status(conn, uid, status_id):
    key = 'status:%s'%status_id
    lock = acquire_lock_with_timeout(conn, key, 1)
    if not lock:
        return None

    if conn.hget(key, 'uid') != str(uid):
        release_lock(conn, key, lock)
        return None

    pipeline = conn.pipeline(True)
    # 获取状态消息，
    # 以便流过滤器可以通过执行相同的过滤器来判断是否需要将被删除的消息传递给客户端。
    status = conn.hgetall(key)                                 
    # 将状态消息标记为“已被删除”。
    status['deleted'] = True                                   
    # 将已被删除的状态消息发送到流里面。
    pipeline.publish('streaming:status:', json.dumps(status))   
    pipeline.delete(key)
    pipeline.zrem('profile:%s'%uid, status_id)
    pipeline.zrem('home:%s'%uid, status_id)
    pipeline.hincrby('user:%s'%uid, 'posts', -1)
    pipeline.execute()

    release_lock(conn, key, lock)
    return True

# 使用第 5 章介绍的自动连接装饰器。
@redis_connection('social-network')                        
def filter_content(conn, id, method, name, args, quit):
    # 创建一个过滤器，让它来判断是否应该将消息发送给客户端。
    match = create_filters(id, method, name, args)         

    # 执行订阅前的准备工作。
    pubsub = conn.pubsub()                     
    pubsub.subscribe(['streaming:status:'])    

    # 通过订阅来获取消息。
    for item in pubsub.listen():                
        # 从订阅结构中取出状态消息。
        message = item['data']                  
        decoded = json.loads(message)           

        # 检查状态消息是否与过滤器相匹配。
        if match(decoded):                     
            # 在发送被删除的消息之前，
            # 先给消息添加一个特殊的“已被删除”占位符。
            if decoded.get('deleted'):                      
                yield json.dumps({                          
                    'id': decoded['id'], 'deleted': True})  
            else:
                # 对于未被删除的消息，程序直接发送消息本身。
                yield message                  

        # 如果服务器与客户端之间的连接已经断开，那么停止过滤消息。
        if quit[0]:                             
            break                               

    # 重置 Redis 连接，
    # 清空因为连接速度不够快而滞留在 Redis 服务器输出缓冲区里面的数据。
    pubsub.reset()                             

def create_filters(id, method, name, args):
    # sample 方法不需要用到 name 参数，
    # 只需要给定 id 参数和 args 参数即可。
    if method == 'sample':                     
        return SampleFilter(id, args)          
    elif name == 'track':                       # filter 方法需要创建并返回用户指定的过滤器。
        return TrackFilter(args)                #
    elif name == 'follow':                      #
        return FollowFilter(args)               #
    elif name == 'location':                    #
        return LocationFilter(args)             #
    # 如果没有任何过滤器被选中，那么引发一个异常。
    raise Exception("Unknown filter")          

# 定义一个 SampleFilter 函数，它接受 id 和 args 两个参数。
def SampleFilter(id, args):                             
    # args 参数是一个字典，它来源于 GET 请求传递的参数。
    percent = int(args.get('percent', ['10'])[0], 10)   
    # 使用 id 参数来随机地选择其中一部分消息 ID ，
    # 被选中 ID 的数量由传入的 percent 参数决定。
    ids = range(100)                                    
    shuffler = random.Random(id)                        
    shuffler.shuffle(ids)                               
    # 使用 Python 集合来快速地判断给定的状态消息是否符合过滤器的标准。
    keep = set(ids[:max(percent, 1)])                  

    # 创建并返回一个闭包函数，
    # 这个函数就是被创建出来的随机取样消息过滤器。
    def check(status):                                  
        # 为了对状态消息进行过滤，
        # 程序会获取给定状态消息的 ID ，
        # 并将 ID 的值取模 100 ，
        # 然后通过检查取模结果是否存在于 keep 集合来判断给定的状态消息是否符合过滤器的标准。
        return (status['id'] % 100) in keep             
    return check

def TrackFilter(list_of_strings):
    # 函数接受一个由词组构成的列表为参数，
    # 如果一条状态消息包含某个词组里面的所有单词，
    # 那么这条消息就与过滤器相匹配。
    groups = []                                
    for group in list_of_strings:              
        group = set(group.lower().split())     
        if group:
            # 每个词组至少需要包含一个单词。
            groups.append(group)                

    def check(status):
        # 以空格为分隔符，从消息里面分割出多个单词。
        message_words = set(status['message'].lower().split())  
        # 遍历所有词组。
        for group in groups:                               
            # 如果某个词组的所有单词都在消息里面出现了，
            # 那么过滤器将接受（accept）这条消息。
            if len(group & message_words) == len(group):    
                return True                                 
        return False
    return check

def FollowFilter(names):
    # 过滤器会根据给定的用户名，对消息内容以及消息的发送者进行匹配。
    nset = set()                                 
    # 以“@用户名”的形式储存所有给定用户的名字。
    for name in names:                              
        nset.add('@' + name.lower().lstrip('@'))   

    def check(status):
        # 根据消息内容以及消息发布者的名字，构建一个由空格分割的词组。
        message_words = set(status['message'].lower().split())  
        message_words.add('@' + status['login'].lower())        

        # 如果给定的用户名与词组中的某个词语相同，
        # 那么这条消息与过滤器相匹配。
        return message_words & nset                            
    return check

def LocationFilter(list_of_boxes):
    # 创建一个区域集合，这个集合定义了过滤器接受的消息来自于哪些区域。
    boxes = []                                                 
    for start in xrange(0, len(list_of_boxes)-3, 4):            
        boxes.append(map(float, list_of_boxes[start:start+4]))  

    def check(self, status):
        # 尝试从状态消息里面取出位置数据。
        location = status.get('location')           
        # 如果消息未包含任何位置数据，
        # 那么这条消息不在任何区域的范围之内。
        if not location:                            
            return False                           

        # 如果消息包含位置数据，那么取出纬度和经度。
        lat, lon = map(float, location.split(','))  
        # 遍历所有区域，尝试进行匹配。
        for box in self.boxes:                      
            # 如果状态消息的位置在给定区域的经纬度范围之内，
            # 那么这条状态消息与过滤器相匹配。
            if (box[1] <= lat <= box[3] and         
                box[0] <= lon <= box[2]):           
                return True                         
        return False
    return check

_filter_content = filter_content
def filter_content(identifier, method, name, args, quit):
    print "got:", identifier, method, name, args
    for i in xrange(10):
        yield json.dumps({'id':i})
        if quit[0]:
            break
        time.sleep(.1)
```


# 单元测试
```
class TestCh08(unittest.TestCase):
    def setUp(self):
        self.conn = redis.Redis(db=15)
        self.conn.flushdb()
    def tearDown(self):
        self.conn.flushdb()

    def test_create_user_and_status(self):
        self.assertEquals(create_user(self.conn, 'TestUser', 'Test User'), 1)
        self.assertEquals(create_user(self.conn, 'TestUser', 'Test User2'), None)

        self.assertEquals(create_status(self.conn, 1, "This is a new status message"), 1)
        self.assertEquals(self.conn.hget('user:1', 'posts'), '1')

    def test_follow_unfollow_user(self):
        self.assertEquals(create_user(self.conn, 'TestUser', 'Test User'), 1)
        self.assertEquals(create_user(self.conn, 'TestUser2', 'Test User2'), 2)
        
        self.assertTrue(follow_user(self.conn, 1, 2))
        self.assertEquals(self.conn.zcard('followers:2'), 1)
        self.assertEquals(self.conn.zcard('followers:1'), 0)
        self.assertEquals(self.conn.zcard('following:1'), 1)
        self.assertEquals(self.conn.zcard('following:2'), 0)
        self.assertEquals(self.conn.hget('user:1', 'following'), '1')
        self.assertEquals(self.conn.hget('user:2', 'following'), '0')
        self.assertEquals(self.conn.hget('user:1', 'followers'), '0')
        self.assertEquals(self.conn.hget('user:2', 'followers'), '1')

        self.assertEquals(unfollow_user(self.conn, 2, 1), None)
        self.assertEquals(unfollow_user(self.conn, 1, 2), True)
        self.assertEquals(self.conn.zcard('followers:2'), 0)
        self.assertEquals(self.conn.zcard('followers:1'), 0)
        self.assertEquals(self.conn.zcard('following:1'), 0)
        self.assertEquals(self.conn.zcard('following:2'), 0)
        self.assertEquals(self.conn.hget('user:1', 'following'), '0')
        self.assertEquals(self.conn.hget('user:2', 'following'), '0')
        self.assertEquals(self.conn.hget('user:1', 'followers'), '0')
        self.assertEquals(self.conn.hget('user:2', 'followers'), '0')
        
    def test_syndicate_status(self):
        self.assertEquals(create_user(self.conn, 'TestUser', 'Test User'), 1)
        self.assertEquals(create_user(self.conn, 'TestUser2', 'Test User2'), 2)
        self.assertTrue(follow_user(self.conn, 1, 2))
        self.assertEquals(self.conn.zcard('followers:2'), 1)
        self.assertEquals(self.conn.hget('user:1', 'following'), '1')
        self.assertEquals(post_status(self.conn, 2, 'this is some message content'), 1)
        self.assertEquals(len(get_status_messages(self.conn, 1)), 1)

        for i in xrange(3, 11):
            self.assertEquals(create_user(self.conn, 'TestUser%s'%i, 'Test User%s'%i), i)
            follow_user(self.conn, i, 2)

        global POSTS_PER_PASS
        POSTS_PER_PASS = 5
        
        self.assertEquals(post_status(self.conn, 2, 'this is some other message content'), 2)
        time.sleep(.1)
        self.assertEquals(len(get_status_messages(self.conn, 9)), 2)

        self.assertTrue(unfollow_user(self.conn, 1, 2))
        self.assertEquals(len(get_status_messages(self.conn, 1)), 0)

    def test_refill_timeline(self):
        self.assertEquals(create_user(self.conn, 'TestUser', 'Test User'), 1)
        self.assertEquals(create_user(self.conn, 'TestUser2', 'Test User2'), 2)
        self.assertEquals(create_user(self.conn, 'TestUser3', 'Test User3'), 3)
        
        self.assertTrue(follow_user(self.conn, 1, 2))
        self.assertTrue(follow_user(self.conn, 1, 3))

        global HOME_TIMELINE_SIZE
        HOME_TIMELINE_SIZE = 5
        
        for i in xrange(10):
            self.assertTrue(post_status(self.conn, 2, 'message'))
            self.assertTrue(post_status(self.conn, 3, 'message'))
            time.sleep(.05)

        self.assertEquals(len(get_status_messages(self.conn, 1)), 5)
        self.assertTrue(unfollow_user(self.conn, 1, 2))
        self.assertTrue(len(get_status_messages(self.conn, 1)) < 5)

        refill_timeline(self.conn, 'following:1', 'home:1')
        messages = get_status_messages(self.conn, 1)
        self.assertEquals(len(messages), 5)
        for msg in messages:
            self.assertEquals(msg['uid'], '3')
        
        delete_status(self.conn, '3', messages[-1]['id'])
        self.assertEquals(len(get_status_messages(self.conn, 1)), 4)
        self.assertEquals(self.conn.zcard('home:1'), 5)
        clean_timelines(self.conn, '3', messages[-1]['id'])
        self.assertEquals(self.conn.zcard('home:1'), 4)

if __name__ == '__main__':
    unittest.main()
```