title: mongodb入门 

#  MongoDB入门 
**What is MongoDB**
**MongoDB 是一个面向文档的NoSQL数据库系统。**使用C++编写，不支持SQL，但**有自己功能强大的查询语法。MongoDB使用BSON作为数据存储和传输的格式**。` BSON `是一种类似JSON的二进制序列化文档，支持嵌套对象和数组。MongoDB很像MySQL，document对应MySQL的row，collection对应MySQL的table。
 
MongoDB在Windows上的安装运行很方便。直接下载、解压，然后运行 bin/mongod 即可启动服务器，运行 bin/mongo 即可运行命令行客户端。更多关于MongoDB的运行看这里 。MongoDB命令行客户端的脚本语法有些类似MySQL的：
```

show dbs // 列出所有数据库  
use memo // 使用数据库memo。即使这个数据库不存在也可以执行，但该数据库不会立刻被新建，要等到执行了insert之类的操作时，才会建立这个数据库  
show collections // 列出当前数据库的collections  
db // 显示当前数据库  
show users // 列出用户  
 
MongoDB的查询语法很强大。例如，很多SQL可以做的，它都可以做：
coll.find() // select * from coll  
coll.find().limit(10) // select * from coll limit 10  
coll.find().sort({x:1}) // select * from coll order by x asc  
coll.find().sort({x:1}).skip(5).limit(10) // select * from coll order by x asc limit 5, 10  
coll.find({x:10}) // select * from coll where x = 10  
coll.find({x: {$lt:10}}) // select * from coll where x <= 10  
coll.find({}, {y:true}) // select y from coll  
 
一些SQL不能做的，MongoDB也可以做：
coll.find({"address.city":"gz"}) // 搜索嵌套文档address中city值为gz的记录  
coll.find({likes:"math"}) // 搜索数组  
coll.ensureIndex({"address.city":1}) // 在嵌套文档的字段上建索引 

``` 
 
```

索引：
coll.ensureIndex({productid:1}) // 在productid上建立普通索引  
coll.ensureIndex({district:1, plate:1}) // 多字段索引  
coll.ensureIndex({productid:1}, {unique:true}) // 唯一索引  
coll.ensureIndex({productid:1}, {unique:true, dropDups:true|) // 建索引时，如果遇到索引字段值已经出现过的情况，则删除重复记录  
coll.getIndexes() // 查看索引  
coll.dropIndex({productid:1}) // 删除单个索引  
 
安全与认证（该版本的MongoDB仅支持很基本的安全策略）：
use shine // 如果要root权限，就用admin库  
db.addUser("username", "password") // 普通权限，可读写  
db.addUser("username", "password", true)  // 只可读，不可写  
db.system.users.remove({user: username}) // 删除用户 

``` 
 
```

数据导出、导入：
// json或csv格式，每次一个collection  
mongoexport -d producttrade -c basic -o /home/data/mongo_backup/producttrade_100504.json  
mongoimport -d producttrade -c basic --drop /home/data/mongo_backup/producttrade_100504.json  
  
// 二进制数据格式，常用于备份、还原  
mongodump -d shine -o /home/data/mongo_backup  
mongorestore -d shine --drop /home/data/mongo_backup/shine 

``` 
 
##  MongoDB in Java 
到这里 下一个MongoDB的Java驱动，把jar包扔到项目里去就行了。上面提到的通过脚本操作的功能，基本上都能在Java中找到实现。进行数据库连接的代码也十分简洁：
```

Mongo mongo = new Mongo();  
db = mongo.getDB("shine");  
coll = db.getCollection("producttrade");  
DBCursor cur = coll.find();  
// 对cur进行操作。。。 

``` 
  * 每个BSON对象大小不能超过4MB。MongoDB使用GridFS 来储存大文件。
  * 字段名限制：不能以"$"开头；不能包含"."；**"_id"是系统保留的字段**，但用户可以自己储存唯一性的数据在字段中。
  * MongoDB为每个数据库分配一系列文件。每个数据文件都会被预分配一个大小，第一个文件名字为".0"，大小为64MB，第二个文件".1"为128MB
  * **MongoDB没有新建数据库或者collection的命令，只要进行insert或其它操作，MongoDB就会自动帮你建立数据库和collection。当查询一个不存在的collection时也不会出错，Mongo会认为那是一个空的collection。**
  * 一个对象被插入到数据库中时，**如果它没有ID，会自动生成一个"_id"字段，**为24位16进制数。
  * **Java中，` Mongo对象是线程安全的 `，一个应用中应该只使用一个Mongo对象。Mongo对象会自动维护一个连接池，默认连接数为10**。

##  主要概念 
**集合collection是一组MongoDB的文档。它相当于一个RDBMS表**。收集存在于一个单一的数据库。集合不执行模式。**集合(表)内的文档(行)可以有不同的领域(列)**。通常情况下，一个集合中的所有文件是相同或相关的目的。 
**文档document是一组键 - 值对。它相当于一个RDBMS行**文件动态模式。动态模式是指，在相同集合中的文档不需要具有相同的字段或结构组的公共字段的集合的文档，可以容纳不同类型的数据。

下面给出的表显示RDBMS术语使用 MongoDB 的关系
![](/data/dokuwiki/database/pasted/20151013-101729.png)

示列文档：
下面给出的示例显示了一个博客网站，这简直是一个逗号分隔的键值对文档结构。
```

{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'yiibai.com',
   url: 'http://www.yiibai.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100, 
   comments: [	
      {
         user:'user1',
         message: 'My first comment',
         dateCreated: new Date(2011,1,20,2,15),
         like: 0 
      },
      {
         user:'user2',
         message: 'My second comments',
         dateCreated: new Date(2011,1,25,7,45),
         like: 5
      }
   ]
} 

``` 
_id是一个12字节的十六进制数，保证每一份文件的唯一性。您可以提供_id同时插入文档。如果没有提供，那么MongoDB的每个文档提供了一个独特的ID。这12个字节，前4个字节为当前时间戳，未来3个字节的机器ID，接下来的2个字节的进程id MongoDB的服务器及剩余3个字节是简单的增量值。

##  MongoDB语法 
**创建数据库**
MongoDB`  use DATABASE_NAME ` 用于创建数据库。该命令如果数据库不存在，将创建一个新的数据库， 否则将返回现有的数据库。

```

>show dbs
local     0.78125GB
test      0.23012GB

```
所创建的数据库（mydb）不存在于列表中。要显示的数据库，需要至少插入一个文档进去。

```

>use mydb
>db.movie.insert({"name":"yiibai tutorials"})
>show dbs
local      0.78125GB
mydb       0.23012GB
test       0.23012GB

```
MongoDB的默认数据库是test。 如果没有创建任何数据库，那么集合将被保存在测试数据库。

**删除数据库**
` db.dropDatabase() ` 命令用于删除现有的数据库。
这将删除选定的数据库。如果没有选择任何数据库，那么它会删除默认的“test”数据库
```

>use mydb
switched to db mydb
>db.dropDatabase()
>{ "dropped" : "mydb", "ok" : 1 }

```

**创建集合**
MongoDB 的 ` db.createCollection(name, options) ` 用于创建集合。 在命令中, **name 是要创建集合的名称。 Options 是一个文档，用于指定集合的配置**
选项参数是可选的，所以需要指定集合的唯一名字。

```

>use test
switched to db test
>db.createCollection("mycollection")
{ "ok" : 1 }
>

```
可以通过使用 ` show collections ` 命令来检查创建的集合

```

>show collections
mycollection
system.indexes

```
**第二个参数：选项列表**
字段	类型	描述
capped	Boolean	（可选）如果为true，它启用上限集合。上限集合是一个固定大小的集合，当它达到其最大尺寸会自动覆盖最老的条目。 如果指定true，则还需要指定参数的大小。
autoIndexID	Boolean	（可选）如果为true，自动创建索引_id字段。默认的值是 false.
size	number	（可选）指定的上限集合字节的最大尺寸。如果capped 是true，那么还需要指定这个字段。
max	number	（可选）指定上限集合允许的最大文件数。
尽管插入文档，MongoDB首先检查字段集合的上限大小，那么它会检查最大字段。

语法 :
```

>db.createCollection("mycol", { capped : true, autoIndexID : true, size : 6142800, max : 10000 } )
{ "ok" : 1 }
>

```
**在MongoDB中并不需要创建集合。 当插入一些文档 MongoDB 会自动创建集合。**

```

>db.yiibai.insert({"name" : "yiibai"})
>show collections
mycol
mycollection
system.indexes
yiibai
>

```
**删除集合**

MongoDB 的 db.collection.drop() 用于从数据库中删除集合。
` db.COLLECTION_NAME.drop() `
例子：
下面给出的例子将删除给定名称的集合：mycollection
```

>use mydb
switched to db mydb
>db.mycollection.drop()
true

```

**插入文档**
将数据插入到MongoDB集合，需要使用MongoDB 的 ` insert() ` 方法。
语法
insert()命令的基本语法如下：
` db.COLLECTION_NAME.insert(document) `
例子
```

>db.mycol.insert({
   _id: ObjectId(7df78ad8902c),
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'yiibai tutorials',
   url: 'http://www.yiibai.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
})

```
这里 mycol 是我们的集合名称，它是在之前的教程中创建。**如果集合不存在于数据库中，那么MongoDB创建此集合，然后插入文档进去。**
**在如果我们不指定_id参数插入的文档，那么 MongoDB 将为文档分配一个唯一的ObjectId。**
_id 是12个字节十六进制数在一个集合的每个文档是唯一的。 12个字节被划分如下：
_id: ObjectId(4 bytes timestamp, 3 bytes machine id, 2 bytes process id, 3 bytes incrementer)
要以单个查询**插入多个文档，可以通过文档 insert() 命令的数组方式。**
```

>db.post.insert([
{
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'yiibai tutorials',
   url: 'http://www.yiibai.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   title: 'NoSQL Database', 
   description: 'NoSQL database doesn't have tables',
   by: 'yiibai tutorials',
   url: 'http://www.yiibai.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 20, 
   comments: [	
      {
         user:'user1',
         message: 'My first comment',
         dateCreated: new Date(2013,11,10,2,35),
         like: 0 
      }
   ]
}
])

```
**查询文档**
要从集合查询MongoDB数据，需要使用MongoDB的 ` find() `方法。
` db.COLLECTION_NAME.find() `
**find() 方法将在非结构化的方式显示所有的文件。** 如果显示结果是**格式化**的，那么可以用` pretty() ` 方法。
` db.mycol.find().pretty() `
例子
```

>db.mycol.find().pretty()
{
   "_id": ObjectId(7df78ad8902c),
   "title": "MongoDB Overview", 
   "description": "MongoDB is no sql database",
   "by": "yiibai tutorials",
   "url": "http://www.yiibai.com",
   "tags": ["mongodb", "database", "NoSQL"],
   "likes": "100"
}
>

```
**除了find()方法还有` findOne() `方法**，仅返回一个文档。

**RDBMS Where子句与MongoDB等效语法对比**
![](/data/dokuwiki/database/pasted/20151013-103208.png)

**AND 在 MongoDB 语法**
在 find()方法，如果您传递多个键通过","将它们分开，那么MongoDB对待它就如AND条件一样。基本语法如下所示：
` db.mycol.find({key1:value1, key2:value2}).pretty() `

下面给出的例子将显示所有教程含“yiibai tutorials”和其标题是“MongoDB Overview”
```

>db.mycol.find({"by":"yiibai tutorials","title": "MongoDB Overview"}).pretty()
{
   "_id": ObjectId(7df78ad8902c),
   "title": "MongoDB Overview", 
   "description": "MongoDB is no sql database",
   "by": "yiibai tutorials",
   "url": "http://www.yiibai.com",
   "tags": ["mongodb", "database", "NoSQL"],
   "likes": "100"
}
>

```

**OR 在 MongoDB 语法**
要查询基于OR条件的文件，需要使用` $or关键字 `。OR的基本语法如下所示：
```

>db.mycol.find(
   {
      $or: [
	     {key1: value1}, {key2:value2}
      ]
   }
).pretty()

```
下面给出的例子将显示所有撰写含有 'yiibai tutorials' 或是标题为 'MongoDB Overview' 的教程
```

>db.mycol.find({$or:[{"by":"tutorials point"},{"title": "MongoDB Overview"}]}).pretty()
{
   "_id": ObjectId(7df78ad8902c),
   "title": "MongoDB Overview", 
   "description": "MongoDB is no sql database",
   "by": "yiibai tutorials",
   "url": "http://www.yiibai.com",
   "tags": ["mongodb", "database", "NoSQL"],
   "likes": "100"
}
>

```

**使用 AND 和 OR 在一起 例子**
下面给出的例子显示有喜欢数大于100 的文档，其标题要么是 'MongoDB Overview' 或 'yiibai tutorials'. 等效于SQL的where子句：'where likes>10 AND (by = 'yiibai tutorials' OR title = 'MongoDB Overview')'
```

>db.mycol.find("likes": {$gt:10}, $or: [{"by": "yiibai tutorials"}, {"title": "MongoDB Overview"}] }).pretty()
{
   "_id": ObjectId(7df78ad8902c),
   "title": "MongoDB Overview", 
   "description": "MongoDB is no sql database",
   "by": "yiibai tutorials",
   "url": "http://www.yiibai.com",
   "tags": ["mongodb", "database", "NoSQL"],
   "likes": "100"
}
>

```

**更新文档**
MongoDB的` update()和save() `方法用于更新文档到一个集合。 update()方法将现有的文档中的值更新，而save()方法使用传递到save()方法的文档替换现有的文档。
update()方法的基本语法如下
` db.COLLECTION_NAME.update(SELECTIOIN_CRITERIA, UPDATED_DATA) `
考虑mycol集合有如下数据。
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Yiibai Overview"}
下面的例子将设置其标题“MongoDB Overview”的文件为新标题为“New MongoDB Tutorial”
**db.mycol.update({'title':'MongoDB Overview'},{` $set `:{'title':'New MongoDB Tutorial'}})**
db.mycol.find()
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"New MongoDB Tutorial"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Tutorial Overview"}

默认情况下，MongoDB将只更新单一文件，**更新多，需要一个参数 'multi' 设置为 true。**
**db.mycol.update({'title':'MongoDB Overview'},{` $set `:{'title':'New MongoDB Tutorial'}},` {multi:true} `)**

**MongoDB Save() 方法**
` db.COLLECTION_NAME.save({_id:ObjectId(),NEW_DATA}) `
例子
下面的例子将替换该文件_id '5983548781331adf45ec7'
```

>db.mycol.save(
   {
      "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Yiibai New Topic", "by":"Yiibai Yiibai"
   }
)
>db.mycol.find()
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"Yiibai Yiibai New Topic", "by":"Yiibai Yiibai"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Yiibai Overview"}
>

```
  
**删除文档**
MongoDB 的`  remove() `方法用于从集合中删除文档。**remove()方法接受两个参数。一个是标准缺失，第二是justOne标志**
  * deletion criteria : 根据文件（可选）删除条件将被删除。
  * justOne : （可选）如果设置为true或1，然后取出只有一个文档。
` db.COLLECTION_NAME.remove(DELLETION_CRITTERIA) `
考虑mycol集合有如下数据。
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Yiibai Overview"}
下面的例子将删除所有的文件，其标题为 'MongoDB Overview'
db.mycol.remove({'title':'MongoDB Overview'})
db.mycol.find()
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Toturials Overview"}

**只删除一个**
如果有多个记录，并要删除仅第一条记录，然后在** remove()方法设置参数 justOne 。**
` db.COLLECTION_NAME.remove(DELETION_CRITERIA,1) `
**删除所有文件**
如果没有指定删除条件，则MongoDB将从集合中删除整个文件。这相当于SQL的 truncate 命令。
db.mycol.remove()
db.mycol.find()

##  MongoDB投影 
mongodb投影意义是**只选择需要的数据，而不是选择整个一个文档的数据**。如果一个文档有5个字段，只需要显示3个，只从中选择3个字段。
MongoDB的` find()方法 `，解释了MongoDB中查询文档接收的**第二个可选的参数是要检索的字段列表**。在MongoDB中，**当执行find()方法，那么它会显示一个文档的所有字段。要限制这一点，需要设置字段列表值为1或0。1是用来显示字段，而0被用来隐藏字段。**
` db.COLLECTION_NAME.find({},{KEY:1}) `
考虑集合 myycol 有下列数据
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Yiibai Overview"}
下面的例子将显示文档的标题，在查询文档时。
```

>db.mycol.find({},{"title":1,_id:0})
{"title":"MongoDB Overview"}
{"title":"NoSQL Overview"}
{"title":"Yiibai Yiibai Overview"}
>

```
请注意在执行find()方法时_id字段始终显示，**如果不想要显示这个字段，那么需要将其设置为0**

##  限制文档 
**MongoDB Limit() 方法**
要在MongoDB中限制记录，需要使用` limit() `方法。 limit() 方法接受一个数字类型的参数，这是要显示的文档数量。
` db.COLLECTION_NAME.find().limit(NUMBER) `
考虑集合 myycol 有下列数据
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Yiibai Overview"}
下面的例子将只显示2个文档，在查询文档时。
```

>db.mycol.find({},{"title":1,_id:0}).limit(2)
{"title":"MongoDB Overview"}
{"title":"NoSQL Overview"}
>

```
如果不指定 limit()方法的参数数量，然后它会显示集合中的所有文档

**MongoDB Skip() 方法**
除了 limit()方法还有一个方法 ` skip() `也接受数字类型参数并用于跳过文件数。
` db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER) `
下面的例子将仅显示第二个文档。
```

>db.mycol.find({},{"title":1,_id:0}).limit(1).skip(1)
{"title":"NoSQL Overview"}
>

```
请注意，skip() 方法的默认值是 0

**文档排序**
要排序MongoDB中的文档，需要使用`  sort() `方法。 sort() 方法接受一个**包含字段列表以及排序顺序的文档。 要使用1和-1指定排序顺序。1用于升序，而-1是用于降序。**
` db.COLLECTION_NAME.find().sort({KEY:1}) `
考虑集合 myycol 有如下数据
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Yiibai Overview"}
下面的例子将显示的文件排序按标题降序排序。
```

>db.mycol.find({},{"title":1,_id:0}).sort({"title":-1})
{"title":"Yiibai Yiibai Overview"}
{"title":"NoSQL Overview"}
{"title":"MongoDB Overview"}
>

```
请注意，如果不指定排序类型，那么 sort() 方法将以升序排列文档。

##  MongoDB 聚合 
聚合操作处理数据记录并返回计算结果。从多个文档聚合分组操作数值，并可以执行多种对分组数据业务返回一个结果。 在SQL中的count(*)，使用group by 与mongodb的聚合是等效的。
对于MongoDB的聚合，使用的是` aggregate() `方法。
` db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION) `
在集合中有以下数据：
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by_user: 'Yiibai Yiibai ',
   url: 'http://www.yiibai.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   _id: ObjectId(7df78ad8902d)
   title: 'NoSQL Overview', 
   description: 'No sql database is very fast',
   by_user: 'Yiibai Yiibai',
   url: 'http://www.yiibai.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 10
},
{
   _id: ObjectId(7df78ad8902e)
   title: 'Neo4j Overview', 
   description: 'Neo4j is no sql database',
   by_user: 'Neo4j',
   url: 'http://www.neo4j.com',
   tags: ['neo4j', 'database', 'NoSQL'],
   likes: 750
},
现在从上面的集合，如果想知道每一个用户编写的教程是多少，那么使用aggregate()方法，如下图所示的列表：
db.mycol.aggregate(` [ `{` $group ` : {_id : "` $by_user `", num_tutorial : {` $sum ` : 1}}}])
{
   "result" : [
      {
         "_id" : "Yiibai Yiibai",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
用于上述用途将等效于sql查询： select by_user, count(*) from mycol group by by_user
另外，在上述例子中，我们已经使用字段by_user进行分组并计算总和，也就是by_user 出现各个次数。一个列表中可用的聚集表达式。
![](/data/dokuwiki/database/pasted/20151013-105557.png)


##  MongoDB索引 
索引支持查询高效率执行。如果没有索引，MongoDB必须扫描集合中的每一个文档，然后选择那些符合查询语句的文档。若需要 mongod 来处理大量数据，扫描是非常低效的。
索引是特殊的数据结构，存储在一个易于设置遍历形式的数据的一小部分。索引存储在索引中指定特定字段的值或一组字段，并排序字段的值。
要创建索引，需要使用MongoDB的` ensureIndex() `方法。
ensureIndex()方法的基本语法如下
` db.COLLECTION_NAME.ensureIndex({KEY:1}) `
这里键是要创建索引字段，1是按名称升序排序。若以按降序创建索引，需要使用 -1.
```

>db.mycol.ensureIndex({"title":1})
>
在 ensureIndex()方法，可以通过多个字段，来创建多个字段索引。

>db.mycol.ensureIndex({"title":1,"description":-1})
>

```
ensureIndex() 方法还接受选项列表（这是可选），其列表如下：
![](/data/dokuwiki/database/pasted/20151013-105201.png)

##  MongoDB 复制 
复制是同步在多个服务器上的数据过程。复制提供了冗余和数据在不同的数据库服务器上的多个副本提高数据的可用性，复制防止在单个服务器上丢失数据库。 复制也可以从硬件故障和服务中断中恢复。带有数据的其他副本，可以选择其中一个灾难恢复，报告或备份。

为什么要复制？
  * 为了让数据安全
  * 数据的高（24*7）可用性
  * 灾难恢复
  * 无停机维护（如备份，索引重建，压缩）
  * 读取缩放（额外的副本来读取）
  * 副本集是透明的应用
MongoDB复制的工作原理
MongoDB通过使用**副本集的复制**来实现。副本集是一组承载同一个数据集的mongod实例。**在副本的一个节点是接收所有的写操作主节点**。所有的实例，次级，应用操作` 从主 `以便它们具有相同的数据集。**副本集只能有一个主节点。**
  * 副本集是一组两个或更多个节点（**通常至少3节点是必需的**）。
  * 在副本集一个节点是主节点和其余的节点都是次要的。
  * **所有的数据复制是从主到次节点。**
  * 在自动故障转移或维护时，选建立了主要和一个新的主节点被选择。
  * 故障节点的恢复后，再次加入副本集，并可以作为一个辅助节点。
mongodb复制的典型图如下图，其中客户端应用程序总是与主节点和主节点交互，然后将数据复制到辅助节点。
![](/data/dokuwiki/database/pasted/20151013-105935.png)
副本集特征
  * N个节点的集群
  * 任何节点可为原发/主节点
  * 所有的写操作进入到主节点
  * 自动故障转移
  * 自动恢复
  * 协商一致选择主节点

建立一个副本集
在本教程中，我们将独立的 mongod 实例转换为副本集。 要转换为副本集，按照以下的步骤：
关闭已经运行的 MongoDB 服务器。
```

现在，通过指定--replSet选项启动 MongoDB 服务器。--replSet 的基本语法如下：

mongod --port "PORT" --dbpath "YOUR_DB_DATA_PATH" --replSet "REPLICA_SET_INSTANCE_NAME"
例子
mongod --port 27017 --dbpath "D:\software\MongoDB\Server\3.0\mongodb\data" --replSet rs0
这将启动一个名为rs0的一个mongod实例，端口为： 27017

```
现在打开启动命令提示符，然后连接到mongod实例
在Mongo的客户端使用命令` rs.initiate() `来启动一个新的副本集
要检查副本设置配置，则使用命令` rs.conf() `
要检查副本集发行的状态，使用命令` rs.status() `

##  MongoDB数据备份与恢复 
**MongoDB数据转储**
要使用 ` mongodump ` 命令来执行 MongoDB 数据库备份。此命令将转储服务器的所有数据到转储目录。有许多可用的选项，通过它可以限制数据量或创建远程服务器备份。
mongodump
启动 mongod 服务器。假设 mongod 服务器运行在本地主机和端口 ` 27017 `. 现在打开一个命令提示符，然后转到你的MongoDB实例的bin目录，然后输入命令mongodump。
考虑mycol集合有以下数据。
mongodump
```

mongodump --host HOST_NAME --port PORT_NUMBER	这个命令将备份指定的mongod实例的所有数据库	mongodump --host yiibai.com --port 27017
mongodump --dbpath DB_PATH --out BACKUP_DIRECTORY	 	mongodump --dbpath /data/db/ --out /data/backup/
mongodump --collection COLLECTION --db DB_NAME	此命令将仅备份指定的特定数据库集合	mongodump --collection mycol --db test

```
要恢复备份的MongoDB数据，则使用` mongorestore `命令。该命令将从备份目录恢复所有的数据。
mongorestore命令的基本语法
mongorestore
##  MongoDB 数据模型 
在 MongoDB 中的数据有灵活的模式。在相同集合中文档并不需要有相同的一组字段或结构的公共字段的集合，文档可容纳不同类型的数据。
假设一个客户端需要一个数据库设计，设计一个博客网站，来看看 RDBMS 和 MongoDB 架构设计之间的差异。网站有以下要求。
每一个文章内容都有独特的标题，描述和网址。
每一个文章内容可以有一个或多个标签。
每一个文章内容都有其出版商总数喜欢的名称。
每一个文章内容有评论以及名字，消息，时间和喜欢的用户。
对于每个文章，可以是零个或多个评论。
上述要求在RDBMS模式设计，将有至少三个表。
![](/data/dokuwiki/database/pasted/20151026-112207.png)
在MongoDB 模式设计就文章一个集合，并具有以下结构：
```

{
   _id: POST_ID
   title: TITLE_OF_POST, 
   description: POST_DESCRIPTION,
   by: POST_BY,
   url: URL_OF_POST,
   tags: [TAG1, TAG2, TAG3],
   likes: TOTAL_LIKES, 
   comments: [	
      {
         user:'COMMENT_BY',
         message: TEXT,
         dateCreated: DATE_TIME,
         like: LIKES 
      },
      {
         user:'COMMENT_BY',
         message: TEXT,
         dateCreated: DATE_TIME,
         like: LIKES
      }
   ]
}

```
因此，尽管RDBMS要显示数据，需要加入三个表，而在MongoDB数据只是从一个集合。
##  MongoDB 数据类型 
MongoDB支持许多数据类型的列表下面给出：
  * String : 这是最常用的数据类型来存储数据。在MongoDB中的字符串必须是有效的` UTF-8 `。
  * Integer : 这种类型是用来存储一个数值。整数可以是32位或64位，这取决于您的服务器。
  * Boolean : 此类型用于存储一个布尔值 (true/ false) 。
  * Double : 这种类型是用来存储浮点值。
  * Min/ Max keys : 这种类型被用来对BSON元素的最低和最高值比较。
  * Arrays : 使用此类型的数组或列表或多个值存储到一个键。
  * Timestamp : 时间戳。这可以方便记录时的文件已被修改或添加。
  * Object : 此数据类型用于嵌入式的文件。
  * Null : 这种类型是用来存储一个` Null值 `。
  * Symbol : 此数据类型用于字符串相同，但它通常是保留给特定符号类型的语言使用。
  * Date : 此数据类型用于存储当前日期或时间的UNIX时间格式。可以指定自己的日期和时间，日期和年，月，日到创建对象。
  * Object ID : 此数据类型用于存储文档的ID。
  * Binary data : 此数据类型用于存储` 二进制数据 `。
  * Code : 此数据类型用于存储到文档中的JavaScript代码。
  * Regular expression : 此数据类型用于存储正则表达式


参考：http://renial.iteye.com/blog/684829
http://www.yiibai.com/mongodb/
http://www.yiibai.com/mongodb/mongodb_quick_guide.html