title: flask-sqlalchemy 
createAt:2016-12-12 00:42:26
location:dokuwiki/python/flask-sqlalchemy
modifyAt:2016-12-12 00:42:26
author:xbynet

#  Flask-sqlalchemy ORM插件集成 
官网：http://flask-sqlalchemy.pocoo.org/2.1/
中文：http://www.pythondoc.com/flask-sqlalchemy/index.html
API：http://flask-sqlalchemy.pocoo.org/2.1/api/
当前版本:2.1

pip install flask-sqlalchemy


##  快速预览 
```

from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:////tmp/test.db'
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)

    def __init__(self, username, email):
        self.username = username
        self.email = email

    def __repr__(self):
        return '<User %r>' % self.username

```
为了创建初始数据库，只需要从交互式 Python shell 中导入 db 对象并且调用 SQLAlchemy.create_all() 方法来创建表和数据库:
```

>>> from yourapplication import db
>>> db.create_all()

```
Boom, 您的数据库已经生成。现在来创建一些用户:
```

>>> from yourapplication import User
>>> admin = User('admin', 'admin@example.com')
>>> guest = User('guest', 'guest@example.com')

```
但是它们还没有真正地写入到数据库中，因此让我们来确保它们已经写入到数据库中:
```

>>> db.session.add(admin)
>>> db.session.add(guest)
>>> db.session.commit()

```
访问数据库中的数据也是十分简单的:
```

>>> users = User.query.all()
[<User u'admin'>, <User u'guest'>]
>>> admin = User.query.filter_by(username='admin').first()
<User u'admin'>

```
简单的关系
```

from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:////tmp/test.db'
db = SQLAlchemy(app)
class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(80))
    body = db.Column(db.Text)
    pub_date = db.Column(db.DateTime)

    category_id = db.Column(db.Integer, db.ForeignKey('category.id'))
    category = db.relationship('Category',
        backref=db.backref('posts', lazy='dynamic'))

    def __init__(self, title, body, category, pub_date=None):
        self.title = title
        self.body = body
        if pub_date is None:
            pub_date = datetime.utcnow()
        self.pub_date = pub_date
        self.category = category

    def __repr__(self):
        return '<Post %r>' % self.title


class Category(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50))

    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return '<Category %r>' % self.name

```
首先让我们创建一些对象:
```

>>> py = Category('Python')
>>> p = Post('Hello Python!', 'Python is pretty cool', py)
>>> db.session.add(py)
>>> db.session.add(p)
>>> db.session.commit()
>>> py.posts.all()

``` 
**flask.ext.sqlalchemy与普通的 SQLAlchemy 不同之处:**
允许您访问下面的东西:
  * sqlalchemy 和 sqlalchemy.orm 下所有的函数和类
  * 一个叫做 session 的预配置范围的会话(session)
  * metadata 属性
  * engine 属性
  * SQLAlchemy.create_all() 和 SQLAlchemy.drop_all()，根据模型用来创建以及删除表格的方法
  * 一个 Model 基类，即是一个已配置的声明(declarative)的基础(base)
  * 您必须提交会话，**但是没有必要在每个请求后删除它(session)**，Flask-SQLAlchemy 会帮您完成删除操作。

##  引入上下文 
如果您计划只使用一个应用程序，您大可跳过这一章节。只需要把您的应用程序传给 SQLAlchemy 构造函数，一般情况下您就设置好了。
然而您想要使用不止一个应用或者在一个函数中动态地创建应用的话，您需要仔细阅读。
如果您在一个函数中定义您的应用，但是 SQLAlchemy 对象是全局的，后者如何知道前者了？答案就是 init_app() 函数:
```

from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    return app

```
**那么 SQLAlchemy 是如何知道您的应用的？您必须配置一个应用上下文**。
如果您在一个 Flask 视图函数中进行工作，这会自动实现。但如果您在交互式的 shell 中，您需要手动这么做。
```

>>> from yourapp import create_app
>>> app = create_app()
>>> app.app_context().push()

```
在脚本里面使用 with 声明都样也有作用:
```

def my_function():
    with app.app_context():
        user = db.User(...)
        db.session.add(user)
        db.session.commit()

```
Flask-SQLAlchemy 里的一些函数也可以接受要在其上进行操作的应用作为参数:
```

>>> from yourapp import db, create_app
>>> db.create_all(app=create_app())

```

##  配置 
下面是 Flask-SQLAlchemy 中存在的配置值。Flask-SQLAlchemy 从您的 Flask 主配置中加载这些值。
SQLALCHEMY_DATABASE_URI	
用于连接数据的数据库。例如：
  * SQLALCHEMY_DATABASE_URI	用于连接数据的数据库。&lt;nowiki&gt;例如：sqlite:////tmp/test.db  ， mysql://username:password@server/db&lt;/nowiki&gt;
  * SQLALCHEMY_BINDS	一个映射绑定 (bind) 键到 SQLAlchemy 连接 URIs 的字典。 **更多的信息请参阅后面提到的绑定多个数据库。**
  * SQLALCHEMY_ECHO	如果设置成 True，SQLAlchemy 将会记录所有 发到标准输出(stderr)的语句，这对调试很有帮助。
  * SQLALCHEMY_RECORD_QUERIES	可以用于显式地禁用或者启用查询记录。查询记录 在调试或者测试模式下自动启用。更多信息请参阅 get_debug_queries()。
  * SQLALCHEMY_NATIVE_UNICODE	可以用于显式地禁用支持原生的 unicode。这是 某些数据库适配器必须的（像在 Ubuntu 某些版本上的 PostgreSQL），当使用不合适的指定无编码的数据库 默认值时。
  * SQLALCHEMY_POOL_SIZE	数据库连接池的大小。默认是数据库引擎的默认值 （通常是 5）。
  * SQLALCHEMY_POOL_TIMEOUT	指定数据库连接池的超时时间。默认是 10。
  * SQLALCHEMY_POOL_RECYCLE	自动回收连接的秒数。**这对 MySQL 是必须的，默认 情况下 MySQL 会自动移除闲置 8 小时或者以上的连接。** 需要注意地是如果使用 MySQL 的话， Flask-SQLAlchemy 会自动地设置这个值为 2 小时。
  * SQLALCHEMY_MAX_OVERFLOW	控制在连接池达到最大值后可以创建的连接数。当这些额外的 连接回收到连接池后将会被断开和抛弃。
  * SQLALCHEMY_TRACK_MODIFICATIONS	如果设置成 True (默认情况)，Flask-SQLAlchemy **将会追踪对象的修改并且` 发送信号 `(注意：禁用了无法支持信号)**。这需要额外的内存， 如果不必要的可以禁用它。
###  连接 URI 格式 
参考http://www.sqlalchemy.org/docs/core/engines.html
dialect+driver://username:password@host:port/database 该字符串中的许多部分是可选的。如果没有指定驱动器，会选择默认的（确保在这种情况下 不 包含 + ）。

```
Postgres:
postgresql://scott:tiger@localhost/mydatabase

MySQL:
mysql://scott:tiger@localhost/mydatabase

Oracle:
oracle://scott:tiger@127.0.0.1:1521/sidname

SQLite (注意开头的四个斜线):
sqlite:////absolute/path/to/foo.db
MySQL¶
The MySQL dialect uses mysql-python as the default DBAPI. There are many MySQL DBAPIs available, including MySQL-connector-python and OurSQL:
# default
engine = create_engine('mysql://scott:tiger@localhost/foo')
# mysql-python
engine = create_engine('mysql+mysqldb://scott:tiger@localhost/foo')
# MySQL-connector-python
engine = create_engine('mysql+mysqlconnector://scott:tiger@localhost/foo')
# OurSQL
engine = create_engine('mysql+oursql://scott:tiger@localhost/foo')
 
Oracle
The Oracle dialect uses cx_oracle as the default DBAPI:
engine = create_engine('oracle://scott:tiger@127.0.0.1:1521/sidname')
engine = create_engine('oracle+cx_oracle://scott:tiger@tnsname')
 
SQLite
using the Python built-in module sqlite3 by default.
# sqlite://&lt;nohostname&gt;/&lt;path&gt;
# where &lt;path&gt; is relative:
engine = create_engine('sqlite:///foo.db')
#Unix/Mac - 4 initial slashes in total
engine = create_engine('sqlite:////absolute/path/to/foo.db')
#Windows
engine = create_engine('sqlite:///C:\\path\\to\\foo.db')
#Windows alternative using raw string
engine = create_engine(r'sqlite:///C:\path\to\foo.db')
To use a SQLite :memory: database, specify an empty URL:
engine = create_engine('sqlite://')
```

**使用自定义的元数据和命名约定**
使用一个自定义的 MetaData 对象来构造 SQLAlchemy 对象.这允许你指定一个 自定义约束命名约定。**这样做对数据库的迁移是很重要的**。因为 SQL 没有定义一个标准的命名约定，无法保证数据库之间实现是兼容的。你可以自定义命名约定像 SQLAlchemy 文档建议那样:
```

from sqlalchemy import MetaData
from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy

convention = {
    "ix": 'ix_%(column_0_label)s', #index命名约定
    "uq": "uq_%(table_name)s_%(column_0_name)s",#unique命名约定
    "ck": "ck_%(table_name)s_%(constraint_name)s",#check命名约定
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",#foregin key命名约定
    "pk": "pk_%(table_name)s" #primary key命名约定
}

metadata = MetaData(naming_convention=convention)
db = SQLAlchemy(app, metadata=metadata)

```
##  声明模型 
您的所有模型的基类叫做SQLAlchemy.Model。
有一些部分在 SQLAlchemy 上是必选的，但是在 Flask-SQLAlchemy 上是可选的。 比如**表名是自动地为您设置好的，除非您想要覆盖它。它是从转成小写的类名派生出来的，即 “CamelCase” 转换为 “camel_case”。**
```

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)

    def __init__(self, username, email):
        self.username = username
        self.email = email

    def __repr__(self):
        return '<User %r>' % self.username

```
用 **Column** 来定义一列。
  * 列名就是您赋值给那个变量的名称。
  * 如果您想要在表中使用不同的名称，您可以提供一个想要的列名的字符串作为可选第一个参数。
  * 主键用 primary_key=True 标记。可以把多个键标记为主键，此时它们作为复合主键。

下面的列类型是最常用的:
  * Integer	一个整数
  * String (size)	有长度限制的字符串
  * Text	一些较长的 unicode 文本
  * DateTime	表示为 Python datetime 对象的 时间和日期
  * Float	存储浮点值
  * Boolean	存储布尔值
  * PickleType	存储为一个持久化的 Python 对象
  * LargeBinary	存储一个任意大的二进制数据
###  一对多(one-to-many)关系 
关系使用 **relationship**() 函数表示。然而外键必须用类 sqlalchemy.schema.**ForeignKey** 来单独声明:
```

class Person(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50))
    addresses = db.relationship('Address', backref='person',
                                lazy='dynamic')

class Address(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(50))
    person_id = db.Column(db.Integer, db.ForeignKey('person.id'))

```
db.relationship() 做了什么？这个函数返回一个可以做许多事情的新属性。在本案例中，我们让它指向 Address 类并加载多个地址。
它如何知道会返回不止一个地址？因为 SQLALchemy 从您的声明中猜测了一个有用的默认值。 
**如果您想要一对一关系，您可以把 uselist=False 传给 relationship() 。**

那么 backref 和 lazy 意味着什么了？
backref 是一个在 Address 类上声明新属性的简单方法。您也可以使用 my_address.person 来获取使用该地址(address)的人(person)。
lazy 决定了 SQLAlchemy 什么时候从数据库中加载数据:
  * 'select' (默认值) 就是说 SQLAlchemy 会使用一个标准的 select 语句必要时一次加载数据。
  * 'joined' 告诉 SQLAlchemy 使用 JOIN 语句作为父级在同一查询中来加载关系。
  * 'subquery' 类似 'joined' ，但是 SQLAlchemy 会使用子查询。
  * 'dynamic' 在有多条数据的时候是特别有用的。不是直接加载这些数据，SQLAlchemy 会返回一个查询对象，在加载数据前您可以过滤（提取）它们。

您如何为反向引用（backrefs）定义惰性（lazy）状态？使用 **backref**() 函数:
```

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50))
    addresses = db.relationship('Address',
        backref=db.backref('person', lazy='joined'), lazy='dynamic')

```
###  多对多(many-to-many)关系 
如果您想要用多对多关系，您需要定义一个用于关系的辅助表。**对于这个辅助表， 强烈建议不使用模型，而是采用一个实际的表**:
```

tags = db.Table('tags',
    db.Column('tag_id', db.Integer, db.ForeignKey('tag.id')),
    db.Column('page_id', db.Integer, db.ForeignKey('page.id'))
)

class Page(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    tags = db.relationship('Tag', secondary=tags,
        backref=db.backref('pages', lazy='dynamic'))

class Tag(db.Model):
    id = db.Column(db.Integer, primary_key=True)

```
这里我们配置 Page.tags 加载后作为标签的列表，因为我们并不期望每页出现太多的标签。**而每个 tag 的页面列表（ Tag.pages）是一个动态的反向引用**。 正如上面提到的，**这意味着您会得到一个可以发起 select 的查询对象。**

##  选择(Select),插入(Insert), 删除(Delete)、更新(Update) 
我们将会使用 快速入门 章节中定义的数据模型。
**插入记录**
您的所有模型都应该有一个构造函数，如果您 忘记了，请确保加上一个。只有您自己使用这些构造函数而 SQLAlchemy 在内部不会使用它， 所以如何定义这些构造函数完全取决与您。
向数据库插入数据分为三个步骤:
  * 创建 Python 对象
  * 把它添加到会话 ，这里的会话不是 Flask 的会话，而是 Flask-SQLAlchemy 的会话。
  * 提交会话
```

>>> me = User('admin', 'admin@example.com')
>>> db.session.add(me)
>>> db.session.commit()

```
批量插入：
http://stackoverflow.com/questions/3659142/bulk-insert-with-sqlalchemy-orm
```

s = Session()
objects = [
    User(name="u1"),
    User(name="u2"),
    User(name="u3")
]
s.bulk_save_objects(objects)

```
或者：
```

users = get_users_to_insert() 
db.engine.execute(user.__table__.insert(), users)

```
或者：
```

Foo.__table__.insert().execute([{'bar': 1}, {'bar': 2}, {'bar': 3}])
# INSERT INTO foo (bar) VALUES ((1,), (2,), (3,))

```
**删除记录**
```

>>> db.session.delete(me)
>>> db.session.commit()

```

**查询记录**
Flask-SQLAlchemy 在您的 Model 类上提供了 **query 属性**。当您访问它时，您会得到一个新的所有记录的查询对象。
在使用 all() 或者 first() 发起查询之前可以使用方法 filter() 来过滤记录。如果您想要用主键查询的话，也可以使用 get()。
下面的查询假设数据库中有如下条目:
id	username	email
1	admin	admin@example.com
2	peter	peter@example.org
3	guest	guest@example.com
```

通过用户名查询用户:
>>> peter = User.query.filter_by(username='peter').first()
>>> peter.id
1
>>> peter.email
u'peter@example.org'
同上但是查询一个不存在的用户名返回 None:
>>> missing = User.query.filter_by(username='missing').first()
>>> missing is None
True
使用更复杂的表达式查询一些用户:
>>> User.query.filter(User.email.endswith('@example.com')).all()
[<User u'admin'>, <User u'guest'>]
按某种规则对用户排序:
>>> User.query.order_by(User.username)
[<User u'admin'>, <User u'guest'>, <User u'peter'>]
限制返回用户的数量:
>>> User.query.limit(1).all()
[<User u'admin'>]
用主键查询用户:
>>> User.query.get(1)
<User u'admin'>

```

**修改更新数据：**
```

>>>p=db.session.query(Parent).get(1)     #先查询出需要修改的条目
或：
>>>Parent.query.get(1)
然后
>>>p.name='p2'     #修改
>>>db.session.commit()     #修改成功，的确很方便

或者直接用一条语句：

#直接查询出后修改，update采用字典修改{修要修改的列：'修改后的值'}
>>>db.session.query(Child).filter(Child.id==1).update({Child.name:'c3'}) 
>>>db.session.commit()

```
###  在flask-sqlalchemy中使用分页 
https://my.oschina.net/ranvane/blog/196906
http://pythonhosted.org/Flask-SQLAlchemy/api.html#utilities
sqlalchemy中使用query查询，而flask-sqlalchemy中使用basequery查询
具体的使用方法：
paginate = User.query.paginate(curPage, POSTS_PER_PAGE, False)  #paginate(page=None, per_page=None, error_out=True) 返回 Pagination 
object_list = paginate.items
return render_template('simplecd_list.html',pagination = paginate,object_list = object_list)

User是我的model，curPage是显示的第几页页数，POSTS_PER_PAGE每页显示多少条，paginate.items才是分页好的数据，下面是一个分页导航的例子：
Pagination这个类包含很多的属性，可以用来在模板中生成分页的链接，因此可以将其作为参数传入模板。 
Pagination类对象的**属性**主要有： 
  * has_next：如果在目前页后至少还有一页的话，返回 True。 
  * has_prev：如果在目前页之前至少还有一页的话，返回 True。 
  * page:当前的页数
  * next_num：下一页的页面数。 
  * prev_num：前一页的页面数。 
另外还有如下的**可调用方法**： 
  * iter_pages():一个迭代器，返回一个在分页导航中显示的页数列表。 
  * prev():上一页的分页对象。 
  * next():下一页的分页对象。
```

<div class="pagination  "> 
    <div class="row-fluid"> 
        <div class="span3 offset2"> 
            {% if pagination.has_prev %} 
                <a href="/index/![](/data/dokuwiki pagination.prev_num )">previous</a> 
            {% endif %} 
        </div> 
        <div class="span3 "> 
            <a href="">Page ![](/data/dokuwiki pagination.page ) of ![](/data/dokuwiki pagination.pages ).</a> 
        </div> 
        <div class="span3 ">

            {% if pagination.has_next %} 
                <a href="/index/![](/data/dokuwiki pagination.next_num )">next</a> 
            {% endif %} 
        </div> 
    </div> 
</div>

```

当然，你也可以使用宏来定义一个通用的分页，例如
```

{% macro render_pagination(pagination, endpoint) %}
  <div class=pagination>
  {%- for page in pagination.iter_pages() %}
    {% if page %}
      {% if page != pagination.page %}
        <a href="![](/data/dokuwiki url_for(endpoint, page=page) )">![](/data/dokuwiki page )</a>
      {% else %}
        <strong>![](/data/dokuwiki page )</strong>
      {% endif %}
    {% else %}
      <span class=ellipsis>…</span>
    {% endif %}
  {%- endfor %}
  </div>
{% endmacro %}

```

###  在视图中查询 
当您编写 Flask **视图函数**，对于不存在的条目返回一个 404 错误是非常方便的。
因为这是一个很常见的问题，Flask-SQLAlchemy 为了解决这个问题提供了一个帮助函数。
可以使用 **get_or_404**() 来代替 get()，使用 **first_or_404**() 来代替 first()。这样会抛出一个 404 错误，而不是返回 None:
```

@app.route('/user/<username>')
def show_user(username):
    user = User.query.filter_by(username=username).first_or_404()
    return render_template('show_user.html', user=user)

```
##  绑定多个数据库 
为了实现这个功能，预配置了 SQLAlchemy 来支持多个 “binds”。
什么是绑定(binds)?
在 SQLAlchemy 中一个绑定(bind)是能执行 SQL 语句并且通常是一个连接或者引擎类的东东。
在 Flask-SQLAlchemy 中，绑定(bind)总是背后自动为您创建好的引擎。这些引擎中的每个之后都会关联一个短键（bind key）。这个键会在模型声明时使用来把一个模型关联到一个特定引擎。
如果模型没有关联一个特定的引擎的话，就会使用默认的连接(SQLALCHEMY_DATABASE_URI 配置值)。
下面的配置声明了三个数据库连接。特殊的默认值和另外两个分别名为 users`（用于用户）和 `appmeta 数据库:
```

SQLALCHEMY_DATABASE_URI = 'postgres://localhost/main'
SQLALCHEMY_BINDS = {
    'users':        'mysqldb://localhost/users',
    'appmeta':      'sqlite:////path/to/appmeta.db'
}

```
create_all() 和 drop_all() 方法默认作用于所有声明的绑定(bind)，包括默认的。这个行为可以通过提供 bind 参数来定制。
默认的绑定(bind)(SQLALCHEMY_DATABASE_URI) 名为 None
**创建和删除表**
```

>>> db.create_all()
>>> db.create_all(bind=['users'])
>>> db.create_all(bind='appmeta')
>>> db.drop_all(bind=None)

```
**引用绑定(Binds)**
当您声明模型时，您可以用 __bind_key__ 属性指定绑定(bind):
```

class User(db.Model):
    __bind_key__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)

```
    
bind key 存储在表中的 info 字典中作为 'bind_key' 键值。了解这个很重要，因为**当您想要直接创建一个表对象时**，您会需要把它放在那:
```

user_favorites = db.Table('user_favorites',
    db.Column('user_id', db.Integer, db.ForeignKey('user.id')),
    db.Column('message_id', db.Integer, db.ForeignKey('message.id')),
    info={'bind_key': 'users'}
)

```
##  信号支持 
您可以订阅如下这些信号以便在更新提交到数据库**之前以及之后**得到通知。
如果配置中 SQLALCHEMY_TRACK_MODIFICATIONS 启用的话，这些更新变化才能被追踪。
**存在以下两个信号:**
  * models_committed
这个信号在修改的模型提交到数据库时发出。发送者是发送修改的应用，模型和操作描述符以 (model, operation) 形式作为元组，这样的元组列表传递给接受者的 changes 参数。
该模型是发送到数据库的模型实例，当一个模型已经插入，操作是 'insert' ，而已删除是 'delete' ，如果更新了任何列，会是 'update' 。

  * before_models_committed
工作机制和 models_committed 完全一样，但是在提交之前发送。

##  常用的SQLAlchemy列类型 
类型名	Python类型	说 明
  * Integer	int	普通整数,一般是 32 位
  * SmallInteger	int	取值范围小的整数,一般是 16 位
  * BigInteger	int 或 long	不限制精度的整数
  * Float	float	浮点数
  * Numeric	decimal.Decimal	定点数
  * String	str	变长字符串
  * Text	str	变长字符串,对较长或不限长度的字符串做了优化
  * Unicode	unicode	变长 Unicode 字符串
  * UnicodeText	unicode	变长 Unicode 字符串,对较长或不限长度的字符串做了优化
  * Boolean	bool	布尔值
  * Date	datetime.date	日期
  * Time	datetime.time	时间
  * DateTime	datetime.datetime	日期和时间
  * Interval	datetime.timedelta	时间间隔
  * Enum	str	一组字符串
  * PickleType	任何 Python 对象	自动使用 Pickle 序列化
  * LargeBinary	str	二进制文件
##  最常使用的SQLAlchemy列选项 
  * primary_key	如果设为 True ,这列就是表的主键
  * unique	如果设为 True ,这列不允许出现重复的值
  * index	如果设为 True ,为这列创建索引,提升查询效率
  * nullable	如果设为 True ,这列允许使用空值;如果设为 False ,这列不允许使用空值
  * default	为这列定义默认值
##  常用的SQLAlchemy关系选项 
  * backref	在关系的另一个模型中添加反向引用
  * primaryjoin	明确指定两个模型之间使用的联结条件。只在模棱两可的关系中需要指定
  * lazy	指定如何加载相关记录。可选值有 select (首次访问时按需加载)、 immediate (源对象加载后就加载)、 joined (加载记录,但使用联结)、 subquery (立即加载,但使用子查询),noload (永不加载)和 dynamic (不加载记录,但提供加载记录的查询)
  * uselist	如果设为 Fales ,则为一对一 。
  * order_by	指定关系中记录的排序方式
  * secondary	指定 多对多 关系中关系表的名字
  * secondaryjoin	SQLAlchemy 无法自行决定时,指定多对多关系中的二级联结条件

##  常用过滤器 
  * filter()	把过滤器添加到原查询上,返回一个新查询
  * filter_by()	把等值过滤器添加到原查询上,返回一个新查询
  * limit()	使用指定的值限制原查询返回的结果数量,返回一个新查询
  * offset()	偏移原查询返回的结果,返回一个新查询
  * order_by()	根据指定条件对原查询结果进行排序,返回一个新查询
  * group_by()	根据指定条件对原查询结果进行分组,返回一个新查询
##  最常使用的SQLAlchemy查询执行函数 
  * all()	以列表形式返回查询的所有结果
  * first()	返回查询的第一个结果,如果没有结果,则返回 None
  * first_or_404()	返回查询的第一个结果,如果没有结果,则终止请求,返回 404 错误响应
  * get()	返回指定主键对应的行,如果没有对应的行,则返回 None
  * get_or_404()	返回指定主键对应的行,如果没找到指定的主键,则终止请求,返回 404 错误响应
  * count()	返回查询结果的数量
  * paginate()	返回一个 Paginate 对象,它包含指定范围内的结果
