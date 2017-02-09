createAt:2017-01-15 12:34:15
author:xbynet
modifyAt:2017-01-15 12:34:15
location:db/pymongo
title:PyMongo

相关网址：
https://github.com/mongodb/mongo-python-driver
http://api.mongodb.com/python/current/
http://mongoengine.org/
https://github.com/MongoEngine/mongoengine
http://docs.mongoengine.org/index.html

```
pip install pymongo
```
```
import pymongo
from pymongo import MongoClient
client = MongoClient()
#client = MongoClient('localhost', 27017)
#client = MongoClient('mongodb://localhost:27017')
```
一旦你有一个连接的MongoClient实例，你可以在Mongo服务器中访问任何数据库。
```
db = client.pymongo_test
db = client['pymongo_test']
```
插入文档
```
posts = db.posts
post_data = {
    'title': 'Python and MongoDB',
    'content': 'PyMongo is fun, you guys',
    'author': 'Scott'
}
result = posts.insert_one(post_data)
print('One post: {0}'.format(result.inserted_id))
```
如果你有很多的文档添加到数据库中，可以使用方法`insert_many()`。此方法接受一个list参数：
```
post_1 = {
    'title': 'Python and MongoDB',
    'content': 'PyMongo is fun, you guys',
    'author': 'Scott'
}
post_2 = {
    'title': 'Virtual Environments',
    'content': 'Use virtual environments, you guys',
    'author': 'Scott'
}
post_3 = {
    'title': 'Learning Python',
    'content': 'Learn Python, it is easy',
    'author': 'Bill'
}
new_result = posts.insert_many([post_1, post_2, post_3])
print('Multiple posts: {0}'.format(new_result.inserted_ids))
```
检索文档
检索文档可以使用`find_one()`方法，比如要找到author为Bill的记录:
```
bills_post = posts.find_one({'author': 'Bill'})
print(bills_post)
 
运行结果:
{
    'author': 'Bill',
    'title': 'Learning Python',
    'content': 'Learn Python, it is easy',
    '_id': ObjectId('584c4afdea542a766d254241')
}
```
如果需要查询多条记录可以使用`find()`方法：
```
scotts_posts = posts.find({'author': 'Scott'})
print(scotts_posts)
 
结果:
<pymongo.cursor.Cursor object at 0x109852f98>
```
他的主要区别在于文档数据不是作为数组直接返回给我们。相反，我们得到一个**游标对象**的实例。
```
for post in scotts_posts:
    print(post)

```
# MongoEngine 
PyMongo之上提供了一个更高的抽象一个库是MongoEngine。MongoEngine是一个对象文档映射器（ODM），它大致相当于一个基于SQL的对象关系映射器（ORM）。
```
pip install mongoengine
```

```
from mongoengine import *
connect('mongoengine_test', host='localhost', port=27017)
```
定义文档
```
import datetime
 
class Post(Document):
    title = StringField(required=True, max_length=200)
    content = StringField(required=True)
    author = StringField(required=True, max_length=50)
    published = DateTimeField(default=datetime.datetime.now)
```
我们甚至可以进一步利用这个并添加更多的限制：

* required：设置必须；
* default：如果没有其他值给出使用指定的默认值
* unique：确保集合中没有其他document有此字段的值相同
* choices：确保该字段的值等于数组中的给定值之一

保存文档
```
post_1 = Post(
    title='Sample Post',
    content='Some engaging content',
    author='Scott'
)
post_1.save()       # This will perform an insert
print(post_1.title)
post_1.title = 'A Better Post Title'
post_1.save()       # This will perform an atomic edit on "title"
print(post_1.title)
```

使用MongoEngine是面向对象的，你也可以添加方法到你的子类文档。例如下面的示例，其中函数用于修改默认查询集（返回集合的所有对象）。通过使用它，**我们可以对类应用默认过滤器，并只获取所需的对象**
```
class Post(Document):
    title = StringField()
    published = BooleanField()

    @queryset_manager
    def live_posts(clazz, queryset):
        return queryset.filter(published=True)
```
**关联其他文档**
您还可以使用`ReferenceField`对象来创建从一个文档到另一个文档的引用。MongoEngine在访问时**自动惰性**处理引用。
```
class Author(Document):
    name = StringField()
 
class Post(Document):
    author = ReferenceField(Author)
 
Post.objects.first().author.name
```


参考：http://python.jobbole.com/87157/