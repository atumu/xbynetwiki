title: flask-migrate 

#  Flask-migrate数据迁移插件 
官网：http://flask-migrate.readthedocs.io/en/latest/
Flask-Migrate is an extension that handles SQLAlchemy database migrations for Flask applications **using Alembic**.
当前版本 2.0.0
pip install Flask-Migrate

##  使用flask命令方式 
设置FLASK_APP环境变量
```

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

db = SQLAlchemy(app)
migrate = Migrate(app, db)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(128))

```
$ flask db init
$ flask db migrate
$ flask db upgrade
$ flask db --help


##  使用Flask-Script方式 
```

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

db = SQLAlchemy(app)
migrate = Migrate(app, db)

manager = Manager(app)
manager.add_command('db', MigrateCommand)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(128))

if __name__ == '__main__':
    manager.run()

```
$ python manage.py db init
$ python manage.py db migrate
$ python manage.py db upgrade
$ python manage.py db --help

##  Configuration Callbacks 
```

@migrate.configure
def configure_alembic(config):
    # modify config object
    return config

```
##  Multiple Database Support 
Flask-Migrate can integrate with **the binds feature** of Flask-SQLAlchemy, making it possible to track migrations to multiple databases associated with an application.
To create a multiple database migration repository, add the - -multidb argument to the init command:
```

$ flask db init --multidb

```

