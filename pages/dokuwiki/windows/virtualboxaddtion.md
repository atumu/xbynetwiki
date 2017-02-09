title: virtualboxaddtion 

#  Installing VirtualBox Guest Additions on Ubuntu Server 
http://en.ig.ma/notebook/2012/virtualbox-guest-additions-on-ubuntu-server
http://askubuntu.com/questions/155947/virtualbox-guest-additions-wont-install-on-ubuntu-server-12-04
http://unix.stackexchange.com/questions/52667/file-permission-issues-with-shared-folders-under-virtual-box-ubuntu-guest-wind

1、**启动ubuntu-server,点击设备-安装增强功能，获取Guest Additions CD image 镜像。**
2、Mount the CD Rom with the shell command:
```

$ sudo mount /dev/cdrom /media/cdrom

```

3、After that the install scripts should be accessible in the /media/cdrom/ directory:
```

$ ls -l /media/cdrom/

total 41942
dr-xr-xr-x 3 root root     2048 Dec 19 13:11 32Bit
dr-xr-xr-x 2 root root     2048 Dec 19 13:11 64Bit
-r-xr-xr-x 1 root root      647 Aug 16  2011 AUTORUN.INF
-r-xr-xr-x 1 root root     6966 Dec 19 13:02 autorun.sh
-r-xr-xr-x 1 root root     5523 Dec 19 13:02 runasroot.sh
-r-xr-xr-x 1 root root  7361995 Dec 19 13:06 VBoxLinuxAdditions.run
-r-xr-xr-x 1 root root 14634496 Dec 19 13:08 VBoxSolarisAdditions.pkg
-r-xr-xr-x 1 root root 13270208 Dec 19 12:55 VBoxWindowsAdditions-amd64.exe
-r-xr-xr-x 2 root root   278832 Dec 19 12:48 VBoxWindowsAdditions.exe
-r-xr-xr-x 1 root root  7383936 Dec 19 12:49 VBoxWindowsAdditions-x86.exe

```
4、Install necessary build tools and build dependencies:
```

$ sudo apt-get install -y dkms build-essential linux-headers-generic linux-headers-$(uname -r)

```

5、Build and install the Guest Additions:
```

$ sudo /media/cdrom/VBoxLinuxAdditions.run --nox11

```
6、File permission issues with shared folders under Virtual Box (Ubuntu Guest, Windows Host)
```

usermod -aG vboxsf <youruser>
ln -s /media/sf_Ubuntu /home/m/Desktop/vbox_shared

```
then logout, login,ok.


##  centos版安装： 
```

 yum update
 yum install gcc kernel-devel bzip2
yum install kernel-devel-3.10.0-327.el7.x86_64
 mkdir -p /media/cdrom
 mount /dev/sr0 /media/cdrom
 sh /media/cdrom/VBoxLinuxAdditions.run
reboot

```
