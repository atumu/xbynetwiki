title: nginx反向代理tomcat导致session失效 

#  nginx反向代理(proxy_pass)tomcat导致session失效的问题解决 
Nginx反向代理tomcat，很是方便，但是也有些细节的问题需要注意；今天遇到了这样一个问题，tomcat中路径“host/web1”，nginx中直接“host/”代理，这时候session就无法正常进行了。
登录功能不好用。最后发现是session每次刷新都在变。我又没有做负载均衡，就只有一个反向代理，还导致session丢失，百思不得其解。
```

location / {
            proxy_pass   http://192.168.1.12:11080/v1/;
    
     
            proxy_set_header   Host    $host;
           # proxy_set_header   Remote_Addr    $remote_addr;
            proxy_set_header   X-Real-IP    $remote_addr;
            proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
        }

```
而后检查是**由于cookies path问题导致**，**阅读官方资料中显示proxy_cookie_path**，遂调整 还是要多看官网文档啊。
```

location / {
            proxy_pass   http://192.168.1.12:11080/v1/;
            proxy_cookie_path /v1/ /;
    
            proxy_set_header   Host    $host;
           # proxy_set_header   Remote_Addr    $remote_addr;
            proxy_set_header   X-Real-IP    $remote_addr;
            proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
  		 proxy_set_header   Cookie $http_cookie;
        }

```
2，tomcat应用工程(网站程序配置)获取客户IP方法
` 用 String ip = request.getHeader("X-Real-IP");替代String ip = request.getRemoteAddr(); `

测试一切正常。
一开始以为是程序的问题，但是在本机测试都是ＯＫ，本机与线上的环境只差一个代理。
花了3－4小时，就一段` proxy_cookie_path /v1/ /; `配置就搞定/晕
   