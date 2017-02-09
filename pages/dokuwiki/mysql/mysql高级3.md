title: mysql高级3 

#  MySQL高级之mysqld_multi主从在一台主机配置 
##  安装MySQL 
1、安装MySQL过程略。
以下过程参考:http://sharadchhetri.com/2013/12/02/create-multiple-mysql-instance-centos-6-4-red-hat-6-4/
##  多实例配置 
2、To create multiple MySQL instance,follow the given below steps
Step (1): Create a new MySQL data directory.
```

mkdir -p /var/lib/mysql2

```
Step 2: Now give permission and ownership .Here we are taking ownership and permission reference from original /var/lib/mysql data directory.
```

chmod --reference /var/lib/mysql /var/lib/mysql2
chown --reference /var/lib/mysql /var/lib/mysql2

```
Step 3(**该步骤在使用mysqld_multi的情况下可以略过)**: Now create a new my2.cnf file and paste the below given contents. We will run the new mysql instance in port no. 3337.
Copy the my.cnf file and make it blank 
```

cp -p /etc/my.cnf /etc/my2.cnf
vi /etc/my2.cnf
[mysqld]
datadir=/var/lib/mysql2
socket=/var/lib/mysql/mysql2.sock
port=3337

[mysqld_safe]
log-error=/var/log/mysqld2.log
pid-file=/var/run/mysqld/mysqld2.pid

```
Step 4: Now install the database in the new mysql data directory
```

mysql_install_db --user=mysql --datadir=/var/lib/mysql2

```
To start new mysql instance,use below given command （**由于我们使用的是mysqld_multi，所以这种方式只是看看，具体不会这么启动）**
```

mysqld_safe --defaults-file=/etc/my2.cnf &

To connect new mysql instance
 mysql -S  /var/lib/mysql/mysql2.sock -u username -p 或 mysql -h 127.0.0.1 -P 3307 -u root -p

To stop new mysql instance use below given command
mysqladmin -S /var/lib/mysql/mysql2.sock shutdown -p

```
##  使用mysqld_multi管理多个实例 
这部分参考：https://www.percona.com/blog/2014/08/26/mysqld_multi-how-to-run-multiple-instances-of-mysql/
配置样例可以通过mysqld_multi --example查看
对每个需要mysqld_multi管理的实例都配置一个同样的用户:
```

mysql> GRANT SHUTDOWN ON *.* TO 'multiadmin'@'localhost' IDENTIFIED BY '1234';
mysql> FLUSH PRIVILEGES;

```
上面对mysql-master,mysql-slave都得执行一遍。这个用户用来通过mysqld_multi来关闭mysqld进程。否则无法关闭。
```

vim /etc/my_multi.cnf

[mysqld_multi]
mysqld     = /usr/bin/mysqld_safe
mysqladmin = /usr/bin/mysqladmin
user       = multiadmin
pass   = 1234 #这里需要注意的是，网上流传的 password=1234是错误的写法

[mysqld1]
socket     = /var/lib/mysql/mysql.sock
port       = 3306
pid-file   = /var/run/mysqld/mysqld.pid
datadir    = /var/lib/mysql
user       = mysql
server-id  = 100
log-bin    = mysql-bin

[mysqld2]
socket     = /var/lib/mysql/mysql2.sock
port       = 3307
pid-file   = /var/run/mysqld/mysqld2.pid
datadir    = /var/lib/mysql2
user       = mysql
server-id  = 300
#skip-log-bin
relay-log-index = slave-relay-bin.index
relay-log = slave-relay-bin
#read_only = 1

[mysql]
default-character-set =utf8


```
启动查看关闭等命令：
```

mysqld_multi --defaults-extra-file=/etc/my_multi.cnf report 查看运行状态
mysqld_multi --defaults-extra-file=/etc/my_multi.cnf start
mysqld_multi --defaults-extra-file=/etc/my_multi.cnf stop
mysqld_multi --defaults-extra-file=/etc/my_multi.cnf start 2 （这个编号对应上面配置的mysqld2）
mysqld_multi --defaults-extra-file=/etc/my_multi.cnf stop 1


```

##  主从配置 
参考[[database:mysql主从读写分离配置]]

##  可能遇到的问题 
主从连接正确，但是从库不拷贝，或者拷贝执行SQL报错（通过show slave status\G看到的）。遇到这种情形，直接先把主库的完整的sql导出拷贝到从库执行一遍。然后再重新配置一遍主从即可。具体参考：
http://stackoverflow.com/questions/2366018/how-to-re-sync-the-mysql-db-if-master-and-slave-have-different-database-incase-o
http://www.111cn.net/database/mysql/86336.htm
参考：
http://sharadchhetri.com/2013/12/02/create-multiple-mysql-instance-centos-6-4-red-hat-6-4/
http://www.ducea.com/2009/01/19/running-multiple-instances-of-mysql-on-the-same-machine/
https://www.percona.com/blog/2014/08/26/mysqld_multi-how-to-run-multiple-instances-of-mysql/
http://www.cnblogs.com/super-d2/p/3852192.html
http://blog.163.com/kkfngu@126/blog/static/101251907201001472455735/