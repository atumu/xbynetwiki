title: python学习笔记4 

#  Python2.x与3.x版本区别 
主要变化
Python 3.0的变化主要在以下几个方面:
` 原生支持UTF-8字符编码 `
##  print()函数 
**print语句没有了，取而代之的是print()函数。** 
八进制数必须写成0o777，原来的形式0777不能用了；二进制必须写成0b111。新增了一个bin()函数用于将一个整数转换成二进制字串。 Python 2.6已经支援这两种语法。
dict.keys(), dict.values​​(), dict.items(), map(), filter(), range(), zip()**不再返回列表，而是迭代器。**
合并int与long类型。
多个模块被改名（根据PEP8）：
旧的名字	新的名字
_winreg	winreg
ConfigParser	configparser
copy_reg	copyreg
Queue	queue
SocketServer	socketserver
repr	reprlib
StringIO模块现在被合并到新的io模组内。 new, md5, gopherlib等模块被删除。 Python 2.6已经支援新的io模组。
httplib, BaseHTTPServer, CGIHTTPServer, SimpleHTTPServer, Cookie, cookielib被合并到http包内。
取消了exec语句，只剩下exec()函数。 Python 2.6已经支援exec()函数。

##  整数除法 
由于人们常常会忽视Python 3在整数除法上的改动（写错了也不会触发Syntax Error），所以在移植代码或在Python 2中执行Python 3的代码时，需要特别注意这个改动。
所以，我还是会在Python 3的脚本中尝试用float(3)/2或 3/2.0代替3/2，以此来避免代码在Python 2环境下可能导致的错误（或与之相反，在Python 2脚本中用from __future__ import division来使用Python 3的除法）。
Python 2
```

print 'Python', python_version()
print '3 / 2 =', 3 / 2
print '3 // 2 =', 3 // 2
print '3 / 2.0 =', 3 / 2.0
print '3 // 2.0 =', 3 // 2.0
Python 2.7.6
3 / 2 = 1
3 // 2 = 1
3 / 2.0 = 1.5
3 // 2.0 = 1.0

```

Python 3
```

print('Python', python_version())
print('3 / 2 =', 3 / 2)
print('3 // 2 =', 3 // 2)
print('3 / 2.0 =', 3 / 2.0)
print('3 // 2.0 =', 3 // 2.0)
Python 3.4.1
3 / 2 = 1.5
3 // 2 = 1
3 / 2.0 = 1.5
3 // 2.0 = 1.0

```
##  Unicode 
Python 2有基于ASCII的str()类型，其可通过单独的unicode()函数转成unicode类型，但没有byte类型。
而在Python 3中，终于有了Unicode（utf-8）字符串，以及两个字节类：bytes和bytearrays。
##  取消xrange函数 
在Python 2.x中，经常会用xrange()创建一个可迭代对象，通常出现在“for循环”或“列表/集合/字典推导式”中。
这种行为与生成器非常相似（如”惰性求值“），但这里的xrange-iterable无尽的，意味着可能在这个xrange上无限迭代。
由于xrange的“惰性求知“特性，如果只需迭代一次（如for循环中），range()通常比xrange()快一些。不过不建议在多次迭代中使用range()，因为range()每次都会在内存中重新生成一个列表。
**在Python 3中，range()的实现方式与xrange()函数相同，所以就不存在专用的xrange()（在Python 3中使用xrange()会触发NameError）。**
##  Python 3中的range对象 
另一个值得一提的是，在Python 3.x中，range有了一个新的__contains__方法。__contains__方法可以有效的加快Python 3.x中整数和布尔型的“查找”速度。
##  触发异常 
Python 2支持新旧两种异常触发语法，而Python 3只接受带括号的的语法（不然会触发SyntaxError）：
Python 2.7
raise IOError, "file error"
Python3
raise IOError("file error")
##  异常处理 
Python 3中的异常处理也发生了一点变化。在Python 3中必须使用“as”关键字。
```

Python 2
try:
    let_us_cause_a_NameError
except NameError, err:
    print err, '--> our error message'

Python 3
try:
    let_us_cause_a_NameError
except NameError as err:
    print(err, '--> our error message')

```

##  next()函数(保留)和.next()方法(已取消) 
由于会经常用到next()（.next()）函数（方法），所以还要提到另一个语法改动（实现方面也做了改动）：在Python 2.7.5中，函数形式和方法形式都可以使用，
**而在Python 3中，只能使用next()函数**（试图调用.next()方法会触发AttributeError）。
```

Python 2
my_generator = (letter for letter in 'abcdefg')
next(my_generator)
my_generator.next()
'b'

Python 3
my_generator = (letter for letter in 'abcdefg')
next(my_generator)
'a'

```
##  For循环变量与全局命名空间泄漏 
好消息是：**在Python 3.x中，for循环中的变量不再会泄漏到全局命名空间中了！**
这是Python 3.x中做的一个改动，在“What’s New In Python 3.0”中有如下描述：
“列表推导不再支持[... for var in item1, item2, ...]这样的语法，使用[... for var in (item1, item2, ...)]代替。还要注意列表推导有不同的语义：现在列表推导更接近list()构造器中的生成器表达式这样的语法糖，特别要注意的是，循环控制变量不会再泄漏到循环周围的空间中了。”
```

Python 2
i = 1
print 'before: i =', i
print 'comprehension: ', [i for i in range(5)]
print 'after: i =', i
输出
before: i = 1
comprehension: [0, 1, 2, 3, 4]
after: i = 4

Python 3
i = 1
print('before: i =', i)
print('comprehension:', [i for i in range(5)])
print('after: i =', i)
输出
before: i = 1
comprehension: [0, 1, 2, 3, 4]
after: i = 1

```
##  比较无序类型 
Python 3中另一个优秀的改动是，如果我们试图比较无序类型，会触发一个TypeError。
##  通过input()解析用户的输入 
幸运的是，**Python 3改进了input()函数**，这样该函数就会**总是将用户的输入存储为str对象**。在Python 2中，为了避免读取非字符串类型会发生的一些危险行为，不得不使用raw_input()代替input()。
```

>>> my_input = input('enter a number: ')
enter a number: 123
>>> type(my_input)
<class 'str'>

```
##  返回可迭代对象，而不是列表 
在xrange一节中可以看到，某些函数和方法在Python中返回的是可迭代对象，而不像在Python 2中返回列表。
由于通常对这些对象只遍历一次，所以这种方式会节省很多内存。然而，如果通过生成器来多次迭代这些对象，效率就不高了。
此时我们的确需要列表对象，**` 可以通过list()函数简单的将可迭代对象转成列表。 `**
```

Python 2
print range(3)
print type(range(3))
Python 2.7.6
[0, 1, 2]
<type 'list'>

Python 3
print(range(3))
print(type(range(3)))
print(list(range(3)))
range(0, 3)
<class 'range'>
[0, 1, 2]

```
下面列出了Python 3中其他不再返回列表的常用函数和方法：
range()
zip()
map()
filter()
字典的.key()方法
字典的.value()方法
字典的.item()方法

参考http://blog.jobbole.com/80006/
