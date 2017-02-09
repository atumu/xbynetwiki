title: linux3 

#  Linux开机启动管理之chkconfig 
 一、Linux的运行级别
什么是运行级别呢？简单点来说，运行级别就是操作系统当前正在运行的功能级别。级别是从0到6，具有不同的功能。这些级别定义在/ect/inittab文件中。这个文件是init程序寻找的主要文件，最先运行的服务是那些放在/ect/rc.d目录下的文件。
Linux下的7个运行级别：
  * 0系统停机状态，系统默认运行级别不能设置为0，否则不能正常启动，机器关闭。
  * 1单用户工作状态，root权限，用于系统维护，禁止远程登陆，就像Windows下的安全模式登录。
  * 2多用户状态，没有NFS支持。
  * 3完整的多用户模式，有NFS，登陆后进入控制台命令行模式。
  * 4系统未使用，保留一般不用，在一些特殊情况下可以用它来做一些事情。例如在笔记本电脑的电池用尽时，可以切换到这个模式来做一些设置。
  * 5X11控制台，登陆后进入图形GUI模式，X Window系统。
  * 6系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动。运行init 6机器就会重启。
运行级别原理：
1.在目录/etc/rc.d/init.d下有许多服务器脚本程序，一般称为服务(service)
2.在/etc/rc.d下有7个名为rcN.d的目录，对应系统的7个运行级别
3.rcN.d目录下都是一些符号链接文件，这些链接文件都指向init.d目录下的service脚本文件，命名规则为K+nn+服务名或S+nn+服务名，其中nn为两位数字。
4.系统会根据指定的运行级别进入对应的rcN.d目录，并按照文件名顺序检索目录下的链接文件：对于以K开头的文件，系统将终止对应的服； 对于以S开头的文件，系统将启动对应的服务
5.查看运行级别用：runlevel
6.进入其它运行级别用：init N，如果init 3则进入终端模式，init 5则又登录图形GUI模式
7.另外init0为关机，init 6为重启系统
 
标准的Linux运行级别为3或5，如果是3的话，系统就在多用户状态；如果是5的话，则是运行着X Window系统。
不同的运行级别有不同的用处，也应该根据自己的不同情形来设置。例如，如果丢失了root口令，那么可以让机器启动进入单用户状态来设置。在启动后的lilo提示符下输入：
init=/bin/sh rw
这样就可以使机器进入运行级别1，并把root文件系统挂为读写。它会路过所有系统认证，让你使用passwd程序来改变root口令，然后启动到一个新的运行级。
----

chkconfig命令主要用来更新（启动或停止）和查询系统服务的运行级信息。谨记chkconfig不是立即自动禁止或激活一个服务，它只是简单的改变了符号连接。
使用语法：
chkconfig [--add][--del][--list][系统服务] 或 chkconfig [--level <等级代号>][系统服务][on/off/reset]

on和off分别指服务被启动和停止，reset指重置服务的启动信息，无论有问题的初始化脚本指定了什么。on和off开关，系统默认只对运行级3，4，5有效，但是reset可以对所有运行级有效。

参数用法：
   --add 　增加所指定的系统服务，让chkconfig指令得以管理它，并同时在系统启动的叙述文件内增加相关数据。
   --del 　删除所指定的系统服务，不再由chkconfig指令管理，并同时在系统启动的叙述文件内删除相关数据。
   --level<等级代号> 　指定读系统服务要在哪一个执行等级中开启或关毕。

  * chkconfig --list [name]：显示所有运行级系统服务的运行状态信息（on或off）。如果指定了name，那么只显示指定的服务在不同运行级的状态。
  * chkconfig --add name：增加一项新的服务。chkconfig确保每个运行级有一项启动(S)或者杀死(K)入口。如有缺少，则会从缺省的init脚本自动建立。
  * chkconfig --del name：删除服务，并把相关符号连接从/etc/rc[0-6].d删除。
  * chkconfig [--level levels] name：设置某一服务在指定的运行级是被启动，停止还是重置。

运行级文件：
每个被chkconfig管理的服务需要在对应的init.d下的脚本加上两行或者更多行的注释。第一行告诉chkconfig缺省启动的运行级以及启动和停止的优先级。如果某服务缺省不在任何运行级启动，那么使用 - 代替运行级。第二行对服务进行描述，可以用\ 跨行注释。
例如，random.init包含三行：
# chkconfig: 2345 20 80
# description: Saves and restores system entropy pool for \
# higher quality random number generation.

使用范例：
chkconfig --list        #列出所有的系统服务
chkconfig --add httpd        #增加httpd服务
chkconfig --del httpd        #删除httpd服务
chkconfig --level httpd 2345 on        #设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态
chkconfig --list        #列出系统所有的服务启动情况
chkconfig --list mysqld        #列出mysqld服务设置情况
chkconfig --level 35 mysqld on        #设定mysqld在等级3和5为开机运行服务，--level 35表示操作只在等级3和5执行，on表示启动，off表示关闭
chkconfig mysqld on        #设定mysqld在各等级为on，“各等级”包括2、3、4、5等级

如何增加一个服务：
1.服务脚本必须存放在/etc/ini.d/目录下；
2.chkconfig --add servicename
在chkconfig工具服务列表中增加此服务，此时服务会被在/etc/rc.d/rcN.d中赋予K/S入口了；
3.chkconfig --level 35 mysqld on
修改服务的默认启动等级。

参考:
http://www.cnblogs.com/panjun-Donet/archive/2010/08/10/1796873.html
http://blog.csdn.net/monkey_d_meng/article/details/5573580