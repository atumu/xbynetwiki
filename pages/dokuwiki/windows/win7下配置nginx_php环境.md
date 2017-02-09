title: win7下配置nginx_php环境 

#  win7下配置nginx+php环境 
下载nginx,php(php下VC11 x86 Non Thread Safe版本，不然存在老是停止工作问题)
下载![](/data/dokuwiki/windows/runhiddenconsole.zip|) RunHiddenConsole.exe 是一个用来隐藏 DOS 窗口的小程序
然后目录结构如下
![](/data/dokuwiki/windows/pasted/20160405-095044.png)
##  配置php 
找到php.ini-recommended，复制一份，然后将名称修改为：php.ini，然后打开该文件，进行配置。
[php.ini配置](/pages/dokuwiki/php/phplearn0)
##  配置nginx 
```

 server {
        #listen       8000;
        listen       localhost:8000;
        server_name  localhost  alias  localhost.alias;

        location / {
            root   html;
            index  index.html index.htm,index.php;
        }
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }

```
##  启动脚本 
start.bat
```

@echo off
rem start nginx

REM 每个进程处理的最大请求数，或设置为 Windows 环境变量
REM set PHP_FCGI_MAX_REQUESTS=1000

echo Starting PHP FastCGI...
RunHiddenConsole php/php-cgi.exe -b 127.0.0.1:9000 -c php/php.ini

echo Starting nginx...
RunHiddenConsole nginx.exe -p .

```
stop.bat
```

@echo off
rem nginx -s stop
rem pkill -9 nginx
echo Stopping nginx...  
taskkill /F /IM nginx.exe > nul
echo Stopping PHP FastCGI...
taskkill /F /IM php-cgi.exe > nul
exit

```
reload.bat
```

nginx -s reload

```

完整配置附件:链接：http://pan.baidu.com/s/1gfoACdP 密码：q3tx

如遇缺少MSVC*.dll的情况，请参考:http://jingyan.baidu.com/article/9faa7231b1c2b9473d28cb43.html
参考:
http://www.jb51.net/article/23902.htm
http://my.oschina.net/xuqiang/blog/351496