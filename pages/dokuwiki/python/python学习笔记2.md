title: python学习笔记2 

#  Python学习笔记之函数、模块 

##  Python函数 
` pass ` 语句什么也不做。它用于那些语法上必须要有什么语句，但程序什么也不做的场合，例如:
```

while True:
	pass  # Busy

```
```

def functionname( parameters ):
   """函数_文档字符串""" 
   function_suite
   return [expression]

```
**按值传递参数和按引用传递参数**
所有参数（自变量）在Python里都是按引用传递。如果你在函数里修改了参数，那么在调用这个函数的函数里，原始的参数也被改变了。例如：
###  参数 
可以参考：http://git.oschina.net/xbynet/StarterLearningPython/blob/master/203.md
以下是调用函数时可使用的正式参数类型：

  * 必备参数，如changeme( mylist );
  * 命名参数，如printme( str = "My string");
  * 缺省参数，调用函数时，缺省参数的值如果没有传入，则被认为是默认值。下例会打印默认的age，如果age没有被传入：
```

#可写函数说明
def printinfo( name, age = 35 ):
   "打印任何传入的字符串"
   print "Name: ", name;
   print "Age ", age;
   return;
 
#调用printinfo函数
printinfo( age=50, name="miki" );
printinfo( name="miki" );

```
**重要警告: ` 默认值只被赋值一次。 `这使得当` 默认值是可变对象时会有所不同 `，比如列表、字典或者大多数类的实例。例如，下面的函数在` 后续调用过程中会累积（前面）传给它的参数: `**

```

def f(a, L=[]):
    L.append(a)
    return L

print(f(1))
print(f(2))
print(f(3))
这将输出:

[1]
[1, 2]
[1, 2, 3]

```
###  不定长参数， 
你可能需要一个函数能处理比当初声明时更多的参数。这些参数叫做不定长参数，和上述2种参数不同，声明时不会命名。基本语法如下：
加了星号（*）的变量名会存放所有未命名的变量参数。选择不多传参数也可。
如：
```

*arg接受一个元组,(注意：元组元素是不可变的，不能对其元素进行赋值)

```
```

# 可写函数说明
def printinfo( arg1, *vartuple ):
   "打印任何传入的参数"
   print "输出: "
   print arg1
   for var in vartuple:
      print var
   return;
 
# 调用printinfo 函数
printinfo( 10 );
printinfo( 70, 60, 50 );

```
###   关键字参数 
函数可以通过 关键字参数 的形式来调用，形如 keyword = value。例如，以下的函数:
```

def parrot(voltage, state='a stiff', action='voom', type='Norwegian Blue'):
    print("-- This parrot wouldn't", action, end=' ')
    print("if you put", voltage, "volts through it.")
    print("-- Lovely plumage, the", type)
    print("-- It's", state, "!")

```
接受一个必选参数 (voltage) 以及三个可选参数 (state, action, 和 type)。
引入一个形如 ```

**name

``` 的参数时，它接收一个字典，**该字典包含了所有未出现在形式参数列表中的关键字参数。** 例如，我们这样定义一个函数:
```

def cheeseshop(kind, *arguments, **keywords):
    print("-- Do you have any", kind, "?")
    print("-- I'm sorry, we're all out of", kind)
    for arg in arguments:
        print(arg)
    print("-" * 40)
    keys = sorted(keywords.keys())
    for kw in keys:
        print(kw, ":", keywords[kw])
它可以像这样调用:

cheeseshop("Limburger", "It's very runny, sir.",
           "It's really very, VERY runny, sir.",
           shopkeeper="Michael Palin",
           client="John Cleese",
           sketch="Cheese Shop Sketch")
当然它会按如下内容打印:

-- Do you have any Limburger ?
-- I'm sorry, we're all out of Limburger
It's very runny, sir.
It's really very, VERY runny, sir.
----------------------------------------
client : John Cleese
shopkeeper : Michael Palin
sketch : Cheese Shop Sketch

```
###   参数列表的分拆 

另有一种相反的情况: 当你要传递的参数已经是一个列表，但要调用的函数却接受分开一个个的参数值。这时候你要把已有的列表拆开来。例如内建函数 range() 需要要独立的 start，stop 参数。
你可以**在调用函数时加一个 * 操作符来自动把参数列表拆开**:


```

>>> list(range(3, 6))            # normal call with separate arguments
[3, 4, 5]
>>> args = [3, 6]
>>> list(range(*args))            # call with arguments unpacked from a list
[3, 4, 5]

```
以同样的方式，**可以使用 * * 操作符分拆关键字参数为字典:**
```

>>> def parrot(voltage, state='a stiff', action='voom'):
...     print("-- This parrot wouldn't", action, end=' ')
...     print("if you put", voltage, "volts through it.", end=' ')
...     print("E's", state, "!")
...
>>> d = {"voltage": "four million", "state": "bleedin' demised", "action": "VOOM"}
>>> parrot(**d)
-- This parrot wouldn't VOOM if you put four million volts through it. E's bleedin' demised !

```
###  函数返回值 
有时候需要返回多个，是以元组形式返回。
```

def fibs(n):
    result = [0,1]
    for i in range(n-2):
        result.append(result[-2] + result[-1])
    return result

if __name__ == "__main__":
    lst = fibs(10)
    print lst
    
>>> def my_fun():
...     return 1,2,3
... 
>>> a = my_fun()
>>> a
(1, 2, 3)

有的函数，没有renturn，一样执行完毕，就算也干了某些活儿吧。事实上，不是没有返回值，也有，只不过是None。比如这样一个函数：

>>> def my_fun():
...     print "I am doing somthin."
... 
>>> a = my_fun()
I am doing somthin.
我们再看看那个变量a，到底是什么
>>> print a
None


```
###  lambda 匿名函数 

用` lambda `关键词能创建小型匿名函数。这种函数得名于省略了用def声明函数的标准步骤。
  * Lambda函数能接收任何数量的参数但只能返回一个表达式的值。
  * 匿名函数不能直接调用print，因为lambda需要一个表达式。
  * lambda函数拥有自己的名字空间，且不能访问自有参数列表之外或全局名字空间里的参数。
  *** lambda表达式可以用于求值，作为函数参数传递，作为函数的返回。**

lambda 关键字，可以创建短小的匿名函数。这里有一个函数返回它的两个参数的和： lambda a, b: a+b出于语法限制，它们只能有一个单独的表达式。
```

通过上面例子，总结一下lambda函数的使用方法：
在lambda后面直接跟变量
变量后面是冒号
冒号后面是表达式，表达式计算结果就是本函数的返回值
为了简明扼要，用一个式子表示是必要的：
lambda arg1, arg2, ...argN : expression using arguments
 
#调用sum函数
print "Value of total : ", sum( 10, 20 )
print "Value of total : ", sum( 20, 20 )

```
再来看一个例子，lambda表达式在函数里头：
```

>>> def make_incrementor(n):
...     return lambda x: x + n //返回的是lambda表达式，而不是一个值
...
>>> f = make_incrementor(42)  //这里返回的f是一个lambda表达式。
>>> f(0)  //这里调用lambda表达式
42
>>> f(1)
43

```
上面的示例使用 lambda 表达式返回一个函数。另一个用途是将一个小函数作为参数传递:
```

>>> pairs = [(1, 'one'), (2, 'two'), (3, 'three'), (4, 'four')]
>>> pairs.sort(key=lambda pair: pair[1])
>>> pairs
[(4, 'four'), (1, 'one'), (3, 'three'), (2, 'two')]

```

###  编写函数的注意事项 
一般情况，你写的函数应该是：
  * 尽量不要使用全局变量。
  * 如果参数是可变类型数据，在函数内，不要修改它。
  * 每个函数的功能和目标要单纯，不要试图一个函数做很多事情。
  * 函数的代码行数尽量少。
  * 函数的独立性越强越好，不要跟其它的外部东西产生关联。
##  map函数 
```

map先看一个例子，还是上面讲述lambda的时候第一个例子，用map也能够实现：
>>> numbers
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]      #把列表中每一项都加3
>>> map(add,numbers)                #add(x)是上面讲述的那个函数，但是这里只引用函数名称即可
[3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
>>> map(lambda x: x+3,numbers)      #用lambda当然可以啦
[3, 4, 5, 6, 7, 8, 9, 10, 11, 12]

```
map()是python的一个内置函数，它的基本样式是：
` map(func,seq) `：func是一个函数，seq是一个序列对象。在执行的时候，序列对象中的每个元素，按照从左到右的顺序，依次被取出来，并塞入到func那个函数里面，并将func的返回值依次存到一个list中。
理解要点：
  * 对iterable中的每个元素，依次应用function的方法（函数）（这本质上就是一个for循环）。
  * 将所有结果返回一个list。
  * 如果参数很多，则对那些参数并行执行function。
例如：
```

>>> lst1 = [1,2,3,4,5]
>>> lst2 = [6,7,8,9,0]
>>> map(lambda x,y: x+y, lst1,lst2)     #将两个列表中的对应项加起来，并返回一个结果列表
[7, 9, 11, 13, 5]

```
请看官注意了，上面这个例子如果用for循环来写，还不是很难，如果扩展一下，下面的例子用for来改写，就要小心了：
```

>>> lst1 = [1,2,3,4,5]
>>> lst2 = [6,7,8,9,0]
>>> lst3 = [7,8,9,2,1]
>>> map(lambda x,y,z: x+y+z, lst1,lst2,lst3)
[14, 17, 20, 15, 6]

```
这才显示出map的简洁优雅。
##  reduce函数 
注意：在python3中，reduce()已经从全局命名空间中移除，放到了functools模块中，如果要是用，需要用` from functools import reduce `引入之。
```

>>> reduce(lambda x,y: x+y,[1,2,3,4,5])
15

```
![](/data/dokuwiki/python/pasted/20150906-163624.png)
还记得map是怎么运算的吗？忘了？看代码：

```

>>> list1 = [1,2,3,4,5,6,7,8,9]
>>> list2 = [9,8,7,6,5,4,3,2,1]
>>> map(lambda x,y: x+y, list1,list2)
[10, 10, 10, 10, 10, 10, 10, 10, 10]

```
看官对比一下，就知道两个的区别了。**原来map是上下运算，reduce是横着逐个元素进行运算。**
##  filter函数 
```

>>> numbers = range(-5,5)
>>> numbers
[-5, -4, -3, -2, -1, 0, 1, 2, 3, 4]

>>> filter(lambda x: x>0, numbers) 
[1, 2, 3, 4]

>>> [x for x in numbers if x>0]     #与上面那句等效
[1, 2, 3, 4]

>>> filter(lambda c: c!='i', 'qiwsir')  #能不能对应上面文档说明那句话呢？
'qwsr'                                  #“If iterable is a string or a tuple, the result also has that type;”

```
至此，介绍了几个函数，这些函数在对程序的性能提高上，并没有显著或者稳定预期，但是，在代码的简洁上，是有目共睹的。有时候是可以用来秀一秀，彰显python的优雅和自己耍酷。如何用、怎么用，看你自己的喜好了。
##  Python 模块 
模块是包括 Python 定义和声明的文件。文件名就是模块名加上 .py 后缀。
**模块也是Python对象**，具有随机的名字属性用来绑定或引用。
模块的模块名（做为一个字符串）可以由全局变量 ```

__name__

``` 得到。
import 语句
**每个模块都有自己私有的符号表，无需担心它与某个用户的全局变量意外冲突。**
出于性能考虑，**每个模块在每个解释器会话中只导入一遍。**因此，如果你**修改了你的模块，需要重启解释器**；或者，如果你就是想交互式的测试这么一个模块，可以用 imp.` reload() 重新加载 `，例如 import imp; imp.reload(modulename)。
From…import 语句
From…import * 语句
**定位模块**
当你导入一个模块，Python解析器对模块位置的搜索顺序是：
  * 当前目录
  * 如果不在当前目录，Python则搜索在 ` sys.path ` 变量中给出的目录列表中查找 sys.path 变量的初始值来自如下：
1、输入脚本的目录（当前目录）。
2、环境变量 PYTHONPATH 表示的目录列表中搜索
  * 如果都找不到，Python会察看默认路径。UNIX下，默认路径一般为/usr/local/lib/python/
模块搜索路径存储在**system模块的sys.path变量**中。变量里包含当前目录，PYTHONPATH和由安装过程决定的默认目录。
set PYTHONPATH=c:\python20\lib;
###  作为脚本来执行模块 
```

python fibo.py <arguments>
模块中的代码会被执行，就像导入它一样，不过此时 __name__ 被设置为 "__main__"。这相当于，如果你在模块后加入如下代码:
if __name__ == "__main__":
    import sys
    fib(int(sys.argv[1]))

```
###  标准模块 
Python 带有一个标准模块库，并发布有独立的文档，名为 Python 库参考手册。有一些模块内置于解释器之中，这些操作的访问接口不是语言内核的一部分，但是已经内置于解释器了。这既是为了提高效率，也是为了给系统调用等操作系统原生访问提供接口。这类模块集合是一个依赖于底层平台的配置选项。
例如，winreg 模块只提供在 Windows 系统上才有。有一个具体的模块值得注意：`  sys ` ，这个模块内置于所有的 Python 解释器。
变量 sys.path 是解释器模块搜索路径的字符串列表。它由环境变量 PYTHONPATH 初始化，如果没有设定 PYTHONPATH ，就由内置的默认值初始化。你可以用标准的字符串操作修改它:

```

>>> import sys
>>> sys.path.append('/ufs/guido/lib/python')

```
###   dir() 函数 

内置函数 dir() 用于按模块名搜索模块定义，它返回一个字符串类型的存储列表:
注意该列表列出了所有类型的名称：变量，模块，函数，等等。
###  包 
名为 A.B 的模块表示了名为 A 的包中名为 B 的子模块。正如同用模块来保存不同的模块架构可以避免全局变量之间的相互冲突，使用圆点模块名保存像 NumPy 或 Python Imaging Library 之类的不同类库架构可以避免模块之间的命名冲突。
你的包可能会是这个样子（通过分级的文件体系来进行分组）:
```

sound/                          Top-level package
      __init__.py               Initialize the sound package
      formats/                  Subpackage for file format conversions
              __init__.py
              wavread.py
              wavwrite.py
              aiffread.py
              aiffwrite.py
              auread.py
              auwrite.py
              ...
      effects/                  Subpackage for sound effects
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...

```
```

为了让 Python 将目录当做内容包，目录中必须包含__init__.py
文件。最简单的情况下，只需要一个空的 __init__.py 文件即可。当然它也可以执行包的初始化代码，或者定义稍后介绍的 __all__ 变量。

```
用户可以每次只导入包里的特定模块，例如:
import sound.effects.echo
这样就导入了 sound.effects.echo 子模块。它必需通过完整的名称来引用:
sound.effects.echo.echofilter(input, output, delay=0.7, atten=4)
导入包时有一个可以选择的方式:
from sound.effects import echo
echo.echofilter(input, output, delay=0.7, atten=4)
###  命名空间和作用域（重点理解） 

变量是拥有匹配对象的名字（标识符）。**命名空间是一个包含了变量名称们（键）和它们各自相应的对象们（值）的字典**。
一个Python表达式可以访问**局部命名空间和全局命名空间**里的变量。如果一个局部变量和一个全局变量重名，则局部变量会覆盖全局变量。
每个函数都有自己的命名空间。类的方法的作用域规则和通常函数的一样。
Python会智能地猜测一个变量是局部的还是全局的，**它假设任何在函数内赋值的变量都是局部的**。
因此，如果要给全局变量在一个函数里赋值，**必须使用global语句。**
**global VarName的表达式会告诉Python， VarName是一个全局变量，这样Python就不会在局部命名空间里寻找这个变量了。**

例如，我们在全局命名空间里定义一个变量money。我们再在函数内给变量money赋值，然后Python会假定money是一个局部变量。然而，我们并没有在访问前声明一个局部变量money，结果就是会出现一个UnboundLocalError的错误。取消` global `语句的注释就能解决这个问题。

```

#coding=utf-8
#!/usr/bin/python
 
Money = 2000
def AddMoney():
   # 想改正代码就取消以下注释:
   # global Money
   Money = Money + 1
 
print Money
AddMoney()
print Money

```
###  dir()函数 

dir()函数一个排好序的字符串列表，内容是一个模块里定义过的名字。
返回的列表容纳了在一个模块里定义的所有模块，变量和函数。如下一个简单的实例：
```

#coding=utf-8
#!/usr/bin/python
 
# 导入内置math模块
import math
 
content = dir(math)
 
print content;
以上实例输出结果：

['__doc__', '__file__', '__name__', 'acos', 'asin', 'atan', 
'atan2', 'ceil', 'cos', 'cosh', 'degrees', 'e', 'exp', 
'fabs', 'floor', 'fmod', 'frexp', 'hypot', 'ldexp', 'log',
'log10', 'modf', 'pi', 'pow', 'radians', 'sin', 'sinh', 
'sqrt', 'tan', 'tanh']

```
在这里，特殊字符串变量__name__指向模块的名字，__file__指向该模块的导入文件名。
###  globals()和locals()函数 

根据调用地方的不同，globals()和locals()函数可被用来返回全局和局部命名空间里的名字。
如果在函数内部调用locals()，返回的是所有能在该函数里访问的命名。
如果在函数内部调用globals()，返回的是所有在该函数里能访问的全局名字。
两个函数的返回类型都是字典。所以名字们能用keys()函数摘取。
###  reload()函数 

当一个模块被导入到一个脚本，模块顶层部分的代码只会被执行一次。
因此，如果你想重新执行模块里顶层部分的代码，可以用reload()函数。该函数会重新导入之前导入过的模块。语法如下：
reload(module_name)
在这里，module_name要直接放模块的名字，而不是一个字符串形式。
##  range() 函数 

如果你需要一个**数值序列**，内置函数 range() 会很方便，它生成一个等差级数链表:
```

>>> for i in range(5):
...     print(i)
...
0
1
2
3
4
  
也可以让 range() 操作从另一个数值开始，或者可以指定一个不同的步进值（甚至是负数，有时这也被称为 “步长”）:
range(0, 10, 3)
   0, 3, 6, 9

range(-10, -100, -30)
  -10, -40, -70

需要迭代链表索引的话，如下所示结合使 用 range() 和 len()
    不过，这种场合可以方便的使用 enumerate()，
>>> a = ['Mary', 'had', 'a', 'little', 'lamb']
>>> for i in range(len(a)):
...     print(i, a[i])
...
0 Mary
1 had
2 a
3 little
4 lamb

```
##  Python中的包 

包是一个分层次的文件目录结构，它定义了一个由模块及子包，和子包下的子包等组成的Python的应用环境。
考虑一个在Phone目录下的pots.py文件。这个文件有如下源代码：
```

#coding=utf-8
#!/usr/bin/python
 
def Pots():
   print "I'm Pots Phone"

```   
同样地，我们有另外两个保存了不同函数的文件：
Phone/Isdn.py 含有函数Isdn()
Phone/G3.py 含有函数G3()
现在，在Phone目录下创建file ```

__init__.py
Phone/__init__.py

```：

当你导入Phone时，为了能够使用所有函数，你需要在__init__.py里使用显式的导入语句，如下：

from Pots import Pots
from Isdn import Isdn
from G3 import G3
当你把这些代码添加到__init__.py之后，导入Phone包的时候这些类就全都是可用的了。

#coding=utf-8
#!/usr/bin/python
 
# Now import your Phone Package.
import Phone
 
Phone.Pots()
Phone.Isdn()
Phone.G3()
以上实例输出结果：

I'm Pots Phone
I'm 3G Phone
I'm ISDN Phone
如上，为了举例，我们只在每个文件里放置了一个函数，但其实你可以放置许多函数。你也可以在这些文件里定义Python的类，然后为这些类建一个包。
