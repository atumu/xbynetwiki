title: linux_nfs磁盘共享 

#  linux nfs网络磁盘共享 
NFS--Network File System :Unix/linux系统间的文件共享，主要用于局域网。
可以将NFS服务器共享的目录挂载到本地，然后像访问本地目录一样使用。
##  NFS服务简介 
　　NFS 是Network File System的缩写，即网络文件系统。一种使用于分散式文件系统的协定，由Sun公司开发，于1984年向外公布。功能是通过网络让不同的机器、不同的操作系统能够彼此分享个别的数据，让应用程序在客户端通过网络访问位于服务器磁盘中的数据，**是在类Unix系统间实现磁盘文件共享的一种方法。**
　　NFS 的基本原则是“**容许不同的客户端及服务端通过一组RPC分享相同的文件系统**”，它是独立于操作系统，容许不同硬件及操作系统的系统共同进行文件的分享。
　　NFS在文件传送或信息传送过程中**依赖于RPC协议**。RPC，远程过程调用 (Remote Procedure Call) 是能使客户端执行其他系统中程序的一种机制。NFS本身是没有提供信息传输的协议和功能的，但NFS却能让我们通过网络进行资料的分享，这是因为NFS使用了一些其它的传输协议。而这些传输协议用到这个RPC功能的。可以说NFS本身就是使用RPC的一个程序。或者说NFS也是一个RPC SERVER。所以只要用到NFS的地方都要启动RPC服务，不论是NFS SERVER或者NFS CLIENT。这样SERVER和CLIENT才能通过RPC来实现PROGRAM PORT的对应。可以这么理解RPC和NFS的关系：**NFS是一个文件系统，而RPC是负责负责信息的传输。**


0. 环境说明
nfs服务端系统：CentOS 6.4 x86_64
nfs服务端IP：192.168.4.211
nfs客户端系统：CentOS 6.4 x86_64
nfs客户端IP：192.168.4.212
一.安装NFS服务
1.检测是否安装过
rpm -qa | grep nfs  #NFS服务

##  1. 安装NFS服务端 
（192.168.4.211）
###  NFS服务器的配置 
NFS服务器的配置相对比较简单，只需要在相应的配置文件中进行设置，然后启动NFS服务器即可。
NFS的常用目录
/etc/exports                           NFS服务的主要配置文件
/usr/sbin/exportfs                   NFS服务的管理命令
/usr/sbin/showmount              客户端的查看命令
/var/lib/nfs/etab                      记录NFS分享出来的目录的完整权限设定值
/var/lib/nfs/xtab                      记录曾经登录过的客户端信息
NFS服务的配置文件为 /etc/exports，这个文件是NFS的主要配置文件，不过系统并没有默认值，所以这个文件不一定会存在，可能要使用vim手动建立，然后在文件里面写入配置内容。
```

/etc/exports文件内容格式：
<输出目录> [客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他）]

```
a. 输出目录：
输出目录是指NFS系统中需要共享给客户机使用的目录；

b. 客户端：
客户端是指网络中可以访问这个NFS输出目录的计算机
客户端常用的指定方式

指定ip地址的主机：192.168.0.200
指定子网中的所有主机：192.168.0.0/24 192.168.0.0/255.255.255.0
指定域名的主机：david.bsmart.cn
指定域中的所有主机：*.bsmart.cn
所有主机：*

c. 选项：
选项用来设置输出目录的访问权限、用户映射等。
NFS主要有3类选项：

访问权限选项
设置输出目录只读：ro
设置输出目录读写：rw

用户映射选项
all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；
no_all_squash：与all_squash取反（默认设置）；
root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；
no_root_squash：与rootsquash取反；
anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；
anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；

其它选项
secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）；
insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；
no_wdelay：若有写操作则立即执行，应与sync配合使用；
subtree：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；
no_subtree：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；


Step-1：安装nfs-utils和rpcbind，运行以下命令：
```

yum install -y nfs-utils rpcbind

```
上述命令将安装rpcbind服务和nfs服务。
 
Step-2：为NFS指定固定端口，(默认有些是动态端口导致被防火墙拦截)。运行以下命令：
```

vi /etc/sysconfig/nfs

```


NFS 用到的服务有 portmapper nfs rquotad nlockmgr mountd
其中 portmapper nfs 服务端口是固定的分别是 111 2049
另外 rquotad nlockmgr mountd 服务端口是随机的。由于端口是随机的，这导致防火墙无法设置。
这时需要配置/etc/sysconfig/nfs 使 rquotad nlockmgr mountd 的端口固定。
找到以下几项，将前面的#号去掉。
RQUOTAD_PORT=875
LOCKD_TCPPORT=32803
LOCKD_UDPPORT=32769
MOUNTD_PORT=892

Step-3：开放防火墙中的上述端口，运行以下命令：
通过命令 rpcinfo -p 可查看nfs使用的端口
从centos7开始默认使用的是firewall作为防火墙，这里改为iptables防火墙。
1、关闭firewall：
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
2、安装iptables防火墙
yum install iptables-services #安装
vi /etc/sysconfig/iptables #编辑防火墙配置文件
systemctl restart iptables.service #最后重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动

二、关闭SELINUX
vi /etc/selinux/config
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq! #保存退出
setenforce 0 #使配置立即生效
```

#service nfs restart
设置防火墙
iptables -I INPUT -s 192.168.0.0/24 -p tcp --dport 111 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p tcp --dport 875 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p tcp --dport 2049 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p tcp --dport 32769 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p tcp --dport 32803 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p tcp --dport 892 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p udp --dport 111 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p udp --dport 875 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p udp --dport 2049 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p udp --dport 32769 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p udp --dport 32803 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/24 -p udp --dport 892 -j ACCEPT

```
查看IPTABLES
# iptables -L
保存IPTABLES
# iptables-save > /etc/sysconfig/iptables
 
Step-4：设置SELinux为许可状态，运行以下命令：
vi /etc/selinux/config

将上述文件中的
SELINUX=enforcing
替换为
SELINUX=permissive

保存上述文件之后，运行以下命令：
setenforce 0
 
Step-5：创建共享目录，运行以下命令：
mkdir -p /data/nfs_share

上述命令将建立共享目录/data/nfs_share。
 
Step-6：配置exports文件，运行以下命令：
vi /etc/exports

在上述文件的末尾新增一行，如下所示：
/data/nfs_share 192.168.4.212(rw,sync,no_root_squash)
/data/nfs_share *(ro)

这一行表示只有192.168.4.212客户端能够以读写权限挂载共享目录，其他客户端只能以只读权限挂载。
 
` 输出共享目录：exportfs -rv `
Step-7：启动NFS相关服务，运行以下命令：
参考：http://blog.csdn.net/wangmj518/article/details/42216953
http://www.crifan.com/centos_7_fstab_add_auto_mount_but_fail_to_run_nfs/
说明：启用服务就是在当前 runlevel 的配置文件目录/etc/systemd/system/multi-user.target.wants/里，建立/usr/lib/systemd/system里面对应服务配置文件的软链接;禁用服务就是删除此软链接
```

sudo systemctl enable nfs-server.service  #/usr/lib/systemd/system/nfs-server.service
sudo systemctl enable rpcbind.service
sudo systemctl start rpcbind.service (注意顺序问题)
sudo systemctl start nfs-server.service

```
 要先保证rpcbind是开启状态，检测是否开启：pstree | grep rpcbind
Step-8：检查NFS的相关端口是否已经启用，运行以下命令：
service iptables status
rpcinfo -p localhost

4.检测服务
rpcinfo -p [ip]    #客户端上运行该命令，用于验证NFSServer是否正常

参考：http://www.blogjava.net/envoydada/archive/2012/03/14/371875.html
http://linux.it.net.cn/CentOS/fast/2015/0110/11567.html
##  配置服务 
vi /etc/exports  #不存在就创建
格式：共享目录  允许访问主机（权限）
示例：/public                        #允许所有主机访问
 /website  192.168.247.131(ro)  #允许192.168.247.131只读访问/website
访问主机：
可以使用完整的 IP 或者是网域，例如 192.168.100.10 或 192.168.100.0/24 ，或192.168.100.0/255.255.255.0 都可以接受！
 
也可以使用主机名称，但这个主机名称必须要在 /etc/hosts 内，或可使用 DNS 找到该名称才行啊！反正重点是可找到 IP 就是了。如果是主机名称的话，那麼他可以支援万用字元，例如 * 或 ? 均可接受。
```

 权限：man exports
ro          只读访问  
rw          读写访问  
sync        所有数据在请求时写入共享  
async       NFS在写入数据前可以相应请求  
secure      NFS通过1024以下的安全TCP/IP端口发送  
insecure    NFS通过1024以上的端口发送  
wdelay      如果多个用户要写入NFS目录，则归组写入（默认）  
no_wdelay   如果多个用户要写入NFS目录，则立即写入，当使用async时，无需此设置。  
hide        在NFS共享目录中不共享其子目录  
no_hide     共享NFS目录的子目录  
subtree_check           如果共享/usr/bin之类的子目录时，强制NFS检查父目录的权限（默认）  
no_subtree_check        和上面相对，不检查父目录权限  
all_squash  共享文件的UID和GID映射匿名用户anonymous，适合公用目录。  
no_all_squash           保留共享文件的UID和GID（默认）  
root_squash root用户的所有请求映射成如anonymous用户一样的权限（默认）  
no_root_squash          root用户具有根目录的完全管理访问权限  
anonuid=xxx 指定NFS服务器/etc/passwd文件中匿名用户的UID  
anongid=xxx 指定NFS服务器/etc/passwd文件中匿名用户的GID

``` 
 网段示例：
 /opt   192.168.133.0/255.255.255.0(rw,no_root_squash,sync) #133.0~133.255
 /nfs/web 192.168.133.0/255.255.255.0(rw,no_root_squash,sync) *(ro) #133.0~133.255可写，其它网段只读
/nfsdir  *(rw,sync,no_wdelay,anonuid=0,anongid=0,no_subtree_check) #所有客户端都可以访问

实战示例：只允许特定的两个IP访问，并且必须使用UID=9999和GID=9999的用户才能访问[这里root用户会自动转换成匿名用户权限，除非配置了no_root_squash]
/nfsdir  192.168.3.111(rw,sync,no_wdelay,anonuid=9999,anongid=9999,no_subtree_check)
/nfsdir  192.168.3.222(rw,sync,no_wdelay,anonuid=9999,anongid=9999,no_subtree_check)
所以需要在NFS服务器和客户端上分别创建该用户，这里用户名为nfsuser，组名也为nfsuser：
groupadd nfsuser -g 9999
useradd nfsuser -u 9999 -g 9999
之后将共享的目录/nfsdir授予nfsuser管理：
chmod -R 775 /nfsdir    #如果要授权给其它用户或组，可以使用ACL授权
chown -R nfsuser.nfsuser /nfsdir
` **输出共享目录格式：exportfs -rv** ` 

##  2. 安装NFS客户端 
（192.168.4.212）
NFS客户端不需要启动NFS服务，但需要安装nfs-utils，运行以下命令：
```

yum install -y nfs-utils

```
 
##  3. 手动挂载NFS共享目录 
Step-1：确定挂载点，运行以下命令：
showmount -e 192.168.4.211

-e选项显示NFS服务端的导出列表。
 
Step-2：创建挂载目录，运行以下命令：
mkdir -p /root/remote_dir

其中，/root/remote_dir为共享目录的挂载点目录。
 
Step-3：挂载共享目录，运行以下命令：
mount -t nfs 192.168.4.211:/data/nfs_share /root/remote_dir

其中，-t选项用于指定文件系统的类型为nfs。
 
Step-4：共享目录使用结束之后，卸载共享目录，运行以下命令：
umount /root/remote_dir


##  4. 开机自动挂载 
向fstab文件中添加共享目录的挂载条目，即可实现开机自动挂载，但是随后与NFS服务端的连接将始终处于活动状态。运行以下命令：
mkdir -p /root/remote_dir
vi /etc/fstab

在上述文件末尾加入共享目录的挂载条目，如下所示：
192.168.4.211:/data/nfs_share /root/remote_dir nfs defaults 0 0

其中，第5个字段设置为0表示共享目录的文件系统不需要使用dump命令进行转储，第6个字段设置为0表示共享目录的文件系统不需要使用fsck命令进行检查。
 
除此之外，还可以使用自动挂载器（autofs）实现按需自动挂载网络共享目录。当共享不再使用，并处于不活动状态一定时间之后，自动挂载器会对共享解除挂载。
 

4.查看是否挂载成功
df -h
 
5.卸载NFS挂载的共享目录
umount /mnt/website
 
6.开机自动挂载NFS共享
vi /etc/fstab
```

格式：NFS共享目录               本机挂载点  文件系统  权限  是否检测  检测顺序
      192.168.247.130:/website /mnt/website   nfs      rw      0         0

```
或者：
vi /etc/rc.d/rc.local #这个是启动系统时最后一个执行的程序
加入：mount -t nfs 192.168.247.130:/website /mnt/website
-t :指定挂载的文件类型，这里是nfs
如要马上生效：命令行/etc/rc.d/rc.local 回车即可


##  5. 按需自动挂载（特殊映射） 
 
当autofs服务运行时，系统中存在一个名为/net的特殊目录，但是该目录将显示为空。NFS客户端通过特殊映射实现按需自动挂载共享目录的步骤如下所示：
 
Step-1：修改不活动状态的超时时间，运行以下命令：
vi /etc/sysconfig/autofs

将上述文件中的
TIMEOUT=300
替换为
TIMEOUT=600
也就是将不活动状态的超时时间由5分钟修改为10分钟。

配置完成之后，重启autofs服务：
service autofs restart
 
Step-2：访问网络共享目录，运行以下命令：
cd /net/192.168.4.211/data/nfs_share

运行上述命令时，autofs会自动挂载NFS服务端中的网络共享目录。
 
Step-3：卸载已挂载的网络共享目录，详情如下所示：
在/net/192.168.4.211/data/nfs_share之下的所有文件和目录停止使用且超时期满之后（10分钟），autofs将卸载共享目录。
 
##  6. 按需自动挂载（间接映射） 
 
Step-1：修改不活动状态的超时时间，运行以下命令：
vi /etc/sysconfig/autofs

将上述文件中的
TIMEOUT=300
替换为
TIMEOUT=600
也就是将不活动状态的超时时间由5分钟修改为10分钟。
 
Step-2：建立共享目录挂载点的父目录，运行以下命令：
mkdir -p /root/demo
 
Step-3：配置共享目录挂载点的父目录，运行以下命令：
vi /etc/auto.master

上述文件的内容如下所示：
/root/demo    /etc/auto.demo

其中，/root/demo是挂载点的父目录，这个目录在系统中始终可见，并由autofs服务监控，以确定是否“需要”挂载/创建子目录挂载点。/etc/auto.demo为单个配置文件，包含由autofs服务在此父目录下管理的子目录挂载点的列表。
 
Step-4：配置共享目录挂载点目录，运行以下命令：
vi /etc/auto.demo

上述文件的内容如下所示：
remote_dir -rw 192.168.4.211:/data/nfs_share

其中，remote_dir为子目录挂载点，此目录通常不可见，只有当autofs服务创建此目录和挂载共享之后对其进行直接命名/访问时，它才会变为可见。-rw为挂载网络共享时要使用的挂载选项。192.168.4.211:/data/nfs_share为需要挂载的NFS服务端和共享目录。
 
Step-5：重新启动autofs服务，运行以下命令：
service autofs restart
 
Step-6：访问网络共享目录，运行以下命令：
cd /root/demo/remote_dir

运行上述命令之后，autofs将自动创建挂载点目录，并且挂载共享目录。
 
Step-7：卸载网络共享目录，详情如下所示：
在/root/demo/remote_dir之下的所有文件和目录停止使用且超时期满之后（10分钟），autofs将卸载共享目录。


##  附录：实践环节 
十、NFS网络磁盘共享配置
NFS 是Network File System的缩写，即网络文件系统。一种使用于分散式文件系统的协定，由Sun公司开发，于1984年向外公布。功能是通过网络让不同的机器、不同的操作系统能够彼此分享个别的数据，让应用程序在客户端通过网络访问位于服务器磁盘中的数据，是在类Unix系统间实现磁盘文件共享的一种方法。
NFS 的基本原则是“容许不同的客户端及服务端通过一组RPC分享相同的文件系统”，它是独立于操作系统，容许不同硬件及操作系统的系统共同进行文件的分享。

环境说明
nfs服务端系统：CentOS 7.0 x86_64
nfs服务端IP：10.12.10.90
nfs客户端系统：CentOS 7.0 x86_64
nfs客户端IP：10.12.10.49

安装NFS服务端
Step-1：安装nfs-utils和rpcbind，运行以下命令
yum install -y nfs-utils rpcbind
上述命令将安装rpcbind服务和nfs服务。不过centos7默认已经安装好了，所以我们可以省略这一步。
Step-2：为NFS指定固定端口，(默认有些是动态端口导致被防火墙拦截)。
NFS 用到的服务有 portmapper nfs rquotad nlockmgr mountd
其中 portmapper nfs 服务端口是固定的分别是 111 2049
另外 rquotad nlockmgr mountd 服务端口是随机的。由于端口是随机的，这导致防火墙无法设置。
这时需要配置/etc/sysconfig/nfs 使 rquotad nlockmgr mountd 的端口固定。
运行以下命令：
vi /etc/sysconfig/nfs
RQUOTAD_PORT=875
LOCKD_TCPPORT=32803
LOCKD_UDPPORT=32769
MOUNTD_PORT=892
Step-3：开放防火墙中的上述端口，运行以下命令：
通过命令 rpcinfo -p 可查看nfs使用的端口
从centos7开始默认使用的是firewall作为防火墙，这里改为iptables防火墙。
1、关闭firewall：
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
2、安装iptables防火墙
yum install iptables-services #安装
vi /etc/sysconfig/iptables #编辑防火墙配置文件
systemctl restart iptables.service #最后重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动

创建脚本文件打开端口 vim enableport.sh
#!/bin/bash
iptables -I INPUT -s 10.12.10.0/24 -p tcp --dport 111 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp --dport 875 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp --dport 2049 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp --dport 32769 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp --dport 32803 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp --dport 892 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp --dport 111 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp --dport 875 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp --dport 2049 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp --dport 32769 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp --dport 32803 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp --dport 892 -j ACCEPT
iptables-save > /etc/sysconfig/iptables
保存之后sudo chmod +x enableport.sh
sudo ./enableport.sh即可开启nfs相关端口。
Step-5：创建共享目录，运行以下命令：
mkdir -p /opt/webroot
Step-6：配置exports文件，运行以下命令：
sudo vim /etc/exports
在上述文件的末尾新增一行，如下所示：
/opt/webroot 10.12.10.0/24(rw,sync,no_root_squash)
这一行表示只有10.12.10.0/24网段的客户端能够以读写权限挂载共享目录
保存之后执行输出共享目录：sudo exportfs –rv

Step-7：启动NFS相关服务，运行以下命令：
sudo systemctl enable nfs.service 
sudo systemctl enable rpcbind.service
sudo systemctl start rpcbind.service (注意顺序问题)
sudo systemctl start nfs.service

通过showmount –e可以查看本机的共享目录
通过show mount –e IP地址 查看远程的共享目录

安装NFS客户端

NFS客户端不需要启动NFS服务，但需要安装nfs-utils，运行以下命令：
yum install -y nfs-utils
不过centos7已经默认安装了。
手动挂载NFS共享目录
Step-1：确定挂载点，运行以下命令：
showmount -e 10.12.10.90
-e选项显示NFS服务端的导出列表。
Step-2：创建挂载目录，运行以下命令：
mkdir -p /opt/webroot
Step-3：挂载共享目录，运行以下命令：
mount -t nfs 10.12.10.90:/opt/webroot /opt/webroot
其中，-t选项用于指定文件系统的类型为nfs。
Step-4：共享目录使用结束之后，卸载共享目录，运行以下命令：
umount /opt/webroot

开机自动挂载
向fstab文件中添加共享目录的挂载条目，即可实现开机自动挂载，随后与NFS服务端的连接将始终处于活动状态。运行以下命令：
mkdir -p /opt/webroot
vim /etc/fstab
在上述文件末尾加入共享目录的挂载条目，如下所示：
10.12.10.90:/opt/webroot /opt/webroot nfs defaults 0 0
其中，第5个字段设置为0表示共享目录的文件系统不需要使用dump命令进行转储，第6个字段设置为0表示共享目录的文件系统不需要使用fsck命令进行检查。
4.查看是否挂载成功
df -h
5.卸载NFS挂载的共享目录
umount /opt/webroot


参考：http://www.linuxidc.com/Linux/2015-01/112051.htm
http://hanqunfeng.iteye.com/blog/2005726
http://www.cnblogs.com/mchina/archive/2013/01/03/2840040.html