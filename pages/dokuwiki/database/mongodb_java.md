title: mongodb_java 

#  MongoDB Java 
要在 java 中使用MongoDB，需要到 classpath 包括 mongo.jar。可以下载 jar包从路径 下载mongo.jar。请一定要下载它的最新版本。
**建立连接**
要连接，需要指定数据库名称，**如果数据库不存在，则 MongoDB 会自动创建它。**
代码片段连接到数据库，将如下：
```

import com.mongodb.MongoClient;
import com.mongodb.MongoException;
import com.mongodb.WriteConcern;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.BasicDBObject;
import com.mongodb.DBObject;
import com.mongodb.DBCursor;
import com.mongodb.ServerAddress;

import java.util.Arrays;

// To connect to mongodb server
MongoClient mongoClient = new MongoClient( "localhost" , 27017 );
// Now connect to your databases
DB db = mongoClient.getDB( "test" );
boolean auth = db.authenticate(myUserName, myPassword);

```
**身份验证值是 true，那么所选数据库的用户名和密码是有效的。**

**获取一个集合列表**
为了从数据库获得集合列表，com.mongodb.DB类使用getCollectionNames()方法。
代码片段集合列表：
```

Set colls = db.getCollectionNames();
for (String s : colls) {
   System.out.println(s);
}

```
**获取/选择一个集合**
要 获得/选择数 据库中一个集合，使用com.mongodb.DBCollection类的 getCollection()方法。 
代码片段获得/选择数一个集合：
```

DBCollection coll = db.getCollection("mycol");

```
**插入文档**
要插入到 MongoDB 文档, 使用com.mongodb.DBCollection类的insert() 方法
```

BasicDBObject doc = new BasicDBObject("title", "MongoDB").
   append("description", "database").
   append("likes", 100).
   append("url", "http://www.yiibai.com/mongodb/").
   append("by", "yiibai.com").
   ;
coll.insert(doc);

```
