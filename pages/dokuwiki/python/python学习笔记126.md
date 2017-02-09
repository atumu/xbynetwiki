title: python学习笔记126 

#  Python学习笔记之subprocess模块执行Shell命令 
os.system() 这个方法如os.system('cat /proc/cpuinfo')**但是这样是无法获得到输出和返回值的**。要获得返回值，
以前有个模块叫做commands模块专门用于调用Linux shell命令，并返回状态和结果.但是Python3中被移除了。**取而代之的是subprocess模块**
**subprocess的目标是启动一个新的进程并与之进行通讯。**

##  subprocess.call* 
` subprocess.call* 会导致父进程等待子进程完成 `

**subprocess.call()**父进程等待子进程完成。返回退出信息(returncode，相当于Linux exit code)
**subprocess.check_call()**父进程等待子进程完成，返回0
检查退出信息，如果returncode不为0，则举出错误subprocess.CalledProcessError，该对象包含有returncode属性，可用try…except…来检查
**subprocess.check_output()**父进程等待子进程完成，**返回子进程向标准输出的输出结果**
检查退出信息，如果returncode不为0，则举出错误subprocess.CalledProcessError，该对象包含有returncode属性和output属性，output属性为标准输出的输出结果，可用try…except…来检查。
这三个函数的使用方法相类似，下面来以subprocess.call()举例说明:
```

>>> import subprocess
>>> retcode = subprocess.call(["ls", "-l"])
#和shell中命令ls -a显示结果一样
>>> print retcode
0

```
将程序名(ls)和所带的参数(-l)一起放在一个表中传递给subprocess.call()
shell默认为False，在Linux下，shell=False时, Popen调用os.execvp()执行args指定的程序；shell=True时，如果args是字符串，Popen直接调用系统的Shell来执行args指定的程序，如果args是一个序列，则args的第一项是定义程序命令字符串，其它项是调用系统Shell时的附加参数。
上面例子也可以写成如下：
复制代码 代码如下:
```

>>> retcode = subprocess.call("ls -l",shell=True)

```
##  subprocess.Popen 
**Popen对象创建后，主程序不会自动等待子进程完成。**
这个模块主要就提供一个类Popen：
```

class subprocess.Popen( args, 
      bufsize=0, 
      executable=None,
      stdin=None,
      stdout=None, 
      stderr=None, 
      preexec_fn=None, 
      close_fds=False, 
      shell=False, 
      cwd=None, 
      env=None, 
      universal_newlines=False, 
      startupinfo=None, 
      creationflags=0)

```
![](/data/dokuwiki/python/pasted/20160318-144506.png)

当初最感到困扰的就是 args 参数。可以是一个字符串，可以是一个列表。
```

subprocess.Popen(["gedit","abc.txt"])
subprocess.Popen("gedit abc.txt")

```
**这两个之中，后者将不会工作。因为如果是一个字符串的话，必须是程序的路径才可以。**(考虑unix的api函数 exec，接受的是字符串列表)
但是下面的可以工作
```

subprocess.Popen("gedit abc.txt", shell=True)
这是因为它相当于
subprocess.Popen(["/bin/sh", "-c", "gedit abc.txt"])

```
都成了sh的参数，就无所谓了
在Windows下，下面的却又是可以工作的
subprocess.Popen(["notepad.exe", "abc.txt"])
subprocess.Popen("notepad.exe abc.txt")
这是由于windows下的api函数CreateProcess接受的是一个字符串。即使是列表形式的参数，也需要先合并成字符串再传递给api函数。
类似上面
subprocess.Popen("notepad.exe abc.txt" shell=True)
等价于
subprocess.Popen("cmd.exe /C "+"notepad.exe abc.txt" shell=True)

` Popen对象创建后，主程序不会自动等待子进程完成。我们必须调用对象的wait()方法，父进程才会等待 (也就是阻塞block) `

###  Popen对象 
该对象提供有不少方法函数可用。而且前面已经用到了wait()/poll()/communicate()
![](/data/dokuwiki/python/pasted/20160318-144914.png)
` 与call不同，Popen对象创建后，主程序不会自动等待子进程完成。我们必须调用对象的wait()方法，父进程才会等待 (也就是阻塞block)： `
  * 1.Popen.poll()：用于检查子进程是否已经结束。设置并返回returncode属性。
  * 2.Popen.wait()：等待子进程结束。设置并返回returncode属性。
  * 3**.Popen.communicate(input=None)：与子进程进行交互。向stdin发送数据，或从stdout和stderr中读取数据。可选参数input指定发送到子进程的参数。Communicate()返回一个元组：(stdoutdata, stderrdata)。**注意：如果希望通过进程的stdin向其发送数据，在创建Popen对象的时候，参数stdin必须被设置为PIPE。同样，如果希望从stdout和stderr获取数据，必须将stdout和stderr设置为PIPE。要注意的是，` communicate()方法会阻塞父进程，直到子进程完成。 `subprocess.PIPE实际上为文本流提供一个缓存区。直到communicate()方法从PIPE中读取出PIPE中的文本.
  * 4.Popen.send_signal(signal)：向子进程发送信号。
  * 5.Popen.terminate()：停止(stop)子进程。在windows平台下，该方法将调用Windows API TerminateProcess（）来结束子进程。
  * 6.Popen.kill()：杀死子进程。
  * 7.Popen.stdin：如果在创建Popen对象是，参数stdin被设置为PIPE，Popen.stdin将返回一个文件对象用于策子进程发送指令。否则返回None。
  * 8.Popen.stdout：如果在创建Popen对象是，参数stdout被设置为PIPE，Popen.stdout将返回一个文件对象用于策子进程发送指令。否则返回None。
  * 9.Popen.stderr：如果在创建Popen对象是，参数stdout被设置为PIPE，Popen.stdout将返回一个文件对象用于策子进程发送指令。否则返回None。
  * 10.Popen.pid：获取子进程的进程ID。
  * 11.Popen.returncode：获取进程的返回值。如果进程还没有结束，返回None。
  * 12.subprocess.call(*popenargs, * *kwargs)：运行命令。该函数将一直等待到子进程运行结束，并返回进程的returncode。文章一开始的例子就演示了call函数。如果子进程不需要进行交互,就可以使用该函数来创建。
  * 13.subprocess.check_call(*popenargs, * *kwargs)：与subprocess.call(*popenargs, * *kwargs)功能一样，只是如果子进程返回的returncode不为0的话，将触发CalledProcessError异常。在异常对象中，包括进程的returncode信息。
##  示列 
```

>>> subprocess.check_output(["echo", "Hello World!"])
b'Hello World!\n'

>>> subprocess.check_output(["echo", "Hello World!"], universal_newlines=True)
'Hello World!\n'

>>> subprocess.check_output(["sed", "-e", "s/foo/bar/"],
...                         input=b"when in the course of fooman events\n")
b'when in the course of barman events\n'

```
```

>>> subprocess.check_call(["ls", "-l"])
0

```
**1）Popen启动新的进程与父进程并行执行，` 默认父进程不等待新进程结束 `。**
```

def TestPopen():
  import subprocess
  p=subprocess.Popen("dir",shell=True)
  for i in range(250) :
    print ("other things")

```
**2）p.wait函数使得父进程等待新创建的进程运行结束，然后再继续父进程的其他任务。且此时可以在p.returncode中得到新进程的返回值。**
```

def TestWait():
  import subprocess
  import datetime
  print (datetime.datetime.now())
  p=subprocess.Popen("sleep 10",shell=True)
  p.wait()
  print (p.returncode)
  print (datetime.datetime.now())

```
**3) p.poll函数可以用来检测新创建的进程是否结束。**
```

def TestPoll():
  import subprocess
  import datetime
  import time
  print (datetime.datetime.now())
  p=subprocess.Popen("sleep 10",shell=True)
  t = 1
  while(t <= 5):
    time.sleep(1)
    p.poll()
    print (p.returncode)
    t+=1
  print (datetime.datetime.now())

```
**4) p.kill或p.terminate用来结束创建的新进程，在windows系统上相当于调用TerminateProcess()，在posix系统上相当于发送信号SIGTERM和SIGKILL。**
```

def TestKillAndTerminate():
    p=subprocess.Popen("notepad.exe")
    t = 1
    while(t <= 5):
      time.sleep(1)
      t +=1
    p.kill()
    #p.terminate()
    print ("new process was killed")

```
**5) p.communicate可以与新进程交互，但是必须要在popen构造时候将管道重定向。**
```

def TestCommunicate():
  import subprocess
  cmd = "dir"
  p=subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
  (stdoutdata, stderrdata) = p.communicate()
  
  if p.returncode != 0:
        print (cmd + "error !")
  #defaultly the return stdoutdata is bytes, need convert to str and utf8
  for r in str(stdoutdata,encoding='utf8' ).split("\n"):
    print (r)
  print (p.returncode)

def TestCommunicate2():
  import subprocess
  cmd = "dir"
  #universal_newlines=True, it means by text way to open stdout and stderr
  p = subprocess.Popen(cmd, shell=True, universal_newlines=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
  curline = p.stdout.readline()
  while(curline != ""):
        print (curline)
        curline = p.stdout.readline()
  p.wait()
  print (p.returncode)

```
**6) call函数可以认为是对popen和wait的分装，直接对call函数传入要执行的命令行，将命令行的退出code返回。**
```

def TestCall():
  retcode = subprocess.call("c:\\test.bat")
  print (retcode)

```
**7）subprocess.getoutput 和 subprocess.getstatusoutput ，基本上等价于subprocess.call函数，但是可以返回output，或者同时返回退出code和output。**
但是可惜的是好像不能在windows平台使用，在windows上有如下错误：'{' is not recognized as an internal or external command, operable program or batch file. 
```

def TestGetOutput():
  outp = subprocess.getoutput("ls -la")
  print (outp)
def TestGetStatusOutput():
  (status, outp) = subprocess.getstatusoutput('ls -la')
  print (status)
  print (outp)

```

参考https://docs.python.org/3.4/library/subprocess.html#module-subprocess
http://www.cnblogs.com/vamei/archive/2012/09/23/2698014.html
http://www.oschina.net/question/234345_52660
http://python.jobbole.com/81517/