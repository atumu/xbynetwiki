title: python学习笔记027 

#  Python学习笔记之Flask之信号 
依赖库blinker
从 Flask 0.6 开始，** Flask 集成了信号支持。这个支持由 blinker 库提供**， 并且当它不可用时会优雅地退回。
什么是信号？信号通过发送发生在核心框架的其它地方或 Flask 扩展的动作时的通知来帮助你解耦应用。简而言之，信号允许特定的发送端通知订阅者发生了什么。

Flask 提供了几个信号，其它的扩展可能会提供更多。另外，请注意信号倾向于通知订阅者，而不应该鼓励订阅者修改数据。
你会注意到，信号似乎和一些内置的装饰器做同样的事情（例如： request_started 与 before_request() 十分相似）。
然而它们工作的方式是有差异的。譬如核心的 before_request() 处理程序以特定的顺序执行，并且可以在返回响应之前放弃请求。相比之下，所有的信号处理器执行的顺序没有定义，并且不修改任何数据。

##  订阅信号 
**你可以使用信号的 connect() 方法来订阅信号。**
` 该函数的第一个参数是信号发出时**要调用的函数**，第二个参数是可选的，用于确定**信号的发送端**。 `
**退订一个信号，可以使用 disconnect() 方法**。
对于所有的核心 Flask 信号，发送端都是发出信号的应用。当你订阅一个信号，请确保也提供一个发送端，除非你确实想监听全部应用的信号。这在你开发一个扩展的时候尤其正确。
比如这里有一个用于在单元测试中找出哪个模板被渲染和传入模板的变量的助手上下文管理器:
```

from flask import template_rendered  #这是一个Flask内置信号
from contextlib import contextmanager
@contextmanager
def captured_templates(app):
    recorded = []
    def record(sender, template, context, **extra):
        recorded.append((template, context))
    template_rendered.connect(record, app) #使用record函数订阅信号flask的内置信号template_rendered。这个信号会传送三个参数sender,template,context，以及额外的关键字参数给record
    try:
        yield recorded
    finally:
        template_rendered.disconnect(record, app) #断开信号连接

```
这可以很容易地与一个测试客户端配对:
```

with captured_templates(app) as templates:
    rv = app.test_client().get('/')
    assert rv.status_code == 200
    assert len(templates) == 1
    template, context = templates[0]
    assert template.name == 'index.html'
    assert len(context['items']) == 10

```
**确保订阅使用了一个额外的 * *extra 参数，这样当 Flask 对信号引入新参数时你的调用不会失败。**
代码中，从 with 块的应用 app 中流出的渲染的所有模板现在会被记录到 templates 变量。无论何时模板被渲染，模板对象和上下文中都会被添加到它里面。
此外，也有一个方便的助手方法（ connected_to() ） ，它允许你临时地把函数订阅到信号并使用信号自己的上下文管理器。因为这个上下文管理器的返回值不能由我们决定，所以必须把列表作为参数传入:
```

from flask import template_rendered
def captured_templates(app, recorded, **extra):
    def record(sender, template, context):
        recorded.append((template, context))
    return template_rendered.connected_to(record, app)

```
上面的例子会看起来是这样:
```

templates = []
with captured_templates(app, templates, **extra):
    ...
    template, context = templates[0]

```
Blinker API 变更
connected_to() 方法出现于 Blinker 1.1 。

##  创建信号 
**如果你想要在自己的应用中使用信号，你可以直接使用 blinker 库。最常见的用法是在自定义的 Namespace 中命名信号。**这也是大多数时候推荐的做法:
```

from blinker import Namespace
my_signals = Namespace()
model_saved = my_signals.signal('model-saved')

```
这里使用唯一的信号名，简化调试。可以用 name 属性来访问信号名。
给扩展开发者：如果你在编写一个 Flask 扩展并且你想优雅地在没有 blinker 安装时退化，你可以用 flask.signals.Namespace 这么做。
##  发送信号 
**如果你想要发出信号，调用 send() 方法可以做到。 它接受发送端作为第一个参数，和一些推送到信号订阅者的可选关键字参数:**
```

class Model(object):
    ...

    def save(self):
        model_saved.send(self)

```
` 永远尝试选择一个合适的发送端。如果你有一个发出信号的类，把 self 作为发送端。如果你从一个随机的函数发出信号，把 current_app._get_current_object() 作为发送端。 `
永远不要向信号传递 current_app 作为发送端，使用 current_app._get_current_object() 作为替代。这样的原因是， current_app 是一个代理，而不是真正的应用对象。
##  信号与 Flask 的请求上下文 
**信号在接收时，完全支持 请求上下文 。上下文本地的变量在 request_started 和 request_finished 一贯可用**， 所以你可以信任 flask.g 和其它需要的东西。
注意 发送信号 和 request_tearing_down 信号中描述的限制。
##  基于装饰器的信号订阅 
你可以在 Blinker 1.1 中容易地用**新的 connect_via() 装饰器订阅信号**:
```

from flask import template_rendered
@template_rendered.connect_via(app)
def when_template_rendered(sender, template, context, **extra):
    print 'Template %s is rendered with %s' % (template.name, context)

```

##  flask的核心信号 
下列是 Flask 中存在的信号:
  * 1、flask.template_rendered：当模板成功渲染的时候，这个信号会发出。
  * 2、flask.request_started：这个信号在处建立请求上下文之外的任何请求处理开始前发送。
  * 3、flask.request_finished：这个信号恰好在请求发送给客户端之前发送。
  * 4、flask.got_request_exception：这个信号在请求处理中抛出异常时发送。
  * 5、flask.request_tearing_down：这个信号在请求销毁时发送。它总是被调用，即使发生异常。
  * 6、flask.appcontext_tearing_down：这个信号在应用上下文销毁时发送。它总是被调用，即使发生异常。
  * 7、flask.appcontext_pushed：这个信号在应用上下文压入栈时发送。发送者是应用对象。
  * 8、flask.appcontext_popped：这个信号在应用上下文弹出栈时发送。发送者是应用对象。这通常在 appcontext_tearing_down 信号发送后发送。
  * 9、flask.message_flashed这个信号在应用对象闪现一个消息时发送。消息作为 message 命名参数发送， 分类则是 category 参数。
下面分别介绍他们
**1、flask.template_rendered：当模板成功渲染的时候，这个信号会发出。这个信号与模板实例 template 和上下文的字典（名为 context ）一起调用。**
订阅示例:
```

def log_template_renders(sender, template, context, **extra):
    sender.logger.debug('Rendering template "%s" with context %s',
                        template.name or 'string template',
                        context)

from flask import template_rendered
template_rendered.connect(log_template_renders, app)

```
**2、flask.request_started：这个信号在处建立请求上下文之外的任何请求处理开始前发送。**因为请求上下文已经被约束，订阅者可以用 request 之类的标准全局代理访问请求。
订阅示例:
```

def log_request(sender, **extra):
    sender.logger.debug('Request context is set up')

from flask import request_started
request_started.connect(log_request, app)

```
**3、flask.request_finished：这个信号恰好在请求发送给客户端之前发送。它传递名为 response 的响应。**
订阅示例:
```

def log_response(sender, response, **extra):
    sender.logger.debug('Request context is about to close down.  '
                        'Response: %s', response)

from flask import request_finished
request_finished.connect(log_response, app)


```
**4、flask.got_request_exception：这个信号在请求处理中抛出异常时发送。**它在标准异常处理生效 之前 ，甚至是在没有异常处理的情况下发送**。异常本身会通过 exception 传递到订阅函数。**
订阅示例:
```

def log_exception(sender, exception, **extra):
    sender.logger.debug('Got exception during processing: %s', exception)

from flask import got_request_exception
got_request_exception.connect(log_exception, app)

```
**5、flask.request_tearing_down：这个信号在请求销毁时发送。它总是被调用，即使发生异常。**当前监听这个信号的函数会在常规销毁处理后被调用，但这不是你可以信赖的。
订阅示例:
```

def close_db_connection(sender, **extra):
    session.close()

from flask import request_tearing_down
request_tearing_down.connect(close_db_connection, app)

```
从 Flask 0.9 ，如果有异常的话它会被传递一个 exc 关键字参数引用导致销毁的异常。
**6、flask.appcontext_tearing_down：这个信号在应用上下文销毁时发送。它总是被调用，即使发生异常。**当前监听这个信号的函数会在常规销毁处理后被调用，但这不是你可以信赖的。
订阅示例:
```

def close_db_connection(sender, **extra):
    session.close()

from flask import request_tearing_down
appcontext_tearing_down.connect(close_db_connection, app)

```
如果有异常它会被传递一个 exc 关键字参数引用导致销毁的异常。
**7、flask.appcontext_pushed：这个信号在应用上下文压入栈时发送。发送者是应用对象。**这通常在单元测试中为了暂时地钩住信息比较有用。例如这可以用来提前在 g 对象上设置一些资源。
用法示例:
```

from contextlib import contextmanager
from flask import appcontext_pushed
@contextmanager
def user_set(app, user):
    def handler(sender, **kwargs):
        g.user = user
    with appcontext_pushed.connected_to(handler, app):
        yield
测试代码:
def test_user_me(self):
    with user_set(app, 'john'):
        c = app.test_client()
        resp = c.get('/users/me')
        assert resp.data == 'username=john'

```
0.10 新版功能.
**8、flask.appcontext_popped：这个信号在应用上下文弹出栈时发送。发送者是应用对象。这通常在 appcontext_tearing_down 信号发送后发送。**

**9、flask.message_flashed这个信号在应用对象闪现一个消息时发送。消息作为 message 命名参数发送， 分类则是 category 参数。**
```

recorded = []
def record(sender, message, category, **extra):
    recorded.append((message, category))

from flask import message_flashed
message_flashed.connect(record, app)

```
