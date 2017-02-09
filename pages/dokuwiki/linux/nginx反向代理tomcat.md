title: nginx反向代理tomcat 

#  nginx反向代理tomcat 
nginx支持十万级的并发连接量，**而tomcat默认请求线程池只有200**.所以在高并发情况下需要采用nignx+tomcat集群

关键：         proxy_pass http://127.0.0.1:8080/app/;  #此处需要注意，**` 最后的/必须加上 `**，否则会出问题（404）。
` proxy_cookie_path /app/ /; `   #此处需注意，解决反向代理导致的session丢失的问题. 否则登录这一块废掉。
```

server {
    listen       80;
    server_name  app.baidu.net;
    #root /opt/website/tomcat7/webapps/app;
    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
    #    root   /usr/share/nginx/html;
    #    index  index.php index.html index.htm index.php;
         proxy_pass http://127.0.0.1:8080/app/;  #此处需要注意，最后的/必须加上，否则会出问题（404）。
         proxy_set_header Host $host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	 proxy_cookie_path /app/ /;   #此处需注意，解决反向代理导致的session丢失的问题
         proxy_redirect off;
         client_max_body_size 100m;
         client_body_buffer_size 128k;
         proxy_connect_timeout   300;
         proxy_send_timeout      300;
         proxy_read_timeout      300;
         proxy_buffer_size 4k;
         proxy_buffers 4 32k;
         proxy_busy_buffers_size 64k;
         proxy_temp_file_write_size 64k;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #

    location / {
    #    root   /usr/share/nginx/html;
    #    index  index.php index.html index.htm index.php;
         proxy_pass http://127.0.0.1:8080/app/;
         proxy_set_header Host $host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

         proxy_redirect off;
         client_max_body_size 100m;
         client_body_buffer_size 128k;
         proxy_connect_timeout   300;
         proxy_send_timeout      300;
         proxy_read_timeout      300;
         proxy_buffer_size 4k;
         proxy_buffers 4 32k;
         proxy_busy_buffers_size 64k;
         proxy_temp_file_write_size 64k;
    }
  ...
}

```
参考：
http://nginx.org/en/docs/http/ngx_http_proxy_module.html