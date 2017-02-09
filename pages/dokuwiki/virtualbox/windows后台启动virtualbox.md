title: windows后台启动virtualbox 

#  windows后台(命令行)启动VirtualBox 
http://www.jb51.net/os/windows/124883.html
在Windows下用SSH连接Linux。但Virtualbox启动总会有界面出现，感觉老别扭，下面为大家提供个不错的解决方法，其实Virtualbox是提供了后台启动的。
查看有哪些虚拟机
VBoxManage list vms
查看虚拟的详细信息
VBoxManage list vms --long
查看运行着的虚拟机
VBoxManage list runningvms
开启虚拟机在后台运行
VBoxManage startvm <vm_name> -type headless
开启虚拟机并开启远程桌面连接的支持
VBoxManage startvm <vm_name> -type vrdp
改变虚拟机的远程连接端口,用于多个vbox虚拟机同时运行
VBoxManage controlvm <vm_name> vrdpprot <ports>
关闭虚拟机
VBoxManage controlvm <vm_name> acpipowerbutton
强制关闭虚拟机
VBoxManage controlvm <vm_name> poweroff

启动脚本:
```

@echo off
:: by xby
cd /d D:\Program Files\Oracle\VirtualBox
VBoxManage startvm "ubuntu-server" -type headless

```
关闭脚本:
```

@echo off
:: by xby
cd /d "D:\Program Files\Oracle\VirtualBox"
VBoxManage controlvm "ubuntu-server" acpipowerbutton

```