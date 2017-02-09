title: python学习笔记021 

#  Python学习笔记之Flask框架入门 
官网:http://flask.pocoo.org/
github:https://github.com/mitsuhiko/flask
中文翻译:http://docs.jinkan.org/docs/flask/
**Flask 依赖两个外部库： Jinja2 模板引擎和 Werkzeug WSGI 工具集。**
Jinja2文档:http://jinja.pocoo.org/docs/dev/
Werkzeug WSGI文档:http://werkzeug.pocoo.org/docs/0.11/
Flask扩展:http://flask.pocoo.org/extensions/
微框架中的“微”意味着 Flask 旨在保持核心简单而易于扩展。
Flask 不会替你做出太多决策——比如使用何种数据库。而那些 Flask 所选择的——比如使用何种模板引擎——则很容易替换。除此之外的一切都由可由你掌握。
默认情况下，Flask 不包含数据库抽象层、表单验证，或是其它任何已有多种库可以胜任的功能。然而，Flask 支持用扩展来给应用添加这些功能，如同是 Flask 本身实现的一样。
众多的扩展提供了数据库集成、表单验证、上传处理、各种各样的开放认证技术等功能。Flask 也许是“微小”的，但它已准备好在需求繁杂的生产环境中投入使用。

##  概述 
配置约定: 
按照惯例，**模板和静态文件分别存储在应用 Python 源代码树下的子目录 templates 和 static 里。**


安装:Flask 依赖两个外部库：Werkzeug 和 Jinja2 。 
Werkzeug 是一个 WSGI（在 Web 应用和多种服务器之间的标准 Python 接口) 工具集。Jinja2 负责渲染模板。
```

pip install Flask
pip install Jinja2
pip install Werkzeug

```
##  快速入门 
一个最小的 Flask 应用看起来会是这样:
```

from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()

```
把它保存为 hello.py （或是类似的），然后用 Python 解释器来运行。 确保你的应用文件名不是 flask.py ，因为这将与 Flask 本身冲突。
$ python hello.py
 * Running on http://127.0.0.1:5000/
现在访问 http://127.0.0.1:5000/ ，你会看见 Hello World 问候。
那么，这段代码做了什么？
  * 首先，我们导入了 Flask 类。这个类的实例将会是我们的 WSGI 应用程序。
  * 接下来，我们创建一个该类的实例，**第一个参数是应用模块或者包的名称。** 如果你使用单一的模块（如本例），你应该使用 __name__ ，**因为模块的名称将会因其作为单独应用启动还是作为模块导入而有不同（ 也即是 '__main__' 或实际的导入名）。这是必须的，这样 Flask 才知道到哪去找模板、静态文件等等。**详情见 Flask 的文档。
  * 然后，我们使用 ` route() 装饰器 `告诉 Flask 什么样的URL 能触发我们的函数。这个函数的名字也在生成 URL 时被特定的函数采用**，这个函数返回我们想要显示在用户浏览器中的信息。**
最后我们用 run() 函数来让应用运行在本地服务器上。 其中 if __name__ == '__main__': 确保服务器只会在该脚本被 Python 解释器直接执行的时候才会运行，而不是作为模块导入的时候。

###  外部可访问的服务器 
1、禁用debug 
2、app.run(host='0.0.0.0')

###  调试模式 
**如果你启用了调试支持，服务器会在代码修改后自动重新载入**，并在发生错误时提供一个相当有用的调试器。
有两种途径来启用调试模式。一种是直接在应用对象上设置:
app.debug = True
另一种是作为 run 方法的一个参数传入:
app.run(debug=True)

###  路由 
如上所见， route() 装饰器把一个函数绑定到对应的 URL 上。
这里是一些基本的例子:
```

@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello World'

```
但是，不仅如此！你可以构造含有动态部分的 URL，也可以在一个函数上附着多个规则。
###  变量规则 
要给 URL 添加变量部分，你可以把这些特殊的字段标记为 <variable_name> ， **这个部分将会作为命名参数传递到你的函数**。规则可以用 <converter:variable_name> 指定一个**可选的转换器**。这里有一些不错的例子:
```

@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return 'User %s' % username

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return 'Post %d' % post_id

```
转换器有下面几种：
  * int	接受整数
  * float	同 int ，但是接受浮点数
  * path	和默认的相似，但也接受斜线
###  唯一 URL / 重定向行为 
**Flask 的 URL 规则基于 Werkzeug 的路由模块。**
保证优雅且唯一的 URL。以这两个规则为例:
```

@app.route('/about/')
def projects():
    return 'The about page'

@app.route('/about')
def about():
    return 'The about page'

```
虽然它们看起来着实相似，但它们结尾斜线的使用在 URL 定义 中不同。
**推荐使用不带斜线的第二种方式**：因为第一种方式会导致搜索引擎索引同一个页面两次。
###  构造 URL 
如果 Flask 能匹配 URL，那么 Flask 可以生成它们吗？当然可以。
**你可以用 url_for() 来给指定的函数构造 URL。它接受函数名作为第一个参数**，也接受对应 URL 规则的变量部分的命名参数。**未知变量部分会添加到 URL 末尾作为查询参数。**这里有一些例子:
```

>>> from flask import Flask, url_for
>>> app = Flask(__name__)
>>> @app.route('/')
... def index(): pass
...
>>> @app.route('/login')
... def login(): pass
...
>>> @app.route('/user/<username>')
... def profile(username): pass
...
>>> with app.test_request_context():
...  print url_for('index')
...  print url_for('login')
...  print url_for('login', next='/')
...  print url_for('profile', username='John Doe')
...
/
/login
/login?next=/
/user/John%20Doe

```
（这里也用到了`  test_request_context() 方法 `，即使我们正在通过 Python 的 shell 进行交互，**它依然会告诉 Flask 要表现为正在处理一个请求**。请看下面的解释。 环境局部变量 ）
为什么你要构建 URL 而非在模板中硬编码？这里有三个绝妙的理由：
  * 它允许你一次性修改 URL， 而不是到处边找边改。
  * URL 构建会转义特殊字符和 Unicode 数据。
  * 如果你的应用不位于 URL 的根路径（比如，在 /myapplication 下，而不是 / ）， url_for() 会妥善处理这个问题。

###  HTTP 方法 
HTTP （与 Web 应用会话的协议）有许多不同的访问 URL 方法。
**默认情况下，路由只回应 GET 请求，但是通过 route() 装饰器传递 methods 参数可以改变这个行为。**这里有一些例子:
```

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()
    else:
        show_the_login_form()

```

###  静态文件 
动态 web 应用也会需要静态文件，通常是 CSS 和 JavaScript 文件。 
**创建一个名为 static 的文件夹，在应用中使用 /static 即可访问**。
给静态文件生成 URL ，使用特殊的 'static' 端点名:
```

url_for('static', filename='style.css')

```
这个文件应该存储在文件系统上的 static/style.css 。

###  模板渲染 
Flask 配备了 Jinja2 模板引擎。
你可以使用 ` render_template() 方法来渲染模板 `。你需要做的一切就是**将模板名和你想作为关键字的参数传入模板的变量**。这里有一个展示如何渲染模板的简例:
```

from flask import render_template
@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)

```
Flask 会在 templates 文件夹里寻找模板。所以，如果你的应用是个模块，这个文件夹应该与模块同级；如果它是一个包，那么这个文件夹作为包的子目录:
```

情况 1: 模块:
/application.py
/templates
    /hello.html
情况 2: 包:
/application
    /__init__.py
    /templates
        /hello.html

```
关于模板，你可以发挥 Jinja2 模板的全部实例。更多信息请见 Jinja2 模板文档 。
这里有一个模板实例：
```

<!doctype html>
<title>Hello from Flask</title>
{% if name %}
  <h1>Hello ![](/data/dokuwiki name )!</h1>
{% else %}
  <h1>Hello World!</h1>
{% endif %}

```
**在模板里，你也可以访问 request 、 session 和 g对象(后面会介绍，相当于应用上下文)， 以及 get_flashed_messages() 函数。**
**自动转义功能默认是开启的，所以如果 name 包含 HTML ，它将会被自动转义。**如果你能信任一个变量，并且你知道它是安全的（例如一个模块把 Wiki 标记转换为 HTML），**你可以用 Markup 类或 |safe 过滤器在模板中把它标记为安全的。**

Markup 类使用示例：
```

>>> from flask import Markup
>>> Markup('<strong>Hello %s!</strong>') % '<blink>hacker</blink>'
Markup(u'<strong>Hello &lt;blink&gt;hacker&lt;/blink&gt;!</strong>')
>>> Markup.escape('<blink>hacker</blink>')
Markup(u'&lt;blink&gt;hacker&lt;/blink&gt;')
>>> Markup('<em>Marked up</em> &raquo; HTML').striptags()
u'Marked up \xbb HTML'

```
在 0.5 版更改: 自动转义不再在所有模板中启用。下列扩展名的模板会触发自动转义： .html 、 .htm 、.xml 、 .xhtml 。从字符串加载的模板会禁用自动转义。

###  访问请求数据 
**在 Flask 中由全局的 request 对象来提供这些信息。**
**当前请求的 HTTP 方法可通过 method 属性来访问。通过:attr:~flask.request.form 属性来访问表单数据（ POST 或 PUT 请求提交的数据）**。这里有一个用到上面提到的那两个属性的完整实例:
```

from flask import request
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # the code below is executed if the request method
    # was GET or the credentials were invalid
    return render_template('login.html', error=error)

```
**当访问 form 属性中的不存在的键会发生什么？会抛出一个特殊的 KeyError 异常。如果你不捕获，它会显示一个 HTTP 400 Bad Request 错误页面。**所以，多数情况下你并不需要干预这个行为。

GET/URL参数获取
```

#你可以通过 args 属性来访问 URL 中提交的参数 （ ?key=value ）:
searchword = request.args.get('q', '')

```
**我们推荐用 get 来访问 URL 参数或捕获 KeyError ，**因为用户可能会修改 URL，向他们展现一个 400 bad request 页面会影响用户体验。

###  环境局部变量 
**Flask 中的某些对象是全局对象，但却不是通常的那种。这些对象实际上是特定环境的局部对象的代理。**
想象一下处理线程的环境。一个请求传入，Web 服务器决定生成一个新线程。 当 Flask 开始它内部的请求处理时，它认定当前线程是活动的环境，并绑定当前的应用和 WSGI 环境到那个环境上（线程）。它的实现很巧妙，能保证一个应用调用另一个应用时不会出现问题。
所以，这对你来说意味着什么？除非你要做类似单元测试的东西，否则你基本上可以完全无视它。
你会发现依赖于一段请求对象的代码，因没有请求对象无法正常运行。解决方案是，自行创建一个请求对象并且把它绑定到环境中。
**单元测试的最简单的解决方案是：用 test_request_context() 环境管理器。结合 with 声明，绑定一个测试请求，这样你才能与之交互。下面是一个例子:**

```

from flask import request
with app.test_request_context('/hello', method='POST'):
    # now you can do something with the request until the
    # end of the with block, such as basic assertions:
    assert request.path == '/hello'
    assert request.method == 'POST'

```
另一种可能是：传递整个 WSGI 环境给 request_context() 方法:
```

from flask import request
with app.request_context(environ):
    assert request.method == 'POST'

```

###  文件上传 
用 Flask 处理文件上传很简单。只要确保你没忘记在 HTML 表单中设置 **enctype="multipart/form-data" 属性**，不然你的浏览器根本不会发送文件。
**已上传的文件存储在内存或是文件系统中一个临时的位置。你可以通过请求对象的 files 属性访问它们。每个上传的文件都会存储在这个字典里。它表现近乎为一个标准的 Python file 对象，但它还有一个 save() 方法，这个方法允许你把文件保存到服务器的文件系统上。**这里是一个用它保存文件的例子:
```

from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
    ...

```
如果你想知道上传前文件在客户端的文件名是什么，你可以访问 filename 属性。但请记住， 永远不要信任这个值，这个值是可以伪造的。
**如果你要把文件按客户端提供的文件名存储在服务器上，那么请把它传递给 Werkzeug 提供的 secure_filename() 函数:**
```

from flask import request
from werkzeug import secure_filename

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/' + secure_filename(f.filename))
    ...

```
一些更好的例子，见 上传文件 模式

###  Cookies 
**你可以通过 cookies 属性来访问 Cookies，用响应对象的 set_cookie 方法来设置 Cookies**。请求对象的 cookies 属性是一个内容为客户端提交的所有 Cookies 的字典。如果你想使用会话，请不要直接使用 Cookies，请参考 会话 一节。在 Flask 中，已经注意处理了一些 Cookies 安全细节。
读取 cookies:
```

from flask import request
@app.route('/')
def index():
    username = request.cookies.get('username')
    # use cookies.get(key) instead of cookies[key] to not get a
    # KeyError if the cookie is missing.

```
存储 cookies:
```

from flask import make_response
@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp

```
可注意到的是，Cookies 是设置在响应对象上的。由于通常视图函数只是返回字符串，之后 Flask 将字符串转换为响应对象。如果你要显式地转换，你可以使用 ` make_response() 函数 `然后再进行修改。
有时候你想设置 Cookie，但响应对象不能醋在。这可以利用 延迟请求回调 模式实现。为此，也可以阅读 关于响应 。

###  重定向和错误 
你可以用 ` redirect() 函数 `把用户重定向到其它地方。放弃请求并返回错误代码，用 abort() 函数。这里是一个它们如何使用的例子:
```

from flask import abort, redirect, url_for
@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()

```
这是一个相当无意义的例子因为用户会从主页重定向到一个不能访问的页面 （401 意味着禁止访问），但是它展示了重定向是如何工作的。
默认情况下，错误代码会显示一个黑白的错误页面。如果你要定制错误页面， 可以使用`  errorhandler() 装饰器 `:
```

from flask import render_template
@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404

```
**注意 render_template() 调用之后的 404 。这告诉 Flask，该页的错误代码是 404 ，即没有找到。默认为 200，也就是一切正常。**

###  关于响应 
几种方式：
  * 1、视图函数返回的内容包括：相应字符串和状态码。
  * 2、返回Response对象。使用make_response(),返回Response对象。
  * 3、重定向redirect。这种响应没有页面文档,只告诉浏览器一个新地址用以加载新页面。
  * 4、特殊相应。abort 函数生成,用于处理错误。

**视图函数的返回值会被自动转换为一个响应对象。如果返回值是一个字符串， 它被转换为该字符串为主体的、状态码为 200 OK``的 、 MIME 类型是 ``text/html 的响应对象。**
Flask 把返回值转换为响应对象的逻辑是这样：
  * 如果返回的是一个合法的响应对象，它会从视图直接返回。
  * 如果返回的是一个字符串，响应对象会用字符串数据和默认参数创建。
  * 如果返回的是一个元组，且元组中的元素可以提供额外的信息。**这样的元组必须是 (response, status, headers) 的形式，且至少包含一个元素**。 status 值会覆盖状态代码， **headers 可以是一个列表或字典，作为额外的消息标头值。**
如果上述条件均不满足， Flask 会假设返回值是一个合法的 WSGI 应用程序，并转换为一个请求对象。
**如果你想在视图里操纵上述步骤结果的响应对象，可以使用`  make_response() 函数 `。**

譬如你有这样一个视图:
```

@app.errorhandler(404)
def not_found(error):
    return render_template('error.html'), 404

```
你只需要把返回值表达式传递给 make_response() ，获取结果对象并修改，然后再返回它:
```

@app.errorhandler(404)
def not_found(error):
    resp = make_response(render_template('error.html'), 404)
    resp.headers['X-Something'] = 'A value'
    return resp

```
###  会话 
除` 请求request对象 `之外，还有一个 ` session 对象 `。它允许你在不同请求间存储特定用户的信息。**它是在 Cookies 的基础上实现的，并且对 Cookies 进行密钥签名。这意味着用户可以查看你 Cookie 的内容，但却不能修改它，除非用户知道签名的密钥。**
**要使用会话，你需要设置一个密钥。**这里介绍会话如何工作:
```

from flask import Flask, session, redirect, url_for, escape, request
app = Flask(__name__)
@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form action="" method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))

# set the secret key.  keep this really secret:
app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'

```
这里提到的`  escape() 可以在你模板引擎外做转义 `（如同本例）。
####  如何生成强壮的密钥 
随机的问题在于很难判断什么是真随机。一个密钥应该足够随机。你的操作系统可以基于一个密钥随机生成器来生成漂亮的随机值，这个值可以用来做密钥:
```

>>> import os
>>> os.urandom(24)
'\xfd{H\xe5<\x95\xf9\xe3\x96.5\xd1\x01O<!\xd5\xa2\xa0\x9fR"\xa1\xa8'

```
把这个值复制粘贴进你的代码中，你就有了密钥。
使用基于 cookie 的会话需注意: ` Flask 会将你放进会话对象的值序列化至 Cookies。如果你发现某些值在请求之间并没有持久存在，然而确实已经启用了 Cookies，但也没有得到明确的错误信息。这时，请检查你的页面响应中的 Cookies 的大小，并与 Web 浏览器所支持的大小对比。 `
###  消息闪现 
Flask 提供了消息闪现系统，可以简单地给用户反馈。 消息闪现系统通常会在请求结束时记录信息，并在下一个（且仅在下一个）请求中访问记录的信息。展现这些消息通常结合要模板布局。
使用 **flash**() 方法可以闪现一条消息。要操作消息本身，请使用 **get_flashed_messages**() 函数，并且在模板中也可以使用。
###  日志记录 
**Flask已经预置了日志系统。**
这里有一些调用日志记录的例子:
```

app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')

```
**附带的 logger 是一个标准日志类 Logger** ，所以更多信息请查阅 logging 的文档 。

##  Flask 扩展 
Flask 扩展用多种不同的方式扩充 Flask 的功能。比如加入数据库支持和其它的常见任务。
寻找扩展:http://flask.pocoo.org/extensions/
Flask Extension Registry 中列出了 Flask 扩展，并且可以通过 easy_install 或 pip 下载。如果你把一个 Flask 扩展添加到 requirements.rst 或 setup.py 文件的依赖关系中，它们通常可以用一个简单的命令或是在你应用安装时被安装。
使用扩展
扩展通常附带有文档，来展示如何使用它。扩展的行为没有一个可以预测的一般性规则，除了它们是从同一个位置导入的**。如果你有一个名为 Flask-Foo 或是 Foo-Flask 的扩展，你可以从 flask.ext.foo 导入它**:
```

from flask.ext import foo

```