title: flask-bootstrap 

#  Flask-bootstrap插件 
官网：http://pythonhosted.org/Flask-Bootstrap/
当前版本3.3.7
pip install flask-bootstrap

```

from flask import Flask
from flask_bootstrap import Bootstrap

def create_app():
  app = Flask(__name__)
  Bootstrap(app)

  return app

# do something with app...

```
示例程序：https://github.com/mbr/flask-bootstrap/tree/master/sample_app

##  快速入门 
模板：
```

{% extends "bootstrap/base.html" %}
{% block title %}This is an example page{% endblock %}

{% block navbar %}
<div class="navbar navbar-fixed-top">
  <!-- ... -->
</div>
{% endblock %}

{% block content %}
  <h1>Hello, Bootstrap</h1>
{% endblock %}

```
可用的blocks
&lt;nowiki&gt;
  * doc	 	Outermost block.
  * html	doc	Contains the complete content of the &lt;html&gt; tag.
  * html_attribs	doc	Attributes for the HTML tag.
  * head	doc	Contains the complete content of the &lt;head&gt; tag.
  * body	doc	Contains the complete content of the &lt;body&gt; tag.
  * body_attribs	body	Attributes for the Body Tag.
  * title	head	Contains the complete content of the &lt;title&gt; tag.
  * styles	head	Contains all CSS style &lt;link&gt; tags inside head.
  * metas	head	Contains all &lt;meta&gt; tags inside head.
  * navbar	body	An empty block directly above content.
  * content	body	Convenience block inside the body. Put stuff here.
  * scripts	body	Contains all &lt;script&gt; tags at the end of the body.
&lt;/nowiki&gt;
Examples
引入css文件
```

{% block styles %}
![](/data/dokuwikisuper())  <!--这里用到了jinjia2的super()函数 -->
<link rel="stylesheet"
      href="![](/data/dokuwikiurl_for('.static', filename='mystyle.css'))">
{% endblock %}

```
引入js
```

Custom Javascript loaded before Bootstrap’s javascript code:
{% block scripts %}
<script src="![](/data/dokuwikiurl_for('.static', filename='myscripts.js'))"></script>
![](/data/dokuwikisuper())
{% endblock %}

```
添加html标签的属性
```

Adding a lang="en" attribute to the html-tag:
{% block html_attribs %} lang="en"{% endblock %}

```
等等

##  静态资源： 
The url-endpoint **bootstrap.static** is available for refering to Bootstrap resources
也可以通过模板过滤器 bootstrap_find_resource来从CDN上来找到资源

##  配置 
  * BOOTSTRAP_USE_MINIFIED	True	Whether or not to use the minified versions of the css/js files.
  * BOOTSTRAP_SERVE_LOCAL	False	If True, Bootstrap resources will be served from the local app instance every time. See CDN support for details.
  * BOOTSTRAP_LOCAL_SUBDOMAIN	None	Passes a subdomain parameter to the generated Blueprint. Useful when serving assets locally from a different subdomain.
  * BOOTSTRAP_CDN_FORCE_SSL	True	If a CDN resource url starts with / /, prepend 'https:' to it.
  * BOOTSTRAP_QUERYSTRING_REVVING	True	If True, will append a querystring with the current version to all static resources served locally. This ensures that upon upgrading Flask-Bootstrap, these resources are refreshed.

##  宏 
例如
```

{% extends "bootstrap/base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

```
###  浏览器兼容问题 
下载 html5shiv and Respond.js放到应用的static目录下，然后
```

{% import "bootstrap/fixes.html" as fixes %}
{% block head %}
  ![](/data/dokuwikisuper())
  ![](/data/dokuwikifixes.ie8())
{% endblock %}

```

###  Utilities 
bootstrap/utils.html包含了一下工具宏
**1、flashed_messages**(messages=None, container=True, transform=..., default_category=None, dismissible=False)

参数	
  * messages – A list of messages. **If not given, will use get_flashed_messages() to retrieve them.**
  * container – If true, will output a complete <div class="container"> element, otherwise just the messages each wrapped in a <div>.
  * transform – A dictionary of mappings for categories. Will be looked up case-insensitively. Default maps all Python loglevel names to bootstrap CSS classes.** categories的类型有：success, info, warning, danger**
  * default_category – If a category does not has a mapping in transform, it is passed through unchanged. If default_category is set, it is replaced with this instead.
  * dismissible – If true, will output a button to close an alert. For fully functioning, dismissible alerts, you must use the alerts JavaScript plugin.

如果要使category生效：使用flash时需要在最后这样加上category标志
flash('Operation failed', ` 'danger') `

**2、icon**(type, extra_classes, * *kwargs)
Renders a Glyphicon in a <span> element.

Parameters:	
  * messages – The short name for the icon, e.g. remove.
  * extra_classes – A list of additional classes to add to the class attribute.
  * kwargs – Additional html attributes.

**3、form_button**(url, content, method='post', class='btn-link', * *kwargs)
Renders a button/link wrapped in a form.

Parameters:	
  * url – The endpoint to submit to.
  * content – The inner contents of the button element.
  * method – method-attribute of the surrounding form.
  * class – class-attribute of the button element.
  * kwargs – Extra html attributes for the button element.

```

{{form_button(url_for('remove_entry', id=entry_id),
              icon('remove') + ' Remove entry')}}

```

##  WTForms支持 
bootstrap/wtf.html内包含的宏。基于 Flask-WTF 
```

<form class="form form-horizontal" method="post" role="form">
  ![](/data/dokuwiki form.hidden_tag() )
  ![](/data/dokuwiki wtf.form_errors(form, hiddens="only") )

  ![](/data/dokuwiki wtf.form_field(form.field1) )
  ![](/data/dokuwiki wtf.form_field(form.field2) )
</form>

```
或者快速创建：
```

![](/data/dokuwiki wtf.quick_form(form) )

```
相关宏：
quick_form(form, action=".", method="post", extra_classes=None, role="form", form_type="basic", horizontal_columns=('lg', 2, 10), enctype=None, button_map={}, id="")
form_errors(form, hiddens=True)
form_field(field, form_type="basic", horizontal_columns=('lg', 2, 10), button_map={})

##  Flask-SQLAlchemy分页显示支持 
```

{% from "bootstrap/pagination.html" import render_pagination %}

{# ... #}

![](/data/dokuwikirender_pagination(query_results))

```
宏：**render_pagination**(pagination, endpoint=None, prev='«', next='»', ellipses='…', size=None, args={}, * *kwargs)


##  CSN支持 
yourapp.extensions['bootstrap']['cdns']具体略。
如果不需要CSN，而是想从本地加载bootsrap相关文件，static/bootstrap
修改CSDN
```

from flask_bootstrap import WebCDN
app.extensions['bootstrap']['cdns']['jquery'] = WebCDN(
    '//cdnjs.cloudflare.com/ajax/libs/jquery/2.1.1/'
)

```
不用CDN变成本地加载jquery
```

from flask_bootstrap import StaticCDN
app.extensions['bootstrap']['cdns']['jquery'] = StaticCDN()

```
Note that in this case you need to download a suitable jquery.js and/or jquery.min.js and put it into** your apps static-folder.**
