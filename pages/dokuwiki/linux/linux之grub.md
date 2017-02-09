title: linux之grub 

#  linux之Grub与亮度调节 
##  一、grub修复： 

假设boot单独分区挂载在/dev/sda3
sudo mount /dev/sda3 /mnt
sudo grub-install --boot-direcotry=/mnt /dev/sda
sudo update-grub
sudo reboot
这样一般都能修复.假如还不能修复，那就是mbr区的主程序已经损坏，需要这样
在grub shell下：
grub>ls (hd0,2)
确认是boot分区
grub>set root=(hd0,2)或者root (hd0,2)
grub>setup (hd0)
grub>quit
重启。

##  二、grub-shell启动iso镜像 

grub>root /path/to/hava/ISOimage
grub>loopback loop /iso_imagename替换为你的iso文件名
grub>linux (loop)/casper/(loop)/casper/vmlinuz.efi boot=casper iso-scan/filename=/iso_imagename noprompt noeject
grub>initrd (loop)/casper/initrd.lz
grub>boot

##  三、linux亮度无法调节解决方案： 

所以我们只要sudo nano /etc/default/grub,
GRUB_CMDLINE_LINUX="“改为GRUB_CMDLINE_LINUX="acpi_osi=` L `inux acpi_backlight=vendor"重启即可，注意大小写Linux的L为大写。
然后sudo update-grub && sudo reboot