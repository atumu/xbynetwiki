title: chapter14_多线程 


#  Chapter14 多线程 
Created Monday 16 March 2015
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072633.png)

#  创建并启动线程的步骤： 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072700.png)
#  中断线程（重点理解） 
线程终止的原由：run()方法由return返回时；出现了没有捕获的异常。
interrupt()方法可以用来请求终止线程（注意：仅仅只是请求，而不是强制，这是一种协作式的商量方式。）该方法会置位线程的中断状态（boolean，设置为true）。故而每个线程都应该不时通过isInterrupted()方法检查这个标志。以判断线程是否被设置为中断。
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072725.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072747.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072812.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072829.png)
#  线程状态 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072837.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072845.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072849.png)
##  被阻塞线程和等待线程： 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072855.png)

##  线程终止 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072909.png)

##  线程的属性 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072933.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-072952.png)

#  同步 
竞争条件（race condition）的概念

##  锁对象 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073013.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073030.png)
##  条件对象 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073049.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073108.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073121.png)
##  synchronized关键字 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073141.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073156.png)
##  同步阻塞 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073214.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073227.png)
##  监视器的概念 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073234.png)

##  Volatile域 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073254.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073308.png)
##  final变量 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073321.png)

##  原子性 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073328.png)

##  死锁 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073345.png)

##  线程局部变量 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073355.png)

##  锁测试与超时 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073401.png)

##  读写锁 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073405.png)

##  阻塞对列 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073431.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073447.png)
#  线程安全的集合： 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073508.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073521.png)
#  Callable和Future 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073537.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073557.png)
#  执行器Executor 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073603.png)

##  线程池 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073610.png)

##  预定执行 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073625.png)

##  控制任务组 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073639.png)

##  Fork-Join框架 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073652.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073700.png)

#  同步器 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073712.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073722.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073730.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073756.png)

#  线程与Swing 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073840.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073856.png)
##  使用SwingWorker 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073920.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073940.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-073957.png)
##  单一线程规则 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-074019.png)
#  需要学习的API： 
java.lang.Thread
java.lang.Runnable
