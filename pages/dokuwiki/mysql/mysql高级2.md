title: mysql高级2 

#  mysql高级之主从(master-slave)复制配置 
##  主服务器配置 
1、要设置主从服务器架构，**必须确认主服务器上启用了二进制日志记录。**，然后分配server-id。编辑my.ini或my.cnf文件。
```

[mysqld]
log-bin
server-id=1

```
log-bin开启二进制日志记录
server-id=1给主服务器分配一个唯一的ID

2、在主服务器上创建一个用户，该用户将用于从服务器连接主服务器
```

grant replication slave 
on *.*
to 'rep_slave'@'%' identified by '123';

```

3、获取当前时间数据库的一个快照
```

flush tables with read_lock;
show master_status;
unlock tables;

```
记录结果，类似如下:请记住File和Position的值。我们需要这些信息来配置从服务器
```

mysql> show master status;
+------------------+----------+--------------+--------------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB         | Executed_Gtid_Set |
+------------------+----------+--------------+--------------------------+-------------------+
| mysql-bin.000002 |      154 |              | mysql,information_schema |                   |
+------------------+----------+--------------+--------------------------+-------------------+
1 row in set (0.00 sec)

```
##  配置从服务器 
在从服务器上运行如下所示查询：
```

mysql> change master to master_host='your_server',master_port=3306,master_user='rep_slave',master_password='123',master_log_file='mysql-bin.000002',master_log_pos=154;
Mysql>start slave;    //启动从服务器复制功能
#master_log_pos=154这个154不是随便填写的，它与master_log_file='mysql-bin.000002'一样，是根据上面再master mysql运行show master status得到的。

```
如果没有获取快照，可以执行如下语句:
```

load data from master;

```