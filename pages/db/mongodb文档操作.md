createAt:2016-12-31 13:13:18
author:xbynet
modifyAt:2016-12-31 13:15:01
location:db/mongodb文档操作
title:MongoDB权威指南2e学习二文档操作

增加、删除、修改键时，应该使用$开头的修改器，否则可能会将整个文档替换掉。
# 插入文档
db.foo.insert({"bar":"baz"})
批量插入：
db.foo.batchInsert({"_id":0},{"_id":1},{"_id":2})
在批量插入遇到错误时，如果希望batchInsert忽略错误并继续执行后续插入，可以使用continueOnError选项。(shell并不支持，但驱动程序支持该选项)
插入校验:Object.bsonsize(doc),所有文档都必须小于16M

# 删除文档
db.foo.remove()
db.foo.remove({"_id":10})
如果要清空整个集合，那么直接使用drop删除集合会更快。
db.foo.drop()

# 更新文档
Update有两个必须参数：一是查询文档，用于定位需要更新的目标文档，二是修改器文档，用于说明要找到的文档进行哪些修改
**修改器:以$开头，原子性修改。**

```
> db.foo.update({"name":"yyb"},{"name":"joe"})//将yyb这个文档修改成{“name”:“joe”}
```
**文档替换**
用一个新文档完全替换匹配的文档，这适合大规模模式迁移的情况。
```
//将这个文档的两个字段移到子文档为realtionships中
> var joe=db.user.findOne({"name":"joe"})
> joe.relationships={"friends":joe.friends,"enemies":joe.enemies};
{ "friends" : 32, "enemies" : 2 }
> delete joe.friends
true
> delete joe.enemies
true
> db.user.update({"name":"joe"},joe);
```
文档替换常见的错误是查询条件匹配到了多个文档，然后更新时由于第二个参数的存在就**产生重复的 _id 值**。数据库会抛出异常。

## 使用修改器
使用**原子性**的更新修改器，指定对文档的某些字段进行更新。更新修改器是种特殊的键，用来指定复杂的更新操作，比如修改、添加或者删除键，还可能是**操作数组或者内嵌文档**。
**$inc 原子自增**,该键不存在那就创建一个.
```
> db.user.update({"name":"yyb"},{$inc:{"age":5}})
```
注意：MongoDB**默认只会更新匹配的第一条**，如果要更新多条，还得指定参数。

**$set与$unset**
用来指定一个字段的值。如果这个字段不存在，则创建它。
```
>> db.user.update({"name":"yyb"},{"$set":{"email":"123@qq.com"}})
```
用 $set 甚至可以修改键的类型。用 $unset 可以将键完全删除。
也可以用 $set 修改内嵌文档：用**点(.)表达式**
```
> db.blog.update(
... {"author.name":"yyb"},
... {"$set":{"author.name":"joe"}})
```

### 数组修改器
**$push **添加元素。如果数组已经存在，会向已有的数组末尾加入一个元素，要是没有就创建一个新的数组。
```
> db.blog.post.update(
... {"title":"a blog post"},
... {"$push":{"comments":{"name":"joe","email":"joe@qq.com","content":"nice post"}}})
```
**$each**
可以将它应用在一些比较复杂的数组操作中。**使用 $each 子操作符，可以通过一次 $push 操作添加多个值。**
比如：下面将三个新元素添加到数组中。
```
db.stock.ticket.update(
... ... ... {"_id":"goog"},
... ... ... {"$push":{"hourly":{"$each":[562.776,562.790,559.123]}}})
```
注意：而下面这样是**不行的**
```
 db.stock.ticket.update(
... ... ... ... {"_id":"goog"},
... ... ... ... {"$push":{"hourly":[562.776,562.790,559.123]}})
```

**$slice**
如果**希望数组的最大长度是固定的，那么可以将 $slice 和 $push 组合在一起使用**，就可以保证数组不会超出设定好的最大长度。$slice 的值必须是**负整数**。
假设$slice的值为10，如果$push 后的数组的元素个数小于10，那么所有元素都会保留。反之，只有最后那10个元素会保留。因此，**$slice 可以用来在文档中创建一个队列**。
```
db.class.update(
... {"班级":"1班"},
... {"$push":{"students":{
... "$each":["zs","ls","ww"],
... "$slice":-5}}})
```
也可以在清理元素之前使用$sort，只要向数组中添加子对象就需清理，先排序后保留指定的个数。
```
>  db.class.update(
...  {"班级":"一班"},
... {"$push":{"students":
...  {"$each":[{"name":"aaa","age":1},{"name":"bbb","age":2},{"name":"ccc","age":3}],
...  "$slice":-2,
...  "$sort":{"age":-1}}}})
```

**$ne与$addToSet**
如果想将数组作为数据集使用，**保证数组内的元素不会重复**。可以在查询文档中用$ne或者$addToSet来实现。有些情况$ne根本行不通，有些时候更适合用$addToSet。
```
> db.papers.update({"authors cited":{"$ne":"Richie"}}, {"$push":{"authors cited":"Richie"}})
```
```
> db.user.update(
... ... {"username":"joe"},
... ... {"$addToSet":{"emails":"joe@gmail.com"}})
```
**将$addToSet和$each组合起来，可以添加多个不同的值。而用$ne和$push组合就不能实现。**
**$addToSet与$push的区别：前者添加到数组中时会去重，后者不会。**
```
>  db.user.update(
... {"username" : "joe"},
... 
... {"$addToSet":
...  {"emails" :{"$each": [
... "joe@example.com",
...  "joe@gmail.com",
... "joe@yahoo.com",
... "joe@hotmail.com",
...  "joe@hotmail.com"]}}})
```


-----


从数组中删除元素
**$pop**可以从数组**任何一端**删除元素。
{“$pop”:{“key”:1}}从数组末尾删除一个元素
{“$pop”:{“key”:-1}}从数组头部删除一个元素
```
> db.class.update(
... {"班级" : "一班"},
... {"$pop":{"students":1}})
```

**$pull**
有时需要基于特定条件来删除，而不仅仅是依据元素位置，这时可以使用$pull。**$pull会将所有匹配的文档删除，而不是只删除一个。**
```
> db.list.update({},{"$pull":{"todo":"laundry"}})
```


-----


基于位置的数组修改器
**有两种方法操作数组中的值：通过位置或者定位操作符($)**
位置
通过数组位置来操作。数组都是以0开头的，可以将下标直接作为键来选择元素。
```
db.blog.update({"content":"..."},{"$inc":{"comments.0.votes":1}})
```
定位操作符$
如果无法知道要修改的数组的下标，可以使用定位操作符$,用来定位查询文档已经匹配的元素，并进行更新。
```
> db.blog.update(
... {"comments.author":"john"},
... {"$set":{"comments.$.author":"jim"}})
```

## upsert
upsert是update()的第三个参数。没有则创建，有则正常更新。
```
> db.analytics.update({"url":"/blog"},{"$inc":{"pageviews":1}},true)
```
**$setOnInsert**
在创建文档的同时创建字段并为它赋值，但是在之后的所有更新操作中在，**这个字段的值都不在改变。**
$setOnInsert只会在文档插入时设置字段的值。
在预置或者初始化计数器时，或者对于不使用ObjectIds的集合来说，“$setOnInsert”是非常有用的。
```
> db.user.update({},{"$setOnInsert":{"createAt":new Date()}},true)
```
save
一个shell函数，不存在创建，反之则更新文档。
它只有一个参数：文档。要是这个文档含有“_id”键，save会调用upsert。否则会调用insert。在shell中快速对文档进行修改。

## 更新多个文档
**默认情况下，只会更新匹配的第一个文档**。要更新多个文档，需要将**update的第4个参数设置为true。**
想要知道多文档更新到底更新了多少文档，可以运行**getLastError命令**。键n的值就是被更新文档的数量。
```
> db.coll.update({},{"$set":{"x":10}},false,true )
 > db.runCommand({getLastError:1})
{
        "connectionId" : 1,
        "updatedExisting" : true,
        "n" : 3,
        "syncMillis" : 0,
        "writtenTo" : null,
        "err" : null,
        "ok" : 1
}
```
返回被更新的文档
通过**findAndModify**命令得到被更新的文档。**返回的是修改之前的文档。**
对于操作队列以及执行其他需要进行原子性取值和赋值的操作非常方便。
```
ps=db.runCommand({"findAndModify":"processes", "query":{"status":"ready"}, 
"sort":{"pirority":-1}, 
"update":{"$set":{"status":"Running"}}})

{
        "lastErrorObject" : {
                "updatedExisting" : true,
                "n" : 1
        },
        "value" : {
                "_id" : ObjectId("5857c939d7a32a888bd79c47"),
                "pirority" : 2,
                "status" : "ready"
        },
        "ok" : 1
}
```
findAndModify可以使用update键也可以使用remove键。Remove键表示将匹配的文档从集合里面删除。
```
> ps=db.runCommand({"findAndModify":"processes", "query":{"status":"ready"}, 
"sort":{"priority":-1}, 
"remove":true}).value 

{
        "_id" : ObjectId("5857c9b1d7a32a888bd79c48"),
        "pirority" : 2,
        "status" : "ready"
}
```
findAndModify可以使用的字段：
query,sort,update,remove,new,fields,upsert

# 查询
数据准备
```
{ "goods_id" : 1, "goods_name" : "KD876", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : "4", "goods_name" : "诺基亚N85原装充电器", "createTime" : ISODate("2016-09-11T00:00:00Z") }
{ "goods_id" : 3, "goods_name" : "诺基亚原装5800耳机", "createTime" : ISODate("2016-10-09T00:00:00Z") }
{ "goods_id" : 5, "goods_name" : "索爱原装M2卡读卡器", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 6, "goods_name" : "胜创KINGMAX内存卡", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 7, "goods_name" : "诺基亚N85", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 8, "goods_name" : "飞利浦9@9v", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 9, "goods_name" : "诺基亚E66", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : "10", "goods_name" : "索爱C702c", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 11, "goods_name" : "索爱C702c", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 12, "goods_name" : "摩托罗拉A810", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 13, "goods_name" : "诺基亚5320", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 14, "goods_name" : "诺基亚5800XM", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : "15", "goods_name" : "摩托罗拉A810", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 16, "goods_name" : "恒基伟业G101", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 17, "goods_name" : "夏新N7", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 18, "goods_name" : "夏新T5", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 19, "goods_name" : "三星SGH-F258", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 20, "goods_name" : "三星BC01", "createTime" : ISODate("2016-12-21T11:19:39.010Z") }
{ "goods_id" : 2, "createTime" : ISODate("2015-10-01T00:00:00Z") }
```
**find**
find的第一个参数决定了要返回哪些文档，用于指定查询条件。要不指定查询文档，默认就是{}，指**定多个键/值对，相当于sql的and。**第二个参数来**指定想要的键**（默认情况下，"_id"总是显示）。
## 查询条件
**And查询**
使用AND型查询时，应尽可能用最少的条件来限定结果的范围。
```
> db.product.find({"goods_id":"4","goods_name":"诺基亚N85原装充电器"},{"_id":0})
{ "goods_id" : "4", "goods_name" : "诺基亚N85原装充电器", "createTime" : ISODate("2016-09-11T00:00:00Z") }
```

**Or查询**
方式一：**$in、$nin**
$in可以用来查询一个键的多个值,可以指定不同类型的条件和值。
$nin将返回与数组中所有条件都不匹配的文档。
```
> db.product.find({"goods_id":{"$in":["4",5,6]}},{"_id":0})
```
方式二：**$or**
$or更通用一些，可以在多个键中查询任意的给定值
使用$or时，第一个条件条件应尽可能匹配更多的文档，这样才是最为高效的。
```
> db.product.find({"$or":[{"goods_id":16},{"goods_name":"夏新T5"},{"createTime":{"$lt":new Date("2016-01-01")}}]},{"_id":0})
```

**$lt、 $lte、 $gt、 $gte**分别对应sql中的<、<= 、>、>=。组合查找一个范围的值。
```
> db.product.find({"goods_id":{"$gte":3,"$lt":5}},{"_id":0} )
```
```
> start=new Date("01/01/2016")
> db.product.find({"createTime":{"$lt":start}})//查询指定日期之前的数据
```

**$mod**
取模运算符，会将查询的值除以第一个给定值，若余数等于第二个给定的值则匹配成功。
```
> db.product.find({"goods_id":{"$mod":[5,1]}},{"_id":0})
```

**$not**
$not是元条件句，即可以用在其他任何其他条件之上。查询和匹配的件相反的数据。
**$not与正则表达式联合使用时极为有用**，用来查找那些与特定模式不匹配的文档。
```
> db.product.find({"goods_id":{"$not":{"$mod":[5,1]}}},{"_id":0})
```
条件语义
在查询中，$lt在内层文档，而在更新中$inc则是外层文档的键。
基本可以肯定：**条件语句是内层文档的键，而修改器则是外层文档的键。**
**一个键可以有任意多个条件，但是一个键不能对应多个更新修改器。**
有一些”元操作符”也位于外层文档中，比如$and、$or、$nor


-----


特定类型的查询
**null**
null不仅会匹配某个键的值为null的文档，而且还会匹配不含这个键的文档。
**如果仅想匹配键值为null的文档，既要检查该键的值是否为null，还要通过$exists条件判断键值已存在。**
```
> db.product.find({"goods_no":{"$in":[null],"$exists":true}},{"_id":0})
```

**正则表达式**
正则表达式能够灵活有效地匹配字符串，系统可以接收正则表达式标志i (忽略大小写)，但不一定要有。
MongoDB可以为前缀型正则表达式(比如：/^joey/)查询创建索引，所以这种类型的查询会非常高效。
```
> db.product.find({"goods_name":/^三星/},{"_id":0})
```
## 查询数组
`查询数组元素与查询标量值是一样的。`
```
> db.product.update({"goods_id":"10"},{"$push":{"goods_type":{"$each":["华为", "乐视", "小米"]}}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.product.find({"goods_type":"小米"},{"_id":0})
```
**$all**
通过多个元素来匹配数组，这样就会匹配一组元素。
```
> db.product.find({"goods_type":{"$all":["苹果","华为"]}},{"_id":0})
```
当然也可以使用整个数组进行精确匹配。但是，精确匹配必须一模一样的才查的出来，比如顺序，个数都得一样。

**查找特定位置的元素**
要想查询数组特定位置的元素，需使用key.index语法指定下标，数组下标都是从0开始的。
```
> db.product.find({"goods_type.1":"苹果"},{"_id":0})
```
**$size**
$size查询特定长度的数组
```
> db.product.find({"goods_type":{"$size":3}},{"_id":0})
```
**$slice**
$slice操作符可以返回某个键匹配的数组元素的一个子集。这个操作符是用来获取数组的前几个或者后几个元素，对整个数据进行过滤没有作用（比如： db.product.find({},{"goods_type":{"$slice":2}}) 会返回文档的所有数据）
```
> db.product.find({"goods_type":{"$size":3}},{"goods_type":{"$slice":-2},"_id":0})
```
$slice也可以指定偏移值以及希望返回的元素数量，来返回元素集合中间位置的某些结果。
```
> db.product.find({"goods_type":{"$size":3}},{"goods_type":{"$slice":[1,1]},"_id":0})//跳过1个，取1个
```
返回一个匹配的数组元素
$ 返回与查询条件相匹配的任意一个数组元素。只会返回第一个匹配的文档。
```
> db.blog.find({"comments.votes":3},{"comments.$":1}).pretty()
```
### 数组和范围查询的相互作用
文档中的标量（非数组元素）必须与查询条件中的每一条语句相匹配。比如，如果使用 {"$gt":10,"$lt":20} 进行查询，只会匹配x键的值大于10并且小于20的文档。
**但是，假如某个文档的x字段是一个数组，如果x键的某一个元素与查询条件得到任意一条语句相匹配，那么这个文档也会被返回。**
```
{ "_id" : ObjectId("585d404b53fe663a4c5c202e"), "x" : 5 }
{ "_id" : ObjectId("585d404b53fe663a4c5c202f"), "x" : 15 }
{ "_id" : ObjectId("585d404b53fe663a4c5c2030"), "x" : 25 }
{ "_id" : ObjectId("585d404b53fe663a4c5c2031"), "x" : [ 5, 25 ] }
> db.foo.find({"x":{"$gt":5,"$lt":20}})
{ "_id" : ObjectId("585d404b53fe663a4c5c202f"), "x" : 15 }
{ "_id" : ObjectId("585d404b53fe663a4c5c2031"), "x" : [ 5, 25 ] }
```
上述方式对数组使用范围查询没有用：范围会匹配任意多个元素数组。有几种方式可以得到预期的结果。
方式1：**$elemMatch**
$elemMatch要求MongoDB同时使用查询条件中的两个语句与一个数组元素进行比较。**但是，它不会匹配非数组元素。**
```
> db.foo.find({"x":{"$elemMatch":{"$gt":10,"$lt":20}}})
```
方式2：**使用min()和max()**
如果当前查询的字段上**创建过索引**，可以使用min()和max()将查询条件遍历的索引范围限制为”$gt”和”$lt”的值。而且，必须为这个索引的所有字段指定min()和max().
```
> db.foo.find({"x":{"$gt":10,"$lt":20}}).min({"x":10}).max({"x":20})
```

## 查询内嵌文档
方式1：查询整个内嵌文档
与普通查询完全相同。但是，**如果要查询一个完整的子文档，那么子文档必须精确匹配（顺序以及个数都要一样）。**
方式2：只针对其键/值对进行查询
```
> db.blog.find({"comments.author":"yyb","comments.votes":10}).pretty()
```
$elemMatch
要正确地指定一组条件，而不必指定每个键,就需要使用 **$elemMatch** 。这种模糊的命名条件句能用来在查询条件中部分指定匹配数组中的单个内嵌文档。
**$elemMatch将限定条件进行分组，仅当需要对一个内嵌文档的多个键操作时才会用到。**
```
> db.blog.find({"comments":{"$elemMatch":{"author":"yyb","votes":{"$gte":10}}}}).pretty()
```

 ### $where
用它可以在查询中执行任意的js。这样就能在查询中做任何事情，为安全起见，应该严格限制或消除$where语句的使用。应该禁止终端用户使用任意的$where语句。
如果函数返回true,文档就作为结果集的一部分返回；如果为false，就不返回。
**使用$where在速度上要比常规查询慢很多。**每个文档都要从BSON转换成js对象，然后通过$where表达式来运行。**而且不能使用索引**。实在不得已，最好先使用常规查询（或索引）进行过滤，然后使用$where语句，这样组合使用可以降低性能损失。
$where语句最常见的应用就是比较文档中的两个键的值是否相等。比如：
```
> db.foo.insert({"apple":1,"banana":6,"pach":3})
WriteResult({ "nInserted" : 1 })
> db.foo.insert({"apple":8,"banana":4,"pach":4})
WriteResult({ "nInserted" : 1 })
> db.foo.find({"$where":function(){
... for(var cur in this){
...   for(var other in this){
...      if(cur!=other && this[cur]==this[other]){
...        return true;
...       }
...     }
...   }
...   return false;
...   }});
{ "_id" : ObjectId("585f768398992d5a44085fc2"), "apple" : 8, "banana" : 4, "pach" : 4 }
```

## 游标
数据库使用游标返回find的执行结果。客户端对游标的实现通常能够对最终结果进行有效地控制。可以限制结果的数量，略过部分结果，根据任意键按任意顺序的组合对结果进行各种排序，或者执行其他一些强大的操作。
```
> var cursor=db.coll.find()
> while(cursor.hasNext()){
... obj=cursor.next();
... print(obj.x+"============"+obj._id);
... }
```
cursor.hasNext()检查是否有后续结果存在，然后用cursor.next()获取它。
游标类还实现了js的迭代接口，所以可以在forEach循环中使用：

```
> var coll=db.coll.find({"x":{"$lte":3}})
> coll.forEach(function(row){
... print(row._id+"========="+row.x);
... })
```
调用find时，shell并不立即查询数据库，而是等待真正开始获得结果时才发送查询，这样在执行之前可以给查询附加额外的选项。
**几乎游标对象的每个方法都返回游标本身**，这样就可以按照任意顺序组成**方法链**。例如，下面几种表达是**等价**的：
```
> var cursor=db.coll.find().sort({"x":-1}).skip(10).limit(1)//实现分页的方式：先倒序，跳过10条，取1条
> var cursor=db.coll.find().limit(1).sort({"x":-1}).skip(10) 
> var cursor= db.coll.find().skip(10).limit(1).sort({"x":-1})
```
此时，查询还没有真正执行，所有这些函数都只是构造查询。

**注意：避免使用skip略过大量结果，用skip略过少量的文档还是不错的。但是要是数量非常多的话，skip就会变得很慢，通常可以利用上次的结果来计算下一次查询条件。**
比如可以利用最后一个文档中的“date”的值最为查询条件，来获取下一页：
```
> var page1=db.coll.find().sort({"date":-1}).limit(10)
> var lastest=null;
> while(page1.hasNext()){ latest=page1.next(); print(latest.x)}
> var page2=db.coll.find({"date":{"$gt":latest.date}}).sort({"date":-1}).limit(10)
```

**获取一致结果**
数据处理通常做法就是先把数据从MongoDB中取出来，然后做一些变换，最后在存回去。
结果比较少时，没什么问题。但是如果结果集比较大，MongoDB可能会多次返回同一个文档。原因是：体积变大的文档，可能无法保存回原先的位置。MongoDB会为更新后无法放回原位置的文档重新分配存储空间。
解决方案：对查询进行快照。这样查询就在_id索引上遍历执行，这样可以保证每个文档只被返回一次。
`> db.coll.find().snapshot()`
快照会使查询变慢，如非必要，不建议使用。

**游标生命周期**
看待游标的两种角度：客户端游标以及客户端游标表示的数据库游标。前面所说的游标都是客户端游标。
在服务器端，游标消耗内存和其他资源。游标遍历尽了结果之后，获取客户端发来消息要求终止，数据库就会释放这些资源。还有一些情况导致游标终止：
游标完成匹配结果的迭代时。
客户端的游标已经不再作用域内，驱动程序会向服务端发送一条特别的消息，让其销毁游标。
一个游标在10分钟之内没有使用。

**数据库命令**
在shell中使用的辅助函数，都是对数据库命令的封装，而提供的更加简单的接口。
```
> db.runCommand({"drop":"coll"})
{ "ns" : "test.coll", "nIndexesWas" : 1, "ok" : 1 }
> db.coll.drop()
false
```
查看所有的数据库命令
```
> db.listCommands()
```
数据库命令总会返回一个包含ok键的文档，如果值是1,则表示成功，反之失败，那么命令的返回文档中就会有一个额外的键errmsg。用于描述失败的原因。
```
>  db.runCommand({"drop":"coll"})
{ "ok" : 0, "errmsg" : "ns not found", "code" : 26 }
```
MongoDB中的命令被实现为一种特殊类型的查询，这些特殊的查询会在`$cmd集合`上执行。
**有些命令需要管理员权限，而且要在admin数据库上才能执行。如果当前位于其他的数据库，但是需要执行一个管理员命令，可以使用 adminCommand**
```
> db.runCommand({"shutdown":1})
{
        "ok" : 0,
        "errmsg" : "shutdown may only be run against the admin database.",
        "code" : 13
}
> db.adminCommand({"shutdown":1})
```
参考：http://www.cnblogs.com/ginb/p/6184916.html
http://www.cnblogs.com/ginb/p/6200721.html
http://www.cnblogs.com/ginb/p/6219571.html