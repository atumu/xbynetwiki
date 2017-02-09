title: mysql主从读写分离配置 

#  Mysql主从读写分离配置 
参考：http://dev.mysql.com/doc/refman/5.7/en/replication-howto-repuser.html
http://www.111cn.net/sys/CentOS/65581.htm
http://www.cnblogs.com/alvin_xp/p/4162249.html
http://369369.blog.51cto.com/319630/790921
http://jingyan.baidu.com/album/90808022ae0b8ffd91c80f8b.html

**安装mysql时候需要注意安装的方式：**
采用ubuntu官方源安装后，主从分离成功。
sudo apt-get install mysql-server
如果采用从第三方安装，如本人试验过从搜狐mysql下载一些deb安装会**导致失败**，甚至mysql只要开启主从配置就启动失败。
 原因猜测：由于我没有将所有的mysql相关的deb包都安装，所以可能是由于缺少了主从功能相关的依赖而导致。可以试试就其余相关deb包安装之后能不能成功。
##  主mysql(master mysql:配置 
# master slaver config
```

[mysqld]
server-id=111
log-bin=mysql-bin
#replicate-do-db=test
#sync-binlog=1 #开启同步到磁盘
#binlog-ignore-db=mysql
#binlog-ignore-db=information_schema

```

```

mysql> CREATE USER 'repl'@'%' IDENTIFIED BY 'mysql';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

```
```

service mysql restart

```
```

mysql> show master status;
+------------------+----------+--------------+--------------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB         | Executed_Gtid_Set |
+------------------+----------+--------------+--------------------------+-------------------+
| mysql-bin.000002 |      154 |              | mysql,information_schema |                   |
+------------------+----------+--------------+--------------------------+-------------------+
1 row in set (0.00 sec)

```

-------------------------------------------------------------------------------------------------------------------------------
##  从mysql: slave mysql配置 
```

[mysqld]
server-id=112
relay-log=slave-relay-bin

```
```

mysql> change master to master_host='mysqlmaster',master_port=3306,master_user='repl',master_password='mysql',master_log_file='mysql-bin.000002',master_log_pos=154;

Mysql>start slave;    //启动从服务器复制功能

#master_log_pos=154这个154不是随便填写的，它与master_log_file='mysql-bin.000001'一样，是根据上面再master mysql运行show master status得到的。

#!/bin/bash
docker run -d -p 3306:3306 --name mysqlmaster -v /opt/mysqldata:/opt/mysqldata mysqlmaster /usr/bin/supervisord
docker run -d -p 3307:3306 --link mysqlmaster:mysqlmaster -v /opt/mysqldata_slave:/opt/mysqldata mysqlslave1 /usr/bin/supervisord

```
