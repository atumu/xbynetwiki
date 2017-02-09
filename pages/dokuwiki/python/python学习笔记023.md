title: python学习笔记023 

#  Python学习笔记之Flask开发应用教程 
想要用 Python 和 Flask 开发一个应用？在此，你将有机会通过实例来学习。 在本教程中，我们会创建一个简单的微博客应用。它只支持单用户和纯文本条目，并且没有推送或评论功能，但是它仍然有你需要开始的一切。我们将使用 Flask ，采用 Python 自带的 SQLite 数据库，所以你不需要其它的东西。
如果你想预先拿到完整源码或是用于对照，请查看 示例源码 https://github.com/mitsuhiko/flask/tree/master/examples/flaskr/。
  * 步骤 0: 创建文件夹
  * 步骤 1: 数据库模式
  * 步骤 2: 应用设置代码
  * 步骤 3: 数据库连接
  * 步骤 4: 创建数据库
  * 步骤 5: 视图函数
  * 步骤 6: 模板
  * 步骤 7: 添加样式
  * 福利: 应用测试
**步骤 0: 创建文件夹**
在我们真正开始之前，让我们创建这个应用所需的文件夹:
```

/flaskr
    /static
    /templates

```
**flaskr 文件夹不是一个 Python 包，只是个我们放置文件的地方。**在接下来的步骤中，**我们会直接把数据库模式和主模块放在这个目录中。** 用户可以通过 HTTP 访问 static 文件夹中的文件，也即存放 css 和 javascript 文件的地方。Flask 会在 templates 文件夹里寻找 Jinja2 模板，之后教程中创建的模板将会放在这个文件夹里。

**步骤 1: 数据库模式**
首先我们要创建数据库模式。对于这个应用来说，一张表就足够了，而且只需支持 SQLite，所以会很简单。只需要把下面的内容放进一个名为 schema.sql 的文件，放在刚才创建的 flaskr 文件夹中:
```

drop table if exists entries;
create table entries (
  id integer primary key autoincrement,
  title string not null,
  text string not null
);

```
这个模式包含一个名为 entries 的表，该表中的每行都包含一个 id 、一个 title 和一个 text 。 id 是一个自增的整数，也是主键；其余的两个是字符串，且不允许为空。

##  步骤 2: 应用设置代码 
现在我们已经有了数据库模式**，我们可以创建应用的模块了。让我们把它叫做 flaskr.py ，并放置在 flaskr 目录下。**我们从添加所需的导入语句和添加配置部分开始。对于小型应用，可以直接把配置放在主模块里，正如我们现在要做的一样。但更简洁的方案是创建独立的 .ini 或 .py 文件，并载入或导入里面的值。
首先在 flaskr.py 里导入:
```

# all the imports
import os
import sqlite3
from flask import Flask, request, session, g, redirect, url_for, abort, \
     render_template, flash

```
Config 对象的用法如同字典，所以我们可以用新值更新它。
数据库路径
操作系统有进程当前工作目录的概念。不幸的是，你在 Web 应用中不能依赖此概念，因为你可能会在相同的进程中运行多个应用。
**为此，提供了 app.root_path 属性以获取应用的路径。配合 os.path 模块使用，轻松可达任意文件。**在本例中，我们把数据库放在根目录下。
对于实际生产环境的应用，推荐使用 实例文件夹 。
通常，加载一个单独的、环境特定的配置文件是个好主意。Flask 允许你导入多份配置，并且使用最后的导入中定义的设置。这使得配置设定过程更可靠。 ` from_envvar() ` 可用于达此目的。
```

app.config.from_envvar(‘FLASKR_SETTINGS’, silent=True)

```
**只需设置一个名为 FLASKR_SETTINGS 的环境变量，指向要加载的配置文件。**启用静默模式告诉 Flask 在没有设置该环境变量的情况下噤声。
此外，**你可以使用配置对象上的 from_object() 方法，并传递一个模块的导入名作为参数。Flask 会从这个模块初始化变量。 注意，` 只有名称全为大写字母的变量才会被采用 `。**
` secret_key 是保证客户端会话的安全的要点。 `正确选择一个尽可能难猜测、尽可能复杂的密钥。调试标志关系交互式调试器的开启。 永远不要在生产系统中激活调试模式 ，因为它将允许用户在服务器上执行代码。
我们还添加了一个让连接到指定数据库变得很简单的方法，这个方法用于在请求时开启一个数据库连接，并且在交互式 Python shell 和脚本中也能使用。这为以后的操作提供了相当的便利。我们创建了一个简单的 SQLite 数据库的连接，**并让它用 sqlite3.Row 表示数据库中的行。这使得我们可以通过字典而不是元组的形式访问行:**
```

def connect_db():
    """Connects to the specific database."""
    rv = sqlite3.connect(app.config['DATABASE'])
    rv.row_factory = sqlite3.Row
    return rv

```
最后，如果我们想要把这个文件当做独立应用来运行，我们只需在可启动服务器文件的末尾添加这一行:
```

if __name__ == '__main__':
    app.run()

```
如此我们便可以开始顺利运行这个应用，使用如下命令:
python flaskr.py

##  步骤 4: 创建数据库 
可以通过管道把 schema.sql 作为 sqlite3 命令的输入来创建这个模式，命令为如下:
sqlite3 /tmp/flaskr.db < schema.sql
这种方法的缺点是需要安装 sqlite3 命令，而并不是每个系统都有安装。而且你必须提供数据库的路径，否则将报错。用函数来初始化数据库是个不错的想法。
要这么做，我们可以创建一个名为 init_db 的函数来初始化数据库。 让我们首先看看代码。只需要把这个函数放在 flaskr.py 里的 connect_db 函数的后面:
```

def init_db():
    with app.app_context():
        db = get_db()
        with app.open_resource('schema.sql', mode='r') as f:
            db.cursor().executescript(f.read())
        db.commit()

```
那么，这段代码会发生什么？还记得吗？上个章节中提到，应用环境在每次请求传入时创建。这里我们并没有请求，所以我们需要手动创建一个应用环境。 g 在应用环境外无法获知它属于哪个应用，因为可能会有多个应用同时存在。
**with app.app_context() 语句为我们建立了应用环境。在 with 语句的内部， g 对象会与 app 关联。在语句的结束处，会释放这个关联兵执行所有销毁函数。这意味着数据库连接在提交后断开。**
**应用对象的 open_resource() 方法是一个很方便的辅助函数，可以打开应用提供的资源。这个函数从资源所在位置（ 你的 flaskr 文件夹）打开文件，并允许你读取它。**我们在此用它来在数据库连接上执行脚本。
SQLite 的数据库连接对象提供了一个游标对象。游标上有一个方法可以执行完整的脚本。最后我们只需提交变更。SQLite 3 和其它支持事务的数据库只会在你显式提交的时候提交。
现在可以在 Python shell 导入并调用这个函数来创建数据库:
如果你遇到了表无法找到的异常，请检查你是否确实调用过 init_db 函数并且表的名称是正确的（比如弄混了单数和复数）
##  步骤 5: 视图函数 
现在数据库连接已经正常工作，我们终于可以开始写视图函数了。我们一共需要写四个:
显示条目
这个视图显示数据库中存储的所有条目。它绑定在应用的根地址，并从数据库查询出文章的标题和正文。id 值最大的条目（最新的条目）会显示在最上方。从指针返回的行是按 select 语句中声明的列组织的元组。这对像我们这样的小应用已经足够了， 但是你可能会想把它转换成字典。如果你对这方面有兴趣，请参考 简化查询 的例子。
视图函数会将条目作为字典传递给 show_entries.html 模板，并返回渲染结果:
```

@app.route('/')
def show_entries():
    cur = g.db.execute('select title, text from entries order by id desc')
    entries = [dict(title=row[0], text=row[1]) for row in cur.fetchall()]
    return render_template('show_entries.html', entries=entries)

```
添加条目
这个视图允许已登入的用户添加新条目，并只响应 POST 请求，实际的表单显示在 show_entries 页。如果一切工作正常，我们会用 flash() 向下一次请求发送提示消息，并重定向回 show_entries 页:
```

@app.route('/add', methods=['POST'])
def add_entry():
    if not session.get('logged_in'):
        abort(401)
    g.db.execute('insert into entries (title, text) values (?, ?)',
                 [request.form['title'], request.form['text']])
    g.db.commit()
    flash('New entry was successfully posted')
    return redirect(url_for('show_entries'))

```
注意这里的用户登入检查（ logged_in 键在会话中存在，并且为 True ）
安全提示
**确保像上面例子中一样，使用问号标记来构建 SQL 语句。否则，当你使用格式化字符串构建 SQL 语句时，你的应用容易遭受 SQL 注入。 更多请见 在 Flask 中使用 SQLite 3 。**
登入和登出
这些函数用来让用户登入登出。登入通过与配置文件中的数据比较检查用户名和密码， 并设定会话中的 logged_in 键值。如果用户成功登入，那么这个键值会被设为 True ，并跳转回 show_entries 页。此外，会有消息闪现来提示用户登入成功。 如果发生一个错误，模板会通知，并提示重新登录。
```

@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        if request.form['username'] != app.config['USERNAME']:
            error = 'Invalid username'
        elif request.form['password'] != app.config['PASSWORD']:
            error = 'Invalid password'
        else:
            session['logged_in'] = True
            flash('You were logged in')
            return redirect(url_for('show_entries'))
    return render_template('login.html', error=error)

```
登出函数，做相反的事情，从会话中删除 logged_in 键。我们这里使用了一个简洁的方法：如果你使用字典的 pop() 方法并传入第二个参数（默认）， 这个方法会从字典中删除这个键，如果这个键不存在则什么都不做。这很有用，因为我们不需要检查用户是否已经登入。
```

@app.route('/logout')
def logout():
    session.pop('logged_in', None)
    flash('You were logged out')
    return redirect(url_for('show_entries'))

```
##  步骤 6: 模板 
接下来我们应该创建模板了。如果我们现在请求 URL，只会得到 Flask 无法找到模板的异常。 
**模板使用 Jinja2 语法并默认开启自动转义。这意味着除非你使用 Markup 标记或在模板中使用 |safe 过滤器，否则 Jinja 2 会确保特殊字符，比如 < 或 > 被转义为等价的 XML 实体。**
我们也会使用**模板继承**在网站的所有页面中重用布局。
将下面的**模板放在 templates 文件夹**里:
layout.html
这个模板包含 HTML 主体结构、标题和一个登入链接（用户已登入则提供登出）。 如果有，它也会显示闪现消息。** {% block body %} 块可以被子模板中相同名字的块（ body ）替换。**
**session 字典在模板中也是可用的**。你可以用它来检查用户是否已登入。 注意，在 Jinja 中你可以访问不存在的对象/字典属性或成员。比如下面的代码， 即便 'logged_in' 键不存在，仍然可以正常工作:
```

<!doctype html>
<title>Flaskr</title>
<link rel=stylesheet type=text/css href="![](/data/dokuwiki url_for('static', filename='style.css') )">
<div class=page>
  <h1>Flaskr</h1>
  <div class=metanav>
  {% if not session.logged_in %}
    <a href="![](/data/dokuwiki url_for('login') )">log in</a>
  {% else %}
    <a href="![](/data/dokuwiki url_for('logout') )">log out</a>
  {% endif %}
  </div>
  {% for message in get_flashed_messages() %}
    <div class=flash>![](/data/dokuwiki message )</div>
  {% endfor %}
  {% block body %}{% endblock %}
</div>

```
show_entries.html
这个模板继承了上面的 layout.html 模板来显示消息。注意 for 循环会遍历并输出所有 render_template() 函数传入的消息。我们还告诉表单使用 HTTP 的 POST 方法提交信息到 add_entry 函数:
```

{% extends "layout.html" %}
{% block body %}
  {% if session.logged_in %}
    <form action="![](/data/dokuwiki url_for('add_entry') )" method=post class=add-entry>
      <dl>
        <dt>Title:
        <dd><input type=text size=30 name=title>
        <dt>Text:
        <dd><textarea name=text rows=5 cols=40></textarea>
        <dd><input type=submit value=Share>
      </dl>
    </form>
  {% endif %}
  <ul class=entries>
  {% for entry in entries %}
    <li><h2>![](/data/dokuwiki entry.title )</h2>![](/data/dokuwiki entry.text|safe )
  {% else %}
    <li><em>Unbelievable.  No entries here so far</em>
  {% endfor %}
  </ul>
{% endblock %}

```
login.html
最后是登入模板，只是简单地显示一个允许用户登入的表单:
```

{% extends "layout.html" %}
{% block body %}
  <h2>Login</h2>
  {% if error %}<p class=error><strong>Error:</strong> ![](/data/dokuwiki error ){% endif %}
  <form action="![](/data/dokuwiki url_for('login') )" method=post>
    <dl>
      <dt>Username:
      <dd><input type=text name=username>
      <dt>Password:
      <dd><input type=password name=password>
      <dd><input type=submit value=Login>
    </dl>
  </form>
{% endblock %}

```
步骤 7: 添加样式
现在其它的一切都可以正常工作，是时候给应用添加样式了。**只需在之前创建的 static 文件夹中创建一个名为 style.css 的样式表**:
```

body            { font-family: sans-serif; background: #eee; }
a, h1, h2       { color: #377BA8; }
h1, h2          { font-family: 'Georgia', serif; margin: 0; }
h1              { border-bottom: 2px solid #eee; }
h2              { font-size: 1.2em; }

.page           { margin: 2em auto; width: 35em; border: 5px solid #ccc;
                  padding: 0.8em; background: white; }
.entries        { list-style: none; margin: 0; padding: 0; }
.entries li     { margin: 0.8em 1.2em; }
.entries li h2  { margin-left: -1em; }
.add-entry      { font-size: 0.9em; border-bottom: 1px solid #ccc; }
.add-entry dl   { font-weight: bold; }
.metanav        { text-align: right; font-size: 0.8em; padding: 0.3em;
                  margin-bottom: 1em; background: #fafafa; }
.flash          { background: #CEE5F5; padding: 0.5em;
                  border: 1px solid #AACBE2; }
.error          { background: #F0D6D6; padding: 0.5em; }

```
##  测试 Flask 应用 
没有经过测试的东西都是不完整的
Flask 提供了一种方法用于测试您的应用，那就是将 Werkzeug 测试 Client 暴露出来，并且为您操作这些内容的本地上下文变量。然后您就可以将自己最喜欢的测试解决方案应用于其上了。 在这片文档中，我们将会使用**Python自带的 unittest 包。**
测试的大框架
为了测试这个引用，我们添加了第二个模块(flaskr_tests.py)， 并且创建了一个框架如下:
```

import os
import flaskr
import unittest
import tempfile
class FlaskrTestCase(unittest.TestCase):
    def setUp(self):
        self.db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
        flaskr.app.config['TESTING'] = True
        self.app = flaskr.app.test_client()
        flaskr.init_db()

    def tearDown(self):
        os.close(self.db_fd)
        os.unlink(flaskr.app.config['DATABASE'])

if __name__ == '__main__':
    unittest.main()

```
在 setUp() 方法的代码创建了一个新的测试客户端并且初始化了一个新的数据库。这个函数将会在每次独立的测试函数运行之前运行。要在测试之后删除这个数据库，我们在 tearDown() 函数当中关闭这个文件，并将它从文件系统中删除。同时，在初始化的时候 TESTING 配置标志被激活，这将会使得处理请求时的错误捕捉失效，以便于您在进行对应用发出请求的测试时获得更好的错误反馈。
这个测试客户端将会给我们一个通向应用的简单接口，我们可以激发对向应用发送请求的测试，并且此客户端也会帮我们记录 Cookie 的动态。
因为 SQLite3 是基于文件系统的，我们可以很容易的**使用临时文件模块**来创建一个临时的数据库并初始化它，**函数 mkstemp() 实际上完成了两件事情：它返回了一个底层的文件指针以及一个随机的文件名**，后者我们用作数据库的名字。我们只需要将 db_fd 变量保存起来，就可以使用 os.close 方法来关闭这个文件。
如果我们运行这套测试，我们应该会得到如下的输出:
$ python flaskr_tests.py
Ran 0 tests in 0.000s
OK
虽然现在还未进行任何实际的测试，我们已经可以知道我们的 flaskr 程序没有语法错误了。否则，在 import 的时候就会抛出一个致死的错误了。
第一个测试
是进行第一个应用功能的测试的时候了。让我们检查当我们访问根路径(/)时应用程序是否正确地返回了了“No entries here so far” 字样。为此，我们添加了一个新的测试函数到我们的类当中， 如下面的代码所示:
```

class FlaskrTestCase(unittest.TestCase):
    def setUp(self):
        self.db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
        self.app = flaskr.app.test_client()
        flaskr.init_db()

    def tearDown(self):
        os.close(self.db_fd)
        os.unlink(flaskr.DATABASE)

    def test_empty_db(self):
        rv = self.app.get('/')
        assert 'No entries here so far' in rv.data

```
**注意到我们的测试函数以 test 开头，这允许 unittest 模块自动识别出哪些方法是一个测试方法，并且运行它。**
通过使用 self.app.get 我们可以发送一个 HTTP GET 请求给应用的某个给定路径。返回值将会是一个 response_class 对象。我们可以使用 data 属性来检查程序的返回值(以字符串类型)。在这里，我们检查 'No entries here so far' 是不是输出内容的一部分。
再次运行，您应该看到一个测试成功通过了:
$ python flaskr_tests.py
Ran 1 test in 0.034s
OK
登陆和登出
我们应用的大部分功能只允许具有管理员资格的用户访问。所以我们需要一种方法来帮助我们的测试客户端登陆和登出。为此，我们向登陆和登出页面发送一些请求，这些请求都携带了表单数据（用户名和密码），因为登陆和登出页面都会重定向，我们将客户端设置为 follow_redirects 。
将如下两个方法加入到您的 FlaskrTestCase 类:
```

def login(self, username, password):
    return self.app.post('/login', data=dict(
        username=username,
        password=password
    ), follow_redirects=True)

def logout(self):
    return self.app.get('/logout', follow_redirects=True)

```
现在我们可以轻松的测试登陆和登出是正常工作还是因认证失败而出错， 添加新的测试函数到类中:
```

def test_login_logout(self):
    rv = self.login('admin', 'default')
    assert 'You were logged in' in rv.data
    rv = self.logout()
    assert 'You were logged out' in rv.data
    rv = self.login('adminx', 'default')
    assert 'Invalid username' in rv.data
    rv = self.login('admin', 'defaultx')
    assert 'Invalid password' in rv.data

```
测试消息的添加
我们同时应该测试消息的添加功能是否正常，添加一个新的测试方法如下:
```

def test_messages(self):
    self.login('admin', 'default')
    rv = self.app.post('/add', data=dict(
        title='<Hello>',
        text='<strong>HTML</strong> allowed here'
    ), follow_redirects=True)
    assert 'No entries here so far' not in rv.data
    assert '&lt;Hello&gt;' in rv.data
    assert '<strong>HTML</strong> allowed here' in rv.data

```
这里我们测试计划的行为是否能够正常工作，即在正文中可以出现 HTML 标签，而在标题中不允许。
运行这个测试，我们应该得到三个通过的测试:
$ python flaskr_tests.py
Ran 3 tests in 0.332s
OK
关于请求的头信息和状态值等更复杂的测试，请参考 MiniTwit Example ，在这个例子的源代码里包含一套更长的测试。

其他测试技巧
除了如上文演示的使用测试客户端完成测试的方法，也有一个 test_request_context() 方法可以配合 with 语句用于激活一个临时的请求上下文。通过它，您可以访问 request 、g 和 session 类的对象，就像在视图中一样。 这里有一个完整的例子示范了这种用法:

```

app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    assert flask.request.path == '/'
    assert flask.request.args['name'] == 'Peter'

```
所有其他的和上下文绑定的对象都可以使用同样的方法访问。
如果您希望测试应用在不同配置的情况下的表现，这里似乎没有一个很好的方法，考虑使用应用的工厂函数(参考 应用程序的工厂函数)
注意，尽管你在使用一个测试用的请求环境，函数 before_request() 以及 after_request() 都不会自动运行。 然而，teardown_request() 函数在测试请求的上下文离开 with 块的时候会执行。如果您希望 before_request() 函数仍然执行。 您需要手动调用 preprocess_request() 方法:
```

app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    app.preprocess_request()
    ...

```
这对于打开数据库连接或者其他类似的操作来说，很可能是必须的，这视您应用的设计方式而定。
如果您希望调用 after_request() 函数， 您需要使用 process_response() 方法。 这个方法需要您传入一个 response 对象:
```

app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    resp = Response('...')
    resp = app.process_response(resp)
    ...

```
这通常不是很有效，因为这时您可以直接转向使用测试客户端。
伪造资源和上下文
0.10 新版功能.
在应用上下文或 flask.g 对象上存储用户认证信息和数据库连接非常常见。一般的模式是在第一次使用对象时，把对象放在应用上下文或 flask.g 上面，而在请求销毁时移除对象。试想一下例如下面的获取当前用户的代码:
```

def get_user():
    user = getattr(g, 'user', None)
    if user is None:
        user = fetch_current_user_from_database()
        g.user = user
    return user

```
对于测试，这样易于从外部覆盖这个用户，而不用修改代码。连接 flask.appcontext_pushed 信号可以很容易地完成这个任务:
```

from contextlib import contextmanager
from flask import appcontext_pushed

@contextmanager
def user_set(app, user):
    def handler(sender, **kwargs):
        g.user = user
    with appcontext_pushed.connected_to(handler, app):
        yield

```
并且之后使用它:

```

from flask import json, jsonify

@app.route('/users/me')
def users_me():
    return jsonify(username=g.user.username)

with user_set(app, my_user):
    with app.test_client() as c:
        resp = c.get('/users/me')
        data = json.loads(resp.data)
        self.assert_equal(data['username'], my_user.username)

```
保存上下文
0.4 新版功能.
有时，激发一个通常的请求，但是将当前的上下文保存更长的时间，以便于附加的内省发生是很有用的。 在 Flask 0.4 中，通过 test_client() 函数和 with 块的使用可以实现:
```

app = flask.Flask(__name__)

with app.test_client() as c:
    rv = c.get('/?tequila=42')
    assert request.args['tequila'] == '42'

```
如果您仅仅使用 test_client() 方法，而不使用 with 代码块， assert 断言会失败，因为 request 不再可访问(因为您试图在非真正请求中时候访问它)。

访问和修改 Sessions
0.8 新版功能.

有时，在测试客户端里访问和修改 Sesstions 可能会非常有用。 通常有两种方法实现这种需求。如果您仅仅希望确保一个 Session 拥有某个特定的键，且此键的值是某个特定的值，那么您可以只保存起上下文，并且访问 ```

flask.session:

with app.test_client() as c:
    rv = c.get('/')
    assert flask.session['foo'] == 42

```
但是这样做并不能使您修改 Session 或在请求发出之前访问 Session。 从 Flask 0.8 开始，我们提供一个叫做 “Session 事务” 的东西用于模拟适当的调用，从而在测试客户端的上下文中打开一个 Session，并用于修改。在事务的结尾，Session 将被恢复为原来的样子。这些都独立于 Session 的后端使用:

```

with app.test_client() as c:
    with c.session_transaction() as sess:
        sess['a_key'] = 'a value'

    # once this is reached the session was stored

```
注意到，在此时，您必须使用这个 sess 对象而不是调用 flask.session 代理，而这个对象本身提供了同样的接口。


##  路由注册 
http://docs.jinkan.org/docs/flask/api.html#url-route-registrations
URL 路由注册
在路由系统中定义规则可以的方法可以概括为三种:
使用 flask.Flask.route() 装饰器
使用 flask.Flask.add_url_rule() 函数
直接访问暴露为 flask.Flask.url_map 的底层的 Werkzeug 路由系统
路由中的变量部分可以用尖括号指定（ /user/<username>）。默认情况下，URL 中的变量部分接受任何不带斜线的字符串，而 <converter:name> 也可以指定不同的转换器。
变量部分以关键字参数传递给视图函数。
下面的转换器是可用的:
  * string	接受任何不带斜线的字符串（默认的转换器）
  * int	接受整数
  * float	同 int ，但是接受浮点数
  * path	和默认的相似，但也接受斜线

你可以为同一个函数定义多个规则。无论如何，他们也要唯一。也可以给定默认值。 
```

@app.route('/users/', defaults={'page': 1})
@app.route('/users/page/<int:page>')
def show_users(page):
    pass

```
以下是 route() 和 add_url_rule() 接受的参数。两者唯一的区别是，带有路由参数的视图函数用装饰器定义，而不是 view_func 参数。
  * rule	URL 规则的字符串
  * endpoint	注册的 URL 规则的末端。如果没有显式地规定，Flask 本身假设末端的名称是视图函数的名称，。
  * view_func	当请求呈递到给定的末端时调用的函数。
  * defaults	规则默认值的字典。
  * subdomain	当使用子域名匹配的时候，为子域名设定规则。
  * * *options	这些选项会被推送给底层的 Rule 对象。比如 **method** 选项被限定的方法列表**（ GET ， POST** 等等）。默认情况下，规则只监听 GET （也隐式地监听 HEAD ）
```

app.add_url_rule('/', index)

```