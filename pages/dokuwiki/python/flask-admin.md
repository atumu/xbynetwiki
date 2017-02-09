title: flask-admin 

#  flask-admin快速原型插件 
Flask-Admin是一个功能齐全、简单易用的Flask扩展，让你可以为Flask应用程序增加管理界面。它受django-admin包的影响，但用这样一种方式实现，开发者拥有最终应用程序的外观、感觉和功能的全部控制权。
官网：http://flask-admin.readthedocs.io/en/latest/
当前版本:1.4.2

live demo：http://examples.flask-admin.org/ 
examples:cd flask-admin  ； python examples/simple/app.py

依赖安装：pip install -r requirements-dev.txt
安装：pip install flask-admin


How does it work?以视图类，来组织不同的视图：前提的每个页面对应视图类中的某个方法。当时有数据模型时，你可以将它的CRUD的操作都组织到一个视图类中。

##  快速入门 
```

from flask import Flask
from flask_admin import Admin

app = Flask(__name__)

admin = Admin(app, name='microblog', template_mode='bootstrap3')
# Add administrative views here

app.run()

```
你也可以使用 init_app() 方法
http://localhost:5000/admin/

添加ModelView
```

from flask_admin.contrib.sqla import ModelView

# Flask and Flask-SQLAlchemy initialization here
admin = Admin(app, name='microblog', template_mode='bootstrap3')
admin.add_view(ModelView(User, db.session))
admin.add_view(ModelView(Post, db.session))

```
添加内容到首页
```

{% extends 'admin/master.html' %}

{% block body %}
  <p>Hello world</p>
{% endblock %}

```

```

from flask import Flask
from flask.ext.admin import Admin, BaseView, expose

class MyView(BaseView):
    @expose('/')
    def index(self):
        return self.render('index.html')

app = Flask(__name__)

admin = Admin(app)
admin.add_view(MyView(name='Hello'))

app.run()

```

认证与权限：
结合flask-login:https://github.com/flask-admin/Flask-Admin/tree/master/examples/auth-flask-login.
结合flask-security:https://github.com/flask-admin/Flask-Admin/tree/master/examples/auth.
Flask-Admin没有设想任何你可以使用的身份验证系统。因此，默认的，管理界面是完全开放的。
要控制使用管理界面，当扩展BaseView类时你可以**指定is_accessible方法**。那么，举例，如果你使用Flask-Login做身份验证，下面的代码确保只有已登入的用户能访问视图：
```

class MyView(BaseView):
    def is_accessible(self):
        return login.current_user.is_authenticated()

```
你也可以实施基于策略的保密，有条件的允许或不允许使用管理界面的某些部分。如果一个用户无权使用某个特定视图，则菜单项目不可见。

在内部，**视图类工作于Flask蓝图的顶部**，因此你可以使用**url_for附带一个.前缀**来获得局部视图的URL：
```

from flask import url_for
class MyView(BaseView):
    @expose('/')
    def index(self)
        # Get URL for the test view method
        url = url_for('.test')
        return self.render('index.html', url=url)

    @expose('/test/')
    def test(self):
        return self.render('test.html')

```

**如果你要在外部生成一个特定视图的URL，应用下面的规则：**
1、你可以覆盖endpoint名称通过传送endpoint参数给视图类构造函数：
```

admin = Admin(app)
admin.add_view(MyView(endpoint='testadmin'))
# In this case, you can generate links by concatenating the view method name with an endpoint:
url_for('testadmin.index')

```
2、如果你不覆盖endpoint名称，类名的小写形式会用于生成URL，像这样：
```

url_for('myview.index')

```
3、对基于模型的视图规则不一样——模型类名称会被使用如果没有提供endpoint名称。基于模型的视图下一节解释。

自定义内置view
Adding Your Own Views：

模型视图
模型视图允许你为数据库中的每个模型增加专用的管理页面。通过创建ModelView类实例做这个，ModelView类可从Flask-Admin内置的ORM后端引入。一个SQLAlchemy后端的例子，你可以这样使用：
```

from flask.ext.admin.contrib.sqla import ModelView

# Flask and Flask-SQLAlchemy initialization here
admin = Admin(app)
admin.add_view(ModelView(User, db.session))

```
**要定制这些模型视图，你有两个选择：一是覆盖ModelView类的公有属性，二是覆盖它的方法。**
例如，假如你要禁用模型创建功能并且只在列表视力显示某些列，你可以这样做：
```

from flask.ext.admin.contrib.sqla import ModelView
# Flask and Flask-SQLAlchemy initialization here
class MyView(ModelView):
    # Disable model creation
    can_create = False

    # Override displayed fields
    column_list = ('login', 'email')

    def __init__(self, session, **kwargs):
        # You can pass name and other parameters if you want to
        super(MyView, self).__init__(User, session, **kwargs)

admin = Admin(app)
admin.add_view(MyView(db.session))

```
文件管理
Flask-Admin拥有另一个便利的特性——文件管理。它给予你管理服务器文件的能力（上传、删除、重命名等）。
这是一个简单的例子：
```

from flask.ext.admin.contrib.fileadmin import FileAdmin
import os.path as op
# Flask setup here
admin = Admin(app)
path = op.join(op.dirname(__file__), 'static')
admin.add_view(FileAdmin(path, '/static/', name='Static Files'))

```
你可以禁用上传、禁用文件或目录删除、限制文件上传类型等等。关于怎么做这些请查看flask.ext.admin.contrib.fileadmin文档。
参考：https://segmentfault.com/a/1190000002485326
