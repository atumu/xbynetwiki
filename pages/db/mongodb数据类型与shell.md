author:xbynet
title:MongoDB权威指南2e学习一数据类型与shell
modifyAt:2016-12-27 00:26:58
location:db/mongodb数据类型与shell
createAt:2016-12-27 00:25:39

MongoDB是一个面向文档(document-oriented)的数据库。不采用关系模型的主要原因是为了获得更好的扩展性（横向扩展）。MongoDB设计就是为了作为分布式数据库。MongoDB具有卓越的性能。
# 相关概念
关于纵向扩展(scale up)与横向扩展(scale out):前者使用计算能力更强的机器，后者通过分区将数据分散到更多廉价的机器上。

| 关系数据库 | MongoDB |
| -------- | -------- |
| 实例     | 实例     |
| 数据库     | 数据库     |
| 表     | 集合(collection)     |
| 记录/行     | 文档(没有与定义模式(schema),文档键和值不再是固定的类型和大小，支持文档嵌套)，每个文档都有一个特殊的键"_id"用于标识唯一性，类似于主键的概念    |
| 字段/列     | 无     |
| SQL     | JavaScript     |

丰富的功能
除了基本的CRUD之外，还提供：

* **索引**:支持通用的二级索引，允许多种快速查询，且提供唯一索引，复合索引，地理空间索引，全文索引。
* **聚合**：支持聚合管道(aggregation pipeline)。
* **特殊的集合类型**：支持存在时间有限的集合，固定大小的集合等。
* **文件存储**

Mongodb并不支持关系数据库中的join,multirow transaction等。
MongoDB自带了一个JavaScript Shell用于管理数据库操作。

**文档：**
文档就是键值对的一个有序集。如:
{"greeting":"hello world!","foo":3}
文档的键是字符串，不能有重复的键，键值对是有序的。
键的值区分类型和大小写。

**集合：**
集合就是一组文档，集合是动态模式的，其中的文档可以是各式各样的，没有强制规定。
集合命名：不能以system.开头，不能包含$这个保留字符。子集合组织通过"."分隔符。
集合获取方式:

* db.collectionname
* db[collectionname]
* db.getCollection(collectionname)

**数据库：**
多个文档组成集合，多个集合组成数据库。一个实例可以承载多个数据库。
保留数据库名：admin,local,config

命令行工具
mongod:服务程序，默认数据目录为/data/db,windows下为C:\data\db，请确保此目录存在并且可写。默认端口27017 可选选项:--norc
mongo:客户端，是一个javascript shell(完备的，可以使用JS标准库)。可选选项:--nodb
比如mongo, mongo localhost:27017/test,

# 数据类型
* null
* true/false
* 数值:{"x":3.14},{"x":3},{"x":NumberInt("3")},{"x":NumberLong("3")},
* 字符串:{"x":"foo"} 支持UTF-8
* 日期:{"x":new Date()} 时区无关性
* 正则表达式：主要用于查询时作为限定条件，语法与JS正则语法一致.{"x":/foobar/i}
* 数组:{"x":["a","b"]}，可包含不同类型元素，支持数组嵌套。支持`原子更新数据内容`
* 内嵌文档:{"x":{"foo":"bar"}}
* 对象id:是文档唯一标识.{"x":`ObjectId`()} .一般`_id`这个键对应的值就是这个类型
* 二进制数据
* js代码

# MongoDB shell与js脚本
全局变量:db
全局类:Mongo
全局函数:load()
特有语法糖：(仅在终端可用，脚本需通过特定函数)
use dbname 切换数据库
help 查看特有命令列表,还可以db.help(),db.collectionname.help()
直接输入函数名可以直接查看函数代码:db.foo.update输出函数定义的代码
js脚本可使用变量与函数:

* db,Mongo,print()控制台打印
* db.getSisterDB("foo")与use foo功能一致
* db.getMongo().getDBs()与show dbs一致
* db.getCollectionNames()与show collections功能一致

执行js脚本：mongo [localhost:27017/test] script1.js script2.js [--quiet]，或者进入shell之后：load("script1.js")


## 基本CRUD操作：
```
>post={"title":"hello","content":"111","date":new Date()}
>db.blog.insert(post)
>db.blog.find() //db.blog.findOne()
>post.comments=[]
>db.blog.update({"title":"hello"},post)
>db.blog.remove({"title":"hello"})
```

创建`.mongorc.js`文件进行全局启动js脚本加载执行。在其中还可以注入并初始化后续要用到的一些全局变量。重写内置函数，或起别名等。 如果以mongod --norc指定不加载这个配置文件。

## shell提示符修改
```
prompt=function(){
    return (new Date())+">";
}
```
```
prompt=function(){
    if(typeof db=='undefined'){
        return '(nodb)>'
    }
    //检查最后的数据库操作
    try{
        db.runCommand({getLastError:1});
    }catch(e){
        print(e);
    }
    return db+">";
}
```

## 修改编辑器
EDITOR变量指定编辑器路径即可。比如修改为vim等。之后如果想编辑一个编辑，直接 edit 变量名即可





