title: linux2 

#  ubuntu开机启动管理之update-rc.d 
一、Linux系统主要启动步骤
　　读取 MBR 的信息，启动 Boot Manager。
　　加载系统内核，启动 init 进程， init 进程是 Linux 的根进程，所有的系统进程都是它的子进程。
　　init 进程读取 /etc/inittab 文件中的信息，并进入预设的运行级别。通常情况下 /etc/rcS.d/ 目录下的启动脚本首先被执行，然后是/etc/rcN.d/ 目录。
　　根据 /etc/rcS.d/ 文件夹中对应的脚本启动 Xwindow 服务器 xorg，Xwindow 为 Linux 下的图形用户界面系统。
　　启动登录管理器，等待用户登录。

----
在Linux系统下，一个Services的启动、停止以及重启通常是通过/etc/init.d目录下的脚本来控制的。然而，在启动或改变运行级别时，是在/etc/rcX.d中来搜索脚本。其中X是运行级别的number。
如果没有启用开机自启，手动执行指令：
```

/etc/init.d/apache2 start
#或者
service apache2 start

```

----


Ubuntu使用` update-rc.d `命令代替` chkconfig `命令即可完成开机启动管理.
使用命令“ update-rc.d -f mysql remove “可移除mysql的自启动服务。
使用”update-rc.d mysql defaults“可以将mysql添加到随机启动项里。
可以使用”update-rc.d mysql defaults 90“指定服务启动的顺序，数字越小，启动顺序越靠前。
update-rc.d mysql defaults 90 80 指定服务启动顺序为90，关闭顺序为80


----

**删除一个服务**
如果你想手动的完全禁用Apache2服务，你需要删除其中的所有在/etc/rcX.d中的单一链路。但是如果使用update-rc.d，则非常简单： 
update-rc.d -f apache2 remove
参数-f是强制删除符号链接，即使/etc/init.d/apache2仍然存在。 Note：这个命令仅仅禁止该服务，直到该服务被升级。如果你想在服务升级后仍然保持被禁用。应该执行如下的命令：
update-rc.d apache2 stop 80 0 1 2 3 4 5 6 .

----
**增加一个服务**
如果你想重新添加这个服务并让它开机自动执行，你需要执行以下命令： 
update-rc.d apache2 defaults
并且可以指定该服务的启动顺序：
update-rc.d apache2 defaults 90
还可以更详细的控制start与kill顺序：
update-rc.d apache2 defaults 20 80
其中前面的20是start时的运行顺序级别，80为kill时的级别。也可以写成：
update-rc.d apache2 start 20 2 3 4 5 . stop 80 0 1 6 .
其中0～6为运行级别。 update-rc.d命令不仅适用Linux服务，编写的脚本同样可以用这个命令设为开机自动运行

----

**指定开机运行级别：**
Ubuntu中的运行级别
　　0(关闭系统)
　　1(单用户模式，只允许root用户对系统进行维护。)
　　2 到 5(多用户模式，其中3为字符界面，5为图形界面。)
　　6(重启系统)
例如：init 0 命令关机; init 6 命令重新启动
实例：update-rc.d apachectl start 20 2 3 4 5 . stop 20 0 1 6 .
解析：表示在2、3、4、5这五个运行级别中，由小到大，第20个开始运行apachectl;在 0 1 6这3个运行级别中，第20个关闭apachectl。这是合并起来的写法，` 注意它有2个点号，最后还有一个 `，效果等于下面方法：
代码如下:
```

update-rc.d apachectl defaults

```

----

**决相互依赖关系的启动**
下面这段shell命令说明如何解决相互依赖关系的启动：A启动后B才能启动，B关闭后A才关闭
```

　　update-rc.d A defaults 80 20
　　update-rc.d B defaults 90 10

```
参考:
http://jingyan.baidu.com/article/ff42efa904a9d7c19e22029d.html
http://www.linuxidc.com/Linux/2013-01/77553.htm
http://www.jb51.net/os/Ubuntu/182768.html
http://www.jb51.net/os/Ubuntu/45289.html
http://blog.sina.com.cn/s/blog_79159ef50100z1ax.html