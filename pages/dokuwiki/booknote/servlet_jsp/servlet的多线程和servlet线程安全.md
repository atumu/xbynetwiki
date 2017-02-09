title: servlet的多线程和servlet线程安全 

#  servlet的多线程和servlet线程安全 
原文：http://blog.sina.com.cn/s/blog_6151984a0100khbf.html
##  JSP/Servlet的多线程原理: 
1.servelet就是一个CGI,但比传统的CGI要快得过
传统CGI是多进程的,**servlet是多线程的**
以多线程方式执行可大大降低对系统的资源需求,提高 系统的并发量及响应时间.
JSP/Servlet容器默认是采用` 单实例多线程 `(这是造成线程安全的主因)方式处理多个请求的：
当客户端第一次请求某一个JSP文件时(有的servlet是随容器启动就startup)：
服务端把该JSP编译成一个CLASS文件
并创建一个该类的实例
然后创建一个线程处理CLIENT端的请求。
**多请求，多线程**
如果有多个客户端同时请求该JSP文件，则服务端会创建多个线程。` 每个客户端请求对应一个线程 `。

##  servlet的线程安全 
**servlet里的 实例变量**
servlet里的**实例变量，是被所有线程共享的**,**所以不是线程安全的.**
**servlet方法里的局部变量**
因为每个线程都有它自己的堆栈空间,方法内局部变量存储在这个线程堆栈空间内,且参数传入方法是按传值volue copy的方式所以是线程安全的
**Application对象**
在container运行期间,**被整个系统内所有用户共同使用,所以不是线程安全 的**
**ServletContext对象**
**ServletContext是可以多线程同时读/写属性的,线程是不安全的。**
（struts2 的ServletContext采用的是TreadLocal模式,是线程安全的）
**HttpServletRequest对象和HttpServletResponse对象**
` 每一个请求，由一个工作线程来执行， `都会创建有一对新的ServletRequest对象和ServletResponse,然后传入service()方法内
**所以每个ServletRequest对象对应每个线程，而不是多线程共享，是线程安全的**。所以不用担心request参数和属性的线程安全性
**HttpSession**
Session对象在用户session期间存在，**只能在属于同一个SessionID的请求的线程中被访问，**因此Session对象的**理论上是线程安全的。**
(当用户**打开多个同属于一个进程的浏览器窗口**（常见的弹出窗口），在这些窗口的访问属于同一个Session，会出现多次请求，需要多个工作线程来处理请求,这时就有可能的**出现线程安全问题**)
而且在**分布式环境**中也会造成线程安全问题

servlet尽量用方法内变量,就一定线程安全么? 局部变量的数据也来自request对象或session对象啊,它们线程安全么?
**servletRequest 线程是安全的**
因为:每个 request 都会创建一个新线程,每个新线程,容器又都会创建一对servletRequest和servletResponse对象(这是servlet基本原理)
所以servletRequest对象和servletResponse对象只在一个线程内被创建,存在,被访问
##  常见的线程安全的解决办法: 
1.使用方法内**局部变量**  是因为各线程有自己堆栈空间,存储局部变量方法参数传入,多采用传值（volue copy)传入方法内
**2.对操作共享资源的语句,方法,对象, 使用同步**
比如写入磁盘文件,采用同步锁，但建议尽量用同步代码块，不要用同步方法
**3.使用同步的集合类**
**4.不要在 Servlet中再创建自己的线程来完成某个功能。**
Servlet本身就是多线程的，在Servlet中再创建线程，将导致执行情况复杂化