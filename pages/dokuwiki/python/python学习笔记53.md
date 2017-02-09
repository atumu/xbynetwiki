title: python学习笔记53 

#  Python学习笔记之gevent协程与并发库进阶 
##  事件 
事件(event)是一个在Greenlet之间异步通信的形式。
```

import gevent
from gevent.event import Event

evt = Event()
def setter():
    ` 'After 3 seconds, wake all threads waiting on the value of evt `'
    print('A: Hey wait for me, I have to do something')
    gevent.sleep(3)
    print("Ok, I'm done")
    evt.set()

def waiter():
    ` 'After 3 seconds the get call will unblock `'
    print("I'll wait for you")
    evt.wait()  # blocking
    print("It's about time")

def main():
    gevent.joinall([
        gevent.spawn(setter),
        gevent.spawn(waiter),
        gevent.spawn(waiter),
        gevent.spawn(waiter),
        gevent.spawn(waiter),
        gevent.spawn(waiter)
    ])

if __name__ == '__main__': main()

```
**事件对象的一个扩展是AsyncResult，它允许你在唤醒调用上附加一个值。 它有时也被称作是future或defered，因为它持有一个指向将来任意时间可设置 为任何值的引用。**
```

import gevent
from gevent.event import AsyncResult
a = AsyncResult()

def setter():
    """
    After 3 seconds set the result of a.
    """
    gevent.sleep(3)
    a.set('Hello!')

def waiter():
    """
    After 3 seconds the get call will unblock after the setter
    puts a value into the AsyncResult.
    """
    print(a.get())

gevent.joinall([
    gevent.spawn(setter),
    gevent.spawn(waiter),
])

```
##  队列 
队列是一个排序的数据集合，它有常见的put / get操作， 但是它是以在Greenlet之间可以安全操作的方式来实现的。
举例来说，如果一个Greenlet从队列中取出一项，此项就不会被 同时执行的其它Greenlet再取到了。
```

import gevent
from gevent.queue import Queue

tasks = Queue()

def worker(n):
    while not tasks.empty():
        task = tasks.get()
        print('Worker %s got task %s' % (n, task))
        gevent.sleep(0)

    print('Quitting time!')

def boss():
    for i in xrange(1,25):
        tasks.put_nowait(i)

gevent.spawn(boss).join()

gevent.joinall([
    gevent.spawn(worker, 'steve'),
    gevent.spawn(worker, 'john'),
    gevent.spawn(worker, 'nancy'),
])

```
**如果需要，队列也可以阻塞在put或get操作上。
put和get操作都有非阻塞的版本，put_nowait和get_nowait不会阻塞， 然而在操作不能完成时抛出gevent.queue.Empty或gevent.queue.Full异常。**
在下面例子中，我们让boss与多个worker同时运行，并限制了queue不能放入多于3个元素。 这个限制意味着，直到queue有空余空间之间，put操作会被阻塞。相反地，如果队列中 没有元素，get操作会被阻塞。它同时带一个timeout参数，允许在超时时间内如果 队列没有元素无法完成操作就抛出gevent.queue.Empty异常。

```

import gevent
from gevent.queue import Queue, Empty

tasks = Queue(maxsize=3)

def worker(n):
    try:
        while True:
            task = tasks.get(timeout=1) # decrements queue size by 1
            print('Worker %s got task %s' % (n, task))
            gevent.sleep(0)
    except Empty:
        print('Quitting time!')

def boss():
    """
    Boss will wait to hand out work until a individual worker is
    free since the maxsize of the task queue is 3.
    """

    for i in xrange(1,10):
        tasks.put(i)
    print('Assigned all work in iteration 1')

    for i in xrange(10,20):
        tasks.put(i)
    print('Assigned all work in iteration 2')

gevent.joinall([
    gevent.spawn(boss),
    gevent.spawn(worker, 'steve'),
    gevent.spawn(worker, 'john'),
    gevent.spawn(worker, 'bob'),
])

```
##  组和池 
**组(group)是一个运行中greenlet的集合，集合中的greenlet会被共同管理和调度。** 
```

import gevent
from gevent.pool import Group
def talk(msg):
    for i in xrange(3):
        print(msg)

g1 = gevent.spawn(talk, 'bar')
g2 = gevent.spawn(talk, 'foo')
g3 = gevent.spawn(talk, 'fizz')

group = Group()
group.add(g1)
group.add(g2)
group.join()

group.add(g3)
group.join()

```
在管理异步任务的分组上它是非常有用的。
就像上面所说，Group也以不同的方式为分组greenlet/分发工作和收集它们的结果也提供了API。
```

import gevent
from gevent import getcurrent
from gevent.pool import Group
group = Group()
def hello_from(n):
    print('Size of group %s' % len(group))
    print('Hello from Greenlet %s' % id(getcurrent()))
    
group.map(hello_from, xrange(3))

def intensive(n):
    gevent.sleep(3 - n)
    return 'task', n

print('Ordered')

ogroup = Group()
for i in ogroup.imap(intensive, xrange(3)):
    print(i)
    
print('Unordered')
igroup = Group()
for i in igroup.imap_unordered(intensive, xrange(3)):
    print(i)

```
**池(pool)是一个为处理数量变化并且需要限制并发的greenlet而设计的结构。 在需要并行地做很多受限于网络和IO的任务时常常需要用到它。**
```

import gevent
from gevent.pool import Pool

pool = Pool(2)

def hello_from(n):
    print('Size of pool %s' % len(pool))

pool.map(hello_from, xrange(3))

```
当构造gevent驱动的服务时，经常会将围绕一个池结构的整个服务作为中心。 一个例子就是在各个socket上轮询的类。
```

from gevent.pool import Pool
class SocketPool(object):

    def __init__(self):
        self.pool = Pool(1000)
        self.pool.start()

    def listen(self, socket):
        while True:
            socket.recv()

    def add_handler(self, socket):
        if self.pool.full():
            raise Exception("At maximum pool size")
        else:
            self.pool.spawn(self.listen, socket)

    def shutdown(self):
        self.pool.kill()

```
        
##  锁和信号量 
**信号量是一个允许greenlet相互合作，限制并发访问或运行的低层次的同步原语。** ` 信号量有两个方法，acquire和release。 `在信号量是否已经被 acquire或release，和拥有资源的数量之间不同，被称为此信号量的范围 (the bound of the semaphore)。如果一个信号量的范围已经降低到0，它会 阻塞acquire操作直到另一个已经获得信号量的greenlet作出释放。
```

from gevent import sleep
from gevent.pool import Pool
from gevent.coros import BoundedSemaphore

sem = BoundedSemaphore(2)

def worker1(n):
    sem.acquire()
    print('Worker %i acquired semaphore' % n)
    sleep(0)
    sem.release()
    print('Worker %i released semaphore' % n)

def worker2(n):
    with sem:
        print('Worker %i acquired semaphore' % n)
        sleep(0)
    print('Worker %i released semaphore' % n)

pool = Pool()
pool.map(worker1, xrange(0,2))
pool.map(worker2, xrange(3,6))

```
**范围为1的信号量也称为锁(lock)。它向单个greenlet提供了互斥访问。 信号量和锁常常用来保证资源只在程序上下文被单次使用。**
##  协程局部变量 
**Gevent也允许你指定局部于greenlet上下文的数据。 在内部，它被实现为以greenlet的getcurrent()为键， 在一个私有命名空间寻址的全局查找。**
```

import gevent,threading
from gevent.local import local
stash = local()
def f1():
    stash.x = 1
    print(stash.x)
    print(threading.current_thread)

def f2():
    stash.y = 2
    print(stash.y)
    print(threading.current_thread)
    try:
        stash.x
    except AttributeError:
        print("x is not local to f2")

g1 = gevent.spawn(f1)
g2 = gevent.spawn(f2)

gevent.joinall([g1, g2])

```
输出：
1
<function current_thread at 0x00000000026771E0>
2
<function current_thread at 0x00000000026771E0>
x is not local to f2
` 我们可以看到他们仍然是在一个线程中运行，但是处于不同的协程。可见gevent.local用于创建协程本地变量。 `

##  子进程 
自gevent 1.0起，gevent.subprocess，一个Python subprocess模块 的修补版本已经添加。它支持协作式的等待子进程。
```

import gevent
from gevent.subprocess import Popen, PIPE
def cron():
    while True:
        print("cron")
        gevent.sleep(0.2)

g = gevent.spawn(cron)
sub = Popen(['sleep 1; uname'], stdout=PIPE, shell=True)
out, err = sub.communicate()
g.kill()
print(out.rstrip())

```
##  Actors 
actor模型是一个由于Erlang变得普及的更高层的并发模型。 
简单的说它的**主要思想就是许多个独立的Actor，每个Actor有一个可以从 其它Actor接收消息的收件箱。Actor内部的主循环遍历它收到的消息，并 根据它期望的行为来采取行动。**
Gevent没有原生的Actor类型，但在一个子类化的Greenlet内使用队列， 我们可以定义一个非常简单的。
```

import gevent
from gevent.queue import Queue
class Actor(gevent.Greenlet):

    def __init__(self):
        self.inbox = Queue()
        Greenlet.__init__(self)

    def receive(self, message):
        """
        Define in your subclass.
        """
        raise NotImplemented()

    def _run(self):
        self.running = True
        while self.running:
            message = self.inbox.get()
            self.receive(message)

```
下面是一个使用的例子：
```

import gevent
from gevent.queue import Queue
from gevent import Greenlet

class Pinger(Actor):
    def receive(self, message):
        print(message)
        pong.inbox.put('ping')
        gevent.sleep(0)

class Ponger(Actor):
    def receive(self, message):
        print(message)
        ping.inbox.put('pong')
        gevent.sleep(0)

ping = Pinger()
pong = Ponger()

ping.start()
pong.start()

ping.inbox.put('start')
gevent.joinall([ping, pong])

```

