title: 更改ubuntu_mysql_data目录位置 

#  更改ubuntu mysql data目录位置 
1.设置新的存放路径
mkdir -p /data/mysql
2.复制原有数据
cp -R /var/lib/mysql/* /data/mysql
3.修改权限
chown -R mysql:mysql /data/mysql
4.修改配置文件
vim /etc/mysql/my.cnf
datadir = /data/mysql
5.修改启动文件
```

vim /etc/apparmor.d/usr.sbin.mysqld
#把
/var/lib/mysql r,
/var/lib/mysql/** rwk,
#改成
/data/mysql r,
/data/mysql/** rwk,

```
6.重启服务
重启apparmor
/etc/init.d/apparmor restart
/etc/init.d/mysql restart