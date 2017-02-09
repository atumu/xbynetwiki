title: python学习笔记16 

#  Python学习笔记之MySQL操作 
安装MySQL驱动
目前，有三个MySQL驱动：
  * mysql-connector-python：是MySQL官方的纯Python驱动；
  * MySQL-python：是封装了MySQL C驱动的Python驱动。
  * mysqlclient
由于MySQL服务器以独立的进程运行，并通过网络对外服务，所以，需要支持Python的MySQL驱动来连接到MySQL服务器。*
官网文档:http://dev.mysql.com/doc/connector-python/en/
```

$ pip install mysql-connector-python
但是不知为什么一直出现问题。所以采用如下方式安装:参考http://stackoverflow.com/questions/27394426/python-pip-install-mysql-connector-python-2-0-1-fails
wget https://cdn.mysql.com/Downloads/Connector-Python/mysql-connector-python-2.1.3.zip
unzip mysql-connector-python-2.1.3.zip
cd mysql-connector-python-2.1.3
python setup.py install

```
如果你想使用mysql-python可以pip install mysql-python

我们演示如何连接到MySQL服务器的test数据库：
```

# 导入MySQL驱动:
>>> import mysql.connector
# 注意把password设为你的root口令:
>>> conn = mysql.connector.connect(user='root', password='password', database='test')
>>> cursor = conn.cursor()
# 创建user表:
>>> cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')
# 插入一行记录，注意MySQL的占位符是%s:
>>> cursor.execute('insert into user (id, name) values (%s, %s)', ['1', 'Michael'])
>>> cursor.rowcount
1
# 提交事务:
>>> conn.commit()
>>> cursor.close()
# 运行查询:
>>> cursor = conn.cursor()
>>> cursor.execute('select * from user where id = %s', ['1'])
>>> values = cursor.fetchall()
>>> values
[('1', 'Michael')]
# 关闭Cursor和Connection:
>>> cursor.close()
True
>>> conn.close()

```
由于Python的DB-API定义都是通用的，所以，操作MySQL的数据库代码和SQLite类似。
```

>>> cur = conn.cursor()

```
此后，就可以利用游标对象的方法对数据库进行操作。那么还得了解游标对象的常用方法：
名称	描述
  * close()	关闭游标。之后游标不可用
  * execute(query[,args])	执行一条SQL语句，可以带参数
  * executemany(query, pseq)	对序列pseq中的每个参数执行sql语句
  * fetchone()	返回一条查询结果
  * fetchall()	返回所有查询结果
  * fetchmany([size])	返回size条结果
  * nextset()	移动到下一个结果
  * scroll(value,mode='relative')移动游标到指定行，如果mode='relative',则表示从当前所在行移动value条,如果mode='absolute',则表示从结果集的第一行移动value条.

小结
  * 执行INSERT等操作后要调用commit()提交事务；
  * MySQL的SQL占位符是%s。
API:
```

mysql.connector
  errorcode
  errors
  connection
  constants
  conversion
  cursor
  dbapi
  locales
    eng
      client_error
  protocol
  utils

```