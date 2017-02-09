modifyAt:2016-12-18 14:07:33
title:FlaskWeb开发读书笔记之jinja2模板
createAt:2016-12-18 14:07:33
location:dokuwiki/python/python_flask学习笔记2
author:xbynet

官网：http://jinja.pocoo.org/docs/dev/
中文文档:http://docs.jinkan.org/docs/jinja2
当前版本2.8

安装：`pip install Jinja2`
**Flask 使用 Jinja 2 作为模板引擎。**当然，你也可以自由使用其它的模板引擎，但运行 Flask 本身仍然需要 Jinja2 依赖 ，这对启用富扩展是必要的，扩展可以依赖 Jinja2 存在。
Jinja 2 默认配置如下:

  * 所有扩展名为 .html 、 .htm 、 .xml 以及 .xhtml 的模板会**开启自动转义**
  * 模板可以利用 `{% autoescape %}` 标签选择自动转义的开关。
  * Flask 在 Jinja2 上下文中插入了几个全局函数和助手，另外还有一些目前默认的值
  * Jinja2 内部使用 Unicode 
  * **自动转义，防止XSS攻击**

 
# 标准上下文
下面的全局变量默认在 Jinja2 模板中可用:

* config：当前的配置对象 (flask.config)
* request当前的请求对象 (flask.request)。当模版不是在活动的请求上下文中渲染时这个变量不可用。
* session当前的会话对象 (flask.session)。当模版不是在活动的请求上下文中渲染时这个变量不可用。
* g**请求相关的全局变量** (flask.g)。当模版不是在活动的请求上下文中渲染时这个变量不可用。
* url_for()：flask.url_for() 函数
* get_flashed_messages()：flask.get_flashed_messages() 函数

## 问题：Jinja 上下文行为与模板导入import
这些变量被添加到了请求的上下文中，而非全局变量。
区别在于，**他们默认不会在导入(import)模板的上下文中出现**。这样做，一方面是考虑到性能，另一方面是为了让事情显式透明。**注：模板包含include不存在此问题。**
这对你来说意味着什么？如果你想要导入一个需要访问请求对象的宏，
有两种可能的方法:

* 显式地传入请求或请求对象的属性作为宏的参数。
* 使用`with context`导入宏。

**with context导入的方式如下:**
{% from '_helpers.html' import my_macro with context %}


-----


```
<title>{% block title %}{% endblock %}</title>
<ul>
{% for user in users %}
  <li><a href="{{ user.url }}">{{ user.username }}</a></li>
{% endfor %}
</ul>
```
简单使用

```
>>> from jinja2 import Template
>>> template = Template('Hello {{ name }}')
>>> template.render(name='John Doe')
u'Hello John Doe!'
```


#  基础 
Jinja2 使用一个名为 **Environment** 的中心对象。这个类的实例用于存储配 置、全局对象，并用于从文件系统或其它位置加载模板。
配置 Jinja2 为你的应用加载文档的最简单方式看起来大概是这样:
```

from jinja2 import Environment, PackageLoader
env = Environment(loader=PackageLoader('yourapplication', 'templates'))

```
获取并渲染模板：
```

template = env.get_template('mytemplate.html')
#用若干变量来渲染它，调用 render() 方法:
print template.render(the='variables', go='here')

```

##  变量过滤器:  
```

hello，{{ name|capitalize }}

```
常用变量过滤器列表:

  * safe 渲染时不进行html转义(默认会进行转义)。
  * capitalize 把值得首字母大写，其余字符转小写
  * lower
  * upper
  * title
  * trim
  * striptags 渲染之前把值中所有的HTML标签都删掉
  
[内置过滤器清单](http://jinja.pocoo.org/docs/dev/templates/#builtin-filters)

###  自定义过滤器 
自定义过滤器只是常规的 Python 函数，过滤器左边作为第一个参数，其余的参数作 为额外的参数或关键字参数传递到过滤器。

例如在过滤器 {{ 42|myfilter(23) }} 中，函数被以 myfilter(42, 23) 调 用。这里给出一个简单的过滤器示例，可以应用到 datetime 对象来格式化它们:
```

def datetimeformat(value, format='%H:%M / %d-%m-%Y'):
    return value.strftime(format)

```
你可以更新环境上的 filters 字典来把它注册到模板环境上:
```

environment.filters['datetimeformat'] = datetimeformat

```

在模板中使用如下:
```

written on: {{ article.pub_date|datetimeformat }}
publication date: {{ article.pub_date|datetimeformat('%d-%m-%Y') }}

```

除了上面提到的原生方式注册，在Flask中如果你要在 Jinja2 中注册你自己的过滤器，你还有2种方法。你可以把它们手动添加到应用的  `jinja_env` 或者使用 `template_filter() `装饰器。 
下面两个例子作用相同，都是反转一个对象:

```
#第一种
@app.template_filter('reverse')
def reverse_filter(s):
    return s[::-1]
#第二种
def reverse_filter(s):
    return s[::-1]
app.jinja_env.filters['reverse'] = reverse_filter

```

##  控制自动转义 
有三种可行的解决方案:

  * 在传递到模板之前，用 Markup 对象封装 HTML字符串。一般推荐这个方法。
  * 在模板中，使用 |`safe `过滤器显式地标记一个字符串为安全的 HTML {{ myvariable|safe }}
  * 临时地完全禁用自动转义系统。`{% autoescape false %}`
在模板中禁用自动转义系统，可以使用 {%autoescape %} 块:

```
{% autoescape false %}
    <p>autoescaping is disabled here
    <p>{{ will_not_be_escaped }}
{% endautoescape %}
```

##  上下文处理器 
Flask 上下文处理器自动向模板的上下文中插入新变量。上下文处理器在模板渲染之前运行，并且可以在模板上下文中插入新值。
**上下文处理器是一个返回`字典`的函数，这个字典的键值最终将传入应用中所有模板的上下文:**
```
@app.context_processor
def inject_user():
    return dict(user=g.user)
```

上面的上下文处理器使得模板可以使用一个名为 user ，值为 g.user 的变量。 不过这个例子不是很有意思，因为 g 在模板中本来就是可用的，但它解释了上下文处理器是如何工作的。
**变量不仅限于值，上下文处理器也可以使某个函数在模板中可用**（由于 Python 允许传递函数）:
```
@app.context_processor
def utility_processor():
    def format_price(amount, currency=u'€'):
        return u'{0:.2f}{1}.format(amount, currency)
    return dict(format_price=format_price)
```

**上面的上下文处理器使得 format_price 函数在所有模板中可用:**

```
{{ format_price(0.33) }}
```

你也可以构建 format_price 为一个模板过滤器（见 注册过滤器 ）， 但这展示了上下文处理器`传递函数`的工作过程。

#  模板语法 
这里有两种分隔符: {% ... %} 和{{  }} 。前者用于执行诸如 for 循环 或赋值的语句，后者把表达式的结果打印到模板上。
要把模板中的部分注释掉，默认使用** {# ... #}** 注释语法。
true，false,none，这一点与python内置有点区别


变量: 
Jinja2能够识别所有类型的变量，例如列表，字典和对象。如下:

```
{{ mydict['key'] }}
{{ mylist[3] }}
{{ mylist[myintvar] }}
{{ myobj.somemethod() }}
```
算术，比较，逻辑语法均和python相同

##  流程控制： 

1、条件控制:
```

{% if user %}
hello {{ user }}!
{% else %}
hello, world
{% endif %}

```

2、for循环:
```

<ul>
{% for comment in comments %}
	<li>{{ comment }}</li>
{% endfor %}
</ul>

```
**在一个 for 循环块中你可以访问这些特殊的变量**:

  * loop.index	当前循环迭代的次数（从 1 开始）
  * loop.index0	当前循环迭代的次数（从 0 开始）
  * loop.revindex	到循环结束需要迭代的次数（从 1 开始）
  * loop.revindex0	到循环结束需要迭代的次数（从 0 开始）
  * loop.first	如果是第一次迭代，为 True 。
  * loop.last	如果是最后一次迭代，为 True 。
  * loop.length	序列中的项目数。
  * loop.cycle	在一串序列间期取值的辅助函数。见下面的解释。

##  宏支持 
宏类似于函数。如:
```

{% macro render_comment(comment) -%}
	<li>{{ comment }}</li>
{%- endmacro %}

<ul>
{% for comment in comments %}
	{{ render_comment(comment) }}
{% endfor %}
</ul>

```
为了重复使用宏，可以将宏定义保存在单独的文件中，然后导入:
```

{% import 'macros.html' as macros %}
<ul>
{% for comment in comments %}
	{{ macros.render_comment(comment) }}
{% endfor %}
</ul>

```
此外你也可以从模板中导入名称到当前的命名空间:
```

{% from 'forms.html' import input as input_field, textarea %}

```

##  模板继承 
```

<html>
<head>
  {% block head %}
  <title>{% block title %}{% endblock %}</title>
  {% endblock %}
</head>
<body>
  {% block body %}
  {% endblock %}
</body>
</html>

```
上述定义了三个块(block)：head、title、body,衍生模板可以修改这三个块
```

{% extends 'base.html' %}
{% block title %}Index{% endblock %}
{% block head %}
	{{ super() }}
	<style></style>
{% endblock %}
{% block body %}
<h1>hello</h1>
{% endblock %}

```
在extends指令之后，基模板中的3个块被重新定义，模板引擎会将其插入适当位置。**super()可以用来获取父模板原来的内容。**

如果你想要多次打印一个块，无论如何你可以使用**特殊的 self 变量**并调用与块同名 的函数:
```

<title>{% block title %}{% endblock %}</title>
<h1>{{ self.title() }}</h1>
{% block body %}{% endblock %}

```
##  嵌套块和作用域 
嵌套块可以胜任更复杂的布局。**而默认的块不允许访问块外作用域中的变量**:
```

{% for item in seq %}
    <li>{% block loop_item %}{{ item }}{% endblock %}</li>
{% endfor %}

```
**这个例子会输出空的 < li> 项，因为 item 在块中是不可用的。**其原因是，如果 块被子模板替换，变量在其块中可能是未定义的或未被传递到上下文。
` 从 Jinja 2.2 开始，你可以显式地指定在块中可用的变量，只需在块声明中添加 scoped 修饰，就把块设定到作用域中: `
```

{% for item in seq %}
    <li>{% block loop_item scoped %}{{ item }}{% endblock %}</li>
{% endfor %}

```
**当覆盖一个块时，不需要提供 scoped 修饰。**
##  包含 
include 语句用于包含一个模板，并在当前命名空间中返回那个文件的内容渲 染结果:
```

{% include 'header.html' %}
    Body
{% include 'footer.html' %}

```
##  导入与包含的上下文行为 
` 默认下，每个**包含**的模板会被传递到当前上下文，而**导入**的模板不会。 `这样做的原因 import会被缓存，因为导入量经常只作容纳宏的模块。并且默认下导入的 模板不能访问当前模板中的非全局变量。这当然也可以显式地更改。**通过在 import/include 声明中直接添加 with context 或 without context ，当前的上下文可以传递到模板**，而且不会 自动禁用缓存。
```

{% from 'forms.html' import input with context %}
{% include 'header.html' %}

```
## 变量定义与赋值 
在代码块中，你也可以为变量赋值。在顶层的（块、宏、循环之外）赋值是可导出的，即 可以从别的模板中导入。
赋值使用 set 标签，并且可以为多个变量赋值:
{% set navigation = [('index.html', 'Index'), ('about.html', 'About')] %}
{% set key, value = call_something() %}
##  不进行Jinjia2语法解释 
```

{% raw %}
{%blahblahblah%}
{% endraw %}

```
##  自定义错误页面 
```

@app.errorhandler(404)
def page_not_found(e):
	return render_template('404.html'),404


```

##  链接 
url_for('index')输出'/'
url_for('index',_external=True)输出http://localhost:5000/

##  静态文件 
特殊路由/static/< filename>
url_for('static',filename='css/style.css') 输出http://localhost:5000/static/css/style.css
默认设置下，Flask在程序目录中名为static的子目录中寻找静态文件。
  
##  内置工具函数 
http://docs.jinkan.org/docs/jinja2/templates.html#builtin-globals

##  国际化 
_(' ')

```
{{ _('Hello World!') }}
```

###  With 语句 
如果应用启用了 With 语句 ，将允许在模板中使用 with 关键 字。
这使得创建一个新的内作用域。这个作用域中的变量在外部是不可见的。
With 用法简介:

```
{% with %}
    {% set foo = 42 %}
    {{ foo }}           foo is 42 here
{% endwith %}
foo is not visible here any longer
```

#  使用Flask-Moment本地化日期和时间 
详情请[参考](http://wiki.xby1993.net/pages/dokuwiki/python/flask-moment)

# 使用Flask-Bootstrap
详情[参考](http://wiki.xby1993.net/pages/dokuwiki/python/flask-bootstrap)
