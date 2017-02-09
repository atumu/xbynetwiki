title: centos特定命令 

#  centos特定命令 
##  软件管理 
首先必须先明确一点，yum并不是一种新的软件包管理形式，我们的rpm则是一种新的软件包管理形式，yum只是rpm的一个前端程序，**yum最主要的功能就是帮助我们解决软件包的依赖性问题**！！！
yum类似于ubuntu的apt-get, rpm类似于ubuntu的dpkg

##  yum介绍 
**YUM仓库**:yum使用的是仓库来保持管理我们的rpm软件包，仓库的配置文件是存放在`  /etc/yum.repos.d/ ` 支持多个仓库,这个仓库既可以是本地的，也可以是互联网上的.
```

[root@xiaoluo home]# cd /etc/yum.repos.d/
[root@xiaoluo yum.repos.d]# ls -l
总用量 16
-rw-r--r--. 1 root root 1926 2月  25 16:57 CentOS-Base.repo #主要源文件
-rw-r--r--. 1 root root  638 2月  25 16:57 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  630 2月  25 16:57 CentOS-Media.repo
-rw-r--r--. 1 root root 3664 2月  25 16:57 CentOS-Vault.repo

```
yum仓库的配置格式：
```

[base]：代表容器的名字！中刮号一定要存在，里面的名称则可以随意取。但是不能有两个相同的容器名称， 否则 yum 会不晓得该到哪里去找容器相关软件列表档案。 
 name：只是说明一下这个容器的意思而已，重要性不高！ 
 mirrorlist=：列出这个容器可以使用的映射站台，如果不想使用，可以批注到这行； 
 baseurl=：这个最重要，因为后面接的就是容器的实际网址！ mirrorlist 是由 yum 程序自行去捉映像站台， baseurl 则是指定固定的一个容器网址！ 
 enable=1：就是让这个容器被启劢。如果不想启劢可以使用 enable=0 喔！ 
 gpgcheck=1：还记得 RPM 的数字签名吗？这就是指定是否需要查阅 RPM 档案内的数字签名！ 
 gpgkey=：就是数字签名的公钥文件所在位置！使用默认值即可

```

修改yum源：` /etc/yum.repos.d/CentOS-Base.repo `
查询仓库列表
```

yum repolist all

``` 
yum 会先下载容器的清单到本机的 ` /var/cache/yum ` 里面去，如果我们在一个容器里面修改了网址，却没有修改容器名称（中括号里面的文字），可能就会造成本机的列表与yum 服务器的列表不同步，此时就会出现无法更新的问题了。**所以需要清理一下**：
```

yum clean [packages|headers|all]
yum clean all

```

----

` yum -y ` 自动确认
安装:
yum install
使用YUM启用已存在的软件源
如果你安装了第三方的软件源，你需要先启用该软件源才能从其安装软件，输入下面的命令启用EPEL软件源： yum --enablerepo=epel install rsnapshot
使用YUM禁用软件源
如果你安装了第三方软件源但不想从其安装软件，可以用下面的命令禁用： yum --disablerepo=epel install 软件名称

YUM软件集合
YUM软件集合是指多个共同协作的软件统称，比如“Development Tools”（开发工具）。 下面介绍下怎么用yum groupinstall命令来查看/安装/卸载yum软件集合
安装yum软件集合
yum groupinstall 'Development Tools'
卸载yum软件集合
yum groupremove 'Development Tools'
升级yum软件集合
yum groupupdate 'Development Tools'
查看yum软件集合信息
yum groupinfo 'Development Tools'
查看有哪些软件集合
yum grouplist | more

----

删除
yum remove 软件名称 或者 yum erase 软件名称

----

查询：yum [list|search]
基于关键字搜索软件：yum search 关键字
列出全部的、安装的、最近的、更新的软件　　yum list (all | installed | recent | updates)　　
列出YUM仓库中全部软件：yum list all
使用YUM输出已安装软件包列表
yum list installed
显示软件信息：yum info packagename
yum deplist package1 查看程序package1依赖情况

----

包的更新
1.1)检查可更新包: yum check-update
1.2)更新所有包: yum update
1.3)更新指定包: yum update package_name
1.4)版本升级: yum upgrade

----

**更新软件源及其缓存**
[1] 首先备份/etc/yum.repos.d/CentOS-Base.repo
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
[2] 进入yum源配置文件所在文件夹
[root@localhost yum.repos.d]# cd /etc/yum.repos.d/
[3] 下载163的yum源配置文件，放入/etc/yum.repos.d/(操作前请做好相应备份)
[root@localhost yum.repos.d]# wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
**[4] 运行yum makecache生成缓存**
```

yum makecache

```

----

5 清除缓存
yum clean packages 清除缓存目录下的软件包
yum clean headers 清除缓存目录下的 headers
yum clean oldheaders 清除缓存目录下旧的 headers
yum clean, yum clean all (= yum clean packages; yum clean oldheaders) 清除缓存目录下的软件包及旧的headers

##  TARBALL 
```

 ./configure 
make clean 
make
make install

```
①./configure　　检查编译环境、相关库文件以及配置参数并生成makefile
②make　　将源代码编译成可执行的二进制文件
③make install　　安装编译好的可执行文件
##  rpm 
RPM管理包管理器**支持网络安装**和查询；（可以不用事先下载）
安装软件：rpm -ivh xiaoluo-1.1.0-5.el6.x86_64.rpm
卸载软件：rpm -e xiaoluo
升级形式安装：rpm -Uvh xiaoluo-1.1.0-5.el6.x86_64.rpm
列出所有已安装的rpm软件 rpm -qa  | grep mysql

常用命令组合：
```

－ivh：安装显示安装进度--install--verbose--hash
－Uvh：升级软件包--Update；
－qpl：列出RPM软件包内的文件信息[Query Package list]；
－qpi：列出RPM软件包的描述信息[Query Package install package(s)]；
－qf：查找指定文件属于哪个RPM软件包[Query File]；
－Va：校验所有的RPM软件包，查找丢失的文件[View Lost]；
－e：删除包


```
----

选项
```

-i, --install                     install package(s)
-v, --verbose                     provide more detailed output
-h, --hash                        print hash marks as package installs (good with -v)
-e, --erase                       erase (uninstall) package
-U, --upgrade=<packagefile>+      upgrade package(s)
－-replacepkge                    无论软件包是否已被安装，都强行安装软件包
--test                            安装测试，并不实际安装
--nodeps                          忽略软件包的依赖关系强行安装
--force                           忽略软件包及文件的冲突

Query options (with -q or --query):
-a, --all                         query/verify all packages
-p, --package                     query/verify a package file
-l, --list                        list files in package
-d, --docfiles                    list all documentation files
-f, --file                        query/verify package(s) owning file

```

----

```

rpm -q samba //查询程序是否安装

rpm -ivh  /media/cdrom/RedHat/RPMS/samba-3.0.10-1.4E.i386.rpm //按路径安装并显示进度
rpm -ivh --relocate /=/opt/gaim gaim-1.3.0-1.fc4.i386.rpm    //指定安装目录

rpm -ivh --test gaim-1.3.0-1.fc4.i386.rpm　　　 //用来检查依赖关系；并不是真正的安装；
rpm -Uvh --oldpackage gaim-1.3.0-1.fc4.i386.rpm //新版本降级为旧版本

rpm -qa | grep httpd　　　　　 ＃[搜索指定rpm包是否安装]--all搜索*httpd*
rpm -ql httpd　　　　　　　　　＃[搜索rpm包]--list所有文件安装目录

rpm -qpi Linux-1.4-6.i368.rpm　＃[查看rpm包]--query--package--install package信息
rpm -qpf Linux-1.4-6.i368.rpm　＃[查看rpm包]--file
rpm -qpR file.rpm　　　　　　　＃[查看包]依赖关系
rpm2cpio file.rpm |cpio -div    ＃[抽出文件]

rpm -ivh file.rpm 　＃[安装新的rpm]--install--verbose--hash
rpm -ivh

rpm -Uvh file.rpm    ＃[升级一个rpm]--upgrade
rpm -e file.rpm      ＃[删除一个rpm包]--erase

```

----

安装/升级:
RPM源代码包装安装
.src.rpm结尾的文件，这些文件是由软件的源代码包装而成的，用户要安装这类RPM软件包，必须使用命令：
```

rpm　--recompile　vim-4.6-4.src.rpm   ＃这个命令会把源代码解包并编译、安装它，如果用户使用命令：
rpm　--rebuild　vim-4.6-4.src.rpm　　＃在安装完成后，还会把编译生成的可执行文件重新包装成i386.rpm的RPM软件包。

```
```

rpm -ivh rpm文件或者网址 --prefix /usr/local (--prefix 指定安装路径)

```
```

rpm -vih file.rpm 注：这个是用来安装一个新的rpm 包；
rpm -Uvh file.rpm 注：这是用来升级一个rpm 包；

```
忽略依赖强制安装(不推荐)：
```

rpm -ivh file.rpm --nodeps --force
rpm -Uvh file.rpm --nodeps --force

```
 - -replacepkgs 参数是以已安装的软件再安装一次；有时没有太大的必要；
测试安装参数 - -test ，用来检查依赖关系；并不是真正的安装；
由新版本降级为旧版本，要加 - -oldpackage 参数；
为软件包指定安装目录：要加 - -relocate 参数或- -prefix；
```

rpm -Uvh --oldpackage gaim-1.3.0-1.fc4.i386.rpm
rpm -ivh --relocate /=/opt/gaim gaim-1.3.0-1.fc4.i386.rpm
rpm -ivh gaim-1.3.0-1.fc4.i386.rpm --prefix /opt/gaim

```

----

查询：
rpm -q[ali]
rpm -q 软件名（可使用通配符*）

对于已安装软件查询
**rpm -qa 列出所有已安装的rpm软件**
rpm -qC 软件名 列出该软件的一些配置文件路径等。
rpm -ql 软件名 列出该软件的所有文件
rpm -qi softname 查看软件信息
rpm -qf filename  查询指定文件属于哪个rpm包

对于rpm文件查询使用rpm -qp[options] abc.rpm
rpm -qpi 查询rpm文件的信息
rpm -qpil software.rpm  查询rpm文件包含的文件

例：
```

rpm -qa |grep gaim

```

查询一个已经安装的文件属于哪个软件包；
语法 rpm -qf 文件名
注：文件名所在的绝对路径要指出

查询已安装软件包都安装到何处；
语法：rpm -ql 软件名 或 rpm rpmquery -ql 软件名
```

rpm -ql lynx
rpmquery -ql lynx

```

查询一个已安装软件包的信息
语法格式： rpm -qi 软件名
```

rpm -qi lynx

```

查看一下已安装软件的配置文件；
语法格式：rpm -qc 软件名
```

rpm -qc lynx

```

查看一个已经安装软件的doc文档安装位置：
语法格式： rpm -qd 软件名
```

rpm -qd lynx

```

查看一下已安装软件所依赖的软件包及文件；
语法格式： rpm -qR 软件名
```

rpm -qR rpm-python

```
查询已安装软件的总结：对于一个软件包已经安装，我们可以把一系列的参数组合起来用；比如 rpm -qil ；比如：
```

rpm -qil lynx

```


----

删除： 
```

先查询出名称:rpm -qa libxml2*
rpm -e 软件名
rpm -e --allmatches libxml2-2.6.26-2.1.12 --nodeps

```
```

选项--allmatches 匹配多个，--nodeps 不检查依赖.
--allmatches是把与这个rpm包所有相匹配的rpm包全部删除掉;--nodeps是在删除时不进行依赖性的读取

```

卸载多个:
```

rpm -e `rpm -qa |grep libxml2`
rpm -e `rpm -qa |grep openoffice` `rpm -qa |grep ooobasis`

```

----

重建索引：因为某些动作，可能导致RPM 数据库`  /var/lib/rpm/ ` 内的档案破损，则需要重建RPM数据库  
```

rpm --initdb
rpm --rebuilddb

```
注：这两个参数是极为有用，有时rpm 系统出了问题，不能安装和查询，大多是这里出了问题；
----
###  导入签名 

```

[root@localhost RPMS]# rpm --import 签名文件
[root@localhost fc40]# rpm --import RPM-GPG-KEY
[root@localhost fc40]# rpm --import RPM-GPG-KEY-fedora

```
关于RPM的签名功能，详情请参见 man rpm

----

###  从rpm软件包抽取文件 
命令格式： rpm2cpio file.rpm |cpio -div
```

[root@localhost RPMS]# rpm2cpio gaim-1.3.0-1.fc4.i386.rpm |cpio -div

```
抽取出来的文件就在当用操作目录中的 usr 和etc中；
其实这样抽到文件不如指定安装目录来安装软件来的方便；也一样可以抽出文件；
为软件包指定安装目录：要加 -relocate 参数；下面的举例是把gaim-1.3.0-1.fc4.i386.rpm指定安装在 /opt/gaim 目录中；
```

[root@localhost RPMS]# rpm -ivh --relocate /=/opt/gaim gaim-1.3.0-1.fc4.i386.rpm

```
这样也能一目了然；gaim的所有文件都是安装在 /opt/gaim 中，我们只是把gaim 目录备份一下，然后卸掉gaim；这样其实也算提取文件的一点用法；

参考：http://www.cnblogs.com/xiaochaohuashengmi/archive/2011/10/08/2203153.html




    
##  服务管理 
我们对service和chkconfig两个命令都不陌生，systemctl 是管制服务的主要工具， 它整合了chkconfig 与 service功能于一体。
systemctl is-enabled iptables.service
systemctl is-enabled servicename.service #查询服务是否开机启动
systemctl enable *.service #开机运行服务
systemctl disable *.service #取消开机运行
systemctl start *.service #启动服务
systemctl stop *.service #停止服务
systemctl restart *.service #重启服务
systemctl reload *.service #重新加载服务配置文件
systemctl status *.service #查询服务运行状态
systemctl --failed #显示启动失败的服务

注：*代表某个服务的名字，如http的服务名为httpd
[root@CentOS7 ~]# yum -y install httpd
启动服务（等同于service httpd start）
systemctl start httpd.service
停止服务（等同于service httpd stop）
systemctl stop httpd.service
重启服务（等同于service httpd restart）
systemctl restart httpd.service
查看服务是否运行（等同于service httpd status）
systemctl status httpd.service
开机自启动服务（等同于chkconfig httpd on）
systemctl enable httpd.service
开机时禁用服务（等同于chkconfig httpd on）
systemctl disable httpd.service
查看服务是否开机启动 （等同于chkconfig --list）
参考：http://www.cnblogs.com/xiaoluo501395377/archive/2013/05/20/3089554.html
http://www.cnblogs.com/xiaoluo501395377/archive/2013/05/20/3089554.html
http://www.2cto.com/os/201411/348547.html
http://www.linuxidc.com/Linux/2014-11/109236.htm