title: docker2 

#  Docker+supervisor+tomcat+nginx+php-fpm配置 
注意点:
1、` 使用docker启动，supervisor不能在后台运行 `，需要使用**/usr/bin/supervisord -n -c/etc/supervisor/supervisord.conf**（其中` -n为--nodaemon `）或者/usr/bin/supervisord并配置
```

[supervisord]
nodaemon=true

``` 
2、` supervisor管理的进程不能以后台程序运行 `。比如ph5-fpm启动不能command=/usr/sbin/php5-fpm，而应该使用command=/usr/sbin/php5-fpm --nodaemonize 参考http://stackoverflow.com/questions/32965149/supervisord-php5-fpm-exited-too-quickly
3、` supervisor管理tomcat时，需要使用catalina.sh而非startup.sh `。原因就是不能以非后台运行。所以command=/opt/tomcat7/bin/catalina run
4、**supervisor管理nginx时，由于不能管理后台进程，所以需要配置` /etc/nginx/nginx.conf `，在nginx.conf的顶端配置` daemon off; `** 参考http://serverfault.com/questions/647357/running-and-monitoring-nginx-with-supervisord 
```

user www-data;
worker_processes auto;
pid /run/nginx.pid;

daemon off;
...

```
不过也可以通过` /usr/sbin/nginx -g "daemon off;" `以前台进程方式进行
具体配置如下:
```

[inet_http_server]
port=localhost:9000
username=xxxx
password=111

[supervisord]
nodaemon=true
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

[program:chmod]
command=/bin/bash -c "chown mysql:mysql -R /opt/website/mysqldata && chown www-data:www-data -R /opt/website/www"
[program:sshd]
command=/usr/sbin/sshd -D
[program:nginx]
command=/usr/sbin/nginx
stopsignal=QUIT
[program:php-fpm]
command=/usr/sbin/php5-fpm  --nodaemonize
stopsignal=QUIT
[program:tomcat]
command=/opt/website/tomcat7/bin/catalina.sh run
startsecs=10 
stopsignal=QUIT ;指定退出信号 
user=root

```
![](/data/dokuwiki/docker/pasted/20160428-161840.png)