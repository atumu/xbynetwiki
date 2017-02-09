title: mysql高级1 

#  mysql高级管理 
连接数据库:
mysql -D dbname -h hostname -u username -p
选择数据库:
user dbname

##  查看关于数据库的更多信息 
1、使用show获取信息
```

show databases
show tables
show tables from mydb
show columns from mydb.mytable

show grants for mydb
show grants for 'liub'@'localhost' #注意这里是for而不是from

show status 
show table status from DBname

show index from mytable

show create table tablename
show create database dbname
show create procedure myproc;
show create function myfunc;

...

```
2、使用DESCRIBE获取关于列的信息
describe table [column];

3、用explain理解查询操作的工作过程
explain table;
explain select * from mytable


##  用户与授权 
http://blog.csdn.net/bengda/article/details/7853831
http://www.cnblogs.com/Richardzhu/p/3318595.html
```

列级权限
grant select(id,username) on g_user to 'liub'@'localhost';
表级权限
grant select on g_user to 'liub'@'localhost';
 库级权限
grant select on test.* to 'liub'@'localhost';
flush privileges;#刷新，也可以在操作系统命令行中使用mysqladmin flush-privileges或者mysqladmin-reload
show grants for 'liub'@'localhost';

```
MySQL 的权限系统在实现上比较简单，相关权限信息主要存储在几个被称为grant tables 的系统表中，即： mysql.User，mysql.db，mysql.Host，mysql.table_priv 和mysql.column_priv。
**我们每次手工修改了权限相关的表之后，都需要执行FLUSH PRIVILEGES命令重新加载MySQL的权限信息。**


**一, 创建用户:**
命令:CREATE USER 'username'@'host' IDENTIFIED BY 'password';
说明:username - 你将创建的用户名, host - 指定该用户在哪个主机上可以登陆,如果是本地用户可用localhost, 如果想让该用户可以从任意远程主机登陆,可以使用通配符%. password - 该用户的登陆密码,密码可以为空,如果为空则该用户可以不需要密码登陆服务器.
```

例子:    CREATE USER 'dog'@'localhost' IDENTIFIED BY '123456';
               CREATE USER 'pig'@'192.168.1.101' IDENDIFIED BY '123456';
               CREATE USER 'pig'@'%' IDENTIFIED BY '123456';
               CREATE USER 'pig'@'%' IDENTIFIED BY '';
               CREATE USER 'pig'@'%';

```

**二,授权GRANT:**
命令:GRANT privileges ON databasename.tablename TO 'username'@'host'
说明: privileges - 用户的操作权限,如SELECT , INSERT , UPDATE 等.**如果要授予所的权限则使用ALL**.;databasename - 数据库名,tablename-表名,如果要授予该用户对所有数据库和表的相应操作权限则可用*表示, **如*.***.
```

      例子: GRANT SELECT, INSERT ON test.user TO 'pig'@'%';
               GRANT ALL  ON *.* TO 'pig'@'%';
               grant all privileges on *.* to jack@'localhost' identified by "jack" with grant option;

```
注意:用以上命令授权的用户不能给其它用户授权,**如果想让该用户可以给其他用户授权,用以下命令:**
```

GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;

```
**其中 privileges 代表权限，允许多个.权限分为用户权限和管理员权限,**如下：
1、权限分布(就是针对表可以设置什么权限，针对列可以设置什么权限等等)
  * 全局
  * 数据库
  * 表权限'Select', 'Insert', 'Update', 'Delete', 'Create', 'Drop', 'Grant', 'References', 'Index', 'Alter'
  * 列权限'Select', 'Insert', 'Update', 'References'
  * 过程权限'Execute', 'Alter Routine', 'Grant'
2、用户权限
![](/data/dokuwiki/mysql/pasted/20160409-130413.png)
3、管理员权限：
![](/data/dokuwiki/mysql/pasted/20160409-130434.png)

**GRANT命令使用说明：**
先来看一个例子，创建一个只允许从本地登录的超级用户jack，并允许将权限赋予别的用户，密码为：jack.
```

mysql> grant all privileges on *.* to jack@'localhost' identified by "jack" with grant option;
Query OK, 0 rows affected (0.01 sec)

```
GRANT命令说明：
  * ALL PRIVILEGES 是表示所有权限，你也可以使用select、update等权限。
  * ON 用来指定权限针对哪些库和表。
  * *.* 中前面的*号用来指定数据库名，后面的*号用来指定表名。
  * TO 表示将权限赋予某个用户。
  * jack@'localhost' 表示jack用户，@后面接限制的主机，可以是IP、IP段、域名以及%，%表示任何地方。注意：这里%有的版本不包括本地，以前碰到过给某个用户设置了%允许任何地方登录，但是在本地登录不了，这个和版本有关系，遇到这个问题再加一个localhost的用户就可以了。
  * IDENTIFIED BY 指定用户的登录密码。
  * WITH GRANT OPTION 这个选项表示该用户可以将自己拥有的权限授权给别人。注意：经常有人在创建操作用户的时候不指定WITH GRANT OPTION选项导致后来该用户不能使用GRANT命令创建用户或者给其它用户授权。
备注：**可以使用GRANT重复给用户添加权限，权限叠加**，比如你先给用户添加一个select权限，然后又给用户添加一个insert权限，那么该用户就同时拥有了select和insert权限。

**2、刷新权限**
使用这个命令使权限生效，尤其是你对那些权限表user、db、host等做了update或者delete更新的时候。以前遇到过使用grant后权限没有更新的情况，只要对权限做了更改就使用FLUSH PRIVILEGES命令来刷新权限。
```

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

```
查看当前用户的权限：
```

mysql> show grants;

```
查看某个用户的权限：
```

mysql> show grants for 'jack'@'%';

```

**三.设置与更改用户密码**
```

     对账户重命名mysql> rename user 'jack'@'%' to 'jim'@'%';
     命令:SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');如果是当前登陆用户用SET PASSWORD = PASSWORD("newpassword");
     例子: SET PASSWORD FOR 'pig'@'%' = PASSWORD("123456");
     删除用户:drop user 'jack'@'localhost';

```
` **在丢失root密码的时候** `
```

[root@rhel5 ~]# mysqld_safe --skip-grant-tables &
[root@rhel5 ~]# mysql -u root
mysql> use mysql
mysql> update user set password = PASSWORD('123456') where user = 'root';
mysql> flush privileges;

```
**mysql5.7中重置密码的变化：**
```

mysql> update user set authentication_string = PASSWORD('123456') where user = 'root';

```


**四.撤销用户权限**
命令: REVOKE privilege ON databasename.tablename FROM 'username'@'host';
说明: privilege, databasename, tablename - 同授权部分.
```

例子: REVOKE SELECT ON *.* FROM 'pig'@'%';

```
注意: 假如你在给用户'pig'@'%'授权的时候是这样的(或类似的):GRANT SELECT ON test.user TO 'pig'@'%', 则在使用REVOKE SELECT ON *.* FROM 'pig'@'%';命令并不能撤销该用户对test数据库中user表的SELECT 操作.相反,如果授权使用的是GRANT SELECT ON *.* TO 'pig'@'%';则REVOKE SELECT ON test.user FROM 'pig'@'%';命令也不能撤销该用户对test数据库中user表的Select 权限.
**具体信息可以用命令SHOW GRANTS FOR 'pig'@'%'; 查看.**

**五.删除用户**
```

命令: DROP USER 'username'@'host';

```

###  创建一个web用户 
比如为php脚本创建一个用户。我们应该遵循最少权限原则。
```

grant select , insert, update ,delete on books.* to 'myuser' identified by '123';

```

##  备份与还原 
http://www.cnblogs.com/xcxc/archive/2013/01/30/2882840.html
例如：
数据库地址：127.0.0.1
数据库用户名：root
数据库密码：pass
数据库名称：myweb

<blockquote>备份数据库到D盘跟目录
mysqldump -h127.0.0.1 -uroot -ppass myweb > d:/backupfile.sql
备份到当前目录 备份MySQL数据库为带删除表的格式，能够让该备份覆盖已有数据库而不需要手动删除原有数据库
mysqldump --add-drop-table -h127.0.0.1 -uroot -ppass myweb > backupfile.sql
直接将MySQL数据库压缩备份  备份到D盘跟目录
mysqldump -h127.0.0.1 -uroot -ppass myweb | gzip > d:/backupfile.sql.gz
备份MySQL数据库某个(些)表。此例备份table1表和table2表。备份到linux主机的/home下
mysqldump -h127.0.0.1 -uroot -ppass myweb table1 table2 > /home/backupfile.sql
同时备份多个MySQL数据库
mysqldump -h127.0.0.1 -uroot -ppass --databases myweb myweb2 > multibackupfile.sql
仅仅备份数据库结构。同时备份名为myweb数据库和名为myweb2数据库
mysqldump --no-data -h127.0.0.1 -uroot -ppass --databases myweb myweb2 > structurebackupfile.sql
备份服务器上所有数据库
mysqldump --all-databases -h127.0.0.1 -uroot -ppass > allbackupfile.sql
还原MySQL数据库的命令。还原当前备份名为backupfile.sql的数据库
mysql -h127.0.0.1 -uroot -ppass myweb < backupfile.sql
还原压缩的MySQL数据库
gunzip < backupfile.sql.gz | mysql -h127.0.0.1 -uroot -ppass myweb
将数据库转移到新服务器。此例为将本地数据库myweb复制到远程数据库名为serweb中，其中远程数据库必须有名为serweb的数据库
mysqldump -h127.0.0.1 -uroot -ppass myweb | mysql --host=remove_host -u数据库用户名 -p数据库密码 -C serweb</blockquote>
