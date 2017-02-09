title: win7下virtualbox问题总结 

#  win7下virtualbox问题总结 
##  virtualbox与win7共享文件夹 
![](/data/dokuwiki/windows/pasted/20151013-081639.png?500x200)
![](/data/dokuwiki/windows/pasted/20151013-081628.png?200x200)
在Ubuntu下挂载Windows下的共享文件夹
挂载步骤如下：
1. mkdir share_windows (新建挂载点)
2. mount -t vboxsf shareFolder(第一截图的第一列的那个名称) /mnt/share (挂载共享文件夹）
3. cd share_windows 加入共享文件夹

参考:http://www.linuxidc.com/Linux/2014-02/96713.htm 
http://www.v2ex.com/t/206102#reply1
##  win7下virtualbox无法选择安装64位系统 
[Microsoft® Hardware-Assisted Virtualization Detection Tool](http://www.microsoft.com/en-us/download/details.aspx?id=592)
即下载运行[havdetectiontool.exe](http://pan.baidu.com/s/1gdCXcrx)即可

##  VirtualBox提示Unable to load R3 module 
今天在使用VirtualBox 安装虚拟机的时候，有提示Unable to load R3 module C:\Program Files\Oracle\VirtualBox/VBoxDD.DLL (VBoxDD): GetLastError=1790 (VERR_UNRESOLVED_ERROR).的错误，后来自己摸索了下，找到了下面的办法可以进行解决。

解决方法：windows系统的[主题文件被破解]的原因，恢复系统主题文件即可。
找到 C:/windows/system32 目录 下面的uxtheme.dll文件<记住此文件,就是这个文件导致的>
查看下你的操作系统类型，是32位的还是64位的。.然后去其它的电脑上面将上面的C:\Windows\system32\uxtheme.dll 文件拷贝过来，直接覆盖上面的文件或者去网上下载一个这个文件。进行替换。
[主题破解及还原工具]->http://pan.baidu.com/s/1eQtgLKA
参考：http://jingyan.baidu.com/article/ab69b270bb7b2a2ca6189f6d.html http://www.aixq.com/post-328.html
##  不能为虚拟电脑 xxx 打开一个新任务. 
![](/data/dokuwiki/windows/pasted/20151013-081450.png)
复制错误信息为：
不能为虚拟电脑 boot2docker-vm 打开一个新任务.
The virtual machine 'boot2docker-vm' has terminated unexpectedly during startup with exit code 1 (0x1). More details may be available in 'C:\Users\Ericye\VirtualBox VMs\boot2docker-vm\Logs\VBoxStartup.log'.
返回 代码: E_FAIL (0x80004005) 
组件: Machine 
界面: IMachine {480cf695-2d8d-4256-9c7c-cce4184fa048} 

解决办法：
那是因为vboxdrv服务没有安装或没有成功启动，
64位的系统经常这样，
找到安装目录下的vboxdrv文件夹，
如D:\Program Files\Oracle\VirtualBox\drivers\vboxdrv，
右击VBoxDrv.inf，选安装，然后重启。

参考：http://tieba.baidu.com/p/3495196597

