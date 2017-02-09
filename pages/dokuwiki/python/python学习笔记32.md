title: python学习笔记32 

#  Python学习笔记之View模版简介 
![](/data/dokuwiki/python/pasted/20160314-105301.png)
这就是传说中的MVC：Model-View-Controller，中文名“模型-视图-控制器”。
  * Python处理URL的函数就是C：Controller，Controller负责业务逻辑，比如检查用户名是否存在，取出用户信息等等；
  * 包含变量{ { name } }的模板就是V：View，View负责显示逻辑，通过简单地替换一些变量，View最终输出的就是用户看到的HTML。
  * MVC中的Model在哪？Model是用来传给View的，这样View在替换变量的时候，就可以从Model中取出相应的数据。
上面的例子中，Model就是一个dict：
{ 'name': 'Michael' }
只是因为Python支持关键字参数，很多Web框架允许传入关键字参数，然后，在框架内部组装出一个dict作为Model。
现在，我们把上次直接输出字符串作为HTML的例子用高端大气上档次的MVC模式改写一下：
```

from flask import Flask, request, render_template
app = Flask(__name__)
@app.route('/', methods=['GET', 'POST'])
def home():
    return render_template('home.html')

@app.route('/signin', methods=['GET'])
def signin_form():
    return render_template('form.html')

@app.route('/signin', methods=['POST'])
def signin():
    username = request.form['username']
    password = request.form['password']
    if username=='admin' and password=='password':
        return render_template('signin-ok.html', username=username)
    return render_template('form.html', message='Bad username or password', username=username)

if __name__ == '__main__':
    app.run()

```
**Flask通过render_template()函数来实现模板的渲染。**和Web框架类似，Python的模板也有很多种。**Flask默认支持的模板是jinja2**，所以我们先直接安装jinja2：
```

$ pip install jinja2

```
然后，开始编写jinja2模板：
home.html
用来显示首页的模板：
```

<html>
<head>
  <title>Home</title>
</head>
<body>
  <h1 style="font-style:italic">Home</h1>
</body>
</html>

```
form.html
用来显示登录表单的模板：
```

<html>
<head>
  <title>Please Sign In</title>
</head>
<body>
  {% if message %}
  <p style="color:red">![](/data/dokuwiki message )</p>
  {% endif %}
  <form action="/signin" method="post">
    <legend>Please sign in:</legend>
    <p><input name="username" placeholder="Username" value="![](/data/dokuwiki username )"></p>
    <p><input name="password" placeholder="Password" type="password"></p>
    <p><button type="submit">Sign In</button></p>
  </form>
</body>
</html>

```
signin-ok.html
登录成功的模板：
```

<html>
<head>
  <title>Welcome, ![](/data/dokuwiki username )</title>
</head>
<body>
  <p>Welcome, ![](/data/dokuwiki username )!</p>
</body>
</html>

```
登录失败的模板呢？我们在form.html中加了一点条件判断，把form.html重用为登录失败的模板。
最后，一定要把模板放到正确的templates目录下，templates和app.py在同级目录下：
![](/data/dokuwiki/python/pasted/20160314-105512.png)

除了Jinja2，常见的模板还有：
  * Mako：用<% ... %>和${xxx}的一个模板；
  * Cheetah：也是用<% ... %>和${xxx}的一个模板；
  * Django：Django是一站式框架，内置一个用{% ... %}和![](/data/dokuwiki xxx )的模板。