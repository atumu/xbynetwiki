title: getwritable_readable_db_conn 

#  android:getReadableDatabase与getWritableDatabase的区别及数据库连接与关闭的问题 

` 关于数据库连接，由于数据库SQLiteDatabase对象会被缓存，不使用时无需手动关闭。除非你确定不再有任何使用SQLiteDatabase的时候，否则完全没有必要手动关闭数据库。 `

Android使用getWritableDatabase()和getReadableDatabase()方法都可以获取一个用于操作数据库的SQLiteDatabase实例。(` getReadableDatabase()方法中会调用getWritableDatabase()方法 `)
其中getWritableDatabase() 方法以读写方式打开数据库，一旦数据库的磁盘空间满了，数据库就只能读而不能写，倘若使用的是getWritableDatabase() 方法就会出错。

getReadableDatabase()方法则是` 先以读写方式打开数据库 `，如果数据库的磁盘空间满了，就会打开失败，当打开失败后会继续尝试以只读方式打开数据库。如果该问题成功解决，则只读数据库对象就会关闭，然后返回一个可读写的数据库对象。

但是我们还是提倡对于创建或者更改数据库操作时还是先调用getWritableDatabase(),如果不成功再调用getReadableDatabase().
推荐形式：
```

   	try {  
          db=helper.getWritableDatabase();  
        } catch (SQLiteException e) {  
           db=helper.getReadableDatabase();
        }  

```