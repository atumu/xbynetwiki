title: nginx_https 

#  nginx+https配置 
https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
TLS或传输层安全( transport layer security)，它的前身是SSL(安全套接字层secure sockets layer)，是Web协议用来包裹在一个受保护，加密封装正常通道。
采用这种技术，服务器和客户端之间可以安全地进行交互，而不用担心消息将被拦截和读取。证书系统帮助用户在核实它们与连接站点的身份。
**步骤1：Create the SSL Certificate**
```

sudo mkdir /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt

```
创建了有效期100年，加密强度为RSA2048的SSL密钥key和X509证书文件。
参数说明:
  * req: 配置参数-x509指定使用 X.509证书签名请求管理(certificate signing request (CSR)). "X.509" 是一个公钥代表that SSL and TLS adheres to for its key and certificate management.
  * -nodes: 告诉OpenSSL生产证书时忽略密码环节.(因为我们需要Nginx自动读取这个文件，而不是以用户交互的形式)。
  * -days 36500: 证书有效期，100年
  * -newkey rsa:2048: 同时产生一个新证书和一个新的SSL key(加密强度为RSA 2048)
  * -keyout:SSL输出文件名
  * -out:证书生成文件名
它会问一些问题。需要注意的是在common name中填入网站域名，如wiki.xby1993.net即可生成该站点的证书，同时也可以使用泛域名如*.xby1993.net来生成所有二级域名可用的网站证书。
整个问题应该如下所示:
```

Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York City
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Bouncy Castles, Inc.
Organizational Unit Name (eg, section) []:Ministry of Water Slides
Common Name (e.g. server FQDN or YOUR name) []:your_domain.com
Email Address []:admin@your_domain.com

```

**步骤2： Configure Nginx to Use SSL**
http://nginx.org/en/docs/http/configuring_https_servers.html
http://manual.seafile.com/deploy/https_with_nginx.html
https://s.how/nginx-ssl/
首先配置HTTP请求重定向
```

server {
        listen       80;
        server_name  www.yourdomain.com;
        rewrite ^ https://$http_host$request_uri? permanent;    # force redirect http to https
	#return 301 https://$http_host$request_uri;
    }

```
```

server {
        listen 443 ssl;

        ssl_certificate /etc/nginx/ssl/nginx.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx.key;
	keepalive_timeout   70;
        server_name www.yourdomain.com;
	#禁止在header中出现服务器版本，防止黑客利用版本漏洞攻击
	server_tokens off;
	#如果是全站 HTTPS 并且不考虑 HTTP 的话，可以加入 HSTS 告诉你的浏览器本网站全站加密，并且强制用 HTTPS 访问
	#add_header Strict-Transport-Security "max-age=31536000; includeSubdomains";
        # ......
        fastcgi_param   HTTPS               on;
        fastcgi_param   HTTP_SCHEME         https;

	access_log      /var/log/nginx/wiki.xby1993.net.access.log;
        error_log       /var/log/nginx/wiki.xby1993.net.error.log;
    }

```
如果想同时启用HTTP和HTTPS
```

server {
    listen              80;
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ...
}

```
步骤3：重启nginx
```

sudo service nginx restart

```
##  附录1、证书格式说明 
  * .crt：自签名的证书
  * .csr：证书的请求(用于向证书颁发机构申请crt证书时使用，nginx配置时不会用到)
  * .key：SSL Key (分为不带口令和带口令版本)。
我们自签名证书配置nginx需要的是.crt证书，和不带口令的SSL Key的.key文件。

##  附录2、可靠的第三方SSL证书颁发机构 
目前一般市面上针对中小站长和企业的 SSL 证书颁发机构有：
StartSSL
Comodo / 子品牌 Positive SSL
GlobalSign / 子品牌 AlphaSSL
GeoTrust / 子品牌 RapidSSL

更多参考资料:
http://www.liaoxuefeng.com/article/0014189023237367e8d42829de24b6eaf893ca47df4fb5e000
https://github.com/michaelliao/itranswarp.js/blob/master/conf/ssl/gencert.sh
http://www.21andy.com/new/20100224/1714.html