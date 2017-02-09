title: linux之用户及用户组管理命令 

#  linux之用户及用户组管理命令 
参考：http://blog.csdn.net/qq1603013767/article/details/8192221

相关的管理命令汇总
用户管理相关命令
  * useradd        添加用户
  * adduser        添加用户
  * userdel         删除用户
  * passwd         为用户设置密码
  * usermod       修改用户命令，可以通过usermod 来修改登录名、用户的家目录等等
 
用户组管理相关命令
  * groupadd     添加用户组
  * groupdel      删除用户组
  * groupmod    修改用户组信息
  * groups        ** 显示用户所属的用户组**
  * newgrp        切换到相应用用户组

**增加附加群组**：
	sudo uermod -aG docker admin

##  1、增加新用户、编辑用户与删除用户 
在linux系统中增加用户的过程了吧，这个过程就是：
  * 在 ` /etc/passwd ` 里面建立一行与账号相关的数据，包括建立 UID/GID/家目录等；
  * 在 ` /etc/shadow ` 里面将此账号的密码相关参数填入，但是尚未有密码；
  * 在 ` /etc/group ` 里面加入一个与账号名称一模一样的组名；
  * 在 ` /home ` 底下建立一个与账号同名的目录作为用户家目录，且权限为 700
  * 从` /etc/skel/ `中COPY相应的文件到用户的家目录
  * 最后通过` passwd命令 `，把密码加密后写到/etc/shadow中

  * 用户账号/密码相关参数：/etc/passwd, /etc/shadow
  * 用户相关参数：/etc/group, /etc/gshadow
  * 用户个人文件数据： /home/username, /var/spool/mail/username

相关命令：useradd, passwd, usermod, userdel
###  新增用户useradd/adduser 
语法：` useradd ` [options] LOGIN
options有很多（可以用useradd –hlep 或者man useradd查看），我们简单介绍几个。

-d 目录       指定用户主目录，（默认是在/home目录下创建和用户名一样的目录）
-g 用户组    指定用户所属的用户组(主组）
-G 用户组   指定用户所属的附加组（这些组必需事先已经增加过了或者是系统中已经存在）
-s Shell      指定用户的登录Shell
-u UID        指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号
-c 描述        指定一段注释性描述
-m              使用者目录若不存在则自动建立（默认选项）
 
` userdel ` 这个命令用于删除用户，如果不加附加参数，则默认只删除用户，如果加上-r参数，则同时删除用户目录和用户帐户。
 
###  修改密码passwd 
` passwd ` 这个命令用于修改用户密码，也可以修改群组密码。常用附加参数如下：
其实在Linux系统中，命令的作用就是改配置文件，而这个` passwd命令就是把密码加密后写入/etc/shadow `（二栏）中，我们也知道这个文件中的栏位有9栏，那么其它栏要如何通过passwd这个命令来改呢？
看帮助吧
passwd –help
格式：passwd [OPTION...] <accountName>
-l  ：是Lock的意思，会将 /etc/shadow 第二栏最前面加上”!”使密码失效
-u ：与-l相对，是Unlock的意思
-g 修改组密码。
-S ：列出密码相关参数，即shadow文件的大部分信息
-n ：后面接天数，shadow的第4字段，不可修改密码天数
-x ：后面接天数，shadow的第5字段，多长时间内必须要修改密码
-w ：后面接天数，shadow的第6字段，密码过期前的警告天数
-i  ：后面接日期，shadow 的第7字段，密码失效日期
-d 关闭用户的密码认证功能，使其在登录时不需要输入密码，需要root权限。

###  usermod编辑用户 
新增用户已经搞定，但要是对用户的相关信息进行一下修改，除了通过改文件外，还有没有其他的方法了？当然有啊，命令啊，命令的最终目的也是改配置文件。那么下面就来看看如何通过usermod来修改用户的相关0信息。
[root@yufei ~]# usermod -h
Usage: usermod [options] LOGIN
-c ：后面接账号的说明，即/etc/passwd第五栏的说明栏，可以加入一些账号的说明
-d ：后面接账号的家目录，即修改/etc/passwd的第六栏
-e ：后面接日期，格式是YYYY-MM-DD也就是在/etc/shadow内的第八栏
-f  ：后面接天数，修改shadow的第七栏
-g ：**后面接主群**组，修改/etc/passwd的第四个字段，即是GID的字段
-G：**后面接附加群组**，修改这个使用者能够支持的群组，修改的是` /etc/group `
-a ：**与 -G 合用**，可**增加附加群组的支持而非设定**
-l  ：后面接账号名称。修改账号名称，/etc/passwd的第一栏
-s ：后面接Shell的文件，例如/bin/bash或/bin/csh等等
-u ：后面接 UID 数字，修改用户的UID /etc/passwd第三栏
-L ：暂时将用户的密码冻结，让他无法登入。其实就是在/etc/shadow的密码栏前面加上了“!”
-U：将/etc/shadow 密码栏的“!”去掉

其实，这个usermod和useradd的用法非常相似，只是增加了用户锁定与解锁。

###  删除用户userdel 
这个命令很简单，
[root@yufei ~]# userdel -h
Usage: userdel [options] LOGIN
  * -f ：强制删除，包括用户的一切相关内容，这个参数是危险的参数，不建议大家使用。详细说明看MAN
  * -r ：删除用户的家目录和用户的邮件池
其实这个-r参数就是删除用户的相关配置文件中的信息
  * 用户账号/密码相关参数：/etc/passwd, /etc/shadow
  * 用户相关参数：/etc/group, /etc/gshadow
  * 用户个人文件数据： /home/username, /var/spool/mail/username

##  用户组管理 
这也和上面的用户管理差不多，只是修改的文件（` /etc/group, /etc/gshadow `）不同
###  增加用户组groupadd 
[root@yufei ~]# groupadd -h
Usage: groupadd [options] GROUP
-g gid ：设置用户组，并指定相应的GID
-r ：这个参数和我们的useradd -r 是一样的道理

###  编辑用户组groupmod 
与usermod也是类似的
[root@yufei ~]# groupmod -h
Usage: groupmod [options] GROUP
-g ：修改既有的 GID 数字；
-n ：修改既有的组名

###  删除用户组groupdel 
这个命令更简单，没有任何的参数，后面直接跟上想删除的用户组名

###  用户组的管理员设置gpasswd 
[root@yufei ~]# gpasswd
Usage: gpasswd [option] GROUP
   ：没有参数，设置用户组密码
-a : 增加用户到用户组中
-d ：从用户组中删除用户
-r ：删除用户组的密码
-M ：设置用户组成员（多成员）
-A ：设置用户组管理员（列表）

 
UBUNTU 用户及用户组管理
创建组：
$sudo addgroup ccache  
 
创建用户：
$sudo useradd ccache -g ccache -M  
 
创新wfz用户并创建HOME目录，指定用户组为ccache
$sudo useradd wfz -g ccache -m  
 
**增加已存在用户到指定组**
$sudo adduser $USER ccache  
$sudo adduser dbh ccache  
$sudo adduser paul ccache  
$sudo adduser wfz ccache  
 
显示用户ID及组信息：
~$ ` id  ` 
uid=1001(dbh) gid=1001(dbh) groups=115(admin),1001(dbh)  
$ ` cat /etc/group  ` 
ccache:x:1002:dbh,paul,wfz  

##  示例，新建用户 
ubuntu下：
adduser demo
usermod -a -G sudo demo

Centos下:
adduser demo
passwd demo
usermod -a -G wheel demo

## 附录： 用户和组配置文件介绍 
（1）用户帐号文件——passwd
Passwd 是一个文本文件（每一行标识1个用户），定义了系统的用户帐号，该文件位于“/etc”目录下。文件中包含了一个系统帐户列表，存放了每个账户一些有用的 信息，如用户ID,组ID，主目录，shell等等（用“：”分隔开来）。只定义了用户帐号，而不保存口令（用“x”表示，如果没有 sun：：则表示没有密码）。真正的密码存放在Shadow文件中，普通用户根本不能读，加密后的密文无法读到就可以提高用户帐号的安全性。
例如：
[root@sun root]# head /etc/passwd  
root:x:0:0:root:/root:/bin/bash 表示有7个字段：登录名:有无口令：用户ID:组ID:账户备注信息：用户Home目录：登录时用户shell的名称（超级用户有权限修改）
 
（2）用户口令文件——shadow
每行定义了一个用户信息，行中各字段用：分开，为进一步提高安全性，口令文件存放用户已经加密的口令：*,特殊符号
[root@sun root]# head /etc/shadow  
 
登录名:加密的口令（用*或其他特殊字符表示）:上次更改口令距离1970.1.1的天数：口令更改后不可更改的天数：口令更改后必须再更改的天数（有效期）：口令失效前警告用户的天数：口令失效后距帐号被查封的天数：帐号被封时距1970.1.1的天数：保留未用。
 
（3）用户组帐号文件——group
用户组：逻辑的组织用户帐号的集合的方式，用户允许在其组内共享文件，系统每个文件都有一个用户和附属的用户组。使用“ls -l”命令可以查看每个文件的属性和组。
[root@sun root]# head /etc/group  
 
root:x:0:root,tom,mary (组名：组加密口令：GID：组成员列表（用，隔开的每个组用户名）)
 
（4）用户组口令文件——gshadow
用于定义用户组口令，用户组管理员信息。该文件只有超级用户root才可以读取
每行记录信息：
[root@sun root]# head /etc/gshadow  
 
用户组：用户组加密口令：组管理员帐号（管理员有权进行增删帐号）：组成员列表
 
2. 用户和用户组账户维护的命令：
 
（1）增加用户账户：useradd 用户名
useradd –g 组名 用户名 指定该用户所使用的私有组名，默认是与用户帐号同名的私有组。
useradd –D [-g group][-b base][-s shell][-f inactive][-e expire] 用于显示和设置useradd该命令所使用的默认值。
```

例如：#useradd sun //建立用户帐号
#tail -l /etc/passwd //查询passwd中添加的用户账户的信息
#tail –l /etc/shadow
#ls /home //查看所建立帐号的主目录
 
（2）修改用户帐号属性：usermod [-LU][-c ][-d ][-e ][-f ][-g ][-G][-l][-s][-u][用户帐号]
 
（3）删除用户帐号：userdel [-r][用户帐号] //如果不加参数则只删除用户帐号，不删除文件，否则两者都删除。
userdel [-r][用户帐号] //-r用来删除帐号登入目录和目录中所有文件
 
举例：#grep sun /etc/passwd //查询用户帐号sun是否存在
#userdel sun //删除用户帐号sun
#grep sun /etc/passwd //再次查询用户帐号sun是否存在
#ll –d /home //查询用户sun主目录是否存在
#userdel –r sun //删除用户的同时，删除其工作主目录
 
（4） 增加用户组帐号：groupadd [-r][组帐号]
【注意】帐号ID唯一，数值不可为负，预设最小值不得小于500，且每增加一个，组帐号ID逐次自增1。其中-r参数是用来建立系统帐号的。0～499是给系统帐号准备的。
 
举例：#groupadd magicSun //建立组账户magicSun
#grep magicSun /etc/group //查询group文件中magicSun组账户是否建立
#groupadd –r sysWang //建立系统组账户sysWang
#grep sysWang /etc/group //查询group文件中sysWang系统组账户是否建立

```
 
（5）修改组帐号：groupmod [-g ][-n][群组名称]
其中-o表示重复使用群组识别码
 
（6）删除组帐号：groupdel [群组名称]
【注意】必须先删除组中的用户才能删除该组
 
（7） 口令维护：passwd [-s][-l][-u][-d][用户名] 超级用户可以为每一位新增的用户设置口令，普通用户只能用不带参数的passwd命令来修改自己的口令。其中参数-s表示用于查询指定用户帐号的状 态，-l用户锁定帐号的口令，-u解锁帐号口令，-d删除指定帐号的口令。
 
（8）组用户成员维护：将一个账户添加到组、或将一个账户从组中删除、将一个账户设为组管理员。
添加用户到组：gpasswd –a 用户帐号名 组帐号名
从组中删除用户：gpasswd –d用户帐号名 组帐号名
设置用户为组管理员：gpasswd –A 组管理员用户列表 用户组
 
（9）用户和组的状态命令：
id [选项] [用户名称] 用于显示用户当前UID，gid以及所属群组的组列表
[选项]参数有：
-g :显示用户所属群组的id
-G:显示用户所属附加群组的id
-n:显示用户所属群组或附加群组的名称
-r:显示实际ID
-u:显示用户ID
whoami 用于显示登录者自身的名称(=id -un)
```

su [-flmp] [-][-c ][-s][用户帐号] //用来将当前用户转换为其他用户身份，暂时变更自己的登录身份，用其他人的身份来登录系统。前提是必须知道对方的口令。其中参数-c表示执行完指定的指 令后恢复原来的身份。-f适用于csh和tsch,使shell不用去读取启动文件。-表示改变身份时也同时变更工作目录，以及 HOME，SHELL,USER,LOGNAME，此外也会变更PATH环境变量。-m,-p 变更身份时不变更环境变量。-s 指定要执行的shell。若不指定要变更的用户账户，那么预设为root超级用户。


``` 
groups [用户名称] 用于显示指定用户所属的组，若未指定用户则显示当前用户所属组
来源：http://my.oschina.net/zhangqingcai/blog/32094
更多： https://help.ubuntu.com/13.04/serverguide/user-management.html
 
runlevel
查看当前的运行级别，Ubuntu 桌面默认是2。
runlevel
 
Ubuntu 的系统运行级别：
0        系统停机状态
1        单用户或系统维护状态
2~5      多用户状态
6        重新启动 
S
 
切换运行级别，执行命令：
init [0123456Ss]
 
即在 init 命令后跟一个参数，此参数是要切换到的运行级的运行级代号，如：用 init 0 命令关机；用 init 6 命令重新启动。
 
whois
功能说明：查找并显示用户信息。
语　　法：whois [帐号名称]
补充说明：whois指令会去查找并显示指定帐号的用户相关信息，因为它是到Network Solutions 的WHOIS数据库去查找，所以该帐号名称必须在上面注册方能寻获，且名称没有大小写的差别。

---------------------------------------------------------

whoami

功能说明：先似乎用户名称。
语　　法：whoami [--help][--version]
补充说明：显示自身的用户名称，本指令相当于执行"id -un"指令。
参　　数：
--help 　在线帮助。
--version 　显示版本信息。

---------------------------------------------------

who

功能说明：显示目前登入系统的用户信息。
语　　法：who [-Himqsw][--help][--version][am i][记录文件]
补充说明：执行这项指令可得知目前有那些用户登入系统，单独执行who指令会列出登入帐号，使用的 　　　终端机，登入时间以及从何处登入或正在使用哪个X显示器。
参　　数：
-H或--heading 　显示各栏位的标题信息列。
-i或-u或--idle 　显示闲置时间，若该用户在前一分钟之内有进行任何动作，将标示成"."号，如果该用户已超过24小时没有任何动作，则标示出"old"字符串。
-m 　此参数的效果和指定"am i"字符串相同。
-q或--count 　只显示登入系统的帐号名称和总人数。
-s 　此参数将忽略不予处理，仅负责解决who指令其他版本的兼容性问题。
-w或-T或--mesg或--message或--writable 　显示用户的信息状态栏。
--help 　在线帮助。
--version 　显示版本信息。

----------------------------------------------------

w

功能说明：显示目前登入系统的用户信息。
语　　法：w [-fhlsuV][用户名称]
补充说明：执行这项指令可得知目前登入系统的用户有那些人，以及他们正在执行的程序。单独执行w
指令会显示所有的用户，您也可指定用户名称，仅显示某位用户的相关信息。
参　　数：
-f 　开启或关闭显示用户从何处登入系统。
-h 　不显示各栏位的标题信息列。
-l 　使用详细格式列表，此为预设值。
-s 　使用简洁格式列表，不显示用户登入时间，终端机阶段作业和程序所耗费的CPU时间。
-u 　忽略执行程序的名称，以及该程序耗费CPU时间的信息。
-V 　显示版本信息。

-----------------------------------------------------

finger命令

finger命令的功能是查询用户的信息，通常会显示系统中某个用户的用户名、主目录、停滞时间、登录时间、登录shell等信息。如果要查询远程机上的 用户信息，需要在用户名后面接“@主机名”，采用[用户名@主机名]的格式，不过要查询的网络主机需要运行finger守护进程。

该命令的一般格式为：
finger [选项] [使用者] [用户@主机]
命令中各选项的含义如下：
-s 显示用户的注册名、实际姓名、终端名称、写状态、停滞时间、登录时间等信息。
-l 除了用-s选项显示的信息外，还显示用户主目录、登录shell、邮件状态等信息，以及用户主目录下的.plan、.project和.forward文件的内容。
-p 除了不显示.plan文件和.project文件以外，与-l选项相同。　

[例]在本地机上使用finger命令。
Java代码  收藏代码
$ finger xxq  
Login: xxq Name:  
Directory: /home/xxq Shell: /bin/bash  
Last login Thu Jan 1 21:43 （CST） on tty1  
No mail.  
No Plan.　  
$ finger  
Login Name Tty Idle Login Time Office Office Phone  
root root *1 28 Nov 25 09:17  
 
……

------------------------------------------------------------------

/etc/group文件包含所有组
/etc/shadow和/etc/passwd系统存在的所有用户名

修改当前用户所属组的方法
usermod 或者可以直接修改 /etc/paaawd文件即可

 
1、id 工具: 查询用户所对应的UID 和GID 及GID所对应的用户组；

id 工具是用来查询用户信息，比如用户所归属的用户组，UID 和GID等；id 用法极为简单；我们举个例子说明一下；语法格式： id [参数] [用户名]
至于有哪些参数，自己查一下 id --help 或man id ；如果id后面不接任何参数和任何用户，默认显示当前操作用户的用户名、所归属的用户组、UID和GID等；

实例一：不加任何参数和用户名；
Java代码  收藏代码
[beinan@localhost ~]$ id  
uid=500(beinan) gid=500(beinan) groups=500(beinan)  
 
注解：在没有加任何参数的情况下，查询的是当前操作用户的用户名、UID 、GID 和所处的主用户组和附属用户组；在本例中，用户名是beinan，UID是500,所归属的主用户组是beinan，GID是500 ；

**实例二： id 后面接用户名；**

如果我们想查询系统中用户的UID和GID 相应的内容，可以直接接用户名，但用户名必须是真实的 ，能在/etc/passwd中查到的；
Java代码  收藏代码
[beinan@localhost ~]$ id linuxsir  
uid=505(linuxsir) gid=502(linuxsir) groups=502(linuxsir),0(root),500(beinan)  
 
注解：查询用户linuxsir 的信息，用户linuxsir ，UID 为505，所归属的主用户组是linuxsir，主用户组的GID是502；同时linuxsir用户也是GID为0的root用户组成员，也是GID为500用户组beinan的成员；
这个例子和实例一在用户组方面有所不同，我们在 《Linux 用户（user）和用户组（group）管理概述》 中有提到；用户和用户组的对应关系，可以是一对一、一对多、多对一、或多对多的交叉关系，请参考之；另外您还需要掌握《用户（user）和用户组（group）配置文件详解》一文；
参考：
http://justcoding.iteye.com/blog/1929063