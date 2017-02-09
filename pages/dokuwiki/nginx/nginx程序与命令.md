title: nginx程序与命令 

#  nginx程序与命令 
启动，停止，重载配置：
```

nginx -s signal

```
Where signal may be one of the following:
  * stop — fast shutdown
  * quit — graceful shutdown
  * reload — reloading the configuration file
  * reopen — reopening the log files
命令行参数：
-v — print nginx version.
-V — print nginx version, compiler version, and configure parameters.
-h ——help
-c file ——指定配置文件
-g directives — set global configuration directives, for example,nginx -g "pid /var/run/nginx.pid; worker_processes `sysctl -n hw.ncpu`;"
-p prefix — set nginx path prefix, i.e. a directory that will keep server files (default value is /usr/local/nginx).
-t 在不启动Nginx的情况下，测试配置信息是否有错误
-q 配合-t参数使用，在测试配置阶段可以不把error级别以下的信息输出到屏幕。/usr/local/nginx/sbin/nginx -t -q

----

**日志文件回滚**
使用-s reopen参数可以重新打开日志文件，这样可以先把当前日志文件改名或转移到其他目录中进行备份，再重新打开时就会生成新的日志文件。这个功能使得日志文件不至于过大。例如：
```

/usr/local/nginx/sbin/nginx -s reopen

```

----

快速地停止服务
```

/usr/local/nginx/sbin/nginx -s stop

```
实际上，如果通过kill命令直接向nginx master进程发送TERM或者INT信号，效果是一样的。例如，先通过ps命令来查看nginx master的进程ID：
```

root > ps -ef | grep nginx

```
root     10800     1  0 02:27 ?        00:00:00 nginx: master process ./nginx
root     10801 10800  0 02:27 ?        00:00:00 nginx: worker process
接下来直接通过kill命令来发送信号：
kill -s SIGTERM 10800
“优雅”地停止服务
如果希望Nginx服务可以正常地处理完当前所有请求再停止服务，那么可以使用-s quit参数来停止服务。例如：
```

/usr/local/nginx/sbin/nginx -s quit

```

参考：
http://nginx.org/en/docs/switches.html
http://book.2cto.com/201304/19621.html
http://nginx.org/en/docs/beginners_guide.html