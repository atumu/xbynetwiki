title: nginx编译安装 

#  nginx编译安装 
##  编译安装 
首先安装依赖
sudo apt-get install gcc gcc-c++ make libtool zlib zlib-devel openssl openssl-devel pcre pcre-devel
从 http://nginx.org/en/download.html 下载稳定版nginx-1.6.3.tar.gz到/usr/local/src下解压。
为了后续准备我们另外下载2个插件模块：nginx_upstream_check_module-0.3.0.tar.gz —— 检查后端服务器的状态，nginx-goodies-nginx-sticky-module-ng-bd312d586752.tar.gz（建议在/usr/local/src下解压后将目录重命名为nginx-sticky-module-ng-1.2.5） —— 后端做负载均衡解决session sticky问题
请注意插件与nginx的版本兼容问题，一般插件越新越好，nginx不用追新，稳定第一。nginx-1.4.7，nginx-sticky-module-1.1，nginx_upstream_check_module-0.2.0，这个搭配也没问题。sticky-1.1与nginx-1.6版本由于更新没跟上编译出错。（可以直接使用Tengine，默认就包括了这些模块）
```

[root@cachets nginx-1.6.3]
# pwd
/usr/local/src/nginx-1.6.3
[root@cachets nginx-1.6.3]
# ./configure --prefix=/usr/local/nginx-1.6 --with-pcre \
> --with-http_stub_status_module --with-http_ssl_module \
> --with-http_gzip_static_module --with-http_realip_module \
> --add-module=../nginx_upstream_check_module-0.3.0
  
[root@cachets nginx-1.6.3]
# make && make install

```
##  常用编译选项说明 
nginx大部分常用模块，编译时./configure --help以--with开头的都默认安装。

--prefix=PATH ： 指定nginx的安装目录。默认 /usr/local/nginx
--conf-path=PATH ： 设置nginx.conf配置文件的路径。nginx允许使用不同的配置文件启动，通过命令行中的-c选项。默认为prefix/conf/nginx.conf
--user=name： 设置nginx工作进程的用户。安装完成后，可以随时在nginx.conf配置文件更改user指令。默认的用户名是nobody。--group=name类似
--with-pcre ： 设置PCRE库的源码路径，如果已通过yum方式安装，使用--with-pcre自动找到库文件。使用--with-pcre=PATH时，需要从PCRE网站下载pcre库的源码（版本4.4 – 8.30）并解压，剩下的就交给Nginx的./configure和make来完成。perl正则表达式使用在location指令和 ngx_http_rewrite_module模块中。
--with-zlib=PATH ： 指定 zlib（版本1.1.3 – 1.2.5）的源码解压目录。在默认就启用的网络传输压缩模块ngx_http_gzip_module时需要使用zlib 。
--with-http_ssl_module ： 使用https协议模块。默认情况下，该模块没有被构建。前提是openssl与openssl-devel已安装
--with-http_stub_status_module ： 用来监控 Nginx 的当前状态
--with-http_realip_module ： 通过这个模块允许我们改变客户端请求头中客户端IP地址值(例如X-Real-IP 或 X-Forwarded-For)，意义在于能够使得后台服务器记录原始客户端的IP地址
--add-module=PATH ： 添加第三方外部模块，如nginx-sticky-module-ng或缓存模块。每次添加新的模块都要重新编译（Tengine可以在新加入module时无需重新编译）

##  再提供一种编译方案 
```

./configure \
> --prefix=/usr \
> --sbin-path=/usr/sbin/nginx \
> --conf-path=/etc/nginx/nginx.conf \
> --error-log-path=/var/log/nginx/error.log \
> --http-log-path=/var/log/nginx/access.log \
> --pid-path=/var/run/nginx/nginx.pid \
> --lock-path=/var/lock/nginx.lock \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_stub_status_module \
> --with-http_gzip_static_module \
> --http-client-body-temp-path=/var/tmp/nginx/client/ \
> --http-proxy-temp-path=/var/tmp/nginx/proxy/ \
> --http-fastcgi-temp-path=/var/tmp/nginx/fcgi/ \
> --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi \
> --with-pcre=../pcre-7.8
> --with-zlib=../zlib-1.2.3

```
##  启动关闭nginx 
## 检查配置文件是否正确
# /usr/local/nginx-1.6/sbin/nginx -t
# ./sbin/nginx -V # 可以看到编译选项
  
## 启动、关闭
# ./sbin/nginx # 默认配置文件 conf/nginx.conf，-c 指定
# ./sbin/nginx -s stop
或 pkill nginx
  
## 重启，不会改变启动时指定的配置文件
# ./sbin/nginx -s reload
或 kill -HUP `cat /usr/local/nginx-1.6/logs/nginx.pid`
当然也可以将 nginx 作为系统服务管理，下载 nginx 到/etc/init.d/，修改里面的路径然后赋予可执行权限。
# service nginx {start|stop|status|restart|reload|configtest}

##  nginx启动脚本编写 
###  linux下 
```

[root@kallen init.d]# vi /etc/init.d/nginx

```
```

#!/bin/bash
#        
# Startup script for the Nginx HTTP Server
#
# Kallen Ding, Apr 23 2015
 
NGINX_PATH="/usr/local/nginx/"
OPTIONS="-c ${NGINX_PATH}conf/nginx.conf"
prog=nginx
nginx=${NGINX_PATH}sbin/nginx
pidfile=/var/run/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
start() {
    echo -n "Starting $prog: "
    daemon --pidfile=${pidfile} $nginx $OPTIONS
    RETVAL=$?
    echo
    return $RETVAL
}
 
stop() {
    echo -n "Stopping $prog: "
    killproc -p ${pidfile} $nginx
    RETVAL=$?
    echo
}
reload() {
    echo -n $"Reloading $prog: "
    killproc -p ${pidfile} $nginx -HUP
    RETVAL=$?
    echo
}
 
# See how we were called.
case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart)
    stop
    start
    ;;
  reload)
        reload
    ;;
  status)
    status $prog
    RETVAL=$?
    ;;
  *)
    echo "Usage: $prog {start|stop|restart|reload|status}"
    RETVAL=3
esac
 
exit $RETVAL

```
保存后，更改/etc/init.d/nginx的权限
```

[root@kallen ~]# chmod 755 /etc/init.d/nginx
[root@kallen ~]# chkconfig --add nginx
[root@kallen ~]# chkconfig nginx on

```
###  windows下 
start.bat:
```

start nginx

```
stop.bat:
```

nginx -s stop
pkill -9 nginx

```
reload.bat:
```

nginx -s reload

```
  
参考：http://blog.jobbole.com/91107/
