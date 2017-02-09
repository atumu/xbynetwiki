title: python学习笔记15 

#  Python学习笔记之HTTP请求 
##  urllib.parse 
urllib.parse模块是一个解析与反解析Web网址URL字符串的一个工具。
比较典型的函数有:
urllib.parse.urlparse函数会将一个普通的url解析为6个部分，返回的数据类型为ParseResult对象，通过访问其属性可以获取对应的值。
同时，它还可以将已经分解后的url再组合成一个url地址。返回的6个部分，分别是：scheme(机制)、netloc(网络位置)、path(路径)、params(路径段参数)、query(查询)、fragment(片段)。
如:
二、urllib.parse模块方法
1 )、 urlparse(url)，分解url返回ParseResult对象，可以得到很多关于这个url的数据，网络协议、目录层次等。
2 )、 urlunparse(parts)，它接收一个元组类型，将元组内对应元素重新组后为一个url网址，与上面功能正好相反。
3 )、 urlsplit(url)，作用与urlparse非常相似，它不会分解url参数，对于遵循RFC2396的URL很有用处。
4 )、 urljoin(base, url ) 功能是基于一个base url和另一个url构造一个绝对URL。
5)、urlencode(query, doseq=False, safe=' ', encoding=None, errors=None),注意：query参数是一个序列对象


##  标准库中的urllib 
urllib提供了一系列用于操作URL的功能。
###  Get 
urllib的request模块可以非常方便地抓取URL内容，也就是发送一个GET请求到指定的页面，然后返回HTTP的响应：
例如，对豆瓣的一个URL https://api.douban.com/v2/book/2129650进行抓取，并返回响应：
```

from urllib import request
with request.urlopen('https://api.douban.com/v2/book/2129650') as f:
    data = f.read()
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', data.decode('utf-8'))

```
可以看到HTTP响应的头和JSON数据：
如果我们要想模拟浏览器发送GET请求，就需要使用Request对象，通过往Request对象添加HTTP头User-Agent，我们就可以把请求**伪装成浏览器**。例如，模拟iPhone 6去请求豆瓣首页：
```

from urllib import request
req = request.Request('http://www.douban.com/') #设置请求头
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25')
with request.urlopen(req) as f:
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', f.read().decode('utf-8'))

```
这样豆瓣会返回适合iPhone的移动版网页：
###  Post 
如果要以POST发送一个请求，只需要把参数data以bytes形式传入。
我们模拟一个微博登录，先读取登录的邮箱和口令，然后按照weibo.cn的登录页的格式以username=xxx&password=xxx的编码传入：
```

from urllib import request, parse
print('Login to weibo.cn...')
email = input('Email: ')
passwd = input('Password: ')
login_data = parse.urlencode([ #进行url编码参数
    ('username', email),
    ('password', passwd),
    ('entry', 'mweibo'),
    ('client_id', ''),
    ('savestate', '1'),
    ('ec', ''),
    ('pagerefer', 'https://passport.weibo.cn/signin/welcome?entry=mweibo&r=http%3A%2F%2Fm.weibo.cn%2F')
])
req = request.Request('https://passport.weibo.cn/sso/login') 
req.add_header('Origin', 'https://passport.weibo.cn')
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25')
req.add_header('Referer', 'https://passport.weibo.cn/signin/login?entry=mweibo&res=wel&wm=3349&r=http%3A%2F%2Fm.weibo.cn%2F')
with request.urlopen(req, data=login_data.encode('utf-8')) as f:
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', f.read().decode('utf-8'))

```
###  代理处理 
**如果还需要更复杂的控制，比如通过一个Proxy去访问网站，我们需要利用` ProxyHandler `来处理**，示例代码如下：
```

proxy_handler = urllib.request.ProxyHandler({'http': 'http://www.example.com:3128/'})
proxy_auth_handler = urllib.request.ProxyBasicAuthHandler()
proxy_auth_handler.add_password('realm', 'host', 'username', 'password')
opener = urllib.request.build_opener(proxy_handler, proxy_auth_handler)
with opener.open('http://www.example.com/login.html') as f:
    pass

```
小结
urllib提供的功能就是利用程序去执行各种HTTP请求。如果要模拟浏览器完成特定功能，需要把请求伪装成浏览器。伪装的方法是先监控浏览器发出的请求，再根据浏览器的请求头来伪装，User-Agent头就是用来标识浏览器的。

参考：http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432688314740a0aed473a39f47b09c8c7274c9ab6aee000

##  requests库 
安装:pip install requests
官网:http://www.python-requests.org/en/master/
前面介绍了urllib标准库的模块，与之有类似功能的第三方库中requests也是一个用于在程序中进行http协议下的get和post请求的模块，并且被网友说成“好用的要哭”。
```

>>> dir(requests)
['ConnectionError', 'FileModeWarning', 'HTTPError', 'NullHandler', 'PreparedRequest', 'Request', 'RequestException', 'Response', 'Session', 'Timeout', 'TooManyRedirects', 'URLRequired', '__author__', '__build__', '__builtins__', '__cached__', '__copyright__', '__doc__', '__file__', '__license__', '__loader__', '__name__', '__package__', '__path__', '__spec__', '__title__', '__version__', 'adapters', 'api', 'auth', 'certs', 'codes', 'compat', 'cookies', 'delete', 'exceptions', 'get', 'head', 'hooks', 'logging', 'models', 'options', 'packages', 'patch', 'post', 'put', 'request', 'session', 'sessions', 'status_codes', 'structures', 'utils', 'warnings']
#包括cookies、get、head、post、session等常用功能属性

```
###  get请求 
```

>>> r = requests.get("http://www.1world0x00.com")
>>> r.cookies
<<class 'requests.cookies.RequestsCookieJar'>[Cookie(version=0, name='PHPSESSID', value='buqj70k7f9rrg51emsvatveda2',。。。>

```
原来这样呀。继续，还有别的属性可以看看。
```

>>> r.headers
{'x-powered-by': 'PHP/5.3.3', 'transfer-encoding': 'chunked', 'set-cookie': 'PHPSESSID=buqj70k7f9rrg51emsvatveda2; 。。。。}
>>> r.encoding
'UTF-8'
>>> r.status_code
200
>>> print（r.text）
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  ......

```
请求发出后，requests会基于http头部对相应的编码做出有根据的推测，**当你访问r.text之时，requests会使用其推测的文本编码**。你可以找出requests使用了什么编码，并且能够使用r.coding属性来改变它。
**但是如果使用r.content则获取的是原生的二进制流**
```

>>> r.content
'\xef\xbb\xbf\xef\xbb\xbf<!DOCTYPE html>\n<html lang="zh-CN">\n  <head>\n  。。。>\n  

```       
以二进制的方式打开服务器并返回数据。

####  自定义get头部 
```

>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}
>>> r = requests.get(url, headers=headers)

```
###  post请求 
**requests发送post请求，通常你会想要发送一些编码为表单的数据**——非常像一个html表单。要实现这个，只需要简单地传递一个字典给data参数。你的数据字典在发出请求时会自动编码为表单形式。
```

>>> import requests
>>> payload = {"key1":"value1","key2":"value2"}
>>>headers={'Uer-Agent':'','Spam':'eggs'}
>>> r1 = requests.post("http://httpbin.org/post", data=payload,headers=headers)

```
![](/data/dokuwiki/python/pasted/20160313-124638.png)
多了form项。
值得一提的是，requests库能够以多种方式从请求中返回响应结果的内容。
  * resp.text会自动进行解码然后得到的是解码后的文本。
  * resp.content是原始的二进制数据。
  * resp.json得到JSON格式的响应内容

###  http头部 
```

>>> r.headers['content-type']
'application/json'

```
注意，在引号里面的内容，不区分大小写'CONTENT-TYPE'也可以。
还能够自定义头部：
```

>>> r.headers['content-type'] = 'adad'
>>> r.headers['content-type']
'adad'

```
注意，当定制头部的时候，如果需要定制的项目有很多，需要用到数据类型为字典。

###  cookies 
下面示例将一个请求中得到的HTTP cookies传递给下一个请求:
```

import requests
resp1=requests.get(url)
...
resp2=requests.get(url,cookies=resp1.cookies)

```

###  文件上传 
```

import requests
url='http://assa.com/upload'
files={'file':('data.csv',open('data.csv',rb))}
r=requests.post(url,files=files)

```

##  流式获取返回值 
有个时候返回内容可能很大，如果一下子加载到内存会导致内存很快就消耗殆尽。所以我们需要进行流式获取
**流式获取设置 stream=True** 
```

>>> r = requests.get('https://api.github.com/events', stream=True) #流式获取
>>> r.raw
<requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
>>> r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'

```
一般需要通过以下方式进行迭代返回内容流。
**当内容是二进制时:r.iter_content(chunk_size=512)**
```

with open(filename, 'wb') as fd:
    for chunk in r.iter_content(512):
        fd.write(chunk)

```
**当内容是文本时：iter_lines(chunk_size=512)**
```

with open(filename, 'w') as fd:
    for chunk in r.iter_lines(512):
        fd.write(chunk)

```
        
###  请求超时 
timeout=30 单位秒
```

>>> requests.get('http://github.com', timeout=30)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)

```
###  允许重定向 
```

>>> r = requests.head('http://github.com', allow_redirects=True)

>>> r.url
'https://github.com/'

>>> r.history
[<Response [301]>]

```
###  Cookies 
If a response contains some Cookies, you can quickly access them:
```

>>> url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)

>>> r.cookies['example_cookie_name']
'example_cookie_value'

```
To send your own cookies to the server, you can use the cookies parameter:
```

>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'

```

###  各种数据形式的POST请求 
```

#形式一
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.post("http://httpbin.org/post", data=payload)

#形式二
>>> import json
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
>>> r = requests.post(url, data=json.dumps(payload))

#形式三 json格式
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, json=payload)

```

###  Session对象 
Session对象允许你对同一主机的多个请求**进行参数保留、cookies保留、连接重用**。
Session对象拥有requests API的所有方法。
```

s = requests.Session()
s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
r = s.get('http://httpbin.org/cookies')
print(r.text)
# '{"cookies": {"sessioncookie": "123456789"}}'

```
```

s = requests.Session()
s.auth = ('user', 'pass')
s.headers.update({'x-test': 'true'})

# both 'x-test' and 'x-test2' are sent
s.get('http://httpbin.org/headers', headers={'x-test2': 'true'})

```
**Session还可以用于with/as上下文管理器**
```

with requests.Session() as s:
    s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')

```
###  SSL Cert Verification¶ 
略。
###  Proxies 
略。。
参考:http://git.oschina.net/xbynet/StarterLearningPython/blob/master/228.md