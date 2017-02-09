title: greendao 

#  GreenDao 

GreenDao是Android当中的高性能ORM框架。（其他的有OrmLite等）
项目地址：https://github.com/greenrobot/greenDAO 
![](/data/dokuwiki/pasted/20150511-055225.png)
同时GreenDao还有一个子项目为GreenDao Code Generator:
![](/data/dokuwiki/pasted/20150511-055231.png)
GreenDao的核心类及其工作如下：
![](/data/dokuwiki/pasted/20150511-055237.png)

#  使用初始化 

使用greendao添加greendao.jar;
（使用greendao-generator则需要添加greendao-generator.jar与freemarker.jar这一点在以后讲）
相关jar包可以通过maven中央仓库下载。
```

helper = new DaoMaster.DevOpenHelper(this, "notes-db", null);//greendao会创建notes-db这个数据库  
db = helper.getWritableDatabase();  
daoMaster = new DaoMaster(db);  
daoSession = daoMaster.newSession();  
noteDao = daoSession.getNoteDao();  
使用示例：
[java] view plaincopy
Note note = new Note(null, noteText, comment, new Date());  
noteDao.insert(note);  
Log.d("DaoExample", "Inserted new note, ID: " + note.getId());  
[java] view plaincopy
noteDao.deleteByKey(id);  

```
#  数据模型与代码生成 
完整示例：
```


import java.io.IOException;

import de.greenrobot.daogenerator.*;
public class GenConfig{
public static void main(String[] args) throws IOException, Exception{
		Schema schema = new Schema(1, "net.xby1993.baiduttstest");  
  
Entity note= schema.addEntity("RSSModel");  
note.addIdProperty().primaryKey().autoincrement();  
note.addStringProperty("title").notNull();  
note.addStringProperty("url");  
note.addDateProperty("date");  
note.addStringProperty("summary");
note.addStringProperty("encoding");
  
new DaoGenerator().generateAll(schema, "D:/lucene/src");  //记得保证该目录必须存在，否则报错
}
}

```
一般情况下，你使用GreeDao需要创建两个项目，一个是你的Android项目添加greendao.jar依赖，另外一个是普通的java se工程.添加greendao-generator.jar与freemarker.jar依赖。后者用于数据模型domain,dao,DaoMaster等代码的生成。
DoMaster与DaoSession及XxxDao以及实体类均会生成。DoMaster与DaoSession对应于你当前的数据模型。这两个类中每个数据模型生成的方法均不同
数据模型代码生成示例-使用greendao-generator:
```

Schema schema = new Schema(1, "de.greenrobot.daoexample");  
  
Entity note= schema.addEntity("Note");  
note.addIdProperty();  
note.addStringProperty("text").notNull();  
note.addStringProperty("comment");  
note.addDateProperty("date");  
  
new DaoGenerator().generateAll(schema, "../DaoExample/src-gen");  

```
Schema代表你的数据库，Entity代表你要生成的数据表结构。向Entity添加属性相当于添加列结构。
<note>注意：../DaoExample/src-gen路径必须存在否则会报错</note>

#  模型化实体Entities 
![](/data/dokuwiki/pasted/20150511-055438.png)

##  Schema: 
```

Schema schema = new Schema(1, "de.greenrobot.daoexample");//第一个参数代表版本，第二个参数代表要生成代码的包名  
默认情况下Dao类与Test类是在一个包下，如果你想分开他们，可以这样：
schema.setDefaultJavaPackageTest("de.greenrobot.daoexample.test");  
schema.setDefaultJavaPackageDao("de.greenrobot.daoexample.dao");  
Schema对于Entity还有两个默认的标志Flags可以设置：
schema2.enableKeepSectionsByDefault();  
schema2.enableActiveEntitiesByDefault();

```
##  Entity 

Schema可以用于添加Entity:
Entity user = schema.addEntity("User");  
为实体添加属性：
user.addIdProperty();  
user.addStringProperty("name");  
user.addStringProperty("password");  
user.addIntProperty("yearOfBirth");  

###  为实体添加主键 

注意：greendao的主键支持目前并不完善，还处于开发中，但是我们可以使用下面的方式添加主键：
```

user.addIdProperty().primaryKey().autoIncrement();  

```
#  关于Java属性与对应的数据库表名列名命名的规则与区别 

Java中属性一般采用驼峰命名法。
**你可以通过Entity.setTableName修改表名**
^表名/domain类名 ^属性/列I	^属性/列II
|Java	|User	|name	|myName
|数据库 |USER	|NAME	|MY_NAME
#  Inheritance, Interfaces, and Serializable 

对于继承：（不推荐）
myEntity.setSuperclass("MyCommonBehavior");  
推荐使用接口将一些公共的属性提取出来。
entityA.implementsInterface("C");  
entityB.implementsInterface("C");  
entityB.implementsSerializable();  
#  触发代码生成 

DaoGenerator daoGenerator = new DaoGenerator();  
daoGenerator.generateAll(schema, "../MyProject/src-gen");  
还可以指定第三个参数来将test代码分开。
#  Keep sections片段 

由于GreenDaoGenerator会在每次运行后覆盖原有的生成的实体代码,` 为了允许添加兵保留你自定义代码到你的实体当中 `，greendao使用"keep sections"来允许你添加，但是要先调用Schema的enableKeepSectionsByDefault()或者setHasKeepSections(true) .运行generator之后就会在生成的实体类当中生成如下注释，我们只需要往这些注释中添加自定义代码以后每次运行generator后都会保留这部分代码。
```

// KEEP INCLUDES - put your custom includes here  
// KEEP INCLUDES END  
...  
// KEEP FIELDS - put your custom fields here  
// KEEP FIELDS END  
...  
// KEEP METHODS - put your custom methods here  
// KEEP METHODS END  

```
<note tip>不要删除这些注释。</note>

#  Sessions 

##  DaoMaster and DaoSession 
```

daoMaster = new DaoMaster(db);  
daoSession = daoMaster.newSession();  
noteDao = daoSession.getNoteDao();  

```
注意数据库连接是属于DaoMaster的，每个Session都需要分配内存，**对于实体，greendao采用对应的session缓存cache**

##  Identity scope and session “cache” 

**greendao默认的行为是多个不同的查询返回同一个java objects,**举例：从USER表中加载一个ID为42的对象，结果对于每一个查询都会返回同一个java对象。
另一个作用就是**缓存实体**。greendao是用weak reference在内存中保存实体，所以当再次加载时，greendao不会从数据库加载，而是直接返回该session缓存中的对象。
<note important>注意：一旦需要对其进行更改，及时你提交到了数据库，但是缓存中的对象数据仍然没有更新，这个时候需要你手动进行更新缓存中的对象。切记！！！</note>

#  多表关系映射 

##  To-One 

**相当于外键关系。**
```

// The variables "user" and "picture" are just regular entities  
Property pictureIdProperty = user.addLongProperty("pictureId").getProperty();  
user.addToOne(picture, pictureIdProperty);  

```
这将导致产生的User实体类中有一个Picture属性(getPicture/setPicture);
##  Relation Names and multiple Relations 

每一个关联都有一个名称。默认情况下关联的名称就是目标实体的名称。所以一般情况下` 建议主动设置该关联的名称以免重名 `。可以通过setName()来设置。
```

Property pictureIdProperty = user.addLongProperty("pictureId").getProperty();  
Property thumbnailIdProperty = user.addLongProperty("thumbnailId").getProperty();  
user.addToOne(picture, pictureIdProperty);//使用默认的关系名  
user.addToOne(picture, thumbnailIdProperty, "thumbnail");//为了防止重名，设置关系名为thumbnail  

Property customerId = order.addLongProperty("customerId").notNull().getProperty();  
ToMany customerToOrders = customer.addToMany(order, customerId);  
customerToOrders.setName("orders"); // Optional  
customerToOrders.orderAsc(orderDate); // Optional  
产生的代码中Customer类将多出一个getOrders()
List orders = customer.getOrders();  

```

##  Resolving and Updating To-Many Relations 

` To-Many解析第一次使用懒加载，但是一旦加载之后to-many list就会被缓存到一个List当中，后续的请求不会再通过数据库，而是直接从缓存中返回，所以一旦修改之后，需要对缓存中的数据进行更新。 `
由于缓存的作用下面的代码会产生令人困惑的结果：
```

List orders1 = customer.getOrders();  
int size1 = orders1.size();  
  
Order order = new Order();  
order.setCustomerId(customer.getId());  
daoSession.insert(order);  
  
Listorders2 = customer.getOrders();  
// size1 == orders2.size(); // NOT updated  
// orders1 == orders2; // SAME list object  

```
` 所以我们需要对缓存进行Updating `
**改正后的代码如下：**
```

List orders = customer.getOrders();  
newOrder.setCustomerId(customer.getId());  
daoSession.insert(newOrder);  
orders.add(newOrder);//更新缓存  
对于删除操作也是一样的。：
List orders = customer.getOrders();  
daoSession.delete(newOrder);  
orders.remove(newOrder);//更新缓存  

```
但是如果有个时候这些没法达到你预期的要求或者是更新缓存比较困难的情况下，没关系greendao还提供如下方法resetXxx()` 重置缓存： `
```

customer.resetOrders();  
List orders2 = customer.getOrders();  

```

#  双向关联To-One与To-many结合使用 
```

Entity customer = schema.addEntity("Customer");  
customer.addIdProperty();  
customer.addStringProperty("name").notNull();  
  
Entity order = schema.addEntity("Order");  
order.setTableName("ORDERS"); // "ORDER" is a reserved keyword  
order.addIdProperty();  
Property orderDate = order.addDateProperty("date").getProperty();  
Property customerId = order.addLongProperty("customerId").notNull().getProperty();  
order.addToOne(customer, customerId);  
  
ToMany customerToOrders = customer.addToMany(order, customerId);  
customerToOrders.setName("orders");  
customerToOrders.orderAsc(orderDate);  
这样便产生了双向关联了。
List allOrdersOfCustomer = order.getCustomer().getOrders(); 

```

##  Many-to-Many Relations (n:m) 

<note>目前greendao还没有实现。</note>
##  Modelling Tree Relations 

You can model a tree relation by modelling an entity ` having a to-one and a to-many relation pointing to itself: `
```

Entity treeEntity = schema.addEntity("TreeEntity");  
treeEntity.addIdProperty();  
Property parentIdProperty = treeEntity.addLongProperty("parentId").getProperty();  
treeEntity.addToOne(treeEntity, parentIdProperty).setName("parent");  
treeEntity.addToMany(treeEntity, parentIdProperty).setName("children");  
然后再生成的代码中我们就可以进行导航了：
[java] view plaincopy
TreeEntity parent = child.getParent();  
List grandChildren = child.getChildren();  

```