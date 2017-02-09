author:xbynet
title:fabric+supervisor+nginx+gunicorn配置实战
modifyAt:2016-12-12 03:13:47
location:server/fabric+supervisor+nginx+gunicorn配置实战
createAt:2016-12-12 03:13:47

以ubuntu为例：

# 系统初始化步骤
```
#!/bin/bash

#初始化用户
sudo useradd -rm -s /bin/bash demo
sudo adduser demo sudo
sudo passwd demo

sudo apt-get install build-essential python-software-properties software-properties-common -y
sudo apt-get install vim nano -y
sudo apt-get install supervisor -y

sudo add-apt-repository ppa:nginx/stable 
sudo add-apt-repository -y ppa:rwky/redis
sudo apt-get update

sudo apt-get install nginx aria2 axel wget curl -y
sudo apt-get install  redis-server -y

sudo apt-get install build-essential libssl-dev libffi-dev python-dev python3-dev -y
sudo apt-get install python3-pip -y
sudo apt-get install  convmv libevent-dev libssl-dev libffi-dev libsasl2-dev libpq-dev  libxml2-dev libxslt1-dev libldap2-dev  -y

vim ~/.pip/pip.conf
#[global]
#index-url = https://pypi.douban.com/simple #豆瓣源，可以换成其他的源
#disable-pip-version-check = true          #取消pip版本检查，排除每次都报最新的pip
#timeout = 120

sudo pip3 install virtualenv
mkdir venv && cd venv 
virtualenv mdwiki
source mdwiki/bin/activate

pip3 install gunicorn

#具体配置文件后续说明
sudo vim  mdwiki/gunicorn.conf.py
sudo vim /etc/supervisor/conf.d/default.conf
sudo vim /etc/nginx/conf.d/default.conf

#添加开机自启
sudo vim /etc/rc.local
#/usr/bin/supervisord -c /etc/supervisor/supervisord.conf

sudo update-rc.d nginx disable
```
# supervisor配置

celery+virtualenv+supervisor的情形，其实只要指定celery程序为virtaulenv下面的那个即可，例如/home/xby/venv/mdwiki/bin/celery

**如果在gunicorn下出现'ascii' codec can't encode...报错**，那么请在supervisor加入environment如下

    environment=LANG="en_US.utf8", LC_ALL="en_US.UTF-8", LC_LANG="en_US.UTF-8"

```
[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
stopsignal=QUIT
priority=1
;user=www-data
[program:celeryworker]
directory=/opt/www/mdwiki
command=/path/to/celery worker -A app.util.tasks.celery_app  -f celery.worker.log -l info 
priority=5
autostart=true
autorestart=true
startsecs=10
user=www-data
[program:celerybeat]
directory=/opt/www/mdwiki
command=/path/to/celery beat -A app.util.tasks.celery_app  -f celery.beat.log -l info 
priority=6
autostart=true
autorestart=true
startsecs=10
user=www-data

[program:mdwiki]
;environment=SECRET_KEY=value,aliyun_api_key=value,aliyun_secret_key=value,MAIL_PASSWORD=value
;command=/usr/bin/gunicorn -n mdwiki -w 4 -b 127.0.0.1:4000 -k gevent app:app
environment=LANG="en_US.utf8", LC_ALL="en_US.UTF-8", LC_LANG="en_US.UTF-8"
command=/path/to/gunicorn app:app -c /path/to/gunicorn.conf.py
directory=/opt/www/mdwiki
;user=www-data
autostart=true
autorestart=true
priority=10
redirect_stderr = true  
stdout_logfile_maxbytes = 20MB  
stdout_logfile_backups = 20 
stdout_logfile = /var/log/mdwiki/mdwiki.log

; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```
# nginx配置

```
server {
    listen              80;
    listen              443 ssl;
    server_name         demo.com;
    ssl_certificate     /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;
    server_tokens off;
    charset utf-8;
    client_max_body_size 20M;
    set $projdir "/opt/www/mdwiki";
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect off;
        proxy_pass http://127.0.0.1:4000;
    }
    location ~ ^/[^static].*\.(jpg|png|gif|bmp|zip|docx)$ {
        expires 30d;
        root $projdir;
    }
    location ~ ^/static.*\.(js|css|png|jpg|gif|bmp|map|ico|eot|svg|ttf|woff)$ {
        expires 30d;
        root $projdir/app;
    }

}

```
# gunicorn.conf.py配置

```
#!/bin/bash

import multiprocessing

bind = "127.0.0.1:4000"
workers = multiprocessing.cpu_count() * 2 + 1
worker_class='gevent'
proc_name = "mdwiki"
user = "www-data"
chdir='/opt/www/mdwiki'
#daemon=False
#group = "nginx"
loglevel = "info"
errorlog = chdir+"/log/gunicorn/error.log"
accesslog= chdir+"/log/gunicorn/access.log"
raw_env = [
   r'MAIL_PASSWORD=pass',
   r'SECRET_KEY=\xe6'
]
#ssl
#keyfile=
#certfile=
#ca_certs=

```

# fabric远程发布

    fab -f fabfile.py deploy

fabfile.py文件
```
from fabric.api import *
import os,sys
import tarfile
from contextlib import contextmanager
from fabric.contrib.files import exists

#$ fab -f fabfile.py -H localhost,remote host_type
def host_type():
    run('uname -s')

env.user= os.environ.get('USER','')
env.hosts= os.environ.get('HOST','').split(',')
env.password= os.environ.get('PASSWORD','')
env.sudo_password= os.environ.get('PASSWORD','')

active='source /home/xby/venv/mdwiki/bin/activate'

srcPath=r'C:\Users\taojw\Desktop\pywork\mdwiki'
distPath=r'C:\Users\taojw\Desktop\pywork\mdwiki\dist'
distFile=distPath+os.sep+'mdwiki.tar.gz'

#用于处理virtualenv环境,将其包装成with上下文
@contextmanager
def virtualenv():
    with prefix(active):
        yield


if not os.path.exists(distPath):
    os.mkdir(distPath)

#本地打包分发文件
def pack():
    def ecludefiles(path):
        for name in ['venv','node_modules','websrc','__pycache__','.git','.idea','dist']:
            if path.find(os.sep+name)>0:
                return True
        return False

    if os.path.exists(distFile):
        os.remove(distFile)
    #压缩成tar.gz格式
    with tarfile.open(distFile,'w:gz') as f:
        f.add(srcPath,arcname='mdwiki',exclude=ecludefiles)

#部署
def deploy():
    #local pack dist file
    pack()
    
    remote_tmp='/tmp/mdwiki.tar.gz'

    localsize=os.path.getsize(distFile)
    remotesize=0
    #check if should upload again if there is a same file
    if exists(remote_tmp):
        remotesize=int(run("stat -c '%s' {0}".format(remote_tmp)))
        print(str(localsize)+":"+str(remotesize))
    if localsize!=remotesize:
        sudo('rm -f %s' % remote_tmp)
        # upload dist file
        put(distFile,remote_tmp)
    if not exists('/opt/www'):
        sudo('mkdir /opt/www')
        sudo('chown www-data:www-data /opt/www')

    #stop app and bak now
    with settings(warn_only=True):
        #delete previous bak
        sudo('rm -rf /opt/www/mdwiki_bak')
        sudo('supervisorctl stop all')
        if exists('/opt/www/mdwiki'):
            sudo('mv /opt/www/mdwiki /opt/www/mdwiki_bak')

    sudo('tar -zxvf /tmp/mdwiki.tar.gz -C /opt/www/')

    with cd('/opt/www/'):
        #replace data dir
        if exists('mdwiki_bak/data'):
            sudo('rm -rf mdwiki/data')
            sudo('cp -R mdwiki_bak/data mdwiki/')
        if exists('mdwiki_bak/app.db'):
            sudo('cp mdwiki_bak/app.db mdwiki/')

        sudo('chown -R www-data:www-data mdwiki')

        with virtualenv():
            run('pip3 install -r  mdwiki/requirements.txt')

    sudo('rm -f %s' % remote_tmp)
    sudo('supervisorctl start all')

#in your local shell run 'fab deploy' command
```