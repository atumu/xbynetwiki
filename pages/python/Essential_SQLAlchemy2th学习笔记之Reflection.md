modifyAt:2016-12-17 16:48:53
title:Essential SQLAlchemy2th学习笔记之反射Reflection
createAt:2016-12-17 14:56:30
location:python/Essential_SQLAlchemy2th学习笔记之Reflection
author:xbynet

示例数据库下载:http://chinookdatabase.codeplex.com/.
在SQLALchemy中，我们使用反射技术来获取相关database schema信息,如tables,views,indexes等等
# Core模块反射
## 反射单表
```
from sqlalchemy import MetaData, create_engine
metadata = MetaData()
engine = create_engine('sqlite:///Chinook_Sqlite.sqlite')
from sqlalchemy import Table
#这里我们并没有使用Column来定义列，而是使用autoload,autoload_with参数来从已有数据表中反射出相关列定义。
artist = Table('Artist', metadata, autoload=True, autoload_with=engine)
#接下来使用反射出来的表对象进行相关查询
artist.columns.keys() #列出所有的列名
from sqlalchemy import select
s = select([artist]).limit(10)
engine.execute(s).fetchall()
#metadata.tables['Artist']
#artist.foreign_keys
#from sqlalchemy import ForeignKeyConstraint
#album.append_constraint(ForeignKeyConstraint(['ArtistId'], ['artist.ArtistId']))
#str(artist.join(album))
```

## 反射整个数据库
```
#使用reflect()方法，它不会返回任何值
metadata.reflect(bind=engine)
#但是我们仍然可以进行相关检索
metadata.tables.keys() #获取所有的表名
```

注意：从SQLAlchemy1.0版本开始，我们不能通过反射获取CheckConstraints, comments, or triggers.You also can’t reflect client-side defaults or an association between a sequence and a column.(这意味着我们会失去注释，或者表关联关系)但是我们可以通过相关方法或函数手动添加它们。

## 基于反射对象进行查询
```
playlist = metadata.tables['Playlist']
from sqlalchemy import select
s = select([playlist]).limit(10)
engine.execute(s).fetchall()
```

# ORM模块反射
## Reflecting a Database with Automap
这里我们不再使用declarative_base而是使用扩展模块的automap_base

```
from sqlalchemy.ext.automap import automap_base
Base = automap_base()
from sqlalchemy import create_engine
engine = create_engine('sqlite:///Chinook_Sqlite.sqlite')
Base.prepare(engine, reflect=True)
Base.classes.keys() #获取所有的对象名
#获取表对象
Artist = Base.classes.Artist
Album = Base.classes.Album

#进行操作
from sqlalchemy.orm import Session
session = Session(engine)
for artist in session.query(Artist).limit(10):
    print(artist.ArtistId, artist.Name)
```

## Reflected Relationships反射关联关系
Automap可以反射并建立表之间的many-to-one, one-to-many, and many-to-many relationships.
但是建立关联列的命名为< related_object>_collection例如：

```
artist = session.query(Artist).first()
for album in artist.album_collection:
    print('{} - {}'.format(artist.Name, album.Title))
```

关于Automap更多信息请详细参看[官方文档](http://docs.sqlalchemy.org/en/latest/orm/extensions/automap.html)
