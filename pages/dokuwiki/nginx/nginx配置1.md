title: nginx配置1 

#  nginx配置之location与try_files、rewrite 

##  location指令 
语法：location [=|~|~*|^~|@] /uri/ { … }
默认值：无
作用域：server
  * 没有前缀表示匹配开头，但是不能使用正则表达式
  * = 开头表示**精确匹配**
  * ^~ 匹配情况类似 (None)没有前缀 的情况，以指定匹配模式开头的 URI 被匹配.` 不是正则表达式 `。不同的是，一旦匹配成功，那么 Nginx 就停止去寻找其他的 Location 块进行匹配了（与 Location 匹配顺序有关）
  * ~ 开头表示**区分大小写的正则匹配**
  * ~*  开头表示**不区分大小写的正则匹配**
  * !~和!~*分别为区分大小写不匹配及不区分大小写不匹配 的正则
  * / 通用匹配，任何请求都会匹配到。
  * @ 用于定义一个**命名 Location 块**，且该块不能被外部 Client 所访问，**只能被 Nginx 内部配置指令所访问，比如 try_files or error_page**

**多个location配置的情况下匹配顺序为**:
要注意的是，写在配置文件中每个 Server 块中的 Location 块的次序是不重要的，Nginx 会按 location modifier 的优先级来依次用 URI 去匹配 pattern ，顺序如下：
  * 1. =
  * 2. (None)    如果 pattern 完全匹配 URI（不是只匹配 URI 的头部）
  * 3. ^~
  * 4. ~ 或 ~*
  * 5. (None)    pattern 匹配 URI 的头部
<WRAP center round important 60%>
location 匹配的优先级(来自实践总结中)
` (location = ) > (location 完整路径 >) >(location ^~ 路径) >(location ~* 正则) >(location 路径) `
只要匹配到，其它的都会忽略，然后返回到改匹配。
</WRAP>
参考：http://www.jb51.net/article/47761.htm

关于location正则规则，请参考http://blog.xinshangshangxin.com/2016/01/20/nginx-location/
例子，有如下匹配规则：
```

location = / {
   #规则A
}
location = /login {
   #规则B
}
location ^~ /static/ {
   #规则C
}
location ~ \.(gif|jpg|png|js|css)$ {
   #规则D
}
location ~* \.png$ {
   #规则E
}
location !~ \.xhtml$ {
   #规则F
}
location !~* \.xhtml$ {
   #规则G
}
location / {
   #规则H
}

```
那么产生的效果如下：
访问根目录/， 比如http://localhost/ 将匹配规则A
访问 http://localhost/login 将匹配规则B，http://localhost/register 则匹配规则H
访问 http://localhost/static/a.html 将匹配规则C
访问 http://localhost/a.gif, http://localhost/b.jpg 将匹配规则D和规则E，但是规则D顺序优先，规则E不起作用，而 http://localhost/static/c.png 则优先匹配到规则C
访问 http://localhost/a.PNG 则匹配规则E，而不会匹配规则D，因为规则E不区分大小写。
访问 http://localhost/a.xhtml 不会匹配规则F和规则G，http://localhost/a.XHTML不会匹配规则G，因为不区分大小写。规则F，规则G属于排除法，符合匹配规则但是不会匹配到，所以想想看实际应用中哪里会用到。
访问 http://localhost/category/id/1111 则最终匹配到规则H，因为以上规则都不匹配，这个时候应该是nginx转发请求给后端应用服务器，比如FastCGI（php），tomcat（jsp），nginx作为方向代理服务器存在。
所以实际使用中，个人觉得至少有三个匹配规则定义，如下：
#直接匹配**网站根**，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
#这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
```

location = / {
    proxy_pass http://tomcat:8080/index
}

```
# 第二个必选规则是**处理静态文件请求**，**这是nginx作为http服务器的强项**
# **有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用**
```

location ^~ /static/ {
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}

```
#第三个规则就是**通用规则，用来转发动态请求到后端应用服务器**
#非静态文件请求就默认是动态请求，自己根据实际把握
#**毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了**
```

location / {
    proxy_pass http://tomcat:8080/
}

```

###  rewrite语法 
` 官方文档:http://nginx.org/en/docs/http/ngx_http_rewrite_module.htm ` 
http://www.linuxidc.com/Linux/2015-06/119398.htm
**rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。**
rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用，例如http://seanlook.com/a/we/index.php?id=1&u=str 只对/a/we/index.php重写。
**语法rewrite regex replacement [flag];** 例如:rewrite ^/ http:/ /$host/11.png redirect;

如果相对域名或参数字符串起作用，可以使用全局变量匹配，也可以使用proxy_pass反向代理。
表明看rewrite和location功能有点像，都能实现跳转，主要区别在于rewrite是在同一域名内更改获取资源的路径，而location是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。
  * 很多情况下rewrite也会写在location里，它们的执行顺序是：
  * 执行server块的rewrite指令
  * 执行location匹配
  * 执行选定的location中的rewrite指令
如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件；循环超过10次，则返回500 Internal Server Error错误。
Flag标志
  * last – 表示完成rewrite
  * break – **中止**Rewirte，不再继续匹配后续rewrite指令集
  * redirect – 返回**临时重定向**的HTTP状态302
  * permanent – 返回**永久重定向**的HTTP状态301
这里 last 和 break 区别有点难以理解：
  * last一般写在server和if中，而break一般使用在location中
  * last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后的匹配
  * break和last都能**阻止继续执行后面的rewrite指令**
####  if指令与全局变量 
if判断指令
语法为if(condition){...}
对给定的条件condition进行判断。如果为真，大括号内的rewrite指令将被执行，if条件(conditon)可以是如下任何内容：
  * 当表达式只是一个变量时，如果值为空或任何以0开头的字符串都会当做false
  * 直接比较变量和内容时，使用=或!=
  * ~正则表达式匹配，~*不区分大小写的匹配，!~区分大小写的不匹配
1、下面是可以用来判断的**表达式**：
  * -f和!-f用来判断是否存在文件
  * -d和!-d用来判断是否存在目录
  * -e和!-e用来判断是否存在文件或目录
  * -x和!-x用来判断文件是否可执行

Nginx中实现if多重判断写法
```

set $doRewrite "0";
if ($request_uri ~ ^/admin/) {
set $doRewrite "1";
}
if ($request_uri ~ ^/admin/images/) {
set $doRewrite "0";
}
if ($doRewrite = "1") {
// do rewrite
}

```
要注意的是，` Nginx配置文件里 if 和后面的括号之间要有一个空格，不然会报unknown directive错误 `。
```

if ($http_user_agent ~ MSIE){
rewrite ^(.*)$ /msie/$1 break;
}//如果UA包含"MSIE"，rewrite请求到/msid/目录下
if($http_cookie ~*"id=([^;]+)(?:;|$)"){
set $id $1;
}//如果cookie匹配正则，设置变量$id等于正则引用部分
if($request_method = POST){
return 405;
}//如果提交方法为POST，则返回状态405（Method not allowed）。return不能返回301,302
if($slow){
limit_rate 10k;
}//限速，$slow可以通过 set 指令设置
if(!-f $request_filename){
break;
proxy_pass http://127.0.0.1;
}//如果请求的文件名不存在，则反向代理到localhost 。这里的break也是停止rewrite检查
if($args ~ post=140){
rewrite ^ http://example.com/ permanent;
}//如果query string中包含"post=140"，永久重定向到example.com
location ~* \.(gif|jpg|png|swf|flv)$ {
valid_referers none blocked www.jefflei.com www.leizhenfang.com;
if($invalid_referer){
return404;
}//防盗链
}

```
```

if ($http_user_agent ~ MSIE) {  
    rewrite ^(.*)$ /msie/$1 break;  //Flag标志可以放在后面
}

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;   //设置自定义变量为正则表达式的某个值
}

if ($request_method = POST) {
    return 405; //通过return指令，直接返回状态码
}

if ($slow) {
    limit_rate 10k; //网络限速
}

if ($invalid_referer) {
    return 403;
}

```
####  全局变量 
2、下面是可以用作判断的**全局变量**
例：http://localhost:88/test1/test2/test.php
  * $host：localhost
  * $server_port：88
  * $request_uri：/test1/test2/test.php
  * $document_uri：/test1/test2/test.php
  * $document_root：D:\nginx/html
  * $request_filename：D:\nginx/html/test1/test2/test.php
下面是可以用作if判断的全局变量
  * $args ： #这个变量等于请求行中的参数，同$query_string
  * $content_length ： 请求头中的Content-length字段。
  * $content_type ： 请求头中的Content-Type字段。
  * $document_root ： 当前请求在root指令中指定的值。
  * $host ： 请求主机头字段，否则为服务器名称。
  * $http_user_agent ： 客户端agent信息
  * $http_cookie ： 客户端cookie信息
  * $limit_rate ： 这个变量可以限制连接速率。
  * $request_method ： 客户端请求的动作，通常为GET或POST。
  * $remote_addr ： 客户端的IP地址。
  * $remote_port ： 客户端的端口。
  * $remote_user ： 已经经过Auth Basic Module验证的用户名。
  * $request_filename ： 当前请求的文件路径，由root或alias指令与URI请求生成。
  * $scheme ： HTTP方法（如http，https）。
  * $server_protocol ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
  * $server_addr ： 服务器地址，在完成一次系统调用后可以确定这个值。
  * $server_name ： 服务器名称。
  * $server_port ： 请求到达服务器的端口号。
  * $request_uri ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
  * $uri ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
  * $document_uri ： 与$uri相同。
####  redirect语法 
```

server {
listen 80;
server_name start.igrow.cn;
index index.html index.php;
root html;
if ($http_host !~ “^star\.igrow\.cn$&quot {
rewrite ^(.*) http://star.igrow.cn$1 redirect;
}
}

```
####  根据文件类型设置过期时间 
```

location ~* \.(js|css|jpg|jpeg|gif|png|swf)$ {
if (-f $request_filename) {
	expires 1h;
	break;
}
}

```
####  禁止访问某个目录 
```

location ~* \.(txt|doc)${
root /data/www/wwwroot/linuxtone/test;
	deny all;
}

```

####  常用正则 
. ： 匹配除换行符以外的任意字符
? ： 重复0次或1次
+ ： 重复1次或更多次
* ： 重复0次或更多次
\d ：匹配数字
^ ： 匹配字符串的开始
$ ： 匹配字符串的介绍
{n} ： 重复n次
{n,} ： 重复n次或更多次
[c] ： 匹配单个字符c
[a-z] ： 匹配a-z小写字母的任意一个
**小括号()之间匹配的内容，可以在后面通过$1来引用，$2表示的是前面第二个()里的内容。**正则里面容易让人困惑的是\转义特殊字符。

####  rewrite实例 
```

http {
# 定义image日志格式
log_format imagelog '[$time_local] ' $image_file ' ' $image_type ' ' $body_bytes_sent ' ' $status;
# 开启重写日志
rewrite_log on;
server {
root /home/www;
location /{
# 重写规则信息
error_log logs/rewrite.log notice;
# 注意这里要用‘’单引号引起来，避免{}
rewrite '^/images/([a-z]{2})/([a-z0-9]{5})/(.*)\.(png|jpg|gif)$' /data?file=$3.$4;
# 注意不能在上面这条规则后面加上“last”参数，否则下面的set指令不会执行
set $image_file $3;
set $image_type $4;
}
location /data {
# 指定针对图片的日志格式，来分析图片类型和大小
access_log logs/images.log mian;
root /data/images;
# 应用前面定义的变量。判断首先文件在不在，不在再判断目录在不在，如果还不在就跳转到最后一个url里
try_files /$arg_file /image404.html;
}
location =/image404.html {
# 图片不��在返回特定的信息
return 404 "image not found\n";
}
}

```
对形如/images/ef/uh7b3/test.png的请求，重写到/data?file=test.png，于是匹配到location /data，先看/data/images/test.png文件存不存在，如果存在则正常响应，如果不存在则重写tryfiles到新的image404 location，直接返回404状态码。
例2：
```

rewrite ^/images/(.*)_(\d+)x(\d+)\.(png|jpg|gif)$ /resizer/$1.$4?width=$2&height=$3?last;

```
对形如/images/bla_500x400.jpg的文件请求，重写到/resizer/bla.jpg?width=500&height=400地址，并会继续尝试匹配location。


##  try_files指令 
http://luokr.com/p/14
Nginx的配置语法灵活，可控制度非常高。在0.7以后的版本中加入了一个try_files指令，配合命名location，可以部分替代原本常用的rewrite配置方式，提高解析效率。

**try_files**
语法：try_files file ... uri 或 try_files file ... = code
默认值：无
作用域：server location

其作用是**按顺序检查文件是否存在**，返回第一个找到的文件或文件夹（结尾加斜线表示为文件夹），**如果所有的文件或文件夹都找不到，会进行一个内部重定向到最后一个参数。**
需要注意的是，**只有最后一个参数可以引起一个内部重定向，之前的参数只设置内部URI的指向。**最后一个参数是回退URI且必须存在，否则会出现内部500错误。` 命名的location也可以使用在最后一个参数中。与rewrite指令不同，如果回退URI不是命名的location那么$args不会自动保留，如果你想保留$args，则必须明确声明。 `
当然nginx新的改良是可以出现如下的使用形式的
```

try_files $uri = 404

```
什么意思呢？uri不能成功访问，那好，那就给你个404吧。
比较完整的例子是这样的:
```

try_files $uri $uri/ /index.php$is_args$args;

```
如上所解释的，**挨个访问$uri $uri/(也就是前一个路径在加上个/)如果这俩都不能访问，那就访问最后一个路径**。
  * $args就是get请求中的url参数e.g. foo=123&bar=blahblah; 
  * $uri是不带参数的，也不包含协议主机名端口号等。例如http://wiki.xby1993.net/foo/bar.html?id=1得到的$uri是/foo/bar.html，它有可能和$request_uri不同($request_uri是浏览器发送的原始url，没有经过修改的)。

```

upstream tornado {  //负载均衡
        server 127.0.0.1:8001;
}
 
server {
        server_name luokr.com;
        return 301 $scheme://www.luokr.com$request_uri; //注意重定向写法
}
 
server {
        listen 80;
        server_name www.luokr.com;
 
        root /var/www/www.luokr.com/V0.3/www;
        index index.html index.htm;
 
        try_files $uri @tornado;   //最后一个文件引用一个location
 
        location @tornado {
                proxy_pass_header Server;
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Scheme $scheme;
 
                proxy_pass http://tornado;   //反向代理到负载均衡服务
        }
}

```
**location指令是用来为匹配的URI进行配置，URI即语法中的"/uri/"，可以是字符串或正则表达式。**但如果要使用正则表达式，则必须指定前缀。
** [@] 即是命名location，一般只用于内部重定向请求。**


##  nginx-某cms发布系统多站点配置规则示例 
```


#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    access_log  logs/access.log;
    error_log logs/error.log;
    sendfile        on;
    keepalive_timeout  65;
    client_max_body_size 2048m;

    upstream myServer {   
        server 127.0.0.1:8081; #负载均衡后台tomcat
    }

    server {
        listen       8080;
        server_name  localhost;

        set $appdir "E:/workspace/eclipse/.metadata/.plugins/org.eclipse.wst.server.core/tmp1/wtpwebapps/cms-manager";
        set $publishdir "E:/program/tomcat7/webapps";
        location = / {
		 proxy_set_header Host $host:$server_port;
		 proxy_set_header X-Real-IP $remote_addr;
		 proxy_set_header request_uri $request_uri;
		 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://myServer; #首页直接跳转tomcat
        }
        location ^~ /sword {
		 proxy_set_header Host $host:$server_port;
		 proxy_set_header X-Real-IP $remote_addr;
		 proxy_set_header request_uri $request_uri;
		 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://myServer; #以sword开头直接跳转tomcat
        }
        location ~ \.(html|json)$ {  #处理html与json文件访问
            if ($uri !~ ^/static) {    #如果不是以static开头，
                root $publishdir;#root定位到发布到的静态站点。
                rewrite ^(.*)$ /$args$uri; #对于类似url ,/qq22/index.html?qq11重写为/qq11/qq22/index.html，以便于预览访问。
		error_page 405 =200 $uri; #解决post请求静态文件如json文件时导致的405 not allowed问题。注意=号的位置。由于前面执行过rewrite，所以这时候的$uri就相当于rewrite之前的/$args$uri.
                break; #禁止后续规则匹配
                
            }
		#如果以static开头，定位root到tomcat apps引用下
            root $appdir;
        }
        location ~ ^/.*?/static/{ #如果uri以/qqq111/static/qq2的形式，那么定位root到发布的静态站点目录
            root $publishdir;
        }
	  location ~ \.jsp$ {
	   	proxy_set_header Host $host:$server_port;
	   	proxy_set_header X-Real-IP $remote_addr;
	   	proxy_set_header request_uri $request_uri;
	   	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	   	proxy_pass http://myServer;
	  }
        location  /static{  #为了确保static下的jsp能够正确解析，将它的优先级将为最低
            root $appdir;
        }

        location ^~ /resource {
            root $publishdir;
        }

    }   

}


```
##  文件上传大小限制 
Syntax:	client_max_body_size size;
Default:	
client_max_body_size 1m;
Context:	http, server, location