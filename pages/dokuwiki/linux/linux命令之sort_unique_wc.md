title: linux命令之sort_unique_wc 

#  Linux命令之sort,unique,wc,cut 
##  文本排序sort 
1.sort排序是根据从输入行抽取的一个或多个关键字进行比较来完成的。排序关键字定义了用来排序的最小的字符序列。缺省情况下以整行为关键字按”ASCII字符顺序”进行排序。
```

[root@www ~]# sort [-fbMnrtuk] [file or stdin]

```
选项与参数：
  * -f  ：忽略大小写的差异，例如 A 与 a 视为编码相同；
  * -b  ：忽略最前面的空格符部分；
  * -M  ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
  * -n  ：使用『纯数字』进行排序(默认是以文字型态来排序的)；
  * -r  ：反向排序；
  * -u  ：就是 uniq ，相同的数据中，仅出现一行代表；
  * -t  ：分隔符，默认是用 [tab] 键来分隔；
  * -k  ：以那个区间 (field) 来进行排序的意思
```

[root@server74 sort]# cat sort_default     ##示例文件
23
123
345
111
333
[root@server74 sort]# sort sort_default
111
123
23
333
345

```
Sort默认排序并非按数值大小，而是按ASCII字符顺序依次排序，若第一个字符相同，则比较第二个字符，直到出现不相同字符，用升序进行排列。

----

若按数值大小进行排序，则需要用到sort的参数：-n：--numeric-sort数值顺序
```

sort -n sort_default     //若需逆序排序，加-r(reverse)参数
23
111
123
333
345

```

----

对某个文件的某个特定字段进行排序，以/etc/passwd文件为例，以第三个字段为关键字段，对数值进行升序排序：
```

  -k： --key=POS1指定以哪个字段为关键字进行排序
  -t： --field-separator=SEP指定分隔符

```
```

[root@server74 sort]# sort -t ':' -k 3 -n /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

```
默认是以字符串来排序的，如果想要使用数字排序：
```

cat /etc/passwd | sort -t ':' -k 3n
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh

```
默认是升序排序，如果要倒序排序，如下
```

cat /etc/passwd | sort -t ':' -k 3nr
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
ntp:x:106:113::/home/ntp:/bin/false
messagebus:x:105:109::/var/run/dbus:/bin/false
sshd:x:104:65534::/var/run/sshd:/usr/sbin/nologin

```
如果要对/etc/passwd,先以第六个域的第2个字符到第4个字符进行正向排序，再基于第一个域进行反向排序。
```

cat /etc/passwd |  sort -t':' -k 6.2,6.4 -k 1r      
sync:x:4:65534:sync:/bin:/bin/sync
proxy:x:13:13:proxy:/bin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh

```
查看/etc/passwd有多少个shell:对/etc/passwd的第七个域进行排序，然后去重:
```

cat /etc/passwd |  sort -t':' -k 7 -u
root:x:0:0:root:/root:/bin/bash
syslog:x:101:102::/home/syslog:/bin/false
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
sshd:x:104:65534::/var/run/sshd:/usr/sbin/nologin

```
##  uniq 
uniq命令可以去除排序过的文件中的重复行，因此uniq经常和sort合用。也就是说，为了使uniq起作用，所有的重复行必须是相邻的。
[root@www ~]# uniq [-icu]
选项与参数：
  * -i   ：忽略大小写字符的不同；
  * -c  ：进行计数
  * -u  ：只显示唯一的行
```

[root@server74 sort]# cat sort_default     ##实例文件
111
44   相同且相邻
44
333
678
44    与上相同但不相邻
[root@server74 sort]# uniq sort_default
111
44
333
678
44

```
2. 显示重复行行数 -c：count 

##  文本统计wc 
man下wc命令即可查到它的很多用法，一般常用的有一下几个参数：
  * -c, --bytes：print the byte counts统计字节数
  * -m, --chars：print the character counts统计字符数
  * -w, --words：print the word counts统计字数
  * -l, --lines：print the newline counts统计行数
  * -L, --max-line-length：print the length of the longest line统计最长行的长度
```

#wc -l /etc/passwd   #统计行数，在对记录数时，很常用
40 /etc/passwd       #表示系统有40个账户

#wc -w /etc/passwd  #统计单词出现次数
45 /etc/passwd

#wc -m /etc/passwd  #统计文件的字节数
1719

```
##  cut取文本列 
cut命令可以从一个文本文件或者文本流中提取文本列。
```

[root@www ~]# cut -d'分隔字符' -f fields <==用于有特定分隔字符
[root@www ~]# cut -c 字符区间            <==用于排列整齐的信息

```
选项与参数：
  * -d  ：后面接分隔字符。与 -f 一起使用；
  * -f  ：依据 -d 的分隔字符将一段信息分割成为数段，用 -f 取出第几段的意思；
  * -c  ：以字符 (characters) 的单位取出固定字符区间；
 
PATH 变量如下
```

[root@www ~]# echo $PATH
/bin:/usr/bin:/sbin:/usr/sbin:/usr/local/bin:/usr/X11R6/bin:/usr/games
# 1 | 2       | 3   | 4       | 5            | 6            | 7

```
将 PATH 变量取出，我要找出第五个路径。
```

#echo $PATH | cut -d ':' -f 5
/usr/local/bin

```
将 PATH 变量取出，我要找出第三和第五个路径。
```

#echo $PATH | cut -d ':' -f 3,5
/sbin:/usr/local/bin

```
将 PATH 变量取出，我要找出第三到最后一个路径。
```

echo $PATH | cut -d ':' -f 3-
/sbin:/usr/sbin:/usr/local/bin:/usr/X11R6/bin:/usr/games

```
将 PATH 变量取出，我要找出第一到第三个路径。
```

#echo $PATH | cut -d ':' -f 1-3
/bin:/usr/bin:/sbin:

```
将 PATH 变量取出，我要找出第一到第三，还有第五个路径。
```

echo $PATH | cut -d ':' -f 1-3,5
/bin:/usr/bin:/sbin:/usr/local/bin

```
实用例子:只显示/etc/passwd的用户和shell
```

#cat /etc/passwd | cut -d ':' -f 1,7 
root:/bin/bash
daemon:/bin/sh
bin:/bin/sh

```
参考:http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858385.html
http://my.oschina.net/u/2408078/blog/478015
