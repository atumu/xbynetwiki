title: sqlalchemy 

#  sqlalchemy学习 
SQLAlchemy是python的一个数据库ORM工具，提供了强大的对象模型间的转换，可以满足绝大多数数据库操作的需求，并且支持多种数据库引擎（sqlite，mysql，postgres,Oracle, mongodb等）
官网：http://www.sqlalchemy.org/
架构：http://aosabook.org/en/sqlalchemy.html
当前版本1.1.2 

pip install SQLAlchemy

首先是连接到数据库，SQLALchemy支持多个数据库引擎，不同的数据库引擎连接字符串不一样，常用的有
http://docs.sqlalchemy.org/en/latest/core/engines.html?highlight=create_engine#database-urls
```

mysql://username:password@hostname/database
postgresql://username:password@hostname/database
sqlite:////absolute/path/to/database
sqlite:///c:/absolute/path/to/database
oracle://scott:tiger@127.0.0.1:1521/orcl

```
```

#Unix/Mac - 4 initial slashes in total
engine = create_engine('sqlite:////absolute/path/to/foo.db')
#Windows
engine = create_engine('sqlite:///C:\\path\\to\\foo.db')
#Windows alternative using raw string
engine = create_engine(r'sqlite:///C:\path\to\foo.db')

```
##  1、使用传统的connection的方式连接和操作数据库 
```

from sqlalchemy import create_engine
 
# 数据库连接字符串
DB_CONNECT_STRING = 'sqlite:///:memory:'
 
# 创建数据库引擎,echo为True,会打印所有的sql语句
engine = create_engine(DB_CONNECT_STRING, echo=True)
 
# 创建一个connection，这里的使用方式与python自带的sqlite的使用方式类似
with engine.connect() as con:
    # 执行sql语句，如果是增删改，则直接生效，不需要commit
    rs = con.execute('SELECT 5')
    data = rs.fetchone()[0]
    print "Data: %s" % data

```
与python自带的sqlite不同，这里不需要Cursor光标，使用传统的connection的方式(非事务)，执行sql语句不需要commit

##  2、connection事务 
使用事务可以进行批量提交和回滚
```

from sqlalchemy import create_engine
 
# 数据库连接字符串
DB_CONNECT_STRING = 'sqlite:////Users/zhengxiankai/Desktop/Document/db.sqlite'
engine = create_engine(DB_CONNECT_STRING, echo=True)
 
with engine.connect() as connection:
    trans = connection.begin()
    try:
        r1 = connection.execute("select * from User")
        r2 = connection.execute("insert into User(name, age) values(?, ?)", 'bomo', 24)
        trans.commit()
    except:
        trans.rollback()
        raise

```
##  3. session 
connection是一般使用数据库的方式，sqlalchemy还提供了另一种操作数据库的方式，通过session对象，
session可以记录和跟踪数据的改变，在适当的时候提交，并且支持强大的ORM的功能，下面是基本使用
```

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
 
# 数据库连接字符串
DB_CONNECT_STRING = 'sqlite:////Users/zhengxiankai/Desktop/Document/db.sqlite'
 
# 创建数据库引擎,echo为True,会打印所有的sql语句
engine = create_engine(DB_CONNECT_STRING, echo=True)
 
# 创建会话类
DB_Session = sessionmaker(bind=engine)
 
# 创建会话对象
session = DB_Session()
 
# dosomething with session
session.execute('select * from User')
session.execute("insert into User(name, age) values('bomo', 13)")
session.execute("insert into User(name, age) values(:name, :age)", {'name': 'bomo', 'age':12})
 
# 如果是增删改，需要commit
session.commit()
# 用完记得关闭，也可以用with
session.close()


```
**注意参数使用dict，并在sql语句中使用:key占位**

##  4. ORM 
上面简单介绍了sql的简单用法，既然是ORM框架，我们先定义两个模型类User和Role，
**sqlalchemy的模型类继承自一个由declarative_base()方法生成的类**，我们先定义一个模块Models.py生成Base类
```

from sqlalchemy.ext.declarative import declarative_base
 
Base = declarative_base()

```
```

from sqlalchemy import Column, Integer, String
from Models import Base
 
class User(Base):
    __tablename__ = 'User'
    id = Column('id', Integer, primary_key=True, autoincrement=True)
    name = Column('name', String(50))
    age = Column('age', Integer)

```
**模型支持的类型有Integer, String, Boolean, Date, DateTime, Float等**，更多类型支持参考[这里](http://docs.sqlalchemy.org/en/latest/core/type_basics.html?highlight=column%20type#generic-types)

**Column构造函数相关设置**
  * name：名称
  * type_：列类型
  * autoincrement：自增
  * default：默认值
  * index：索引
  * nullable：可空
  * primary_key：外键
更多参考[这里](http://docs.sqlalchemy.org/en/latest/core/metadata.html?highlight=column%20autoincrement#sqlalchemy.schema.Column.__init__)

##  demo 
接下来通过session进行增删改查
```

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from User import User
from Role import Role
from Models import Base

DB_CONNECT_STRING = 'sqlite:////Users/zhengxiankai/Desktop/Document/db.sqlite'
engine = create_engine(DB_CONNECT_STRING, echo=True)
DB_Session = sessionmaker(bind=engine)
session = DB_Session()

# 1. 创建表（如果表已经存在，则不会创建）
Base.metadata.create_all(engine)

# 2. 插入数据
u = User(name = 'tobi', age = 200)
r = Role(name = 'user')

# 2.1 使用add，如果已经存在，会报错
session.add(u)
session.add(r)
session.commit()
print r.id

# 3 修改数据
# 3.1 使用merge方法，如果存在则修改，如果不存在则插入
r.name = 'admin'
session.merge(r)

# 3.2 也可以通过这种方式修改
session.query(Role).filter(Role.id == 1).update({'name': 'admin'})

# 4. 删除数据
session.query(Role).filter(Role.id == 1).delete()

# 5. 查询数据
# 5.1 返回结果集的第二项
user = session.query(User).get(2)

# 5.2 返回结果集中的第2-3项
users = session.query(User)[1:3]

# 5.3 查询条件
user = session.query(User).filter(User.id < 6).first()

# 5.4 排序
users = session.query(User).order_by(User.name)

# 5.5 降序（需要导入desc方法）
from sqlalchemy import desc
users = session.query(User).order_by(desc(User.name))

# 5.6 只查询部分属性
users = session.query(User.name).order_by(desc(User.name))
for user in users:
    print user.name

# 5.7 给结果集的列取别名
users = session.query(User.name.label('user_name')).all()
for user in users:
    print user.user_name

# 5.8 去重查询（需要导入distinct方法）
from sqlalchemy import distinct
users = session.query(distinct(User.name).label('name')).all()

# 5.9 统计查询
user_count = session.query(User.name).order_by(User.name).count()
age_avg = session.query(func.avg(User.age)).first()
age_sum = session.query(func.sum(User.age)).first()

# 5.10 分组查询
users = session.query(func.count(User.name).label('count'), User.age).group_by(User.age)
for user in users:
    print 'age:{0}, count:{1}'.format(user.age, user.count)

session.close()

```
##  filter 和 filter_by的区别 
http://stackoverflow.com/questions/2128505/whats-the-difference-between-filter-and-filter-by-in-sqlalchemy
filter_by is used for simple queries on the column names like
db.users.filter_by(name='Joe')
The same can be accomplished with filter by writing
db.users.filter(db.users.name=='Joe')
but you can also write more powerful queries containing expressions like
db.users.filter(or_(db.users.name=='Ryan', db.users.country=='England'))
##  5. 多表关系 
```

from sqlalchemy import Column, Integer, String
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship
from Models import Base

class User(Base):
    __tablename__ = 'users'
    id = Column('id', Integer, primary_key=True, autoincrement=True)
    name = Column('name', String(50))
    age = Column('age', Integer)

    # 添加角色id外键(关联到Role表的id属性)
    role_id = Column('role_id', Integer, ForeignKey('roles.id'))
    # 添加同表外键
    second_role_id = Column('second_role_id', Integer, ForeignKey('roles.id'))

    # 添加关系属性，关联到role_id外键上
    role = relationship('Role', foreign_keys='User.role_id', backref='User_role_id')
    # 添加关系属性，关联到second_role_id外键上
    second_role = relationship('Role', foreign_keys='User.second_role_id', backref='User_second_role_id')

```
```

from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import relationship
from Models import Base

class Role(Base):
    __tablename__ = 'roles'
    id = Column('id', Integer, primary_key=True, autoincrement=True)
    name = Column('name', String(50))

    # 添加关系属性，关联到User.role_id属性上
    users = relationship("User", foreign_keys='User.role_id', backref="Role_users")
    # 添加关系属性，关联到User.second_role_id属性上
    second_users = relationship("User", foreign_keys='User.second_role_id', backref="Role_second_users")

```
这里有一点需要注意的是，**设置外键的时候ForeignKey('roles.id')这里面使用的是表名和表列，在设置关联属性的时候relationship('Role', foreign_keys='User.role_id', backref='User_role_id')，这里的foreign_keys使用的时候类名和属性名**
```

u = User(name='tobi', age=200)
 
r1 = Role(name='admin')
r2 = Role(name='user')
 
u.role = r1
u.second_role = r2
 
session.add(u)
session.commit()
 
# 查询（对于外键关联的关系属性可以直接访问，在需要用到的时候session会到数据库查询）
roles = session.query(Role).all()
for role in roles:
    print 'role:{0} users'
    for user in role.users:
        print '\t{0}'.format(user.name)
    print 'role:{0} second_users'
    for user in role.second_users:
        print '\t{0}'.format(user.name)

```
上面表示的是一对多（多对一）的关系，还有一对一，多对多，
**如果要表示一对一的关系，在定义relationship的时候设置uselist为False（默认为True）**，如在Role中
```

class Role(Base):
    ...
    user = relationship("User", uselist=False, foreign_keys='User.role_id', backref="Role_user")

```
##  6. 多表查询 
多表查询通常使用join进行表连接，第一个参数为表名，第二个参数为条件，例如
```

users = db.session.query(User).join(Role, Role.id == User.role_id)
 
for u in users:
    print u.name

```
join为内连接，还有左连接outerjoin，用法与join类似，右连接和全外链接在1.0版本上不支持，通常来说有这两个结合查询的方法基本够用了，
1.1版本貌似添加了右连接和全外连接的支持.

**还可以直接查询多个表，如下**
```

result = db.session.query(User, Role).filter(User.role_id = Role.id)
# 这里选择的是两个表，使用元组获取数据
for u, r in result:
      print u.name

```
##  三、数据库迁移 
参考：http://python.jobbole.com/86482/
sqlalchemy的数据库迁移/升级有两个库支持[alembic](http://alembic.zzzcomputing.com/en/latest/)和[sqlalchemy-migrate](https://sqlalchemy-migrate.readthedocs.io/en/latest/)
由于sqlalchemy-migrate在2011年发布了0.7.2版本后，就已经停止更新了，并且已经不维护了，也积累了很多bug，而alembic是较后来才出现，而且是sqlalchemy的作者开发的，有良好的社区支持，所以在这里只学习alembic这个库

pip install alembic

安装完成后再项目根目录运行
$ alembic init YOUR_ALEMBIC_DIR
alembic会在根目录创建YOUR_ALEMBIC_DIR目录和alembic.ini文件，如下
```

yourproject/
    alembic.ini
    YOUR_ALEMBIC_DIR/
        env.py
        README
        script.py.mako
        versions/
            3512b954651e_add_account.py
            2b1ae634e5cd_add_order_id.py
            3adcc9a56557_rename_username_field.py

```
其中
  * alembic.ini 提供了一些基本的配置
  * env.py 每次执行Alembic都会加载这个模块，主要提供项目Sqlalchemy Model 的连接
  * script.py.mako 迁移脚本生成模版
  * versions 存放生成的迁移脚本目录

默认情况下创建的是基于单个数据库的，如果需要支持多个数据库或其他，可以通过alembic list_templates查看支持的模板
```

$ alembic list_templates
Available templates:

generic - Generic single-database configuration.
multidb - Rudimentary multi-database configuration.
pylons - Configuration that reads from a Pylons project environment.

Templates are used via the 'init' command, e.g.:

  alembic init --template generic ./scripts

```
配置
使用之前，需要配置一下链接字符串，打开alembic.ini文件，设置sqlalchemy.url连接字符串，例如
sqlalchemy.url = sqlite:////Users/zhengxiankai/Desktop/database.db

**4. 创建数据库版本**
接下来我们创建一个数据库版本，并新建两个表
$ alembic revision -m 'create table'
创建一个版本（会在yourproject/YOUR_ALEMBIC_DIR/versions/文件夹中创建一个python文件1a8a0d799b33_create_table.py）
该python模块包含upgrade和downgrade两个方法，在这里添加一些新增表的逻辑
```

"""create table

Revision ID: 4fd533a56b34
Revises:
Create Date: 2016-09-18 17:20:27.667100

"""
from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision = '4fd533a56b34'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    # 添加表
    op.create_table(
        'account',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('name', sa.String(50), nullable=False),
        sa.Column('description', sa.Unicode(200)),
    )

    # 添加列
    # op.add_column('account', sa.Column('last_transaction_date', sa.DateTime))



def downgrade():
    # 删除表
    op.drop_table('account')

    # 删除列
    # op.drop_column('account', 'last_transaction_date')


```
这里使用到了了op对象，关于op对象的更多API使用，参见[这里](http://alembic.zzzcomputing.com/en/latest/ops.html#ops)

**5. 升级数据库**
刚刚实现了升级和降级的方法，通过下面命令升级数据库到最新版本
$ alembic upgrade head
这时候可以看到数据库多了两个表alembic_version和account，alembic_version存放数据库版本
关于升级和降级的其他命令还有下面这些
```

# 升到最高版本
$ alembic upgrade head

# 降到最初版本
$ alembic downgrade base

# 升两级
$ alembic upgrade +2

# 降一级
$ alembic downgrade -1

# 升级到制定版本
$ alembic upgrade e93b8d488143

# 查看当前版本
$ alembic current

# 查看历史版本详情
$ alembic history --verbose

# 查看历史版本（-r参数）类似切片
$ alembic history -r1975ea:ae1027
$ alembic history -r-3:current
$ alembic history -r1975ea:

```
**6. 通过元数据升级数据库**
上面我们是通过API升级和降级，我们也可以直接通过元数据更新数据库，也就是自动生成升级代码，先定义两个Model（User, Role），这里我定义成三个文件
```

yourproject/
    YOUR_ALEMBIC_DIR/
    tutorial/Db
        Models.py
        User.py
        Role.py

```
代码就放在一起了
```

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String
Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column('id', Integer, primary_key=True, autoincrement=True)
    name = Column('name', String)

class Role(Base):
    __tablename__ = 'roles'

    id = Column('id', Integer, primary_key=True, autoincrement=True)
    name = Column('name', String)


```
在YOUR_ALEMBIC_DIR/env.py配置元数据
target_metadata = None改为如下：
```

import os
import sys
 
# 这里需要添加相对路径到sys.path，否则会引用失败，尝试过使用相对路径，但各种不好使，还是使用这种方法靠谱些
sys.path.append(os.path.abspath(os.path.join(os.getcwd(), "../yourproject/tutorial/Db")))
 
from User import User
from Role import Role
from Models import Base
target_metadata = Base.metadata

```
os.path.join(os.getcwd()这个获取到的地址不是env.py的路径，而是根目录
在创建数据库版本的时候添加--autogenerate参数，就会从Base.metadata元数据中生成脚本
$ alembic revision --autogenerate -m "add user table"
这时候会在生成升级代码
```

"""add user table

Revision ID: 97de1533584a
Revises: 8678ab6d48c1
Create Date: 2016-09-19 21:58:00.758410

"""
from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision = '97de1533584a'
down_revision = '8678ab6d48c1'
branch_labels = None
depends_on = None

def upgrade():
    ### commands auto generated by Alembic - please adjust! ###
    op.create_table('roles',
    sa.Column('id', sa.Integer(), nullable=False),
    sa.Column('name', sa.String(), nullable=True),
    sa.PrimaryKeyConstraint('id')
    )
    op.create_table('users',
    sa.Column('id', sa.Integer(), nullable=False),
    sa.Column('name', sa.String(), nullable=True),
    sa.PrimaryKeyConstraint('id')
    )
    op.drop_table('account')
    ### end Alembic commands ###


def downgrade():
    ### commands auto generated by Alembic - please adjust! ###
    op.create_table('account',
    sa.Column('id', sa.INTEGER(), nullable=False),
    sa.Column('name', sa.VARCHAR(length=50), nullable=False),
    sa.Column('description', sa.VARCHAR(length=200), nullable=True),
    sa.Column('last_transaction_date', sa.DATETIME(), nullable=True),
    sa.PrimaryKeyConstraint('id')
    )
    op.drop_table('users')
    op.drop_table('roles')
    ### end Alembic commands ###

```
由于我没有定义account模型，会被识别为删除，如果删除了model的列的声明，则会被识别为删除列，自动生成的版本我们也可以自己修改，然后执行升级命令即可升级alembic upgrade head

需要注意的是
  * Base.metadata声明的类必须以数据库中的一一对应，**如果数据库中有的表，而在元数据中没有，会识别成删除表**
  * revision创建版本之前执行之前需要升级到最新版本
  * 配置Base之前，需要保证所有的Model都已经执行（即导入）过一次了，否则无法读取到，也就是需要把所有Model都import进来
##  四、常见问题 
1. String长度问题
如果使用mysql数据库，String类型对应的是VARCHAR类型，需要指定长度，否则会报下面错误，而在sqlite不会出现

##  批量插入数据 
```

session.execute(
    User.__table__.insert(),
    [{'name': `randint(1, 100)`,'age': randint(1, 100)} for i in xrange(10000)]
)
session.commit()

```
上面我批量插入了 10000 条记录，半秒内就执行完了；而 ORM 方式会花掉很长时间。

##  如何让执行的 SQL 语句增加前缀？ 
使用 query 对象的 prefix_with() 方法：
session.query(User.name).prefix_with('HIGH_PRIORITY').all()
##  如何替换一个已有主键的记录？ 
使用 session.merge() 方法替代 session.add()，其实就是 SELECT + UPDATE：
user = User(id=1, name='ooxx')
session.merge(user)
session.commit()
##  模型的属性名需要和表的字段名不一样怎么办？ 
开发时遇到过一个奇怪的需求，有个其他系统的表里包含了一个“from”字段，这在 Python 里是关键字，于是只能这样处理了：
from_ = Column('from', CHAR(10))


##  事件监听器 
SQLAlchemy可以监听core与orm模块发送的事件。
注册事件监听的方法：使用listen()方法，或者listen_for()装饰器：

sqlalchemy.event.listen(target, identifier, fn, *args, * *kw)
sqlalchemy.event.listens_for(target, identifier, *args, * *kw)

一个事件的名称和相应的侦听器函数的参数签名来自一个类绑定规范方法，例如：
PoolEvents.connect(dbapi_connection, connection_record)，事件名为connect,监听器函数需要接受和connect相同的两个参数。
```

from sqlalchemy.event import listen
from sqlalchemy.pool import Pool

def my_on_connect(dbapi_con, connection_record):
    print("New DBAPI connection:", dbapi_con)

listen(Pool, 'connect', my_on_connect)

```
```

from sqlalchemy.event import listens_for
from sqlalchemy.pool import Pool

@listens_for(Pool, "connect")
def my_on_connect(dbapi_con, connection_record):
    print("New DBAPI connection:", dbapi_con)

```
##  命名参数风格 
当然你也可以采用命名参数的形式。 named=True
```

from sqlalchemy.event import listens_for
from sqlalchemy.pool import Pool

@listens_for(Pool, "connect", named=True)
def my_on_connect(**kw):
    print("New DBAPI connection:", kw['dbapi_connection'])

```
你也可以这样
```

from sqlalchemy.event import listens_for
from sqlalchemy.pool import Pool

@listens_for(Pool, "connect", named=True)
def my_on_connect(dbapi_connection, **kw):
    print("New DBAPI connection:", dbapi_connection)
    print("Connection record:", kw['connection_record'])

```
	

参数说明：
第一个参数targets，指定订阅的目标，可以很灵活。
```

from sqlalchemy.event import listen
from sqlalchemy.pool import Pool, QueuePool
from sqlalchemy import create_engine
from sqlalchemy.engine import Engine
import psycopg2

def connect():
    return psycopg2.connect(username='ed', host='127.0.0.1', dbname='test')

my_pool = QueuePool(connect)
my_engine = create_engine('postgresql://ed@localhost/test')

# associate listener with all instances of Pool
listen(Pool, 'connect', my_on_connect)

# associate listener with all instances of Pool
# via the Engine class
listen(Engine, 'connect', my_on_connect)

# associate listener with my_pool
listen(my_pool, 'connect', my_on_connect)

# associate listener with my_engine.pool
listen(my_engine, 'connect', my_on_connect)

```

##  监听修饰符 
可以添加监听修饰符，来添加额外特性,
比如可以运行listener返回值，并给后续的监听器。**retval=True**
```

#定义一个回调函数用于响应触发事件
def setPassword(target, value, oldvalue, initiator):
    if value == oldvalue:#如果新设置的值与原有的值相等，那么说明用户并没有修改密码，返回原先的值
        return oldvalue
    #如果新值与旧值不同，说明密码发生改变，进行加密，加密方法可以根据自己需求改变
    return hashlib.md5("%s%s" % (password_prefix, value)).hexdigest()
#设置事件监听，event.listen(表单或表单字段, 触发事件, 回调函数, 是否改变插入值)
event.listen(User.password, "set", setPassword, retval=True)

```
也可以使监听器只允许一次： **once=True**
```

def on_config():
    do_config()

event.listen(Mapper, "before_configure", on_config, once=True)

```

##  移除监听器 
```

# if a function was registered like this...
@event.listens_for(SomeMappedClass, "before_insert", propagate=True)
def my_listener_function(*arg):
    pass

# ... it's removed like this
event.remove(SomeMappedClass, "before_insert", my_listener_function)

```
**propagate=True**代表同时作用于SomeMappedClass的子类。

其余接口
sqlalchemy.event.**contains**(target, identifier, fn)
Return True if the given target/ident/fn is set up to listen.

##  Core Events 
Core Events -包含一些钩子，如
  * connection pool lifecycle,
  * SQL statement execution, 
  * transaction lifecycle, 
  * schema creation and teardown.

事件基类sqlalchemy.event.base.Events，比如
class sqlalchemy.events.PoolEvents，由sqlalchemy.pool.Pool来发送
```

from sqlalchemy import event

def my_on_checkout(dbapi_conn, connection_rec, connection_proxy):
    "handle an on checkout event"

event.listen(Pool, 'checkout', my_on_checkout)

```
也接受Engine作为target参数
```

engine = create_engine("postgresql://scott:tiger@localhost/test")

# will associate with engine.pool
event.listen(engine, 'checkout', my_on_checkout)

```
###  连接池事件 
sqlalchemy.events.PoolEvents，由sqlalchemy.pool.Pool来发送
  * checkin(dbapi_connection, connection_record)，当一个连接被释放返回池的时候。
  * checkout(dbapi_connection, connection_record, connection_proxy)。当从池中获取一个连接时调用。
  * close(dbapi_connection, connection_record)
  * close_detached(dbapi_connection)
  * connect(dbapi_connection, connection_record) 池中的某个连接第一次创建时调用
  * first_connect(dbapi_connection, connection_record)池中的某个连接被第一次检出时调用
  * detach(dbapi_connection, connection_record)
  * invalidate(dbapi_connection, connection_record, exception)连接失效时
  * reset(dbapi_connection, connection_record)，在reset动作调用之前调用。
  * soft_invalidate(dbapi_connection, connection_record, exception)
###  SQL执行与连接事件 
sqlalchemy.events.**ConnectionEvents**
基于 sqlalchemy.event.base.Events，由Connectable的实现类来发送，有sqlalchemy.engine.Connection and sqlalchemy.engine.Engine.
**before_execute**(conn, clauseelement, multiparams, params)  execute() events, receiving uncompiled SQL constructs and other objects prior to rendering into SQL.
**after_execute**(conn, clauseelement, multiparams, params, result) Intercept high level execute() events after execute.
参数说明	
  * conn – Connection object
  * clauseelement¶ – SQL expression construct, Compiled instance, or string statement passed to Connection.execute().
  * multiparams – Multiple parameter sets, a list of dictionaries.
  * params – Single parameter set, a single dictionary.
  * result – ResultProxy generated by the execution.
**before_cursor_execute**(conn, cursor, statement, parameters, context, executemany) cursor execute() events before execution
**after_cursor_execute**(conn, cursor, statement, parameters, context, executemany) cursor execute() events after execution.
参数说明	
  * conn – Connection object
  * cursor – DBAPI cursor object. Will have results pending if the statement was a SELECT, but these should not be consumed as they will be needed by the ResultProxy.
  * statement – string SQL statement, as passed to the DBAPI
  * parameters¶ – Dictionary, tuple, or list of parameters being passed to the execute() or executemany() method of the DBAPI cursor. In some cases may be None.
  * context – ExecutionContext object in use. May be None.
  * executemany – boolean, if True, this is an executemany() call, if False, this is an execute() call.

**begin**(conn) Intercept begin() events.
**commit**(conn)
**release_savepoint**(conn, name, context)
**rollback**(conn)
**rollback_savepoint**(conn, name, context)
**savepoint**(conn, name)

**begin_twophase**(conn, xid)
**commit_twophase**(conn, xid, is_prepared)
**prepare_twophase**(conn, xid)
**rollback_twophase**(conn, xid, is_prepared)

**dbapi_error**(conn, cursor, statement, parameters, context, exception)
**handle_error**(exception_context) Intercept all exceptions processed by the Connection.

**engine_connect**(conn, branch) Intercept the creation of a new Connection.
Parameters:	
  * conn – Connection object.
  * branch – if True, this is a “branch” of an existing Connection. A branch is generated within the course of a statement execution to invoke supplemental statements, most typically to pre-execute a SELECT of a default value for the purposes of an INSERT statement.

**engine_disposed**(engine)
**set_connection_execution_options**(conn, opts) Intercept when the Connection.execution_options() method is called.
**set_engine_execution_options**(engine, opts) Intercept when the Engine.execution_options() method is called.


----

**class sqlalchemy.events.DialectEvents**  event interface for execution-replacement functions.
略。

###  Schema Events 
sqlalchemy.events.**DDLEvents**。 由, SchemaItem and other sqlalchemy.events.SchemaEventTarget子类, including MetaData, Table, Column.发送
例如：
```

from sqlalchemy import event
from sqlalchemy import Table, Column, Metadata, Integer

m = MetaData()
some_table = Table('some_table', m, Column('data', Integer))

def after_create(target, connection, **kw):
    connection.execute("ALTER TABLE %s SET name=foo_%s" %
                            (target.name, target.name))

event.listen(some_table, "after_create", after_create)

```
DDL事件也可以整合DDL类和DDLElement类系列：更多参考[这里](http://docs.sqlalchemy.org/en/latest/core/ddl.html#schema-ddl-sequences)
```

from sqlalchemy import DDL
event.listen(
    some_table,
    "after_create",
    DDL("ALTER TABLE %(table)s SET name=foo_%(table)s")
)

```

* before_create(target, connection, * *kw)
  * before_drop(target, connection, * *kw)
  * after_create(target, connection, * *kw) Called after CREATE statements are emitted.
  * after_drop(target, connection, * *kw) Called after DROP statements are emitted.
  * before_parent_attach(target, parent)
  * after_parent_attach(target, parent) Called after a SchemaItem is associated with a parent SchemaItem.

  * column_reflect(inspector, table, column_info) 检出列信息的时候调用

更多事件，参考：http://docs.sqlalchemy.org/en/latest/core/events.html#sqlalchemy.events.PoolEvents.connect

##  ORM Events 
具体见：http://docs.sqlalchemy.org/en/latest/orm/events.html，http://docs.sqlalchemy.org/en/latest/orm/session_events.html
ORM Events -包含一些ORM事件，如 
  * class and attribute instrumentation,
  * object initialization hooks, 
  * attribute on-change hooks, 
  * session state, flush, and commit hooks,
  * mapper initialization,
  * object/result population, 
  * per-instance persistence hooks.


###  属性事件 
由ORM模型对象发送事件
sqlalchemy.orm.events.**AttributeEvents**
如：
```

from sqlalchemy import event

def my_append_listener(target, value, initiator):
    print "received append event for target: %s" % target

event.listen(MyClass.collection, 'append', my_append_listener)

```
**可用的监听参数：**
  * **active_history=False** – When True, indicates that the “set” event would like to receive the “old” value being replaced unconditionally, even if this requires firing off database loads. Note that active_history can also be set directly via column_property() and relationship().
  * **propagate=False** – When True, 可以将监听函数传递应用到子类。
  * **raw=False** – When True, the “target” 参数会变成 InstanceState management object, 而不是mapped instance itself.
  * **retval=False** – when True, t事件监听函数必须返回一个值。使得有机会改变最终会被set,append事件使用的值。

**append**(target, value, initiator) Receive a collection append event.
参数
  * target – the object instance receiving the event. If the listener is registered with raw=True, this will be the InstanceState object.
  * value – the value being appended. If this listener is registered with retval=True, the listener function must return this value, or a new value which replaces it.
  * initiator –attributes.Event的实例，代表事件对象. 在事件传递链上可以被修改。
返回值：当注册时包含标志retval=True，则需要返回值。	

**init_collection**(target, collection, collection_adapter) Receive a ‘collection init’ event.
**init_scalar**(target, value, dict_) Receive a scalar “init” event.
**remove**(target, value, initiator) Receive a collection remove event.
**set**(target, value, oldvalue, initiator) Receive a scalar set event.
  * value – the value being set.如果配置了retval=True, 监听器必须返回该值，或者一个更新的值。
  * oldvalue – the previous value being replaced.如果定义了active_history=True ，该值可能为 NEVER_SET or NO_VALUE. 

###  Mapper Events 
Available targets include:
  * mapped classes
  * unmapped superclasses of mapped or to-be-mapped classes (using the propagate=True flag)
  * Mapper objects
  * the Mapper class itself and the mapper() function indicate listening for all mappers.

sqlalchemy.orm.events.**MapperEvents**
  * before/after_configured() Called after a series of mappers have been configured.
  * before/after_delete(mapper, connection, target) Receive an object instance after a DELETE statement has been emitted corresponding to that instance.
  * before/after_insert(mapper, connection, target) Receive an object instance after an INSERT statement is emitted corresponding to that instance.
  * before/after_update(mapper, connection, target)

  * instrument_class(mapper, class_) Receive a class when the mapper is first constructed, before instrumentation is applied to the mapped class.
  * mapper_configured(mapper, class_)

###  Instance Events 
sqlalchemy.orm.events.**InstanceEvents**  Define events specific to object lifecycle.
Available targets include:
  * mapped classes
  * unmapped superclasses of mapped or to-be-mapped classes (using the propagate=True flag)
  * Mapper objects
  * the Mapper class itself and the mapper() function indicate listening for all mappers.

expire(target, attrs) Receive an object instance after its attributes or some subset have been expired.
first_init(manager, cls) Called when the first instance of a particular mapping is called.
init(target, args, kwargs)
init_failure(target, args, kwargs)
load(target, context)
 context – the QueryContext corresponding to the current Query in progress. This argument may be None if the load does not correspond to a Query, such as during Session.merge().

pickle(target, state_dict)
unpickle(target, state_dict)
refresh(target, context, attrs)
refresh_flush(target, flush_context, attrs)

###  Session Events 
sqlalchemy.orm.events.SessionEvents 由Session 对象发送
```

from sqlalchemy import event
from sqlalchemy.orm import sessionmaker

def my_before_commit(session):
    print "before commit!"

Session = sessionmaker()

event.listen(Session, "before_commit", my_before_commit)

```
before_attach(session, instance)
after_attach(session, instance) Execute after an instance is attached to a session.This is called after an add, delete or merge.
deleted_to_detached(session, instance)

after_bulk_delete(delete_context)
Parameters:	delete_context 
  * session - the Session involved
  * query -the Query object that this update operation was called upon.
  * context The QueryContext object, corresponding to the invocation of an ORM query.
  * result the ResultProxy returned as a result of the bulk DELETE operation.

after_bulk_update(update_context)

after_begin(session, transaction, connection) Execute after a transaction is begun on a connection
after_transaction_create(session, transaction)
before_commit(session)
after_commit(session)
after_transaction_end(session, transaction)
before_flush(session, flush_context, instances)
after_flush(session, flush_context)
after_rollback(session)
after_soft_rollback(session, previous_transaction)
after_flush_postexec(session, flush_context)


loaded_as_persistent(session, instance)
detached_to_persistent(session, instance)
pending_to_persistent(session, instance)
deleted_to_persistent(session, instance)
pending_to_transient(session, instance)
persistent_to_deleted(session, instance)
persistent_to_detached(session, instance)
persistent_to_transient(session, instance)
transient_to_pending(session, instance)
###  Query Events 
sqlalchemy.orm.events.QueryEvents 由sqlalchemy.orm.query.Query对象发送
**before_compile**(query)
```

@event.listens_for(Query, "before_compile", retval=True)
def no_deleted(query):
    for desc in query.column_descriptions:
        if desc['type'] is User:
            entity = desc['entity']
            query = query.filter(entity.deleted == False)
    return query

```
###  Instrumentation Events 
sqlalchemy.orm.events.InstrumentationEvents 
略。

参考：https://my.oschina.net/zhengnazhi/blog/120800
