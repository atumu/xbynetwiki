title: python学习笔记17 

#  Python学习笔记之SQLite操作 
建立连接对象
**由于SQLite数据库的驱动已经在python里面了**，所以，只要引用就可以直接使用
```

>>> import sqlite3
>>> conn = sqlite3.connect("23301.db")

```
这样就得到了连接对象，是不是比mysql连接要简化了很多呢。在sqlite3.connect("23301.db")语句中，如果已经有了那个数据库，就连接上它；如果没有，就新建一个。注意，这里的路径可以随意指定的。
连接对象建立起来之后，就要使用连接对象的方法继续工作了。
```

>>> dir(conn)
['DataError', 'DatabaseError', 'Error', 'IntegrityError', 'InterfaceError', 'InternalError', 'NotSupportedError', 'OperationalError', 'ProgrammingError', 'Warning', '__call__', '__class__', '__delattr__', '__doc__', '__enter__', '__exit__', '__format__', '__getattribute__', '__hash__', '__init__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'close', 'commit', 'create_aggregate', 'create_collation', 'create_function', 'cursor', 'enable_load_extension', 'execute', 'executemany', 'executescript', 'interrupt', 'isolation_level', 'iterdump', 'load_extension', 'rollback', 'row_factory', 'set_authorizer', 'set_progress_handler', 'text_factory', 'total_changes']

```
游标对象
```

>>> cur = conn.cursor()

```
接下来对数据库内容的操作，都是用游标对象方法来实现了：
```

>>> dir(cur)
['__class__', '__delattr__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__iter__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'arraysize', 'close', 'connection', 'description', 'execute', 'executemany', 'executescript', 'fetchall', 'fetchmany', 'fetchone', 'lastrowid', 'next', 'row_factory', 'rowcount', 'setinputsizes', 'setoutputsize']

```
是不是看到熟悉的名称了：close(), execute(), executemany(), fetchall()
创建数据库表
在mysql中，我们演示的是利用mysql的shell来创建的表。其实，当然可以使用sql语句，在python中实现这个功能。这里对sqlite数据库，就如此操作一番。
```

>>> create_table = "create table books (title text, author text, lang text) "
>>> cur.execute(create_table)
<sqlite3.Cursor object at 0xb73ed5a0>
这样就在数据库23301.db中建立了一个表books。对这个表可以增加数据了：
>>> cur.execute('insert into books values ("from beginner to master", "laoqi", "python")')
<sqlite3.Cursor object at 0xb73ed5a0>
为了保证数据能够保存，还要（这是多么熟悉的操作流程和命令呀）：
>>> conn.commit()
>>> cur.close()
>>> conn.close()

```
支持，刚才建立的那个数据库中，已经有了一个表books，表中已经有了一条记录。
整个流程都不陌生。
**查询**
存进去了，总要看看，这算强迫症吗？
```

>>> conn = sqlite3.connect("23301.db")
>>> cur = conn.cursor()
>>> cur.execute('select * from books')
<sqlite3.Cursor object at 0xb73edea0>
>>> print cur.fetchall()
[(u'from beginner to master', u'laoqi', u'python')]

```
**批量插入**
多增加点内容，以便于做别的操作：
```

>>> books = [("first book","first","c"), ("second book","second","c"), ("third book","second","python")]
这回来一个批量插入
>>> cur.executemany('insert into books values (?,?,?)', books)
<sqlite3.Cursor object at 0xb73edea0>
>>> conn.commit()

```
用循环语句打印一下查询结果：
```

>>> rows = cur.execute('select * from books')
>>> for row in rows:
...     print row
... 
(u'from beginner to master', u'laoqi', u'python')
(u'first book', u'first', u'c')
(u'second book', u'second', u'c')
(u'third book', u'second', u'python')

```
**更新**
正如前面所说，在cur.execute()中，你可以写SQL语句，来操作数据库。
```

>>> cur.execute("update books set title='physics' where author='first'")
<sqlite3.Cursor object at 0xb73edea0>
>>> conn.commit()
按照条件查处来看一看：
>>> cur.execute("select * from books where author='first'")
<sqlite3.Cursor object at 0xb73edea0>
>>> cur.fetchone()
(u'physics', u'first', u'c')

```
**删除**
在sql语句中，这也是常用的。
```

>>> cur.execute("delete from books where author='second'")
<sqlite3.Cursor object at 0xb73edea0>
>>> conn.commit()

>>> cur.execute("select * from books")
<sqlite3.Cursor object at 0xb73edea0>
>>> cur.fetchall()

[(u'from beginner to master', u'laoqi', u'python'), (u'physics', u'first', u'c')]
不要忘记，在你完成对数据库的操作是，一定要关门才能走人：
>>> cur.close()
>>> conn.close()

```