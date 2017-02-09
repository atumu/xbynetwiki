title: oracle入门之dblink 

#  oracle入门之dblink 
database link概述 
**database link是定义一个数据库到另一个数据库的路径的对象**，database link允许你**查询远程表及执行远程程序**。在任何分布式环境里，database都是必要的。另外要注意的是database link是**单向的连接**。
在创建database link的时候，Oracle再数据字典中保存相关的database link的信息，**在使用database link的时候，Oracle通过Oracle Net用用户预先定义好的连接信息访问相应的远程数据库以完成相应的工作。**
建立database link之前需要确认的事项：
确认从local database到remote database的网络连接是正常的，**tnsping**要能成功。
确认在remote database上面有相应的访问**权限**。

**database link分类**
类型Owner描述
  * private	创建database link的user拥有该database link。在本地数据库的特定的schema下建立的database link。只有建立该database link的schema的session能使用这个database link来访问远程的数据库。同时也只有Owner能删除它自己的private database link。
  * Public	Owner是PUBLIC.	Public的database link是数据库级的，本地数据库中所有的拥有数据库访问权限的用户或pl/sql程序都能使用此database link来访问相应的远程数据库。
  * Global	Owner是PUBLIC.	Global的database link是网络级的，When an Oracle network uses a directory server, the directory server automatically create and manages global database links (as net service names) for every Oracle Database in the network. Users and PL/SQL subprograms in any database can use a global link to access objects in the corresponding remote database. 
Note: In earlier releases of Oracle Database, a global database link referred to a database link that was registered with an Oracle Names server. The use of an Oracle Names server has been deprecated. In this document, global database links refer to the use of net service names from the directory server.

**创建dblink所需的权限**
PrivilegeDatabaseRequired For
CREATE DATABASE LINK	Local	Creation of a private database link.
CREATE PUBLIC DATABASE LINK	Local	Creation of a public database link.
CREATE SESSION	Remote	Creation of any type of database link.

database link的使用 
基本语法
```

CREATE [SHARED][PUBLIC] database link link_name
      [CONNECT TO [user][current_user] IDENTIFIED BY password]
      [AUTHENTICATED BY user IDENTIFIED BY password]
      [USING 'connect_string']

```
不使用TNS Name一例：
```

CREATE database link link_name
CONNECT TO user IDENTIFIED BY screct
USING '(DESCRIPTION =
    (ADDRESS_LIST =
        (ADDRESS = (PROTOCOL = TCP)(HOST = sales.company.com)(PORT = 1521))
    )
    (CONNECT_DATA =
        (SERVICE_NAME = sales)
    )
)';

```

**database link的使用** 
-- 最简单的用法
SELECT * FROM table_name@database link; 
-- 不想让使用的人知道database link的名字的时候
-- 建一个别名包装一下 
CREATE SYNONYM table_name FOR table_name@database link;
SELECT * FROM table_name; 
-- 或者，也可以建立一个视图来封装
CREATE VIEW table_name AS SELECT * FROM  table_name@database link; 
database link删除
-- 删除public类型的database link
DROP PUBLIC database link link_name; 
-- 删除非public类型的database link
-- 注意：只有owner自己能删除自己的非public类型database link
DROP database link link_name; 

**查看database link的信息**
查看系统database link的基本信息
DBA_DB_LINKS (ALL_DB_LINKS/USER_DB_LINKS)


删除
注意:用户有create public database link 或者create database link 权限.
drop public database link dblinkname;

参考：http://czmmiao.iteye.com/blog/1236562
http://blog.itpub.net/24104981/viewspace-1116085/