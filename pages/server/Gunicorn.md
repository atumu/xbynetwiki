modifyAt:2016-12-11 14:13:15
location:server/Gunicorn
title:Gunicorn独立WSGI容器
author:xbynet
createAt:2016-12-11 14:13:15

官网：http://gunicorn.org/
与Flask集成:http://docs.jinkan.org/docs/flask/deploying/wsgi-standalone.html
注意事项：不支持windows
安装
`$ pip install gunicorn`
但是官方不建议这么做，这样安装有些缺点，建议使用系统包的方式安装：
`$ sudo apt-get update`
`$ sudo apt-get install gunicorn`
以系统包的方式安装优点：
 
* 通过配置/etc/gunicorn.d可以自动启动多个gunicorn实例
* 在/var/log/gunicorn记录滚动日志
* 安全性更好，可以以对应linux用户/组来运行不同的gunicorn实例
* 更便捷得版本更新

如果需要配合greenlet处理异步workers：
`$ sudo apt-get install libevent`
```
$ pip install greenlet 
#$ pip install eventlet  # For eventlet workers
$ pip install gevent    # For gevent workers
```

Gunicorn ‘Green Unicorn’ 是一个给 UNIX 用的 WSGI HTTP 服务器。它既支持 eventlet ，也支持 greenlet 。在这个服务器上运行 Flask 应用是相当简单的:
`gunicorn myproject:app`

Gunicorn 提供了许多命令行选项 —— 见 gunicorn -h 。 例如，用四个 worker 进程（ gunicorn -h ）来运行一个 Flask 应用，绑定到 localhost 的4000 端口（ -b 127.0.0.1:4000 ）:
`gunicorn -w 4 -b 127.0.0.1:4000 myproject:app`

## 代理设置
如果你在一个 HTTP 代理后把你的应用部署到这些服务器中的之一，你需要重写一些标头来让应用正常工作。在 WSGI 环境中两个有问题的值通常是` REMOTE_ADDR` 和 `HTTP_HOST` 。你可以配置你的 httpd 来传递这些标头，或者**在中间件中手动修正**。 Werkzeug 带有一个修正工具来解决常见的配置，但是你可能想要为特定的安装自己写 WSGI 中间件。

这是一个简单的 nginx 配置，它监听 localhost 的 8000 端口，并提供到一个应用的代理，设置了合适的标头:
```
server {
    listen 80;
    server_name _;
    
    access_log  /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    location / {
        proxy_pass         http://127.0.0.1:8000/;
        proxy_redirect     off;

        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```

**如果你的 httpd 不提供这些标头**，最常见的配置引用从 X-Forwarded-Host 设置的主机名和从 X-Forwarded-For 设置的远程地址（**不建议这么做**，因为它盲目地信任一个可能由恶意客户端伪造的标头。）:
```
from werkzeug.contrib.fixers import ProxyFix
app.wsgi_app = ProxyFix(app.wsgi_app)
```

## 常用命令行选项
**-c CONFIG**, --config=CONFIG - Specify a config file in the form $(PATH), file:$(PATH), or python:$(MODULE_NAME)
**-b BIND**, --bind=BIND - Specify a server socket to bind. Server sockets can be any of $(HOST), $(HOST):$(PORT), or unix:$(PATH). An IP is a valid $(HOST)
**-w WORKERS,** --workers=WORKERS - The number of worker processes. 
**-k WORKERCLASS, **--worker-class=WORKERCLASS - The type of worker process to run.. You can set this to $(NAME) where $(NAME) is one of `sync, eventlet, gevent, or tornado, gthread, gaiohttp`. **sync is the default.**
-n APP_NAME, --name=APP_NAME，If setproctitle is installed you can adjust the name of Gunicorn process as they appear in the process system table (which affects tools like `ps and top`).
--chdir
-D, --daemon
-e ENV, --env ENV，Set environment variable (key=value).
-u USER, --user USER
-g GROUP, --group GROUP

## supervisor集成
```
[program:gunicorn]
command=/path/to/gunicorn main:application -c /path/to/gunicorn.conf.py
directory=/path/to/project
user=nobody
autostart=true
autorestart=true
redirect_stderr=true
```
