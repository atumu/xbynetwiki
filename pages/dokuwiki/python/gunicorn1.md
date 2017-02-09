title: gunicorn1 

#  gunicorn入门 
文档：http://docs.gunicorn.org/en/stable/

Gunicorn 绿色独角兽'是一个Python WSGI UNIX的HTTP服务器。
这是一个pre-fork worker的模型，从Ruby的独角兽（Unicorn ）项目移植。
该Gunicorn服务器大致与各种Web框架兼容，只需非常简单的执行，轻量级的资源消耗，以及相当迅速。
Gunicorn运行Python的网站真是非常简单.

pip install gunicorn

Async Workers
$ pip install greenlet  # Required for both
$ pip install eventlet  # For eventlet workers
$ pip install gevent    # For gevent workers

不过还是建议使用系统包的形式安装，而不是pip安装gunicorn
$ sudo apt-get install gunicorn
/etc/gunicorn.d. /var/log/gunicorn

运行：
$ gunicorn [OPTIONS] APP_MODULE
Where APP_MODULE is of the pattern $(MODULE_NAME):$(VARIABLE_NAME)

```

def app(environ, start_response):
    """Simplest possible application object"""
    data = 'Hello, World!\n'
    status = '200 OK'
    response_headers = [
        ('Content-type','text/plain'),
        ('Content-Length', str(len(data)))
    ]
    start_response(status, response_headers)
    return iter([data])

```
```

$ gunicorn --workers=2 test:app
$ gunicorn -b 127.0.0.1:8000 -b [::1]:8000 test:app


```
常用命令行选项：
-c CONFIG, - -config=CONFIG - Specify a config file in the form $(PATH), file:$(PATH), or python:$(MODULE_NAME).
-b BIND, - -bind=BIND - Specify a server socket to bind. Server sockets can be any of $(HOST), $(HOST):$(PORT), or unix:$(PATH). An IP is a valid $(HOST).
-w WORKERS, - -workers=WORKERS - The number of worker processes. This number should **generally be between 2-4 workers per core** in the server.
**-k** WORKERCLASS, - -worker-class=WORKERCLASS - The type of worker process to run. You’ll definitely want to read the production page for the implications of this parameter. You can set this to $(NAME) **where $(NAME) is one of sync, eventlet, gevent, or tornado, gthread, gaiohttp. sync is the default.**
-n APP_NAME, - -name=APP_NAME - If setproctitle is installed you can adjust the name of Gunicorn process as they appear in the process system table (which affects tools like ps and top).
##  配置大全 
http://docs.gunicorn.org/en/stable/settings.html
##  与Nginx配合 
```

worker_processes 1;

user nobody nogroup;
# 'user nobody nobody;' for systems with 'nobody' as a group instead
pid /tmp/nginx.pid;
error_log /tmp/nginx.error.log;

events {
  worker_connections 1024; # increase if you have lots of clients
  accept_mutex off; # set to 'on' if nginx worker_processes > 1
  # 'use epoll;' to enable for Linux 2.6+
  # 'use kqueue;' to enable for FreeBSD, OSX
}

http {
  include mime.types;
  # fallback in case we can't determine a type
  default_type application/octet-stream;
  access_log /tmp/nginx.access.log combined;
  sendfile on;

  upstream app_server {
    # fail_timeout=0 means we always retry an upstream even if it failed
    # to return a good HTTP response

    # for UNIX domain socket setups
    server unix:/tmp/gunicorn.sock fail_timeout=0;

    # for a TCP configuration
    # server 192.168.0.7:8000 fail_timeout=0;
  }

  server {
    # if no Host match, close the connection to prevent host spoofing
    listen 80 default_server;
    return 444;
  }

  server {
    # use 'listen 80 deferred;' for Linux
    # use 'listen 80 accept_filter=httpready;' for FreeBSD
    listen 80;
    client_max_body_size 4G;

    # set the correct host(s) for your site
    server_name example.com www.example.com;

    keepalive_timeout 5;

    # path for static files
    root /path/to/app/current/public;

    location / {
      # checks for static file, if not found proxy to app
      try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      # enable this if and only if you use HTTPS
      # proxy_set_header X-Forwarded-Proto https;
      proxy_set_header Host $http_host;
      # we don't want nginx trying to do something clever with
      # redirects, we set the Host: header above already.
      proxy_redirect off;
      proxy_pass http://app_server;
    }

    error_page 500 502 503 504 /500.html;
    location = /500.html {
      root /path/to/app/current/public;
    }
  }
}

```

**如果你想能处理流式请求/相应，或者类似于Comet,长轮询，Web sockets等，你需要关闭代理缓存.proxy_buffering off;
同时你必须给gunicorn启动时指定一个异步worker类async worker classes.**
```

location @proxy_to_app {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; #确保用户真实IP发送给应用服务器REMOTE_ADDR
    #proxy_set_header Host $http_host;
    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_redirect off;
    proxy_buffering off;

    proxy_pass http://app_server;
}

```
**如果采用https，记得配置：
proxy_set_header X-Forwarded-Proto $scheme;**

**如果你的nginx和gunicorn不在同一台机器上**，那么你需要让gunicorn信任Nginx发送的 X-Forwarded-*头。
默认情况下 Gunicorn 只会信任本地的。
```

$ gunicorn -w 3 --forwarded-allow-ips="10.170.3.217,10.170.3.220" test:app

```


##  结合Supervisor 
```

[program:gunicorn]
command=/path/to/gunicorn main:application -c /path/to/gunicorn.conf.py
directory=/path/to/project
user=nobody
autostart=true
autorestart=true
redirect_stderr=true

```

##  Flask+Gunicorn+Gevent 
http://docs.jinkan.org/docs/flask/deploying/wsgi-standalone.html
在这个服务器上运行 Flask 应用是相当简单的:
```

gunicorn myproject:app

```
Gunicorn 提供了许多命令行选项 —— 见 gunicorn -h 。
例如，用四个 worker 进程（ gunicorn -h ）来运行一个 Flask 应用，绑定到 localhost 的4000 端口（ -b 127.0.0.1:4000 ）:
```

gunicorn -w 4 -b 127.0.0.1:4000 myproject:app

```
greenlet是一个轻量级的协程库。gevent是基于greenlet的网络库。 
guincorn是支持wsgi协议的http server，gevent只是它支持的模式之一
**配合gevent**
另外， gunicorn 默认使用同步阻塞的网络模型(-k sync)，对于大并发的访问可能表现不够好，它还支持其它更好的模式，比如：gevent或meinheld。
```

# gevent
gunicorn -k gevent code:application

```

**指定配置文件**
以上设置还可以通过 -c 参数传入一个配置文件实现。 
```

gunicorn -c gun.py hello:app

```
```

# cat gun.py
import os
import multiprocessing

bind = '0.0.0.0:8000'
workers = 4
#backlog = 2048
worker_class = "gevent"
#debug = True
proc_name = 'gunicorn.proc'
#pidfile = '/tmp/gunicorn.pid'
logfile = '/var/log/gunicorn/debug.log'
loglevel = 'debug'
#启动的进程数
#workers = multiprocessing.cpu_count() * 2 + 1 
#worker_class = 'gunicorn.workers.ggevent.GeventWorker'
#x_forwarded_for_header = 'X-FORWARDED-FOR'

```

参考http://blog.csdn.net/dutsoft/article/details/51452598
http://www.jianshu.com/p/be9dd421fb8d
http://www.open-open.com/lib/view/open1423107543014.html
http://www.tuicool.com/articles/aiami2