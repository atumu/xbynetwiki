modifyAt:2016-12-18 02:22:01
title:Essential_SQLAlchemy2th学习笔记
createAt:2016-12-14 17:50:13
location:python/Essential_SQLAlchemy2th学习笔记
author:xbynet

SQL Expression Language对原生SQL语言进行了简单的封装
两大模块SQLAlchemy Core and ORM：

* Core：提供执行SQL Expression Language的接口
* ORM

安装：SQLAlchemy及相关数据库驱动
`pip install sqlalchemy pymysql redis`

# 连接到数据库
数据库连接字符串格式:请参考[这里](http://docs.sqlalchemy.org/en/latest/core/engines.html?highlight=create_engine#database-urls)
```
mysql://username:password@hostname/database
postgresql://username:password@hostname/database
sqlite:////absolute/path/to/database
oracle://scott:tiger@127.0.0.1:1521/orcl
```
比如SQLite如下：
```
from sqlalchemy import create_engine
engine = create_engine('sqlite:///cookies.db')
engine2 = create_engine('sqlite:///:memory:')
engine3 = create_engine('sqlite:////home/cookiemonster/cookies.db')
engine4 = create_engine('sqlite:///c:\\Users\\cookiemonster\\cookies.db')
```

`注意：create_engine函数返回以一个engine实例，但是不会立即获取数据库连接，直到在engine上进行操作如查询时才会去获取connection`

**`关于MySQL空闲连接8小时自动关闭的解决方案：传入 pool_recycle=3600参数`**
```
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://cookiemonster:chocolatechip@mysql01.monster.internal/cookies', pool_recycle=3600)
```

create_engine其余的一些参数：

* echo：是否log打印执行的sql语句及其参数。默认为False
* encoding:默认utf-8
* isolation_level:隔离级别
* pool_recycle：指定连接回收间隔，这对于MySQL连接的8小时机制特别重要。默认-1

获取连接
```
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://cookiemonster:chocolatechip' \
'@mysql01.monster.internal/cookies', pool_recycle=3600)
connection = engine.connect()
```

# Schema and Types
四种类型集合：
• Generic
• SQL standard
• Vendor specific
• User defined

SQLAlchemy定义了很多generic types以兼容不同数据库。这些类型都定义在`sqlalchemy.types`模块中，为了方便也可以从sqlalchemy直接导入这些类型。
类型对应表如下：

|SQLAlchemy |Python |SQL|
| -------- | -------- | -------- |
|BigInteger |int |BIGINT|
|Boolean| bool |BOOLEAN or SMALLINT|
|Date |datetime.date |DATE (SQLite: STRING)|
|DateTime |datetime.datetime |DATETIME (SQLite: STRING)|
|Enum |str |ENUM or VARCHAR|
|Float |float or Decimal |FLOAT or REAL|
|Integer |int |INTEGER|
|Interval |datetime.timedelta |INTERVAL or DATE from epoch|
|LargeBinary |byte |BLOB or BYTEA|
|Numeric |decimal.Decimal |NUMERIC or DECIMAL|
|Unicode |unicode |UNICODE or VARCHAR|
|Text |str |CLOB or TEXT|
|Time |datetime.time |DATETIME|

如果这些类型不能满足你，比如有些数据库支持json类型，那么你需要用到`sqlalchemy.dialects`模块中对应数据库的类型。比如`from sqlalchemy.dialects.postgresql import JSON`

## Metadata & Table & Column
Metadata为了快速访问数据库。可以看作是很多Table对象的集合，还有一些关于engin,connection的信息。可以通过`MetaData.tables`访问这些表对象字典
定义表对象之前需要先实例化Metadata:
```
from sqlalchemy import MetaData
metadata = MetaData()
```

Table对象构建如下：第一个参数为名称，第二个参数为Metadata对象，后续参数为Column对象. Column对象参数为,名称，类型，及其余等
```
from sqlalchemy import Table, Column, Integer, Numeric, String, ForeignKey
cookies = Table('cookies', metadata,
Column('cookie_id', Integer(), primary_key=True),
Column('cookie_name', String(50), index=True),
Column('cookie_recipe_url', String(255)),
Column('cookie_sku', String(55)),
Column('quantity', Integer()),
Column('unit_cost', Numeric(12, 2))
)
```

```
from datetime import datetime
from sqlalchemy import DateTime
users = Table('users', metadata,
Column('user_id', Integer(), primary_key=True),
Column('username', String(15), nullable=False, unique=True),
Column('email_address', String(255), nullable=False),
Column('phone', String(20), nullable=False),
Column('password', String(25), nullable=False),
Column('created_on', DateTime(), default=datetime.now),
Column('updated_on', DateTime(), default=datetime.now, onupdate=datetime.now)
```
**注意：这里default,onupdate属性是一个callable对象而不是直接值，比如datetime.now(),因为这样的话，就永远是这个值，而不是每个实例实例化、更新时的时间了。**
比较有用的就是`onupdate`,每次更新时都会调用该方法或函数。

**键和约束(Keys and Constraints)**
键和约束既可以像上面那样通过kwargs定义在Column中，也可以在之后通过对象添加。相关类定义在基础的 sqlalchemy模块中，比如最常用的三个：
`from sqlalchemy import PrimaryKeyConstraint, UniqueConstraint, CheckConstraint`
```
PrimaryKeyConstraint('user_id', name='user_pk'),它也支持同时定义多个形成联合主键。
UniqueConstraint('username', name='uix_username')
CheckConstraint('unit_cost >= 0.00', name='unit_cost_positive')
```

**索引(Index)**
```
from sqlalchemy import Index
Index('ix_cookies_cookie_name', 'cookie_name')
```
这个定义需要放置在Table构造器中。也可以在之后定义，比如
`Index('ix_test', mytable.c.cookie_sku, mytable.c.cookie_name))`

**关联关系和外键约束(Relationships and ForeignKeyConstraints)**

```
from sqlalchemy import ForeignKey
orders = Table('orders', metadata,
Column('order_id', Integer(), primary_key=True),
Column('user_id', ForeignKey('users.user_id')),
Column('shipped', Boolean(), default=False)
)
line_items = Table('line_items', metadata,
Column('line_items_id', Integer(), primary_key=True),
Column('order_id', ForeignKey('orders.order_id')),
Column('cookie_id', ForeignKey('cookies.cookie_id')),
Column('quantity', Integer()),
Column('extended_cost', Numeric(12, 2))
)
```

**注意：这里ForeignKey用的是字符串参数(这些字符串对应的是数据库中的表名.列名)，而非引用。这样隔离了模块间相互依赖**
我们也可以使用：
`ForeignKeyConstraint(['order_id'], ['orders.order_id'])`

创建或持久化表模式(Persisting the Tables)
通过示例代码我们知道所有的Table定义，以及额外的模式定义都会与一个metadata对象关联。我们可以通过这个metadata对象来创建表：

```
metadata.create_all(engine)
```

注意：**默认情况下create_all`不会`重新创建已有表**，所以**它可以安全地多次调用**，而且也非常友好地与数据库迁移库如Ablembic集成而不需要你进行额外手动编码。

本节代码完整如下：

```
from datetime import datetime
from sqlalchemy import (MetaData, Table, Column, Integer, Numeric, String,
DateTime, ForeignKey, create_engine)
metadata = MetaData()
cookies = Table('cookies', metadata,
Column('cookie_id', Integer(), primary_key=True),
Column('cookie_name', String(50), index=True),
Column('cookie_recipe_url', String(255)),
Column('cookie_sku', String(55)),
Column('quantity', Integer()),
Column('unit_cost', Numeric(12, 2))
)
users = Table('users', metadata,
Column('user_id', Integer(), primary_key=True),
Column('customer_number', Integer(), autoincrement=True),
Column('username', String(15), nullable=False, unique=True),
Column('email_address', String(255), nullable=False),
Column('phone', String(20), nullable=False),
Column('password', String(25), nullable=False),
Column('created_on', DateTime(), default=datetime.now),
Column('updated_on', DateTime(), default=datetime.now, onupdate=datetime.now)
)
orders = Table('orders', metadata,
Column('order_id', Integer(), primary_key=True),
Column('user_id', ForeignKey('users.user_id'))
)
line_items = Table('line_items', metadata,
Column('line_items_id', Integer(), primary_key=True),
Column('order_id', ForeignKey('orders.order_id')),
Column('cookie_id', ForeignKey('cookies.cookie_id')),
Column('quantity', Integer()),
Column('extended_cost', Numeric(12, 2))
)
engine = create_engine('sqlite:///:memory:')
metadata.create_all(engine)
```

# SQLAlchemy-Core模块
## 插入数据：
```
ins = cookies.insert().values(
cookie_name="chocolate chip",
cookie_recipe_url="http://some.aweso.me/cookie/recipe.html",
cookie_sku="CC01",
quantity="12",
unit_cost="0.50"
)
print(str(ins))
```
当然你也可以这么做：
```
from sqlalchemy import insert
ins = insert(cookies).values(
cookie_name="chocolate chip",
cookie_recipe_url="http://some.aweso.me/cookie/recipe.html",
cookie_sku="CC01",
quantity="12",
unit_cost="0.50"
)
```
上述编译成预编译语句如下：
```
INSERT INTO cookies
(cookie_name, cookie_recipe_url, cookie_sku, quantity, unit_cost)
VALUES
(:cookie_name, :cookie_recipe_url, :cookie_sku, :quantity, :unit_cost)
```
实际过程会是如下ins对象内部会调用compile()方法编译成上述语句，然后将参数存储到ins.compile().params字典中。
接下来我们通过前面获取的connection对象执行statement：
```
result = connection.execute(ins)
```

当然你也可以这么查询：

```python
ins = cookies.insert()
result = connection.execute(
ins,
cookie_name='dark chocolate chip',
cookie_recipe_url='http://some.aweso.me/cookie/recipe_dark.html',
cookie_sku='CC02',
quantity='1',
unit_cost='0.75'
)
result.inserted_primary_key
```

### 批量插入：

```
inventory_list = [
{
'cookie_name': 'peanut butter',
'cookie_recipe_url': 'http://some.aweso.me/cookie/peanut.html',
'cookie_sku': 'PB01',
'quantity': '24',
'unit_cost': '0.25'
},
{
'cookie_name': 'oatmeal raisin',
'cookie_recipe_url': 'http://some.okay.me/cookie/raisin.html',
'cookie_sku': 'EWW01',
'quantity': '100',
'unit_cost': '1.00'
}
]
result = connection.execute(ins, inventory_list)
```
**注意：一定要确保所有字典参数拥有相同的keys**


## 查询

```
from sqlalchemy.sql import select
s = select([cookies])
rp = connection.execute(s)
results = rp.fetchall()
```
当然我们也可以使用字符串来代替:
```
s = select("""SELECT cookies.cookie_id, cookies.cookie_name,
cookies.cookie_recipe_url, cookies.cookie_sku, cookies.quantity,
cookies.unit_cost FROM cookies""")
```

connection.execute返回的rp变量是一个`ResultProxy`对象(它是DBAPI中cursor对象的封装)。

我们也可以这样写：

```
from sqlalchemy.sql import select
s = cookies.select()
rp = connection.execute(s)
results = rp.fetchall()
```

ResultProxy使得查询结果可以通过index,name,or Column object访问列数据。例如：

```
first_row = results[0]
first_row[1] #游标列索引从1开始,by index
first_row.cookie_name # by name
first_row[cookies.c.cookie_name] #by Column object.
```

你也可以**迭代ResultProxy**,如下：

```
rp = connection.execute(s)
for record in rp:
print(record.cookie_name)
```

ResultProxy其余可用来获取结果集的方法

* first()
* fetchone()
* fetchall()
* scalar():Returns a single value if a query results in a single record with one column.
* keys() 获取列名

**关于选择ResultProxy上述的方法的建议：**
1、使用first()而不是fetchone()来获取单条记录，因为fetchone()调用之后仍然保留着打开的connections共后续使用，如果不小心的话很容易引起问题。
2、使用迭代方式获取所有结果，而不是fetchall(),更加省内存。
3、使用scalar()获取单行单列结果时需要注意，如果返回多于一行，它会抛出异常。

**控制返回列的数目**

```
s = select([cookies.c.cookie_name, cookies.c.quantity])
rp = connection.execute(s)
print(rp.keys())
result = rp.first()
```

**排序**

```
s = select([cookies.c.cookie_name, cookies.c.quantity])
s = s.order_by(cookies.c.quantity)
rp = connection.execute(s)
for cookie in rp:
print('{} - {}'.format(cookie.quantity, cookie.cookie_name))

#倒序desc
from sqlalchemy import desc
s = select([cookies.c.cookie_name, cookies.c.quantity])
s = s.order_by(desc(cookies.c.quantity))
```

**限制返回结果集的条数**

```
s = select([cookies.c.cookie_name, cookies.c.quantity])
s = s.order_by(cookies.c.quantity)
s = s.limit(2)
rp = connection.execute(s)
print([result.cookie_name for result in rp])
```


### 内置SQL函数
在sqlalchemy.sql.func模块中

```
#sum
from sqlalchemy.sql import func
s = select([func.sum(cookies.c.quantity)])
rp = connection.execute(s)
print(rp.scalar())

#count
s = select([func.count(cookies.c.cookie_name)])
rp = connection.execute(s)
record = rp.first()
print(record.keys())
print(record.count_1) #字段名是自动生成的，<func_name>_<position>，可以设置别名的，看下面

#设置别名
s = select([func.count(cookies.c.cookie_name).label('inventory_count')])
rp = connection.execute(s)
record = rp.first()
print(record.keys())
print(record.inventory_count)
```


-----


**过滤**

```
#where
s = select([cookies]).where(cookies.c.cookie_name == 'chocolate chip')
rp = connection.execute(s)
record = rp.first()
print(record.items()) #调用row对象的items()方法。

#like
s = select([cookies]).where(cookies.c.cookie_name.like('%chocolate%'))
rp = connection.execute(s)
for record in rp.fetchall():
    print(record.cookie_name)
```
可以在where中使用的子句元素

* between(cleft, cright) 
* concat(column_two) Concatenate column with column_two
* distinct() 
* in_([list]) 
* is_(None) Find where the column is None (commonly used for Null checks with None)
* contains(string) Find where the column has string in it (case-sensitive)
* endswith(string) Find where the column ends with string (case-sensitive)
* like(string) Find where the column is like string (case-sensitive)
* startswith(string) Find where the column begins with string (case-sensitive)
* ilike(string) Find where the column is like string (this is not case-sensitive)

当然还包括一系列的**notxxx**方法，比如notin_(),唯一的例外是**isnot()**

**操作符**

* +，-，*,/,%
* ==,!=,<,>,<=,>=
* AND,OR,NOT,由于python关键字的原因，使用and_(),or_(),not_()来代替

+号还可以用于字符串拼接：

```
s = select([cookies.c.cookie_name, 'SKU-' + cookies.c.cookie_sku])
for row in connection.execute(s):
print(row)
```

```
from sqlalchemy import cast
s = select([cookies.c.cookie_name,
    cast((cookies.c.quantity * cookies.c.unit_cost),
        Numeric(12,2)).label('inv_cost')])
for row in connection.execute(s):
    print('{} - {}'.format(row.cookie_name, row.inv_cost))
```
注意：cast是另外一个函数，允许我们进行类型转换，上述转换是将数字转换为货币形式，和
print('{} - {:.2f}'.format(row.cookie_name, row.inv_cost)).这个行为一致。

```
from sqlalchemy import and_, or_, not_
s = select([cookies]).where(
    and_(
        cookies.c.quantity > 23,
        cookies.c.unit_cost < 0.40
    )
)
for row in connection.execute(s):
    print(row.cookie_name)


from sqlalchemy import and_, or_, not_
s = select([cookies]).where(
    or_(
        cookies.c.quantity.between(10, 50),
        cookies.c.cookie_name.contains('chip')
    )
)
for row in connection.execute(s):
    print(row.cookie_name)

```

## update
```
from sqlalchemy import update
u = update(cookies).where(cookies.c.cookie_name == "chocolate chip")
u = u.values(quantity=(cookies.c.quantity + 120))
result = connection.execute(u)
print(result.rowcount)
s = select([cookies]).where(cookies.c.cookie_name == "chocolate chip")
result = connection.execute(s).first()
for key in result.keys():
    print('{:>20}: {}'.format(key, result[key]))
```

## delete
```
from sqlalchemy import delete
u = delete(cookies).where(cookies.c.cookie_name == "dark chocolate chip")
result = connection.execute(u)
print(result.rowcount)

s = select([cookies]).where(cookies.c.cookie_name == "dark chocolate chip")
result = connection.execute(s).fetchall()
print(len(result))
```

## joins
join()，outerjoin()函数，select_from()函数

```
columns = [orders.c.order_id, users.c.username, users.c.phone,
           cookies.c.cookie_name, line_items.c.quantity,
           line_items.c.extended_cost]
cookiemon_orders = select(columns)
cookiemon_orders = cookiemon_orders.select_from(orders.join(users).join(
    line_items).join(cookies)).where(users.c.username ==
                                     'cookiemon')
result = connection.execute(cookiemon_orders).fetchall()
for row in result:
    print(row)

```

最终产生的SQL语句如下：

```
SELECT orders.order_id, users.username, users.phone, cookies.cookie_name,
line_items.quantity, line_items.extended_cost FROM users JOIN orders ON
users.user_id = orders.user_id JOIN line_items ON orders.order_id =
line_items.order_id JOIN cookies ON cookies.cookie_id = line_items.cookie_id
WHERE users.username = :username_1
```

outerjoin

```
columns = [users.c.username, func.count(orders.c.order_id)]
all_orders = select(columns)
all_orders = all_orders.select_from(users.outerjoin(orders))
all_orders = all_orders.group_by(users.c.username)
result = connection.execute(all_orders).fetchall()
for row in result:
    print(row)
```

表别名函数alias()

```
>>> manager = employee_table.alias('mgr')
>>> stmt = select([employee_table.c.name],
            ... and_(employee_table.c.manager_id==manager.c.id,
            ... manager.c.name=='Fred'))
>>> print(stmt)
SELECT employee.name
FROM employee, employee AS mgr
WHERE employee.manager_id = mgr.id AND mgr.name = ?
```

## 分组
```
columns = [users.c.username, func.count(orders.c.order_id)]
all_orders = select(columns)
all_orders = all_orders.select_from(users.outerjoin(orders))
all_orders = all_orders.group_by(users.c.username)
result = connection.execute(all_orders).fetchall()
for row in result:
    print(row)
```

## chaining
```
def get_orders_by_customer(cust_name, shipped=None, details=False):
    columns = [orders.c.order_id, users.c.username, users.c.phone]
    joins = users.join(orders)
    if details:
        columns.extend([cookies.c.cookie_name, line_items.c.quantity,
            line_items.c.extended_cost])
        joins = joins.join(line_items).join(cookies)
    cust_orders = select(columns)
    cust_orders = cust_orders.select_from(joins)

    cust_orders = cust_orders.where(users.c.username == cust_name)
    if shipped is not None:
        cust_orders = cust_orders.where(orders.c.shipped == shipped)
    result = connection.execute(cust_orders).fetchall()
    return result
```

## 执行原生SQL
返回的还是ResultProxy对象
1、完全采用原始SQL
```
result = connection.execute("select * from orders").fetchall()
print(result)
```

2、部分采用原始SQL，text()函数

```
from sqlalchemy import text
stmt = select([users]).where(text("username='cookiemon'"))
print(connection.execute(stmt).fetchall())
```

## 异常
SQLALchemy定义了很多异常。我们通过关心：AttributeErrors，IntegrityErrors.等
为了进行相关试验与说明，请先执行下面这些语句

```
from datetime import datetime
from sqlalchemy import (MetaData, Table, Column, Integer, Numeric, String,
                        DateTime, ForeignKey, Boolean, create_engine,
                        CheckConstraint)
metadata = MetaData()
cookies = Table('cookies', metadata,
                Column('cookie_id', Integer(), primary_key=True),
                37
                Column('cookie_name', String(50), index=True),
                Column('cookie_recipe_url', String(255)),
                Column('cookie_sku', String(55)),
                Column('quantity', Integer()),
                Column('unit_cost', Numeric(12, 2)),
                CheckConstraint('quantity > 0', name='quantity_positive')
                )
users = Table('users', metadata,
              Column('user_id', Integer(), primary_key=True),
              Column('username', String(15), nullable=False, unique=True),
              Column('email_address', String(255), nullable=False),
              Column('phone', String(20), nullable=False),
              Column('password', String(25), nullable=False),
              Column('created_on', DateTime(), default=datetime.now),
              Column('updated_on', DateTime(),
                     default=datetime.now, onupdate=datetime.now)
              )
orders = Table('orders', metadata,
               Column('order_id', Integer()),
               Column('user_id', ForeignKey('users.user_id')),
               Column('shipped', Boolean(), default=False)
               )
line_items = Table('line_items', metadata,
                   Column('line_items_id', Integer(), primary_key=True),
                   Column('order_id', ForeignKey('orders.order_id')),
                   Column('cookie_id', ForeignKey('cookies.cookie_id')),
                   Column('quantity', Integer()),
                   Column('extended_cost', Numeric(12, 2))
                   )
engine = create_engine('sqlite:///:memory:')
metadata.create_all(engine)
connection = engine.connect()

```

```
from sqlalchemy import select, insert
ins = insert(users).values(
username="cookiemon",
email_address="mon@cookie.com",
phone="111-111-1111",
password="password"
)
result = connection.execute(ins)
s = select([users.c.username])
results = connection.execute(s)
for result in results:
print(result.username)
print(result.password) #此处包AttributeError异常
```

在违反约束的情况下会出现IntegrityError异常。比如违反唯一性约束等。

```
s = select([users.c.username])
connection.execute(s).fetchall()
[(u'cookiemon',)]
ins = insert(users).values(
    username="cookiemon",
    email_address="damon@cookie.com",
    phone="111-111-1111",
    password="password"
)
result = connection.execute(ins) #此处报IntegrityError, UNIQUE constraint failed: users.username
#异常处理
try:
    result = connection.execute(ins)
except IntegrityError as error:
    print(error.orig.message, error.params)
```
所有的SQLAlchemy异常处理方式都是上面那种思路，通过[SQLAlchemyError](http://docs.sqlal
chemy.org/en/latest/core/exceptions.html)可以获取到的信息由如下：

* orig ：The DBAPI exception object.
* params：The parameter list being used when this exception occurred.
* statement ：The string SQL statement being invoked when this exception occurred.

## 事务Transactions
![013cd282-c420-11e6-b432-00163e2eed34.png](/data/upload/013cd282-c420-11e6-b432-00163e2eed34.png)
![0bac9bbc-c420-11e6-b432-00163e2eed34.png](/data/upload/0bac9bbc-c420-11e6-b432-00163e2eed34.png)

```
from sqlalchemy.exc import IntegrityError


def ship_it(order_id):
    s = select([line_items.c.cookie_id, line_items.c.quantity])
    s = s.where(line_items.c.order_id == order_id)
    transaction = connection.begin() #开启事务
    cookies_to_ship = connection.execute(s).fetchall()
    try:
        for cookie in cookies_to_ship:
            u = update(cookies).where(cookies.c.cookie_id == cookie.cookie_id)
            u = u.values(quantity=cookies.c.quantity - cookie.quantity)
            result = connection.execute(u)
        u = update(orders).where(orders.c.order_id == order_id)
        u = u.values(shipped=True)
        result = connection.execute(u)
        print("Shipped order ID: {}".format(order_id))
        transaction.commit() #提交事务
    except IntegrityError as error:
        transaction.rollback()   #事务回滚
        print(error)

```
