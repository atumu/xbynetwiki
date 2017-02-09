title: flask-appbuilder 

#  Flask-appbuilder快速原型插件 
官网：http://flask-appbuilder.readthedocs.io/en/latest/
当前版本：1.8.1

pip install flask-appbuilder


[exmaples](https://github.com/dpgaspar/Flask-AppBuilder/tree/master/examples)
[live demo](http://flaskappbuilder.pythonanywhere.com/)

使用Flask-appbuilder的项目:
[caravel](https://github.com/airbnb/caravel):数据可视化查询平台
[Flog](https://github.com/cryptonomicon314/flog):博客系统。[demo](http://demo-ninmesara.rhcloud.com/category/about)

##  特点： 
Flask-AppBuilder是Flask开发的一个快速原型框架。把flask的工具链做了整合，使其功能全面
includes detailed security, auto CRUD generation for your models, google charts and much more

包含的特性：

**Database**
  * SQLAlchemy, multiple database support: sqlite, MySQL, ORACLE, MSSQL, DB2 etc.
  * Partial support for MongoDB using MongoEngine.
  * Multiple database connections support (Vertical partitioning).
  * Easy mixin audit to models (created/changed by user, and timestamps).

**Security**
  * Automatic permissions lookup, based on exposed methods. It will grant all permissions to the Admin Role.
  * Inserts on the Database all the detailed permissions possible on your application.
  * Public (no authentication needed) and Private permissions.
  * Role based permissions.
  * Authentication support for OAuth, OpenID, Database, LDAP and REMOTE_USER environ var.
  * Support for self user registration.

**Views and Widgets**
  * Automatic menu generation.
  * Automatic CRUD generation.
  * Multiple actions on db records.
  * Big variety of filters for your lists.
  * Various view widgets: lists, master-detail, list of thumbnails etc
  * Select2, Datepicker, DateTimePicker
  * Related Select2 fields.
  * Google charts with automatic group by or direct values and filters.
  * AddOn system, write your own and contribute.

**Forms**
  * Automatic, Add, Edit and Show from Database Models
  * Labels and descriptions for each field.
  * Automatic base validators from model's definition.
  * Custom validators, extra fields, custom filters for related dropdown lists.
  * Image and File support for upload and database field association. It will handle everything for you.
  * Field sets for Form's (Django style).

**i18n**
  * Support for multi-language via Babel
  * Bootstrap 3.1.1 CSS and js, with Select2 and DatePicker
  * Font-Awesome icons, for menu icons and actions.


依赖：
  * flask
  * click
  * colorama
  * flask-sqlalchemy
  * flask-login
  * flask-openid
  * flask-wtform
  * flask-Babel

##  初始化安装 
pip install flask-appbuilder
一旦安装好之后，就可以使用命令行工具fabmanager 来创建应用的骨架，同时创建管理员用户。
(venv)$** fabmanager create-app**
Your new app name: first_app
Your engine type, SQLAlchemy or MongoEngine [SQLAlchemy]:
Downloaded the skeleton app, good coding!
**(venv)$ cd first_app
(venv)$ fabmanager create-admin**
Username [admin]:
User first name [admin]:
User last name [user]:
Email [admin@fab.org]:
Password:
Repeat for confirmation:
注意：你有两种骨架(skeleton)的选择, SQLAlchemy default or MongoEngine ，如果要使用MongoDB，你需要安装相应扩展： **flask-mongoengine**

然后便可以运行骨架应用了：
(venv)$ fabmanager run
之后便可以访问： http://localhost:8080.

**语言配置：**
```

BABEL_DEFAULT_LOCALE = 'en'
BABEL_DEFAULT_FOLDER = 'translations'
LANGUAGES = {
    'en':{'flag':'gb','name':'English'}
}

```

如果需要保存图片到数据库的话，你还需要安装pip install pillow

##  命令行管理工具fabmanager 
&lt;nowiki&gt;fabmanager，默认会从app/__init__.py来加载AppBuilder实例appbuilder,即 import appbuilder from app/__init__.py &lt;/nowiki&gt;
  * babel-compile - Babel, Compiles all translations
  * babel-extract - Babel, Extracts and updates all messages.
  * create-admin - Creates an admin user
  * create-app - Create a Skeleton application (SQLAlchemy or MongoEngine).
  * create-addon - Create a Skeleton AddOn.
  * create-db - Create all your database objects (SQLAlchemy only)
  * collect-static - 从flask-appbuilder 复制static文件夹到当前应用
  * list-users - List all users on the database.
  * **list-views** - List all registered views.
  * reset-password - Resets a user’s password.
  * run - Runs Flask dev web server.
  * security-cleanup - Cleanup unused permissions from views and roles.
  * upgrade-db - Upgrade your database after F.A.B upgrade.
  * version - Flask-AppBuilder package version.
帮助：$ fabmanager create-app --help

##  配置项 
http://flask-appbuilder.readthedocs.io/en/latest/config.html
config.py
**SQLALCHEMY_DATABASE_URI**	DB connection string (flask-sqlalchemy)	Cond.(条件性选择)
MONGODB_SETTINGS	DB connection string (flask-mongoengine)	Cond.(条件性选择)
**AUTH_TYPE** = 0 | 1 | 2 | 3 | 4 or AUTH_TYPE = AUTH_OID, AUTH_DB,AUTH_LDAP, AUTH_REMOTE AUTH_OAUTH 必须配置

AUTH_USER_REGISTRATION = True|False	是否允许用户注册	（可选）
AUTH_USER_REGISTRATION_ROLE	设置一个用户注册后的角色名，如果允许用户注册，这是必须配置的。

AUTH_LDAP_XXX系列配置略，请看官网

AUTH_ROLE_ADMIN	      Configure the name of the admin role.	No
AUTH_ROLE_PUBLIC	Special Role that holds the public permissions, no authentication needed.	No

**APP_NAME**	The name of your application.	No
APP_THEME	Various themes for you to choose from (bootwatch).	No
APP_ICON	path of your application icons will be shown on the left side of the menu	No

ADDON_MANAGERS	A list of addon manager’s classes Take a look at addon chapter on docs.	No

**UPLOAD_FOLDER**	Files upload folder. Mandatory for file uploads.	No
**FILE_ALLOWED_EXTENSIONS**	Tuple with allower extensions. FILE_ALLOWED_EXTENSIONS = (‘txt’,’doc’)	No
**IMG_UPLOAD_FOLDER**	Image upload folder. Mandatory for image uploads.	No
**IMG_UPLOAD_URL**	Image relative URL. Mandatory for image uploads.	No
**IMG_SIZE**	tuple to define default image resize. (width, height, True|False).	No

**BABEL_DEFAULT_LOCALE**	Babel’s default language.	No
**LANGUAGES**	A dictionary mapping the existing languages with the countries name and flag


Using config.py
```

app = Flask(__name__)
app.config.from_object('config')

```

项目骨架的config.py内容：
```

import os
from flask_appbuilder.security.manager import AUTH_OID, AUTH_REMOTE_USER, AUTH_DB, AUTH_LDAP, AUTH_OAUTH
basedir = os.path.abspath(os.path.dirname(__file__))

# Your App secret key
SECRET_KEY = '\2\1thisismyscretkey\1\2\e\y\y\h'

# The SQLAlchemy connection string.
SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'app.db')
#SQLALCHEMY_DATABASE_URI = 'mysql://myapp@localhost/myapp'
#SQLALCHEMY_DATABASE_URI = 'postgresql://root:password@localhost/myapp'

# Flask-WTF flag for CSRF
CSRF_ENABLED = True

#------------------------------
# GLOBALS FOR APP Builder 
#------------------------------
# Uncomment to setup Your App name
#APP_NAME = "My App Name"

# Uncomment to setup Setup an App icon
#APP_ICON = "static/img/logo.jpg"

#----------------------------------------------------
# AUTHENTICATION CONFIG
#----------------------------------------------------
# The authentication type
# AUTH_OID : Is for OpenID
# AUTH_DB : Is for database (username/password()
# AUTH_LDAP : Is for LDAP
# AUTH_REMOTE_USER : Is for using REMOTE_USER from web server
AUTH_TYPE = AUTH_DB

# Uncomment to setup Full admin role name
#AUTH_ROLE_ADMIN = 'Admin'

# Uncomment to setup Public role name, no authentication needed
#AUTH_ROLE_PUBLIC = 'Public'

# Will allow user self registration
#AUTH_USER_REGISTRATION = True

# The default user self registration role
#AUTH_USER_REGISTRATION_ROLE = "Public"

# When using LDAP Auth, setup the ldap server
#AUTH_LDAP_SERVER = "ldap://ldapserver.new"

# Uncomment to setup OpenID providers example for OpenID authentication
#OPENID_PROVIDERS = [
#    { 'name': 'Yahoo', 'url': 'https://me.yahoo.com' },
#    { 'name': 'AOL', 'url': 'http://openid.aol.com/<username>' },
#    { 'name': 'Flickr', 'url': 'http://www.flickr.com/<username>' },
#    { 'name': 'MyOpenID', 'url': 'https://www.myopenid.com' }]
#---------------------------------------------------
# Babel config for translations
#---------------------------------------------------
# Setup default language
BABEL_DEFAULT_LOCALE = 'en'
# Your application default translation path
BABEL_DEFAULT_FOLDER = 'translations'
# The allowed translation for you app
LANGUAGES = {
    'en': {'flag':'gb', 'name':'English'},
    'pt': {'flag':'pt', 'name':'Portuguese'},
    'pt_BR': {'flag':'br', 'name': 'Pt Brazil'},
    'es': {'flag':'es', 'name':'Spanish'},
    'de': {'flag':'de', 'name':'German'},
    'zh': {'flag':'cn', 'name':'Chinese'},
    'ru': {'flag':'ru', 'name':'Russian'},
    'pl': {'flag':'pl', 'name':'Polish'}
}
#---------------------------------------------------
# Image and file configuration
#---------------------------------------------------
# The file upload folder, when using models with files
UPLOAD_FOLDER = basedir + '/app/static/uploads/'

# The image upload folder, when using models with images
IMG_UPLOAD_FOLDER = basedir + '/app/static/uploads/'

# The image upload url, when using models with images
IMG_UPLOAD_URL = '/static/uploads/'
# Setup image size default is (300, 200, True)
#IMG_SIZE = (300, 200, True)

# Theme configuration
# these are located on static/appbuilder/css/themes
# you can create your own and easily use them placing them on the same dir structure to override
#APP_THEME = "bootstrap-theme.css"  # default bootstrap
#APP_THEME = "cerulean.css"
#APP_THEME = "amelia.css"
#APP_THEME = "cosmo.css"
#APP_THEME = "cyborg.css"  
#APP_THEME = "flatly.css"
#APP_THEME = "journal.css"
#APP_THEME = "readable.css"
#APP_THEME = "simplex.css"
#APP_THEME = "slate.css"   
#APP_THEME = "spacelab.css"
#APP_THEME = "united.css"
#APP_THEME = "yeti.css"

```

##  视图 
##  基础视图 
继承BaseView，指定route_base，使用@expose导出方法路径，然后由F.A.B帮你将视图类注册到路由。
```

from flask.ext.appbuilder import AppBuilder, expose, BaseView
from app import appbuilder

class MyView(BaseView):
    route_base = "/myview"

    @expose('/method1/<string:param1>')
    def method1(self, param1):
        # do something with param1
        # and return it
        return param1

    @expose('/method2/<string:param1>')
    def method2(self, param1):
        # do something with param1
        # and render it
        param1 = 'Hello %s' % (param1)
        return param1

appbuilder.add_view_no_menu(MyView())

@appbuilder.app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html', base_template=appbuilder.base_template, appbuilder=appbuilder), 404

```

上面注册了两个路由：
/myview/method1/<string:param1>
/myview/method2/<string:param1>

flask.ext.appbuilder.baseviews.expose(url='/', methods=('GET', ))
flask.ext.appbuilder.security.decorators.has_access(f) By default the permission’s name is the methods name.

现在方法没有进行任何权限控制，如果你需要进行权限控制，可以使用 @has_access 装饰器来装饰。
```

from flask.ext.appbuilder import AppBuilder, BaseView, expose, has_access
from app import appbuilder
class MyView(BaseView):
    default_view = 'method1'
    
    @expose('/method1/')
    @has_access
    def method1(self):
        # do something with param1
        # and return to previous page or index
        return 'Hello'
        
    @expose('/method2/<string:param1>')
    @has_access
    def method2(self, param1):
        # do something with param1
        # and render template with param
        param1 = 'Goodbye %s' % (param1)
        return param1

appbuilder.add_view(MyView, "Method1", category='My View')
appbuilder.add_link("Method2", href='/myview/method2/john', category='My View')

```
上面会创建一个菜单。
![](/data/dokuwiki/python/pasted/20161021-220322.png)

接下来创建模板：<PROJECT_NAME>/app/templates/method3.html
```

{% extends "appbuilder/base.html" %}
{% block content %}
    <h1>![](/data/dokuwikiparam1)</h1>
{% endblock %}

```
```

@expose('/method3/<string:param1>')
@has_access
def method3(self, param1):
    # do something with param1
    # and render template with param
    param1 = 'Goodbye %s' % (param1)
    self.update_redirect()
    return self.render_template('method3.html',
                           param1 = param1)
appbuilder.add_link("Method3", href='/myview/method3/john', category='My View')

```
self.update_redirect()的用处：redirect的算法使用cookie保持5条导航历史，这对于redirect back是非常方便的，这样做可以保持URL参数，较好的用户体验。
但是这样一来，你必须调用self.update_redirect() 来讲当前url插入导航历史。但是有个时候可能你也不会需要这个特性，比如表单验证失败。

##  Form Views 
继承SimpleFormView or PublicFormView，来方便地处理表单。
如果需要结合WTForms使用，那么你的表单对象必须继承F.A.B的DynamicForm类
```

from wtforms import Form, StringField
from wtforms.validators import DataRequired
from flask.ext.appbuilder.fieldwidgets import BS3TextFieldWidget
from flask.ext.appbuilder.forms import DynamicForm

from flask_appbuilder import SimpleFormView
from flask.ext.babelpkg import lazy_gettext as _
#先定义表单
class MyForm(DynamicForm):
    field1 = StringField(('Field1'),
        description=('Your field number one!'),
        validators = [DataRequired()], widget=BS3TextFieldWidget())
    field2 = StringField(('Field2'),
        description=('Your field number two!'), widget=BS3TextFieldWidget())
#再定义表单视图类
class MyFormView(SimpleFormView):
    form = MyForm
    form_title = 'This is my first form view'
    message = 'My form submitted'
    
	#form_get用于表单返回前的预处理，如填充数据
    def form_get(self, form):
        form.field1.data = 'This was prefilled'
        
	#form_post用于表单提交后的处理。
    def form_post(self, form):
        # post process form
        flash(self.message, 'info')

appbuilder.add_view(MyFormView, "My form View", icon="fa-group", label=_('My form View'),
                     category="My Forms", category_icon="fa-cogs")

```
重要的几个属性：
  * form_title:	The title to be presented (this is mandatory)
  * form_columns:	The form column names to include
  * form:	Your form class (WTForm) (this is mandatory)

SimpleFormView继承自BaseView

生成的URL为：**/myformview/form**

##  创建一个简单的联系人应用 
http://flask-appbuilder.readthedocs.io/en/latest/quickhowto.html
1、先定义模型：
ContactGroup，Contact
```

from sqlalchemy import Column, Integer, String, ForeignKey, Date
from sqlalchemy.orm import relationship
from flask.ext.appbuilder import Model

class ContactGroup(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique = True, nullable=False)

    def __repr__(self):
        return self.name
      
class Contact(Model):
    id = Column(Integer, primary_key=True)
    name =  Column(String(150), unique = True, nullable=False)
    address =  Column(String(564), default='Street ')
    birthday = Column(Date)
    personal_phone = Column(String(20))
    personal_celphone = Column(String(20))
    contact_group_id = Column(Integer, ForeignKey('contact_group.id'))
    contact_group = relationship("ContactGroup")

    def __repr__(self):
        return self.name

```
2、定义视图类：继承自ModelView（它继承自BaseCRUDView-》BaseModelView ）
```

from flask.ext.appbuilder import ModelView
from flask.ext.appbuilder.models.sqla.interface import SQLAInterface

class GroupModelView(ModelView):
    datamodel = SQLAInterface(ContactGroup)
    related_views = [ContactModelView]

```
必须属性：datamodel:	is the db abstraction layer. Initialize it with your view’s model.
可选属性：related_views:	if you want a master/detail view on the show and edit. F.A.B. will relate 1/N relations automatically, it will display a show or edit view with tab (or accordion) with a list related record. You can relate charts also.

```

class ContactModelView(ModelView):
    datamodel = SQLAInterface(Contact)

    label_columns = {'contact_group':'Contacts Group'}
    list_columns = ['name','personal_celphone','birthday','contact_group']

    show_fieldsets = [
        ('Summary',{'fields':['name','address','contact_group']}),
        ('Personal Info',{'fields':['birthday','personal_phone','personal_celphone'],'expanded':False}),
        ]
    search_columns = ['name','address']

```
说明：
  * label_columns:	defines the labels for your columns. The framework will define the missing ones for you, with a pretty version of your column names.
  * show_fieldsets:	A fieldset (Django style). You can use show_fieldsets, add_fieldsets, edit_fieldsets customize the show, add and edit views independently.
  * search_columns

创建菜单：
```

db.create_all()
appbuilder.add_view(GroupModelView, "List Groups",icon = "fa-folder-open-o",category = "Contacts",
                category_icon = "fa-envelope")
appbuilder.add_view(ContactModelView, "List Contacts",icon = "fa-envelope",category = "Contacts")

```

###  ModelView输出的endpoint 
Your ModelView classes expose the following methods has flask endpoints
  * list
  * show
  * add
  * edit
  * delete
  * download
  * action
  * API methods

具体介绍见http://flask-appbuilder.readthedocs.io/en/latest/quickhowto.html#exposed-methods

###  Extra Views 
ModelView，MultipleView，MasterDetailView都继承自BaseCRUDView

**MultipleView：将多个视图合并到一个页面显示：**
```

from flask.ext.appbuilder import MultipleView
class MultipleViewsExp(MultipleView):
    views = [GroupModelView, ContactModelView]

appbuilder.add_view(MultipleViewsExp, "Multiple Views", icon="fa-envelope", category="Contacts")

```

**MasterDetailView ：如名字一样：**
```

class GroupMasterView(MasterDetailView):
    datamodel = SQLAInterface(ContactGroup)
    related_views = [ContactModelView]

```
    

##  Chart Views 
To implement views with **google charts**, use all inherited classes from **BaseChartView**, these are:
  * DirectChartView:Display direct data charts with multiple series, no group by is applied.
  * GroupByChartView:Displays grouped data with multiple series.
###  Direct Data Charts 
例如：显示失业率的演变与高等教育的人口的比例
定义模型
```

class CountryStats(Model):
    id = Column(Integer, primary_key=True)
    stat_date = Column(Date, nullable=True)
    population = Column(Float)
    unemployed_perc = Column(Float)
    poor_perc = Column(Float)
    college = Column(Float)

```
定义一个函数计算大学生比例：
```

def college_perc(self):
    if self.population != 0:
        return (self.college*100)/self.population
    else:
        return 0.0

```
定义视图
```

from flask.ext.appbuilder.charts.views import DirectByChartView
from flask.ext.appbuilder.model.sqla.interface import SQLAInterface

class CountryDirectChartView(DirectByChartView):
    datamodel = SQLAInterface(CountryStats)
    chart_title = 'Direct Data Example'

    definitions = [
    {
        'label': 'Unemployment',
        'group': 'stat_date',
        'series': ['unemployed_perc',
                   'college_perc']
    }
]

```
![](/data/dokuwiki/python/pasted/20161021-230110.png)
我们也可以同时定义多张图表：
```

class CountryDirectChartView(DirectByChartView):
    datamodel = SQLAInterface(CountryStats)
    chart_title = 'Direct Data Example'

    definitions = [
    {
        'label': 'Unemployment',
        'group': 'stat_date',
        'series': ['unemployed_perc',
                   'college_perc']
    },
    {
        'label': 'Poor',
        'group': 'stat_date',
        'series': ['poor_perc',
                   'college_perc']
    }
]
    
appbuilder.add_view(CountryDirectChartView, "Show Country Chart", icon="fa-dashboard", category="Statistics")

```
BaseChartView继承自BaseModelView 
**BaseChartView比较重要的属性：**
  * chart_title:	The Title of the chart (can be used with babel of course).
  * group_by_label:	The label that will be displayed before the buttons for choosing the chart.
  * chart_type:	The chart type PieChart, ColumnChart or LineChart
  * chart_3d:	= True or false label like: ‘true’
  * width:	The charts width
  * height:	The charts height
###  Grouped Data Charts 
支持分组与聚合
```

from flask.ext.appbuilder import Model

class Country(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique = True, nullable=False)

    def __repr__(self):
        return self.name


class CountryStats(Model):
    id = Column(Integer, primary_key=True)
    stat_date = Column(Date, nullable=True)
    population = Column(Float)
    unemployed_perc = Column(Float)
    poor_perc = Column(Float)
    college = Column(Float)
    country_id = Column(Integer, ForeignKey('country.id'), nullable=False)
    country = relationship("Country")

    def college_perc(self):
        if self.population != 0:
            return (self.college*100)/self.population
        else:
            return 0.0

    def month_year(self):
        return datetime.datetime(self.stat_date.year, self.stat_date.month, 1)

```
```

from flask.ext.appbuilder.charts.views import GroupByChartView
from flask.ext.appbuilder.models.group import aggregate_count, aggregate_sum, aggregate_avg
from flask.ext.appbuilder.model.sqla.interface import SQLAInterface


class CountryGroupByChartView(GroupByChartView):
    datamodel = SQLAInterface(CountryStats)
    chart_title = 'Statistics'

    definitions = [
        {
            'label': 'Country Stat',
            'group': 'country',
            'series': [(aggregate_avg, 'unemployed_perc'),
                       (aggregate_avg, 'population'),
                       (aggregate_avg, 'college_perc')
                      ]
        }
    ]

appbuilder.add_view(CountryGroupByChartView, "Show Country Chart", icon="fa-dashboard", category="Statistics")

```
F.A.B. has already some aggregation functions that you can use, for** count, sum and average**. On this example we are using average

A different and interesting example is to group data monthly from all countries, this will show the use of **formater** property:
```

import calendar
from flask.ext.appbuilder.charts.views import GroupByChartView
from flask.ext.appbuilder.models.group import aggregate_count, aggregate_sum, aggregate_avg
from flask.ext.appbuilder.model.sqla.interface import SQLAInterface

def pretty_month_year(value):
    return calendar.month_name[value.month] + ' ' + str(value.year)


class CountryGroupByChartView(GroupByChartView):
    datamodel = SQLAInterface(CountryStats)
    chart_title = 'Statistics'

    definitions = [
        {
            'group': 'month_year',
            'formatter': pretty_month_year,
            'series': [(aggregate_avg, 'unemployed_perc'),
                       (aggregate_avg, 'college_perc')
            ]
        }
    ]

```
自定义聚合函数支持，以内置的aggregate_count为例，写法如下
```

@aggregate(_('Count of'))
def aggregate_count(items, col):
    return len(list(items))

```

##  Model Views with Files and Images 
Define your model (models.py)
```

from flask.ext.appbuilder import Model
from flask.ext.appbuilder.model.mixins import ImageColumn

class Person(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(150), unique = True, nullable=False)
    photo = Column(ImageColumn(size=(300, 300, True), thumbnail_size=(30, 30, True)))

    def photo_img(self):
        im = ImageManager()
        if self.photo:
            return Markup('<a href="' + url_for('PersonModelView.show',pk=str(self.id)) +\
             '" class="thumbnail"><img src="' + im.get_url(self.photo) +\
              '" alt="Photo" class="img-rounded img-responsive"></a>')
        else:
            return Markup('<a href="' + url_for('PersonModelView.show',pk=str(self.id)) +\
             '" class="thumbnail"><img src="//:0" alt="Photo" class="img-responsive"></a>')

    def photo_img_thumbnail(self):
        im = ImageManager()
        if self.photo:
            return Markup('<a href="' + url_for('PersonModelView.show',pk=str(self.id)) +\
             '" class="thumbnail"><img src="' + im.get_url_thumbnail(self.photo) +\
              '" alt="Photo" class="img-rounded img-responsive"></a>')
        else:
            return Markup('<a href="' + url_for('PersonModelView.show',pk=str(self.id)) +\
             '" class="thumbnail"><img src="//:0" alt="Photo" class="img-responsive"></a>')

```
Define your Views (views.py)
```

from flask.ext.appbuilder import ModelView
from flask.ext.appbuilder.models.sqla.interface import SQLAInterface

class PersonModelView(ModelView):
    datamodel = SQLAInterface(Person)

    list_widget = ListThumbnail

    label_columns = {'name':'Name','photo':'Photo','photo_img':'Photo', 'photo_img_thumbnail':'Photo'}
    list_columns = ['photo_img_thumbnail', 'name']
    show_columns = ['photo_img','name']

```
##  最小化的应用 
```

import os
from flask import Flask
from flask.ext.appbuilder import SQLA, AppBuilder

# init Flask
app = Flask(__name__)

# Basic config with security for forms and session cookie
basedir = os.path.abspath(os.path.dirname(__file__))
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'app.db')
app.config['CSRF_ENABLED'] = True
app.config['SECRET_KEY'] = 'thisismyscretkey'

# Init SQLAlchemy
db = SQLA(app)
# Init F.A.B.
appbuilder = AppBuilder(app, db.session)

# Run the development server
app.run(host='0.0.0.0', port=8080, debug=True)

```
##  模型关系 
Many to One，Many to Many
###  Many to One 
```

import datetime
from sqlalchemy import Column, Integer, String, ForeignKey, Date, Text
from sqlalchemy.orm import relationship
from flask.ext.appbuilder import Model


class Department(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)

    def __repr__(self):
        return self.name


class Function(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)

    def __repr__(self):
        return self.name


def today():
    return datetime.datetime.today().strftime('%Y-%m-%d')


class Employee(Model):
    id = Column(Integer, primary_key=True)
    full_name = Column(String(150), nullable=False)
    address = Column(Text(250), nullable=False)
    fiscal_number = Column(Integer, nullable=False)
    employee_number = Column(Integer, nullable=False)
    
    department_id = Column(Integer, ForeignKey('department.id'), nullable=False)
    department = relationship("Department")
    
    function_id = Column(Integer, ForeignKey('function.id'), nullable=False)
    function = relationship("Function")
    
    begin_date = Column(Date, default=today, nullable=False)
    end_date = Column(Date, nullable=True)

    def __repr__(self):
        return self.full_name

```
This has two, one to many relations:
  * One employee belongs to a department and a department has many employees
  * One employee executes a function and a function is executed by many employees.

###  Many to Many 
```

class Benefit(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)

    def __repr__(self):
        return self.name
#Then define the association table between Employee and Benefit, then add the relation to benefit on the Employee model:
assoc_benefits_employee = Table('benefits_employee', Model.metadata,
                                      Column('id', Integer, primary_key=True),
                                      Column('benefit_id', Integer, ForeignKey('benefit.id')),
                                      Column('employee_id', Integer, ForeignKey('employee.id'))
)


class Employee(Model):
    id = Column(Integer, primary_key=True)
    full_name = Column(String(150), nullable=False)
    address = Column(Text(250), nullable=False)
    fiscal_number = Column(Integer, nullable=False)
    employee_number = Column(Integer, nullable=False)
    
    department_id = Column(Integer, ForeignKey('department.id'), nullable=False)
    department = relationship("Department")
    
    function_id = Column(Integer, ForeignKey('function.id'), nullable=False)
    function = relationship("Function")
    
    benefits = relationship('Benefit', secondary=assoc_benefits_employee, backref='employee')

    begin_date = Column(Date, default=today, nullable=False)
    end_date = Column(Date, nullable=True)

    def __repr__(self):
        return self.full_name

```
##  定义模型视图中的action 
![](/data/dokuwiki/python/pasted/20161021-233644.png)
```

from flask.ext.appbuilder.actions import action
from flask.ext.appbuilder import ModeView
from flask.ext.appbuilder.models.sqla.interface import SQLAInterface

class GroupModelView(ModelView):
    datamodel = SQLAInterface(Group)
    related_views = [ContactModelView]

    @action("myaction","Do something on this record","Do you really want to?","fa-rocket")
    def myaction(self, item):
        """
            do something with the item record
        """
        return redirect(self.get_redirect())
    @action("muldelete", "Delete", "Delete all Really?", "fa-rocket", single=False)
    def muldelete(self, items):
    	self.datamodel.delete_all(items)
      self.update_redirect()
    	return redirect(self.get_redirect())

```
##  高级配置 
**安全：**
base_permissions属性
```

class GroupModelView(ModelView):
    datamodel = SQLAInterface(Group)
    base_permissions = ['can_add','can_delete']

```
** Custom Fields：**
```

from flask.ext.appbuilder.models.decorators import renders

class MyModel(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique = True, nullable=False)
    custom = Column(Integer(20))

    @renders('custom')
    def my_custom(self):
    # will render this columns as bold on ListWidget
        return Markup('<b>' + custom + '</b>')

```
On your view reference your method as a column on list:
```

class MyModelView(ModelView):
    datamodel = SQLAInterface(MyTable)
    list_columns = ['name', 'my_custom']

```

**Base Filtering：**
base_filters属性
```

from flask import g
from flask.ext.appbuilder import ModelView
from flask.ext.appbuilder.models.sqla.interface import SQLAInterface
from flask_appbuilder.models.sqla.filters import FilterStartsWith, FilterEqualFunction
# If your using Mongo Engine you should import filters like this, everything else is exactly the same
# from flask_appbuilder.models.mongoengine.filters import FilterStartsWith, FilterEqualFunction

from .models import MyTable
def get_user():
    return g.user
class MyView(ModelView):
    datamodel = SQLAInterface(MyTable)
    base_filters = [['created_by', FilterEqualFunction, get_user],
                    ['name', FilterStartsWith, 'a']]

```
**Default Order**
```

class MyView(ModelView):
    datamodel = SQLAInterface(MyTable)
    base_order = ('my_col_to_be_ordered','asc')

```
   
**Template Extra Arguments**：
```

class MyView(ModelView):
    datamodel = SQLAInterface(MyTable)
    extra_args = {'my_extra_arg':'SOMEVALUE'}
    show_template = 'my_show_template.html'

```
##  定制 
Changing themes：
APP_THEME = "spacelab.css"
或
app.config['APP_THEME'] = "spacelab.css"

改变首页
1、模板<PROJECT_NAME>/app/templates/my_index.html
2 - Define an IndexView
don’t define this view on views.py, put it on a separate file like **index.py**:
```

from flask.ext.appbuilder import IndexView

class MyIndexView(IndexView):
    index_template = 'index.html'

```
3 - Tell F.A.B to use your index view, when initializing AppBuilder:
```

from app.index import MyIndexView
app = Flask(__name__)
app.config.from_object('config')
db = SQLA(app)
appbuilder = AppBuilder(app, db.session, indexview=MyIndexView)

```
You can override IndexView index function to display a different view if a user is logged in or not.

改版页脚
./your_root_project_path/app/templates/appbuilder/footer.html
Actually you can override any given F.A.B. template.

改变菜单结构：
Change the reversed navbar style, on AppBuilder initialization:
```

appbuilder = AppBuilder(app, db, menu=Menu(reverse=False))

```
Add your own menu links, on a default reversed navbar:
```

# Register a view, rendering a top menu without icon
appbuilder.add_view(MyModelView, "My View")
# Register a view, a submenu "Other View" from "Other" with a phone icon
appbuilder.add_view(MyOtherModelView, "Other View", icon='fa-phone', category="Others")
# Register a view, with label for babel support (internationalization), setup an icon for the category.
appbuilder.add_view(MyOtherModelView, "Other View", icon='fa-phone', label=lazy_gettext('Other View'),
                category="Others", category_label=lazy_gettext('Other'), category_label='fa-envelope')
# Add a link
appbuilder.add_link("google", href="www.google.com", icon = "fa-google-plus")

```
Add separators:
```

# Register a view, rendering a top menu without icon
appbuilder.add_view(MyModelView1, "My View 1", category="My Views")
appbuilder.add_view(MyModelView2, "My View 2", category="My Views")
appbuilder.add_separator("My Views")
appbuilder.add_view(MyModelView3, "My View 3", category="My Views")

```

Changing Widgets and Templates：
F.A.B. has a collection of widgets to change your views presentation, you can create your own and override
All views have templates that will display widgets in a certain layout. For example, on the edit or show view, you can display the related list (from related_views) on the same page, or as tab (default).
```

class ServerDiskTypeModelView(ModelView):
    datamodel = SQLAInterface(ServerDiskType)
    list_columns = ['quantity', 'disktype']
    list_widget = ListBlock

class ServerModelView(ModelView):
    datamodel = SQLAInterface(Server)
    related_views = [ServerDiskTypeModelView]

    show_template = 'appbuilder/general/model/show_cascade.html'
    edit_template = 'appbuilder/general/model/edit_cascade.html'

    list_columns = ['name', 'serial']
    order_columns = ['name', 'serial']
    search_columns = ['name', 'serial']

```
    
###  模板说明 
CSS and Javascript：
```

{% extends 'appbuilder/baselayout.html' %}

{% block head_css %}
    ![](/data/dokuwiki super() )
    <script src="![](/data/dokuwikiurl_for('static',filename='css/your_css_file.js'))"></script>
{% endblock %}

{% block head_js %}
    ![](/data/dokuwiki super() )
    <script src="![](/data/dokuwikiurl_for('static',filename='js/your_js_file.js'))"></script>
{% endblock %}
{% block tail_js %}
    ![](/data/dokuwiki super() )
    <script src="![](/data/dokuwikiurl_for('static',filename='js/your_js_file.js'))"></script>
{% endblock %}

```

**改变base_template:**
```

appbuilder = AppBuilder(app, db.session, base_template='mybase.html')

```
appbuilder/baselayout.htm主要的可覆盖块如下：
```

{% block head_meta %}
    ... HTML Meta
{% endblock %}
{% block head_css %}
    ... CSS imports (bootstrap, fontAwesome, select2, fab specific etc...
{% endblock %}
{% block head_js %}
    ... JS imports (JQuery, fab specific)
{% endblock %}
{% block body %}
    {% block navbar %}
        ... The navigation bar (Menu)
    {% endblock %}
     {% block messages %}
        ... Where the flask flash messages are shown ("Added row", etc)
      {% endblock %}
      {% block content %}
        ... All the content goes here, forms, lists, index, charts etc..
      {% endblock %}
    {% block footer %}
        ... The footer, by default its almost empty.
    {% endblock %}
{% block tail_js %}
{% endblock %}

```

导航条：
/templates/appbuilder/navbar_menu.html: This will render the navbar menus.
/templates/appbuilder/navbar_right.html: This will render the right part of the navigation bar (locale and user).

List Templates：
/templates/appbuilder/general/model/list.html
![](/data/dokuwiki/python/pasted/20161021-235946.png)
```

{% extends "appbuilder/general/model/list.html" %}
    {% block list_search scoped %}
        This Text will replace the search widget
    {% endblock %}

```
```

class ContactModelView(ModelView):
    datamodel = SQLAInterface(Contact)
    list_template = 'list_contacts.html'

```

Add Templates：
appbuilder/general/model/add.html
Show/Edit Templates：
appbuilder/general/model/edit.html

Widgets:
Library Functions:

##  多个数据源 
```

SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'app.db')

SQLALCHEMY_BINDS = {
    'my_sql1': 'mysql://root:password@localhost/quickhowto'
    'my_sql2': 'mysql://root:password@externalserver.domain.com/quickhowto2'
}
class Model1(Model):
    __bind_key__ = 'my_sql1'
    id = Column(Integer, primary_key=True)
    name =  Column(String(150), unique = True, nullable=False)


class Model2(Model):
    __bind_key__ = 'my_sql2'
    id = Column(Integer, primary_key=True)
    name =  Column(String(150), unique = True, nullable=False)


class Model3(Model):
    id = Column(Integer, primary_key=True)
    name =  Column(String(150), unique = True, nullable=False)

```
On this example:
  * Model1 will be on the local MySql instance with db ‘quickhowto’.
  * Model2 will be on the externalserver.domain.com MySql instance with db ‘quickhowto2’.
  * Model3 will be on the default connection using sqlite.


##  国际化 
Flask-Babel 支持
在项目的根创建一个babel文件夹，然后创建一个文件babel/babel.cfg，内容如下：
```

[python: **.py]
[jinja2: **/templates/**.html]
encoding = utf-8

```
创建translations:, execute on you projects root
pybabel init -i ./babel/messages.pot -d app/translations -l zh
抽取需要翻译的字符串, execute on you projects root
$ fabmanager babel-extract

代码中:
```

from flask.ext.babel import lazy_gettext as _
aaa=_('List Groups')

```

然后进行翻译工作
On app/translations/zh/LC_MESSAGES/messages.po ：
msgid "List Groups"
msgstr ""

编译翻译文件, from the root directory of your project:
$ fabmanager babel-compile
添加语言配置：
```

LANGUAGES = {
   'en': {'flag':'gb', 'name':'English'},
   'zh': {'flag':'zh', 'name':'Chinese'}
}

```


##  权限 
Auth Type支持：Database、Open ID、LDAP、REMOTE_USER、OAUTH
结合Flask-Login, OpenID requires Flask-OpenID.

权限-角色-用户-视图/菜单
两类特殊的角色：
  * Admin Role:	The framework will assign all the existing permission on views and menus to this role, automatically, this role is for authenticated users only.
  * Public Role:	This is a special role for non authenticated users, you can assign all the permissions on views and menus to this role, and everyone will access specific parts of you application.

权限：
**The framework automatically creates for you all the possible existing permissions on your views or menus, by “inspecting” your code.**
Each time you create a new view based on a model (inherit from ModelView) it will create the following permissions:
  * can list
  * can show
  * can add
  * can edit
  * can delete
  * can download
These base permissions will be associated to your view, so if you create a view named “MyModelView” you can assign to any role these permissions:
  * can list on MyModelView
  * can show on MyModelView
  * can add on MyModelView
  * can edit on MyModelView
  * can delete on MyModelView
  * can doanload on MyModelView
通过@permission_name可以覆盖默认的生成
```

class MyModelView(ModelView):
    datamodel = SQLAInterface(MyModel)
    @has_access
    @permission_name('GeneralXPTO_Permission')
    @expose(url='/xpto')
    def xpto(self):
        return "Your on xpto"

```

```

class MyModelView(ModelView):
    datamodel = SQLAInterdace(Group)

    @has_access  #所需权限can mymethod on MyModelView
    @expose('/mymethod/')
    def mymethod(self):
        # do something
        pass

```

清理:appbuilder.security_cleanup()
```

appbuilder.add_view(GroupModelView, "List Groups", category="Contacts")
appbuilder.add_view(ContactModelView, "List Contacts", category="Contacts")
appbuilder.add_separator("Contacts")
appbuilder.add_view(ContactChartView, "Contacts Chart", category="Contacts")
appbuilder.add_view(ContactTimeChartView, "Contacts Birth Chart", category="Contacts")

appbuilder.security_cleanup()

```
      
用户信息审计(Auditing):AuditMixin
```

from flask_appbuilder.models.mixins import AuditMixin
from flask_appbuilder import Model
from sqlalchemy import Column, Integer, String


class Project(AuditMixin, Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(150), unique=True, nullable=False)

```
AuditMixin will add the following columns to your model:
  * created_on: The date and time of the record creation.
  * changed_on: The last date and time of record update.
  * created_by: Who created the record.
  * changed_by: Who last modified the record.
These columns will be automatically updated by the framework upon creation or update of records. 


认证方法：
```

from flask_appbuilder.security.manager import AUTH_OID, \
                                          AUTH_REMOTE_USER, \
                                          AUTH_DB, AUTH_LDAP, \
                                          AUTH_OAUTH, \
                                          AUTH_OAUTH
AUTH_TYPE = AUTH_DB
AUTH_ROLE_ADMIN = 'My Admin Role Name'
AUTH_ROLE_PUBLIC = 'My Public Role Name'
AUTH_USER_REGISTRATION = True
AUTH_USER_REGISTRATION_ROLE = "My Public Role Name"


```

##  用户注册 
允许用户注册
通过数据库认证方式：
AUTH_TYPE = 1 # Database Authentication
AUTH_USER_REGISTRATION = True
AUTH_USER_REGISTRATION_ROLE = 'Public'
# Config for Flask-WTF Recaptcha necessary for user registration
RECAPTCHA_PUBLIC_KEY = 'GOOGLE PUBLIC KEY FOR RECAPTCHA'
RECAPTCHA_PRIVATE_KEY = 'GOOGLE PRIVATE KEY FOR RECAPTCHA'
# Config for Flask-Mail necessary for user registration
MAIL_SERVER = 'smtp.gmail.com'
MAIL_USE_TLS = True
MAIL_USERNAME = 'yourappemail@gmail.com'
MAIL_PASSWORD = 'passwordformail'
MAIL_DEFAULT_SENDER = 'fabtest10@gmail.com'

你可以通过覆盖**RegisterUserDBView**类控制这一行为
```

from flask.ext.appbuilder.security.registerviews import RegisterUserDBView

class MyRegisterUserDBView(RegisterUserDBView):
    email_template = 'register_mail.html'
    email_subject = lazy_gettext('Your Account activation')
    activation_template = 'activation.html'
    form_title = lazy_gettext('Fill out the registration form')
    error_message = lazy_gettext('Not possible to register you at the moment, try again later')
    message = lazy_gettext('Registration sent to your email')

```
After defining your own class, override **SecurityManager** class and set the registeruserdbview property with your own class:
```

from flask_appbuilder.security.sqla.manager import SecurityManager
class MySecurityManager(SecurityManager):
    registeruserdbview = MyRegisterUserDBView
    
appbuilder = AppBuilder(app, db.session, security_manager_class=MySecurityManager)

```

##  扩展开发 
http://flask-appbuilder.readthedocs.io/en/latest/addons.html

