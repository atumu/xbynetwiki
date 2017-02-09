title: android关机重启重启至recovery应用开发思路 

#  android关机重启重启至recovery应用开发思路 

概述

Android关机，重启，重启至recovery以及热重启(快速重启)等一般都是通过电源键操作的。
对于软件实现颇感兴趣。其实对于实现root的设备来说确实很容易，因为就是执行几个linux shell命令。
所以实现前提：设备已获取root权限。
本文只打算说一说思路，并不提供具体实现。

思路

Android中没有shutdown命令，但是有个reboot命令，我们就从这个命令入手解决我们的问题。
```

关机：
 Runtime.getRuntime().exec(
                            new String[] { "/system/bin/su", "-c", "reboot -p" });
重启:
 Runtime.getRuntime().exec(
                            new String[] { "/system/bin/su", "-c", "reboot now" });
重启至recovery：
 Runtime.getRuntime().exec(
                            new String[] { "/system/bin/su", "-c", "reboot recovery" });
热重启(快速重启):
Runtime.getRuntime().exec(
                            new String[] { "/system/bin/su", "-c", "busybox killall system_server" });

```