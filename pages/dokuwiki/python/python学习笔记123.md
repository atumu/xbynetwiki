title: python学习笔记123 

#  Python总结之并发 
#线程安全/竞争条件,锁/死锁检测,同步/异步,阻塞/非阻塞,epoll非阻塞IO,信号量/事件,线程池,生产消费模型,伪并发,微线程,协程
#Stackless Python 是Python编程语言的一个增强版本，它使程序员从基于线程的编程方式中获得好处，并避免传统线程所带来的性能与复杂度问题。Stackless为 Python带来的微线程扩展，是一种低开销、轻量级的便利工具
###  threading多线程 
```

		thread
			start_new_thread(function,args kwargs=None)    # 产生一个新的线程
			allocate_lock()                                # 分配一个LockType类型的锁对象
			exit()                                         # 让线程退出
			acquire(wait=None)                             # 尝试获取锁对象
			locked()                                       # 如果获取了锁对象返回True
			release()                                      # 释放锁

		thread例子

			#!/usr/bin/env python
			#thread_test.py
			#不支持守护进程
			import thread
			from time import sleep,ctime

			loops = [4,2]

			def loop(nloop,nsec,lock):
				print 'start loop %s at:%s' % (nloop,ctime())
				sleep(nsec)
				print 'loop %s done at: %s' % (nloop, ctime())
				lock.release()              # 分配已获得的锁,操作结束后释放相应的锁通知主线程

			def main():
				print 'starting at:',ctime()
				locks = []
				nloops = range(len(loops))
				
				for i in nloops:
					lock = thread.allocate_lock()     # 创建一个锁
					lock.acquire()                    # 调用各个锁的acquire()函数获得锁
					locks.append(lock)                # 把锁放到锁列表locks中
				for i in nloops:
					thread.start_new_thread(loop,(i,loops[i],locks[i]))   # 创建线程
				for i in nloops:
					while locks[i].locked():pass      # 等待全部解锁才继续运行
				print 'all DONE at:',ctime()

			if __name__ == '__main__':
				main()

		thread例子1

			#coding=utf-8
			import thread,time,os

			def f(name):
					i =3
					while i:
							time.sleep(1)
							print name
							i -= 1
					# os._exit()   会把整个进程关闭
					os._exit(22)

			if __name__ == '__main__':
					thread.start_new_thread(f,("th1",))
					while 1:
							pass
					os._exit(0)
				
		threading
			Thread                   # 表示一个线程的执行的对象
				start()              # 开始线程的执行
				run()                # 定义线程的功能的函数(一般会被子类重写)
				join(timeout=None)   # 允许主线程等待线程结束,程序挂起,直到线程结束;如果给了timeout,则最多等待timeout秒.
				getName()            # 返回线程的名字
				setName(name)        # 设置线程的名字
				isAlive()            # 布尔标志,表示这个线程是否还在运行中
				isDaemon()           # 返回线程的daemon标志
				setDaemon(daemonic)  # 后台线程,把线程的daemon标志设置为daemonic(一定要在调用start()函数前调用)
				# 默认主线程在退出时会等待所有子线程的结束。如果希望主线程不等待子线程，而是在退出时自动结束所有的子线程，就需要设置子线程为后台线程(daemon)
			Lock              # 锁原语对象
			Rlock             # 可重入锁对象.使单线程可以在此获得已获得了的锁(递归锁定)
			Condition         # 条件变量对象能让一个线程停下来,等待其他线程满足了某个条件.如状态改变或值的改变
			Event             # 通用的条件变量.多个线程可以等待某个事件的发生,在事件发生后,所有的线程都会被激活
			Semaphore         # 为等待锁的线程提供一个类似等候室的结构
			BoundedSemaphore  # 与Semaphore类似,只是不允许超过初始值
			Time              # 与Thread相似,只是他要等待一段时间后才开始运行
			activeCount()     # 当前活动的线程对象的数量
			currentThread()   # 返回当前线程对象
			enumerate()       # 返回当前活动线程的列表
			settrace(func)    # 为所有线程设置一个跟踪函数
			setprofile(func)  # 为所有线程设置一个profile函数

		threading例子1
			
			#!/usr/bin/env python
			#encoding:utf8
			import threading
			from Queue import Queue
			from time import sleep,ctime

			class ThreadFunc(object):
					def __init__(self,func,args,name=''):
							self.name=name
							self.func=func                    # loop
							self.args=args                    # (i,iplist[i],queue)
					def __call__(self):
							apply(self.func,self.args)        # 函数apply() 执行loop函数并传递元组参数
			def loop(nloop,ip,queue):
					print 'start',nloop,'at:',ctime()
					queue.put(ip)
					sleep(2)
					print 'loop',nloop,'done at:',ctime()
			if __name__ == '__main__':
					threads = []
					queue = Queue()
					iplist = ['192.168.1.2','192.168.1.3','192.168.1.4','192.168.1.5','192.168.1.6','192.168.1.7','192.168.1.8']
					nloops = range(len(iplist))

					for i in nloops:
							t = threading.Thread(target=ThreadFunc(loop,(i,iplist[i],queue),loop.__name__))
							threads.append(t)
					for i in nloops:
							threads[i].start()
					for i in nloops:
							threads[i].join()
					for i in nloops:
							print queue.get()

		threading例子2

			#!/usr/bin/env python
			#encoding:utf8
			from Queue import Queue
			import random,time,threading
			
			class Producer(threading.Thread):
				def __init__(self, t_name, queue):
					threading.Thread.__init__(self, name=t_name)
					self.data=queue
				def run(self):
					for i in range(5):
						print "%s: %s is producing %d to the queue!\n" %(time.ctime(), self.getName(), i)
						self.data.put(i)
						self.data.put(i*i)
						time.sleep(2)
					print "%s: %s finished!" %(time.ctime(), self.getName())

			class Consumer(threading.Thread):
				def __init__(self, t_name, queue):
					threading.Thread.__init__(self, name=t_name)
					self.data=queue
				def run(self):
					for i in range(10):
						val = self.data.get()
						print "%s: %s is consuming. %d in the queue is consumed!\n" %(time.ctime(), self.getName(), val)
					print "%s: %s finished!" %(time.ctime(), self.getName())

			if __name__ == '__main__':
				queue = Queue()
				producer = Producer('Pro.', queue)
				consumer = Consumer('Con.', queue)
				producer.start()
				consumer.start()
				producer.join()
				consumer.join()

		threading例子3
		
			# 启动线程后自动执行 run函数其他不可以
			import threading
			import time

			class Th(threading.Thread):
				def __init__(self,name):
					threading.Thread.__init__(self)
					self.t_name=name
					self.daemon = True     # 默认为false，让主线程等待处理完成
				def run(self):
					time.sleep(1)
					print "this is " + self.t_name

			if __name__ == '__main__':
				thread1 = Th("Th_1")
				thread1.start()

		threading例子4
		
			import threading
			import time
			class Th(threading.Thread):
			def __init__(self,thread_name):
				threading.Thread.__init__(self)
				self.setName(thread_name)
			def run(self):
				threadLock.acquire()
				print self.getName()
				for i in range(3):
					time.sleep(1)
					print str(i)
				print self.getName() +  " is over"
				threadLock.release()

			if __name__ == '__main__':
				threadLock = threading.Lock()
				thread1 = Th("Th_1")
				thread2 = Th("Th_2")
				thread1.start()
				thread2.start()

```
###  后台线程 
```

			import threading
			import time,random

			class MyThread(threading.Thread):
				def run(self):
					wait_time=random.randrange(1,10)
					print "%s will wait %d seconds" % (self.name, wait_time)
					time.sleep(wait_time)
					print "%s finished!" % self.name

			if __name__=="__main__":
				for i in range(5):
					t = MyThread()
					t.setDaemon(True)    # 设置为后台线程,主线程完成时不等待子线程完成就结束
					t.start()

```
###  threading控制最大并发_查询日志中IP信息 
```

			#!/usr/bin/env python
			#coding:utf-8
			import urllib2
			import json
			import threading
			import time

			'''
			by:某大牛
			QQ:185635687
			这个是多线程并发控制. 如果要改成多进程，只需把threading 换成 mulitprocessing.Process ， 对， 就是换个名字而已.
			'''

			#获取ip 及其出现次数
			def ip_dic(file_obj, dic):
				for i in file_obj:
					if i:
						ip=i.split('-')[0].strip()
						if ip in dic.keys():
							dic[ip]=dic[ip] + 1
						else:
							dic[ip]=1
				return dic.iteritems()

			#目标函数
			def get_data(url, ipcounts):
				data=urllib2.urlopen(url).read()
				datadict=json.loads(data)
				fdata = u"ip:%s---%s,%s,%s,%s,%s" %(datadict["data"]["ip"],ipcounts,datadict["data"]["country"],datadict["data"]["region"],datadict["data"]["city"],datadict["data"]["isp"])
				print fdata

			#多线程
			def threads(iters):
				thread_pool = []
				for k in iters:
					url = "http://ip.taobao.com/service/getIpInfo.php?ip="
					ipcounts = k[1]
					url = (url + k[0]).strip()
					t = threading.Thread(target=get_data, args=(url, ipcounts))
					thread_pool.append(t)
				return thread_pool

			#控制多线程
			def startt(t_list, max,second):
				l = len(t_list)
				n = max
				while l > 0:
					if l > max:
						nl = t_list[:max]
						t_list = t_list[max:]
						for t in nl:
							t.start()
						time.sleep(second)
						for t in nl:
							t.join()
						print '*'*15,  str(n)+ ' ip has been queried'+'*'*15
						n += max
						l = len(t_list)
						continue
					elif l <= max:
						nl = t_list
						for t in nl:
							t.start()
						for t in nl:
							t.join()
						print '>>> Totally ' + str(n+l ) + ' ip has been queried'
						l = 0

			if __name__ =="__main__":
				dic={}
				with open('access.log') as file_obj:
					it = ip_dic(file_obj, dic)
					t_list= threads(it)
					startt(t_list, 15, 1)


```
###  多线程取队列 
```

			#!/usr/bin/python

			import Queue
			import threading
			import time

			exitFlag = 0

			class myThread (threading.Thread):
				def __init__(self, threadID, name, q):
					threading.Thread.__init__(self)
					self.threadID = threadID
					self.name = name
					self.q = q
				def run(self):
					print "Starting " + self.name
					process_data(self.name, self.q)
					print "Exiting " + self.name

			def process_data(threadName, q):
				while not exitFlag:      # 死循环等待
					queueLock.acquire()
					if not q.empty():    # 判断队列是否为空
						data = q.get()
						print "%s processing %s" % (threadName, data)
					queueLock.release()
					time.sleep(1)

			threadList = ["Thread-1", "Thread-2", "Thread-3"]
			nameList = ["One", "Two", "Three", "Four", "Five"]
			queueLock = threading.Lock()     # 锁与队列并无任何关联，其他线程也进行取锁操作的时候就会检查是否有被占用，有就阻塞等待解锁为止
			workQueue = Queue.Queue(10)
			threads = []
			threadID = 1

			# Create new threads
			for tName in threadList:
				thread = myThread(threadID, tName, workQueue)
				thread.start()
				threads.append(thread)
				threadID += 1

			# Fill the queue
			queueLock.acquire()
			for word in nameList:
				workQueue.put(word)
			queueLock.release()

			# Wait for queue to empty
			while not workQueue.empty():   # 死循环判断队列被处理完毕
				pass

			# Notify threads it's time to exit
			exitFlag = 1

			# Wait for all threads to complete
			for t in threads:
				t.join()
			print "Exiting Main Thread"

```
###  Queue通用队列 
		q=Queue(size)       # 创建大小size的Queue对象
		qsize()             # 返回队列的大小(返回时候,可能被其他进程修改,近似值)
		empty()             # 如果队列为空返回True，否则Fales
		full()              # 如果队列已满返回True，否则Fales
		put(item,block0)    # 把item放到队列中,如果给了block(不为0),函数会一直阻塞到队列中有空间为止
		get(block=0)        # 从队列中取一个对象,如果给了block(不为0),函数会一直阻塞到队列中有对象为止
		get_nowait          # 默认get阻塞，这个不阻塞

##  multiprocessing [多进程并发] 
```

		多线程
		
			import urllib2
			from multiprocessing.dummy import Pool as ThreadPool

			urls=['http://www.baidu.com','http://www.sohu.com']

			pool=ThreadPool(4)   # 线程池
			results=pool.map(urllib2.urlopen,urls)
			pool.close()
			pool.join()

		多进程并发

			#!/usr/bin/env python
			#encoding:utf8
			from multiprocessing import Process
			import time,os
			def f(name):
				time.sleep(1)
				print 'hello ',name
				print os.getppid()   # 取得父进程ID
				print os.getpid()    # 取得进程ID
			process_list = []

			for i in range(10):
				p = Process(target=f,args=(i,))
				p.start()
				process_list.append(p)
			for j in process_list:
				j.join()

		Queue进程间通信

			from multiprocessing import Process,Queue
			import time
			def f(name):
				time.sleep(1)
				q.put(['hello'+str(name)])
			process_list = []
			q = Queue()
			if __name__ == '__main__':
				for i in range(10):
					p = Process(target=f,args=(i,))
					p.start()
					process_list.append(p)
				for j in process_list:
					j.join()
				for i in range(10):
					print q.get()

		Pipe管道
		
			from multiprocessing import Process,Pipe
			import time
			import os

			def f(conn,name):
				time.sleep(1)
				conn.send(['hello'+str(name)])
				print os.getppid(),'-----------',os.getpid()
			process_list = []
			parent_conn,child_conn = Pipe()
			if __name__ == '__main__':
				for i in range(10):
					p = Process(target=f,args=(child_conn,i))
					p.start()
					process_list.append(p)
				for j in process_list:
					j.join()
				for p in range(10):
					print parent_conn.recv()

		进程间同步
			#加锁，使某一时刻只有一个进程 print
			from multiprocessing import Process,Lock
			import time
			import os

			def f(name):
				lock.acquire()
				time.sleep(1)
				print 'hello--'+str(name)
				print os.getppid(),'-----------',os.getpid()
				lock.release()
			process_list = []
			lock = Lock()
			if __name__ == '__main__':
				for i in range(10):
					p = Process(target=f,args=(i,))
					p.start()
					process_list.append(p)
				for j in process_list:
					j.join()

		共享内存

			# 通过使用Value或者Array把数据存储在一个共享的内存表中
			# 'd'和'i'参数是num和arr用来设置类型，d表示一个双精浮点类型，i表示一个带符号的整型。
			from multiprocessing import Process,Value,Array
			import time
			import os

			def f(n,a,name):
				time.sleep(1)
				n.value = name * name
				for i in range(len(a)):
					a[i] = -i
			process_list = []
			if __name__ == '__main__':
				num = Value('d',0.0)
				arr = Array('i',range(10))
				for i in range(10):
					p = Process(target=f,args=(num,arr,i))
					p.start()
					process_list.append(p)
				for j in process_list:
					j.join()
				print num.value
				print arr[:]

		manager

			# 比共享内存灵活,但缓慢
			# 支持list,dict,Namespace,Lock,Semaphore,BoundedSemaphore,Condition,Event,Queue,Ｖalue,Array
			from multiprocessing import Process,Manager
			import time
			import os

			def f(d,name):
				time.sleep(1)
				d[name] = name * name
				print d
			process_list = []
			if __name__ == '__main__':
				manager = Manager()
				d = manager.dict()
				for i in range(10):
					p = Process(target=f,args=(d,i))
					p.start()
					process_list.append(p)
				for j in process_list:
					j.join()
					print d

		最大并发数

			import multiprocessing
			import time,os

			result = []
			def run(h):
				print 'threading:' ,h,os.getpid()
			p = multiprocessing.Pool(processes=20)

			for i in range(100):
				result.append(p.apply_async(run,(i,)))
			p.close()
			
			for res in result:
				res.get(timeout=5)

```