title: ubuntu修改时区 

#  ubuntu修改时区 
参考：http://www.cnblogs.com/xiaoyaoxia/archive/2012/10/30/2746840.html
http://blog.chinaunix.net/uid-22235894-id-1782007.html
首先查看时区：
swfsadmin@swfsubuntu:~$ date -R
Tue, 17 Dec 2013 18:23:01 +0800
如果要修改时区，
执行` sudo tzselect `
2.选择区域：亚洲
```

swfsadmin@swfsubuntu:~$ sudo tzselect
[sudo] password for swfsadmin: 
Sorry, try again.
[sudo] password for swfsadmin: 
Please identify a location so that time zone rules can be set correctly.
Please select a continent or ocean.
 1) Africa
 2) Americas
 3) Antarctica
 4) Arctic Ocean
 5) Asia
 6) Atlantic Ocean
 7) Australia
 8) Europe
 9) Indian Ocean
10) Pacific Ocean
11) none - I want to specify the time zone using the Posix TZ format.
#? 5

```
3.选择国家：中国
```

Please select a country.
 1) Afghanistan           18) Israel                35) Palestine
 2) Armenia               19) Japan                 36) Philippines
 3) Azerbaijan            20) Jordan                37) Qatar
 4) Bahrain               21) Kazakhstan            38) Russia
 5) Bangladesh            22) Korea (North)         39) Saudi Arabia
 6) Bhutan                23) Korea (South)         40) Singapore
 7) Brunei                24) Kuwait                41) Sri Lanka
 8) Cambodia              25) Kyrgyzstan            42) Syria
 9) China                 26) Laos                  43) Taiwan
10) Cyprus                27) Lebanon               44) Tajikistan
11) East Timor            28) Macau                 45) Thailand
12) Georgia               29) Malaysia              46) Turkmenistan
13) Hong Kong             30) Mongolia              47) United Arab Emirates
14) India                 31) Myanmar (Burma)       48) Uzbekistan
15) Indonesia             32) Nepal                 49) Vietnam
16) Iran                  33) Oman                  50) Yemen
17) Iraq                  34) Pakistan
#? 9

```
4.选择时区：北京时间
```

Please select one of the following time zone regions.
1) east China - Beijing, Guangdong, Shanghai, etc.
2) Heilongjiang (except Mohe), Jilin
3) central China - Sichuan, Yunnan, Guangxi, Shaanxi, Guizhou, etc.
4) most of Tibet & Xinjiang
5) west Tibet & Xinjiang
#? 1

```
5.确认验证：
```

The following information has been given:

        China
        east China - Beijing, Guangdong, Shanghai, etc.

Therefore TZ='Asia/Shanghai' will be used.
Local time is now:      Tue Dec 17 18:22:10 CST 2013.
Universal Time is now:  Tue Dec 17 10:22:10 UTC 2013.
Is the above information OK?
1) Yes
2) No
#? 1

You can make this change permanent for yourself by appending the line
        TZ='Asia/Shanghai'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Shanghai

```
成之后，他会有一个提示，告诉你将
TZ='Asia/Shanghai'; export TZ
插入到～/.profile末尾：
cat >>~/.profile<<EOF
` TZ='Asia/Shanghai'; export TZ `
EOF

6.复制文件到/etc目录下
` sudo cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime `

7、原来系统读取的bios时间默认设置为UTC时区，可是我查看了对应的` /etc/default/rcS `配置
# assume that the BIOS clock is set to UTC time (recommended)
` UTC=no `
8.更新时间
sudo ntpdate cn.pool.ntp.org
cn.pool.ntp.org是位于中国的公共NTP服务器，用来同步你的时间(如果你的时间与服务器的时间截不同的话，可能无法同步时间哟，甚至连sudo reboot这样的指令也无法执行)。

9、修改` /etc/timezone `
内容为Asia/Shanghai

然后重启下系统，看到时间能正常同步了。

总结一下，其实主要是修改：

其它解决方案参考：http://www.cnblogs.com/qima/archive/2012/10/26/2740649.html
##################################################分隔线#################################################################################

附录：改时间（没有必要，只是记录这个命令）
```

sudo date -s MM/DD/YY //修改日期
sudo date -s hh:mm:ss //修改时间

```
在修改时间以后，修改硬件CMOS的时间
```

sudo hwclock --systohc //非常重要，如果没有这一步的话，后面时间还是不准

```

