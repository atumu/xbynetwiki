title: flask-babel 

#  Flask-Babel国际化插件 
官网：https://pythonhosted.org/Flask-Babel/
当前版本：0.11.1

pip install Flask-Babel
依赖于 Jinja 2.5以上

**配置：**
在Flask初始化配置之后，创建一个Babel实例。
```

from flask import Flask
from flask.ext.babel import Babel

app = Flask(__name__)
app.config.from_pyfile('mysettings.cfg')
babel = Babel(app)

```
关于Babel的配置选项:
BABEL_DEFAULT_LOCALE 默认en
BABEL_DEFAULT_TIMEZONE 默认UTC
```

# ---------------------------------------------------
# Babel config for translations
# ---------------------------------------------------
# Setup default language
BABEL_DEFAULT_LOCALE = 'en'
# Your application default translation path
BABEL_DEFAULT_FOLDER = 'translations'
# The allowed translation for you app
LANGUAGES = {
    'en': {'flag': 'us', 'name': 'English'},
    # 'fr': {'flag': 'fr', 'name': 'French'},
    # 'zh': {'flag': 'cn', 'name': 'Chinese'},
}

```

**语言动态切换**
```

from flask import g, request

@babel.localeselector
def get_locale():
    # if a user is logged in, use the locale from the user settings
    user = getattr(g, 'user', None)
    if user is not None:
        return user.locale
    # otherwise try to guess the language from the user accept
    # header the browser transmits.  We support de/fr/en in this
    # example.  The best match wins.
    return request.accept_languages.best_match(['de', 'fr', 'en'])

@babel.timezoneselector
def get_timezone():
    user = getattr(g, 'user', None)
    if user is not None:
        return user.timezone


```
可主动调用refresh()函数来清除缓存from flask.ext.babel import refresh; refresh()

      
**格式化日期：**
可使用函数 format_datetime(), format_date(), format_time() and format_timedelta() 他们都接受一个 datetime.datetime (or datetime.date, datetime.time and datetime.timedelta对象作为第一个参数，第二个参数为格式字符串。
```

>>> app.test_request_context().push()
>>> from flask.ext.babel import format_datetime
>>> from datetime import datetime
>>> format_datetime(datetime(1987, 3, 5, 17, 12))
u'Mar 5, 1987 5:12:00 PM'
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'full')
u'Thursday, March 5, 1987 5:12:00 PM World (GMT) Time'
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'short')
u'3/5/87 5:12 PM'
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'dd mm yyy')
u'05 12 1987'
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'dd mm yyyy')
u'05 12 1987'

```

在代码中使用gettext函数来获取国际化字符串。
```

from flask.ext.babel import gettext, ngettext

gettext(u'A simple string')
gettext(u'Value: %(value)s', value=42)
ngettext(u'%(num)s Apple', u'%(num)s Apples', number_of_apples)

```

**翻译应用：**
首先你需要在应用中使用gettext()来标记你所需要翻译的字符串,对于jinja2模板文件需要使用 _()来标记，如&lt;nowiki&gt;![](/data/dokuwiki _("Dashboards") )&lt;/nowiki&gt;
接下来使用babel帮助你生成一个包含字符串的模板.pot文件，它是生成翻译文件.po的前提。创建一个文件babel.cfg，指定需要翻译文件的路径
```

[python: **.py]
[jinja2: **/templates/**.html]
extensions=jinja2.ext.autoescape,jinja2.ext.with_

```
接下来就是字符串的具体提取操作了。
pybabel extract -F babel.cfg -o messages.pot .
然后生成对应语言的翻译文件模板
pybabel init -i messages.pot -d translations -l zh
然后生成文件位于translations/zh/LC_MESSAGES/messages.po，编辑该文件进行翻译。翻译好之后，对翻译文件进行编译
pybabel compile -d translations
如果需要国际化的字符串更新或新增了呢？那就重新按照上述步骤创建一个messages.pot文件，然后调用
pybabel update -i messages.pot -d translations

如果编译过程中出现问题，请确保你的字符编码为UTF-8, export LC_CTYPE=en_US.utf-8

**懒加载**
```

from flask.ext.babel import lazy_gettext
class MyForm(formlibrary.FormBase):
    success_message = lazy_gettext(u'The form was successfully saved.')

```
