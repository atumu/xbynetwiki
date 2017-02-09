title: python学习笔记 

#  Python学习笔记之Python标准库概览 
官方文档：https://docs.python.org/3/library/index.html#library-index
##  sys 
这是一个跟python解释器关系密切的标准
```

import sys
print sys.__doc__
sys.argv是变量，专门用来向python解释器传递参数，所以名曰“命令行参数”。
sys.exit()这是一个方法，意思是退出当前的程序。
sys.path它可以查找模块所在的目录，以列表的形式显示出来。如果用append()方法，就能够向这个列表增加新的模块目录。
sys.stdin, sys.stdout, sys.stderr

```
##  copy 
```

>>> import copy
>>> copy.__all__
['Error', 'copy', 'deepcopy']
这个模块中常用的就是copy和deepcopy。

```

#  操作系统接口 
` os ` 模块提供了很多与操作系统交互的函数:
```

>>> import os
>>> os.getcwd()      # Return the current working directory
'C:\\Python33'
>>> os.chdir('/server/accesslogs')   # Change current working directory
>>> os.system('mkdir today')   # Run the command mkdir in the system shell
0

```
在使用一些像 os 这样的大型模块时内置的`  dir() 和 help() ` 函数非常有用:

```

>>> import os
>>> dir(os)
<returns a list of all module functions>
>>> help(os)
<returns an extensive manual page created from the module's docstrings>

```
**os.rename()重命名：**
```

>>> import os
>>> os.rename("22201.py", "newtemp.py")

```
**os.remove()就是用来删除文件的,` 不能用来删除目录 `。**
```

 os.remove("/home/qw/Documents/VBS/StarterLearningPython/2code/rd/a.py")

```
**os.listdir：显示目录中的文件**
不过，特别注意它返回的值是列表.
```

>>> os.listdir("/home/qw/Documents/VBS/StarterLearningPython/2code/rd")
['b.py', 'c.py']
>>> files = os.listdir("/home/qw/Documents/VBS/StarterLearningPython/2code/rd")
>>> for f in files:
...     print f
... 
b.py
c.py

```
**os.getcwd, os.chdir：当前工作目录，改变当前工作目录**
**os.makedirs, os.removedirs：创建和删除目录 如果目录不空，就不能用os.removedirs()删除,但是，可以用模块shutil的retree方法替代**
与os.makedirs()类似的还有os.mkdir()。os.removedirs()和os.rmdir()也类似
文件和目录属性os.stat()
操作命令os.system(command)：需要注意的是，` os.system() `是在**当前进程中执行命令**，直到它执行结束。如果需要一个新的进程，可以使用` os.exec `或者os.execvp
os.system()是通过shell执行命令，执行结束后将控制权返回到原来的进程，但是os.exec()及相关的函数，则在执行后不将控制权返回到原继承。
##  webbrowser模块 
可以专门用来打开指定网页。
```

>>> import webbrowser
>>> webbrowser.open("http://www.itdiffer.com")
True

```



##  文件和目录操作 
针对日常的文件和目录管理任务，` shutil ` 模块提供了一个易于使用的高级接口:
```

>>> import shutil
>>> shutil.copyfile('data.db', 'archive.db')
'archive.db'
>>> shutil.move('/build/executables', 'installdir')
'installdir'

```
Path路径分隔符 ` os.pathsep ` 相当于java的Path.separator (windows 为; linux为:)
文件路径分隔符 ` os.sep ` 相当于java的File.separator (windows为\ \ , linux为 /)
###  文件通配符 
*，?
` glob ` 模块提供了一个函数用于从目录通配符搜索中生成文件列表:
```

>>> import glob
>>> glob.glob('*.py')
['primes.py', 'random.py', 'quote.py']

```
##  命令行参数 
通用工具脚本经常调用命令行参数。这些命令行参数以链表形式存储于`  sys 模块的 argv 变 `量。例如在命令行中执行 python demo.py one two three 后可以得到以下输出结果:
```


>>> import sys
>>> print(sys.argv)
['demo.py', 'one', 'two', 'three']

```
getopt 模块使用 Unix getopt() 函处理 sys.argv。更多的复杂命令行处理由 argparse 模块提供。
##   错误输出重定向和程序终止 
` sys 还有 stdin， stdout 和 stderr ` 属性，即使在 stdout 被重定向时，后者也可以用于显示警告和错误信息:
sys.stderr.write('Warning, log file not found starting a new one\n')
Warning, log file not found starting a new one
大多脚本的定向终止都使用 sys.exit()。
##  字符串正则匹配 
` re ` 模块为高级字符串处理提供了正则表达式工具。对于复杂的匹配和处理，正则表达式提供了简洁、优化的解决方案:
```

>>> import re
>>> re.findall(r'\bf[a-z]*', 'which foot or hand fell fastest')
['foot', 'fell', 'fastest']
>>> re.sub(r'(\b[a-z]+) \1', r'\1', 'cat in the the hat')
'cat in the hat'
只需简单的操作时，字符串方法最好用，因为它们易读，又容易调试:

>>> 'tea for too'.replace('too', 'two')
'tea for two'

```
##  数学 
` math ` 模块为浮点运算提供了对底层C函数库的访问:
```

>>> import math
>>> math.cos(math.pi / 4.0)
0.70710678118654757
>>> math.log(1024, 2)
10.0

```
` random ` 提供了生成随机数的工具:
```

>>> import random
>>> random.choice(['apple', 'pear', 'banana'])
'apple'
>>> random.sample(range(100), 10)   # sampling without replacement
[30, 83, 16, 4, 8, 81, 41, 50, 18, 33]
>>> random.random()    # random float
0.17970987693706186
>>> random.randrange(6)    # random integer chosen from range(6)
4

```
SciPy <http://scipy.org> 项目提供了许多数值计算的模块。
##  互联网访问 
有几个模块用于访问互联网以及处理网络通信协议。其中最简单的两个是用于处理从 urls 接收的数据的 ` urllib.request ` 以及用于发送电子邮件的`  smtplib: `

```

>>> from urllib.request import urlopen
>>> for line in urlopen('http://tycho.usno.navy.mil/cgi-bin/timer.pl'):
...     line = line.decode('utf-8')  # Decoding the binary data to text.
...     if 'EST' in line or 'EDT' in line:  # look for Eastern Time
...         print(line)

<BR>Nov. 25, 09:43:32 PM EST

>>> import smtplib
>>> server = smtplib.SMTP('localhost')
>>> server.sendmail('soothsayer@example.org', 'jcaesar@example.org',
... """To: jcaesar@example.org
... From: soothsayer@example.org
...
... Beware the Ides of March.
... """)
>>> server.quit()

```
##  日期和时间 
` calendar `
```

calendar
>>> import calendar
>>> cal = calendar.month(2015, 1)
>>> print cal
    January 2015
Mo Tu We Th Fr Sa Su
          1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30 31
轻而易举得到了2015年1月的日历，并且排列的还那么整齐。这就是calendar模块。读者可以用dir()去查看这个模块下的所有内容。

```
` time `
```

>>> import time
>>> time.time()
1430745298.391026

>>> time.localtime()
time.struct_time(tm_year=2015, tm_mon=5, tm_mday=4, tm_hour=21, tm_min=33, tm_sec=39, tm_wday=0, tm_yday=124, tm_isdst=0)

```
time.time()获得的是当前时间（严格说是时间戳），只不过这个时间对人不友好，它是以1970年1月1日0时0分0秒为计时起点，到当前的时间长度（不考虑闰秒）
 time.localtime()就友好多了
http://git.oschina.net/xbynet/StarterLearningPython/blob/master/224.md
```

>>> time.strftime("%y,%m,%d")
'15,05,05'
>>> time.strftime("%y/%m/%d")
'15/05/05'

```

` datetime ` 模块为日期和时间处理同时提供了简单和复杂的方法。支持日期和时间算法的同时，实现的重点放在更有效的处理和格式化输出。该模块还支持时区处理。

```

>>> # dates are easily constructed and formatted
>>> from datetime import date
>>> now = date.today()
>>> now
datetime.date(2003, 12, 2)
>>> now.strftime("%m-%d-%y. %d %b %Y is a %A on the %d day of %B.")
'12-02-03. 02 Dec 2003 is a Tuesday on the 02 day of December.'

>>> # dates support calendar arithmetic
>>> birthday = date(1964, 7, 31)
>>> age = now - birthday
>>> age.days
14368

```
##  数据压缩 
以下模块直接支持通用的数据打包和压缩格式：zlib， gzip， bz2， lzma， zipfile 以及 tarfile。
```

>>> import zlib
>>> s = b'witch which has which witches wrist watch'
>>> len(s)
41
>>> t = zlib.compress(s)
>>> len(t)
37
>>> zlib.decompress(t)
b'witch which has which witches wrist watch'
>>> zlib.crc32(s)
226805979

```
##  性能度量 
Python 提供了一个时间度量工具
例如，使用元组封装和拆封来交换元素看起来要比使用传统的方法要诱人的多。timeit 证明了后者更快一些:
```

>>> from timeit import Timer
>>> Timer('t=a; a=b; b=t', 'a=1; b=2').timeit()
0.57535828626024577
>>> Timer('a,b = b,a', 'a=1; b=2').timeit()
0.54962537085770791
相对于 timeit 的细粒度，profile 和 pstats 模块提供了针对更大代码块的时间度量工具。

```
##  单元测试 
` unittest ` 模块不像 doctest 模块那么容易使用，不过它可以在一个独立的文件里提供一个更全面的测试集:

```

import unittest

class TestStatisticalFunctions(unittest.TestCase):

    def test_average(self):
        self.assertEqual(average([20, 30, 70]), 40.0)
        self.assertEqual(round(average([1, 5, 7]), 1), 4.3)
        with self.assertRaises(ZeroDivisionError):
            average([])
        with self.assertRaises(TypeError):
            average(20, 30, 70)

unittest.main() # Calling from the command line invokes all tests

```
##  “瑞士军刀” 
**Python 展现了“瑞士军刀”的哲学。**这可以通过它更大的包的高级和健壮的功能来得到最好的展现。列如:
  * ` xmlrpc.client 和 xmlrpc.server ` 模块让远程过程调用变得轻而易举。尽管模块有这样的名字，用户无需拥有 XML 的知识或处理 XML。
  * ` email ` 包是一个管理邮件信息的库，包括MIME和其它基于 RFC2822 的信息文档。
不同于实际发送和接收信息的 ` smtplib 和 poplib ` 模块，email 包包含一个构造或解析复杂消息结构（包括附件）及实现互联网编码和头协议的完整工具集。
  * ` xml.dom 和 xml.sax ` 包为流行的信息交换格式提供了强大的支持。同样，`  csv ` 模块支持在通用数据库格式中直接读写
综合起来，这些模块和包大大简化了 Python 应用程序和其它工具之间的数据交换。
  * 国际化由 ` gettext， locale 和 codecs ` 包支持。 
##   多线程 
` 高级模块 threading ` 如何在主程序运行的同时运行任务:
```


import threading, zipfile

class AsyncZip(threading.Thread):
    def __init__(self, infile, outfile):
        threading.Thread.__init__(self)
        self.infile = infile
        self.outfile = outfile
    def run(self):
        f = zipfile.ZipFile(self.outfile, 'w', zipfile.ZIP_DEFLATED)
        f.write(self.infile)
        f.close()
        print('Finished background zip of:', self.infile)

background = AsyncZip('mydata.txt', 'myarchive.zip')
background.start()
print('The main program continues to run in foreground.')

background.join()    # Wait for the background task to finish
print('Main program waited until background was done.')

```
**多线程应用程序的主要挑战是协调线程，**诸如线程间共享数据或其它资源。为了达到那个目的，线程模块提供了许多**同步化的原生支持**，包括：锁，事件，条件变量和信号灯。
尽管这些工具很强大，微小的设计错误也可能造成难以挽回的故障。
因此，**任务协调的首选方法是把对一个资源的所有访问集中在一个单独的线程中**，然后使用`  queue ` 模块用那个线程服务其他线程的请求。**为内部线程通信和协调而使用 Queue 对象的应用程序更**易于设计，更可读，并且更可靠。
##  日志 
` logging 模块 `提供了完整和灵活的日志系统。它最简单的用法是记录信息并发送到一个**文件或 sys.stderr:**
```

import logging
logging.debug('Debugging information')
logging.info('Informational message')
logging.warning('Warning:config file %s not found', 'server.conf')
logging.error('Error occurred')
logging.critical('Critical error -- shutting down')
输出如下:

WARNING:root:Warning:config file server.conf not found
ERROR:root:Error occurred
CRITICAL:root:Critical error -- shutting down

```
默认情况下捕获信息和调试消息并将输出发送到标准错误流。其它可选的路由信息方式通过** email，数据报文，socket 或者 HTTP Server**。基于**消息属性**，新的过滤器可以选择不同的路由： ` DEBUG， INFO， WARNING， ERROR 和 CRITICAL 。 `
日志系统可以直接在 Python 代码中定制，也可以不经过应用程序直接在一个用户可编辑的配置文件中加载。

##  列表工具 
很多数据结构可能会用到内置列表类型。然而，有时可能需要不同性能代价的实现。

` array 模块 `提供了一个类似列表的 ` array() 对象 `，它仅仅是存储数据，更为紧凑。以下的示例演示了一个存储双字节无符号整数的数组（类型编码 "H" ）而非存储 16 字节 Python 整数对象的普通正规列表:

```

>>> from array import array
>>> a = array('H', [4000, 10, 700, 22222])
>>> sum(a)
26932
>>> a[1:3]
array('H', [10, 700])

```
` collections 模块 `提供了类似列表的 ` deque() 对象 `，它从左边添加（append）和弹出（pop）更快，但是在内部查询更慢。这些对象更适用于队列实现和广度优先的树搜索:

```

>>> from collections import deque
>>> d = deque(["task1", "task2", "task3"])
>>> d.append("task4")
>>> print("Handling", d.popleft())
Handling task1
unsearched = deque([starting_node])
def breadth_first_search(unsearched):
    node = unsearched.popleft()
    for m in gen_moves(node):
        if is_goal(m):
            return m
        unsearched.append(m)

```
除了链表的替代实现，该库还提供了`  bisect ` 这样的模块以操作存储链表:
```

>>> import bisect
>>> scores = [(100, 'perl'), (200, 'tcl'), (400, 'lua'), (500, 'python')]
>>> bisect.insort(scores, (300, 'ruby'))
>>> scores
[(100, 'perl'), (200, 'tcl'), (300, 'ruby'), (400, 'lua'), (500, 'python')]

```
` heapq ` 提供了基于正规链表的堆实现。最小的值总是保持在 0 点。这在希望循环访问最小元素但是不想执行完整堆排序的时候非常有用:
```

>>> from heapq import heapify, heappop, heappush
>>> data = [1, 3, 5, 7, 9, 2, 4, 6, 8, 0]
>>> heapify(data)                      # rearrange the list into heap order
>>> heappush(data, -5)                 # add a new entry
>>> [heappop(data) for i in range(3)]  # fetch the three smallest entries
[-5, 0, 1]

```
##  十进制浮点数算法 

` decimal 模块提 `供了一个 Decimal 数据类型用于浮点数计算。相比内置的二进制浮点数实现 float，这个类型有助于
  * 金融应用和其它需要精确十进制表达的场合，
  * 控制精度，
  * 控制舍入以适应法律或者规定要求，
  * 确保十进制数位精度，或者用户希望计算结果与手算相符的场合。