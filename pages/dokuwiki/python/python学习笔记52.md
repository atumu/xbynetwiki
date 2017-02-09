title: python学习笔记52 

#  Python学习笔记之gevent协程与并发库 
官网:http://www.gevent.org/
程序员指南:http://xlambda.com/gevent-tutorial/
gevent是一个基于libev的并发库。它为各种并发和网络相关的任务提供了整洁的API。
gevent是一个使用完全同步编程模型的可扩展的异步I/O框架。

在gevent中用到的主要模式是Greenlet, 它是以C扩展模块形式接入Python的轻量级协程。 Greenlet全部运行在主程序操作系统进程的内部，但它们被协作式地调度。
` 在任何时刻，只有一个协程在运行。 `

在gevent里面最多应用到的就是greenlet，一个轻量级的协程实现。在任何时间点，只有一个greenlet处于运行状态。Greenlet与multiprocessing 和 threading这两个库提供的真正的并行结构的区别在于这两个库会真正的切换进程，POSIX线程是由操作系统来负责调度，并且它们是真正并行的。


##  同步和异步执行 
并发的核心思想在于，大的任务可以分解成一系列的子任务，后者可以被调度成 同时执行或异步执行，而不是一次一个地或者同步地执行。**两个子任务之间的 切换也就是上下文切换。**
**在gevent里面，上下文切换是通过yielding来完成的. 在下面的例子里， 我们有两个上下文，通过调用` gevent.sleep(0) `，它们各自yield向对方。**
```

import gevent
def foo():
    print('Running in foo')
    gevent.sleep(0)
    print('Explicit context switch to foo again')

def bar():
    print('Explicit context to bar')
    gevent.sleep(0)
    print('Implicit context switch back to bar')

gevent.joinall([
    gevent.spawn(foo),
    gevent.spawn(bar),
])

```
Running in foo
Explicit context to bar
Explicit context switch to foo again
Implicit context switch back to bar
下图将控制流形象化，就像在调试器中单步执行整个程序，以说明上下文切换如何发生。
![](/data/dokuwiki/python/pasted/20160318-231850.png)
**当我们在受限于网络或IO的函数中使用gevent，这些函数会被协作式的调度， gevent的真正能力会得到发挥。Gevent处理了所有的细节， 来保证你的网络库会在可能的时候，隐式交出greenlet上下文的执行权。** 这样的一种用法是如何强大，怎么强调都不为过。
下面例子中的select()函数通常是一个在各种文件描述符上轮询的阻塞调用。
```

import time
import gevent
from gevent import select
start = time.time()
tic = lambda: 'at %1.1f seconds' % (time.time() - start)
def gr1():
    # Busy waits for a second, but we don't want to stick around...
    print('Started Polling: %s' % tic())
    select.select([], [], [], 2)
    print('Ended Polling: %s' % tic())
def gr2():
    # Busy waits for a second, but we don't want to stick around...
    print('Started Polling: %s' % tic())
    select.select([], [], [], 2)
    print('Ended Polling: %s' % tic())
def gr3():
    print("Hey lets do some stuff while the greenlets poll, %s" % tic())
    gevent.sleep(1)
gevent.joinall([
    gevent.spawn(gr1),
    gevent.spawn(gr2),
    gevent.spawn(gr3),
])

```
Started Polling: at 0.0 seconds
Started Polling: at 0.0 seconds
Hey lets do some stuff while the greenlets poll, at 0.0 seconds
Ended Polling: at 2.0 seconds
Ended Polling: at 2.0 seconds
```

import gevent
import random

def task(pid):
    """
    Some non-deterministic task
    """
    print("start",pid)
    gevent.sleep(random.randint(0,2)*0.001)
    print('Task %s done' % pid)

def synchronous():
    for i in range(1,10):
        task(i)

def asynchronous():
    threads = [gevent.spawn(task, i) for i in range(10)]
    gevent.joinall(threads)

print('Synchronous:')
synchronous()

print('Asynchronous:')
asynchronous()


```
上例中，在同步的部分，所有的task都同步的执行， 结果当每个task在执行时主流程被阻塞(主流程的执行暂时停住)。
程序的重要部分是**将task函数封装到Greenlet内部线程的gevent.spawn。 初始化的greenlet列表存放在数组threads中，此数组被传给gevent.joinall 函数，后者阻塞当前流程，并执行所有给定的greenlet。执行流程只会在 所有greenlet执行完后才会继续向下走。**
事实上，同步部分的最大运行时间，即是每个task停0.002秒，结果整个 队列要停0.02秒。而异步部分的最大运行时间大致为0.002秒，因为没有任何一个task会 阻塞其它task的执行。
一个更常见的应用场景，如异步地向服务器取数据，取数据操作的执行时间 依赖于发起取数据请求时远端服务器的负载，各个请求的执行时间会有差别。
```

import gevent.monkey
gevent.monkey.patch_socket()

import gevent
import requests
import json
def fetch(pid):
    response = requests.get('http://www.baidu.com')
    #result = response.read()
    #json_result = json.loads(result)
    #datetime = json_result['datetime']

    print('Process %s:%s' % (pid,response.text[:10]))
    #return json_result['datetime']

def synchronous():
    for i in range(1,10):
        fetch(i)

def asynchronous():
    threads = []
    for i in range(1,10):
        threads.append(gevent.spawn(fetch, i))
    gevent.joinall(threads)

print('Synchronous:')
synchronous()

print('Asynchronous:')
asynchronous()

```
` 此处最重要的是gevent.monkey.patch_socket()这一行，因为它能运行时动态修改socket库以支持协作式。 `
##  确定性 
就像之前所提到的，**greenlet具有确定性。在相同配置相同输入的情况下，它们总是 会产生相同的输出。**
下面就有例子，我们在**multiprocessing的pool之间执行一系列的 任务，与在gevent的pool之间执行作比较**。
```

import time
def echo(i):
    time.sleep(0.001)
    return i
# Non Deterministic Process Pool
from multiprocessing.pool import Pool
p = Pool(10)
run1 = [a for a in p.imap_unordered(echo, xrange(10))]
run2 = [a for a in p.imap_unordered(echo, xrange(10))]
run3 = [a for a in p.imap_unordered(echo, xrange(10))]
run4 = [a for a in p.imap_unordered(echo, xrange(10))]

print(run1 == run2 == run3 == run4)

# Deterministic Gevent Pool

from gevent.pool import Pool

p = Pool(10)
run1 = [a for a in p.imap_unordered(echo, xrange(10))]
run2 = [a for a in p.imap_unordered(echo, xrange(10))]
run3 = [a for a in p.imap_unordered(echo, xrange(10))]
run4 = [a for a in p.imap_unordered(echo, xrange(10))]

print(run1 == run2 == run3 == run4)

```
输出
False
True
涉及并发长期存在的问题就是**竞争条件(race condition)**。简单来说， 当两个并发线程/进程都依赖于某个共享资源同时都尝试去修改它的时候， 就会出现竞争条件。这会导致资源修改的结果状态依赖于时间和执行顺序。 这是个问题，我们一般会做很多努力尝试避免竞争条件， 因为它会导致整个程序行为变得不确定
最好的办法是始终避免所有全局的状态。全局状态和导入时(import-time)副作用总是会 反咬你一口！

##  创建Greenlets 
gevent对Greenlet初始化提供了一些封装，最常用的使用模板之一有
```

import gevent
from gevent import Greenlet
def foo(message, n):
    gevent.sleep(n)
    print(message)

# Initialize a new Greenlet instance running the named function foo
thread1 = Greenlet.spawn(foo, "Hello", 1)

# Wrapper for creating and running a new Greenlet from the named function foo, with the passed arguments
thread2 = gevent.spawn(foo, "I live!", 2)

# Lambda expressions
thread3 = gevent.spawn(lambda x: (x+1), 2)

threads = [thread1, thread2, thread3]

# Block until all threads complete.
gevent.joinall(threads)

```
Hello
I live!
**除使用基本的Greenlet类之外，你也可以子类化Greenlet类，重载它的_run方法。**
```

import gevent
from gevent import Greenlet
class MyGreenlet(Greenlet):

    def __init__(self, message, n):
        Greenlet.__init__(self)
        self.message = message
        self.n = n

    def _run(self):
        print(self.message)
        gevent.sleep(self.n)

g = MyGreenlet("Hi there!", 3)
g.start()
g.join()

```
Hi there!
##  Greenlet状态 
就像任何其他成段代码，Greenlet也可能以不同的方式运行失败。 Greenlet可能未能成功抛出异常，不能停止运行，或消耗了太多的系统资源。
**一个greenlet的状态通常是一个依赖于时间的参数。在greenlet中有一些标志， 让你可以监视它的线程内部状态：**
  * started -- Boolean, 指示此Greenlet是否已经启动
  * ready() -- Boolean, 指示此Greenlet是否已经停止
  * successful() -- Boolean, 指示此Greenlet是否已经停止而且没抛异常
  * value -- 任意值, 此Greenlet代码返回的值
  * exception -- 异常, 此Greenlet内抛出的未捕获异常
```

import gevent
def win():
    return 'You win!'
def fail():
    raise Exception('You fail at failing.')
winner = gevent.spawn(win)
loser = gevent.spawn(fail)
print(winner.started) # True
print(loser.started)  # True
# Exceptions raised in the Greenlet, stay inside the Greenlet.
try:
    gevent.joinall([winner, loser])
except Exception as e:
    print('This will never be reached')
print(winner.value) # 'You win!'
print(loser.value)  # None
print(winner.ready()) # True
print(loser.ready())  # True
print(winner.successful()) # True
print(loser.successful())  # False

# The exception raised in fail, will not propogate outside the
# greenlet. A stack trace will be printed to stdout but it
# will not unwind the stack of the parent.
print(loser.exception)

```
##  程序停止 
**当主程序(main program)收到一个SIGQUIT信号时，不能成功做yield操作的 Greenlet可能会令意外地挂起程序的执行。这导致了所谓的僵尸进程，** 它需要在Python解释器之外被kill掉。
对此，**一个通用的处理模式就是在主程序中监听SIGQUIT信号，在程序退出 调用gevent.shutdown。**
```

import gevent
import signal

def run_forever():
    gevent.sleep(1000)

if __name__ == '__main__':
    gevent.signal(signal.SIGQUIT, gevent.shutdown)
    thread = gevent.spawn(run_forever)
    thread.join()

```
##  超时 
超时是一种对一块代码或一个Greenlet的运行时间的约束。
```

import gevent
from gevent import Timeout
seconds = 10
timeout = Timeout(seconds)
timeout.start()

def wait():
    gevent.sleep(10)

try:
    gevent.spawn(wait).join()
except Timeout:
    print('Could not complete')

```

**超时类也可以用在上下文管理器(context manager)中, 也就是with语句内。**
```

import gevent
from gevent import Timeout
time_to_wait = 5 # seconds
class TooLong(Exception):
    pass
with Timeout(time_to_wait, TooLong):
    gevent.sleep(10)

```
另外，**对各种Greenlet和数据结构相关的调用，gevent也提供了超时参数。** 例如：
```

import gevent
from gevent import Timeout
def wait():
    gevent.sleep(2)
timer = Timeout(1).start()
thread1 = gevent.spawn(wait)
try:
    thread1.join(timeout=timer)
except Timeout:
    print('Thread 1 timed out')


```
##  猴子补丁(Monkey patching) 
我们现在来到gevent的死角了. 在此之前，我已经避免提到猴子补丁(monkey patching) 以尝试使gevent这个强大的协程模型变得生动有趣，但现在到了讨论猴子补丁的黑色艺术 的时候了。你之前可能注意到我们提到了` monkey.patch_socket()这个命令，这个 纯粹副作用命令是用来改变标准socket库的。 `
```

import socket
print(socket.socket)
print("After monkey patch")
from gevent import monkey
monkey.patch_socket()
print(socket.socket)

import select
print(select.select)
monkey.patch_select()
print("After monkey patch")
print(select.select)

```
输出：
class 'socket.socket'
After monkey patch
class 'gevent.socket.socket'

built-in function select
After monkey patch
function select at 0x1924de8

**Python的运行环境允许我们在运行时修改大部分的对象，包括模块，类甚至函数。** 这是个一般说来令人惊奇的坏主意，因为它创造了“隐式的副作用”，如果出现问题 它很多时候是极难调试的。虽然如此，在极端情况下当一个库需要修改Python本身 的基础行为的时候，猴子补丁就派上用场了。**在这种情况下，gevent能够 修改标准库里面大部分的阻塞式系统调用，包括socket、ssl、threading和 select等模块，而变为协作式运行。**
例如，Redis的python绑定一般使用常规的tcp socket来与redis-server实例通信。 通过简单地调用gevent.monkey.patch_all()，可以使得redis的绑定协作式的调度 请求，与gevent栈的其它部分一起工作。
这让我们可以将一般不能与gevent共同工作的库结合起来，而不用写哪怕一行代码。 虽然猴子补丁仍然是邪恶的(evil)，但在这种情况下它是“有用的邪恶(useful evil)”。

