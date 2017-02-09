modifyAt:2016-12-18 00:59:32
title:Essential SQLAlchemy2th学习笔记之ORM
createAt:2016-12-17 16:10:42
location:python/Essential_SQLAlchemy2th学习笔记之ORM
author:xbynet

# 定义模式Defining Schema
 定义ORM类的4个步骤：
 
* 继承declarative_base()函数返回的类
* 定义__tablename__属性来指定表名
* 定义列属性
* 定义至少一个主键

```
from sqlalchemy import Table, Column, Integer, Numeric, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Cookie(Base):
    __tablename__ = 'cookies'
    
    cookie_id = Column(Integer(), primary_key=True)
    cookie_name = Column(String(50), index=True)
    cookie_recipe_url = Column(String(255))
    cookie_sku = Column(String(55))
    quantity = Column(Integer())
    unit_cost = Column(Numeric(12, 2))

```

你可以查看Cookie类的__table__属性：如下

```
>>> Cookie.__table__
Table('cookies', MetaData(bind=None),
    Column('cookie_id', Integer(), table=<cookies>, primary_key=True,
        nullable=False),
    Column('cookie_name', String(length=50), table=<cookies>),
    Column('cookie_recipe_url', String(length=255), table=<cookies>),
    Column('cookie_sku', String(length=15), table=<cookies>),
    Column('quantity', Integer(), table=<cookies>),
    Column('unit_cost', Numeric(precision=12, scale=2),
        table=<cookies>), schema=None)
```

## Keys, Constraints, and Indexes
```
class SomeDataClass(Base):
__tablename__ = 'somedatatable'
__table_args__ = (ForeignKeyConstraint(['id'], ['other_table.id']),
                  CheckConstraint(unit_cost >= 0.00',
                                  name='unit_cost_positive'))

```

## Relationships关联关系
一对多关系
```
from sqlalchemy import ForeignKey, Boolean
from sqlalchemy.orm import relationship, backref

class Order(Base):
    __tablename__ = 'orders'
    order_id = Column(Integer(), primary_key=True)
    #定义外键
    user_id = Column(Integer(), ForeignKey('users.user_id'))
    shipped = Column(Boolean(), default=False)
    #定义one-to-many关系
    user = relationship("User", backref=backref('orders', order_by=order_id))

```

一对一关系

```
class LineItem(Base):
    __tablename__ = 'line_items'
    line_item_id = Column(Integer(), primary_key=True)
    order_id = Column(Integer(), ForeignKey('orders.order_id'))
    cookie_id = Column(Integer(), ForeignKey('cookies.cookie_id'))
    quantity = Column(Integer())
    extended_cost = Column(Numeric(12, 2))
    order = relationship("Order", backref=backref('line_items',
                                                  order_by=line_item_id))
    #定义one-to-one关系，uselist=False
    cookie = relationship("Cookie", uselist=False)

```

多对多关系

```
engine = create_engine('sqlite:///:memory:')
Base = declarative_base()
Session = sessionmaker(bind=engine)

#定义多对多关系的中间表
cookieingredients_table = Table('cookieingredients', Base.metadata,
    Column('cookie_id', Integer, ForeignKey("cookies.cookie_id"),
        primary_key=True),
    Column('ingredient_id', Integer, ForeignKey("ingredients.ingredient_id"),
        primary_key=True)
)

class Ingredient(Base):
    __tablename__ = 'ingredients'
    ingredient_id = Column(Integer, primary_key=True)
    name = Column(String(255), index=True)

    def __repr__(self):
        return "Ingredient(name='{self.name}')".format(self=self)


class Cookie(Base):
    __tablename__ = 'cookies'
    cookie_id = Column(Integer, primary_key=True)
    cookie_name = Column(String(50), index=True)
    cookie_recipe_url = Column(String(255))
    cookie_sku = Column(String(55))
    quantity = Column(Integer())
    unit_cost = Column(Numeric(12, 2))
    ingredients = relationship("Ingredient",
                               secondary=cookieingredients_table)

```

## Persisting the Schema

```
from sqlalchemy import create_engine
engine = create_engine('sqlite:///:memory:')
Base.metadata.create_all(engine) #这个Base是前面的Base = declarative_base()
```

# Working with Data via SQLAlchemy ORM
# Session
session对象负责与数据库交互，封装了来自engine的connection，transaction.session中的事物会一直打开，除非调用session的commit()或rollback()方法，或close(),remove()方法。
你应该使用sessionmaker工厂类来创建Session类，因为这确保了配置参数的正确性。一个应用应该只调用sessionmaker一次。

```
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
engine = create_engine('sqlite:///:memory:')
Session = sessionmaker(bind=engine)
session = Session()
```

session对象包含创建数据库连接所需的一切信息，它不会立即创建连接对象，而是会在我们进行具体操作时创建。


## 插入数据
```
cc_cookie = Cookie(cookie_name='chocolate chip',
cookie_recipe_url='http://some.aweso.me/cookie/recipe.html',
cookie_sku='CC01',
quantity=12,
unit_cost=0.50)
session.add(cc_cookie)
session.commit()
```

当我们调用add()的时候，它不会在数据库执行insert操作，而当我们调用commit()的时候，将会发生如下步骤：

```
#start a transaction.
INFO:sqlalchemy.engine.base.Engine:BEGIN (implicit)
#Insert the record into the database
INFO:sqlalchemy.engine.base.Engine:INSERT INTO cookies (cookie_name,
cookie_recipe_url, cookie_sku, quantity, unit_cost) VALUES (?, ?, ?, ?, ?)

#The values for the insert.
INFO:sqlalchemy.engine.base.Engine:('chocolate chip',
'http://some.aweso.me/cookie/recipe.html', 'CC01', 12, 0.5) 
#Commit the transaction.
INFO:sqlalchemy.engine.base.Engine:COMMIT
```
如果你想打印这些细节信息，你可以传递`echo=True`到`create_engine`函数中。注意生产环境不要使用这个选项。

## 批量插入
[文档地址](http://docs.sqlalchemy.org/en/latest/orm/persistence_techniques.html#bulk-operations)
Session.bulk_save_objects()，Session.bulk_update_mappings()

```
c1 = Cookie(cookie_name='peanut butter',
            cookie_recipe_url='http://some.aweso.me/cookie/peanut.html',
            cookie_sku='PB01',
            quantity=24,
            unit_cost=0.25)
c2 = Cookie(cookie_name='oatmeal raisin',
            cookie_recipe_url='http://some.okay.me/cookie/raisin.html',
            cookie_sku='EWW01',
            quantity=100,
            unit_cost=1.00)
session.bulk_save_objects([c1, c2])
session.commit()
print(c1.cookie_id)

```

除了bulk_save_objects，还有Session.bulk_update_mappings(), 如下：
它允许我们通过字典列表来进行插入

```
s.bulk_insert_mappings(User,
  [dict(name="u1"), dict(name="u2"), dict(name="u3")]
)
```

对于批量更新，还有个`Session.bulk_update_mappings()`

## 查询
```
cookies = session.query(Cookie).all()
print(cookies)

#使用迭代方式
for cookie in session.query(Cookie):
    print(cookie)
```

其余方法：

* all()
* first()
* one():如果有多条结果，会抛出异常。
* scalar()

关于选择的最佳实践：
1、使用迭代方式获取所有值，而不是all()。内存友好
2、使用first()获取单条数据，而不是one(),scalar()
3、尽量不要使用scalar()

控制查询的列数目

```
print(session.query(Cookie.cookie_name, Cookie.quantity).first())
```

排序

```
for cookie in session.query(Cookie).order_by(Cookie.quantity):
    print('{:3} - {}'.format(cookie.quantity, cookie.cookie_name))
    
from sqlalchemy import desc
for cookie in session.query(Cookie).order_by(desc(Cookie.quantity)):
    print('{:3} - {}'.format(cookie.quantity, cookie.cookie_name))
```

limiting限制返回的结果数

```
query = session.query(Cookie).order_by(Cookie.quantity).limit(2)
print([result.cookie_name for result in query])
```

内置SQL函数与别名

```
from sqlalchemy import func
inv_count = session.query(func.sum(Cookie.quantity)).scalar()
print(inv_count)

rec_count = session.query(func.count(Cookie.cookie_name)).first()
print(rec_count) #(5,) 得到的是一个元组，而不是像scalar()那样得到单个值

#别名
rec_count = session.query(func.count(Cookie.cookie_name) \
.label('inventory_count')).first()
print(rec_count.keys())
print(rec_count.inventory_count)
```

### 过滤
```
record = session.query(Cookie).filter(Cookie.cookie_name == 'chocolate chip').first()
print(record)
record = session.query(Cookie).filter_by(cookie_name='chocolate chip').first()
print(record)
```

注意：filter与filter_by的区别

```
query = session.query(Cookie).filter(Cookie.cookie_name.like('%chocolate%'))
for record in query:
    print(record.cookie_name)
```

操作符

* +，-，*,/,%
* ==,!=,<,>,<=,>=
* AND,OR,NOT,由于python关键字的原因，使用and_(),or_(),not_()来代替

+号还可以用于字符串拼接：

```
results = session.query(Cookie.cookie_name, 'SKU-' + Cookie.cookie_sku).all()
for row in results:
    print(row)
    
from sqlalchemy import and_, or_, not_
query = session.query(Cookie).filter(or_(
    Cookie.quantity.between(10, 50),
    Cookie.cookie_name.contains('chip')
    )
)
for result in query:
    print(result.cookie_name)
```

## 更新Updating Data
```
query = session.query(Cookie)
cc_cookie = query.filter(Cookie.cookie_name == "chocolate chip").first()
cc_cookie.quantity = cc_cookie.quantity + 120
session.commit()
print(cc_cookie.quantity)


#通过字典方式更新
query = session.query(Cookie)
query = query.filter(Cookie.cookie_name == "chocolate chip")
query.update({Cookie.quantity: Cookie.quantity - 20})
cc_cookie = query.first()
print(cc_cookie.quantity)
```

批量更新，请用Session.bulk_update_mappings()

## 删除Deleting Data
```
query = session.query(Cookie)
query = query.filter(Cookie.cookie_name == "dark chocolate chip")
dcc_cookie = query.one()
session.delete(dcc_cookie)
session.commit()
dcc_cookie = query.first()
print(dcc_cookie)

#或者这样
query = session.query(Cookie)
query = query.filter(Cookie.cookie_name == "molasses")
query.delete()
```

添加关联对象

```
o1 = Order()
o1.user = cookiemon
session.add(o1)
cc = session.query(Cookie).filter(Cookie.cookie_name ==
                                  "chocolate chip").one()
line1 = LineItem(cookie=cc, quantity=2, extended_cost=1.00)
pb = session.query(Cookie).filter(Cookie.cookie_name ==
                                  "peanut butter").one()
line2 = LineItem(quantity=12, extended_cost=3.00)
line2.cookie = pb
line2.order = o1
o1.line_items.append(line1)
o1.line_items.append(line2)
session.commit()

```

## Joins
```
query = session.query(Order.order_id, User.username, User.phone,
                      Cookie.cookie_name, LineItem.quantity,
                      LineItem.extended_cost)
query = query.join(User).join(LineItem).join(Cookie)
results = query.filter(User.username == 'cookiemon').all()
print(results)


query = session.query(User.username, func.count(Order.order_id))
query = query.outerjoin(Order).group_by(User.username)
for row in query:
    print(row)
```

### 自关联表的定义

```
class Employee(Base):
    __tablename__ = 'employees'
    id = Column(Integer(), primary_key=True)
    manager_id = Column(Integer(), ForeignKey('employees.id'))
    name = Column(String(255), nullable=False)
    manager = relationship("Employee", backref=backref('reports'),
                           remote_side=[id])
Base.metadata.create_all(engine)

```

注：使用remote_side来定义自关联的多对一关系

```
marsha = Employee(name='Marsha')
fred = Employee(name='Fred')
marsha.reports.append(fred)
session.add(marsha)
session.commit()

for report in marsha.reports:
    print(report.name)
```

## 分组
```
query = session.query(User.username, func.count(Order.order_id))
query = query.outerjoin(Order).group_by(User.username)
for row in query:
    print(row)
```

## Chaining
```
def get_orders_by_customer(cust_name):
    query = session.query(Order.order_id, User.username, User.phone,
                          Cookie.cookie_name, LineItem.quantity,
                          LineItem.extended_cost)
    query = query.join(User).join(LineItem).join(Cookie)
    results = query.filter(User.username == cust_name).all()
    return results
get_orders_by_customer('cakeeater')

```

## 元素SQL查询
```
session.execute('select * from User')
session.execute("insert into User(name, age) values('bomo', 13)")
session.execute("insert into User(name, age) values(:name, :age)", {'name': 'bomo', 'age':12})
```

建议使用text()来执行部分SQL查询
```
from sqlalchemy import text
query = session.query(User).filter(text("username='cookiemon'"))
print(query.all())

[User(username='cookiemon', email_address='mon@cookie.com',
      phone='111-111-1111', password='password')]

```

# Session与异常处理
Session状态：

* Transient：实例不在session和数据库中。
* Pending：对象通过add()方法被添加到session当中，但是并没有flushed或者committed
* Persistent:对象处于session中，同时在数据库中有对应的记录
* Detached:实例不在session中，但是数据库中有相关记录

那么如何查看实例状态呢？可以通过SQLAlchemy的`inspect()`方法来查看,

```
cc_cookie = Cookie('chocolate chip',
                   'http://some.aweso.me/cookie/recipe.html',
                   'CC01', 12, 0.50)
from sqlalchemy import inspect
insp = inspect(cc_cookie)
for state in ['transient', 'pending', 'persistent', 'detached']:
    print('{:>10}: {}'.format(state, getattr(insp, state)))

```
输出：
transient: True
pending: False
persistent: False
detached: False
实际上，你应该使用insp.transient, insp.pending, insp.persistent, and insp.detached来获取某一个状态。

如果要将一个实例变为detached状态，可以调用`session的expunge()方法`

```
session.expunge(cc_cookie)
```

查看改变历史

```
for attr, attr_state in insp.attrs.items():
    if attr_state.history.has_changes():
        print('{}: {}'.format(attr, attr_state.value))
        print('History: {}\n'.format(attr_state.history))

```

### 异常
[文档](http://docs.sqlalchemy.org/en/latest/orm/exceptions.html)
我们关系的主要有两个MultipleResultsFound，DetachedInstanceError.

```
from sqlalchemy.orm.exc import MultipleResultsFound
try:
    results = session.query(Cookie).one()
except MultipleResultsFound as error:
    print('We found too many cookies... is that even possible?')
```

