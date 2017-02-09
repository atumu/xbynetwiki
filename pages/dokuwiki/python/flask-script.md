title: flask-script 

#  Flask-Script命令行插件 
官网：http://flask-script.readthedocs.io/en/latest/
pip install Flask-Script

当前版本:2.0.5

需要一个 Manager实例，如：
```

# manage.py
from flask_script import Manager
from myapp import app
#初始化
manager = Manager(app)
#定义一个命令，名称为hello,选项无。
@manager.command
def hello():
    print "hello"
if __name__ == "__main__":
    manager.run()  #必须的

```
执行
```

python manage.py hello
> hello

```
有三种方式可以定义命令：
  * 继承Command类
  * 使用@Command装饰器
  * 使用@option装饰器
```

from flask_script import Command

class Hello(Command):
    "prints hello world"

    def run(self):
        print "hello world"

manager.add_command('hello', Hello())

```
```

@manager.command
def hello():
    "Just say hello"
    print "hello"

```
```

@manager.option('-n', '--name', help='Your name')
def hello(name):
    print "hello", name

```
    
帮助选项定义：
```

manager = Manager()
manager.help_args = ('-h', '-?', '--help')

```


选项进一步：
```

@manager.option('-n', '--name', dest='name', default='joe')
def hello(name):
    print "hello", name

@manager.option('-n', '--name', dest='name', default='joe')
@manager.option('-u', '--url', dest='url', default=None)
def hello(name, url):
    if url is None:
        print "hello", name
    else:
        print "hello", name, "from", url

```
        
**获取用户输入**
```

from flask_script import Manager, prompt_bool

from myapp import app
from myapp.models import db

manager = Manager(app)

@manager.command
def dropdb():
    if prompt_bool("Are you sure you want to lose all your data"):
        db.drop_all()

```
 > python manage.py dropdb
Are you sure you want to lose all your data ? [N]


默认内置命令，如Server
```

from flask_script import Server, Manager
from myapp import create_app

manager = Manager(create_app)
manager.add_command("runserver", Server())

if __name__ == "__main__":
    manager.run()

```

Sub-Managers： 略。

对于扩展开发者：例如：
```

manager = Manager(usage="Perform database operations")

@manager.command
def drop():
    "Drops database tables"
    if prompt_bool("Are you sure you want to lose all your data"):
        db.drop_all()


@manager.command
def create(default_data=True, sample_data=False):
    "Creates database tables from sqlalchemy models"
    db.create_all()
    populate(default_data, sample_data)


@manager.command
def recreate(default_data=True, sample_data=False):
    "Recreates database tables (same as issuing 'drop' and then 'create')"
    drop()
    create(default_data, sample_data)


@manager.command
def populate(default_data=False, sample_data=False):
    "Populate database with default data"
    from fixtures import dbfixture

    if default_data:
        from fixtures.default_data import all
        default_data = dbfixture.data(*all)
        default_data.setup()

    if sample_data:
        from fixtures.sample_data import all
        sample_data = dbfixture.data(*all)
        sample_data.setup()

```
```

manager = Manager(app)
from flask.ext.database import manager as database_manager
manager.add_command("database", database_manager)

```


错误处理:略。。。

