title: python学习笔记034 

#  Python学习笔记之Flask自定义错误页面 
**Flask 自带了很顺手的 abort() 函数用于以一个 HTTP 失败代码中断一个请求，他也会提供一个非常简单的错误页面，用于提供一些基础的描述。** 这个页面太朴素了以至于缺乏一点灵气。
**错误处理器和要捕捉的错误代码使用 errorhandler() 装饰器注册。 请记住 Flask 不会 替您设置错误代码，所以请确保在返回 response 对象时，提供了对应的 HTTP 状态代码。**
如下实现了一个 “404 Page Not Found” 错误处理的例子:
```

from flask import render_template
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404
一个示例模板可能会如下所示:
{% extends "layout.html" %}
{% block title %}Page Not Found{% endblock %}
{% block body %}
  <h1>Page Not Found</h1>
  <p>What you were looking for is just not there.
  <p><a href="![](/data/dokuwiki url_for('index') )">go somewhere nice</a>
{% endblock %}

```
