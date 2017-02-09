title: linux-java交互同步文件 

#  linux-java交互文件推送与同步 
**场景**
A,B,C分别为三台服务器,A服务器上的CMS系统生成站点文件,需要定时推送到B,C等多台客户机上.而客户机上站点文件一旦被黑客串改,主动报告A服务器,再有A服务器进行校验确认是否真的发生恶意改动.一旦确认,便强制推送文件到B客户机上.
说明:
A服务器ubuntu系统下:使用rsync+sersync实现定时单向文件增量推送
客户机java端:通过定时md5校验,请求A服务器比对MD5,然后由A服务器决定是否调用rsync强制推送.(在定时推送之外的实时强制推送).
A服务器CMS系统不断生成或更新站点相关文件.

----
**Rsync+Inotify-tools与Rsync+sersync这两种架构有什么区别？**
1、Rsync+Inotify-tools
（1）：Inotify-tools只能记录下被监听的目录发生了变化（包括增加、删除、修改），并没有把具体是哪个文件或者哪个目录发生了变化记录下来；
（2）：rsync在同步的时候，并不知道具体是哪个文件或者哪个目录发生了变化，每次都是对整个目录进行同步，当数据量很大时，整个目录同步非常耗时（rsync要对整个目录遍历查找对比文件），因此，效率很低。

2、Rsync+sersync
（1）：sersync可以记录下被监听目录中发生变化的（包括增加、删除、修改）具体某一个文件或某一个目录的名字；
（2）：rsync在同步的时候，只同步发生变化的这个文件或者这个目录（每次发生变化的数据相对整个同步目录数据来说是很小的，rsync在遍历查找比对文件时，速度很快），因此，效率很高。
小结：当同步的目录数据量不大时，建议使用Rsync+Inotify-tools；` 当数据量很大（几百G甚至1T以上）、文件很多时，建议使用Rsync+sersync。 `


**为什么要用Rsync+sersync架构?**
1、sersync是基于Inotify开发的，类似于Inotify-tools的工具
2、sersync可以记录下被监听目录中发生变化的（包括增加、删除、修改）具体某一个文件或某一个目录的名字，然后使用rsync同步的时候，只同步发生变化的这个文件或者这个目录。

----

##  A服务器上配置rsync+sersync 
ubuntu默认已经安装了**rsync**. ` rsyncd默认端口为873 `
A服务器相当于主服务器或者源服务器.

**1、关闭SELINUX**
```

vi /etc/selinux/config  #编辑防火墙配置文件
#SELINUX=enforcing  #注释掉
#SELINUXTYPE=targeted  #注释掉
SELINUX=disabled  #增加
:wq!  #保存退出
setenforce 0   #立即生效

```

----

**安装[inotify-tools](https://github.com/rvoicilas/inotify-tools/wiki):** 
```

apt-get install inotify-tools

```
1、查看服务器内核是否支持inotify
ll /proc/sys/fs/inotify   #列出文件目录，出现下面的内容，说明服务器内核支持inotify
-rw-r--r-- 1 root root 0 Mar  7 02:17 max_queued_events
-rw-r--r-- 1 root root 0 Mar  7 02:17 max_user_instances
-rw-r--r-- 1 root root 0 Mar  7 02:17 max_user_watches
备注：Linux下支持inotify的内核最小为2.6.13，可以输入命令：uname -a查看内核

2、修改inotify默认参数（inotify默认内核参数值太小）
查看系统默认参数值：
sysctl -a | grep max_queued_events
结果是：fs.inotify.max_queued_events = 16384
sysctl -a | grep max_user_watches
结果是：fs.inotify.max_user_watches = 8192
sysctl -a | grep max_user_instances
结果是：fs.inotify.max_user_instances = 128

` 修改参数： `
```

sysctl -w fs.inotify.max_queued_events="99999999"
sysctl -w fs.inotify.max_user_watches="99999999"
sysctl -w fs.inotify.max_user_instances="65535"

vi /etc/sysctl.conf #添加以下代码
fs.inotify.max_queued_events=99999999
fs.inotify.max_user_watches=99999999
fs.inotify.max_user_instances=65535
:wq! #保存退出

```

参数说明：
  * max_queued_events：inotify队列最大长度，如果值太小，会出现"** Event Queue Overflow **"错误，导致监控文件不准确
  * max_user_watches：要同步的文件包含多少目录，可以用：find /home/www.osyunwei.com -type d | wc -l 统计，必须保证max_user_watches值大于统计结果（这里/home/www.osyunwei.com为同步文件目录）
  * max_user_instances：每个用户创建inotify实例最大值

----

**安装与配置[sersync](https://github.com/wsgzao/sersync)**
```

#安装sersync
cd ~
wget https://codeload.github.com/wsgzao/sersync/zip/master
tar zxf sersync2.5.4_64bit_binary_stable_final.tar.gz
mv /app/local/GNU-Linux-x86/ /app/local/sersync
cd /app/local/sersync
#配置下密码文件，因为这个密码是要访问服务器B需要的密码和上面服务器B的密码必须一致
echo "1234" > /app/local/sersync/user.pass
#修改权限
chmod 600 /app/local/sersync/user.pass
#修改confxml.conf
vi /app/local/sersync/confxml.xml

```
```

<?xml version="1.0" encoding="ISO-8859-1"?>
<head version="2.5">
 <host hostip="localhost" port="8008"></host>
 <debug start="true"/>
 <fileSystem xfs="false"/>
 <filter start="false">
 <exclude expression="(.*)\.php"></exclude>
 <exclude expression="^data/*"></exclude>
 </filter>
 <inotify>
 <delete start="true"/>
 <createFolder start="true"/>
 <createFile start="false"/>
 <closeWrite start="true"/>
 <moveFrom start="true"/>
 <moveTo start="true"/>
 <attrib start="false"/>
 <modify start="false"/>
 </inotify>

 <sersync>
 <localpath watch="/home/"> <!-- 这里填写服务器A要同步的文件夹路径-->
 <remote ip="10.1.1.100" name="share2"/> <!-- 这里填写服务器B的IP地址和模块名(此处模块名与B服务器上rsyncd.conf文件中的某个模块名对应)-->
 <!--<remote ip="192.168.28.39" name="tongbu"/>-->
 <!--<remote ip="192.168.28.40" name="tongbu"/>-->
 </localpath>
 <rsync>
 <commonParams params="-artuz"/>
 <auth start="true" users="test" passwordfile="/app/local/sersync/user.pass"/> <!-- rsync用户test及其密码文件 这里填写服务器B的认证信息,start=true代表显式作为命令行选项-->
 <userDefinedPort start="true" port="874"/><!-- port=874, 默认端口为873 -->
 <timeout start="false" time="100"/><!-- timeout=100 -->
 <ssh start="false"/>
 </rsync>
 <failLog path="/tmp/rsync_fail_log.sh" timeToExecute="60"/><!--default every 60mins execute once--><!-- 修改失败日志记录（可选）-->
 <crontab start="true" schedule="600"><!--600mins,执行间隔-->
 <crontabfilter start="false">
 <exclude expression="*.php"></exclude>
 <exclude expression="info/*"></exclude>
 </crontabfilter>
 </crontab>
 <plugin start="false" name="command"/>
 </sersync>

 <!-- 下面这些有关于插件你可以忽略了 -->
 <plugin name="command">
 <param prefix="/bin/sh" suffix="" ignoreError="true"/> <!--prefix /opt/tongbu/mmm.sh suffix-->
 <filter start="false">
 <include expression="(.*)\.php"/>
 <include expression="(.*)\.sh"/>
 </filter>
 </plugin>
</head>

```

` 注意:如果需要同步多个文件夹,那么可以配置多个xml配置文件,然后启动多个sersync2的进程来实现. `

----

**启动**
```

#运行sersync
nohup /app/local/sersync/sersync2 -r -d -o /app/local/sersync/confxml.xml >/app/local/sersync/rsync.log 2>&1 &
nohup /app/local/sersync/sersync2 -r -d -o /app/local/sersync/img.xml >/app/local/sersync/img.log 2>&1 &

-d:启用守护进程模式
-r:在监控前，将监控目录与远程主机用rsync命令推送一遍
-n: 指定开启守护线程的数量，默认为10个
-o:指定配置文件，默认使用confxml.xml文件

```

再进一步就是把sersync添加进开机自启
```

echo "/app/local/sersync/sersync2 -r -d -o /app/local/sersync/confxml.xml"  >> /etc/rc.local

```

命令选项:
```

/usr/local/sersync/bin/sersync -help

```
  * 参数-d:启用守护进程模式
  * 参数-r:在监控前，将监控目录与远程主机用rsync命令推送一遍
  * c参数-n: 指定开启守护线程的数量，默认为10个
  * 参数-o:指定配置文件，默认使用confxml.xml文件
  * 参数-m:单独启用其他模块，使用 -m refreshCDN 开启刷新CDN模块
  * 参数-m:单独启用其他模块，使用 -m socket 开启socket模块
  * 参数-m:单独启用其他模块，使用 -m https 开启https模块
  * 不加-m参数，则默认执行同步程序

关于sersync的插件使用
当plugin标签设置为true时候，在同步文件或路径到远程服务器之后，会调用插件。通过name参数指定需要执行的插件。目前支持的有command、refreshCDN、socket、https四种插件。其中，https插件目前由于兼容性原因已经去除，以后会重新加入。
还可参考:https://www.cnhzz.com/rsync_sersync/
##  B服务器配置 
ubuntu默认已经安装并开机自动rsyncd. 默认端口873. 
B服务器相当于目标服务器或者从服务器,它上面需要安装rsync服务端软件.
1、关闭SELINUX
```

vi /etc/selinux/config #编辑防火墙配置文件
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq! #保存，退出
setenforce 0 #立即生效

```
2、开启防火墙tcp 873端口（Rsync默认端口）
```

vi /etc/sysconfig/iptables #编辑防火墙配置文件
-A INPUT -m state --state NEW -m tcp -p tcp --dport 873 -j ACCEPT
:wq! #保存退出
/etc/init.d/iptables restart #最后重启防火墙使配置生效

```

----

首先要选择服务器启动方式
  * 对于负荷较重的 rsync 服务器应该使用独立运行方式
  * 对于负荷较轻的 rsync 服务器可以使用 xinetd 运行方式

**以 xinetd 运行 rsync 服务**
在 /etc/xinetd.d/rsync。
```

service rsync
{
    disable = no
    socket_type = stream
    wait = no
    user = root
    server = /usr/bin/rsync
    server_args = --daemon
    log_on_failure += USERID
}

```
要配置以 xinetd 运行的 rsync 服务需要执行如下的命令：
```

# chkconfig rsync on
# service xinetd restart

```

----

**独立运行 rsync 服务**
最简单的独立运行 rsync 服务的方法是执行如下的命令：
```

/usr/bin/rsync --daemon

```
您可以将上面的命令写入 **/etc/rc.local** 文件以便在每次启动服务器时运行 rsync 服务。当然，您也可以写一个脚本在开机时自动启动 rysnc 服务。

----
创建配置文件 ` rsyncd.conf `
对于非匿名访问的 rsync 服务器还要创建**认证口令文件**
```

#设置rsync的配置文件
vi /etc/rsyncd.conf

#服务器B上的rsyncd.conf文件内容
uid=root
gid=root
#最大连接数
max connections=36000
#默认为true，修改为no，增加对目录文件软连接的备份 
use chroot=no
#定义日志存放位置
log file=/var/log/rsyncd.log
#忽略无关错误
ignore errors = yes
#设置rsync服务端文件为读写权限
read only = no 
#认证的用户名与系统帐户无关在认证文件做配置，如果没有这行则表明是匿名
auth users = test
#密码认证文件，格式(虚拟用户名:密码）
secrets file = /etc/rsync.pass
#这里是认证的模块名，在client端需要指定，可以设置多个模块和路径
[share2]
#自定义注释
comment  = rsync
#同步到B服务器的文件存放的路径
path=/app/data/site/
[img]
comment  = img
path=/app/data/site/img


#创建rsync认证文件  可以设置多个，每行一个用户名:密码，注意中间以“:”分割
echo "test:1234" > /etc/rsync.pass

#设置文件所有者读取、写入权限
chmod 600 /etc/rsyncd.conf  
chmod 600 /etc/rsync.pass  

#启动服务器B上的rsync服务
#rsync --daemon -v
rsync --daemon

#监听端口873
netstat -an | grep 873
lsof -i tcp:873

COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rsync   31445 root    4u  IPv4 443872      0t0  TCP *:rsync (LISTEN)
rsync   31445 root    5u  IPv6 443873      0t0  TCP *:rsync (LISTEN)


#设置rsync为服务启动项（可选）
echo "/usr/local/bin/rsync --daemon" >> /etc/rc.local

#要 Kill rsync 进程，不要用 kill -HUP {PID} 的方式重启进程，以下3种方式任选
#ps -ef|grep rsync|grep -v grep|awk '{print $2}'|xargs kill -9
#cat /var/run/rsyncd.pid | xargs kill -9
pkill rsync
#再次启动
/usr/local/bin/rsync --daemon

```
7、启动rsync
```

/etc/init.d/xinetd start  #启动
service xinetd stop   #停止
service xinetd restart  #重新启动

```



##  试验 
主动使用rsync命令类似如下:
```

cd /opt/webroot/qww && /usr/bin/rsync -artuz -R --delete ./  --port=22210 testrsync@10.20.20.181::share2 --password-file=/etc/passwd.txt

``` 

采用sersync自动同步类似如下:
```

cd /opt/sersync && sudo ./sersync2 -d -r -o confxml.xml

```



----
##  rsync参数 


```

-v, --verbose 详细模式输出

-q, --quiet 精简输出模式

-c, --checksum 打开校验开关，强制对文件传输进行校验

-a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD

-r, --recursive 对子目录以递归模式处理

-R, --relative 使用相对路径信息

-b, --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。

--backup-dir 将备份文件(如~filename)存放在在目录下。

-suffix=SUFFIX 定义备份文件前缀

-u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件)

-l, --links 保留软链结

-L, --copy-links 想对待常规文件一样处理软链结

--copy-unsafe-links 仅仅拷贝指向SRC路径目录树以外的链结

--safe-links 忽略指向SRC路径目录树以外的链结

-H, --hard-links 保留硬链结

-p, --perms 保持文件权限

-o, --owner 保持文件属主信息

-g, --group 保持文件属组信息

-D, --devices 保持设备文件信息

-t, --times 保持文件时间信息

-S, --sparse 对稀疏文件进行特殊处理以节省DST的空间

-n, --dry-run现实哪些文件将被传输

-W, --whole-file 拷贝文件，不进行增量检测

-x, --one-file-system 不要跨越文件系统边界

-B, --block-size=SIZE 检验算法使用的块尺寸，默认是700字节

-e, --rsh=COMMAND 指定使用rsh、ssh方式进行数据同步

--rsync-path=PATH 指定远程服务器上的rsync命令所在路径信息

-C, --cvs-exclude 使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件

--existing 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件

--delete 删除那些DST中SRC没有的文件

--delete-excluded 同样删除接收端那些被该选项指定排除的文件

--delete-after 传输结束以后再删除

--ignore-errors 及时出现IO错误也进行删除

--max-delete=NUM 最多删除NUM个文件

--partial 保留那些因故没有完全传输的文件，以是加快随后的再次传输

--force 强制删除目录，即使不为空

--numeric-ids 不将数字的用户和组ID匹配为用户名和组名

--timeout=TIME IP超时时间，单位为秒

-I, --ignore-times 不跳过那些有同样的时间和长度的文件

--size-only 当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间

--modify-window=NUM 决定文件是否时间相同时使用的时间戳窗口，默认为0

-T --temp-dir=DIR 在DIR中创建临时文件

--compare-dest=DIR 同样比较DIR中的文件来决定是否需要备份

-P 等同于 --partial

--progress 显示备份过程

-z, --compress 对备份的文件在传输时进行压缩处理

--exclude=PATTERN 指定排除不需要传输的文件模式

--include=PATTERN 指定不排除而需要传输的文件模式

--exclude-from=FILE 排除FILE中指定模式的文件

--include-from=FILE 不排除FILE指定模式匹配的文件

--version 打印版本信息

--address 绑定到特定的地址

--config=FILE 指定其他的配置文件，不使用默认的rsyncd.conf文件

--port=PORT 指定其他的rsync服务端口

--blocking-io 对远程shell使用阻塞IO

-stats 给出某些文件的传输状态

--progress 在传输时现实传输过程

--log-format=formAT 指定日志文件格式

--password-file=FILE 从FILE中得到密码

--bwlimit=KBPS 限制I/O带宽，KBytes per second

-h, --help 显示帮助信息

```


----

##  参考文章 
[Linux下Rsync+sersync实现数据实时同步](http://www.osyunwei.com/archives/7447.html)
[用 sersync 实现多个不同目录向多个节点实时同步](http://blog.csdn.net/hzcyclone/article/details/7421902)
[Rsync命令参数详解](http://www.jb51.net/article/34869.htm)
[基于rsync+sersync的服务器文件同步实战](https://www.markdream.com/technologies/server/syncfile-by-rsync.shtml)
[inotify-tools](https://github.com/rvoicilas/inotify-tools/wiki)
[wsgzao/sersync github](https://github.com/wsgzao/sersync)
[使用 rsync 服务(二)](http://blog.csdn.net/david_xtd/article/details/10149617)
