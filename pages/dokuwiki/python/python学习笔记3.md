title: python学习笔记3 

#  Python学习笔记之面向对象与类 
**Python类继承机制允许多重继承**，派生类可以覆盖（override）基类中的任何方法。
**函数别名机制**:多个名称（在多个作用于中）可以绑定在同一个函数上。在其它语言中被称为别名。(如JS)
每个值都是一个对象，因此每个值都有一个 类( class ) （也称为它的 类型( type ) ），它存储为 ```

object.__class__

``` 。

使用class语句来创建一个新类，` class `之后为类的名称并以冒号结尾，如下实例:
```

class ClassName:
   '类的帮助信息'   #类文档字符串
   class_suite  #类体

```
类的帮助信息可以通过```

ClassName.__doc__

```查看。
```

#coding=utf-8
class Employee:
   '所有员工的基类'
   empCount = 0

   def __init__(self, name, salary):
      self.name = name
      self.salary = salary
      Employee.empCount += 1
   
   def displayCount(self):
     print "Total Employee %d" % Employee.empCount

   def displayEmployee(self):
      print "Name : ", self.name,  ", Salary: ", self.salary

```
empCount变量是一个**类变量**，它的值将在这个类的所有实例之间共享。你可以在内部类或外部类使用Employee.empCount访问。
第一种方法```

__init__()

```方法是一种特殊的方法，被称为类的**构造函数**或初始化方法，当创建了这个类的实例时就会调用该方法
**创建实例对象**
要创建一个类的实例，你可以使用类的名称，并通过__init__方法接受参数。
```


"创建 Employee 类的第一个对象"
emp1 = Employee("Zara", 2000)
"创建 Employee 类的第二个对象"
emp2 = Employee("Manni", 5000)

emp1.displayEmployee()
emp2.displayEmployee()
print "Total Employee %d" % Employee.empCount
  
你可以添加，删除，修改类的属性，如下所示：
emp1.age = 7  # 添加一个 'age' 属性
emp1.age = 8  # 修改 'age' 属性
del emp1.age  # 删除 'age' 属性
  
你也可以使用以下函数的方式来访问属性：
hasattr(emp1, 'age')    # 如果存在 'age' 属性返回 True。
getattr(emp1, 'age')    # 返回 'age' 属性的值
setattr(emp1, 'age', 8) # 添加属性 'age' 值为 8
delattr(empl, 'age')    # 删除属性 'age'

```
###  Python内置类属性 

```

__dict__ : 类的属性（包含一个字典，由类的数据属性组成）
__doc__ :类的文档字符串
__name__: 类名
__module__: 类定义所在的模块（类的全名是'__main__.className'，如果类位于一个导入模块mymod中，那么className.__module__ 等于 mymod）
__bases__ : 类的所有父类构成元素（包含了以个由所有父类组成的元组）

```
```

print "Employee.__doc__:", Employee.__doc__
print "Employee.__name__:", Employee.__name__
print "Employee.__module__:", Employee.__module__
print "Employee.__bases__:", Employee.__bases__
print "Employee.__dict__:", Employee.__dict__
执行以上代码输出结果如下：

Employee.__doc__: 所有员工的基类
Employee.__name__: Employee
Employee.__module__: __main__
Employee.__bases__: ()
Employee.__dict__: {'__module__': '__main__', 'displayCount': <function displayCount at 0x10a939c80>, 'empCount': 0, 'displayEmployee': <function displayEmployee at 0x10a93caa0>, '__doc__': '\xe6\x89\x80\xe6\x9c\x89\xe5\x91\x98\xe5\xb7\xa5\xe7\x9a\x84\xe5\x9f\xba\xe7\xb1\xbb', '__init__': <function __init__ at 0x10a939578>}

```

**python对象销毁(垃圾回收)**
同Java语言一样，Python使用了引用计数这一简单技术来追踪内存中的对象。

在Python内部记录着所有使用中的对象各有多少引用。
一个内部跟踪变量，称为一个引用计数器。

当对象被创建时， 就创建了一个引用计数， 当这个对象不再需要时， 也就是说， 这个对象的引用计数变为0 时， 它被垃圾回收。但是回收不是"立即"的， 由解释器在适当的时机，将垃圾对象占用的内存空间回收。

```

析构函数 __del__ ，__del__在对象消逝的时候被调用，当对象不再被使用时，__del__方法运行：

```

##  类的继承 
继承语法: class 派生类名（基类名）：/ /... 基类名写作括号里，基本类是在类定义的时候，在元组之中指明的。
在python中继承中的一些特点：
1：在继承中基类的构造（__init__()方法）不会被自动调用，它需要在其派生类的构造中亲自专门调用。
2：在调用基类的方法时，需要加上基类的类名前缀，且需要带上self参数变量。区别于在类中调用普通函数时并不需要带上self参数
3：Python总是首先查找对应类型的方法，如果它不能在派生类中找到对应的方法，它才开始到基类中逐个查找。（先在本类中查找调用的方法，找不到才去基类中找）。
如果在继承元组中列了一个以上的类，那么它就被称作"` 多重继承 `" 。
```

class Parent:        # 定义父类
   parentAttr = 100
   def __init__(self):
      print "调用父类构造函数"

   def parentMethod(self):
      print '调用父类方法'

   def setAttr(self, attr):
      Parent.parentAttr = attr

   def getAttr(self):
      print "父类属性 :", Parent.parentAttr

class Child(Parent): # 定义子类
   def __init__(self):
      print "调用子类构造方法"

   def childMethod(self):
      print '调用子类方法 child method'

c = Child()          # 实例化子类
c.childMethod()      # 调用子类的方法
c.parentMethod()     # 调用父类方法
c.setAttr(200)       # 再次调用父类的方法
c.getAttr()          # 再次调用父类的方法

```
你可以使用` issubclass()或者isinstance() `方法来检测。
  * issubclass() - 布尔函数判断一个类是另一个类的子类或者子孙类，语法：issubclass(sub,sup)
  * isinstance(obj, Class) 布尔函数如果obj是Class类的实例对象或者是一个Class子类的实例对象则返回true。

方法重写
如果你的父类方法的功能不能满足你的需求，你可以在子类重写你父类的方法：
```

class Parent:        # 定义父类
   def myMethod(self):
      print '调用父类方法'

class Child(Parent): # 定义子类
   def myMethod(self):
      print '调用子类方法'

c = Child()          # 子类实例
c.myMethod()         # 子类调用重写方法

```
基础重载方法
下表列出了一些通用的功能，你可以在自己的类重写：

```

序号	方法, 描述 & 简单的调用
__init__ ( self [,args...] )  构造函数    简单的调用方法: obj = className(args)
__del__( self )  析构方法, 删除一个对象   简单的调用方法 : dell obj
__repr__( self ) 转化为供解释器读取的形式 简单的调用方法 : repr(obj)
__str__( self ) 用于将值转化为适于人阅读的形式 简单的调用方法 : str(obj)
__cmp__ ( self, x ) 对象比较 简单的调用方法 : cmp(obj, x)

```
Python 有两个用于继承的函数：

  * 函数 ` isinstance() ` 用于检查实例类型： isinstance(obj, int) 只有在 obj.__class__ 是 int 或其它从 int 继承的类型
  * 函数 ` issubclass() ` 用于检查类继承： issubclass(bool, int) 为 True，因为 bool 是 int 的子类。
然而， issubclass(float, int) 为 False，因为 float 不是 int 的子类。

###  多重继承 
```

class C1(C2,C3):
	pass

```
###  抽象超类 
抽象超类只能被继承，不能继承其他类
```

class Super():
	@abstractmethod
	def method(self,value):
		pass


```
##  类属性与方法 
###  作用域 

属性可以是只读过或写的。后一种情况下，可以对属性赋值。你可以这样作： modname.the_answer = 42 。可写的属性也可以用`  del ` 语句删除。例如： **del modname.the_answer 会从 modname 对象中删除 the_answer 属性。**
由解释器在最高层调用执行的语句，不管它是从脚本文件中读入还是来自交互式输入，都是 ```

__main__

``` 模块的一部分，所以它们也拥有自己的命名空间（内置命名也同样被包含在一个模块中，它被称作 ` builtins ` ）。
**python引用变量的顺序： 当前作用域局部变量->外层作用域变量->当前模块中的全局变量->python内置变量**

**Python 的一个特别之处在于**：` 如果没有使用 global 语法，其赋值操作总是在最里层的作用域。 `赋值不会复制数据，只是将命名绑定到对象**。删除也是如此：del x 只是从局部作用域的命名空间中删除命名 x 。**事实上，所有引入新命名的操作都作用于局部作用域。特别是 import 语句和函数定将模块名或函数绑定于局部作用域
（可以使用 global 语句将变量引入到全局作用域` ,即在函数内部想要使用全局变量的时候需要使用global来标识，否则python会在局部创建一个新的同名变量 `）。
  * global关键字用来在函数或其他局部作用域中使用全局变量。
  * nonlocal关键字用来在函数或其他作用域中**使用外层(非全局)变量**。
以下是一个示例，演示了如何引用不同作用域和命名空间，以及 global 和 nonlocal 如何影响变量绑定:
```

def scope_test():
    def do_local():
        spam = "local spam"
    def do_nonlocal():
        nonlocal spam
        spam = "nonlocal spam"
    def do_global():
        global spam
        spam = "global spam"
    spam = "test spam"
    do_local()
    print("After local assignment:", spam)
    do_nonlocal()
    print("After nonlocal assignment:", spam)
    do_global()
    print("After global assignment:", spam)

scope_test()
print("In global scope:", spam)
以上示例代码的输出为:

After local assignment: test spam
After nonlocal assignment: nonlocal spam
After global assignment: nonlocal spam
In global scope: global spam

```
###  类变量与实例变量 
```

class Dog:

    kind = 'canine'         # class variable shared by all instances

    def __init__(self, name):
        self.name = name    # instance variable unique to each instance

```
**关于这一点与Java区别很大，所以使用时需要特别谨慎**
有时类似于 Pascal 中“记录（record）”或 C 中“结构（struct）”的数据类型很有用，它将一组已命名的数据项绑定在一起。一个空的类定义可以很好的实现它:

```

class Employee:
    pass

john = Employee() # Create an empty employee record

# Fill the fields of the record
john.name = 'John Doe'
john.dept = 'computer lab'
john.salary = 1000

```
###  属性与方法 
```

class Bag:
    def __init__(self):
        self.data = []
    def add(self, x):
        self.data.append(x)
    def addtwice(self, x):
        self.add(x)
        self.add(x) 

```
###  类的私有属性 
```

__private_attrs：两个下划线开头，声明该属性为私有，不能在类地外部被使用或直接访问。在类内部的方法中使用时 self.__private_attrs。
需要注意的是，在Python中，变量名类似__xxx__的，也就是以双下划线开头，并且以双下划线结尾的，是特殊变量，特殊变量是可以直接访问的，不是private变量，所以，不能用__name__、__score__这样的变量名。
有些时候，你会看到以一个下划线开头的实例变量名，比如_name，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，当你看到这样的变量时，意思就是，“虽然我可以被访问，但是，请把我视为私有变量，不要随意访问”。
双下划线开头的实例变量是不是一定不能从外部访问呢？其实也不是。不能直接访问__name是因为Python解释器对外把__name变量改成了_Student__name，所以，仍然可以通过_Student__name来访问__name变量：
>>> bart._Student__name
'Bart Simpson'
但是强烈建议你不要这么干，因为不同版本的Python解释器可能会把__name改成不同的变量名。
总的来说就是，Python本身没有任何机制阻止你干坏事，一切全靠自觉。

```
**类的方法**
在类地内部，使用def关键字可以为类定义一个方法，与一般函数定义不同，` **类方法必须包含参数self,且为第一个参数** `
**类的私有方法**
```

__private_method：两个下划线开头，声明该方法为私有方法，不能在类地外部调用。在类的内部调用 slef.__private_methods

```
```

class JustCounter:
	__secretCount = 0  # 私有变量
	publicCount = 0    # 公开变量

	def count(self):
		self.__secretCount += 1
		self.publicCount += 1
		print self.__secretCount

counter = JustCounter()
counter.count()
counter.count()
print counter.publicCount
print counter.__secretCount  # 报错，实例不能访问私有变量

```
Python不允许实例化的类访问私有数据，但你可以使用
object._className__attrName访问属性，将如下代码替换以上代码的最后一行代码：print counter._JustCounter__secretCount
##  super函数 
```

class Person:
    def __init__(self):
        self.height = 160

    def about(self, name):
        print "{} is about {}".format(name, self.height)

class Girl(Person):
    def __init__(self):
        super(Girl, self).__init__()
        self.breast = 90

    def about(self, name):
        print "{} is a hot girl, she is about {}, and her breast is {}".format(name, self.height, self.breast)
        super(Girl, self).about(name)#super函数的参数，第一个是当前子类的类名字，第二个是self

if __name__ == "__main__":
    cang = Girl()
    cang.about("canglaoshi")

```
##  类的命名空间 
命名空间因为对象的不同，也有所区别，可以分为如下几种：
  * 内置命名空间(Built-in Namespaces)：Python运行起来，它们就存在了。内置函数的命名空间都属于内置命名空间，所以，我们可以在任何程序中直接运行它们，比如前面的id(),不需要做什么操作，拿过来就直接使用了。
  * 全局命名空间(Module:Global Namespaces)：**每个模块创建它自己所拥有的全局命名空间，不同模块的全局命名空间彼此独立**，不同模块中相同名称的命名空间，也会因为模块的不同而不相互干扰。
  * 本地命名空间(Function&Class: Local Namespaces)：**模块中有函数或者类，每个函数或者类所定义的命名空间就是本地命名空间**。如果函数返回了结果或者抛出异常，则本地命名空间也结束了。

变量的作用域的理解：
```

#!/usr/bin/env python
#coding:utf-8

def outer_foo():
    a = 10
    def inner_foo():
        a = 20
        print "inner_foo,a=",a      #a=20

    inner_foo()
    print "outer_foo,a=",a          #a=10

a = 30
outer_foo()
print "a=",a                #a=30

#运行结果

inner_foo,a= 20
outer_foo,a= 10
a= 30
如果要将某个变量在任何地方都使用，且能够关联，那么在函数内就使用global 声明，其实就是曾经讲过的全局变量。

```
##  Python静态方法和类方法、装饰器（类似于注解） 
```

class StaticMethod:
    @staticmethod
    def foo():
        print "This is static method foo()."

class ClassMethod:
    @classmethod
    def bar(cls):
        print "This is class method bar()."
        print "bar() is part of class:", cls.__name__

if __name__ == "__main__":
    static_foo = StaticMethod()    #实例化
    static_foo.foo()               #实例调用静态方法
    StaticMethod.foo()             #通过类来调用静态方法
    print "********"
    class_bar = ClassMethod()
    class_bar.bar()
    ClassMethod.bar()

```
对于这部分代码，有一处非常特别，那就是包含了“@”符号。在python中：

@staticmethod表示下面的方法是静态方法
@classmethod表示下面的方法是类方法
一个一个来看。
先看静态方法，虽然名为静态方法，但也是方法，所以，依然用def语句来定义。**需要注意的是文件名后面的括号内，没有self**，这和前面定义的类中的方法是不同的，如果没有self，那么也就无法访问实例变量、类和实例的属性了，因为它们都是借助self来传递数据的。
**在看类方法，**同样也具有一般方法的特点，区别也在参数上。类方法的参数也没有self，**但是必须有cls这个参数。**在类方法中，能够方法类属性，但是不能访问实例属性（读者可以自行设计代码检验之）。
简要明确两种方法。下面看调用方法。两种方法都可以通过实例调用，即绑定实例。也可以通过类来调用
如果想要了解Python装饰器的基础，可以看[文章](http://www.pythoncentral.io/python-decorators-overview/)

##  Python装饰器的基础 
装饰器是一个很著名的设计模式，经常被用于有切面需求的场景，较为经典的有插入日志、性能测试、事务处理等。装饰器是解决这类问题的绝佳设计.类似于AOP思想。
也可以参考http://www.cnblogs.com/huxi/archive/2011/03/01/1967600.html
http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386819879946007bbf6ad052463ab18034f0254bf355000
```

def wrapper(fn):
    def inner(a,b):
        return fn(a,b)+1
    return inner
@wrapper
def add(a, b):
    return a+b
print(add(1,2)) #4

```
###  对参数不确定的函数进行装饰 
```

# -*- coding:gbk -*-
'''示例6: 对参数数量不确定的函数进行装饰，
参数用(*args, **kwargs)，自动适应变参和命名参数'''
 
def deco(func):
    def _deco(*args, **kwargs):
        print("before %s called." % func.__name__)
        ret = func(*args, **kwargs)
        print("  after %s called. result: %s" % (func.__name__, ret))
        return ret
    return _deco
 
@deco
def myfunc(a, b):
    print(" myfunc(%s,%s) called." % (a, b))
    return a+b

```
装饰器其实就是一个闭包，把一个函数当做参数然后返回一个替代版函数。只不过是应用了语法糖@xxx的形式。使用标识符@将装饰器应用在函数上，只需要在函数的定义前加上@和装饰器的名称。
###  让装饰器带参数 
```

# -*- coding:gbk -*-
'''示例7: 在示例4的基础上，让装饰器带参数，
和上一示例相比在外层多了一层包装。
装饰函数名实际上应更有意义些'''
 
def deco(arg):
    def _deco(func):
        def __deco():
            print("before %s called [%s]." % (func.__name__, arg))
            func()
            print("  after %s called [%s]." % (func.__name__, arg))
        return __deco
    return _deco
 
@deco("mymodule")
def myfunc():
    print(" myfunc() called.")

```
###  让装饰器带类参数 
```

# -*- coding:gbk -*-
` '示例8: 装饰器带类参数 `'
 
class locker:
    def __init__(self):
        print("locker.__init__() should be not called.")
         
    @staticmethod
    def first():
        print("locker.acquire() called.（这是静态方法）")

 
def deco(cls):
    ` 'cls 必须实现acquire和release静态方法 `'
    def _deco(func):
        def __deco():
            print("before %s called [%s]." % (func.__name__, cls))
            cls.first()
            return func()
        return __deco
    return _deco

```
###  functools模块提供的装饰器 
functools模块提供了两个装饰器。这个模块是Python 2.5后新增的.
` wraps(wrapped[, assigned][, updated]): ` 
这是一个很有用的装饰器。**函数有几个特殊属性比如函数名，` 在被装饰后，上例中的函数名add会变成包装函数的名字inner，如果你希望使用反射，可能会导致意外的结果。这个装饰器可以解决这个问题，它能将装饰过的函数的特殊属性保留。 `**
```

import time
import functools
 
def timeit(func):
    @functools.wraps(func)
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
    return wrapper
 
@timeit
def foo():
    print 'in foo()'
 
foo()
print foo.__name__

```

##  使用装饰器实现的简单日志记录器 
```

def log(func):
    #@functools.wraps(func)
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
@log
def now():
    print(time.strftime("%Y %m %d %H:%M:%S",time.localtime()));

```

输出:
```

>>> now.__name__
"wrapper"     #可以看到now函数名已变成了wrapper,但是如果取消#@functools.wraps(func)注释函数名会恢复为wrapper
>>> now()
call now():
2016 03 10 10:33:52

```