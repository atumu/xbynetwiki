title: chapter11_异常_断言_日志和调试 


#  Chapter11 异常、断言、日志和调试 
Created Sunday 15 March 2015
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065123.png)

#  处理错误 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065127.png)
需要关注的问题:
* 用户输入错误;
* 设备错误
* 物理限制:如磁盘已满
* 代码错误

##  异常分类 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065132.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065157.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065204.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065208.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065213.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065219.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065224.png)

##  创建异常类 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065230.png)

#  捕获异常 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065235.png)

##  捕获多个异常 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065242.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065250.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065320.png)

##  finally字句 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065325.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065328.png)

##  带有资源的try语句 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065334.png)![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065336.png)

##  分析堆栈跟踪元素 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065344.png)

#  使用异常机制的技巧 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065403.png)
#  使用断言 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065457.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065509.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065515.png)
#  记录日志 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065546.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065620.png)
##  处理器 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065630.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065642.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065653.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065701.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-065723.png)
#  调试技巧 

#  使用调试器 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-070319.png)
#  需要学习的API： 
java.util.logging.Logger
java.util.logging.Handler
