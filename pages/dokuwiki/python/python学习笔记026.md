location:dokuwiki/python/python学习笔记026
modifyAt:2016-12-15 22:18:09
author:xbynet
createAt:2016-12-15 22:18:09
title:Flask之请求上下文与应用上下文

#  Python学习笔记之Flask之请求上下文与应用上下文 
##  请求上下文 
比如说你有一个应用函数返回用户应该跳转到的 URL 。想象它总是会跳转到 URL 的 next 参数，或 HTTP referrer ，或索引页:
```

from flask import request, url_for
def redirect_url():
    return request.args.get('next') or \
           request.referrer or \
           url_for('index')

```
如你所见，它访问了请求对象。当你试图在纯 Python shell 中运行这段代码时， 你会看见这样的异常:
```

>>> redirect_url()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'NoneType' object has no attribute 'request'

```
这有很大意义，**因为我们当前并没有可以访问的请求。所以我们需要制造一个请求并且绑定到当前的上下文。 test_request_context 方法为我们创建一个 RequestContext:**
```

>>> ctx = app.test_request_context('/?next=http://example.com/')
可以通过两种方式利用这个上下文：使用 with 声明或是调用 push() 和 pop() 方法:
>>> ctx.push()
从这点开始，你可以使用请求对象:
>>> redirect_url()
u'http://example.com/'
直到你调用 pop:
>>> ctx.pop()

```
**因为请求上下文在内部作为一个栈来维护，所以你可以多次压栈出栈。**这在实现内部重定向之类的东西时很方便。

###  上下文如何工作 
**request_context() 方法返回一个新的 RequestContext 对象，并结合 with 声明来绑定上下文。 从相同线程中被调用的一切，直到 with 声明结束前，都可以访问全局的请求变量（ flask.request 和其它）。
请求上下文内部工作如同一个栈。栈顶是当前活动的请求。 push() 把上下文添加到栈顶， pop() 把它移出栈。**在出栈时，应用的 teardown_request() 函数也会被执行。
另一件需要注意的事是，请求上下文被压入栈时，并且没有当前应用的应用上下文， 它会自动创建一个 应用上下文 。


**上下文**在一个线程中全局可访问的特定变量，**flask保证不同线程之间互不干扰**。**作用区域线程**（对应处理接收的请求）。目的从客户端接收请求时，总需要访问一些通用的对象（如，请求对象）。如果作为参数进行传输，势必增大视图函数工作。避免把视图函数弄复杂，**使用上下文临时把某些对象变为全局可访问的对象。**

上下文分为：程序上下文和请求上下文。
  * **current_app	程序上下文	当前激活程序的程序实例**
  * **g	程序上下文	处理请求时用作临时存储的对象。每次请求都会重设这个变量**
  * request	请求上下文	请求对象,封装了客户端发出的 HTTP 请求中的内容
  * session	请求上下文	用户会话,用于存储请求之间需要“记住”的值的词典
 Flask 在分发请求之前激活(或推送)程序和请求上下文,请求处理完成后再将其删除。**如果使用这些变量时我们没有激活程序上下文或请求上下文,就会导致错误。**


### =请求钩子与回调和错误 
**用途：**处理请求之前或之后的人物。如：在请求开始时,我们可能需要创建数据库连接或者认证发起请求的用户。为了避免在每个视图函数中都使用重复的代码,Flask 提供了注册通用函数的功能,注册的函数可在请求被分发到视图函数之前或之后调用。 
**方法：**同样钩子也是使用修饰器来实现的。Flask支持4中钩子 
• before_first_request :注册一个函数,在处理第一个请求之前运行。 
• before_request :注册一个函数,在每次请求之前运行。 
• after_request :注册一个函数,如果没有未处理的异常抛出,在每次请求之后运行。 
• teardown_request :注册一个函数,即使有未处理的异常抛出,也在每次请求之后运行。 
**在请求钩子函数和视图函数之间共享数据一般使用上下文全局变量g**。例如, before_request处理程序可以从数据库中加载已登录用户,并将其保存到g.user中。随后调用视图函数时,视图函数再使用g.user获取用户。

在 Flask 中，请求处理时发生一个错误时会发生什么？
**在每个请求之前，执行`  before_request() 上绑定的函数 `。 如果这些函数中的某个返回了一个响应，其它的函数将不再被调用。任何情况下，无论如何这个返回值都会替换视图的返回值。**(befire_request相当于前置拦截器)

**视图的返回值之后会被转换成一个实际的响应对象，并交给`  after_request() 上绑定的函数 `适当地替换或修改它。**
**在请求的最后，会执行 ` teardown_request() 上绑定的函数 `。**（即使在一个未处理的异常抛出后或是没有请求前处理器执行过，也会发生） （例如在测试环境中你有时会想不执行请求前回调）。


###  销毁回调 
销毁回调是是特殊的回调，因为它们在不同的点上执行。严格地说，它们不依赖实际的请求处理，因为它们限定在 RequestContext 对象的生命周期。 **当请求上下文出栈时， teardown_request() 上绑定的函数会被调用。**
这对于了解请求上下文的寿命是否因为在 with 声明中使用测试客户端或在命令行中使用请求上下文时被延长很重要:
```

with app.test_client() as client:
    resp = client.get('/foo')
    # the teardown functions are still not called at that point
    # even though the response ended and you have the response
    # object in your hand

# only when the code reaches this point the teardown functions
# are called.  Alternatively the same thing happens if another
# request was triggered from the test client
从这些命令行操作中，很容易看出它的行为:
>>> app = Flask(__name__)
>>> @app.teardown_request
... def teardown_request(exception=None):
...     print 'this runs after request'
...
>>> ctx = app.test_request_context()
>>> ctx.push()
>>> ctx.pop()
this runs after request
>>>

```
**注意销毁回调总是会被执行，即使没有请求前回调执行过，或是异常发生。**测试系统的特定部分也会临时地在不调用请求前处理器的情况下创建请求上下文。确保你写的请求销毁处理器不会报错。
###  代理与真实对象 
Flask 中提供的一些对象是其它对象的代理。背后的原因是，**这些代理在线程间共享**， 并且它们在必要的情景中被调度到限定在一个线程中的实际的对象。
大多数时间你不需要关心它，但是在一些例外情况中，知道一个对象实际上是代理是有益的:
代理对象不会伪造它们继承的类型，所以如果你想运行真正的实例检查，你需要在被代理的实例上这么做（见下面的 ` _get_current_object ` ）。
如果对象引用是重要的（例如发送 信号 ）
**如果你需要访问潜在的被代理的对象，你可以使用 _get_current_object() 方法:**
<wrap em>app = current_app._get_current_object()
my_signal.send(app)</wrap>
###  错误时的上下文保护 
**无论错误出现与否，在请求的最后，请求上下文会出栈，并且相关的所有数据会被销毁。**在开发中，当你想在异常发生时，长期地获取周围的信息，这会成为麻烦。 
从 Flask 0.7 开始，我们设定** PRESERVE_CONTEXT_ON_EXCEPTION 配置变量来更好地控制该行为**。这个值默认与 DEBUG 的设置相关。当应用工作在调试模式下时，上下文会被保护，而生产模式下相反。
不要在生产模式强制激活 PRESERVE_CONTEXT_ON_EXCEPTION ，因为它会导致在异常时应用的内存泄露。不过，它在开发时获取开发模式下相同的错误行为来试图调试一个只有生产设置下才发生的错误时很有用。

##  应用上下文 
Flask 背后的设计理念之一就是，**代码在执行时会处于两种不同的“状态”（states）**。 
初始化状态(第一个请求来到之前的状态)当应用处于这个状态的时候：
  * 程序员可以安全地修改应用对象
  * 目前还没有处理任何请求

**相反，到了第二个状态，在处理请求时：**
**当一个请求激活时，上下文的本地对象（ flask.request 和其它对象等） 指向当前的请求**
这里有一个第三种情况，有一点点差异。有时，你正在用类似请求处理时方式来与应用交互，即使并没有活动的请求。想象一下你用交互式 Python shell 与应用交互的情况，或是一个命令行应用的情况。
**current_app 上下文本地变量就是应用上下文驱动的。current_app是一个代理对象。（current_app.` _get_current_object `()）**
###  应用上下文的作用 
**current_app 代理对象，它被绑定到当前请求的应用的引用。**既然无论如何在没有请求时创建一个这样的请求上下文是一个没有必要的昂贵操作，应用上下文就被引入了。
创建应用上下文
有两种方式来创建应用上下文。**第一种是隐式的：无论何时当一个请求上下文被压栈时，** 如果有必要的话一个应用上下文会被一起创建。由于这个原因，你可以忽略应用上下文的存在，除非你需要它。
**第二种是显式地调用 app_context() 方法:**
```

from flask import Flask, current_app
app = Flask(__name__)
with app.app_context():
    # within this block, current_app points to app.
    print current_app.name

```
在配置了 SERVER_NAME 时，应用上下文也被用于 url_for() 函数。这允许你在没有请求时生成 URL 。

###  应用上下文局部变量 
**应用上下文会在必要时被创建和销毁。它不会在线程间移动，并且也不会在不同的请求之间共享。**
正因为如此，它是一个存储数据库连接信息或是别的东西的最佳位置。内部的栈对象叫做 ` flask._app_ctx_stack ` 。扩展可以在最顶层自由地存储额外信息，想象一下它们用一个充分独特的名字在那里存储信息，
而不是在 flask.g 对象里， flask.g 是留给用户的代码用的。

###  上下文用法 
**上下文的一个典型应用场景就是用来缓存一些我们需要在发生请求之前或者要使用的资源。举个例子，比如数据库连接。**
最常见的应用就是把资源的管理分成如下两个部分：
  * 一个缓存在上下文中的隐式资源
  * 当上下文被销毁时重新分配基础资源
如下是我们刚刚提到的连接数据库的例子:
```

import sqlite3
from flask import g
def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = connect_to_database()
    return db

@app.teardown_appcontext
def teardown_db(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()

```
为了使得看起来更隐式一点我们可以使用 LocalProxy 这个类：
from werkzeug.local import LocalProxy db = LocalProxy(get_db)
这样的话用户就可以直接通过访问 db 来获取数据句柄了， db 已经在内部完成了对 get_db() 的调用。
http://blog.csdn.net/sun_dragon/article/details/51697331