title: python学习笔记030 

#  Python学习笔记之Flask与Shell 
Python 交互式 Shell 允许你实时的运行 Python 命令并且立即得到返回结果。
交互式控制台，因此 g 和 request 以及其他的一些函数不能使用。怎么解决呢？
在阅读本章节之前还是建议大家先阅读 请求上下文 相关章节。

**创建一个请求上下文**
```

>>> ctx = app.test_request_context()

```
一般来说，您可以使用 with 声明来激活这个请求对象， 但是在终端中，调用 push() 方法和 pop() 方法会更简单:
```

>>> ctx.push()

```
从这里往后，您就可以使用这个请求对象直到您调用 pop 方法为止:
```

>>> ctx.pop()

```

**激发请求发送前后的调用**
仅仅创建一个请求上下文，您仍然不能运行请求发送前通常会运行的代码。 如果您在将连接数据库的任务分配给发送请求前的函数调用，或者在当前用户并没有被储存在 g 对象里等等情况下，您可能无法访问到数据库。
您可以很容易的自己完成这件事，仅仅手动调用 preprocess_request() 函数即可:
```

>>> ctx = app.test_request_context()
>>> ctx.push()
>>> app.preprocess_request()

```
请注意， preprocess_request() 函数可能会返回一个响应对象。这时，忽略它就好了。
要关闭一个请求，您需要在请求后的调用函数(由 process_response() 函数激发)运行之前耍一些小小的把戏:
```

>>> app.process_response(app.response_class())
<Response 0 bytes [200 OK]>
>>> ctx.pop()

```
被注册为 teardown_request() 的函数将会在上下文环境出栈之后自动执行。所以这是用来销毁请求上下文(如数据库连接等)资源的最佳地点。

集成 Python shell

##  让 Flask-Script 的 shell 命令自动导入特定的对象 
```

from flask.ext.script import Shell
def make_shell_context():
    return dict(app=app, db=db, User=User, Role=Role)  
manager.add_command("shell", Shell(make_context=make_shell_context))


```
**make_shell_context() 函数注册了程序、数据库实例以及模型,因此这些对象能直接导入 shell**
