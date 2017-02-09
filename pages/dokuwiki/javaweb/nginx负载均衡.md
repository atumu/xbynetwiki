title: nginx负载均衡 

#  nginx负载均衡与配置文件解析 

本篇先从负载均衡服务架构入手，关于负载均衡百度百科的定义如下：负载均衡，英文名称为Load Balance，其意思就是分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。
我的解释：一项任务交由一个开发人员处理总会有上限处理能力，这时可以考虑增加开发人员来共同处理这项任务，多人处理同一项任务时就会涉及到调度问题，即任务分配，这和多线程理念是一致的。nginx在这里的角色相当于任务分配者。
![](/data/dokuwiki/javaweb/pasted/20150925-043058.png)
##  Nginx安装 

` Nginx `是一款轻量级的**Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器**，并在一个BSD-like 协议下发行。由俄罗斯的程序设计师Igor Sysoev所开发，供俄国大型的入口网站及搜索引擎Rambler（俄文：Рамблер）使用。**其特点是占有内存少，并发能力强**，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、新浪、网易、腾讯等。

最新版本的nginx版本为1.9.3，我这下载的是window版本的，一般实际场景都是安装在linux系统下的，由于linux系统目前正在摸索中这里就不介绍。官方下载完成之后解压运行nginx.exe就启动了nginx了，启动后会在进程里面看到nginx。
要实现负载均衡需要修改conf/nginx.conf的配置信息，修改配置信息之后重新启动nginx服务，可以通过nginx -s reload指令实现。这里我们使用 Ants 提供的一个批处理来操作。

##  站点搭建及配置 

1.搭建两个iis站点
站点下只有一个简单的index页面，用来输出当前服务器信息。由于我没有两台机器，所以将两个站点都部署到本机了，分别绑定了**8082和9000**两个端口。

###  2.修改nginx配置信息 
Nginx配置文件主要分成四部分：main（全局设置）、server（主机设置）、upstream（上游服务器设置，主要为反向代理、负载均衡相关配置）和 location（URL匹配特定位置后的设置），每部分包含若干个指令。
  * main部分设置的指令将影响其它所有部分的设置；
  * server部分的指令主要用于指定虚拟主机域名、IP和端口；
  * upstream的指令用于设置一系列的后端服务器，设置反向代理及后端服务器的负载均衡；
  * location部分用于匹配网页位置（比如，根目录“/”,“/images”,等等）。
他们之间的关系式：server继承main，location继承server；upstream既不会继承指令也不会被继承。它有自己的特殊指令，不需要在其他地方的应用。

修改nginx监听端口，修改http server下的listen节点值，由于本机80端口已经被占用，我改为监听8083端口。
listen 8083;
在` http节点 `下添加` upstream（服务器集群） `，**server设置的是集群服务器的信息**，我这里搭建了两个站点，配置了两条信息。

```

#http节点下
#服务器集群名称为Jq_one
upstream Jq_one {
　　server 127.0.0.1:9000; 
　　server 127.0.0.1:8082; 
}

```
在` http节点下找到location节点 `修改

```

#http->server节点下：
location / {
           root   html;
           index  index.aspx index.html index.htm; #修改主页为index.aspx
    #其中jq_one 对应着upstream设置的集群名称
    proxy_pass         http://Jq_one; 
    #设置主机头和客户端真实地址，以便服务器获取客户端真实IP
    proxy_set_header   Host             $host; 
    proxy_set_header   X-Real-IP        $remote_addr; 
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
}

```
修改完成配置文件以后记得**重启nginx服务**，最终完整配置文件信息如下
![](/data/dokuwiki/javaweb/pasted/20150925-031023.png)

**3.运行结果**
访问http://127.0.0.1:8083/index.aspx ，多访问几次，着重关注标红部分。
![](/data/dokuwiki/javaweb/pasted/20150925-031120.png)![](/data/dokuwiki/javaweb/pasted/20150925-031124.png)
可以看到，我们的请求被分发到了8082站点和9000站点，并且第一次是8082站点第二次9000。出现这样的结果证明我们负载均衡搭建成功了。 尝试关闭其中的9000站点，然后刷新页面发现输出的http端口一直是8082，也就是说其中一个站点挂了，只要还有一个站点是好的，我们的还是可以服务.

##  集群问题分析（如Session共享，代码部署，文件共享，加权分配，静态缓存等） 
虽然我们搭建好了负载均衡站点，但是还存在以下问题。
` Session共享，代码部署，文件共享，加权分配，静态缓存，获取用户真实IP等 `

1.如果站点使用了` session `，请求平均分配到两个站点，那么必然**存在session共享问题**，该如何解决？
  * 使用数据库保存session信息
  * 使用nginx**将同一ip的请求分配到固定服务器**，修改如下。ip_hash会计算ip对应hash值，然后分配到固定服务器
```

upstream Jq_one{
 　　server 127.0.0.1:8082 ;
　　 server 127.0.0.1:9000 ;
  　　ip_hash;
　}

```
  * 搭建一台` Redis服务器 `，对session的读取都从该Redis服务器上读取。后面的文章将介绍**分布式缓存Redis**的使用

2.管理员**更新站点代码**，该怎么操作，现在还只有两台服务器，可以手工将文件更新到两台服务器，如果是10台呢，那么手工操作必然是不可行的
多服务器站点更新可以使用GoodSync 文件同步程序，会自动检测文件的修改新增，然后同步到其它服务器上。**在linux下可以使用rsync**

3.站点中的文件上传功能会将文件分配到不同的服务器，**文件共享问题**如何解决。
使用**文件服务器将所有文件存储到该服务器上**，文件操作读取写入都在该服务器上。这里同样会存在一个问题，文件服务器存在读写上限。

4.负载的服务器配置不一样，有的高有的低可不可以让配置高的服务器处理请求多一些
Nginx 的`  upstream ` 目前支持4 种方式的分配——
(1)轮询（默认） :
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
(2)weight ：(加权）
指定轮询几率，weight 和访问比率成正比，用于后端服务器性能不均的情况。
(3)ip_hash ：(同一用户请求分配到同一服务器)
每个请求按访问ip 的hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决session 的问题。可以针对同一个C 类地址段中的客户端选择同一个后端服务器，除非那个后端服务器宕了才会换一个。
(4)fair（第三方）:
按后端服务器的响应时间来分配请求，响应时间短的优先分配。
(5)url_hash（第三方）:
按访问url 的hash 结果来分配请求，使每个url 定向到同一个后端服务器，后端服务器为缓存时比较有效。

这里讲一下，**负载均衡有好几种算法 轮转法，散列法， 最少连接法，最低缺失法，最快响应法，加权法**。我们**这里可以使用` 加权法 `来分配请求**。
```

upstream Jq_one{
   #weigth参数表示权值，权值越高被分配到的几率越大
　　server 127.0.0.1:8082 weight=4;
　　server 127.0.0.1:9000 weight=1;
}

```


5.由于请求是经过nginx转发过来的，可以在代码里面**获取到用户请求的实际ip地址**吗？
答案是肯定的，在localtion节点设置如下请求头信息
#设置主机头和客户端真实地址，以便服务器获取客户端真实IP
```

proxy_set_header Host $host; 
proxy_set_header X-Real-IP $remote_addr; 
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

```
代码里面通过` Request.Headers["X-Real-IP"] `，就能**获取到真实ip**

6.nginx实现**静态文件(image,js,css)缓存**
在server节点下添加新的localtion
```

#静态资源缓存设置
location ~ \.(jpg|png|jpeg|bmp|gif|swf|css)$
{ 
expires 30d;
root /nginx-1.9.3/html;#root: #静态文件存在地址，这里设置在/nginx-1.9.3/html下
break;
}

```

**总结**
通过nginx我们实现了一个简单的负载均衡，实际情况比这复杂很多。比如nginx服务器挂了，那我们的站点就直接挂了，正确的通过keepalived组件来搭建多台nginx服务提供服务。本篇只做为分布式系统的开篇，后续会陆续推出Redis缓存，数据库实现分布式架构的文章，敬请期待！

##  实战配置实例 
```

#用户，没用
user nobody;
#启动进程,通常设置成和cpu的数量相等
worker_processes  2;

#全局错误日志及PID文件
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

#工作模式及连接数上限
events {
    #epoll是多路复用IO(I/O Multiplexing)中的一种方式,
    #仅用于linux2.6以上内核,可以大大提高nginx的性能
    #use   epoll; 

    #单个后台worker process进程的最大并发链接数    
    worker_connections  1024;

    # 并发总数是 worker_processes 和 worker_connections 的乘积
    # 即 max_clients = worker_processes * worker_connections
    # 在设置了反向代理的情况下，max_clients = worker_processes * worker_connections / 4  为什么
    # 为什么上面反向代理要除以4，应该说是一个经验值
    # 根据以上条件，正常情况下的Nginx Server可以应付的最大连接数为：4 * 8000 = 32000
    # worker_connections 值的设置跟物理内存大小有关
    # 因为并发受IO约束，max_clients的值须小于系统可以打开的最大文件数
    # 而系统可以打开的最大文件数和内存大小成正比，一般1GB内存的机器上可以打开的文件数大约是10万左右
    # 我们来看看360M内存的VPS可以打开的文件句柄数是多少：
    # $ cat /proc/sys/fs/file-max
    # 输出 34336
    # 32000 < 34336，即并发连接总数小于系统可以打开的文件句柄总数，这样就在操作系统可以承受的范围之内
    # 所以，worker_connections 的值需根据 worker_processes 进程数目和系统可以打开的最大文件总数进行适当地进行设置
    # 使得并发总数小于操作系统可以打开的最大文件数目
    # 其实质也就是根据主机的物理CPU和内存进行配置
    # 当然，理论上的并发总数可能会和实际有所偏差，因为主机还有其他的工作进程需要消耗系统资源。
    # ulimit -SHn 65535

}


http {
    #设定mime类型,类型由mime.type文件定义
    include    mime.types;
    default_type  application/octet-stream;
    #设定日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，
    #对于普通应用，必须设为 on,
    #如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，
    #以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile     on;
    #tcp_nopush     on;

    #连接超时时间
    #keepalive_timeout  0;
    keepalive_timeout  65;
    tcp_nodelay     on;

    #开启gzip压缩
    gzip  on;
    gzip_disable "MSIE [1-6].";

    #设定请求缓冲
    client_header_buffer_size    128k;
    large_client_header_buffers  4 128k;
    


    #设定负载均衡的服务器列表
    upstream mysvr {
        #weigth参数表示权值，权值越高被分配到的几率越大
        #server 10.12.0.175:8081 weight=5;
	server 10.12.4.57:8080 weight=5;
    }

    #设定虚拟主机配置
    server {
        #侦听80端口
        listen    80;
        #定义使用 www.cssflow.cn访问
        server_name  www.cssflow.com;

        #定义服务器的默认网站根目录位置
        root D:\\cms-website\\cms-root;

        #设定本虚拟主机的访问日志
        access_log  logs/nginx.access.log  main;


        # 定义错误提示页面
        error_page   500 502 503 504 /50x.html;
        location = /50x.html {
        }

	#配置Nginx动静分离，定义的静态页面直接从Nginx发布目录读取。
        location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css|json|ttf|svg|woff|eot|xlsx|xls|doc|docx).*$ { 
	#location ^~ .*/(client|static)/ {
	     #root /usr/local/nginx/html/myloan; 
             #expires定义用户浏览器缓存的时间为7天，如果静态页面不常更新，可以设置更长，这样可以节省带宽和缓解服务器的压力
             expires      7d; 
        } 

        #后端分发
        #location ^~ /CSSFlowServer/ {
	#location ~ .*\.(jsp|php)$ { 
	location / {
            #定义服务器的默认网站根目录位置
            #root     /root; 
            #定义首页索引文件的名称
            #index index.php index.html index.htm;
            
            #请求转向mysvr 定义的服务器列表
            proxy_pass    http://mysvr ;

            #以下是一些反向代理的配置可删除.

            proxy_redirect off;

            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            #允许客户端请求的最大单文件字节数
            client_max_body_size 10m; 

            #缓冲区代理缓冲用户端请求的最大字节数，
            client_body_buffer_size 128k;

            #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_connect_timeout 90;

            #连接成功后，后端服务器响应时间(代理接收超时)
            proxy_read_timeout 90;

            #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffer_size 4k;

            #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
            proxy_buffers 4 32k;

            #高负荷下缓冲大小（proxy_buffers*2）
            proxy_busy_buffers_size 64k; 

            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
            proxy_temp_file_write_size 64k;    

        }

    }
}

```
**mime.types文件**
```


types {
    text/html                             html htm shtml;
    text/css                              css;
    text/xml                              xml;
    image/gif                             gif;
    image/jpeg                            jpeg jpg;
    application/javascript                js;
    application/atom+xml                  atom;
    application/rss+xml                   rss;

    text/mathml                           mml;
    text/plain                            txt;
    text/vnd.sun.j2me.app-descriptor      jad;
    text/vnd.wap.wml                      wml;
    text/x-component                      htc;

    image/png                             png;
    image/tiff                            tif tiff;
    image/vnd.wap.wbmp                    wbmp;
    image/x-icon                          ico;
    image/x-jng                           jng;
    image/x-ms-bmp                        bmp;
    image/svg+xml                         svg svgz;
    image/webp                            webp;

    application/font-woff                 woff;
    application/java-archive              jar war ear;
    application/json                      json;
    application/mac-binhex40              hqx;
    application/msword                    doc;
    application/pdf                       pdf;
    application/postscript                ps eps ai;
    application/rtf                       rtf;
    application/vnd.apple.mpegurl         m3u8;
    application/vnd.ms-excel              xls;
    application/vnd.ms-fontobject         eot;
    application/vnd.ms-powerpoint         ppt;
    application/vnd.wap.wmlc              wmlc;
    application/vnd.google-earth.kml+xml  kml;
    application/vnd.google-earth.kmz      kmz;
    application/x-7z-compressed           7z;
    application/x-cocoa                   cco;
    application/x-java-archive-diff       jardiff;
    application/x-java-jnlp-file          jnlp;
    application/x-makeself                run;
    application/x-perl                    pl pm;
    application/x-pilot                   prc pdb;
    application/x-rar-compressed          rar;
    application/x-redhat-package-manager  rpm;
    application/x-sea                     sea;
    application/x-shockwave-flash         swf;
    application/x-stuffit                 sit;
    application/x-tcl                     tcl tk;
    application/x-x509-ca-cert            der pem crt;
    application/x-xpinstall               xpi;
    application/xhtml+xml                 xhtml;
    application/xspf+xml                  xspf;
    application/zip                       zip;

    application/octet-stream              bin exe dll;
    application/octet-stream              deb;
    application/octet-stream              dmg;
    application/octet-stream              iso img;
    application/octet-stream              msi msp msm;

    application/vnd.openxmlformats-officedocument.wordprocessingml.document    docx;
    application/vnd.openxmlformats-officedocument.spreadsheetml.sheet          xlsx;
    application/vnd.openxmlformats-officedocument.presentationml.presentation  pptx;

    audio/midi                            mid midi kar;
    audio/mpeg                            mp3;
    audio/ogg                             ogg;
    audio/x-m4a                           m4a;
    audio/x-realaudio                     ra;

    video/3gpp                            3gpp 3gp;
    video/mp2t                            ts;
    video/mp4                             mp4;
    video/mpeg                            mpeg mpg;
    video/quicktime                       mov;
    video/webm                            webm;
    video/x-flv                           flv;
    video/x-m4v                           m4v;
    video/x-mng                           mng;
    video/x-ms-asf                        asx asf;
    video/x-ms-wmv                        wmv;
    video/x-msvideo                       avi;
}


```

##  附录配置文件与指令解析 
###  通用配置 
下面的nginx.conf简单的实现nginx在前端做反向代理服务器的例子，处理js、png等静态文件，jsp等动态请求转发到其它服务器tomcat：
```

user www www;
worker_processes 2;
  
error_log logs/error.log;
#error_log logs/error.log notice;
#error_log logs/error.log info;
  
pid logs/nginx.pid;
  
  
events {
use epoll;
worker_connections 2048;
}
  
  
http {
include mime.types;
default_type application/octet-stream;
  
#log_format main '$remote_addr - $remote_user [$time_local] "$request" '
# '$status $body_bytes_sent "$http_referer" '
# '"$http_user_agent" "$http_x_forwarded_for"';
  
#access_log logs/access.log main;
  
sendfile on;
# tcp_nopush on;
  
keepalive_timeout 65;
  
# gzip压缩功能设置
gzip on;
gzip_min_length 1k;
gzip_buffers 4 16k;
gzip_http_version 1.0;
gzip_comp_level 6;
gzip_types text/html text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
gzip_vary on;
  
# http_proxy 设置
client_max_body_size 10m;
client_body_buffer_size 128k;
proxy_connect_timeout 75;
proxy_send_timeout 75;
proxy_read_timeout 75;
proxy_buffer_size 4k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
proxy_temp_path /usr/local/nginx/proxy_temp 1 2;
  
# 设定负载均衡后台服务器列表
upstream backend {
#ip_hash;
server 192.168.10.100:8080 max_fails=2 fail_timeout=30s ;
server 192.168.10.101:8080 max_fails=2 fail_timeout=30s ;
}
  
# 很重要的虚拟主机配置
server {
listen 80;
server_name itoatest.example.com;
root /apps/oaapp;
  
charset utf-8;
access_log logs/host.access.log main;
  
#对 / 所有做负载均衡+反向代理
location / {
root /apps/oaapp;
index index.jsp index.html index.htm;
  
proxy_pass http://backend;
proxy_redirect off;
# 后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
  
}
  
#静态文件，nginx自己处理，不去backend请求tomcat
location ~* /download/ {
root /apps/oa/fs;
  
}
location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)$
{
root /apps/oaapp;
expires 7d;
}
location /nginx_status {
stub_status on;
access_log off;
allow 192.168.10.0/24;
deny all;
}
  
location ~ ^/(WEB-INF)/ {
deny all;
}
#error_page 404 /404.html;
  
# redirect server error pages to the static page /50x.html
#
error_page 500 502 503 504 /50x.html;
location = /50x.html {
root html;
}
}
  
## 其它虚拟主机，server 指令开始
}

```
###  常用指令说明 
####  main全局配置 

nginx在运行时与具体业务功能（比如http服务或者email服务代理）无关的一些参数，比如工作进程数，运行的身份等。

woker_processes 2
在配置文件的顶级main部分，worker角色的工作进程的个数，master进程是接收并分配请求给worker处理。这个数值简单一点可以设置为cpu的核数grep ^processor /proc/cpuinfo | wc -l，也是 auto 值，如果开启了ssl和gzip更应该设置成与逻辑CPU数量一样甚至为2倍，可以减少I/O操作。如果nginx服务器还有其它服务，可以考虑适当减少。

worker_cpu_affinity
也是写在main部分。在高并发情况下，通过设置cpu粘性来降低由于多CPU核切换造成的寄存器等现场重建带来的性能损耗。如worker_cpu_affinity 0001 0010 0100 1000; （四核）。

worker_connections 2048
写在events部分。每一个worker进程能并发处理（发起）的最大连接数（包含与客户端或后端被代理服务器间等所有连接数）。nginx作为反向代理服务器，计算公式 最大连接数 = worker_processes * worker_connections/4，所以这里客户端最大连接数是1024，这个可以增到到8192都没关系，看情况而定，但不能超过后面的worker_rlimit_nofile。当nginx作为http服务器时，计算公式里面是除以2。

worker_rlimit_nofile 10240
写在main部分。默认是没有设置，可以限制为操作系统最大的限制65535。

use epoll
写在events部分。在Linux操作系统下，nginx默认使用epoll事件模型，得益于此，nginx在Linux操作系统下效率相当高。同时Nginx在OpenBSD或FreeBSD操作系统上采用类似于epoll的高效事件模型kqueue。在操作系统不支持这些高效模型时才使用select。

####   http服务器 

与提供http服务相关的一些配置参数。例如：是否使用keepalive啊，是否使用gzip进行压缩等。

sendfile on
开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，减少用户空间到内核空间的上下文切换。对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。

keepalive_timeout 65 : 长连接超时时间，单位是秒，这个参数很敏感，涉及浏览器的种类、后端服务器的超时设置、操作系统的设置，可以另外起一片文章了。长连接请求大量小文件的时候，可以减少重建连接的开销，但假如有大文件上传，65s内没上传完成会导致失败。如果设置时间过长，用户又多，长时间保持连接会占用大量资源。

send_timeout : 用于指定响应客户端的超时时间。这个超时仅限于两个连接活动之间的时间，如果超过这个时间，客户端没有任何活动，Nginx将会关闭连接。

client_max_body_size 10m
允许客户端请求的最大单文件字节数。如果有上传较大文件，请设置它的限制值

client_body_buffer_size 128k
缓冲区代理缓冲用户端请求的最大字节数

模块http_proxy：
这个模块实现的是nginx作为反向代理服务器的功能，包括缓存功能（另见文章）

proxy_connect_timeout 60
nginx跟后端服务器连接超时时间(代理连接超时)

proxy_read_timeout 60
连接成功后，与后端服务器两个成功的响应操作之间超时时间(代理接收超时)

proxy_buffer_size 4k
设置代理服务器（nginx）从后端realserver读取并保存用户头信息的缓冲区大小，默认与proxy_buffers大小相同，其实可以将这个指令值设的小一点
proxy_buffers 4 32k
proxy_buffers缓冲区，nginx针对单个连接缓存来自后端realserver的响应，网页平均在32k以下的话，这样设置
proxy_busy_buffers_size 64k
高负荷下缓冲大小（proxy_buffers*2）
proxy_max_temp_file_size
当 proxy_buffers 放不下后端服务器的响应内容时，会将一部分保存到硬盘的临时文件中，这个值用来设置最大临时文件大小，默认1024M，它与 proxy_cache 没有关系。大于这个值，将从upstream服务器传回。设置为0禁用。
proxy_temp_file_write_size 64k
当缓存被代理的服务器响应到临时文件时，这个选项限制每次写临时文件的大小。proxy_temp_path（可以在编译的时候）指定写到哪那个目录。
proxy_pass，proxy_redirect见 location 部分。

== 模块http_gzip： ==

gzip on : 开启gzip压缩输出，减少网络传输。
  * gzip_min_length 1k ： 设置允许压缩的页面最小字节数，页面字节数从header头得content-length中进行获取。默认值是20。建议设置成大于1k的字节数，小于1k可能会越压越大。
  * gzip_buffers 4 16k ： 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。4 16k代表以16k为单位，安装原始数据大小以16k为单位的4倍申请内存。
  * gzip_http_version 1.0 ： 用于识别 http 协议的版本，早期的浏览器不支持 Gzip 压缩，用户就会看到乱码，所以为了支持前期版本加上了这个选项，如果你用了 Nginx 的反向代理并期望也启用 Gzip 压缩的话，由于末端通信是 http/1.0，故请设置为 1.0。
  * gzip_comp_level 6 ： gzip压缩比，1压缩比最小处理速度最快，9压缩比最大但处理速度最慢(传输快但比较消耗cpu)
  * gzip_types ：匹配mime类型进行压缩，无论是否指定,”text/html”类型总是会被压缩的。
  * gzip_proxied any ： Nginx作为反向代理的时候启用，决定开启或者关闭后端服务器返回的结果是否压缩，匹配的前提是后端服务器必须要返回包含”Via”的 header头。
  * gzip_vary on ： 和http头有关系，会在响应头加个 Vary: Accept-Encoding ，可以让前端的缓存服务器缓存经过gzip压缩的页面，例如，用Squid缓存经过Nginx压缩的数据。。

####  server虚拟主机 

http服务上支持若干虚拟主机。每个虚拟主机一个对应的server配置项，配置项里面包含该虚拟主机相关的配置。在提供mail服务的代理时，也可以建立若干server。每个server通过监听地址或端口来区分。

listen
监听端口，默认80，小于1024的要以root启动。可以为listen *:80、listen 127.0.0.1:80等形式。
server_name
服务器名，如localhost、www.example.com，可以通过正则匹配。
模块http_stream
这个模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡，upstream后接负载均衡器的名字，后端realserver以 host:port options; 方式组织在 {} 中。如果后端被代理的只有一台，也可以直接写在 proxy_pass 。

####  location 

http服务中，某些特定的URL对应的一系列配置项。

root /var/www/html
定义服务器的默认网站根目录位置。如果locationURL匹配的是子目录或文件，root没什么作用，一般放在server指令里面或/下。
index index.jsp index.html index.htm
定义路径下默认访问的文件名，一般跟着root放
proxy_pass http:/backend
请求转向backend定义的服务器列表，即反向代理，对应upstream负载均衡器。也可以proxy_pass http://ip:port。
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
这四个暂且这样设，如果深究的话，每一个都涉及到很复杂的内容，也将通过另一篇文章来解读。

####  访问控制 allow/deny 
Nginx 的访问控制模块默认就会安装，而且写法也非常简单，可以分别有多个allow,deny，允许或禁止某个ip或ip段访问，依次满足任何一个规则就停止往下匹配。如：
```

location /nginx-status {
stub_status on;
access_log off;
# auth_basic "NginxStatus";
# auth_basic_user_file /usr/local/nginx-1.6/htpasswd;
  
allow 192.168.10.100;
allow 172.29.73.0/24;
deny all;
}

```
我们也常用 httpd-devel 工具的 htpasswd 来为访问的路径设置登录密码：
```

# htpasswd -c htpasswd admin
New passwd:
Re-type new password:
Adding password for user admin
  
# htpasswd htpasswd admin //修改admin密码
# htpasswd htpasswd sean //多添加一个认证用户

```
这样就生成了默认使用CRYPT加密的密码文件。打开上面nginx-status的两行注释，重启nginx生效。

####  列出目录 autoindex 

Nginx默认是不允许列出整个目录的。如需此功能，打开nginx.conf文件，在location，server 或 http段中加入autoindex on;，另外两个参数最好也加上去:

autoindex_exact_size off; 默认为on，显示出文件的确切大小，单位是bytes。改为off后，显示出文件的大概大小，单位是kB或者MB或者GB
autoindex_localtime on;
默认为off，显示的文件时间为GMT时间。改为on后，显示的文件时间为文件的服务器时间
```

location /images {
root /var/www/nginx-default/images;
autoindex on;
autoindex_exact_size off;
autoindex_localtime on;
}

```
参考：http://blog.jobbole.com/87538/
参考：http://blog.jobbole.com/91852/