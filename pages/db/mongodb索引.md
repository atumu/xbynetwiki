createAt:2017-01-15 12:14:55
author:xbynet
modifyAt:2017-01-15 12:34:30
location:db/mongodb索引
title:MongoDB权威指南2e学习三索引

不使用索引的查询称为**全表查询**。可以使用`explain()`函数查看MongoDB在执行查询的过程中所做的事情。
```
> db.book.find().explain()
{
    "cursor" : "BasicCursor",//BasicCursor后面所接表示所有索引，本次没有
    "isMultiKey" : false,//用于说明本次查询是否用了多键索引
    "n" : 102,//返回文档数
    "nscannedObjects" : 102,//按照索引指针去磁盘上查找实际文档的次数
    "nscanned" : 102,//完成这个查询过程中扫描的文档总数
    "nscannedObjectsAllPlans" : 102,
    "nscannedAllPlans" : 102,
    "scanAndOrder" : false,//是否在内存中对结果集进行了排序
    "indexOnly" : false,//是否只使用索引就能完成此处查询
    "nYields" : 1,//为了让写入请求能够顺利指向，本次查询暂停的次数。如果有写入请求需要处理，查询会周期性地释放它们的锁，以便写入能够顺利执行。
    "nChunkSkips" : 0,
    "millis" : 62,//这个查询耗费的毫秒数
    "server" : "localhost:27017",
    "filterSet" : false
}
```

# 创建索引
```
>db.collection.ensureIndex({'key':1}) 
```
可有多个key，创建**复合索引 **
在实际的应用程序中，{"sortKey":1,"queryCriteria":1}索引通常是很有用的，因为大多数应用程序在一次查询中只需要得到查询结果最前面的少数结果，而不是所有可能的结果。而且，由于索引在内部的组织形式，这种方式非常容易扩展。

使用覆盖索引(covered index)
如果你的查询值需要查找索引中包含的字段，那就根本没必要获取实际的文档。**当一个索引包含用户请求的所有字段，可以认为这个索引覆盖了本次查询。**

**可以对数组建立索引**，这样就可以高效地搜索数组中的特定元素。

**何时不应该使用索引**
**结果集在原集合中所占的比例越大，索引的速度就越慢**，因为使用索引需要进行两次查找：一次是查找索引条目，一次是根据索引指针去查找相应的文档。而全表扫描只需要进行一次查找：查找文档。

**一般来说，如果查询需要返回集合内30%的文档(或者更多)，那就应该对索引和全表扫描的速度进行比较。**
`**可以用{"$natural":1}强制数据库做全表扫描。**`副作用：返回的结果是按照磁盘上的顺序排列。

# 索引类型

唯一索引
```
db.collection.ensureIndex({"key":1},{"unique":true})
```
稀疏索引
唯一索引会把null看做值，索引无法将多个缺少唯一索引中的键的文档插入到集合中。然而，在有些情况下，你可能希望唯一索引只对包含相应键的文档生效。如果有一个可能存在也可能不存在的字段，但是当它存在时，它必须是唯一的，这时就可以将unique和sparse选项组合在一起使用。

MongoDB中的稀疏索引(sparse index)与关系型数据库中的稀疏索引是完全不同的概念。基本上来说，MongoDB中的稀疏索引至少不需要将每个文档都作为索引条目。
db.ensureIndex({"key":1},{"unique":true,"sparse":true}) 
sparse也可以单独成为一个索引的。

索引管理
所有数据库索引信息都存储在`system.indexes`集合中。这是一个保留集合，不能在其中插入或者删除文档。

## 特殊的索引和集合
包括： 
- 用于类队列数据的**固定**集合(capped collection); 
- 用于缓存的TTL索引； 
- 用于简单字符串搜索的全文本索引； 
- 用于二维平面和球体空间的地理空间索引； 
- 用于存储大文件的GridFS。

**固定集合**
固定集合的行为类似于循环队列。如果已经没有空间了，最老的文档会被删除以释放空间，新插入的文档会占据这块空间。

固定集合不能被分片
创建固定集合
```
db.createCollection("collectionName",{"capped":true,"size":100000}) 
```
创建了一个100000字节的固定结合，max还可以一指定固定集合中文档的数量：size/max

**没有_id索引的集合**
如果在调用createCollection创建集合时指定autoIndextId选项为false，创建集合时就不会自动在”_id”上创建索引了。

**TTL索引**
这种索引运行**为每一个文档设置一个超时时间**。一个文档到达预设值的老化程度之后就会被删除。这种类型的索引对于缓存问题(比如会话的保存)非常有用。
在ensureIndex中指定expireAfterSecs选项就可以创建一个索引（时间单位：秒） 
```
db.collection.ensureIndex({"key":1},{"expireAfterSecs":60*60*24})
```

**全文本索引**
使用正则表达式搜索大块文本的速度非常慢，而且无法处理语言的理解问题(比如entry与entries应该算是匹配的)。使用全文本索引可以非常快地进行文本搜索，就如同内置了多种语言分词机制的支持一样。
**在一个操作频繁的集合上创建全文本索引可能会导致MongoDB过载，所以应该是离线状态下创建全文本索引。**
暂时不能在mongo启动这个功能

# 聚合

聚合工具： 
- 聚合框架 
- MapReduce 
- 几个简单聚合命令：count,distinct和group

## 聚合框架
使用聚合框架可以对集合中的文档进行变换和组合。基本上，可以用多个构件创建一个**管道(pipeline)**，用于对一连串的文档进行处理。这些构件包括**筛选(filtering)，投射(projecting)，分组(grouping)，排序(sorting)，限制(limiting)和跳过(skipping)。**
```
> db.articles.aggregate({"$project" : {"author" : 1}},
... {"$group" : {"_id" : "$author", "count" : {"$sum" : 1}}},
... {"$sort" : {"count" : -1}},
... {"$limit" : 5})
```
aggregate()会返回一个文档数组

管道操作符
每个操作符都会接受一连串的文档，对这些文档做一些类型转换，最后将转换后的文档作为结果传递给下一个操作符(对于最后一个管道操作符，是将结果返回给客户端)。

不同的管道操作符可以按任意顺序组合在以前使用，而且可以被重复任意多次。
$match例: {$match:{"state":"Oregon"}} 
$match可用于对文档集合进行筛选，之后就可以在筛选得到的文档子集上做聚合。它可以使用所有常规的查询操作符{“$gt”,”$lt”等}

通常，在实际使用中应该尽可能将”$match”放在管道的前面位置。一可以快速将不需要的文档过滤掉，以减少管道的工作量，二是如果在投影和分组之前执行”$match”，查询可以使用索引。

$project
投影操作。 
使用”$project”可以从文档中提取字段，可以重命名字段，还可以在这些字段上进行一些有意思的操作。
可以将投影过的字段进行重命名： 
```
> db.users.aggregate({"$project" : {"userId" : "$_id", "_id" : 0}})
> db.articles.aggregate({"$project" : {"newFieldname" : "$originalFieldname"}},
... {"$sort" : {"newFieldname" : 1}})
```
管道表达式
数学表达式
```
> db.employees.aggregate(
... {
...     "$project" : {
...         "totalPay" : {
...             "$add" : ["$salary", "$bonus"]
...         }
...     }
... })

"$add" : [expr1[, expr2, ..., exprN]]
"$subtract" : [expr1, expr2]
"$multiply" : [expr1[, expr2, ..., exprN]]
"$divide" : [expr1, expr2]
"$mod" : [expr1, expr2]
```
日期表达式
```
> db.employees.aggregate(
... {
...     "$project" : {
...         "tenure" : {
...             "$subtract" : [{"$year" : new Date()}, {"$year" : "$hireDate"}]
...         }
...     }
... })
```

字符串表达式
逻辑表达式
$group
$group操作可以将文档依据特定字段的不同值进行分组。
$unwind
拆分(unwind)可以将数组中的每一个值拆分为单独的文档。
```
> db.blog.findOne()
{
    "_id" : ObjectId("50eeffc4c82a5271290530be"),
    "author" : "k",
    "post" : "Hello, world!",
    "comments" : [
        {
            "author" : "mark",
            "date" : ISODate("2013-01-10T17:52:04.148Z"),
            "text" : "Nice post"
        },
        {
            "author" : "bill",
            "date" : ISODate("2013-01-10T17:52:04.148Z"),
            "text" : "I agree"
        }
    ]
}
> db.blog.aggregate({"$unwind" : "$comments"})
{
    "results" :
        {
            "_id" : ObjectId("50eeffc4c82a5271290530be"),
            "author" : "k",
            "post" : "Hello, world!",
            "comments" : {
            "author" : "mark",
            "date" : ISODate("2013-01-10T17:52:04.148Z"),
            "text" : "Nice post"
        }
    },
    {
        "_id" : ObjectId("50eeffc4c82a5271290530be"),
        "author" : "k",
        "post" : "Hello, world!",
        "comments" : {
        "author" : "bill",
        "date" : ISODate("2013-01-10T17:52:04.148Z"),
        "text" : "I agree"
            }
        }
    ],
    "ok" : 1
}
```

$sort
可以根据任何字段(或者多个字段)进行排序。如果要对大量的文档进行排序，强烈建立在管道的第一阶段进行排序，这时的排序操作可以使用索引。
```
> db.employees.aggregate(
... {
...     "$project" : {
...         "compensation" : {
...             "$add" : ["$salary", "$bonus"]
...             },
...             "name" : 1
...         }
... },
... {
...         "$sort" : {"compensation" : -1, "name" : 1}
... })
```
-1降序，1升序。

$limit
$skip
使用管道
应该尽量在管道的开始阶段(执行”$project”,”$group”或者”$unwind”操作之前)就将尽可能多的文档和字段过滤掉。管道如果不是直接从原先的集合中使用数据，那就无法在筛选和排序中使用索引。

## MapReduce
MapReduce使用JavaScript作为”查询语言”，因此它能够表达任意复杂的逻辑。然而，它非常慢。
步骤： 
1. 映射(map)，将操作映射到集合中的每个文档。这个操作要么”无作为”，要么”产生一些键和x个值”。 
2. 洗牌(shuffle)，按照键分组，并将产生的键值组成列表放到对应的键中。 
3. 化简(reduce)，把列表中的值化简成一个单值。这个值返回，然后接着进行洗牌，直到每个键的列表只有一个值为止，这个值也就是最终结果。
```
db.runCommand(
               {
                 mapReduce: <collection>,
                 map: <function>,
                 reduce: <function>,
                 out: <output>,
                 query: <document>,
                 sort: <document>,
                 limit: <number>,
                 finalize: <function>,
                 scope: <document>,
                 jsMode: <boolean>,
                 verbose: <boolean>
               }
             )
```

//另一种
```
var mapFunction = function() { ... };
var reduceFunction = function(key, values) { ... };

db.runCommand(
               {
                 mapReduce: <input-collection>,
                 map: mapFunction,
                 reduce: reduceFunction,
                 out: { merge: <output-collection> },
                 query: <query>
               }
             )
```

示例1：找出集合中的所有键
map函数中emit函数”返回”要处理的值。

```
> map = function() {
... for (var key in this) {
...     emit(key, {count : 1});
... }};

> reduce = function(key, emits) {
... total = 0;
... for (var i in emits) {
...     total += emits[i].count;
... }
... return {"count" : total};
... }

> mr=db.runCommand({"mapreduce":"book","map":map,"reduce":reduce,"out":"keyCount"})
{
    "result" : "keyCount",//存放MapReduce结果的集合名。这是个临时集合,MapReduce的连接关闭后它就被自动删除
    "timeMillis" : 130,//操作花费的时间
    "counts" : {
        "input" : 102,//发送到map函数的文档个数
        "emit" : 612,//在map函数中emit被调用的次数
        "reduce" : 12,//调用reduce的次数
        "output" : 6//结果集合中的文档数量
    },
    "ok" : 1
}
```

## 聚合命令
group操作的简化 
- count 
```
db.collection.count() 
db.collection.count({"key" : 1}) 
```
- distinct 
```
> db.runCommand({"distinct" : "people", "key" : "age"}) 
```
- group
```
> db.runCommand({"group" : {
... "ns" : "stocks",//指定要分组的集合
... "key" : "day",//指定文档分组依据的键
... "initial" : {"time" : 0},//每一组reduce函数调用中的初始"time"，会作为初始文档传递给后续过程。
... "$reduce" : function(doc, prev) {//这个函数会在集合内的每个文档上执行。系统会传递两个人参数：当前文档和累加器文档。
...     if (doc.time > prev.time) {
...         prev.price = doc.price;
...         prev.time = doc.time;
...     }
... }}})
```