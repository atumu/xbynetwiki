title: docker如何限制容器内存大小 

#  docker如何限制容器内存大小？ 
**container中的free命令，看的是Host上的内存。**

要查看container的内存限制用这个：
cat /cgroup/memory/lxc/{full_container_id}/memory.limit_in_bytes
/cgroup/下还可以查看其它资源的限制：

ll /cgroup/
总用量 0
drwxr-xr-x 3 root root 0 3月  17 13:33 blkio
drwxr-xr-x 3 root root 0 3月  17 13:33 cpu
drwxr-xr-x 3 root root 0 3月  17 13:33 cpuacct
drwxr-xr-x 3 root root 0 3月  17 13:33 cpuset
drwxr-xr-x 3 root root 0 3月  17 13:33 devices
drwxr-xr-x 3 root root 0 3月  17 13:33 freezer
drwxr-xr-x 3 root root 0 3月  17 13:33 memory
drwxr-xr-x 3 root root 0 3月  17 13:33 net_cls

楼主，我用ubuntu虚拟机做实验的时候也遇到这个问题，请参照这里解决
http://docs.docker.com/articles/runmetrics/
以下是重点摘录

If you want to enable memory and swap accounting, you must add the following command-line parameters to your kernel:

` $ cgroup_enable=memory swapaccount=1 `
On systems using GRUB (which is the default for Ubuntu), you can add those parameters by editing /etc/default/grub and extending GRUB_CMDLINE_LINUX. Look for the following line:

` $ GRUB_CMDLINE_LINUX="" `
And replace it by the following one:

` $ GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1" `
Then run sudo update-grub, and reboot.

These parameters will help you get rid of the following warnings:

WARNING: Your kernel does not support cgroup swap limit.
WARNING: Your kernel does not support swap limit capabilities. Limitation discarded.


##  centos下解决 
我是修改的/etc/grub.conf 在当前使用的内核的kernel项的最后加上cgroup_enable=memory swapaccount=1
reboot
然后 cat /proc/cmdline 就会发现，配置生效了