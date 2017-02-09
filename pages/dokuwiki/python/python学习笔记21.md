title: python学习笔记21 

#  Python学习笔记之IO、异常 
##  print()与格式化输出 
打印到屏幕print
读取键盘输入raw_input,input:
  * 在Python2.x: raw_input([prompt]) 函数从标准输入读取一个行，并返回一个字符串（去掉结尾的换行符）：
input([prompt]) 函数和raw_input([prompt]) 函数基本可以互换，但是input会假设你的输入是一个有效的Python表达式，并返回运算结果。
  * 在Python3.x：只有input()函数：从标准输入读取一个行，并返回一个字符串（去掉结尾的换行符）
**格式化输出**： ` str.format() `
print()函数打印一行后会加入换行符
将任意值转化为字符串：将它传入 repr() 或 str() 函数。函数 str() 用于将值转化为适于人阅读的形式，而 repr() 转化为供解释器读取的形式
```

 print('{0:2d} {1:3d} {2:4d}'.format(x, x*x, x*x*x))

方法 str.format() 的基本用法如下:

>>> print('We are the {} who say "{}!"'.format('knights', 'Ni'))
We are the knights who say "Ni!"
大括号和其中的字符会被替换成传入 str.format() 的参数。大括号中的数值指明使用传入 str.format() 方法的对象中的哪一个:

>>> print('{0} and {1}'.format('spam', 'eggs'))
spam and eggs
>>> print('{1} and {0}'.format('spam', 'eggs'))
eggs and spam
如果在 str.format() 调用时使用关键字参数，可以通过参数名来引用值:

>>> print('This {food} is {adjective}.'.format(
...       food='spam', adjective='absolutely horrible'))
This spam is absolutely horrible.
  
字段名后允许可选的 ':' 和格式指令。这允许对值的格式化加以更深入的控制。下例将 Pi 转为三位精度。

>>> import math
>>> print('The value of PI is approximately {0:.3f}.'.format(math.pi))
The value of PI is approximately 3.142.
  
也可以用 ‘**’ 标志将这个字典以关键字参数的方式传入:

>>> table = {'Sjoerd': 4127, 'Jack': 4098, 'Dcab': 8637678}
>>> print('Jack: {Jack:d}; Sjoerd: {Sjoerd:d}; Dcab: {Dcab:d}'.format(**table))
Jack: 4098; Sjoerd: 4127; Dcab: 8637678
  
操作符 % 也可以用于字符串格式化。它以类似 sprintf()-style 的方式解析左参数，将右参数应用于此，得到格式化操作生成的字符串，例如:

>>> import math
>>> print('The value of PI is approximately %5.3f.' % math.pi)
The value of PI is approximately 3.142.

```
##  Python 文件I/O 
###  打开和关闭文件 
**open函数**
你必须先用Python内置的open()函数打开一个文件，创建一个file对象，相关的辅助方法才可以调用它进行读写。
语法：file object = open(file_name [, access_mode][, buffering])
各个参数的细节如下：
  * file_name：file_name变量是一个包含了你要访问的文件名称的字符串值。
  * access_mode：access_mode决定了打开文件的模式：只读，写入，追加等。所有可取值见如下的完全列表。这个参数是非强制的，默认文件访问模式为只读(r)。
  * buffering:如果buffering的值被设为0，就不会有寄存。如果buffering的值取1，访问文件时会寄存行。如果将buffering的值设为大于1的整数，表明了这就是的寄存区的缓冲大小。如果取负值，寄存区的缓冲大小则为系统默认

![](/data/dokuwiki/python/pasted/20150824-165519.png)
**File对象的属性**
一个文件被打开后，你有一个file对象，你可以得到有关该文件的各种信息。
以下是和file对象相关的所有属性的列表：
file.closed	返回true如果文件已被关闭，否则返回false。
file.mode	返回被打开文件的访问模式。
file.name	返回文件的名称。
file.softspace	如果用print输出后，必须跟一个空格符，则返回false。否则返回true。
# 打开一个文件
fo = open("foo.txt", "wb")
print "Name of the file: ", fo.name
print "Closed or not : ", fo.closed
print "Opening mode : ", fo.mode
print "Softspace flag : ", fo.softspace
# 关闭打开的文件
fo.close()
**Write()方法**
Write()方法可将任何字符串写入一个打开的文件。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。
Write()方法不在字符串的结尾不添加换行符('\n')：
f.write(string) 方法将 string 的内容写入文件，**并返回写入字符的长度:**

###  用关键字`  with ` 类似于java7 try()
 处理文件对象是个好习惯。它的先进之处在于文件用完后会自动关闭，就算发生异常也没关系。**它是 try-finally 块的简写**:
```

>>> with open('workfile', 'r') as f:
...     read_data = f.read()
>>> f.closed
True

```
# 打开一个文件
fo = open("/tmp/foo.txt", "wb")
fo.write( "Python is a great language.\nYeah its great!!\n");
 
# 关闭打开的文件
fo.close()
**read()方法、readLine(), readLines()**
read（）方法从一个打开的文件中读取一个字符串。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字
如果到了**文件末尾，f.read() 会返回一个空字符串（' '）**:
如果 f.readline() 返回一个空字符串，那就表示到达了文件末尾，

如果你想把文件中的所有行**读到一个列表中**，你也可以使用`  list(f) 或者 f.readlines()。 `

如果文件太大，就不能用read()或者readlines()一次性将全部内容读入内存，可以使用while循环和readlin()来完成这个任务。
此外，还有一个方法：` fileinput `模块

```

>>> import fileinput
>>> for line in fileinput.input("you.md"):
...     print line ,

```
**文件位置**：

` tell() `方法告诉你文件内的**当前位置**；换句话说，下一次的读写会发生在文件开头这么多字节之后：

` seek（offset [,from]） `方法**改变当前文件的位置**。Offset变量表示要移动的字节数。From变量指定开始移动字节的参考位置。
如果from被设为0，这意味着将文件的开头作为移动字节的参考位置。如果设为1，则使用当前的位置作为参考位置。如果它被设为2，那么该文件的末尾将作为参考位置。

```

>>> f.readline()
'This is the first line of the file.\n'
>>> f.readline()
'Second line of the file\n'
>>> f.readline()
>>> for line in f:
...     print(line, end='')
...
This is the first line of the file.
Second line of the file  
# 打开一个文件
fo = open("/tmp/foo.txt", "r+")
str = fo.read(10);
print "Read String is : ", str
 
# 查找当前位置
position = fo.tell();
print "Current file position : ", position
 
# 把指针再次重新定位到文件开头
position = fo.seek(0, 0);
str = fo.read(10);
print "Again read String is : ", str
# 关闭打开的文件
fo.close()

```
##  重命名和删除文件 
Python的` os模块 `提供了帮你执行**文件处理操作**的方法，比如重命名和删除文件。
要使用这个模块，你必须先导入它，然后可以调用相关的各种功能。
` rename() `方法：
rename()方法需要两个参数，当前的文件名和新文件名。
```

import os
 
# 重命名文件test1.txt到test2.txt。
os.rename( "test1.txt", "test2.txt" )

# 删除一个已经存在的文件test2.txt
os.remove("text2.txt")
  
# 创建目录test
os.mkdir("test")

chdir()方法
可以用chdir()方法来改变当前的目录。chdir()方法需要的一个参数是你想设成当前目录的目录名称。
# 将当前目录改为"/home/newdir"
os.chdir("/home/newdir")
  
getcwd()方法：
getcwd()方法显示当前的工作目录。
os.getcwd()

rmdir()方法删除目录，目录名称以参数传递。
在删除这个目录之前，它的所有内容应该先被清除。
# 删除”/tmp/test”目录
os.rmdir( "/tmp/test"  )

```
文件、目录相关的方法
三个重要的方法来源能对Windows和Unix操作系统上的文件及目录进行一个广泛且实用的处理及操控，如下：
  * File 对象方法: file对象提供了操作文件的一系列方法。
  * OS 对象方法: 提供了处理文件及目录的一系列方法。
##  StringIO和BytesIO 
StringIO
**很多时候，数据读写不一定是文件，也可以在内存中读写。**
StringIO顾名思义就是在内存中读写str。
要把str写入StringIO，我们需要先创建一个StringIO，然后，像文件一样写入即可：
```

>>> from io import StringIO
>>> f = StringIO()
>>> f.write('hello')
5
>>> f.write(' ')
1
>>> f.write('world!')
6
>>> print(f.getvalue())
hello world!

```
getvalue()方法用于获得写入后的str。
要读取StringIO，可以用一个str初始化StringIO，然后，像读文件一样读取：
```

>>> from io import StringIO
>>> f = StringIO('Hello!\nHi!\nGoodbye!')
>>> while True:
...     s = f.readline()
...     if s == '':
...         break
...     print(s.strip())
...
Hello!
Hi!
Goodbye!
BytesIO

```
**StringIO操作的只能是str，如果要操作二进制数据，就需要使用BytesIO。**
BytesIO实现了在内存中读写bytes，我们创建一个BytesIO，然后写入一些bytes：
```

>>> from io import BytesIO
>>> f = BytesIO()
>>> f.write('中文'.encode('utf-8'))
6
>>> print(f.getvalue())
b'\xe4\xb8\xad\xe6\x96\x87'

```
**请注意，写入的不是str，而是经过UTF-8编码的bytes。**
和StringIO类似，可以用一个bytes初始化BytesIO，然后，像读文件一样读取：
```

>>> from io import BytesIO
>>> f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
>>> f.read()
b'\xe4\xb8\xad\xe6\x96\x87'

```
小结
**StringIO和BytesIO是在内存中操作str和bytes的方法，使得和读写文件具有一致的接口。**
##  Python 异常处理 
Python 中（至少）有两种错误：语法错误和异常（ syntax errors 和 exceptions ）。
python提供了两个非常重要的功能来处理python程序在运行中出现的异常和错误。你可以使用该功能来调试python程序。

  * 异常处理: 本站Python教程会具体介绍。
  * 断言(Assertions):本站Python教程会具体介绍。

BaseException	所有异常的基类
Exception	常规错误的基类
StandardError	所有的内建标准异常的基类
EnvironmentError	操作系统错误的基类
RuntimeError	一般的运行时错误
IndentationError	缩进错误
。。。
内置异常详情见：https://docs.python.org/3/library/exceptions.html#bltin-exceptions
**异常处理**
捕捉异常可以使用` try/except `语句。
try:
except <名字>：
except <名字>，<数据>:
else:
一个 except 子句可以在括号中列出多个异常的名字，例如:
 except (RuntimeError, TypeError, NameError):
 . . .     pass
**最后一个 except 子句可以省略异常名称，以作为通配符使用**
```

try:
   fh = open("testfile", "w")
   fh.write("This is my test file for exception handling!!")
except IOError:
   print "Error: can\'t find file or read data"
else:
   print "Written content in the file successfully"
   fh.close()
     
使用except而不带任何异常类型语句捕获所有发生的异常
你可以不带任何异常类型使用except，如下实例：

try:
   You do your operations here;
   ......................
except:
   If there is any exception, then execute this block.
   ......................
else:
   If there is no exception then execute this block.
     
使用except而带多种异常类型
你也可以使用相同的except语句来处理多个异常信息，如下所示：

try:
   You do your operations here;
   ......................
except(Exception1[, Exception2[,...ExceptionN]]]):
   If there is any exception from the given exception list, 
   then execute this block.
   ......................
else:
   If there is no exception then execute this block.  

```
###  try-finally 语句 
try-finally 语句无论是否发生异常都将执行最后的代码。
try:
<语句>
finally:
<语句>    #退出try时总会执行
raise
```

try:
   fh = open("testfile", "w")
   fh.write("This is my test file for exception handling!!")
finally:
   print "Error: can\'t find file or read data"

```
**异常的参数**
一个异常可以带上参数，可作为输出的异常信息参数。
你可以通过except语句来捕获异常的参数，如下所示：
```

try:
   You do your operations here;
   ......................
except ExceptionType, Argument:
   You can print value of Argument here...

```
**抛出异常**
我们可以使用` raise `语句自己触发异常
```

def functionName( level ):
   if level < 1:
      raise "Invalid level!", level
      # The code below to this would not be executed
      # if we raise the exception

```
raise [Exception [, args [, traceback]]]
###  用户自定义异常 

通过创建一个新的异常类，程序可以命名它们自己的异常。异常应该是典型的**继承自` Exception `类**，通过直接或间接的方式。
以下为与RuntimeError相关的实例,实例中创建了一个类，基类为RuntimeError，用于在异常触发时输出更多的信息。
在try语句块中，用户自定义的异常后执行except块语句，变量 e 是用于创建Networkerror类的实例。
```

class MyError(Exception):
...     def __init__(self, value):
...         self.value = value
...     def __str__(self):
...         return repr(self.value)
在这个例子中，Exception 默认的 __init__() 被覆盖。新的方式简单的创建 value 属性。这就替换了原来创建 args 属性的方式。
class Networkerror(RuntimeError):
   def __init__(self, arg):
      self.args = arg
        
try:
   raise Networkerror("Bad hostname")
except Networkerror as e: #python2使用except Networkerror , e:
   print(e.args);

```

##  关于sys.exc_info 
