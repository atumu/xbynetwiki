title: linux之进程管理 

#  linux之进程管理 
##  shell后台进程命令 
一、 &
加在一个命令的最后，可以把这个命令放到后台执行 ,如gftp &,

二、ctrl + z
可以将一个正在前台执行的命令放到后台，并且处于暂停状态，不可执行

三、jobs
查看当前有多少在后台运行的命令
jobs -l选项可显示所有任务的PID，jobs的状态可以是running, stopped, Terminated,但是如果任务被终止了（kill），shell 从当前的shell环境已知的列表中删除任务的进程标识；也就是说，jobs命令显示的是当前shell环境中所起的后台正在运行或者被挂起的任务信息；

四、fg
将后台中的命令调至前台继续运行
如果后台中有多个命令，可以用 fg %jobnumber将选中的命令调出，%jobnumber是通过jobs命令查到的后台正在执行的命令的序号(不是pid)

五、bg
将一个在后台暂停的命令，变成继续执行 （在后台执行）
如果后台中有多个命令，可以用bg %jobnumber将选中的命令调出，%jobnumber是通过jobs命令查到的后台正在执行的命令的序号(不是pid)
将任务转移到后台运行：
先ctrl + z；再bg，这样进程就被移到后台运行，终端还能继续接受命令。
概念：当前任务
如果后台的任务号有2个，[1],[2]；如果当第一个后台任务顺利执行完毕，第二个后台任务还在执行中时，当前任务便会自动变成后台任务号码“[2]” 的后台任务。所以可以得出一点，即当前任务是会变动的。当用户输入“fg”、“bg”和“stop”等命令时，如果不加任何引号，则所变动的均是当前任务

###  进程的终止 

后台进程的终止：
方法一：
通过jobs命令查看job号（假设为num），然后执行kill %num
方法二：
通过ps命令查看job的进程号（PID，假设为pid），然后执行kill pid
前台进程的终止：
ctrl+c
kill的其他作用
kill除了可以终止进程，还能给进程发送其它信号，使用kill -l 可以察看kill支持的信号。
SIGTERM是不带参数时kill发送的信号，意思是要进程终止运行，但执行与否还得看进程是否支持。如果进程还没有终止，可以使用kill -SIGKILL pid，这是由内核来终止进程，进程不能监听这个信号。

###  进程的挂起 


1)、后台进程的挂起：

在solaris中通过stop命令执行，通过jobs命令查看job号(假设为num)，然后执行stop %num；
在redhat中，不存在stop命令，可通过执行命令kill -stop PID，将进程挂起；
当要重新执行当前被挂起的任务时，通过bg %num 即可将挂起的job的状态由stopped改为running，仍在后台执行；当需要改为在前台执行时，执行命令fg %num即可；

2)、前台进程的挂起：

ctrl+Z;
##  linux进程管理命令 

包括 ps、pgrep、top 、kill、pkill、killall、nice和renice 等工具。

###  进程概述 

1.1 进程分类；

进程一般分为交互进程、批处理进程和守护进程三类。

值得一提的是守护进程总是活跃的，一般是后台运行，守护进程一般是由系统在开机时通过脚本自动激活启动或超级管理用户root来启动。比如在Fedora或Redhat中，我们可以定义httpd 服务器的启动脚本的运行级别，此文件位于/etc/init.d目录下，文件名是httpd，/etc/init.d/httpd 就是httpd服务器的守护程序，当把它的运行级别设置为3和5时，当系统启动时，它会跟着启动。
```

[root@localhost ~]# chkconfig  --level 35  httpd on

```
由于守护进程是一直运行着的，所以它所处的状态是等待请求处理任务。比如，我们是不是访问 LinuxSir.Org ，LinuxSir.Org 的httpd服务器都在运行，等待着用户来访问，也就是等待着任务处理。
1.2 进程的属性；
进程ID（PID)：是唯一的数值，用来区分进程；
父进程和父进程的ID（PPID)；
启动进程的用户ID（UID）和所归属的组（GID）；
进程状态：状态分为运行R、休眠S、僵尸Z；
进程执行的优先级；
进程所连接的终端名；
进程资源占用：比如占用资源大小（内存、CPU占用量）；

1.3 父进程和子进程；

他们的关系是管理和被管理的关系，当父进程终止时，子进程也随之而终止。但子进程终止，父进程并不一定终止。比如httpd服务器运行时，我们可以杀掉其子进程，父进程并不会因为子进程的终止而终止。
在进程管理中，当我们发现占用资源过多，或无法控制的进程时，应该杀死它，以保护系统的稳定安全运行；

###  2、进程管理； 


对于Linux进程的管理，是通过进程管理工具实现的，比如ps、kill、pgrep等工具；


###  2.1 ps 监视进程工具； 


ps 为我们提供了进程的一次性的查看，它所提供的查看结果并不动态连续的；如果想对进程时间监控，应该用top工具；


2.1.1 ps 的参数说明；

ps 提供了很多的选项参数，常用的有以下几个；

l  长格式输出；
u  按用户名和启动时间的顺序来显示进程；
j  用任务格式来显示进程；
f  用树形格式来显示进程；
a  显示所有用户的所有进程（包括其它用户）；
x  显示无控制终端的进程；
r  显示运行中的进程；
ww 避免详细参数被截断；
我们常用的选项是组合是aux 或lax，还有参数f的应用；

**ps aux** 或lax输出的解释；

USER	进程的属主；
PID	进程的ID；
PPID  父进程；
%CPU	进程占用的CPU百分比；
%MEM	占用内存的百分比；
NI	   进程的NICE值，数值大，表示较少占用CPU时间；
VSZ 进程虚拟大小；
RSS  驻留中页的数量；
WCHAN 
TTY  终端ID
STAT 进程状态

D    Uninterruptible sleep (usually IO)
R    正在运行可中在队列中可过行的； 
S    处于休眠状态；
T    停止或被追踪； 
W    进入内存交换（从内核2.6开始无效）；
X    死掉的进程（从来没见过）；
Z    僵尸进程；

<           优先级高的进程 
N    优先级较低的进程 
L    有些页被锁进内存； 
s    进程的领导者（在它之下有子进程）；
l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
+   	     位于后台的进程组；

WCHAN	正在等待的进程资源；
START 启动进程的时间；
TIME	进程消耗CPU的时间；
COMMAND 命令的名称和参数；

###  2.1.2 ps 应用举例； 


实例一：ps aux 最常用

 

[root@localhost ~]# ps -aux |more
 

可以用 | 管道和 more 连接起来分页查看；

 

[root@localhost ~]# ps -aux  > ps001.txt
[root@localhost ~]# more ps001.txt
 

这里是把所有进程显示出来，并输出到ps001.txt文件，然后再通过more 来分页查看；

实例二：和grep 结合，提取指定程序的进程；

 

[root@localhost ~]# ps aux |grep httpd
root      4187  0.0  1.3  24236 10272 ?        Ss   11:55   0:00 /usr/sbin/httpd
apache    4189  0.0  0.6  24368  4940 ?        S    11:55   0:00 /usr/sbin/httpd
apache    4190  0.0  0.6  24368  4932 ?        S    11:55   0:00 /usr/sbin/httpd
apache    4191  0.0  0.6  24368  4932 ?        S    11:55   0:00 /usr/sbin/httpd
apache    4192  0.0  0.6  24368  4932 ?        S    11:55   0:00 /usr/sbin/httpd
apache    4193  0.0  0.6  24368  4932 ?        S    11:55   0:00 /usr/sbin/httpd
apache    4194  0.0  0.6  24368  4932 ?        S    11:55   0:00 /usr/sbin/httpd
apache    4195  0.0  0.6  24368  4932 ?        S    11:55   0:00 /usr/sbin/httpd
apache    4196  0.0  0.6  24368  4932 ?        S    11:55   0:00 /usr/sbin/httpd
root      4480  0.0  0.0   5160   708 pts/3    R+   12:20   0:00 grep httpd
 

实例二：父进和子进程的关系友好判断的例子

 

[root@localhost ~]# ps auxf  |grep httpd
root      4484  0.0  0.0   5160   704 pts/3    S+   12:21   0:00              /_ grep httpd
root      4187  0.0  1.3  24236 10272 ?        Ss   11:55   0:00 /usr/sbin/httpd
apache    4189  0.0  0.6  24368  4940 ?        S    11:55   0:00  /_ /usr/sbin/httpd
apache    4190  0.0  0.6  24368  4932 ?        S    11:55   0:00  /_ /usr/sbin/httpd
apache    4191  0.0  0.6  24368  4932 ?        S    11:55   0:00  /_ /usr/sbin/httpd
apache    4192  0.0  0.6  24368  4932 ?        S    11:55   0:00  /_ /usr/sbin/httpd
apache    4193  0.0  0.6  24368  4932 ?        S    11:55   0:00  /_ /usr/sbin/httpd
apache    4194  0.0  0.6  24368  4932 ?        S    11:55   0:00  /_ /usr/sbin/httpd
apache    4195  0.0  0.6  24368  4932 ?        S    11:55   0:00  /_ /usr/sbin/httpd
apache    4196  0.0  0.6  24368  4932 ?        S    11:55   0:00  /_ /usr/sbin/httpd
 

这里用到了f参数；父与子关系一目了然；


###  2.2 pgrep 


pgrep 是通过程序的名字来查询进程的工具，一般是用来判断程序是否正在运行。在服务器的配置和管理中，这个工具常被应用，简单明了；

用法：

 

#ps 参数选项   程序名
 

常用参数

 

-l  列出程序名和进程ID；
-o  进程起始的ID；
-n  进程终止的ID；
 

举例：

 

[root@localhost ~]# pgrep -lo httpd
4557 httpd

[root@localhost ~]# pgrep -ln httpd
4566 httpd

[root@localhost ~]# pgrep -l httpd
4557 httpd
4560 httpd
4561 httpd
4562 httpd
4563 httpd
4564 httpd
4565 httpd
4566 httpd

[root@localhost ~]# pgrep httpd
4557
4560
4561
4562
4563
4564
4565
4566
 


###  3、终止进程的工具 kill 、killall、pkill、xkill； 


终止一个进程或终止一个正在运行的程序，一般是通过 kill 、killall、pkill、xkill 等进行。比如一个程序已经死掉，但又不能退出，这时就应该考虑应用这些工具。

另外应用的场合就是在服务器管理中，在不涉及数据库服务器程序的父进程的停止运行，也可以用这些工具来终止。为什么数据库服务器的父进程不能用这些工具杀死呢？原因很简单，这些工具在强行终止数据库服务器时，会让数据库产生更多的文件碎片，当碎片达到一定程度的时候，数据库就有崩溃的危险。比如mysql服务器最好是按其正常的程序关闭，而不是用pkill mysqld 或killall mysqld 这样危险的动作；当然对于占用资源过多的数据库子进程，我们应该用kill 来杀掉。

3.1 kill
kill的应用是和ps 或pgrep 命令结合在一起使用的；
kill 的用法：
kill ［信号代码］   进程ID
注：信号代码可以省略；我们常用的信号代码是 -9 ，表示强制终止；
![](/data/dokuwiki/linux/pasted/20150628-230248.png)
举例：
[root@localhost ~]# ` ps  auxf  |grep   httpd `
root      4939  0.0  0.0   5160   708 pts/3    S+   13:10   0:00              /_ grep httpd
root      4830  0.1  1.3  24232 10272 ?        Ss   13:02   0:00 /usr/sbin/httpd
apache    4833  0.0  0.6  24364  4932 ?        S    13:02   0:00  /_ /usr/sbin/httpd
apache    4834  0.0  0.6  24364  4928 ?        S    13:02   0:00  /_ /usr/sbin/httpd
apache    4835  0.0  0.6  24364  4928 ?        S    13:02   0:00  /_ /usr/sbin/httpd
apache    4836  0.0  0.6  24364  4928 ?        S    13:02   0:00  /_ /usr/sbin/httpd
apache    4837  0.0  0.6  24364  4928 ?        S    13:02   0:00  /_ /usr/sbin/httpd
apache    4838  0.0  0.6  24364  4928 ?        S    13:02   0:00  /_ /usr/sbin/httpd
apache    4839  0.0  0.6  24364  4928 ?        S    13:02   0:00  /_ /usr/sbin/httpd
apache    4840  0.0  0.6  24364  4928 ?        S    13:02   0:00  /_ /usr/sbin/httpd
 
我们查看httpd 服务器的进程；您也可以用pgrep -l httpd 来查看；

我们看上面例子中的第二列，就是进程PID的列，其中4830是httpd服务器的父进程，从4833－4840的进程都是它4830的子进程；如果我们杀掉父进程4830的话，其下的子进程也会跟着死掉；

[root@localhost ~]# kill 4840  注：杀掉4840这个进程；
[root@localhost ~]# ps -auxf  |grep  httpd  注：查看一下会有什么结果？是不是httpd服务器仍在运行？
[root@localhost ~]# kill 4830   注：杀掉httpd的父进程；
[root@localhost ~]# ps -aux |grep httpd  注：查看httpd的其它子进程是否存在，httpd服务器是否仍在运行？
对于僵尸进程，可以用kill -9 来强制终止退出；
比如一个程序已经彻底死掉，如果kill 不加信号强度是没有办法退出，最好的办法就是加信号强度 -9 ，后面要接杀父进程；比如；

```

[root@localhost ~]# ps aux |grep gaim
beinan    5031  9.0  2.3 104996 17484 ?        S    13:23   0:01 gaim
root      5036  0.0  0.0   5160   724 pts/3    S+   13:24   0:00 grep gaim
或
[root@localhost ~]# pgrep -l gaim
5031 gaim
[root@localhost ~]# kill -9 5031

```
 
3.2 killall
killall 通过程序的名字，直接杀死所有进程，咱们简单说一下就行了。

用法：killall 正在运行的程序名
killall 也和ps或pgrep 结合使用，比较方便；通过ps或pgrep 来查看哪些程序在运行；
举例：
[root@localhost beinan]# pgrep -l gaim
2979 gaim
[root@localhost beinan]# killall gaim
 
3.3 pkill
pkill 和killall 应用方法差不多，也是直接杀死运行中的程序；如果您想杀掉单个进程，请用kill 来杀掉。
应用方法：
#pkill  正在运行的程序名
举例：
[root@localhost beinan]# pgrep -l gaim
2979 gaim
[root@localhost beinan]# pkill gaim


4###  、top 监视系统任务的工具； 


和ps 相比，top是动态监视系统任务的工具，top 输出的结果是连续的；


4.1 top 命令用法及参数；

top 调用方法：

 

top 选择参数
 

参数：

 

-b  以批量模式运行，但不能接受命令行输入；
-c	显示命令行，而不仅仅是命令名；
-d N  显示两次刷新时间的间隔，比如 -d 5，表示两次刷新间隔为5秒；
-i 禁止显示空闲进程或僵尸进程；
-n NUM  显示更新次数，然后退出。比如 -n 5，表示top更新5次数据就退出；
-p PID 仅监视指定进程的ID；PID是一个数值；
-q  不经任何延时就刷新；
-s  安全模式运行，禁用一些效互指令；
-S 累积模式，输出每个进程的总的CPU时间，包括已死的子进程；
 


交互式命令键位：

 

space  立即更新；
c 切换到命令名显示，或显示整个命令（包括参数）；
f,F	增加显示字段，或删除显示字段；
h,?	显示有关安全模式及累积模式的帮助信息；
k	提示输入要杀死的进程ID，目的是用来杀死该进程（默人信号为15）
i 禁止空闲进程和僵尸进程；
l	切换到显法负载平均值和正常运行的时间等信息；
m	切换到内存信息，并以内存占用大小排序；
n  提示显示的进程数，比如输入3，就在整屏上显示3个进程；
o,O	改变显示字段的顺序；
r	把renice 应用到一个进程，提示输入PID和renice的值；
s 改变两次刷新时间间隔，以秒为单位；
t	切换到显示进程和CPU状态的信息；
A	按进程生命大小进行排序，最新进程显示在最前；
M 按内存占用大小排序，由大到小；
N	以进程ID大小排序，由大到小；
P	按CPU占用情况排序，由大到小
S 切换到累积时间模式；
T  按时间／累积时间对任务排序；
W	把当前的配置写到~/.toprc中；
 


4.2 top 应用举例；

 

[root@localhost ~]# top
 

然后根据前面所说交互命令按个尝试一下就明白了，比如按M，就按内存占用大小排序；这个例子举不举都没有必要了。呵。。。。。。

当然您可以把top的输出传到一个文件中；

 

[root@localhost ~]# top > mytop.txt
 

然后我们就可以查看mytop文件，以慢慢的分析系统进程状态；


5、进程的优先级：nice和renice；

在Linux 操作系统中，进程之间是竟争资源（比如CPU和内存的占用）关系。这个竟争优劣是通过一个数值来实现的，也就是谦让度。高谦让度表示进程优化级别最低。负值或0表示对高优点级，对其它进程不谦让，也就是拥有优先占用系统资源的权利。谦让度的值从 －20到19。

目前硬件技术发展极速，在大多情况下，不必设置进程的优先级，除非在进程失控而疯狂占用资源的情况下，我们有可能来设置一下优先级，但我个人感觉没有太大的必要，在迫不得已的情况下，我们可以杀掉失控进程。

nice 可以在创建进程时，为进程指定谦让度的值，进程的优先级的值是父进程SHELL的优先级的值与我们所指定谦让度的相加和。所以我们在用nice设置程序的优先级时，所指定数值是一个增量，并不是优先级的绝对值；

nice 的应用举例：

 

[root@localhost ~]# nice -n 5  gaim &   注：运行gaim程序，并为它指定谦让度增量为5；
 

所以nice的最常用的应用就是：

 

nice  -n  谦让度的增量值   程序
 

renice 是通过进程ID（PID）来改变谦让度，进而达到更改进程的优先级。

 

renice  谦让度    PID
 

renice 所设置的谦让度就是进程的绝对值；看下面的例子；

[root@localhost ~]# ps lax   |grep gaim
4     0  4437  3419  10  -5 120924 20492 -      S<   pts/0      0:01 gaim
0     0  4530  3419  10  -5   5160   708 -      R<+  pts/0      0:00 grep gaim

[root@localhost ~]# renice -6  4437
4437: old priority -5, new priority -6

[root@localhost ~]# ps lax   |grep gaim
4     0  4437  3419  14  -6 120924 20492 -      S<   pts/0      0:01 gaim
0     0  4534  3419  11  -5   5160   708 -      R<+  pts/0      0:00 grep gaim

参考：
http://blog.csdn.net/lengyuhong/article/details/6173345
http://www.jb51.net/LINUXjishu/72583.html