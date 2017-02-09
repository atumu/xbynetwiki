title: android使用sqlite保存数据 

#  Android使用SQLite保存数据 
关于SQLite更多信息，请阅读[[booknote:sqlite权威指南2th]]
` android.database.sqlite包 `
##  支持的数据类型 

SQLite3支持 ` NULL、INTEGER、REAL（浮点数字）、TEXT(字符串文本)和BLOB(二进制对象) `数据类型，虽然它支持的类型只有` 五种 `，但实际上sqlite3也接受varchar(n)、char(n)、decimal(p,s) 等数据类型，**只不过在运算或保存时会转成对应的五种数据类型**。 SQLite最大的特点是**你可以把各种类型的数据保存到任何字段中**，而不用关心字段声明的数据类型是什么。例如：可以在Integer类型的字段中存放字符串，或者在布尔型字段中存放浮点数，或者在字符型字段中存放日期型值。 但有一种情况例外：定义为` INTEGER PRIMARY KEY `的字段**只能存储64位整数**， 当向这种字段
保存除整数以外的数据时，将会产生错误。 另外， **SQLite 在解析CREATE TABLE 语句时，会忽略 CREATE TABLE 语句中跟在字段名后面的数据类型信息**，如下面语句会忽略 name字段的类型信息：
CREATE TABLE person (personid integer primary key autoincrement, name varchar(20))
##  两种打开方式的区别 
使用` SQLiteOpenHelper的getWritableDatabase()和getReadableDatabase() `方法都可以获取一个用于操作数据库的SQLiteDatabase实例。但` getWritableDatabase() 方法以读写方式打开数据库 `，一旦数据库的磁盘空间` 满了 `，数据库就只能读而不能写，倘若使用getWritableDatabase()打开数据库就会` 出错 `。` getReadableDatabase()方法先以读写方式打开数据库 `，如果数据库的磁盘空间` 满了 `，就会打开失败，当打开失败后会继续尝试` 以只读方式 `打开数据库。
##  数据库实例缓存机制 
第一次调用getWritableDatabase()或getReadableDatabase()方法后，` SQLiteOpenHelper会缓存当前的SQLiteDatabase实例 `，SQLiteDatabase实例正常情况下会维持数据库的打开状态，所以在你不再需要SQLiteDatabase实例时，请及时调用close()方法释放资源。**一旦SQLiteDatabase实例被缓存，多次调用getWritableDatabase()或getReadableDatabase()方法得到的都是同一实例。**
**一般情况下，无需手动关闭数据库**，但是当你完全不使用或引用退出时才需要关闭它。
##  数据库版本号与更新 
在` onCreate() `方法里可以生成数据库表结构及添加一些应用使用到的初始化数据。` onUpgrade() `方法在数据库的**版本发生变化时**会被调用，一般在软件升级时才需改变版本号，而数据库的版本是由程序员控制的，假设数据库现在的版本是1，由于业务的变更，修改了数据库表结构，这时候就需要升级软件，升级软件时希望更新用户手机里的数据库表结构，为了实现这一目的，可以把原来的数据库版本设置为2，并且在onUpgrade()方法里面实现表结构的更新
```

public class DatabaseHelper extends SQLiteOpenHelper {
//类没有实例化,是不能用作父类构造器的参数,必须声明为静态
private static final String name = "china"; //数据库名称
private static final int version = 1; //数据库版本
public DatabaseHelper(Context context) {
//第三个参数CursorFactory指定在执行查询时获得一个游标实例的工厂类,设置为null,代表使用系统默认的工厂
类
super(context, name, null, version);
}
@Override
public void onCreate(SQLiteDatabase db) {
db.execSQL("CREATE TABLE IF NOT EXISTS person (personid integer primary key autoincrement, name
varchar(20), age INTEGER)");
}
@Override
public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
db.execSQL(" ALTER TABLE person ADD phone VARCHAR(12) NULL "); //往表中增
加一列
// DROP TABLE IF EXISTS person 删除表
}
}

```
在实际项目开发中，当数据库表结构发生更新时，应该避免用户存放于数据库中的数据丢失
##  使用事务操作SQLite数据库 
使用SQLiteDatabase的` beginTransaction() `方法可以开启一个事务，程序执行到` endTransaction() ` 方法时会**检查事务的标志是否为成功**，如果程序执行到endTransaction()之前调用了` setTransactionSuccessful() ` 方法设置事务的标志为成功则提交事务，如果没有调用setTransactionSuccessful() 方法则回滚事务
```

SQLiteDatabase db = ....;
db.beginTransaction();//开始事务
try {
db.execSQL（"insert into person (name, age) values(?,?)", new Object[]{"张三", 4});
db.execSQL("update person set name=? where personid=?", new Object[]{"李四", 1});
db.setTransactionSuccessful();//调用此方法会在执行到endTransaction() 时提交当前事务，如果不调用此方法会回滚事务
} finally {
db.endTransaction();//由事务的标志决定是提交事务，还是回滚事务
}
db.close();
//上面两条SQL语句在同一个事务中执行。

```
##  使用SQLiteDatabase操作SQLite数据库 
使用SQLiteOpenHelper的getWritableDatabase()和getReadableDatabase()方法都可以获取一个用于操作数据库的SQLiteDatabase实例.
SQLiteDatabase类封装了一些操作数据库的API，使用该类可以完成对数据进行` 添加(Create)、查询(Retrieve)、更新(Update)和删除(Delete)操作（这些操作简称为CRUD `）。
对SQLiteDatabase的学习，我们应该重点掌握` execSQL()和rawQuery() `方法。execSQL()方法可以执行insert、delete、update和CREATE TABLE之类有更改行为的SQL语句；rawQuery()方法
用于执行select语句。
```

execSQL()方法的使用例子：
SQLiteDatabase db = ....;
db.execSQL("insert into person(name, age) values('张三', 4)");
db.close();

SQLiteDatabase db = ....;
db.execSQL("insert into person(name, age) values(?,?)", new Object[]{"张三", 4});
db.close();

SQLiteDatabase db = ....;
Cursor cursor = db.rawQuery(“select * from person”, null);
Cursor cursor = db.rawQuery("select * from person where name like ? and age=?", new String[]{"%张三%", "4"});
while (cursor.moveToNext()) {
int personid = cursor.getInt(0); //获取第一列的值,第一列的索引从0开始
String name = cursor.getString(1);//获取第二列的值
int age = cursor.getInt(2);//获取第三列的值
}
cursor.close();
db.close();


```
Cursor是结果集游标，用于对结果集进行随机访问，如果熟悉jdbc， 其实Cursor与JDBC中的ResultSet作用很相似。使用moveToNext()方法可以将游标从当前行移动到下一行，如果已经移过了结果集的最后一行，返回结果为false，否则为true。另外Cursor 还有常用的` moveToPrevious() `方法（用于将游标从当前行移动到上一行，如果已经移过了结果集的第一行，返回值为false，否则为true ）、` moveToFirst( `)方法（用于将游标移动到结果集的第一行，如果结果集为空，返回值为false，否则为true ）和` moveToLast() `方法（用于将游标移动到结果集的最后一行，如果结果集为空，返回值为false，否则为true ） 。
除了前面的` execSQL()和rawQuery() `方法， SQLiteDatabase还专门提供了对应于添加、删除、更新、查询的操作方法： ` insert()、delete()、update()和query() ` 。


##  定义数据列名常量模式和约定 
SQL数据库主要原则之一就是模式：数据库组成方式的正式声明。模式在创建数据库的SQL语句中得以反映。这有助于创建相应配套类即 **contract class** ，它以系统和自记录方式明确规定了主题布局。
**contract class 是一个定义URI、表和列名称的常量集**。 contract class 允许你在同一包内使用相同的常量，由此就能够更改列名称并且在代码中传播。
对整个数据库在根节点进行全局定义是组建 contract class的好办法。**然后为每个枚举列的资料表创建内部类**。
```

public final class FeedReaderContract {
    // To prevent someone from accidentally instantiating the contract class,
    // give it an empty constructor.
    public FeedReaderContract() {}
 
    /* Inner class that defines the table contents */
    public static abstract class FeedEntry implements BaseColumns {
        public static final String TABLE_NAME = "entry";
        public static final String COLUMN_NAME_ENTRY_ID = "entryid";
        public static final String COLUMN_NAME_TITLE = "title";
        public static final String COLUMN_NAME_SUBTITLE = "subtitle";
        ...
    }
}
 

```
##  使用SQLiteOpenHelper创建数据库 
一旦定义了数据库状态，就应该实现创建和维护数据库及表的做法。下面是创建和删除表中的典型语句：
```

private static final String TEXT_TYPE = " TEXT";
private static final String COMMA_SEP = ",";
private static final String SQL_CREATE_ENTRIES =
    "CREATE TABLE " + FeedEntry.TABLE_NAME + " (" +
    FeedEntry._ID + " INTEGER PRIMARY KEY," +
    FeedEntry.COLUMN_NAME_ENTRY_ID + TEXT_TYPE + COMMA_SEP +
    FeedEntry.COLUMN_NAME_TITLE + TEXT_TYPE + COMMA_SEP +
    ... // Any other options for the CREATE command
    " )";
 
private static final String SQL_DELETE_ENTRIES =
    "DROP TABLE IF EXISTS " + FeedEntry.TABLE_NAME;

```
你要做的事情就是调用` getWritableDatabase（）或getReadableDatabase（） `。
注意：因为他们可以长时间运行，请确保你在后台调用 了getWritableDatabase() 或getReadableDatabase()，例如 AsyncTask 或IntentService。
**为了使用 SQLiteOpenHelper，创建一个覆盖 onCreate(), onUpgrade() 和onOpen()回调方法的子类。**你还可以实施 onDowngrade()，虽然这不是必需的。
例如使用上述显示命令行实现SQLiteOpenHelper：
```

public class FeedReaderDbHelper extends SQLiteOpenHelper {
    // If you change the database schema, you must increment the database version.
    public static final int DATABASE_VERSION = 1;
    public static final String DATABASE_NAME = "FeedReader.db";
 
    public FeedReaderDbHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(SQL_CREATE_ENTRIES);
    }
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // This database is only a cache for online data, so its upgrade policy is
        // to simply to discard the data and start over
        db.execSQL(SQL_DELETE_ENTRIES);
        onCreate(db);
    }
    public void onDowngrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        onUpgrade(db, oldVersion, newVersion);
    }
}

```
为了访问数据库，需要实例化 SQLiteOpenHelper的子类：
` FeedReaderDbHelper mDbHelper = new FeedReaderDbHelper(getContext()); `
##  将信息放入数据库 
向 ` insert() `方式传递 ` ContentValues ` 对象就可以将数据插入到数据库。
各个字段的数据使用` ContentValues `进行存放。ContentValues类似于MAP

数据类型，如：String、Integer等。
```

// Gets the data repository in write mode
SQLiteDatabase db = mDbHelper.getWritableDatabase();
 
// Create a new map of values, where column names are the keys
ContentValues values = new ContentValues();
values.put(FeedEntry.COLUMN_NAME_ENTRY_ID, id);
values.put(FeedEntry.COLUMN_NAME_TITLE, title);
values.put(FeedEntry.COLUMN_NAME_CONTENT, content);
 
// Insert the new row, returning the primary key value of the new row
long newRowId;
newRowId = db.insert(
         FeedEntry.TABLE_NAME,
         FeedEntry.COLUMN_NAME_NULLABLE,
         values);

```
不管第三个参数是否包含数据，**执行Insert()方法必然会添加一条记录**，如果第三个参数为空，会添加一条除主键之外其他字段值为Null的记录。Insert()方法内部实际上通过构造insert SQL语句完成数据的添加.
第二个参数用于指定空值字段的名称，该参数的作用是什么？是这样的：如果第三个参数values 为Null或者元素个数为0， 由于Insert()方法要求必须添加一条除了主键之外其它字段为Null值的记录，为了满足SQL语法的需要， insert语句必须给定一个字段名，如：insert into person(name) values(NULL)，倘若不给定字段名， insert语句就成了这样：insert into person() values()，显然这不满足标准SQL的语法。对于字段名，建议使用主键之外的字段，如果使用了INTEGER类型的主键字段，执行类似insert into person(personid)values(NULL)的insert语句后，该主键字段值也不会为NULL。如果第三个参数values 不为Null并且元素的个数大于0 ，可以把第二个参数设置为null。

##  从数据库读取信息 
使用 ` query() ` 方法从数据库中读取，把你的选择标准和所需列传递给它。这个方法把 insert() 和update()元素结合在一起，注意列的列表定义要获取的数据而不是要插入的数据。查询结果作为 ` Cursor `对象返回给你。
` 注意:第一列的索引从0开始 `
```

SQLiteDatabase db = mDbHelper.getReadableDatabase();
 
// Define a projection that specifies which columns from the database
// you will actually use after this query.
String[] projection = {
    FeedEntry._ID,
    FeedEntry.COLUMN_NAME_TITLE,
    FeedEntry.COLUMN_NAME_UPDATED,
    ...
    };
 
// How you want the results sorted in the resulting Cursor
String sortOrder =
    FeedEntry.COLUMN_NAME_UPDATED + " DESC";
 
Cursor c = db.query(
    FeedEntry.TABLE_NAME,  // The table to query
    projection,                               // The columns to return
    selection,                                // The columns for the WHERE clause
    selectionArgs,                            // The values for the WHERE clause
    null,                                     // don't group the rows
    null,                                     // don't filter by row groups
    sortOrder                                 // The sort order
    );

```
查看行指针要使用 Cursor移动方法，**在开始读值之前你必须始终调用该方法**。` 通常开始会调用 moveToFirst() `，对于每一行都可以调用光标方式来进行列表读值，比如`  getString() 或者getLong(). `
对于每一个get方法，必须调用`  getColumnIndex() ` 和getColumnIndexOrThrow()得到所需列的索引位置，进而传递出去。例如:
```

cursor.moveToFirst();
long itemId = cursor.getLong(
    cursor.getColumnIndexOrThrow(FeedEntry._ID)
);

```
```

while (cursor.moveToNext()) {
int personid = cursor.getInt(0); //获取第一列的值,第一列的索引从0开始
String name = cursor.getString(1);//获取第二列的值
}

```
table：表名。相当于select语句from关键字后面的部分。如果是多表联合查询，可以用逗号将
两个表名分开。
columns：要查询出来的列名。相当于select语句select关键字后面的部分。
selection：查询条件子句，相当于select语句where关键字后面的部分，在条件子句允许使用占
位符“?”
selectionArgs：对应于selection语句中占位符的值，值在数组中的位置与占位符在语句中的位
置必须一致，否则就会有异常。
groupBy：相当于select语句group by关键字后面的部分
having：相当于select语句having关键字后面的部分
orderBy：相当于select语句order by关键字后面的部分，如：personid desc, age asc;
limit：指定偏移量和获取的记录数，相当于select语句limit关键字后面的部分。

##  从数据库中删除信息 
要删除表中的行，需要提供确定行的选择标准。数据库API提供了一种用于创建选择标准的机制。该机制把选择规范划分为选择条款和选择参数。这一参数是用来测试绑定条款的值。因为结果与普通SQL语句处理不完全相同，SQL插入不会对它产生影响。
```

// Define 'where' part of query.
String selection = FeedEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
// Specify arguments in placeholder order.
String[] selectionArgs = { String.valueOf(rowId) };
// Issue SQL statement.
db.delete(table_name, selection, selectionArgs);

```
```

SQLiteDatabase db = databaseHelper.getWritableDatabase();
db.delete("person", "personid<?", new String[]{"2"});
db.close();

```
上面代码用于从person表中删除personid小于2的记录。
第二个参数表示选择的条件，相当于sql语句中 where后面的语句。
第三个参数为站位符的值。
##  更新数据库 

当你需要修改数据库值的子集时，使用 update()方式。
结合 insert()和 delete()内容值语法来更新资料表。
```

SQLiteDatabase db = mDbHelper.getReadableDatabase();
 
// New value for one column
ContentValues values = new ContentValues();
values.put(FeedEntry.COLUMN_NAME_TITLE, title);
 
// Which row to update, based on the ID
String selection = FeedEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
String[] selectionArgs = { String.valueOf(rowId) };
 
int count = db.update(
    FeedReaderDbHelper.FeedEntry.TABLE_NAME,
    values,
    selection,
    selectionArgs);

```
