title: linux技巧之进程与端口管理 

#  Linux技巧之进程与端口管理 
##  使用awk批量杀进程的命令 
http://blog.csdn.net/hi_kevin/article/details/17024107
```

ps -ef|grep mysql|grep -v grep|awk  '{print "kill -9 " $2}' |sh

```
而ps -ef|grep boco|grep -v grep|awk '{print "kill -9 "$2}'则列出了要kill掉这些进程的命令，并将之打印在了屏幕上
在ps -ef|grep boco|grep -v grep|awk '{print "kill -9 "$2}'后面加上|sh后，则执行这些命令，进而杀掉了这些进程。

----
##  free显示内存使用情况 
free命令可以显示Linux系统中空闲的、已用的物理内存及swap内存,及被内核使用的buffer。在Linux系统监控的工具中，free命令是最经常使用的命令之一。
free [参数]
-b 　以Byte为单位显示内存使用情况。 
-k 　以KB为单位显示内存使用情况。 
-m 　以MB为单位显示内存使用情况。
-g   以GB为单位显示内存使用情况。 
-o 　不显示缓冲区调节列。 
-s<间隔秒数> 　持续观察内存使用状况。 
-t 　显示内存总和列。 
-V 　显示版本信息。 
具体参考：http://www.cnblogs.com/peida/archive/2012/12/25/2831814.html
##  top命令 
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。下面详细介绍它的使用方法。top是一个动态显示过程,即可以通过用户按键来不断刷新当前状态.如果在前台执行该命令,它将独占前台,直到用户终止该程序为止.比较准确的说,top命令提供了实时的对系统处理器的状态监视.
命令参数：
-b 批处理
-c 显示完整的治命令
-I 忽略失效过程
-s 保密模式
-S 累积模式
-i<时间> 设置间隔时间
-u<用户名> 指定用户名
-p<进程号> 指定进程
-n<次数> 循环显示的次数

具体参考:http://www.cnblogs.com/peida/archive/2012/12/24/2831353.html
##  kill、killall 
略。。。