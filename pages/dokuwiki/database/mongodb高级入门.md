title: mongodb高级入门 

#  MongoDB高级入门 
##  MongoDB关系 
MongoDB中的关系代表文件如何在各种逻辑上彼此相关。关系可以通过嵌入和引用的方法来模拟。这样的关系可以是 1:1, 1: N, N: 1 或 N: N.
让我们考虑存储用户地址的情况(例子)。这样，一个用户可以有多个地址，那么就成为一个 1：N 的关系。
下面是用户文件的示例文档的结构：
```

{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "name": "Tom Hanks",
   "contact": "987654321",
   "dob": "01-01-1991"
}

```
下面是地址文件的示例文档结构：
```

{
   "_id":ObjectId("52ffc4a5d85242602e000000"),
   "building": "22 A, Indiana Apt",
   "pincode": 123456,
   "city": "Los Angeles",
   "state": "California"
}

``` 
嵌入建模关系
在嵌入式方法中，我们将嵌入地址文件到用户文档中。

```

{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address": [
      {
         "building": "22 A, Indiana Apt",
         "pincode": 123456,
         "city": "Los Angeles",
         "state": "California"
      },
      {
         "building": "170 A, Acropolis Apt",
         "pincode": 456789,
         "city": "Chicago",
         "state": "Illinois"
      }]
}

``` 
这种方法保持在一个单一的文件，它可以很容易地检索和维护所有相关数据。 整份文件，可检索这样一个查询：
```

>db.users.findOne({"name":"Tom Benzamin"},{"address":1})

```
需要注意上面的查询，db和用户分别代表数据库和集合。
**缺点是，如果嵌入的文档不断增长过大，这会影响到读/写性能。**

**模型引用关系**
这是设计标准化关系的方法。在这种方法中，**用户和地址的文件将被分别维护**，但用户文档将包含的字段将引用地址文档的id字段。
```

{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address_ids": [
      ObjectId("52ffc4a5d85242602e000000"),
      ObjectId("52ffc4a5d85242602e000001")
   ]
}

``` 
如上图所示，用户文档包含数组字段address_ids，它包含相应地址的ObjectID。使用这些的ObjectID，我们可以查询地址文档，并从中获得地址的详细信息**。通过这种方法，我们将需要两个查询：首先来从用户文档取出address_ids字段，再次从地址集合获取这些地址。**
```

>var result = db.users.findOne({"name":"Tom Benzamin"},{"address_ids":1})
>var addresses = db.address.find({"_id":{"$in":result["address_ids"]}})

```
##  MongoDB数据库引用 
我们使用引用关系的概念，也被称为手动参考，我们在其中手动存储其他文档引用文档的ID。然而，在一些情况下文档中包含**来自不同的集合引用，我们可以使用MongoDB的DBRefs。**
**DBREFS VS 手动参考**
作为一个示例场景，我们将使用DBRefs代替手动参考，考虑当我们要存储不同类型的地址（家庭，办公室，邮寄等）在不同的集合（address_home，address_office，address_mailing等）的数据库。 现在，当用户集合文档引用地址，它也需要指定哪个集合来寻找到基于所述地址类型。在这种情况下，如果一个文档要引用多个文档集合，我们应该使用DBRefs。
使用DBRefs
在DBRefs有三个字段：
  * $ref: 此字段指定引用文档的集合
  * $id: 此字段指定引用文档的_id字段
  * $db: 这是一个可选字段，包含数据库的名称，其中所引用的文档
请考虑示用户文档有DBRef字段地址，如下图所示：
```

{
   "_id":ObjectId("53402597d852426020000002"),
   "address": {
   "$ref": "address_home",
   "$id": ObjectId("534009e4d852427820000002"),
   "$db": "yiibai"},
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin"
}

```
地址DBRef的字段在这里指定引用地址文档在address_home集合（在yiibai数据库），并有一个ID： 534009e4d852427820000002.
下面的代码看起来动态的集合是通过$ref参数指定(address_home 在这个示例中) ID为一个文档由DBRef 的 $id 参数指定。
```

>var user = db.users.findOne({"name":"Tom Benzamin"})
>var dbRef = user.address
>db[dbRef.$ref].findOne({"_id":(dbRef.$id)})

```
上面的代码返回以下存在于地址文档的address_home集合：
```

{
   "_id" : ObjectId("534009e4d852427820000002"),
   "building" : "22 A, Indiana Apt",
   "pincode" : 123456,
   "city" : "Los Angeles",
   "state" : "California"
}

```