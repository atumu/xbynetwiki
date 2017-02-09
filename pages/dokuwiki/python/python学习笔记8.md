title: python学习笔记8 

#  Python学习笔记之函数进阶 
Python中函数本身也是一个对象。类似于JS。可以被当作参数和返回值进行传递。
函数相关的语句和表达式
  * global 全局变量(模块级)
  * nonlocal非本地变量(在嵌套的函数中，代表封闭的函数变量，即里面函数的外部函数中的本地变量)
  * yield用于生成器
  * lambda匿名函数
  * 作用域与命名空间:本地作用域，封闭作用域，全局作用域(限于单个模块文件)，内置作用域(__builtin__)
  * 变量名解析规则:` LEGB原则 `(Local-Enclosing/nolocal-Global,Builtin).默认赋值操作为本地作用域.
![](/data/dokuwiki/python/pasted/20160312-232657.png)
  * 闭包(Closure):能够记住嵌套作用域的变量值的函数，尽管那个作用域或许已经不存在了。闭合作用域中的执行状态信息被保留了下来。还有其他方式可以保持嵌套作用域的状态(如使用默认参数值来保持嵌套作用域的状态，作为函数属性的状态等)
  * 可变参数传入与可变参数解包，keyword-only参数
![](/data/dokuwiki/python/pasted/20160312-232842.png)
![](/data/dokuwiki/python/pasted/20160312-232900.png)
  * 属性和注解:函数内省,函数属性，函数注解

##  函数内省 
```

def func(a):
	b='spam'
	return b*a
func(8)#'spamspamspamspamspamspamspamspam'

```
```

>>> func.__name__
'func'

>>> dir(func)
['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']

>>> func.__code__
<code object func at 0x000000000337DD20, file "<pyshell#151>", line 1>

>>> dir(func.__code__)
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_kwonlyargcount', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_stacksize', 'co_varnames']

>>> func.__code__.co_varnames
('a', 'b')

>>> func.__code__.co_argcount
1

```
##  函数属性 
可以向函数附加任意的用户自定义属性:
```

>>> func
<function func at 0x00000000033FA6A8>

>>> func.count=0
>>> func.count
0

>>> dir(func)
['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'count']
#属性列表中多了一个count

```
**这个属性相当于函数的静态本地变量。可用于保存一些状态。**

##  函数注释 
这是Python3.0新加的
出现的时候直接附加到了函数对象的__annotation__属性中。
语法：分为参数注释和返回值注释
它的用途虽然不是语法级别的硬性要求，但是顾名思义，它可做为函数额外的注释来用。
```

def func(a,b,c):
	return a+b+c
注解写法.注解值可以是任意的，也可以是类型.注解本身也是一个对象
def func(a:'spam',b:(1,10),c:float) -> int:
	return a+b+c

def func(a,b,c=3.0):
	return a+b+c
注解写法
def func(a:'spam',b:(1,10),c:float=3.0) -> int:
	return a+b+c

func.__annotations__ #得到{'return': <class 'int'>, 'b': (1, 10), 'a': 'spam', 'c': <class 'float'>}

```

##  函数陷阱 
` 注意：Python中不允许函数重写,也就是说不允许出现名字相同的函数(即使参数个数不一样) `

**本地变量是静态检测的：**
例如以下语句是错误的
```

X=99
def show()
	print(X) #这一行报错。因此静态检测的原因，此处被当作本地变量，没有初始化所以报错。
	X=88
show()

```
##  偏函数 
**Python的functools模块提供了很多有用的功能，其中一个就是偏函数（Partial function）。**要注意，这里的偏函数和数学意义上的偏函数不一样。
在介绍函数参数的时候，我们讲到，**通过设定参数的默认值，可以降低函数调用的难度。而偏函数也可以做到这一点。**举例如下：
int()函数可以把字符串转换为整数，当仅传入字符串时，int()函数默认按十进制转换：
```

>>> int('12345')
12345
但int()函数还提供额外的base参数，默认值为10。如果传入base参数，就可以做N进制的转换：

>>> int('12345', base=8)
5349
>>> int('12345', 16)
74565
假设要转换大量的二进制字符串，每次都传入int(x, base=2)非常麻烦，于是，我们想到，可以定义一个int2()的函数，默认把base=2传进去：

def int2(x, base=2):
    return int(x, base)
这样，我们转换二进制就非常方便了：

>>> int2('1000000')
64
>>> int2('1010101')
85

```
**functools.partial就是帮助我们创建一个偏函数的，不需要我们自己定义int2()，可以直接使用下面的代码创建一个新的函数int2：**
```

>>> import functools
>>> int2 = functools.partial(int, base=2)
>>> int2('1000000')
64
>>> int2('1010101')
85

```
**所以，简单总结functools.partial的作用就是，把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。**
注意到上面的新的int2函数，仅仅是把base参数重新设定默认值为2，但也可以在函数调用时传入其他值：
int2('1000000', base=10)
1000000
最后，创建偏函数时，实际上可以接收函数对象、*args和* *kw这3个参数，当传入：
int2 = functools.partial(int, base=2)
实际上固定了int()函数的关键字参数base，也就是：
int2('10010')
相当于：
kw = { 'base': 2 }
int('10010', * *kw)
当传入：
max2 = functools.partial(max, 10)
实际上会把10作为*args的一部分自动加到左边，也就是：
max2(5, 6, 7)
相当于：
args = (10, 5, 6, 7)
max(*args)
结果为10。
小结
当函数的参数个数太多，需要简化时，使用functools.partial可以创建一个新的函数，这个新函数可以固定住原函数的部分参数，从而在调用时更简单。
##  functools模块 
参考http://blog.csdn.net/mldxs/article/details/8610231
functools里有partial，reduce，update_wrapper，wraps
partial即上面介绍的偏函数
wraps我们已经介绍过了。[[python:python学习笔记3&#python装饰器的基础]]

  * functools.reduce和python内置的reduce是一样的，
  * wraps主要是用来包装函数，使被包装含数更像原函数，它是对partial(update_wrapper, ...)的简单包装，
  * partial主要是用来修改函数签名，使一些参数固化，以提供一个更简单的函数供以后调用
  * update_wrapper是wraps的主要功能提供者，它负责考贝原函数的属性，默认是：'__module__', '__name__', '__doc__'， '__dict__'。