title: nginx解决post静态文件 

##  nginx解决post静态文件405 not allowed问题 
关键一句:error_page 405 =200 $uri; #解决post请求静态文件如json文件时导致的405 not allowed问题。注意=号的位置。
```

location ~ \.(html|json)$ {  #处理html与json文件访问
            if ($uri !~ ^/static) {    #如果不是以static开头，
                root E:/program/tomcat7/webapps;#root定位到发布到的静态站点。
                rewrite ^(.*)$ /$args$uri; #对于类似url ,/qq22/index.html?qq11重写为/qq11/qq22/index.html，以便于预览访问。
		error_page 405 =200 $uri; #解决post请求静态文件如json文件时导致的405 not allowed问题。注意=号的位置。由于前面执行过rewrite，所以这时候的$uri就相当于rewrite之前的/$args$uri.
                break; #禁止后续规则匹配
                
            }
		#如果以static开头，定位root到tomcat apps引用下
            root E:/workspace/eclipse/.metadata/.plugins/org.eclipse.wst.server.core/tmp1/wtpwebapps/cms-manager;
        }

```

参考:http://www.linuxidc.com/Linux/2012-07/66760.htm
http://www.cnblogs.com/mingaixin/p/4285329.html
http://jingyan.baidu.com/article/a17d528520c7938098c8f280.html