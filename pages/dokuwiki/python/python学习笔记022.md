title: python学习笔记022 

#  Python学习笔记之Nginx+uwsgi PythonWeb应用部署环境 
uWSGI 是在像 nginx 、 lighttpd 以及 cherokee 服务器上的一个部署的选择。更多选择见 FastCGI 和 独立 WSGI 容器 。 
你会首先需要一个 uWSGI 服务器来用 uWSGI 协议来使用你的 WSGI 应用。 uWSGI 是一个协议，同样也是一个应用服务器，可以提供 uWSGI 、FastCGI 和 HTTP 协议。
**最流行的 uWSGI 服务器是 uwsgi** ，我们会在本指导中使用。确保你已经安装好它来跟随下面的说明。

##  Django中Nginx、uWSGI配置 
###  三：安装uwsgi 
uwsgi:https://pypi.python.org/pypi/uWSGI
uwsgi参数详解：http://uwsgi-docs.readthedocs.org/en/latest/Options.html
```

pip install uwsgi
uwsgi --version

```
测试uwsgi是否正常：
新建test.py文件，内容如下：
```

def application(env, start_response):
        start_response('200 OK', [('Content-Type','text/html')])
        return "Hello World"

```
然后在终端运行：
```

uwsgi --http :8001 --wsgi-file test.py

```
在浏览器内输入：http://127.0.0.1:8001，看是否有“Hello World”输出，若没有输出，请检查你的安装过程。

四：安装django
pip install django
测试django是否正常，运行：
django-admin.py startproject demosite
cd demosite
python2.7 manage.py runserver 0.0.0.0:8002
在浏览器内输入：http://127.0.0.1:8002，检查django是否运行正常。

###  六：配置uwsgi 
uWSGI 是一款像php-cgi一样监听同一端口，进行统一管理和负载平衡的工具，uWSGI，既不用wsgi协议也不用fcgi协议，而是自创了一个uwsgi的协议，据说该协议大约是fcgi协议的10倍那么快。
uWSGI的主要特点如下：
  * 超快的性能。
  * 低内存占用（实测为apache2的mod_wsgi的一半左右）。
  * 多app管理。
  * 详尽的日志功能（可以用来分析app性能和瓶颈）。
  * 高度可定制（内存大小限制，服务一定次数后重启等）。
uwsgi的官方文档：http://projects.unbit.it/uwsgi/wiki/Doc
uwsgi支持ini、xml等多种配置方式，但个人感觉ini更方便：
在/ect/目录下新建uwsgi9090.ini，添加如下配置：
```

[uwsgi]
socket = 127.0.0.1:9090
master = true         //主进程
vhost = true          //多站模式
no-stie = true        //多站模式时不设置入口模块和文件
workers = 2           //子进程数
reload-mercy = 10     
vacuum = true         //退出、重启时清理文件
max-requests = 1000   
limit-as = 512
buffer-sizi = 30000
pidfile = /var/run/uwsgi9090.pid    //pid文件，用于下面的脚本启动、停止该进程
daemonize = /website/uwsgi9090.log

```
设置uwsgi开机启动，在/ect/init.d/目录下新建uwsgi9090文件，内容如下：
```

#! /bin/sh
# chkconfig: 2345 55 25
# Description: Startup script for uwsgi webserver on Debian. Place in /etc/init.d and
# run 'update-rc.d -f uwsgi defaults', or use the appropriate command on your
# distro. For CentOS/Redhat run: 'chkconfig --add uwsgi'
 
### BEGIN INIT INFO
# Provides:          uwsgi
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the uwsgi web server
# Description:       starts uwsgi using start-stop-daemon
### END INIT INFO
 
# Author:   licess
# website:  http://lnmp.org
 
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DESC="uwsgi daemon"
NAME=uwsgi9090
DAEMON=/usr/local/bin/uwsgi
CONFIGFILE=/etc/$NAME.ini
PIDFILE=/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
 
set -e
[ -x "$DAEMON" ] || exit 0
 
do_start() {
    $DAEMON $CONFIGFILE || echo -n "uwsgi already running"
}
 
do_stop() {
    $DAEMON --stop $PIDFILE || echo -n "uwsgi not running"
    rm -f $PIDFILE
    echo "$DAEMON STOPED."
}
 
do_reload() {
    $DAEMON --reload $PIDFILE || echo -n "uwsgi can't reload"
}
 
do_status() {
    ps aux|grep $DAEMON
}
 
case "$1" in
 status)
    echo -en "Status $NAME: \n"
    do_status
 ;;
 start)
    echo -en "Starting $NAME: \n"
    do_start
 ;;
 stop)
    echo -en "Stopping $NAME: \n"
    do_stop
 ;;
 reload|graceful)
    echo -en "Reloading $NAME: \n"
    do_reload
 ;;
 *)
    echo "Usage: $SCRIPTNAME {start|stop|reload}" >&2
    exit 3
 ;;
esac
 
exit 0

uwsgi9090

```
然后在终端执行：
```

-- 添加服务
chkconfig --add uwsgi9090 
-- 设置开机启动
chkconfig uwsgi9090 on

```
###  uwsgi的参数 
3.uwsgi的参数
```

uwsgi还是有很多令人称赞的功能的，例如：
并发4个线程：uwsgi -s :9090 -w myapp -p 4 
主控制线程+4个线程：uwsgi -s :9090 -w myapp -M -p 4 
执行超过30秒的client直接放弃：uwsgi -s :9090 -w myapp -M -p 4 -t 30 
限制内存空间128M： uwsgi -s :9090 -w myapp -M -p 4 -t 30 --limit-as 128 
服务超过10000个req自动respawn：uwsgi -s :9090 -w myapp -M -p 4 -t 30 --limit-as 128 -R 10000 
后台运行等：uwsgi -s :9090 -w myapp -M -p 4 -t 30 --limit-as 128 -R 10000 -d uwsgi.log 
4.为uwsgi配置多个站点
为了让多个站点共享一个uwsgi服务，必须把uwsgi运行成虚拟站点：去掉“-w myapp”加上”–vhost”：
uwsgi -s :9090 -M -p 4 -t 30 --limit-as 128 -R 10000 -d uwsgi.log --vhost

``` 

###  七：设置nginx 
找到nginx的安装目录，打开conf/nginx.conf文件，修改server配置
```

server {
        listen       80;
        server_name  localhost;
        
        location / {            
            include  uwsgi_params;
            uwsgi_pass  127.0.0.1:9090;              //必须和uwsgi中的设置一致
            uwsgi_param UWSGI_SCRIPT demosite.wsgi;  //入口文件，即wsgi.py相对于项目根目录的位置，“.”相当于一层目录
            uwsgi_param UWSGI_CHDIR /demosite;       //项目根目录
            index  index.html index.htm;
            client_max_body_size 35m;
        }
    }

```
设置nginx开机启动，在/ect/init.d/目录下新建nginx文件，内容如下：
```

#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:     /var/run/nginx.pid
  
# Source function library.
. /etc/rc.d/init.d/functions
  
# Source networking configuration.
. /etc/sysconfig/network
  
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
  
nginx="/opt/nginx-1.5.6/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/opt/nginx-1.5.6/conf/nginx.conf"
  
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
  
lockfile=/var/lock/subsys/nginx  
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
  
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
  
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
  
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
  
force_reload() {
    restart
}
  
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
  
rh_status() {
    status $prog
}
  
rh_status_q() {
    rh_status >/dev/null 2>&1
}
  
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac

nginx

```
```

然后在终端执行：

-- 添加服务
chkconfig --add nginx 
-- 设置开机启动
chkconfig nginx on

```
八：测试
OK，一切配置完毕，在终端运行
service uwsgi9090 start
service nginx start
在浏览器输入：http://127.0.0.1，恭喜你可以看到django的“It work”了~
###  九：多站配置 
我采用运行多个uwsgi服务的方法来实现多个站点。
重复第六步，创建uwsgi9091.ini，并相应修改文件中的
socket = 127.0.0.1:9091
pidfile = /var/run/uwsgi9091.pid
daemonize = /website/uwsgi9091.log
并创建服务uwsgi9091，设置开机启动。
然后修改nginx的配置文件为：
```

server {
        listen       80;
        server_name  localhost;
        
        location / {            
            include  uwsgi_params;
            uwsgi_pass  127.0.0.1:9090;
            uwsgi_param UWSGI_SCRIPT demosite.wsgi;
            uwsgi_param UWSGI_CHDIR /website/demosite;
            index  index.html index.htm;
            client_max_body_size 35m;
        }
    }

    server {
        listen       1300;
        
        location / {            
            include  uwsgi_params;
            uwsgi_pass  127.0.0.1:9091;
            uwsgi_param UWSGI_SCRIPT DjangoStudy.wsgi;
            uwsgi_param UWSGI_CHDIR /website/DjangoStudy;
            index  index.html index.htm;
        }
    }

nginx

```
十：其他配置
###  防火墙设置 
CentOS默认关闭外部对80、3306等端口的访问，所以要在其他计算机访问这台服务器，就必须修改防火墙配置，打开` /etc/sysconfig/iptables `
```

在“-A INPUT –m state --state NEW –m tcp –p –dport 22 –j ACCEPT”，下添加：
-A INPUT -m state --state NEW -m tcp -p -dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p -dport 3306 -j ACCEPT
然后保存，并关闭该文件，在终端内运行下面的命令，刷新防火墙配置：
service iptables restart

```
安装Mysqldb
yum -y install mysql-devel
easy_install-2.7 MySQL-python
注意红色部分，easy_install-2.7,否则它将默认安装到Python2.6环境内。

2014年12月02日添加： 
CentOS 7中默认使用Firewalld做防火墙，所以修改iptables后，在重启系统后，根本不管用。 
```

Firewalld中添加端口方法如下： 
firewall-cmd --zone=public --add-port=3306/tcp --permanent 
firewall-cmd --reload

```
作者：Xiongpq 
出处：http://xiongpq.cnblogs.com/ 
本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文连接，否则保留追究法律责任的权利。

##  Flask中的nginx配置 
注意使用Flask框架时：**请提前确保你在应用文件中的任何 app.run() 调用在 if __name__ == '__main__': 块中**或是移到一个独立的文件。这是因为它总会启动一个本地的 WSGI 服务器，并且我们在部署应用到 uWSGI 时不需要它。
### = 用 uwsgi 启动你的应用 
uwsgi 被设计为操作在 python 模块中找到的 WSGI 可调用量。
已知在 myapp.py 中有一个 flask 应用，使用下面的命令:
```

$ uwsgi -s /tmp/uwsgi.sock --module myapp --callable app
或者，你喜欢这样:
$ uwsgi -s /tmp/uwsgi.sock -w myapp:app

```
### = 配置 nginx 
**一个基本的 flaks uWSGI 的给 nginx 的配置看起来是这样**:
```

location = /yourapplication { rewrite ^ /yourapplication/; }
location /yourapplication { try_files $uri @yourapplication; }
location @yourapplication {
  include uwsgi_params;
  uwsgi_param SCRIPT_NAME /yourapplication;
  uwsgi_modifier1 30;
  uwsgi_pass unix:/tmp/uwsgi.sock;
}

```
这个配置绑定应用到 /yourapplication 。如果你想要绑定到 URL 根会更简单，因你不许要告诉它 WSGI SCRIPT_NAME 或设置 uwsgi modifier 来使用它:
```

location / { try_files $uri @yourapplication; }
location @yourapplication {
    include uwsgi_params;
    uwsgi_pass unix:/tmp/uwsgi.sock;
}

```
参考：http://www.cnblogs.com/xiongpq/p/3381069.html
http://sjolzy.cn/Nginx-uwsgi-rapidly-deploy-Python-applications.html