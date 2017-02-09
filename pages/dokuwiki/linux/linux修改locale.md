title: linux修改locale 

#   linux修改locale 

原文：http://wiki.ubuntu.org.cn/%E4%BF%AE%E6%94%B9locale
修改locale
[编辑]把语言环境变量改为英文
将Ubuntu系统语言环境改为英文的en_US.UTF-8

查看当前系统语言环境

locale
编辑配置文件，将zh_US.UTF-8改为en_US.UTF-8，zh改为en

sudo nano /etc/default/locale
LANG="en_US.UTF-8"
LANGUAGE="en_US:en"
继续查看更改后的系统语言变量，如果出现下列错误，说明没安装en_US的local

qii@ubuntu:~$ locale
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=en_US.UTF-8
LANGUAGE=en_US:en
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
查看系统内安装的locale

qii@ubuntu:~$ locale -a
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_COLLATE to default locale: No such file or directory
C
POSIX
zh_CN.utf8
zh_SG.utf8
看吧，没装en_US.UTF-8

安装en_US.UTF-8

sudo locale-gen en_US.UTF-8
再次查看，应该一切正常了。

qii@ubuntu:/usr/share/locales$ locale
LANG=en_US.UTF-8
LANGUAGE=en_US:en
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=