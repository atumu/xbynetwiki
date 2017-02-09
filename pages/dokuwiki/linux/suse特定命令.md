title: suse特定命令 

#  suse特定命令 
suse下的包管理器：yast2图形界面操作，或字符终端用zypper
zypper 是 OpenSUSE 命令行下管理软件的程序(类似于Debian／Ubuntu的apt，Fedora/CentOS中的yum),功能十分强大。
/etc/zypp/repo.d/
/var/cache/zypp/packages


**添加软件源**
```

zyppr ar URL alias

```
URL 就是软件源的地址
alias 就是你取另外一个名字
例子：添加11.3的官方软件和升级源
```

zypper ar http://download.opensuse.org/distribution/11.3/repo/oss/ main
zypper ar http://download.opensuse.org/distribution/11.3/repo/non-oss/ nonoss
zypper ar http://download.opensuse.org/update/11.3/ update

```
**刷新软件源**，请耐心等待，尤其是第一次的时候。
```

zypper refresh

```
现在就可以升级软件了
```

zypper update

```
**安装软件**也很简单
```

zypper install 软件名

```
下面是完整的介绍：
```

zypper [--全局选项] <命令> [--命令选项] [参数]

```
全局选项：
```

--help, -h 帮助。.
--version, -V 输出版本号。
--quiet, -q 减少普通输出，仅打印错误信息。
--verbose, -v 增加信息的详细程度
--no-abbrev, -A 表格中不出现缩写文本。
--table-style, -s 表格样式 (整数)。
--rug-compatible, -r 开启与 rug 的兼容。
--non-interactive, -n 不询问任何问题，自动使用默认的回复。
--xmlout, -x 切换到 XML 输出。
--reposd-dir, -D <dir> 使用其他的安装源定义文件目录。
--cache-dir, -C <dir> 使用其他的元数据缓存数据库目录。
--raw-cache-dir <dir> 使用其他的原始元数据缓存目录。

```
源选项:
```

--no-gpg-checks 忽略 GPG 检查失败并继续。
--plus-repo, -p <URI> 使用额外的安装源。
--disable-repositories 不从安装源读取元数据。
--no-refresh 不刷新安装源。

```
目标选项：
```

--root, -R <dir> 在不同的根目录下操作。
--disable-system-sources、-D 不读取系统安装的可解析项。

```
命令：
help, ? 打印帮助。
shell, sh 一次接受多个命令.
安装源操作：
repos, lr 列出所有定义的安装源。
**addrepo, ar 添加一个新的安装源。**
removerepo, rr 删除指定的安装源。
renamerepo, nr 重命名指定的安装源。
modifyrepo, mr 修改指定的安装源。
**refresh, ref 刷新所有安装源。**
**clean 清除本地缓存。**
软件管理：
  * install, in 安装软件包。
  * remove, rm 删除软件包。
  * verify, ve 检验软件包的依赖关系的完整性。
  * update, up 将已经安装的软件包更新到新的版本。
  * dist-upgrade, dup 执行整个系统的升级。
  * source-install, si 安装源代码软件包和它们的编译依赖。
查询：
**search, se 查找符合一个模式的软件包。**
info, if 显示指定软件包的完整信息。
patch-info 显示指定补丁的完整信息。
pattern-info 显示指定模式的完整信息。
product-info 显示指定产品的完整信息。
patch-check, pchk 检查补丁。
list-updates, lu 列出可用的更新。
patches, pch 列出所有可用的补丁。
**packages, pa 列出所有可用的软件包。**
patterns, pt 列出所有可用的模式。
products, pd 列出所有可用的产品。
what-provides, wp 列出能够提供指定功能的软件包。
软件包锁定：
addlock, al 添加一个软件包锁定。
removelock, rl 取消一个软件包锁定。
locks, ll 列出当前的软件包锁定。
其他：
versioncmp, vcmp 比较两个版本
targetos, tos 显示目标操作系统标识字符串
licenses  显示有关许可证、eulas的安装程序包

用 Linux系统总是免不了要接触包Linux管理工具。

比 如，Debian/Ubuntu 的 apt、openSUSE 的 zypper、Fedora 的 yum、Mandriva 的 urpmi、Slackware 的 slackpkg、Archlinux 的 pacman、Gentoo 的 emerge、Foresight 的 conary、Pardus 的 pisi，等等。
安装包 
apt-get install <pkg>
zypper install <pkg>
yum install <pkg> 

移除包 
apt-get remove <pkg>
zypper remove <pkg> 
yum erase <pkg>

更新包列表 
apt-get update
zypper refresh
yum check-update


更新系统
apt-get upgrade 
zypper update 
yum update 

列出源 
cat /etc/apt/sources.list 
zypper repos 
yum repolist


添加源 
(edit /etc/apt/sources.list) 
zypper addrepo <path> <name> 
(add <repo> to /etc/yum.repos.d/)

移除源 
(edit /etc/apt/sources.list) 
zypper removerepo <name> 
(remove <repo> from /etc/yum.repos.d/)

搜索包 
apt-cache search <pkg> 
zypper search <pkg>
yum search <pkg> 

列出已安装的包 
dpkg -l
rpm -qa
rpm -qa 

##  仅下载而不安装 
参考：http://talk2computer.blogspot.com/2011/03/zypperyast-download-all-rpm-packages.html
Zypper/Yast: Download all RPM packages first and then install
Yast is the package management tool in open SuSE and its back-end is zypper. 
One of frequent issue that I have faced with zypper is that it installs a RPM package as soon as it downloads. It does not download the dependent RPMs before installing a RPM package. Sometime this leads to broken installs if there is some network problem while downloading the dependent RPMs. The problem becomes even worse if there is some network problem while the system update is going on. Once I had a broken update for KDE and I was not able to log into the GUI.
The behavior of zypper can be changed by config file. But I do not understand why the Open SuSE comes with behavior by default. The other popular Linux destro Ubuntu does not have this problem.
**Anyway, I here is the solution to the probelm:-**
Search for the line
commit.downloadMode
in  
**/etc/zypp/zypp.conf**
By default this line is commented.** Uncomment this line** and change it to   
**commit.downloadMode = DownloadInAdvance**


其余参考：
1、https://forums.opensuse.org/showthread.php/488443-zypper-how-to-download-all-the-dependencies-of-wine
You can run the command
```

zypper rm -u wine

```
without confirming to continue. Then you have **a list which packages and its dependencies** would be uninstalled. You can **copy the package names into a textfile named packages.txt** and use the following command to download them to **/var/cache/zypp/packages/**:
```

zypper in -d --force $(cat packages.txt)

```
2、You can run the following to see the dependencies:
```

rpm -q --requires wine

```
Or select the package in YaST->Software Management and click on the "Dependencies" tab in the bottom-right...

3、仅下载而不安装：未来可能支持 zypper install --download-only ,
-d, --download-only. The packages will then be stored in /var/cache/zypp/packages/ until they are installed.
Code: zypper in -d wine
参考：https://cn.opensuse.org/Zypper/Usage
http://justwinit.cn/post/4232/