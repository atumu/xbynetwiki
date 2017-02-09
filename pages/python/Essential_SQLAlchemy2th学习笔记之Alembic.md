modifyAt:2016-12-17 19:14:11
title:Essential SQLAlchemy2th学习笔记之Alembic数据库迁移
createAt:2016-12-17 18:27:57
location:python/Essential_SQLAlchemy2th学习笔记之Alembic
author:xbynet

SQLAlchemy默认的create_all()可以增量式创建数据库缺失的表，但是无法做到修改已有的表结构，或删除代码中已经移除的表。这个时候我们就需要用到Alembic这个SQLAlchemy migrations库。
安装：`pip install alembic`
官方文档：http://alembic.zzzcomputing.com/en/latest/
# Creating the Migration Environment
创建一个目录，然后在这个目录下执行
```
alembic init migrations
```
这便创建了migrations目录，该目录结构如下：
├── 
│ ├── README
│ ├── env.py
│ ├── script.py.mako
│ └── versions
└── alembic.ini
env.py配置：

```
sqlalchemy.url = sqlite:///alembictest.db
from app.db import Base
target_metadata = Base.metadata
```

# Building Migrations
自动生成一个Base Empty Migration
```
alembic revision -m "Empty Init"
```
升级数据库到最新版本
```
alembic upgrade head
```
自动生成新版本：
```
alembic revision --autogenerate -m "Added Cookie model"
```
当然你也可以手动修改它。
一些常见的Alembic操作函数如下：

| 函数  | 说明 |
| -------- | -------- | 
|add_column |Adds a new column|
|alter_column |Changes a column type, server default, or name|
|create_check_constraint |Adds a new CheckConstraint|
|create_foreign_key |Adds a new ForeignKey|
|create_index |Adds a new Index|
|create_primary_key |Adds a new PrimaryKey|
|create_table |Adds a new table|
|create_unique_constraint |Adds a new UniqueConstraint|
|drop_column |Removes a column|
|drop_constraint |Removes a constraint|
|drop_index |Drops an index|
|drop_table |Drops a table|
|execute |Run a raw SQL statement|
|rename_table |Renames a table|

**注意：并不是所有的backend database都支持这些函数，比如`SQLite不支持alter_column,drop_column`**

# Controlling Alembic
查看现有数据库的迁移版本：
```
alembic current
```
显示历史
```
alembic history
```

版本降级
```
alembic downgrade 34044511331
```

手动修改数据库的迁移版本：
```
alembic stamp 2e6a6cc63e9
```

**生成SQL**
```
#Upgrading from 34044511331 to 2e6a6cc63e9  sql
alembic upgrade 34044511331:2e6a6cc63e9 --sql
alembic upgrade 34044511331:2e6a6cc63e9 --sql > migration.sql
```

更多参考[这里](https://wiki.xby1993.net/pages/dokuwiki/python/sqlalchemy#_1)

