title: apache反向代理tomcat 

#   apache反向代理tomcat 

Apache HTTP server 和 Tomcat server 整合，一般是希望对于用户只公布 Apache HTTP server 的网址，而 Tomcat 的网址则不公布，扮演一个幕后英雄的角色。访问 Tomcat 的 HTTP 请求，通过 Apache 转发给 Tomcat，Tomcat 处理完后，将 HTTP 回应返回给 Apache，然后 Apache  HTTP 回应发回给用户端浏览器。
Apache HTTP server 和 Tomcat server 直接的 HTTP 数据传输，有很多种方法。

方法一，使用 mod_jk。很多网站上介绍到 Apache HTTP server 和 Tomcat server 整合的时候，都是在介绍  mod_jk.so 的使用，这是一种比较老的方法，并且需要额外下载 mod_jk。Apache 和 Tomcat 的默认配置文件都需要改动。

方法二， URL rewrite，也就是对于指定格式的 URL，转发给某个 Tomcat 的网址。这里所说的指定格式，是指 Apache 所使用的正则表达式，通俗地将，是一种类似 * 的一种比较高级通配符。这种方法不需要下在额外的文件，只需要配置 Apache。

方法三，mod_proxy_ajp，仅在 Apache 2.1 及以后的版本中可用，Apache 自带的一个新功能模块。这时 Apache 使用 Apache JServ Protocol 与 Tomcat 通讯。不需要下在额外的文件，需要改动Apache 和 Tomcat 的默认配置文件都需要改动。

方法四，mod_proxy。其实 mod_proxy 既可以做类似于 Wingate 一样的公司局域网共享上网代理，也可以做反向代理(Reverse proxy)。这里使用的是反向代理功能，用户端浏览器不需要把代理服务器改成这里的 Apache 地址。mod_proxy 是 Apache 自带功能，并且配置比较简单。

这篇文章介绍 Apache 反向代理转发 HTTP 请求到 Tomcat 的配置。比较简单实用。

下载 Apache web server  2.2，安装完成后，修改安装目录下的 conf/httpd.conf 文件，将以下两行前的注释字符 # 去掉。

#LoadModule proxy_module modules/mod_proxy.so
#LoadModule proxy_http_module modules/mod_proxy_http.so

在这个配置文件最后，加上
ProxyPass /app1 http://<tomcat_server_address>:port/url1
ProxyPassReverse /app1 http://<tomcat_server_address>:port/url1
比如：
ProxyPass  /bbs http://mydomain.com:8080/bbs     
ProxyPassReverse  /bbs http://mydomain.com:8080/bbs   

ProxyPass  / http://localhost:8080/    
ProxyPassReverse / http://localhost:8080/
 

最后两个的意思是将root转发到tomcat得root，其他context只要app的名字和tomcat下的工程名一直即可！！！
参考：http://blog.163.com/liucy_18/blog/static/53192906201051210296718/