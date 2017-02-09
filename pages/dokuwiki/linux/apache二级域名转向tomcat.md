title: apache二级域名转向tomcat 

#  apache二级域名转向tomcat 

由于使用的是xampp搭建的。
所以我们编辑/opt/lampp/etc/extra/httpd-vhosts.conf
''
<VirtualHost *:80>
#    ServerAdmin webmaster@dummy-host.example.com
    ServerName apk.xby1993.net
    ProxyPreserveHost On #这行必须加上
    ProxyRequests Off   #这行必须加上
    ProxyPass / http://这里一定要用IP地址:8080/app/   #最后的/必须加上
    ProxyPassReverse / http://这里一定要用IP地址:8080/app/
#    ServerAlias www.dummy-host.example.com
#    ErrorLog "logs/dummy-host.example.com-error_log"
#    CustomLog "logs/dummy-host.example.com-access_log" common
</VirtualHost>
''

<wrap em>注意：
1一定要使用IP地址。
2     ProxyPass / http://这里一定要用IP地址:8080/app/   #最后的/必须加上
    ProxyPassReverse / http://这里一定要用IP地址:8080/app/
这两行的最后的/一定要加上，访问就是apk.xby1993.net，否则访问就是apk.xby1993.net/app
</wrap>

