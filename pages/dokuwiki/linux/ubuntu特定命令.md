title: ubuntu特定命令 

#  ubuntu特定命令 
##  软件管理 
修改软件源:` /etc/apt/sources.list `
` APT ` (高级软件包工具) 是一个强大的包管理系统，而那些图形化程序如 添加/删除 应用程序 和`  Synaptic ` 都是建立 在它的基础之上的。APT **自动处理依赖关系**并在系统软件包执行其他操作以便安装所要的软件包。 运行 APT 要求管理权限 。 
Ubuntu采用Debian的软件包管理器dpkg来管理软件包,类似RPM.系统中所有packages的信息都在` /var/lib/dpkg/ `目录下,其子目录` /var/lib/dpkg/info `用于保存各个软件包的配置文件列表:
(1).conffiles记录了Ubuntu**软件包的配置文件列表**
(2).list**记录安装文件清单**,**用户可以从.list的信息中找到软件包中文件的具体安装位置.**
(3).md5sums记录了软件包的md5信息,这个信息是用来进行包验证的.
(4).prerm脚本在Debian包解包之前运行,主要作用是停止作用于即将升级的Ubuntu软件包的服务,直到软件包安装或升级完成.
(5).postinst脚本是完成Debian包解开之后的配置工作,通常用于执行所安装软件包相关命令和服务重新启动.
` /var/lib/dpkg/available `文件的内容是Ubuntu软件包的描述信息,该软件包括当前系统所使用的Debian安装源中的所有软件包,**其中包括当前系统中已安装的和未安装的Ubuntu软件包.**
` /var/cache/apt/archives `目录是在用apt-getinstall安装软件时，软件包的**临时存放路径**
` /etc/apt/sources.list `存放的是软件源站点,当你执行sudoapt-get install xxx时，Ubuntu就去这些站点下载软件包到本地并执行安装

参考：http://www.360doc.com/content/11/1105/20/5169677_162080622.shtml
http://blog.csdn.net/daisy_chenting/article/details/6894647

查看已安装软件：
	dpkg -l (类似于rpm -qa)
	每条记录对应一个软件包, 注意每条记录的第一, 二, 三个字符. 这就是软件包的状态标识, 后边依此是软件包名称, 版本号, 和简单描述.
	第一字符为期望值,它包括:
		u 状态未知,这意味着软件包未安装,并且用户也未发出安装请求.
		i 用户请求安装软件包.
		r 用户请求卸载软件包.
		p 用户请求清除软件包.
		h 用户请求保持软件包版本锁定.
	第二列,是软件包的当前状态.此列包括软件包的六种状态.
		n 软件包未安装.
		i 软件包安装并完成配置.
		c 软件包以前安装过,现在删除了,但是它的配置文件还留在系统中.
		u 软件包被解包,但还未配置.
		f 试图配置软件包,但是失败了.
		h 软件包安装,但是但是没有成功.
	第三列标识错误状态,可以总结为四种状态. 第一种状态标识没有问题,为空. 其它三种符号则标识相应问题.h 软件包被强制保持,因为有其它软件包依赖需求,无法升级.r 软件包被破坏,可能需要重新安装才能正常使用(包括删除).x 软包件被破坏,并且被强制保持.
查询已安装软件名：(以下几种方式)
	dpkg --get-selections | grep ‘软件相关名称’  推荐
	dpkg -l
	dpkg --list
	dpkg --list|grep -i  'packagename' 
清除所有已删除软件包的残馀配置文件
	dpkg-l|grep^rc|awk'{print $2}'|sudo xargs dpkg -P
备份当前系统安装的所有软件包的列表
	dpkg --get-selections|grep -vde install>~/somefile
从上面备份的安装包的列表文件恢复所有包
	dpkg --set-selections<~/somefile ; sudo dselect
清理旧版本的软件缓存
	sudoapt-get autoclean
清理所有软件缓存
	sudo apt-get clean
删除系统不再使用的孤立软件
	sudoapt-get autoremove
sudo apt-get update 更新源 
sudo apt-get upgrade 更新已安装的包 
sudo apt-get dist-upgrade 升级系统 
sudo apt-get dselect-upgrade 使用 dselect 升级 

###  卸载软件 
1、删除软件 
方法一、如果你知道要删除软件的具体名称，可以使用 
	sudo apt-get remove --purge 软件名称 
	sudo apt-get autoremove --purge 软件名称 
方法二、**如果不知道要删除软件的具体名称，可以使用** 
	dpkg --get-selections | grep ‘软件相关名称’ 
	sudo apt-get purge 

2、清理残留数据 
	dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P 


###  apt命令 
* apt-get update——在修改/etc/apt/sources.list或者/etc/apt/preferences之後运行该命令。此外您需要定期运行这一命令以确保您的软件包列表是最新的。
* apt-get install packagename——安装一个新软件包（参见下文的aptitude ）
* apt-get remove packagename——卸载一个已安装的软件包（**保留配置文件**）
* apt-get -purge remove packagename——卸载一个已安装的软件包（**删除配置文件**）
* apt-get autoclean  apt会把已装或已卸的软件都备份在硬盘上，所以如果需要空间 的话，可以让这个命令来删除你已经删掉的软件
* apt-get clean 这个命令会把安装的软件的备份也删除，不过这样不会影响软件的使用的。
* apt-get upgrade——更新所有已安装的软件包
* apt-get dist-upgrade——将系统升级到新版本

* dpkg –force-all –purge packagename 有些软件很难卸载，而且还阻止了别的软件的应用 ，就可以用这个，不过有点冒险。
* dpkg -l package-name-pattern——列出所有与模式相匹配的软件包。如果您不知道软件包的全名，您可以使用“*package-name-pattern*”。
* aptitude——详细查看已安装或可用的软件包。与apt-get类似，aptitude可以通过命令行方式调用，但仅限于某些命令——最常见的有安装和卸载命令。由于aptitude比apt-get了解更多信息，可以说它更适合用来进行安装和卸载。

* apt-cache search string——在软件包列表中搜索字符串
* apt-cache showpkg pkgs——显示软件包信息。
* apt-cache dumpavail——打印可用软件包列表。
* apt-cache show pkgs——显示软件包记录，类似于dpkg –print-avail。
* apt-cache pkgnames——打印软件包列表中所有软件包的名称。
* dpkg -S file——这个文件属于哪个已安装软件包。
* dpkg -L package——列出软件包中的所有文件。

* apt-file search filename——**查找包含特定文件的软件包（不一定是已安装的）**，这些文件的文件名中含有指定的字符串。**apt-file是一个独立的软件包。您必须先使用apt-get install来安装它，然後运行apt-file update。**如果apt-file search filename输出的内容太多，您可以尝试使用apt-file search filename | grep -w filename（只显示指定字符串作为完整的单词出现在其中的那些文件名）或者类似方法，例如：apt-file search filename | grep /bin/（只显示位于诸如/bin或/usr/bin这些文件夹中的文件，如果您要查找的是某个特定的执行文件的话，这样做是有帮助的）。

常用的APT命令参数 
apt-cache search package 搜索包 
apt-cache show package 获取包的相关信息，如说明、大小、版本等 
sudo apt-get install package 安装包 
sudo apt-get install package - - reinstall 重新安装包 
sudo apt-get -f install 修复安装"-f = ――fix-missing" 
sudo apt-get remove package 删除包 
sudo apt-get remove package - - purge 删除包，包括删除配置文件等 
sudo apt-get update 更新源 
sudo apt-get upgrade 更新已安装的包 
sudo apt-get dist-upgrade 升级系统 
sudo apt-get dselect-upgrade 使用 dselect 升级 
apt-cache depends package 了解使用依赖 
apt-cache rdepends package 是查看该包被哪些包依赖 
sudo apt-get build-dep package 安装相关的编译环境 
apt-get source package 下载该包的源代码 
sudo apt-get clean && sudo apt-get autoclean 清理无用的包 
sudo apt-get check 检查是否有损坏的依赖 
###  dpkg命令 
**安装/卸载 .deb 文件** 
安装
	sudo dpkg -i package_file.deb  
卸载
	sudo dpkg -r package_name   
	dpkg -r 删除，不清除配置
	dpkg -P 删除并清配置 

**将 .rpm 文件转为 .deb 文件** 
安装 alien 程序。 
在终端使用管理权限运行以下命令： 
	sudo alien package_file.rpm   

**dpkg命令**
dpkg -i file.deb 安装软件 
dpkg -x file.deb 解开.deb文件 
dpkg -r 删除，不清除配置
dpkg -p 删除并清配置 
更详细的 用dpkg --help 查询 如下： 
dpkg -i|--install <.deb 文件的文件名> ... | -R|--recursive <目录> ... 
dpkg --unpack <.deb 文件的文件名> ... | -R|--recursive <目录> ... 
dpkg -A|--record-avail <.deb 文件的文件名> ... | -R|--recursive <目录> ... 
dpkg --configure <软件包名> ... | -a|--pending 
dpkg -r|--remove | -P|--purge <软件包名> ... | -a|--pending 
dpkg --get-selections [<表达式> ...] 把已选中的软件包的列表打印到标准输出 
dpkg --set-selections 从标准输入里读出要选择的软件包列表 
dpkg -s|--status <软件包名> ... 显示软件包详尽的状态信息 
dpkg -L|--listfiles <软件包名> ... 列出所有“属于”该软件包(或多个软件包)的文件 
dpkg -l|--list [<表达式> ... 简明地列出软件包的状态 
dpkg -S|--search <表达式> ... 搜寻拥有该文件(或多个文件)的软件包 
dpkg -C|--audit 检查搜寻残损的软件包 

dpkg --force-help | -Dh|--debug=help 强制操作时，有关出错方面的帮助 
