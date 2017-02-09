title: python学习笔记1 

#  Python学习笔记之语法基础 
Python学习资料：
  * Python学习手册第4版
  * [Python基础教程](http://www.runoob.com/python/python-tutorial.html)
  * http://www.pythondoc.com/

[python官网](https://www.python.org/)
[py2exe](http://www.py2exe.org/):python2.x的windows可执行文件打包器
[cx_freeze](http://cx-freeze.sourceforge.net/)python3.x的windows可执行打包器
pyInstaller：python的linux可执行文件打包器
python的GUI库：tkinter,PMW,wxPython,PyGTK,PyQt等
Python的替代：CPython,Jython
python是一门动态类型语言.
如果你要用计算机做很多工作，最后你会发现有一些任务你更希望用自动化的方式进行处理。比如，你想要在大量的文本文件中执行查找/替换，或者以复杂的方式对大量的图片进行重命名和整理。也许你想要编写一个小型的自定义数据库、一个特殊的 GUI 应用程序或一个简单的小游戏。

如果你是一名专业的软件开发者，可能你必须使用几种 C/C++/JAVA 类库，并且发现通常编写/编译/测试/重新编译的周期是如此漫长。也许你正在为这些类库编写测试用例，但是发现这是一个让人烦躁的工作。又或者你已经完成了一个可以使用扩展语言的程序，但你并不想为此重新设计并实现一套全新的语言。

那么 Python 正是你所需要的语言。
虽然 Python 易于使用，但它却是一门完整的编程语言；与 Shell 脚本或批处理文件相比，它为编写大型程序提供了更多的结构和支持。另一方面，Python 提供了比 C 更多的错误检查，并且作为一门 高级语言，它内置支持高级的数据结构类型，例如：灵活的数组和字典。因其更多的通用数据类型，Python 比 Awk 甚至 Perl 都适用于更多问题领域，至少大多数事情在 Python 中与其他语言同样简单。

Python 允许你将程序分割为不同的模块，以便在其他的 Python 程序中重用。Python 内置提供了大量的标准模块，你可以将其用作程序的基础，或者作为学习 Python 编程的示例。这些模块提供了诸如文件 I/O、系统调用、Socket 支持，甚至类似 Tk 的用户图形界面（GUI）工具包接口。

**python是一门动态类型语言.且变量或参数无需声明**。
第一个python程序test.py
```

#test.py file
#coding=utf-8
import sys
print(sys.platform)
print(2 ** 100)
x='spam'
print(x * 8)

```
关于input函数在python2.6与python3.0的区别：python3.0不会求值，python2.6类似的方法为raw_input
关于中文：**python2.x需要在文件开头加入` #coding=utf-8 `** ; Python3.X 源码文件默认使用utf-8编码，所以可以正常解析中文，无需指定 UTF-8 编码。
#  python基础语法 

**Python标识符**
```

以下划线开头的标识符是有特殊意义的。以单下划线开头（_foo）的代表不能直接访问的类属性，需通过类提供的接口进行访问，不能用"from xxx import *"而导入；
以双下划线开头的（__foo）代表类的私有成员；以双下划线开头和结尾的（__foo__）代表python里特殊方法专用的标识，如__init__（）代表类的构造函数。

```
**行和缩进**
学习Python与其他语言最大的区别就是，Python的代码块不使用大括号（{}）来控制类，函数以及其他逻辑判断。python最具特色的就是用缩进来写模块。
缩进的空白数量是可变的，但是所有代码块语句必须包含相同的缩进空白数量，这个必须严格执行。如下所示：
**多行语句**
Python语句中一般以新行作为为语句的结束符。
但是我们可以使用斜杠（ \）将一行的语句分为多行显示，如下所示：
```

total = item_one + \ 
        item_two + \
        item_three

```
语句中包含[], {} 或 () 括号就不需要使用多行连接符。如下实例：
```

 days = ['Monday', 'Tuesday', 'Wednesday',
        'Thursday', 'Friday']

```
**Python 引号**
```

Python 接收单引号(' )，双引号(" )，三引号(''' """) 来表示字符串，引号的开始与结束必须的相同类型的。其中三引号可以由多行组成

```
**Python注释**
python中单行注释采用 # 开头。
python没有块注释，所以现在推荐的多行注释也是采用的 #
函数返回值注释：
```

def f(ham, eggs) -> "这是函数返回值注释":
  print("Annotations:", f.__annotations__)
  print("Arguments:", ham, eggs)

```
**同一行显示多条语句**
Python可以在同一行中使用多条语句，语句之间使用分号(;)分割，以下是一个简单的实例：
import sys; x = 'foo'; sys.stdout.write(x + '\n')
**多个语句构成代码组**
缩进相同的一组语句构成一个代码块，我们称之代码组。
像if、while、def和class这样的复合语句，首行以关键字开始，以冒号( : )结束，该行之后的一行或多行代码构成代码组。
我们将首行及后面的代码组称为一个子句(clause)。
如下实例：
```

if expression : 
   suite 
elif expression :  
   suite  
else :  
   suite

``` 
**命令行参数**
很多程序可以执行一些操作来查看一些基本信，Python可以使用-h参数查看各参数帮助信息：
$ python -h 
-d	在解析时显示调试信息
-O	生成优化代码 ( .pyo 文件 )
-S	启动时不引入查找Python路径的位置
-v	输出Python版本号
-X	从 1.6版本之后基于内建的异常（仅仅用于字符串）已过时。
-c cmd	执行 Python 脚本，并将运行结果作为 cmd 字符串。
file	在给定的python文件执行python脚本。
**Python有五个标准的数据类型：**
Numbers（数字）
String（字符串）
List（列表）
Tuple（元组）
Dictionary（字典）
**Python数字**
Python支持四种不同的数值类型：
  * int（有符号整型）
  * long（长整型[也可以代表八进制和十六进制]）
  * float（浮点型）
  * complex（复数）
```

var1 = 1
var2 = 10
您也可以使用del语句删除一些对象引用。
del var1[,var2[,var3[....,varN]]]]
您可以通过使用del语句删除单个或多个对象。例如：
del var
del var_a, var_b

```
**Python字符串**
python的字串列表有2种取值顺序:
  * 从左到右索引默认0开始的，最大范围是字符串长度少1
  * 从右到左索引默认-1开始的，最大范围是字符串开头
如果你的实要取得一段子串的话，可以用到变量[头下标:尾下标]，就可以截取相应的字符串，其中下标是从0开始算起，可以是正数或负数，下标可以为空表示取到头或尾。
比如:
s = 'ilovepython'
s[1:5]的结果是love。
` 加号（+）是字符串连接运算符，星号（*）是重复操作。 `
```

str = 'Hello World!'
print str # 输出完整字符串
print str[0] # 输出字符串中的第一个字符
print str[2:5] # 输出字符串中第三个至第五个之间的字符串
print str[2:] # 输出从第三个字符开始的字符串
print str * 2 # 输出字符串两次
print str + "TEST" # 输出连接的字符串

```
**Python列表**
List（列表） 是 Python 中使用最频繁的数据类型。
列表可以完成大多数集合类的数据结构实现。它支持字符，数字，字符串甚至可以包含列表（所谓嵌套）。
列表用[ ]标识。是python最通用的复合数据类型。看这段代码就明白。
列表中的值得分割也可以用到变量**[头下标:尾下标]**，就可以截取相应的列表，从左到右索引默认0开始的，从右到左索引默认-1开始，下标可以为空表示取到头或尾。
**加号（+）是列表连接运算符，星号（*）是重复操作。**如下实例：
```

list = [ 'abcd', 786 , 2.23, 'john', 70.2 ]
tinylist = [123, 'john']

print list # 输出完整列表
print list[0] # 输出列表的第一个元素
print list[1:3] # 输出第二个至第三个的元素 
print list[2:] # 输出从第三个开始至列表末尾的所有元素
print tinylist * 2 # 输出列表两次
print list + tinylist # 打印组合的列表

```
**Python元组**
元组是另一个数据类型，类似于List（列表）。
元组用"()"标识。内部元素用逗号隔开。但是**元素不能二次赋值，相当于只读列表**。
```

tuple = ( 'abcd', 786 , 2.23, 'john', 70.2 )
tinytuple = (123, 'john')

print tuple # 输出完整元组
print tuple[0] # 输出元组的第一个元素
print tuple[1:3] # 输出第二个至第三个的元素 
print tuple[2:] # 输出从第三个开始至列表末尾的所有元素
print tinytuple * 2 # 输出元组两次
print tuple + tinytuple # 打印组合的元组

```
` **以下是元组无效的，因为元组是不允许更新的。而列表是允许更新的**： `
```

#coding=utf-8
#!/usr/bin/python
tuple = ( 'abcd', 786 , 2.23, 'john', 70.2 )
list = [ 'abcd', 786 , 2.23, 'john', 70.2 ]
tuple[2] = 1000 # 元组中是非法应用
list[2] = 1000 # 列表中是合法应用

```
**Python元字典**
字典(dictionary)是除列表以外python之中最灵活的内置数据结构类型。列表是有序的对象结合，字典是无序的对象集合。
两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。
字典用"{ }"标识。字典由索引(key)和它对应的值value组成。
```

dict = {}
dict['one'] = "This is one"
dict[2] = "This is two"

tinydict = {'name': 'john','code':6734, 'dept': 'sales'}


print dict['one'] # 输出键为'one' 的值
print dict[2] # 输出键为 2 的值
print tinydict # 输出完整的字典
print tinydict.keys() # 输出所有键
print tinydict.values() # 输出所有值

```
**Python数据类型转换**
有时候，我们需要对数据内置的类型进行转换，**数据类型的转换，你只需要将数据类型作为函数名即可**。
以下几个内置的函数可以执行数据类型之间的转换。这些函数返回一个新的对象，表示转换的值。
![](/data/dokuwiki/python/pasted/20150824-084722.png)![](/data/dokuwiki/python/pasted/20150824-084743.png)
**Python运算符**
![](/data/dokuwiki/python/pasted/20150824-085341.png)![](/data/dokuwiki/python/pasted/20150824-085008.png)![](/data/dokuwiki/python/pasted/20150824-085033.png)![](/data/dokuwiki/python/pasted/20150824-084839.png)![](/data/dokuwiki/python/pasted/20150824-084918.png)![](/data/dokuwiki/python/pasted/20150824-084944.png)
##  Python 条件语句 
**Python程序语言指定任何非0和非空（null）值为true，0 或者 null为false。**
python：` True,False `
Python 编程中 if 语句用于控制程序的执行，基本形式为：
```

if 判断条件：
    执行语句……
else：
    执行语句……

```
```

if 判断条件1:
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……

```
**由于 python 并不支持 switch 语句**，所以多个条件判断，只能用 elif 来实现，如果判断需要多个条件需同时判断时，可以使用 or （或），表示两个条件有一个成立时判断条件成功；使用 and （与）时，表示只有两个条件同时成立的情况下，判断条件才成功。
当if有多个条件时**可使用` 括号 `来区分判断的先后顺序**，括号中的判断优先执行，此外 and 和 or 的优先级低于>（大于）、<（小于）等判断符号，即大于和小于在没有括号的情况下会比与或要优先判断。
if ( var  == 100 ) : print "Value of expression is 100" 

##  if/else三元表达式 
**A= Y if X else Z.**类似于java的 String a= 1>2?"aaa":"bbb"
```

即if X：
	A=Y
  else:
  	A=Z

```

###  深入条件控制 
while 和 if 语句中使用的条件不仅可以使用比较，而且可以包含任意的操作。
比较操作符 in 和 not in 审核值是否在一个区间之内。操作符 is 和 is not 比较两个对象是否相同；
比较操作可以通过逻辑操作符 and 和 or 组合，比较的结果可以用 not 来取反义。

##  Python 循环语句 
Python提供了for循环和while循环（在Python中没有do..while循环）:
循环类型	描述
while 循环	在给定的判断条件为 true 时执行循环体，否则退出循环体。
for 循环	重复执行语句
嵌套循环	你可以在while循环体中嵌套for循环

循环控制语句
循环控制语句可以更改语句执行的顺序。Python支持以下循环控制语句：

控制语句	描述
break 语句	在语句块执行过程中终止循环，并且跳出整个循环
continue 语句	在语句块执行过程中终止当前循环，跳出该次循环，执行下一次循环。
pass 语句	pass是空语句，是为了保持程序结构的完整性。
```

count = 0
while (count < 9):
   print 'The count is:', count
   count = count + 1

print "Good bye!"

```
```

count = 0
while count < 5:
   print count, " is  less than 5"
   count = count + 1
else:
   print count, " is not less than 5"

```
```

for letter in 'Python':     # First Example
   print 'Current Letter :', letter

fruits = ['banana', 'apple',  'mango']
for fruit in fruits:        # Second Example
   print 'Current fruit :', fruit

print "Good bye!"

```    
```

fruits = ['banana', 'apple',  'mango']
for index in range(len(fruits)):
   print 'Current fruit :', fruits[index]

print "Good bye!"

```
```

for num in range(10,20):  #to iterate between 10 to 20
   for i in range(2,num): #to iterate on the factors of the number
      if num%i == 0:      #to determine the first factor
         j=num/i          #to calculate the second factor
         print '%d equals %d * %d' % (num,i,j)
         break #to move to the next number, the #first FOR
   else:                  # else part of the loop
      print num, 'is a prime number'

```
##  Python数字 
![](/data/dokuwiki/python/pasted/20150824-090806.png)![](/data/dokuwiki/python/pasted/20150824-090818.png)
Python数学常量
常量	描述
pi	数学常量 pi（圆周率，一般以π来表示）
e	数学常量 e，e即自然常数（自然常数）。
##  Python 字符串 
```

var1 = 'Hello World!'
var2 = "Python Programming"

print "var1[0]: ", var1[0]
print "var2[1:5]: ", var2[1:5]
  
var1 = 'Hello World!'
print "Updated String :- ", var1[:6] + 'Python'

```
**Python转义字符**
在需要在字符中使用特殊字符时，python用反斜杠(\)转义字符。如下表：
\(在行尾时)	续行符
\\	反斜杠符号
\000	空
**Python字符串运算符**
下表实例变量a值为字符串"Hello"，b变量值为"Python"：
![](/data/dokuwiki/python/pasted/20150824-133408.png)
**Python字符串格式化**
Python 支持格式化字符串的输出 。尽管这样可能会用到非常复杂的表达式，但最基本的用法是将一个值插入到一个有字符串格式符 ` %s ` 的字符串中。
在 Python 中，字符串格式化使用与 C 中 sprintf 函数一样的语法。
print "My name is %s and weight is %d kg!" % ('Zara', 21) 
python字符串格式化符号:

    符   号	描述
      %c	 格式化字符及其ASCII码
      %s	 格式化字符串
      %d	 格式化整数
      %u	 格式化无符号整型
      %o	 格式化无符号八进制数
      %x	 格式化无符号十六进制数
      %X	 格式化无符号十六进制数（大写）
      %f	 格式化浮点数字，可指定小数点后的精度
      %e	 用科学计数法格式化浮点数
      %E	 作用同%e，用科学计数法格式化浮点数
      %g	 %f和%e的简写
      %G	 %f 和 %E 的简写
      %p	 用十六进制数格式化变量的地址
      
格式化操作符辅助指令:

```

符号	功能
*	定义宽度或者小数点精度
-	用做左对齐
+	在正数前面显示加号( + )
<sp>	在正数前面显示空格
#	在八进制数前面显示零('0')，在十六进制前面显示'0x'或者'0X'(取决于用的是'x'还是'X')
0	显示的数字前面填充'0'而不是默认的空格
%	'%%'输出一个单一的'%'
(var)	映射变量(字典参数)
m.n.	m 是显示的最小总宽度,n 是小数点后的位数(如果可用的话)

```
**Python三引号（triple quotes）**
python中三引号可以将复杂的字符串进行复制:
python三引号允许一个字符串**跨多行**，字符串中**可以包含换行符、制表符以及其他特殊字符**。
三引号的语法是一对连续的单引号或者双引号（通常都是成对的用）。
```

 >>> hi = '''hi 
there'''
>>> hi   # repr()
'hi\nthere'
>>> print hi  # str()
hi 
there  

```
**Unicode 字符串**
Python 中定义一个 Unicode 字符串和定义一个普通字符串一样简单：
u'Hello\u0020World !' 引号前小写的"u"表示这里创建的是一个 Unicode 字符串。如果你想加入一个特殊字符，可以使用 Python 的 Unicode-Escape 编码。

###  字符编码转换 
在最新的Python 3版本中，字符串是以Unicode编码的，也就是说，Python的字符串支持多语言，例如：
```

>>> print('包含中文的str')

```
包含中文的str
对于单个字符的编码，Python提供了ord()函数获取字符的整数表示，chr()函数把编码转换为对应的字符：
```

>>> ord('A')
65
>>> ord('中')
20013
>>> chr(66)
'B'
>>> chr(25991)
'文'
如果知道字符的整数编码，还可以用十六进制这么写str：
>>> '\u4e2d\u6587'
'中文'

```
两种写法完全是等价的。
由于Python的字符串类型是str，在内存中以Unicode表示，一个字符对应若干个字节。如果要在网络上传输，或者保存到磁盘上，就需要把str变为以字节为单位的bytes。
**Python对bytes类型的数据用带b前缀的单引号或双引号表示：**
x = b'ABC'
要注意区分'ABC'和b'ABC'，前者是str，后者虽然内容显示得和前者一样，但bytes的每个字符都只占用一个字节。
` 以Unicode表示的str通过encode()方法可以编码为指定的bytes， `例如：
```

>>> 'ABC'.encode('ascii')
b'ABC'
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
>>> '中文'.encode('ascii')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>

```
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
纯英文的str可以用ASCII编码为bytes，内容是一样的，含有中文的str可以用UTF-8编码为bytes。含有中文的str无法用ASCII编码，因为中文编码的范围超过了ASCII编码的范围，Python会报错。
在bytes中，无法显示为ASCII字符的字节，用\x##显示。
反过来，` 如果我们从网络或磁盘上读取了字节流，那么读到的数据就是bytes。要把bytes变为str，就需要用bytes的decode()方法： `
```

>>> b'ABC'.decode('ascii')
'ABC'
>>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
'中文'
要计算str包含多少个字符，可以用len()函数：

>>> len('ABC')
3
>>> len('中文')
2
len()函数计算的是str的字符数，如果换成bytes，len()函数就计算字节数：

>>> len(b'ABC')
3
>>> len(b'\xe4\xb8\xad\xe6\x96\x87')
6
>>> len('中文'.encode('utf-8'))
6

```
**python的字符串内建函数**
这些方法实现了string模块的大部分方法，如下表所示列出了目前字符串内建支持的方法，所有的方法都包含了对Unicode的支持，有一些甚至是专门用于Unicode的。
![](/data/dokuwiki/python/pasted/20150824-134016.png)![](/data/dokuwiki/python/pasted/20150824-134037.png)![](/data/dokuwiki/python/pasted/20150824-134100.png)![](/data/dokuwiki/python/pasted/20150824-134120.png)![](/data/dokuwiki/python/pasted/20150824-134135.png)

##  Python内建函数列表 
|abs() | divmod() | input()| open()| staticmethod()|
|all() | enumerate() | int() | ord() | str()|
|any() | eval() | isinstance()| pow()| sum()|
|basestring() | execfile() | issubclass() | print() | super()|
|bin() | file() | iter()| property()| tuple()|
|bool() | filter() | len() | range() | type()|
|bytearray() | float()| list() |unichr()|
|callable() | format() | locals() | reduce() | unicode()|
|chr() | frozenset() | long() | reload() | vars()|
|classmethod()| getattr()| map() | repr() | xrange()|
|cmp() | globals()| max()| reversed()| zip()|
|compile() |hasattr() | memoryview()| round() | import()|
|complex() |hash() | min()| set() | apply()|
|delattr() |help()| next()| setattr()| buffer()|
|dict() | hex() |object() |slice() | coerce()|
|dir() | id() |oct() |sorted() |intern()|
##  python入口模块main 
模块拥有名称，Python 解释器本身被认为是顶级模块或主模块。当以**交互的方式运行** Python 时，局部`  __name__ 变量被赋予值 '__main__ `' 。同样地，当从**命令行执行 Python 模块**，而不是将其导入另一个模块时，其 ` __name__ 属性被赋予值 '__main__ `' ，而不是该模块的实际名称。这样，模块可以查看其自身的 __name__ 值来自行确定它们自己正被如何使用，是作为另一个程序的支持，还是作为从命令行执行的主应用程序。因此，下面这条惯用的语句在 Python 模块中是很常见的：

```

if __name__ == '__main__':
    # Do something appropriate here, like calling a
    # main() function defined elsewhere in this module.
    main()
else:
    # Do nothing. This module has been imported by another
    # module that wants to make use of the functions,
    # classes and other useful bits it has defined.

```
##  Python 日期和时间 
Python程序能用很多方式处理日期和时间。转换日期格式是一个常见的例行琐事。Python有一个time and calendar模组可以帮忙。
**什么是Tick？**
时间间隔是以秒为单位的浮点小数。
每个时间戳都以自从1970年1月1日午夜（历元）经过了多长时间来表示。
Python附带的受欢迎的time模块下有很多函数可以转换常见日期格式。如函数` time.time() `用ticks计时单位返回从12:00am, January 1, 1970(epoch) 开始的记录的当前操作系统时间, 如下实例:
```

#!/usr/bin/python
import time;  # This is required to include time module.

ticks = time.time()
print "Number of ticks since 12:00am, January 1, 1970:", ticks

```
**什么是时间元组？**
很多Python函数用一个元组装起来的9组数字处理时间:
![](/data/dokuwiki/python/pasted/20150824-160537.png)![](/data/dokuwiki/python/pasted/20150824-160621.png)
**获取当前时间**
从返回浮点数的时间辍方式向时间元组转换，只要将浮点数传递给如` localtime `之类的函数。
```

#!/usr/bin/python
import time;
localtime = time.localtime(time.time())
print "Local current time :", localtime
以上实例输出结果：
Local current time : time.struct_time(tm_year=2013, tm_mon=7, 
tm_mday=17, tm_hour=21, tm_min=26, tm_sec=3, tm_wday=2, tm_yday=198, tm_isdst=0)

```
**获取格式化的时间**
你可以根据需求选取各种格式，但是最简单的获取可读的时间模式的函数是` asctime(): `
#!/usr/bin/python
import time;
localtime = time.asctime( time.localtime(time.time()) )
print "Local current time :", localtime
以上实例输出结果：
Local curent time : Tue Jan 13 10:17:09 2009
**获取某月日历**
` Calendar `模块有很广泛的方法用来处理年历和月历，例如打印某月的月历：
#!/usr/bin/python
import calendar

cal = calendar.month(2008, 1)
print "Here is the calendar:"
print cal;
###  Time模块 
Time模块包含了以下2个非常重要的属性：
  * time.timezone属性time.timezone是当地时区（未启动夏令时）距离格林威治的偏移秒数（>0，美洲;<=0大部分欧洲，亚洲，非洲）。
  * time.tzname属性time.tzname包含一对根据情况的不同而不同的字符串，分别是带夏令时的本地时区名称，和不带的。

Time模块包含了以下内置函数，既有时间处理相的，也有转换时间格式的：
![](/data/dokuwiki/python/pasted/20150824-161222.png)
###  日历（Calendar）模块 

此模块的函数都是日历相关的，例如打印某月的字符月历。

星期一是默认的每周第一天，星期天是默认的最后一天。更改设置需调用calendar.setfirstweekday()函数。模块包含了以下内置函数：
![](/data/dokuwiki/python/pasted/20150824-161507.png)

其他相关模块和函数
在Python种，其他处理日期和时间的模块还有：
  * datetime模块
  * pytz模块
  * dateutil模块

