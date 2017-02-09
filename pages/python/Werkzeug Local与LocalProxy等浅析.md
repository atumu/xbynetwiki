author:xbynet
title:Werkzeug Local与LocalProxy等浅析
modifyAt:2016-12-12 03:18:18
location:python/Werkzeug Local与LocalProxy等浅析
createAt:2016-12-12 03:18:18

首先贴出官方文档地址：http://werkzeug.pocoo.org/docs/0.11/local/
**几个local？**
threading.local
werkzeug.local模块中的：
Local
LocalStack
LocaProxy

**why not threading.local?**

threading.local,以前接触过java的，对这个再熟悉不过了。线程局部变量，也就是每个线程的私有变量，具有线程隔离性。

按我们正常的理解，应该是每一个http请求对应一个处理线程。那么这样看来使用threading.local应该够了，**为什么werkzeug还自己搞了一套？**装逼？非也。

在python中，除了线程之外，还有个叫`协程`的东东，(这里不提进程)。java中貌似是无法实现协程的。而python的协程感觉高大尚的样子，python3.5开始对协程内置支持，而且也有相关开源库greenlet等。

**协程是什么？**
举个例子，比如一个线程在处理IO时，该线程是处于空闲状态的，等待IO返回。但是此时如果不让我们的线程干等着cpu时间片耗光，有没有其他办法，解决思路就是采用协程处理任务，一个线程中可以运行多个协程，当当前协程去处理IO时，线程可以马上调度其他协程继续运行，而不是干等着不干活。

这么一说，我们知道了`协程会复用线程`，WSGI不保证每个请求必须由一个线程来处理，如果WSGI服务器不是每个线程派发一个请求，而是每个协程派发一个请求，所以如果使用thread local变量可能会造成请求间数据相互干扰，因为一个线程中存在多个请求。
所以werkzeug给出了自己的解决方案：`werkzeug.local`模块。

```
from werkzeug.local import Local, LocalManager

local = Local()
local_manager = LocalManager([local])

def application(environ, start_response):
    local.request = request = Request(environ)
    ...

application = local_manager.make_middleware(application)
```

Local配合LocalManager会确保不管是协程还是线程，只要当前请求处理完成之后清除Local中对应的内容。

```
>>> loc = Local()
>>> loc.foo = 42
>>> release_local(loc)
>>> hasattr(loc, 'foo')
```

当然，你也可以调用werkzeug.local.release_local(local)手动释放Local或者LocalStack ，但是不能清除代理对象LocalProxy(代理对象底层保留了对Local对象的引用，以便在之后释放)的数据。

```
>>> ls = LocalStack()
>>> ls.push(42)
>>> ls.top
42
>>> ls.push(23)
>>> ls.top
23
>>> ls.pop()
23
>>> ls.top
```

LocalStack，与Local类似，但是管理数据的方式是采用栈的方式，可以通过LocalManager对象强制释放，但是不建议这么做，而是通过其pop方法弹出。

```
from werkzeug.local import Local
l = Local()

# these are proxies
request = l('request')
user = l('user')


from werkzeug.local import LocalStack
_response_local = LocalStack()

# this is a proxy
response = _response_local()
```

werkzeug.local.LocalProxy:Local对象的一个代理。如果你需要创建Local或LocalStack对象的代理，可以直接call。

```
session = LocalProxy(lambda: get_current_request().session)

from werkzeug.local import Local, LocalProxy
local = Local()
request = LocalProxy(local, 'request')

>>> from werkzeug.local import LocalProxy
>>> isinstance(request, LocalProxy)
True
```

你也可以通过LocalProxy构造一个代理对象，参数为可以调用的对象或者函数。
_get_current_object()返回被代理的对象。


werkzeug.local模块关键部分代码：

    import copy
    from functools import update_wrapper
    from werkzeug.wsgi import ClosingIterator
    from werkzeug._compat import PY2, implements_bool
    try:
        from greenlet import getcurrent as get_ident
    except ImportError:
        try:
            from thread import get_ident
        except ImportError:
            from _thread import get_ident
    
    
    def release_local(local):
        local.__release_local__()
    
    
    class Local(object):
        __slots__ = ('__storage__', '__ident_func__')
    
        def __init__(self):
            object.__setattr__(self, '__storage__', {})
            object.__setattr__(self, '__ident_func__', get_ident)
    
        def __iter__(self):
            return iter(self.__storage__.items())
    
        def __call__(self, proxy):
            """Create a proxy for a name."""
            return LocalProxy(self, proxy)
    
        def __release_local__(self):
            self.__storage__.pop(self.__ident_func__(), None)
    
        def __getattr__(self, name):
            try:
                return self.__storage__[self.__ident_func__()][name]
            except KeyError:
                raise AttributeError(name)
    
        def __setattr__(self, name, value):
            ident = self.__ident_func__()
            storage = self.__storage__
            try:
                storage[ident][name] = value
            except KeyError:
                storage[ident] = {name: value}
    
        def __delattr__(self, name):
            try:
                del self.__storage__[self.__ident_func__()][name]
            except KeyError:
                raise AttributeError(name)
    
    
    class LocalStack(object):
    
        def __init__(self):
            self._local = Local()
    
        def __release_local__(self):
            self._local.__release_local__()
    
        def __call__(self):
            def _lookup():
                rv = self.top
                if rv is None:
                    raise RuntimeError('object unbound')
                return rv
            return LocalProxy(_lookup)
    
        def push(self, obj):
            rv = getattr(self._local, 'stack', None)
            if rv is None:
                self._local.stack = rv = []
            rv.append(obj)
            return rv
    
        def pop(self):
            stack = getattr(self._local, 'stack', None)
            if stack is None:
                return None
            elif len(stack) == 1:
                release_local(self._local)
                return stack[-1]
            else:
                return stack.pop()
    
        @property
        def top(self):
            try:
                return self._local.stack[-1]
            except (AttributeError, IndexError):
                return None
    
    
    class LocalManager(object):
    
        def cleanup(self):
            for local in self.locals:
                release_local(local)
    
        def make_middleware(self, app):
            def application(environ, start_response):
                return ClosingIterator(app(environ, start_response), self.cleanup)
            return application
    
    
    @implements_bool
    class LocalProxy(object):
    
        def __init__(self, local, name=None):
            object.__setattr__(self, '_LocalProxy__local', local)
            object.__setattr__(self, '__name__', name)
    
        def _get_current_object(self):
            if not hasattr(self.__local, '__release_local__'):
                return self.__local()
            try:
                return getattr(self.__local, self.__name__)
            except AttributeError:
                raise RuntimeError('no object bound to %s' % self.__name__)

