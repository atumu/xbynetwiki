author:xbynet
title:python抽象基类abc
modifyAt:2016-12-26 15:04:23
location:python/modules/abc
createAt:2016-12-26 15:00:34

PEP里面关于抽象类的相关介绍:https://www.python.org/dev/peps/pep-3119/
python中并没有提供抽象类与抽象方法，但是提供了内置模块abc(abstract base class)来模拟实现抽象类。
主要类或函数：
`abc.ABCMeta` 这是用来生成抽象基础类的元类。由它生成的类可以被直接继承。
` abc.ABC`辅助类，让你可以不用关心元类概念，直接继承它，就有了ABCMeta元类。使用时注意元类冲突
`@abc.abstractmethod` 定义抽象方法，除了这个装饰器，其余装饰器都被deprecated了。


```
from abc import ABCMeta

class MyABC(metaclass=ABCMeta):
    pass

MyABC.register(tuple)

assert issubclass(tuple, MyABC)
assert isinstance((), MyABC)
```
上面这个例子中，首先生成了一个MyABC的抽象基础类，然后再将tuple变成它的`虚拟子类`。然后通过`issubclass或者isinstance`都可以判断出tuple是不是出于MyABC类。
另外，也可以通过复写`__subclasshook__(subclass)`来改变`issubclass或者isinstance`的行为，`__subclasshook__(subclass)`必须定义为classmethod

```
class Foo:
    def __getitem__(self, index):
        ...
    def __len__(self):
        ...
    def get_iterator(self):
        return iter(self)

class MyIterable(metaclass=ABCMeta):

    @abstractmethod
    def __iter__(self):
        while False:
            yield None

    def get_iterator(self):
        return self.__iter__()

    @classmethod
    def __subclasshook__(cls, C):
        if cls is MyIterable:
            if any("__iter__" in B.__dict__ for B in C.__mro__):
                return True
        return NotImplemented

MyIterable.register(Foo)
```

通过`@abc.abstractmethod`将方法声明为抽象方法。
具体化抽象类可以有两种方式，**一种通过注册（register），另外一种通过继承。**
上面介绍的是注册方式，
**注册方式的缺点**：不会出现在类的`MRO` (Method Resolution Order)，故而也不能通过`super()`来调用抽象方法。当没有实现抽象方法时，实例化时候不会报错，只有在调用时候才会报错。

下面介绍继承方式：
**继承方式的优点**：直接从抽象基类派生子类有一个好处，**除非子类实现抽象基类的抽象方法，否则子类不能实例化。**

继承方式：

```
import abc

class PluginBase(metaclass= abc.ABCMeta):
    #__metaclass__ = abc.ABCMeta
    
    @abc.abstractmethod
    def load(self, input):
        """Retrieve data from the input source and return an object."""
        return
    
    @abc.abstractmethod
    def save(self, output, data):
        """Save the data object to the output."""
        return
        
class SubclassImplementation(PluginBase):
    
    def load(self, input):
        return input.read()
    
    def save(self, output, data):
        return output.write(data)
 
if __name__ == '__main__':
    print 'Subclass:', issubclass(SubclassImplementation, PluginBase)
    print 'Instance:', isinstance(SubclassImplementation(), PluginBase)
```


# collections中的抽象类
[`collections.abc`模块](https://docs.python.org/3/library/collections.abc.html#module-collections.abc)定义了几个抽象类。
具体请看[文档](https://docs.python.org/3/library/collections.abc.html#module-collections.abc)。



参考：
https://docs.python.org/3/library/abc.html#module-abc
https://www.python.org/dev/peps/pep-3119/
https://my.oschina.net/u/225373/blog/201304
http://yansu.org/2013/06/09/learn-python-abc-module.html
http://www.cnblogs.com/Security-Darren/p/4094959.html