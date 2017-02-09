title: android官方文档学习之content_providers 

#  android官方文档学习之content_providers 
Content providers是用来管理对结构化数据集进行访问的一组接口。这组接口对数据进行封装，并提供了用于定义数据安全的机制。Content providers是一个进程使用另一个进程数据的标准接口。
当要使用` content provider `访问数据时，我们需要在应用程序的Context中` 使用ContentResolver对象作为客户端 `，同provider进行通信。与provier对象通信的ContentResolver对象是ContentProvider类的一个实例。provider对象接收从客户端发来的数据，执行请求的动作并返回结果。
**如果你不打算同其他应用程序共享数据，就没必要实现provider。**但是，如果希望在自己的应用程序中**搜索建议**的功能，就需要实现自己的provider。同样的，如果希望在自己的应用程序和其他的应用程序间**拷贝粘贴复杂的数据或文件**，也需要实现自己的provider。
Android系统本身也通过content providers来管理数据，如**音频，视频，图像，个人联系信息**等。我们可以在**android.provider包的参考文档中看到这些providers列表**。在一定条件下，这些providers能够访问任何Android应用程序。

**Content provider管理对于中央数据库的访问。**而provider是Android应用程序的一部分，通常每个provider自己提供界面同数据交互。然而，content providers最主要的目的是为了让其他应用程序使用provider的客户端对象访问provider。所以，providers和provider客户端共同提供了一个一致的标准数据接口用来处理**进程间通信和安全的数据访问。**
##  Content Provider 
在你开始创建一个Provider之前, 做如下:
确定你需要一个Content Provider. 如果你需要提供一个或多个以下特征, 虚拟需要创建Content Provider:
  * 你希望向其他程序提供复杂的数据或者文件
  * 你希望用户从你的程序复制复杂数据到其他程序
  * 你希望提供用搜索引擎框架提供自定义搜索提示
  * 如果用途完全只在你的程序内部, 你就不需要Provider去用SQLite数据库.
接下来, 遵循以下步骤来构建你的Provider:
1、数据类型
文件数据
数据通常装入文件, 诸如图片, 音频, 或者视频. 将文件存储在你的私人空间. 作为其他程序请求你的文件的响应, 你的Provider提供一个文件的handle.
"格式化"数据
数据通常放入数据库, 数组, 或者类似结构. 将数据存储在一个表格,匹配表的行和列。一行表示一个实体, 如一个人或者一个清单条目. 一列表示实体的一些数据, 如人的名字, 或者条目的价格. 储存这种数据的常见方式是用SQLite 数据库, 但你可以使用任何类型的持久性存储.
2.定义ContentProvider类的具体实现及其所需的方法。这个类是你的数据和Android系统的其余部分之间的接口。
3.定义Provider的授权字符串，其内容的URI，和列名。如果你想Provider的程序处理的Intent，也要定义Intent行动，额外的数据，和标志。还可以定义您将需要的应用程序要访问数据的权限。你应该考虑将所**有这些值作为一个单独的contract类中的常量定义**; 之后，你将这个类公开给其他开发者。
4.添加其他可选件，如样本数据或AbstractThreadedSyncAdapter的实现, 能实现Provider和云端数据的同步

##  Content Uri分析 

```

 <provider android:name=".PersonContentProvider" 
           android:authorities="com.ljq.providers.personprovider"/>

```
![](/data/dokuwiki/android/pasted/20150609-033959.png)
要操作person表中id为10的记录，可以构建这样的路径:/person/10
要操作person表中id为10的记录的name字段， person/10/name
要操作person表中的所有记录，可以构建这样的路径:/person
要操作xxx表中的记录，可以构建这样的路径:/xxx
当然要操作的数据不一定来自数据库，也可以是文件、xml或网络等其他存储方式，如下:
要操作xml文件中person节点下的name节点，可以构建这样的路径：/person/name
如果要把一个字符串转换成Uri，可以使用Uri类中的parse()方法，如下：
```

Uri uri = Uri.parse("content://com.ljq.provider.personprovider/person")

```
Android系统提供了两个用于操作Uri的工具类，分别为` UriMatcher和ContentUris ` 。掌握它们的使用，会便于我们的开发工作。
###  UriMatcher 

` UriMatcher `类用于匹配Uri，它的用法如下：
首先第一步把你需要匹配Uri路径全部给注册上，如下：
```

//常量UriMatcher.NO_MATCH表示不匹配任何路径的返回码
UriMatcher  sMatcher = new UriMatcher(UriMatcher.NO_MATCH);
//如果match()方法匹配content://com.ljq.provider.personprovider/person路径，返回匹配码为1
sMatcher.addURI("com.ljq.provider.personprovider", "person", 1);//添加需要匹配uri，如果匹配就会返回匹配码
//如果match()方法匹配content://com.ljq.provider.personprovider/person/230路径，返回匹配码为2
sMatcher.addURI("com.ljq.provider.personprovider", "person/#", 2);//#号为通配符
switch (sMatcher.match(Uri.parse("content://com.ljq.provider.personprovider/person/10"))) { 
   case 1
     break;
   case 2
     break;
   default://不匹配
     break;
}

```
注册完需要匹配的Uri后，就可以使用sMatcher.match(uri)方法对输入的Uri进行匹配**，如果匹配就返回匹配码，匹配码是调用addURI()方法传入的第三个参数**，假设匹配content:/ /com.ljq.provider.personprovider/person路径，返回的匹配码为1 
###  ContentUris 

` ContentUris `类用于操作Uri路径后面的**ID**部分，它有两个比较实用的方法：
` withAppendedId(uri, id) `用于为路径加上ID部分：
` parseId(uri) `方法用于从路径中获取ID部分：
```

Uri uri = Uri.parse("content://com.ljq.provider.personprovider/person")
Uri resultUri = ContentUris.withAppendedId(uri, 10); 
//生成后的Uri为：content://com.ljq.provider.personprovider/person/10

```


Uri uri = Uri.parse("content://com.ljq.provider.personprovider/person/10")
long personid = ContentUris.parseId(uri);//获取的结果为:10
##  实现ContentProvider类 
```

public class PersonContentProvider extends ContentProvider{
   public boolean onCreate()
   public Uri insert(Uri uri, ContentValues values)
   public int delete(Uri uri, String selection, String[] selectionArgs)
   public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
   public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
   public String getType(Uri uri)
}

     <provider android:name=".PersonContentProvider" 
           android:authorities="com.ljq.providers.personprovider"/>

```
onCreate()
初始化您的provider。Android系统一创建你的Provider就调用这个方法。注意直到ContentResolver对象试图访问时,Provider才创建。
注意,这些方法与ContentResolver中同样命名方法有相同的签名。
query()
从您的Provider检索数据。使用参数选择表去查询,返回行和列,按顺序排列结果。返回数据作为Cursor对象。
insert()
插入一个新行到您的Provider。使用参数来选择目标表及获得的列值使用。为新插入的行返回一个Content URI。
update()
在你的Provider更新现有的行。使用参数选择表和行更新并得到更新的列值。返回更新的行数。
delete()
从你的Provider删除行。使用参数选择要删除的表和行。返回删除的行数。
**getType()**返回Content URI对应的IME类型.
  * 如果操作的数据属于` 集合类型 `，那么MIME类型字符串应该以` vnd.android.cursor.dir/ `开头，
例如：要得到所有person记录的Uri为content:/ /com.ljq.provider.personprovider/person，那么返回的MIME类型字符串应该为："` vnd.android.cursor.dir/person `"。
  * 如果要操作的数据属于` 非集合类型数据 `，那么MIME类型字符串应该以` vnd.android.cursor.item/ `开头，
例如：得到id为10的person记录，Uri为content:/ /com.ljq.provider.personprovider/person/10，那么返回的MIME类型字符串为："` vnd.android.cursor.item/person `"。

这些方法的实现应该遵循以下原则：
  * 除了onCreate()， 所有的这些方法可能被多个线程同时调用,所以他们必须是线程安全的。学习更多关于多线程， 查看Processes and Threads专题
  * 避免在onCreate()做耗时的操作。延迟初始化任务,直到他们真正需要。部 Implementing the onCreate() method章节更细致的讨论了这个问题。
  * 尽管您必须实现这些方法,除了返回预期的数据类型，你的代码可以不做任何事。例如,您可能想要阻止其他应用程序将数据插入某些表。要做到这一点,您可以忽略该insert()调用和返回0。


##  ContentResolver 
Content provider以类似于关系数据库中一个或多个表的方式给外部的应用程序提供数据。一行代表provider收集的一些类型数据的一个实例，一列中的每一行代表数据中某个类型的一个实例。
` 当外部应用需要对ContentProvider中的数据进行添加、删除、修改和查询操作时，可以使用ContentResolver 类来完成，要获取ContentResolver 对象，可以使用Activity提供的getContentResolver()方法。 `
 ContentResolver 类提供了与ContentProvider类相同签名的四个方法：
public Uri insert(Uri uri, ContentValues values)：该方法用于往ContentProvider添加数据。
public int delete(Uri uri, String selection, String[] selectionArgs)：该方法用于从ContentProvider删除数据。
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)：该方法用于更新ContentProvider中的数据。
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)：该方法用于从ContentProvider中获取数据。
例如，用户字典就是Android平台内置的providers中的一个，用来保存用户希望保存的非标准词的拼写。表1展示了provider表中数据可能的存储方式：
表1：用户字典表的例子
![](/data/dokuwiki/android/pasted/20150609-023520.png)
注：主键对于provider来说不是必须的，即使有主键，provider也不必一定要使用_ID作为主键的列名。**但是，如果希望将provider的数据绑定到ListView上，就要使用_ID作为一列的名字。**这些会在显示查询结果的小结中详细介绍。

##  访问 provider 
应用程序使用` ContentResolver客户端 `对象来` 访问content provider的数据 `。ContentResolver对象与ContentProvider的一个具体子类的实例拥有相同名字的接口。ContentResolver的方法提供了基本的‘CRUD’（创建，检索，更新和删除）数据存储的功能。
要获取ContentResolver 对象，可以使用` Activity提供的getContentResolver() `方法。
客户端应用程序进程中的ContentResolver对象和我们自己的应用程序中的ContentProvider对象会自动处理进程间通信。ContentProvider也会作为其存储的数据和数据以表的形式展现之间的抽象层。
` 注：为访问provider，应用程序通常需要在manifest文件中添加特定的权限。 `这些将在Content Provider 权限小结中详细介绍。
举例来说，为了从用户字典的Provider中获得单词和它们出现的语言环境列表，就需要调用` ContentResolver.query() `方法。query()方法会调用在用户字典的Provider中定义的ContentProvider.query()方法。下面这行代码展示了ContentResolver.query()是如何调用的：
```

// Queries the user dictionary and returns results
mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,   // 词表的内容URI
    mProjection,                        // 每行中返回数据的列的名称，
                                        // null表示返回所有列的数据。
    mSelectionClause                    // 过滤条件
    mSelectionArgs,                     // 过滤条件的参数
    mSortOrder);                        // 返回行的排序方式

```
表2：Query()函数与SQL的查询语句间的对应关系
![](/data/dokuwiki/android/pasted/20150609-023931.png)

##  从Provider中检索数据 
为了方便起见，本节中的代码会在“UI线程”中调用` ContentResolver.query() `函数。但是，**在实际代码中，应该将查询操作放到一个单独的线程中异步执行。一种方法就是使用` CursorLoader `类，**这个类的详细说明可以在Loaders手册中查到。此外，例子代码仅仅是一部分，它们并没有显示完整的应用程序。 
为了从provider中检索数据，请遵循以下的**基本步骤**：
  * 获得provider的读**访问权限**。
  * 定义发送给provider的查询代码。

用户词表的Provider在它的manifest文件中定义了` android.permission.READ_USER_DICTIONARY `权限，所以如果一个应用程序希望从provider中读取信息，就必须获得此权限。
###  构建查询程序 
```

// projection 定义了返回的结果中需包含的列。
String[] mProjection =
{
    UserDictionary.Words._ID,    // Contract class constant for the _ID column name
    UserDictionary.Words.WORD,   // Contract class constant for the word column name
    UserDictionary.Words.LOCALE  // Contract class constant for the locale column name
};
 
// 定义一个字符串用来包含selection的条件。
String mSelectionClause = null;
 
// 初始化数组包含selection的参数。
String[] mSelectionArgs = {""};

```
##  显示查询结果 
客户端的` ContentResolver.query() `方法会返回一个` Cursor `对象，包含了满足查询条件的行及由查询的projection指定的列。一个Cursor对象为它所包含的行和列提供了随机读取的访问。使用Cursor的方法，可以遍历查询结果中的所有行，确定每列的数据类型，获取一列的数据，及检验结果的其他属性。**当provider的数据改变时，有些Cursor实现了自动更新的功能，或者当Cursor改变时，会触发一个监听对象的方法，或者两者兼而有之。**
 因为Cursor是所有行的列表，所以显示Cursor内容的一个好的方式是使用SimpleCursorAdapter将其连接到ListView上。
下面的代码延续了前面的程序。创建了一个SimpleCursorAdapter对象，其中包含了查询结果的Cursor，并将此对象设置为一个ListView的适配器：
```

// Defines a list of columns to retrieve from the Cursor and load into an output row
String[] mWordListColumns =
{
    UserDictionary.Words.WORD,   // Contract class constant containing the word column name
    UserDictionary.Words.LOCALE  // Contract class constant containing the locale column name
};
 
// Defines a list of View IDs that will receive the Cursor columns for each row
int[] mWordListItems = { R.id.dictWord, R.id.locale};
 
// Creates a new SimpleCursorAdapter
mCursorAdapter = new SimpleCursorAdapter(
    getApplicationContext(),               // The application's Context object
    R.layout.wordlistrow,                  // A layout in XML for one row in the ListView
    mCursor,                               // The result from the query
    mWordListColumns,                      // A string array of column names in the cursor
    mWordListItems,                        // An integer array of view IDs in the row layout
    0);                                    // Flags (usually none are needed)
 
// Sets the adapter for the ListView
mWordList.setAdapter(mCursorAdapter);

```
注：若要用Cursor来备份ListView，**就需要此cursor包含一个名为_ID的列**。正因为这样，前面对单词表的查询会收到_ID这一列，即使ListView并没有显示该列。此限制条件也说明了为什么大多数providers会在它们的每个表中包含一个_ID列。
##  Content Provider 权限 
下面的<uses-permission>标签请求获取用户词典Provider的读权限：
<uses-permission android:name="android.permission.READ_USER_DICTIONARY">
##  插入，更新和删除数据 
你可以调用` ContentResolver.insert() `方法往provide中插入数据` ContentValues `。该方法会往provide中插入一行新的数据并返回该数据的URI。下面的例子演示了如何往User Dictionary Provider中插入一个新单词：
 
```

// Defines a new Uri object that receives the result of the insertion
Uri mNewUri;
 
...
 
// Defines an object to contain the new values to insert
ContentValues mNewValues = new ContentValues();
 
/*
 
* Sets the values of each column and inserts the word. The arguments to the "put"
 
* method are "column name" and "value"
 */
mNewValues.put(UserDictionary.Words.APP_ID,"example.user");
mNewValues.put(UserDictionary.Words.LOCALE, "en_US");
mNewValues.put(UserDictionary.Words.WORD, "insert");
mNewValues.put(UserDictionary.Words.FREQUENCY, "100");
 
mNewUri = getContentResolver().insert(    
UserDictionary.Word.CONTENT_URI,   // the user dictionary content URI    
mNewValues                          // the values to insert
);

``` 
newUri会返回content URI以区分这条新增加的数据，跟下面的格式一样:
```

content://user_dictionary/words/<id_value>

```
在客户端你要使用 [) ContentResolver.update()]方法。你仅仅只需要把你想要更改的字段的添加到ContentValues对象中就可以了。如果你想删除某个字段的内容，可以把它设为null。 下面的代码把所有行的环境语言字段由“en”改为null。返回的值是被更改的行数：
```

// Defines an object to contain the updated values
ContentValues mUpdateValues = new ContentValues();
 
// Defines selection criteria for the rows you want to update
String mSelectionClause = UserDictionary.Words.LOCALE +  "LIKE ?";
String[] mSelectionArgs = {"en_%"};
 
// Defines a variable to contain the number of updated rows
int mRowsUpdated = 0;
 
...
 
/*
 
* Sets the updated value and updates the selected words.
 */
mUpdateValues.putNull(UserDictionary.Words.LOCALE);
 
mRowsUpdated = getContentResolver().update(    
UserDictionary.Words.CONTENT_URI, // the user dictionary contentURI    
mUpdateValues                         // the columns to update    
mSelectionClause                    // the column to select on    
 mSelectionArgs                      // the value to compare to
);
 

```
删除数据与检索数据很类似：你用特定的选择条件选出你要删除的数据，然后客户端的方法会返回删除的行数。下面的代码删除了appid字段的值为“user”的所有数据，并返回了删除的行数。
```

// Defines selection criteria for the rows you want to delete
String mSelectionClause = UserDictionary.Words.APP_ID + " LIKE ?";
String[] mSelectionArgs = {"user"};
 
// Defines a variable to contain the number of rows deleted
int mRowsDeleted = 0;
 
...
 
// Deletes the words that match the selection criteria
mRowsDeleted = getContentResolver().delete(
UserDictionary.Words.CONTENT_URI,   // the user dictionary content URI  mSelectionClause       // the column to select on    mSelectionArgs                      // the value to compare to
);
 

```
##  Provider的其它访问形式 
Provider的其它访问形式在应用程序开发过程中也是很重要的： 
**Batch access**：你可以调用 ` ContentProviderOperation `类中的方法创建批处理访问，然后调用 ` ContentResolver.applyBatch() `来应用它们。 
**异步查询**：你应该在一个单独的线程中执行查询操作。使用 ` CursorLoader ` 对象是这种操作的一种方式。Loaders的引导例子演示了怎么操作这个对象。 
**通过intent访问数据**：尽管你不能直接发送一个intent给一个provider，但是你可以发送一个intent给provider的应用程序，这样做往往是修改provider中数据最好的方式。 下面的内容介绍了批处理访问数据和通过intent修改数据。
 要在**"batch mode**"中访问provider，你要创建一个**ContentProviderOperation对象的数组**，然后通过` ContentResolver.applyBatch() `来把他们分派到content provider中。你应该把content **provider的authority**传到这个方法中，而不是一个特定的content URI,它允许这个数组中的每一个ContentProviderOperation对象能对不同的表进行操作。调用ContentResolver.applyBatch()方法会以数组的形式返回结果。
##  监听ContentProvider中数据的变化 
如果ContentProvider的访问者需要知道ContentProvider中的数据发生变化，可以**在ContentProvider发生数据变化时调用getContentResolver().notifyChange(uri, null)来通知注册在此URI上的访问者**，例子如下：
```

public class PersonContentProvider extends ContentProvider {
   public Uri insert(Uri uri, ContentValues values) {
      db.insert("person", "personid", values);
      getContext().getContentResolver().notifyChange(uri, null);
   }
}

```
如果ContentProvider的访问者需要得到数据变化通知**，必须使用ContentObserver对数据（数据采用uri描述）进行监听，当监听到数据变化通知时，系统就会调用ContentObserver的onChange()方法：**
```

getContentResolver().registerContentObserver(Uri.parse("content://com.ljq.providers.personprovider/person"),
       true, new PersonObserver(new Handler()));
public class PersonObserver extends ContentObserver{
   public PersonObserver(Handler handler) {
      super(handler);
   }
   public void onChange(boolean selfChange) {
      //此处可以进行相应的业务处理
   }
}

```
##  引用MIME类型 
Content provider 能返回标准的MIME media类型，也能返回自定义的MIME类型的字符窜。 MIME类型的格式如下：type/subtype
自定义的MIME类型的字符窜，也被叫做"vendor-specific"MIME类型，还有更多复杂的类型和子类型。这种类型一般是下面这样的
vnd.android.cursor.dir
来返回多行记录，或者下面的
vnd.android.cursor.item
来返回单个记录。 其子类型是特定的provider。Android 内置的provider 通常有一个简单的子类型。例如，当“联系人”应用程序创建一个电话号码时，它会在该行设置如下的MIME类型：
vnd.android.cursor.item/phone_v2
大部分content provider 为它们使用的MIME类型定义了contract 类常量。例如，Contacts Provider 中的contract 类 ` ContactsContract `.RawContacts，就为一个单一的联系人的MIME类型定义了CONTENT_ITEM_TYPE 常量。 单行数据的content URI在Content URIs 部分做了介绍。

##  Contacts Provider 
三个表通常被称为由其协议类的名称。类定义常量为内容的URI、列名和列值为表所用
ContactsContract.Contacts表
 Contacts表包含了不同的联系人的记录
ContactsContract.RawContacts表
 RawContacts表是联系人的数据集合，指定用户账号和类型
ContactsContract.Data表
 Data表是存储具体的联系人信息，包括邮件、电话号码等

##  示例 
PersonContentProvider内容提供者类：
```

package com.ljq.db;

import android.content.ContentProvider;
import android.content.ContentUris;
import android.content.ContentValues;
import android.content.UriMatcher;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.net.Uri;
import android.text.TextUtils;

/**
 * 内容提供者
 * 
 * 作用：统一数据访问方式，方便外部调用
 * 
 * @author jiqinlin
 * 
 */
public class PersonContentProvider extends ContentProvider {
    // 数据集的MIME类型字符串则应该以vnd.android.cursor.dir/开头
    public static final String PERSONS_TYPE = "vnd.android.cursor.dir/person";
    // 单一数据的MIME类型字符串应该以vnd.android.cursor.item/开头
    public static final String PERSONS_ITEM_TYPE = "vnd.android.cursor.item/person";
    public static final String AUTHORITY = "com.ljq.provider.personprovider";// 主机名
    /* 自定义匹配码 */
    public static final int PERSONS = 1;
    /* 自定义匹配码 */
    public static final int PERSON = 2;
    public static final Uri PERSONS_URI = Uri.parse("content://" + AUTHORITY + "/person");
    private DBOpenHelper dbOpenHelper = null;

    // UriMatcher类用来匹配Uri，使用match()方法匹配路径时返回匹配码
    private static final UriMatcher uriMatcher;
    static {
        // 常量UriMatcher.NO_MATCH表示不匹配任何路径的返回码
        uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        // 如果match()方法匹配content://com.ljq.provider.personprovider/person路径，返回匹配码为PERSONS
        uriMatcher.addURI(AUTHORITY, "person", PERSONS);
        // 如果match()方法匹配content://com.ljq.provider.personprovider/person/230路径，返回匹配码为PERSON
        uriMatcher.addURI(AUTHORITY, "person/#", PERSON);
    }
    
    @Override
    public boolean onCreate() {
        dbOpenHelper = new DBOpenHelper(this.getContext());
        return true;
    }
    
    @Override
    public Uri insert(Uri uri, ContentValues values){
        SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
        long id = 0;
        switch (uriMatcher.match(uri)) {
        case PERSONS:
            id = db.insert("person", "name", values);// 返回的是记录的行号，主键为int，实际上就是主键值
            return ContentUris.withAppendedId(uri, id);
        case PERSON:
            id = db.insert("person", "name", values);
            String path = uri.toString();
            return Uri.parse(path.substring(0, path.lastIndexOf("/"))+id); // 替换掉id
        default:
            throw new IllegalArgumentException("Unknown URI " + uri);
        }
    }
    
    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
        int count = 0;
        switch (uriMatcher.match(uri)) {
        case PERSONS:
            count = db.delete("person", selection, selectionArgs);
            break;
        case PERSON:
            // 下面的方法用于从URI中解析出id，对这样的路径content://com.ljq.provider.personprovider/person/10
            // 进行解析，返回值为10
            long personid = ContentUris.parseId(uri);
            String where = "id=" + personid;// 删除指定id的记录
            where += !TextUtils.isEmpty(selection) ? " and (" + selection + ")" : "";// 把其它条件附加上
            count = db.delete("person", where, selectionArgs);
            break;
        default:
            throw new IllegalArgumentException("Unknown URI " + uri);
        }
        db.close();
        return count;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection,
            String[] selectionArgs) {
        SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
        int count = 0;
        switch (uriMatcher.match(uri)) {
        case PERSONS:
            count = db.update("person", values, selection, selectionArgs);
            break;
        case PERSON:
            // 下面的方法用于从URI中解析出id，对这样的路径content://com.ljq.provider.personprovider/person/10
            // 进行解析，返回值为10
            long personid = ContentUris.parseId(uri);
            String where = "id=" + personid;// 获取指定id的记录
            where += !TextUtils.isEmpty(selection) ? " and (" + selection + ")" : "";// 把其它条件附加上
            count = db.update("person", values, where, selectionArgs);
            break;
        default:
            throw new IllegalArgumentException("Unknown URI " + uri);
        }
        db.close();
        return count;
    }
    
    @Override
    public String getType(Uri uri) {
        switch (uriMatcher.match(uri)) {
        case PERSONS:
            return PERSONS_TYPE;
        case PERSON:
            return PERSONS_ITEM_TYPE;
        default:
            throw new IllegalArgumentException("Unknown URI " + uri);
        }
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
            String[] selectionArgs, String sortOrder) {
        SQLiteDatabase db = dbOpenHelper.getReadableDatabase();

        switch (uriMatcher.match(uri)) {
        case PERSONS:
            return db.query("person", projection, selection, selectionArgs, null, null, sortOrder);
        case PERSON:
            // 下面的方法用于从URI中解析出id，对这样的路径content://com.ljq.provider.personprovider/person/10
            // 进行解析，返回值为10
            long personid = ContentUris.parseId(uri);
            String where = "id=" + personid;// 获取指定id的记录
            where += !TextUtils.isEmpty(selection) ? " and (" + selection + ")" : "";// 把其它条件附加上
            return db.query("person", projection, where, selectionArgs, null, null, sortOrder);
        default:
            throw new IllegalArgumentException("Unknown URI " + uri);
        }
    }


}

```
```

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.ljq.sql" android:versionCode="1"
    android:versionName="1.0">
    <application android:icon="@drawable/icon"
        android:label="@string/app_name">
        <uses-library android:name="android.test.runner" />
        <activity android:name=".SqlActivity"
            android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category
                    android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <provider android:name="com.ljq.db.PersonContentProvider" 
              android:authorities="com.ljq.provider.personprovider" />
    </application>
    <uses-sdk android:minSdkVersion="7" />
    <instrumentation
        android:name="android.test.InstrumentationTestRunner"
        android:targetPackage="com.ljq.sql" android:label="Tests for My App" />
</manifest>

```
##  示例2：ContentProvider往通讯录添加联系人和获取联系人 
在Android中，可以使用ContentResolver对通信录中的数据进行添加、删除、修改和查询操作。

在对联系人进行操作时需加入以下两个权限  

''<!-- 添加操作联系人的权限 -->
<uses-permission android:name="android.permission.READ_CONTACTS" />
<uses-permission android:name="android.permission.WRITE_CONTACTS" />''
```

<!-- 联系人相关的uri -->
content://com.android.contacts/contacts 操作的数据是联系人信息Uri
content://com.android.contacts/data/phones 联系人电话Uri
content://com.android.contacts/data/emails 联系人Email Uri  

``` 
```

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.ljq.contact" android:versionCode="1"
    android:versionName="1.0">
    <application android:icon="@drawable/icon"
        android:label="@string/app_name">
        <uses-library android:name="android.test.runner" />
        <activity android:name=".ContactActivity"
            android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category
                    android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>
    <uses-sdk android:minSdkVersion="7" />
    <instrumentation
        android:name="android.test.InstrumentationTestRunner"
        android:targetPackage="com.ljq.contact"
        android:label="Tests for My App" />
    <!-- 添加联系人权限 -->
    <uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.WRITE_CONTACTS" />

</manifest>

```
获取联系人和添加联系人
```

package com.ljq.contact;

import java.util.ArrayList;

import android.content.ContentProviderOperation;
import android.content.ContentProviderResult;
import android.content.ContentResolver;
import android.content.ContentUris;
import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.provider.ContactsContract;
import android.provider.ContactsContract.Data;
import android.provider.ContactsContract.RawContacts;
import android.provider.ContactsContract.CommonDataKinds.Email;
import android.provider.ContactsContract.CommonDataKinds.Phone;
import android.provider.ContactsContract.CommonDataKinds.StructuredName;
import android.test.AndroidTestCase;
import android.util.Log;

public class ContactTest extends AndroidTestCase{
    private static final String TAG = "ContactTest";
    
    /**
     * 获取通讯录中联系人
     */
    public void testGetContact(){
        ContentResolver contentResolver = this.getContext().getContentResolver();
        Uri uri = Uri.parse("content://com.android.contacts/contacts");
        Cursor cursor = contentResolver.query(uri, null, null, null, null);
        while(cursor.moveToNext()){
            // 获取联系人姓名
            StringBuilder sb = new StringBuilder();
            String contactId = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts._ID));  
            String name = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME)); 
            sb.append("contactId=").append(contactId).append(",name=").append(name);
            
            //获取联系人手机号码
            Cursor phones = contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,  
                    null,  
                    ContactsContract.CommonDataKinds.Phone.CONTACT_ID +" = "+ contactId,  
                    null, null);  
            while(phones.moveToNext()){
                String phone = phones.getString(phones.getColumnIndex("data1")); 
                sb.append(",phone=").append(phone);
            }
            
            //获取联系人email
            Cursor emails = contentResolver.query(ContactsContract.CommonDataKinds.Email.CONTENT_URI,  
                       null,  
                       ContactsContract.CommonDataKinds.Email.CONTACT_ID + " = " + contactId,  
                       null, null); 
            while(emails.moveToNext()){
                String email = emails.getString(emails.getColumnIndex("data1")); 
                sb.append(",email=").append(email);
            }
            Log.i(TAG, sb.toString());
        }
    }

    /**
     * 首先向RawContacts.CONTENT_URI执行一个空值插入，目的是获取系统返回的rawContactId 
     * 
     * 这是后面插入data表的数据，只有执行空值插入，才能使插入的联系人在通讯录里可见
     */
    public void testInsert(){
        ContentValues values = new ContentValues();
        //首先向RawContacts.CONTENT_URI执行一个空值插入，目的是获取系统返回的rawContactId
        Uri rawContactUri = this.getContext().getContentResolver().insert(RawContacts.CONTENT_URI, values);
        long rawContactId = ContentUris.parseId(rawContactUri);
        
        //往data表入姓名数据
        values.clear();
        values.put(Data.RAW_CONTACT_ID, rawContactId);
        values.put(Data.MIMETYPE, StructuredName.CONTENT_ITEM_TYPE);
        values.put(StructuredName.GIVEN_NAME, "zhangsan");
        this.getContext().getContentResolver().insert(
                android.provider.ContactsContract.Data.CONTENT_URI, values);
        
        //往data表入电话数据
        values.clear();
        values.put(android.provider.ContactsContract.Contacts.Data.RAW_CONTACT_ID, rawContactId);
        values.put(Data.MIMETYPE, Phone.CONTENT_ITEM_TYPE);
        values.put(Phone.NUMBER, "5554");
        values.put(Phone.TYPE, Phone.TYPE_MOBILE);
        this.getContext().getContentResolver().insert(
                android.provider.ContactsContract.Data.CONTENT_URI, values);

        //往data表入Email数据
        values.clear();
        values.put(android.provider.ContactsContract.Contacts.Data.RAW_CONTACT_ID, rawContactId);
        values.put(Data.MIMETYPE, Email.CONTENT_ITEM_TYPE);
        values.put(Email.DATA, "ljq218@126.com");
        values.put(Email.TYPE, Email.TYPE_WORK);
        this.getContext().getContentResolver().insert(
                android.provider.ContactsContract.Data.CONTENT_URI, values);
    }
    
    /**
     * 批量添加联系人，处于同一个事务中
     */
    public void testSave() throws Throwable{
        //文档位置：reference\android\provider\ContactsContract.RawContacts.html
        ArrayList<ContentProviderOperation> ops = new ArrayList<ContentProviderOperation>();
        int rawContactInsertIndex = 0;
        ops.add(ContentProviderOperation.newInsert(RawContacts.CONTENT_URI)
                .withValue(RawContacts.ACCOUNT_TYPE, null)
                .withValue(RawContacts.ACCOUNT_NAME, null)
                .build());
        
        //文档位置：reference\android\provider\ContactsContract.Data.html
        ops.add(ContentProviderOperation.newInsert(android.provider.ContactsContract.Data.CONTENT_URI)
                .withValueBackReference(Data.RAW_CONTACT_ID, rawContactInsertIndex)
                .withValue(Data.MIMETYPE, StructuredName.CONTENT_ITEM_TYPE)
                .withValue(StructuredName.GIVEN_NAME, "lisi")
                .build());
        ops.add(ContentProviderOperation.newInsert(android.provider.ContactsContract.Data.CONTENT_URI)
                 .withValueBackReference(Data.RAW_CONTACT_ID, rawContactInsertIndex)
                 .withValue(Data.MIMETYPE, Phone.CONTENT_ITEM_TYPE)
                 .withValue(Phone.NUMBER, "5556")
                 .withValue(Phone.TYPE, Phone.TYPE_MOBILE)
                 .withValue(Phone.LABEL, "")
                 .build());
        ops.add(ContentProviderOperation.newInsert(android.provider.ContactsContract.Data.CONTENT_URI)
                 .withValueBackReference(Data.RAW_CONTACT_ID, rawContactInsertIndex)
                 .withValue(Data.MIMETYPE, Email.CONTENT_ITEM_TYPE)
                 .withValue(Email.DATA, "lisi@126.cn")
                 .withValue(Email.TYPE, Email.TYPE_WORK)
                 .build());
        
        ContentProviderResult[] results = this.getContext()
                .getContentResolver().applyBatch(ContactsContract.AUTHORITY,ops);
        for (ContentProviderResult result : results) {
            Log.i(TAG, result.uri.toString());
        }
    }
    
}

```
参考：
http://www.cnblogs.com/linjiqin/archive/2011/05/28/2061396.html