title: linux常用命令2 

#  Linux常用命令2 
/usr/bin  /bin    存放所有用户可以执行的命令。
/usr/sbin /sbin  存放只有root可以执行的命令（s代表super）。
/home             用户缺省宿主目录。
/proc              虚拟文件系统，存放当前内存镜像(cpu,内存,进程信息)。
/dev               存放设备文件(硬盘，网卡等)。
/lib                 存放系统程序运行所需的共享库。
/lost+found      存放一些系统出错的检查结果(每个挂载点的根目录都有)。
/tmp               存放临时文件。
                     拥有特殊权限黏着位，每个用户都有写权限，只能删除自己的文件。
/etc               系统配置文件  (10-20MB左右)。
/var                包含经常变动的文件,如邮件，日志文件，计划任务等。
                      /var/log 日志  /var/spool/caron,/var/spool/at计划任务。
/usr                 存放所有命令、库、手册页等（类windows系统的Windows目录）。
                      linux程序一般安装在/usr/local目录下（类Program Files目录）。
/mnt                临时文件系统的安装点，如网络共享目录、u盘、光盘等。
/boot               内核文件及自举程序文件保存位置（单独划分128MB左右）。
<wrap em>**常见伪设备
/dev/zero
/dev/null  **</wrap> 
##  目录或文件类型 
ls –l 查看到的第一个字符
d=directory 代表目录
- 二进制文件
l=link 软链接文件
b=block 块设备，如硬盘，光盘等。
c=char 字符设备，如终端，打印机等。

mkdir 创建目录
touch 创建文件
rm 删除文件或目录
mv 移动文件
ln 创建软链接
ls 显示列表
cd 切换目录
cp 复制文件或目录
-R备份目录
-p保持备份目录及文件属性
-u增量备份
##  压缩软件 
tar
打包多个目录
tar –zcf /back/sys.tar.tz /etc /boot
打包多个文件
tar –zcf /back/config.tar.tz /etc/passwd /etc/shadow
查看包内文件
tar –ztf /back/config.tar.tz
解包(默认解压到原目录，-C 可以指定还原目录)
tar –zxf /back/config.tar.tz
只恢复备份中的指定文件
tar –zxf /back/config.tar.tz etc/passwd
追加文件
tar –rf /back/config.tar etc/passwd
更新文件
tar –uf /back/config.tar etc/passwd
注：和硬件相关命令  添加硬件后可查看相关信息
dmesg 开机引导流程相关命令
注：系统备份需备份文件
/etc 系统配置文件
##  文件系统常用命令 
**df   查看分区的情况**
df     以数据块的方式查看
df -m 以兆的方式查看
df -h 更人性化的方式查看

**du 查看文件或目录大小**
查看文件大小
du –h /etc/services
查看目录大小 (s=sum统计)
du –sh /etc

**fsck,e2fsck 检测修复文件系统，在单用户模式执行。**
fsck –y 分区名   (y自动应答询问)
e2fsck –p 分区名

**file  判断文件类型**
file /etc/services

**mount 挂载如光盘，u盘等**
挂载光驱
mount /dev/cdrom /mnt/cdrom
卸载光驱
umount /mnt/cdrom
eject  卸载光驱并弹出

##  fdisk 分区管理软件 
查看硬盘分区
fdisk –l  /dev/sdb
硬盘分区
fdisk /dev/sdb
p 打印分区表
n 添加新的分区
d 删除分区
t 改变系统类型
w 写入分区表并退出
q 不保存退出
m 帮助
注：分区类型
主分区(primary) 最多划分4个
扩展分区(extended)
逻辑分区-建立在扩展分区上

**mkfs 格式化**
mkfs.ext3 /dev/sdb1
注：-b 调整数据块大小

**e2lable 添加卷标**
e2lable /dev/sdb1 web

**增加开机识别**
修改/etc/fstable

dd （硬盘对考，创建指定大小文件）
硬盘拷贝
dd if=/dev/sda of=/dev/sdb
注:sdb大于或等于sda硬盘
if infile  of outfile

##  增加虚拟内存swap 
--创建存放虚拟内存文件的文件夹
mkdir /var/swap
--更改权限只有管理员能访问
chmod 700 /var/swap
--创建指定大小空文件
dd if=/dev/zero of=/var/swap/file.swp bs=1024 count=65536   注：字节
--创建虚拟内存
mkswap /var/swap/file.swp
--开启虚拟内存   
swapon /var/swap/file.swp
--关闭虚拟内存
swapoff /var/swap/file.swp
--开机加载
vi /etc/fstab
swp文件名称  swap swap defaults 0 0

##  磁盘配额 (usrquota用户配额，grpquota组配额) 
临时设置：
mount –o remount,usrquota /home
开机生效：
修改/etc/fstab
defaults,usrquota
mount –o remount /home 重新加载
建立配额数据库
quotacheck –cvuga
--会在配额分区下的创建aquota.user
启动配额
quotaon /home
关闭配额
quotaoff /home
配置用户配额
edquota 用户名
edquota –g 用户组
配置信息
限制空间大小（单位kb）
blocks(kb) soft hard
soft 软限制(超过宽限期，多余文件会被删除)
hard 硬限制
限制创建文件的多少
inodes      soft hard
更改宽限期限
edquota –t
查看用户配额情况
quota
管理员查看所有用户配额
repquota –a
复制用户配额信息
edquota  -p 要复制的用户名  被复制的用户名 被复制的用户名  被复制的用户名
参考：http://blog.csdn.net/liuweitoo/article/details/8158423