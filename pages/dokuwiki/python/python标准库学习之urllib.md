title: python标准库学习之urllib 

#  Python标准库学习之urllib 
本系列以python3.4为基础
[urllib](https://docs.python.org/3/library/urllib.html)是Python3的标准网络请求库。包含了网络数据请求，处理cookie,改变请求头和用户代理，重定向，认证等的函数。
**urllib与urllib2?**:python2.x用urllib2,而python3改名为urllib,被分成一些子模块：urllib.request,urllib.parse,urllib.error,urllib.robotparser.尽管函数名称大多和原来一样，但是使用新的urllib库时需要注意哪些函数被移动到子模块里了。

HTTP版本：HTTP/1.1，包含Connection:close 头

特别常用的函数:urllib.request.urlopen()

同类型开源库推荐:[requests](http://requests.readthedocs.org/)

另一部分文章参见[这里](https://wiki.xby1993.net/doku.php?id=python:python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B015)

urllib:用来处理网络请求和操作url。有以下子模块
  * urllib.request 打开后读取url内容
  * urllib.error 包含由urllib.request抛出的异常类
  * urllib.parse 解析URL
  * urllib.robotparser 解析robots.txt files

简单的例子
```

from urllib.request import urlopen
html=urlopen('https://www.baidu.com')
print(html.geturl(),html.info(),html.getcode(),sep='\n')
print(html.read().decode('UTF-8'))


from urllib import request
with request.urlopen('https://api.douban.com/v2/book/2129650') as f:
    data = f.read()
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', data.decode('utf-8'))
    
from urllib import request
req = request.Request('http://www.douban.com/') #设置请求头
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25')
with request.urlopen(req) as f:
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', f.read().decode('utf-8'))

```
```

import urllib.request
data = parse.urlencode([ #进行url编码参数
    ('username', 'xby')]
req = urllib.request.Request(url='https://www.baidu.com',
                     data=data)
with urllib.request.urlopen(req) as f:
    print(f.read().decode('utf-8'))


```
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
##  urllib.request 

**urllib.request.urlopen**(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None) 
  * url参数可以是**字符串或者**urllib.request.**Request对象**
  * data参数必须是字节形式。可以通过from urllib import  parse  parse.urlencode()来处理得到。如果没有提供dat参数则为GET请求，否则为POST请求。
  * [tomeout,]超时单位为秒
  * context参数必须是ssl.SSLContext的实例

返回值：返回一个可以作为contextmanager的对象。它有一些方法和属性：
  * geturl()
  * info()-元数据信息，比如headers
  * getcode()-http响应码，比如200
  * read()-获取内容，字节形式
  * status
  * reason
对于Http(s)请求，返回的一个http.client.HTTPResponse对象。常用方法getheaders(),read()
对于ftp,file请求，返回一个urllib.response.addinfourl对象

**可能抛出的异常urllib.error.URLError,urllib.error.HTTPError** 

**class urllib.request.Request**(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None) 
通过这个对象我们可以设置请求数据，添加请求头，同时可以获取一些url信息：比如协议类型，主机。也可以设置代理Request.set_proxy(host, type) 

class urllib.request.OpenerDirector以及关联的urllib.request.install_opener(opener),urllib.request.build_opener([handler, ...]) 
方法：OpenerDirector.add_handler(handler) ，这个handler对象必须继承urllib.request.BaseHandler，常见的有
urllib.request.BaseHandler -基类
urllib.request.HTTPDefaultErrorHandler 
urllib.request.HTTPRedirectHandler 
urllib.request.HTTPCookieProcessor
urllib.request.ProxyHandler
urllib.request.HTTPBasicAuthHandler
urllib.request.HTTPSHandler
例子：
```

import urllib.request
# Create an OpenerDirector with support for Basic HTTP Authentication...
auth_handler = urllib.request.HTTPBasicAuthHandler()
auth_handler.add_password(realm='PDQ Application',
                          uri='https://mahler:8092/site-updates.py',
                          user='klem',
                          passwd='kadidd!ehopper')
opener = urllib.request.build_opener(auth_handler)
# ...and install it globally so it can be used with urlopen.
urllib.request.install_opener(opener)
urllib.request.urlopen('http://www.example.com/login.html')

proxy_handler = urllib.request.ProxyHandler({'http': 'http://www.example.com:3128/'})
proxy_auth_handler = urllib.request.ProxyBasicAuthHandler()
proxy_auth_handler.add_password('realm', 'host', 'username', 'password')

opener = urllib.request.build_opener(proxy_handler, proxy_auth_handler)
# This time, rather than install the OpenerDirector, we use it directly:
opener.open('http://www.example.com/login.html')


```
##  异常处理 
**可能抛出的异常urllib.error.URLError,urllib.error.HTTPError** 
exception urllib.error.URLError :有以下属性：reason
exception urllib.error.HTTPError 它是URLError的一个子类，有以下属性：
  * code 
  * reason 
  * headers 
```

from urllib.request import Request, urlopen
from urllib.error import URLError, HTTPError
req = Request("http://www.baidu.com/")
try:
	response = urlopen(req)
except HTTPError as e:
	print('Error code: ', e.code)
except URLError as e:
	print('We failed to reach a server.')
	print('Reason: ', e.reason)
else:
	print("good!")
print(response.read().decode("utf8"))

```

##  urllib.parse 
urllib.parse.urlparse函数会将一个普通的url解析为6个部分，返回的数据类型为**ParseResult**对象，通过访问其属性可以获取对应的值。
同时，它还可以将已经分解后的url再组合成一个url地址（通过urlunparse(parts)）。返回的6个部分，分别是：scheme(机制)、netloc(网络位置)、path(路径)、params(路径段参数)、query(查询)、fragment(片段)。
**urllib.parse.urlencode**(query, doseq=False, safe=' ', encoding=None, errors=None),注意：**query参数是一个序列对象**

##  通过urllib.request.urlretrieve下载文件 
urllib.request.urlretrieve(url,savefilepath)
