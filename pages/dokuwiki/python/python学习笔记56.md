title: python学习笔记56 

#  Python学习笔记之异步IO之aiohttp模块客户端部分 
官网:http://aiohttp.readthedocs.org/en/stable/
asyncio是在python3.4中被引进的异步IO库。将会解释你需要知道些什么，以利用它来写异步的代码。
简而言之，有两件事情你需要知道：**协同程序和事件循环。**协同程序像是方法，但是它们可以在代码中的特定点暂停和继续。当在等待一个IO（比如一个HTTP请求），同时执行另一个请求的时候，可以用来暂停一个协同程序。**Python3.4使用关键字yield from来设定一个状态，表明我们需要一个协同程序的返回值。而事件循环则被用来安排协同程序的执行。**
**aiohttp是一个利用Python3.4新增的asyncio的库**，它的API看起来很像请求的API。
特点:
  * Supports both HTTP Client and HTTP Server.
  * Supports both Server WebSockets and Client WebSockets out-of-the-box.
  * Web-server has Middlewares, Signals and pluggable routing.
安装:
```

pip install aiohttp
pip install cchardet (可选，建议安装)

```
##  快速入门以Python3.5asyncio语法 
客户端示例:
```

import asyncio
import aiohttp

async def fetch_page(session, url):
    with aiohttp.Timeout(10):
        async with session.get(url) as response:
            assert response.status == 200
            return await response.read()

loop = asyncio.get_event_loop()
with aiohttp.ClientSession(loop=loop) as session:
    content = loop.run_until_complete(
        fetch_page(session, 'http://python.org'))
    print(content)

```
服务器示例:
```

from aiohttp import web

async def handle(request):
    name = request.match_info.get('name', "Anonymous")
    text = "Hello, " + name
    return web.Response(body=text.encode('utf-8'))

app = web.Application()
app.router.add_route('GET', '/{name}', handle)

web.run_app(app)

```
Python3.4只需要使用 yield from 代替await，使用 @coroutine decorator代替async def即可。例如
```

async def coro(...):
    ret = await f()
#代替为
@asyncio.coroutine
def coro(...):
    ret = yield from f()

```
##  HttpClient 
API：http://aiohttp.readthedocs.org/en/stable/client_reference.html
###  创建一个请求 
```

import aiohttp
with aiohttp.ClientSession() as session:
    async with session.get('https://api.github.com/events') as resp:
        print(resp.status)
        print(await resp.text())

```
 ClientSession called session and a ClientResponse object called resp.
In order to make an HTTP POST request use ClientSession.post() coroutine:
```

session.post('http://httpbin.org/post', data=b'data')
#Other HTTP methods are available as well:
session.put('http://httpbin.org/put', data=b'data')
session.delete('http://httpbin.org/delete')
session.head('http://httpbin.org/get')
session.options('http://httpbin.org/get')
session.patch('http://httpbin.org/patch', data=b'data')

```
###  传递参数 
```

params = {'key1': 'value1', 'key2': 'value2'} #params = [('key', 'value1'), ('key', 'value2')]
async with session.get('http://httpbin.org/get',
                       params=params) as resp:
    assert resp.url == 'http://httpbin.org/get?key2=value2&key1=value1'

```
##  超时设置 
```

with aiohttp.Timeout(0.001):
    async with aiohttp.get('https://github.com') as r:
        await r.text()

```
##  获取相应 
```

async with session.get('https://api.github.com/events') as resp:
    print(await resp.text())
    
#aiohttp会自动解码内容。不过，你也可以自己手动指定text()方法获取的内容编码
await resp.text(encoding='windows-1251')

#二进制响应获取,如果是压缩内容如gzip and deflate 会被自动解压缩
print(await resp.read())

#json响应
print(await resp.json())

#响应码
assert resp.status == 200

#响应头
resp.headers #resp.headers['Content-Type']

#响应cookie
resp.cookies['example_cookie_name']

#历史
resp.history

```
##  流式响应 
使用content属性.它是一个aiohttp.StreamReader对象
```

with open(filename, 'wb') as fd:
    while True:
        chunk = await resp.content.read(chunk_size)
        if not chunk:
            break
        fd.write(chunk)

```
##  释放响应 
千万不要忘记释放响应，这会保证精确的行为和合理的连接池
可以使用with上下文管理器
```

async with session.get(url) as resp:
    pass

```
也可以使用手动释放
```

await resp.release()

```
##  自定义头部 
```

import json
url = 'https://api.github.com/some/endpoint'
payload = {'some': 'data'}
headers = {'content-type': 'application/json'}
await session.post(url,
                   data=json.dumps(payload), #也可以写成 data=payload.区别在于后者会自动进行 form-encoded编码。而前者不会。
                   headers=headers)

```
##  自定义cookie 
```

url = 'http://httpbin.org/cookies'
cookies = dict(cookies_are='working')

async with session.get(url, cookies=cookies) as resp:
    assert await resp.json() == {"cookies":
                                     {"cookies_are": "working"}}

```
##  上传文件POST a Multipart-Encoded File 
```

url = 'http://httpbin.org/post'
files = {'file': open('report.xls', 'rb')}
await session.post(url, data=files)

```
You can set the filename, content_type explicitly:
```

url = 'http://httpbin.org/post'
data = FormData()
data.add_field('file',
               open('report.xls', 'rb'),
               filename='report.xls',
               content_type='application/vnd.ms-excel')

await session.post(url, data=data)

```
##  流式上传 
简单情况下
```

with open('massive-body', 'rb') as f:
   await session.post('http://some.url/streamed', data=f)

```
使用StreamReader对象
```

async def feed_stream(resp, stream):
    h = hashlib.sha256()

    while True:
        chunk = await resp.content.readany()
        if not chunk:
            break
        h.update(chunk)
        s.feed_data(chunk)

    return h.hexdigest()

resp = session.get('http://httpbin.org/post')
stream = StreamReader()
loop.create_task(session.post('http://httpbin.org/post', data=stream))

file_hash = await feed_stream(resp, stream)

```
##  上传压缩数据 
```

async def my_coroutine(session, headers, my_data):
    data = zlib.compress(my_data)
    headers = {'Content-Encoding': 'deflate'}
    async with session.post('http://httpbin.org/post',
                            data=data,
                            headers=headers,
                            compress=False):
        pass

```
##  Keep-Alive, connection pooling and cookie sharing 
ClientSession may be used for sharing cookies between multiple requests:

```

session = aiohttp.ClientSession()
await session.post(
     'http://httpbin.org/cookies/set/my_cookie/my_value')
async with session.get('http://httpbin.org/cookies') as r:
    json = await r.json()
    assert json['cookies']['my_cookie'] == 'my_value'

```
You also can set default headers for all session requests:

```

session = aiohttp.ClientSession(
    headers={"Authorization": "Basic bG9naW46cGFzcw=="})
async with s.get("http://httpbin.org/headers") as r:
    json = yield from r.json()
    assert json['headers']['Authorization'] == 'Basic bG9naW46cGFzcw=='

```
ClientSession supports keep-alive requests and connection pooling out-of-the-box.

##  WebSockets 
```

session = aiohttp.ClientSession()
async with session.ws_connect('http://example.org/websocket') as ws:

    async for msg in ws:
        if msg.tp == aiohttp.MsgType.text:
            if msg.data == 'close cmd':
                await ws.close()
                break
            else:
                ws.send_str(msg.data + '/answer')
        elif msg.tp == aiohttp.MsgType.closed:
            break
        elif msg.tp == aiohttp.MsgType.error:
            break

```
##  SSL control for TCP sockets     
**TCPConnector** constructor accepts mutually exclusive verify_ssl and ssl_context params.
默认使用严格的https检查。可以通过设置verify_ssl=False来忽略证书检查
```

conn = aiohttp.TCPConnector(verify_ssl=False)
session = aiohttp.ClientSession(connector=conn)
r = await session.get('https://example.com')

```
其余http://aiohttp.readthedocs.org/en/stable/client.html
##  代理支持 
略。http://aiohttp.readthedocs.org/en/stable/client.html