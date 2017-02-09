title: can_t_install_64_office_because_32bit_installed 

#  windows:无法安装64位版本的Office，因为在您的PC上找到了以下32位程序 

参考：http://www.yishimei.cn/network/378.html
解决方案：按下win+R键，打开运行，输入regedit，打开注册表，依次定位到 HKEY_CLASSES_ROOT\Installer\Products，展开Products后，会出现若干以“00005”开头注册表键值，如下图所示：
![](/data/dokuwiki/windows/pasted/20150520-064058.png)
将这些“00005”开头的注册表键值都删除掉，然后再尝试安装64位的office2013就可以了。