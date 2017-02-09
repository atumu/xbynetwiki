title: nginx配置01 

#  nginx配置之proxy_pass说明 
在nginx中配置proxy_pass分为四种情况:
例如对http://127.0.0.1/proxy/test.html进行访问
配置一：
```

location /proxy/ {
	proxy_pass http://127.0.0.1:8080/;
}

```
这样会被代理为http://127.0.0.1:8080/test.html

配置二:
```

location /proxy/{
	proxy_pass http://127.0.0.1:8080;
}

```
会被代理为http://127.0.0.1:8080/proxy/test.html

上面两种方式的**区别**是proxy_pass的url路径部分最后` 是否添加了/ `,` 如果像配置一那样添加了/,那么相当于是绝对根路径，nginx不会把location中匹配的路径部分(即/proxy/)代理走.而如果最后没有/，如配置二那样，则nginx会把匹配的路径部分(/proxy/)也给代理走 `

配置三
```

location /proxy/ {
	proxy_pass http://127.0.0.1:8080/app/;
}

```
会被代理到http://127.0.0.1:8080/app/test.html

配置四
```

location /proxy/{
	proxy_pass http://127.0.0.1:8080/app;
}

```
会被代理到http://127.0.0.1:8080/apptest.html

**其实配置一、配置三、配置四是类似的(都不包含匹配的路径部分(即/proxy/))。请仔细注意配置二和配置四的区别。**


**proxy_pass保留客户端信息**：(后端tomcat中用 String ip = request.` getHeader("X-Real-IP") `;替代String ip = request.getRemoteAddr();)
```

location /proxy/{
  	    
	    #proxy_cookie_path / /;
	    proxy_set_header   Host    $host;
            # proxy_set_header   Remote_Addr    $remote_addr;
            proxy_set_header   X-Real-IP    $remote_addr;
            proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
  	    proxy_set_header   Cookie $http_cookie;
	
	    proxy_pass http://127.0.0.1:8080;
}

```

参考http://blog.chinaunix.net/uid-20577907-id-3549417.html