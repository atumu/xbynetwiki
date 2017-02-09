title: nginx反向代理保留ip 

#  nginx反向代理获取Nginx服务器IP与客户端真实IP 
nginx反向代理部分配置：
```

location / {
            proxy_pass   http://192.168.1.12:11080/v1/;
            proxy_cookie_path /v1/ /;
    
            proxy_set_header   Host    $host:$server_port;
           # proxy_set_header   Remote_Addr    $remote_addr;
            proxy_set_header   X-Real-IP    $remote_addr;
		 proxy_set_header request_uri $request_uri;
            proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
  		 proxy_set_header   Cookie $http_cookie;
        }

```
tomcat下获取IP：
```

	@Override
	public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
			throws IOException, ServletException {
		// TODO Auto-generated method stub
		String nginxIp=req.getRemoteAddr();
		System.out.println("nginx代理服务器IP:"+nginxIp);
		String realClientIp=((HttpServletRequest)req).getHeader("X-Real-IP");
		System.out.println("客户端IP:"+realClientIp);
		
	}

```

更多参考:http://www.07net01.com/zhishi/541866.html
http://stackoverflow.com/questions/5834025/how-to-preserve-request-url-with-nginx-proxy-pass


##  多层Nginx代理情况下保留用户IP 
情景：
用户->nginx1->nginx2->tomcat
我们需要保证经过两层nginx之后的tomcat还能够获取到用户的ip
思路：
1、用户访问nginx1时，nginx1通过配置:proxy_set_header X-Real-IP $remote_addr;保留用户IP
2、当nginx1代理访问nginx2时，我们直接代理到tomcat
3、nginx2代理访问tomcat时，在java代码中，我们可以通过String clientIp=req.getHeader("X-Real-IP");来获取真实client IP
具体配置如下:
第一个nginx配置:
```

http{
    ......
    underscores_in_headers on;
    upstream myServer {   
        server 10.20.20.181:80;
    }	
    server {
        listen       80;
        server_name  localhost;

        location  / {
	    #proxy_set_header Host $host:$server_port;
	    proxy_set_header X-Real-IP $remote_addr;
	    #proxy_set_header request_uri $request_uri;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_pass http://myServer;
        }
    }
}

```
##  Nginx读取自定义header 
在参考了资料:
http://stackoverflow.com/questions/8393772/how-to-get-non-standard-http-headers-on-nginx
http://nginx.org/en/docs/http/ngx_http_core_module.html#underscores_in_headers
http://serverfault.com/questions/297225/nginx-passing-back-custom-header
https://easyengine.io/tutorials/nginx/forwarding-visitors-real-ip/
http://www.ttlsa.com/nginx/nginx-proxy_set_header/
后得到如下：
1、nginx是支持读取非nginx标准的用户自定义header的，但是需要在http或者server下开启header的下划线支持:
  * underscores_in_headers on;
2、比如我们自定义header为X-Real-IP,通过第二个nginx获取该header时需要这样:
  * $http_x_real_ip; (一律采用小写，而且前面多了个http_)

3、如果需要把自定义header传递到下一个nginx：
  * 如果是在nginx中自定义采用proxy_set_header X_CUSTOM_HEADER $http_host;
  * 如果是在用户请求时自定义的header，例如curl --head -H "X_CUSTOM_HEADER: foo" http://domain.com/api/test，则需要通过` proxy_pass_header X_CUSTOM_HEADER `来传递


示例：
```

http{
    upstream myServer {   
        server 127.0.0.1:8082;
    }
    underscores_in_headers on;
    server {
        listen       80;
        server_name  localhost;

        location  / {
  	    proxy_set_header Some-Thing $http_x_custom_header;;
  	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  	    proxy_pass http://myServer;
        }
    } 
}

```