title: python学习笔记58 

#  Python学习笔记之文件和目录、路径操作 
  * Path路径分隔符 os.pathsep 相当于java的Path.separator (windows 为; linux为:)
  * 文件路径分隔符 os.sep 相当于java的File.separator (windows为\ \ , linux为 /)
2 )、os.getcwd()获取当前路径，这个在Python代码中比较常用。
3 )、os.listdir() 列出当前目录下的所有文件和文件夹。
4 )、os.remove() 方法可以删除指定的文件。
5 )、os.system() 方法用来运行shell命令。
6 )、os.chdir() 改变当前目录，到指定目录中。
  * os.makedirs, os.removedirs：创建和删除目录 如果目录不空，就不能用os.removedirs()删除,但是，可以用模块shutil的retree方法替代
  * os.walk迭代目录
  * os.path.exists
  * os.path.isfile,isdir,islink.realpath
  * os.path.getsize,getmtime
  * os.stat(filename).st_size
  * sys.getfilesystemencoding
  * 
##  操作文件和目录 
操作文件和目录的函数一部分放在os模块中，一部分放在os.path模块中，这一点要注意一下。查看、创建和删除目录可以这么调用：
# 查看当前目录的绝对路径:
```

>>> os.path.abspath('.')
'/Users/michael'

```
# 在某个目录下创建一个新目录，首先把新目录的完整路径表示出来:
```

>>> os.path.join('/Users/michael', 'testdir')

```
'/Users/michael/testdir'
# 然后创建一个目录:
```

>>> os.mkdir('/Users/michael/testdir')

```
# 删掉一个目录:
```

>>> os.rmdir('/Users/michael/testdir')

```
把两个路径合成一个时，不要直接拼字符串，而要通过os.path.join()函数，这样可以正确处理不同操作系统的路径分隔符。在Linux/Unix/Mac下，os.path.join()返回这样的字符串：
part-1/part-2
而Windows下会返回这样的字符串：
part-1\part-2
同样的道理，要拆分路径时，也不要直接去拆字符串，而要通过os.path.split()函数，这样可以把一个路径拆分为两部分，后一部分总是最后级别的目录或文件名：
```

>>> os.path.split('/Users/michael/testdir/file.txt')
('/Users/michael/testdir', 'file.txt')

```
os.path.splitext()可以直接让你得到文件扩展名，很多时候非常方便：
```

>>> os.path.splitext('/path/to/file.txt')
('/path/to/file', '.txt')

os.path.basename('/path/to/file.txt') #输出file.txt
os.path.dirname('/path/to/file.txt')#输出file.txt
os.path.expanduser("~/aaa") #输出/home/xby/aaa

```
这些合并、拆分路径的函数并不要求目录和文件要真实存在，它们只对字符串进行操作。
文件操作使用下面的函数。假定当前目录下有一个test.txt文件：
```

# 对文件重命名:
>>> os.rename('test.txt', 'test.py')
# 删掉文件:
>>> os.remove('test.py')

```
但是复制文件的函数居然在os模块中不存在！原因是复制文件并非由操作系统提供的系统调用。理论上讲，我们通过上一节的读写文件可以完成文件复制，只不过要多写很多代码。
##  shutil模块 
幸运的是shutil模块提供了copyfile()的函数，你还可以在shutil模块中找到很多实用函数，它们可以看做是os模块的补充。
```

>>> import shutil
>>> shutil.copyfile('data.db', 'archive.db')
>>> shutil.copy2('data.db', 'archive.db') #区别copy2可以保留一些属性，如修改时间，创建时间，权限信息等。相当于cp -p src dest
'archive.db'
>>> shutil.move('/build/executables', 'installdir')
'installdir'
shutil.copytree(src,dest) #相当于cp -R src dst

```

最后看看如何利用Python的特性来过滤文件。比如我们要列出当前目录下的所有目录，只需要一行代码：
```

>>> [x for x in os.listdir('.') if os.path.isdir(x)]
['.lein', '.local', '.m2', '.npm', '.ssh', '.Trash', '.vim', 'Applications', 'Desktop', ...]

```
要列出所有的.py文件，也只需一行代码：
```

>>> [x for x in os.listdir('.') if os.path.isfile(x) and os.path.splitext(x)[1]=='.py']
['apis.py', 'config.py', 'models.py', 'pymonitor.py', 'test_db.py', 'urls.py', 'wsgiapp.py']

```
是不是非常简洁？
##  创建和解压归档文件 
```

import shutil
shutil.unpack_archive("python-3.tgz")
shutil.make_archive('py3','zip','python-3') #即打包python-3文件夹为py3.zip
获取支持的格式shutil.get_archive_formats()

```
##  迭代目录 
 for path, dirs, files in os.walk(dir): path表示当前迭代层的路径，dirs表示目录名，files表示文件名。文件绝对路径=os.path.join(path,files[0])
```

#!/usr/bin/env python3.3

import os
import time

def modified_within(top, seconds):
    now = time.time()
    for path, dirs, files in os.walk(top):
        for name in files:
            fullpath = os.path.join(path, name)
            if os.path.exists(fullpath):
                mtime = os.path.getmtime(fullpath)
                if mtime > (now - seconds):
                    print(fullpath)
            

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 3:
        print('Usage: {} dir seconds'.format(sys.argv[0]))
        raise SystemExit(1)
    
    modified_within(sys.argv[1], float(sys.argv[2]))

```

##  文件通配符 
*，?
glob 模块提供了一个函数用于从目录通配符搜索中生成文件列表:
```

>>> import glob
>>> glob.glob('*.py')
['primes.py', 'random.py', 'quote.py']


```
##  创建临时文件和目录 
```

from tempfile import TemporaryFile,TemporaryDirectory
with TemporaryFile('w+') as f: #w+是read-write mode
	f.write('Hello world!\n')
	
	f.seek(0) #ready for read
	data=f.read()

```
##  读取ini配置文件 
```

    ; config.ini
    ; Sample configuration file

    [installation]
    library=%(prefix)s/lib
    include=%(prefix)s/include
    bin=%(prefix)s/bin
    prefix=/usr/local

    # Setting related to debug configuration
    [debug]
    log_errors=true
    show_warnings=False

    [server]
    port: 8080       
    nworkers: 32
    pid-file=/tmp/spam.pid
    root=/www/root
    signature: 
        # # ### =
        Brought to by the Python Cookbook
        # # ### =

```
```

from configparser import ConfigParser
cfg = ConfigParser()
cfg.read('config.ini')
print('sections:', cfg.sections())
print('installation:library', cfg.get('installation','library'))
print('debug:log_errors', cfg.getboolean('debug','log_errors'))
print('server:port', cfg.getint('server','port'))
print('server:nworkers', cfg.getint('server','nworkers'))
print('server:signature', cfg.get('server','signature'))

```
##  脚本日志 
logconfig.ini
```

[loggers]
keys=root
    
[handlers]
keys=defaultHandler
    
[formatters]
keys=defaultFormatter

[logger_root]
level=INFO
handlers=defaultHandler
qualname=root
    
[handler_defaultHandler]
class=FileHandler
formatter=defaultFormatter
args=('app.log', 'a')
    
[formatter_defaultFormatter]
format=%(levelname)s:%(name)s:%(message)s

```
```

import logging
import logging.config

def main():
    # Configure the logging system
    logging.config.fileConfig('logconfig.ini')

    # Variables (to make the calls that follow work)
    hostname = 'www.python.org'
    item = 'spam'
    filename = 'data.csv'
    mode = 'r'

    # Example logging calls (insert into your program)
    logging.critical('Host %s unknown', hostname)
    logging.error("Couldn't find %r", item)
    logging.warning('Feature is deprecated')
    logging.info('Opening file %r, mode=%r', filename, mode)
    logging.debug('Got here')

if __name__ == '__main__':
    main()

```
