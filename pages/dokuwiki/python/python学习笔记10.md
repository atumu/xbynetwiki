title: python学习笔记10 

#  Python学习笔记之类和对象进阶 
` 注意：Python中不允许方法重写,也就是说不允许出现名字相同的方法(即使参数个数不一样) `
##  类信息获取基本操作 
###  type(): 
但是type()函数返回的是什么类型呢？它返回对应的Class类型。如果我们要在if语句中判断，就需要比较两个变量的type类型是否相同：
```

>>> type(123)
<class 'int'>
>>> type('str')
<class 'str'>
>>> type(None)
<type(None) 'NoneType'>

>>> type(123)==type(456)
True
>>> type(123)==int
True
>>> type('abc')==type('123')
True
>>> type('abc')==str
True
>>> type('abc')==type(123)
False

```
判断基本数据类型可以直接写int，str等，**但如果要判断一个对象是否是函数怎么办？可以使用types模块中定义的常量**：
```

>>> import types
>>> def fn():
...     pass
...
>>> type(fn)==types.FunctionType
True
>>> type(abs)==types.BuiltinFunctionType
True
>>> type(lambda x: x)==types.LambdaType
True
>>> type((x for x in range(10)))==types.GeneratorType
True

```
###  使用isinstance() 
对于class的继承关系来说，使用type()就很不方便。我们要判断class的类型，可以使用isinstance()函数。
###  使用dir() 
如果要获得一个对象的所有属性和方法，可以使用dir()函数，它返回一个包含字符串的list，比如，获得一个str对象的所有属性和方法：
仅仅把属性和方法列出来是不够的，**配合` getattr()、setattr()以及hasattr() `，我们可以直接操作一个对象的状态**：
```

>>> class MyObject(object):
...     def __init__(self):
...         self.x = 9
...     def power(self):
...         return self.x * self.x
...
>>> obj = MyObject()
紧接着，可以测试该对象的属性：

>>> hasattr(obj, 'x') # 有属性'x'吗？
True
>>> obj.x
9
>>> hasattr(obj, 'y') # 有属性'y'吗？
False
>>> setattr(obj, 'y', 19) # 设置一个属性'y'
>>> hasattr(obj, 'y') # 有属性'y'吗？
True
>>> getattr(obj, 'y') # 获取属性'y'
19
>>> obj.y # 获取属性'y'
19

如果试图获取不存在的属性，会抛出AttributeError的错误：
>>> getattr(obj, 'z') # 获取属性'z'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'MyObject' object has no attribute 'z'
可以传入一个default参数，如果属性不存在，就返回默认值：
>>> getattr(obj, 'z', 404) # 获取属性'z'，如果不存在，返回默认值404
404

也可以获得对象的方法：

>>> hasattr(obj, 'power') # 有属性'power'吗？
True
>>> getattr(obj, 'power') # 获取属性'power'
<bound method MyObject.power of <__main__.MyObject object at 0x10077a6a0>>
>>> fn = getattr(obj, 'power') # 获取属性'power'并赋值到变量fn
>>> fn # fn指向obj.power
<bound method MyObject.power of <__main__.MyObject object at 0x10077a6a0>>
>>> fn() # 调用fn()与调用obj.power()是一样的
81

```
##  super函数 
参考：http://www.zhihu.com/question/20040039
http://blog.csdn.net/johnsonguo/article/details/585193
Python中既然可以直接通过父类名调用父类方法为什么还会存在super函数？
比如 
```

class Child(Parent):  
    def __init__(self):  
         Parent.__init__(self) #这种方式与super(Child, self).__init__()有区别么？

``` 
super 其实干的是这件事：
```

def super(cls, inst):
	mro = inst.__class__.mro()
	returnmro[mro.index(cls) + 1]

```
两个参数 cls 和 inst 分别做了两件事：
  * 1. inst 负责生成 MRO 的 list
  * 2. 通过 cls 定位当前 MRO 中的 index, 并返回 mro[index + 1]
这两件事才是 super 的实质，一定要记住！
**MRO 全称 Method Resolution Order，它代表了类继承的顺序。**后面详细说。
```

举个例子
class Root(object):
    def __init__(self):
        print("thisis Root")
class B(Root):
    def __init__(self):
        print("enter B")
# print(self) # this willprint <__main__.D object at0x...>
        super(B, self).__init__()
        print("leave B")
class C(Root):
    def __init__(self):
        print("enter C")
        super(C, self).__init__()
        print("leave C")
class D(B,C):
    pass
d= D()
print(d.__class__.__mro__)
输出
enter B
enter C
this is Root
leave C
leave B
(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.Root'>, <class 'object'>)

```
知道了`  super 和父类其实没有实质关联之后 `，我们就不难理解为什么 enter B 下一句是 enter C 而不是 this is
Root（如果认为 super 代表“调用父类的方法”，会想当然的认为下一句应该是this is Root）。流程如下，在 B 的__init__ 函数中：
super(B,self).__init__()
首先，我们获取 self.__class__.__mro__，**注意这里的 self 是 D 的 instance 而不是 B 的**
然后，通过 B 来定位 MRO 中的 index，并找到下一个。显然 B 的下一个是 C。于是，我们调用 C 的 __init__，打
出 enter C。
顺便说一句` 为什么 B 的 __init__ 会被调用：因为 D 没有定义 __init__，所以会在 MRO 中找下一个类 `，去查看它有没有定义 __init__，也就是去调用 B 的 __init__。
其实这一切逻辑还是很清晰的，关键是理解 super 到底做了什么。根据d.__class__.__mro__的顺序来依次调用对应的__init__方法。
于是，MRO 中类的顺序到底是怎么排的呢？Python’s super() considered super! 中已经有很好的解释，我翻译一下：
**在 MRO 中，基类永远出现在派生类后面，如果有多个基类，基类的相对顺序保持不变。**

**super 是用来解决多重继承问题的，直接用类名调用父类方法在使用单继承的时候没问题，但是如果使用多继承，会涉及到查找顺序（MRO）、重复调用（钻石继承）等种种问题。**
总之前人留下的经验就是：保持一致性。要不全部用类名调用父类，要不就全部用 super，不要一半一半。
如果没有复杂的继承结构，super 作用不大。而复杂的继承结构本身就是不良设计。
**对于多重继承的用法，现在比较推崇 Mixin 的方式，也就是**
普通类多重继承只能有一个普通父类和若干个 Mixin 类（保持主干单一）
  * Mixin 类不能继承普通类（避免钻石继承）
  * Mixin 类应该单一职责（参考 Java 的 interface 设计，Mixin 和此极其相似，只不过附带实现而已）
如果按照上述标准，只使用 Mixin 形式的多继承，那么不会有钻石继承带来的重复方法调用，也不会有复杂的查找顺序 —— 此时 super 是可以有无的了，用不用全看个人喜好，只是记得千万别和类名调用的方式混用就好。
###  MixIn 
在设计类的继承关系时，通常，**主线都是单一继承下来的**，例如，Ostrich继承自Bird。**但是，如果需要“混入”额外的功能，通过多重继承就可以实现**，比如，让Ostrich除了继承自Bird外，再同时继承Runnable。**这种设计通常称之为MixIn。**
为了更好地看出继承关系，我们把Runnable和Flyable改为RunnableMixIn和FlyableMixIn。类似的，你还可以定义出肉食动物CarnivorousMixIn和植食动物HerbivoresMixIn，**让某个动物同时拥有好几个MixIn**：
```

class Dog(Mammal, RunnableMixIn, CarnivorousMixIn):
    pass

```
**MixIn的目的就是给一个类增加多个功能，这样，在设计类的时候，我们优先考虑通过多重继承来组合多个MixIn的功能，而不是设计多层次的复杂的继承关系。**
参考：http://www.jianshu.com/p/933a22ac0eb7
把一些基础和单一的功能比如一般希望通过interface/protocol实现的功能放进Mixin模块里面还是不错的选择：
```

class CommonEqualityMixin(object):

    def __eq__(self, other):
        return (isinstance(other, self.__class__)
            and self.__dict__ == other.__dict__)

    def __ne__(self, other):
        return not self.__eq__(other)

class Foo(CommonEqualityMixin):

    def __init__(self, item):
        self.item = item

```
` 其实整个理解下来无非就是通过组合的方式获得更多的功能,有点像C#， java里面的interface，强调“it can”的意思，但相比起来简单多了，不需要显示的约束，而且mixin模块自带实现。在使用的时候一般把mixin的类放在父类的右边似乎也是为了强调这并不是典型的多继承，是一种特殊的多继承，而是在继承了一个基类的基础上，顺带利用多重继承的功能给这个子类添点料，增加一些其他的功能。保证Mixin的类功能单一具体，混入之后，新的类的MRO树其实也会相对很简单，并不会引起混乱。 `
##  枚举 
```

from enum import Enum
Animal = Enum('Animal', 'ant bee cat dog')
或者使用等价形式：
class Animals(Enum):
    ant = 1
    bee = 2
    cat = 3
    dog = 4

```
##  类的钩子 
```

构造函数__init__
打印对象的时候调用__str__
属性获取getattr(对象,属性名)
属性字典__dict__
__bases__ #超类的元组
__class__类型
__name__类名

```
```

class rec:pass
rec.name='blob'
rec.__name__
rec.__dict__.keys()
rec.__class__
rec.__bases__ #超类的元组

```
##  类与字典的关系 
以字典为基础的记录属性
```

rec={}
rec['name']='xby'

```

以类为基础记录属性
```

class rec:pass
rec.name='xby'

```

以实例为基础记录属性
```

class rec:pass
ps1=rec()
ps1.name='aaa'
ps2=rec()
ps2.name='vvv'

ps1.name,ps2.name
('aaa','vvv')

```
##  调用父类的构造函数或方法 
```

class Manager(Person):
	def __init__(self):
		Person.__init__(self)
		print("调用父类构造函数")
	def met(self,value):
		Person.met(self,value)
	def mest(value):
		Person.mest(value)


```
##  Call表达式: __call__ 
如果为一个类定义了__call__方法，Python就会为这个类的实例应用函数调用表达式运行__call__方法。**这样就可以让类实例的外观和用法类似于函数**，同时还能保存状态。
```

class Prod:
	def __init__(self,value):
		self.value=value
	def __call__(self,other):
		return self.value*other
x=Prod(2)
x(3)#输出6. 是不是很神奇，竟然对实例x应用了函数调用。其实内部还是调用了__call__函数。

```
##  析构函数__del__ 
略。
##  方法的绑定与不绑定 
```

class Prod:
	def __init__(self,value):
		self.value=value
        def vvvc(self,other):
		return other*self.value
	def abc(other):
		return other*2

```
其中vvvc是绑定方法，因为其中第一个参数为self
而abc是**无绑定方法**，就变成了**普通的函数**。**此时只能通过类调用它，而不是实例。**

##  __dict__列出实例属性、dir()列出继承属性 
略。
##  所有对象派生自object 
类型派生自object，object派生自type

##  装饰器和元类 
函数装饰器(function decorator)
```

class C:
	@staticmethod
	def meth():
		pass


```
```

class tracer:
	def __init__(self,func):
		self.calls=0
		self.func=func
	def __call__(self,*args): #*args打包
		self.calls+=1
		print('call %s to %s' % (self.calls,self.func.__name__))
		self.func(*args) #*args解包

@tracer
def spam(a,b,c):
  print(a,b,c)

spam(1,2,3) #因为spam函数是通过tracer装饰器执行的，所以当变量名spam调用时，实际上自习的是类中的__call__方法。

```
类装饰器：
```

def count(clazz):
	clazz.num=0
	return clazz
@count
class Span:
	pass
Span()
print(Span.num) #输出0

```

元类是一种类似的基于类的高级工具。它会把一个类对象的创建导向到顶级type类的一个子类。
```

class Meta(type):
	def __new__(meta,classname,supers,classdict):
		...
class C(metaclass=Meta):
	....

```
元类重新定义type类的__new__或__init__方法，以实现对一个新的类对象的创建和初始化的控制。直接效果就像类装饰器一样，是定义了在类创建时自动运行的代码。

##  使用__slots__限制实例的属性 
但是，如果我们想要限制实例的属性怎么办？比如，只允许对Student实例添加name和age属性。
为了达到限制的目的，Python允许在定义class的时候，定义一个特殊的__slots__变量，来限制该class实例能添加的属性：
```

class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称

```
然后，我们试试：
```

>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name'
>>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'

```
由于'score'没有被放到__slots__中，所以不能绑定score属性，试图绑定score将得到AttributeError的错误。
使用__slots__要注意，**__slots__定义的属性仅对当前类实例起作用，对继承的子类是不起作用的：**
除非在子类中也定义__slots__，这样，子类实例允许定义的属性就是自身的__slots__加上父类的__slots__。


##  使用@property 
在绑定属性时，如果我们直接把属性暴露出去，**虽然写起来很简单，但是，没办法检查参数**，导致可以把成绩随便改：
s = Student()
s.score = 9999
这显然不合逻辑。为了限制score的范围，可以通过一个set_score()方法来设置成绩，再通过一个get_score()来获取成绩，这样，在set_score()方法里，就可以检查参数：
```

class Student(object):
    def get_score(self):
         return self._score

    def set_score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value

```
现在，对任意的Student实例进行操作，就不能随心所欲地设置score了：
```

>>> s = Student()
>>> s.set_score(60) # ok!
>>> s.get_score()
60
>>> s.set_score(9999)
Traceback (most recent call last):
  ...
ValueError: score must between 0 ~ 100!

```
但是，上面的调用方法又略显复杂，没有直接用属性这么直接简单。
有没有既能检查参数，又可以用类似属性这样简单的方式来访问类的变量呢？对于追求完美的Python程序员来说，这是必须要做到的！
还记得装饰器（decorator）可以给函数动态加上功能吗？对于类的方法，装饰器一样起作用。**Python内置的@property装饰器就是负责把一个方法变成属性调用的**：
```

class Student(object):
    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
    @score.deleter 
     def score(self):
        del self._score

```
@property的实现比较复杂，我们先考察如何使用**。把一个getter方法变成属性，只需要加上@property就可以了，此时，@property本身又创建了另一个装饰器@score.setter和@score.deleter ，负责把一个setter方法变成属性赋值，和删除**，于是，我们就拥有一个可控的属性操作：
```

>>> s = Student()
>>> s.score = 60 # OK，实际转化为s.set_score(60)
>>> s.score # OK，实际转化为s.get_score()
60
>>> s.score = 9999
Traceback (most recent call last):
  ...
ValueError: score must between 0 ~ 100!

```
注意到这个神奇的@property，我们在对实例属性操作的时候，就知道该属性很可能不是直接暴露的，而是通过getter和setter方法来实现的。
**还可以定义只读属性，只定义getter方法，不定义setter方法就是一个只读属性：**
```

class Student(object):
    @property
    def birth(self):
        return self._birth

    @birth.setter
    def birth(self, value):
        self._birth = value

    @property
    def age(self):
        return 2015 - self._birth

```
**上面的birth是可读写属性，而age就是一个只读属性**，因为age可以根据birth和当前时间计算出来。
小结
**@property广泛应用在类的定义中，可以让调用者写出简短的代码，同时保证对参数进行必要的检查，这样，程序运行时就减少了出错的可能性。**

更多参考http://python.jobbole.com/80955/

##  定制类 
```

覆盖__str__、__slots__、__iter__、__next__、__getitem__、__getattr__、__call__

```
请参考：http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014319098638265527beb24f7840aa97de564ccc7f20f6000