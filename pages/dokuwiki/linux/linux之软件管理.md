title: linux之软件管理 

#  linux之软件管理 
0.4已安装软件状态信息：
"/var/lib/apt/extended_states"
0.5本地存放的apt安装的软件包副本位置：
/var/cache/apt/archives
没有下载完的软件包在：/var/cache/apt/archives/partial
注意：这里无法通过apt-get autoclean清除，但可以通过apt-get clean清除。
几个清除命令的区别：

sudo apt-get autoclean                清理旧版本的软件缓存
sudo apt-get clean                    清理所有软件缓存
sudo apt-get autoremove             删除系统不再使用的孤立软件
这三个命令主要清理升级缓存以及无用包的。
0.6软件删除与清理常用方式：


sudo apt-get remove --purge 软件名
sudo apt-get autoremove                                                        删除系统不再使用的孤立软件
sudo apt-get autoclean                                                            清理旧版本的软件缓存
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P              清除残余的配置文件，保证干净。

0.7 update-alternatives命令：用于管理相同功能的软件，默认软件。

 sudo update-alternatives --display vi
...
$ sudo update-alternatives --config vi
The Debian alternatives system keeps its selection as symlinks in "/etc/alternatives/". The selection process uses corresponding file in "/var/lib/dpkg/alternatives/".

0.8转换rpm或其他格式包至deb包：alien
0.9:
dpkg 来管理软件包, 类似 RPM. 系统中所有 packages 的信息都在 /var/lib/dpkg/
目录下, 其子目录 /var/lib/dpkg/info 用于保存各个软件包的配置文件列表:
(1)*.conffiles 记录了软件包的配置文件列表
(2)*.list 保存软件包中的文件列表, 用户可以从 .list 的信息中找到软件包中文件的具体安装位置.
(3)*.md5sums 记录了软件包的md5信息, 这个信息是用来进行包验证的.
(4)*.prerm 脚本在 Debian 包解包之前运行, 主要作用是停止作用于即将升级的软件包的服务, 直到软件包安装或升级完成.
(5)*.postinst 脚本是完成 Debian 包解开之后的配置工作, 通常用于执行所安装软件包相关命令和服务重新启动.
0.10:ppa源管理：
添加一个PPA源
sudo add-apt-repository ppa:user/ppa-name
如添加cairo-dock到weekly update源
sudo add-apt-repository ppa:cairo-dock-team/weekly/ubuntu
删除一个PPA源
sudo add-apt-repository -r ppa:user/ppa-name
sudo add-apt-repository -r ppa:cairo-dock-team/weekly/ubuntu
添加apt-key
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 72D340A3

1.dpkg:
一般debian系列发行版本都会使用dpkg作为底层的package management.与其他包管理器不同的是，
注意哦，dpkg并不处理软件包之间的依赖关系，也不会自动下载软件包和其依赖，所以dpkg只能安装local package
显示已安装package:
dpkg -l
显示某个文件属于的软件.
#dpkg -S /etc/apache2/apache2.conf
apache2.2-common: /etc/apache2/apache2.conf
属于apache2.2-common这个软件安装是产生
有用的dpkg命令:
dpkg --set-selections; dpkg --get-selections
dpkg -P <old>移除旧的或残存的包或配置文件
dpkg --configure -a修复软件损坏

使用dpkg安装与卸载软件（不推荐，因为dpkg不处理软件包之间的依赖关系）：dpkg -i xxx.deb安装;dpkg -r xxx (不推荐)

2.Apt-Get(Advance Package Tool)
配置目录 /etc/apt/apt.conf.d/
/etc/cron.daily/apt

ubuntu官方推荐的管理工具。
它由一个或多个远程软件仓库取出要安装的软件包。同时它会在本地建立远程仓库的索引以便随时查找可安装的软件包
配置远程仓库/etc/apt/sources.list与/etc/apt/sources.list.d/目录。
更新索引：apt-get update
更新已安装的软件：apt-get upgrade
apt-get install zip p7zip-full
apt-get remove zip
apt-get remove p7zip-full --purge
apt-get help
日志文件/var/log/dpkg.log
添加key文件apt-key add

常用：

1. 更新或升级操作：

   apt-get update                  # 更新源  

   apt-get upgrade                 # 更新所有已安装的包  

   apt-get dist-upgrade                # 发行版升级（如，从10.10到11.04）  

2. 安装或重装类操作：

   apt-get install <pkg>         # 安装软件包<pkg>，多个软件包用空格隔开  

   apt-get install --reinstall <pkg> # 重新安装软件包<pkg>  

   apt-get install -f <pkg>          # 修复安装（破损的依赖关系）软件包<pkg>  

3. 卸载类操作：

   apt-get remove <pkg>          # 删除软件包<pkg>（不包括配置文件）  

   apt-get purge <pkg>           # 删除软件包<pkg>（包括配置文件）  

4. 下载清除类操作：

   apt-get source <pkg>              # 下载pkg包的源代码到当前目录  

   apt-get download <pkg>            # 下载pkg包的二进制包到当前目录  

   apt-get source -d <pkg>           # 下载完源码包后，编译  

   apt-get build-dep   <pkg>     # 构建pkg源码包的依赖环境（编译环境？）  

   apt-get clean                   # 清除缓存(/var/cache/apt/archives/{,partial}下)中所有已下载的包  

   apt-get autoclean               # 类似于clean，但清除的是缓存中过期的包（即已不能下载或者是无用的包）  

   apt-get autoremove              # 删除因安装软件自动安装的依赖，而现在不需要的依赖包  

5. 查询类操作：

   apt-cache stats             # 显示系统软件包的统计信息  

   apt-cache search <pkg>            # 使用关键字pkg搜索软件包  

   apt-cache show   <pkg_name>   # 显示软件包pkg_name的详细信息  

   apt-cache depends <pkg>       # 查看pkg所依赖的软件包  

   apt-cache rdepends <pkg>      # 查看pkg被那些软件包所依赖  

3.Aptitude
aptitude有一个纯文字界面的管理
不过一般还是使用命令行方便：
aptitude install/remove zip

4.Automatic Update自动更新
安装unattended-upgrades可以进行自动更新，或者有选择性的安全更新。同时可以添加软件黑名单不允许其更新
首先安装它：apt-get install unattended-upgrades
配置文件/etc/apt/apt.conf.d/50-unattended-upgrades,进行调整如下：
```

Unattended-upgrades::Allowed-Origins{
"Ubuntu precise-security";
// "Ubuntu precise-updates";
}
添加软件黑名单不允许更新：
Unattended-upgrades::Package-Blacklist{
"vim";
// “libc6”;
}

```
配置运行自动更新vim /etc/apt/apt.conf.d/02periodic 添加：
```

APT::Periodic::Update-Package-Lists "1";  //每天更新索引
APT::Periodic::Download-Upgradeable-Packages "1";  //每天下载更新
APT::Periodic::AutocleanInterval "7" //每个星期清理一次
APT::Periodic::Unattended-Upgrade "1";
关于此文件的更多设置信息可在/etc/cron.daily/apt中头部注释可见。
日志文件/var/log/unattended-upgrades
通知管理员
在/etc/apt/apt.conf.d/50unattended-upgrades文件中配置Unattended-Upgrade::Mail可以通知管理员要安装的更新和出现的问题
也可以使用apticron来完成通知apt-get install apticron,然后配置/etc/apticron/apticron.conf来进行通知

```