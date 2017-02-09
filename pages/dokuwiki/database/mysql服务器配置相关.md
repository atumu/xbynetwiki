title: mysql服务器配置相关 

#  mysql服务器配置相关 
sudo apt-get install mysql-server mysql-client

远程访问
1、修改localhost
更改 "mysql" 数据库里的 "user" 表里的 "host" 项，从"localhost"改成"%" 
```

mysql>use mysql; 
mysql>update user set host = '%' where user = 'root'; 
mysql>select host, user from user;
mysql>FLUSH PRIVILEGES;

```
##  mysql5.5远程登录失败 
修改mysql的配置文件/etc/mysql/my.conf，将[mysqld]下面的` bind-address `后面增加远程访问IP地址或者**禁掉这句话**就可以让远程机登陆访问了。
**记得要重启mysql服务**哦
service mysql restart
##  phpmyadmin无法登录 
修改phpmyadmin配置文件config.inc.php
将$cfg['Servers'][$i]['host'] = 'localhost'; 
改为$cfg['Servers'][$i]['host'] = '127.0.0.1'; 