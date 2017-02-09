title: linux命令之grep与find_sed_awk 

#  Linux命令之grep与find,sed,awk 
##  grep 
Linux系统中grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。
[root@www ~]# grep [-acinv] [--color=auto] '搜寻字符串' filename
**选项与参数：**
-a ：将 binary 文件以 text 文件的方式搜寻数据
-c ：计算找到 '搜寻字符串' 的次数
-i ：忽略大小写的不同，所以大小写视为相同
-n ：顺便输出行号
-v ：亦即显示出没有 '搜寻字符串' 内容的那一行！即不匹配搜索
-V   --version   #显示版本信息。   
--color=auto ：可以将找到的关键词部分加上颜色的显示喔！
-d <动作>      --directories=<动作>   #当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。
-r   --recursive   #此参数的效果和指定“-d recurse”参数相同。 迭代搜索目录  
-E: 使用扩展的正则表达式。grep -E = egrep
----

**pattern正则表达式主要参数：**
```

^  #行的开始 如：'^grep'匹配所有以grep开头的行。    
$  #行的结束 如：'grep$'匹配所有以grep结尾的行。    
.  #匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。    
*  #匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。    
.*   #一起用代表任意字符。   
[]   #匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。    
[^]  #匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。    
\(..\)  #标记匹配字符，如'\(love\)'，love被标记为1。    
\<      #锚定单词的开始，如:'\<grep'匹配包含以grep开头的单词的行。    
\>      #锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。    
x\{m\}  #重复字符x，m次，如：'0\{5\}'匹配包含5个o的行。    
x\{m,\}  #重复字符x,至少m次，如：'o\{5,\}'匹配至少有5个o的行。    
x\{m,n\}  #重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配5--10个o的行。   
\w    #匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。   
\W    #\w的反置形式，匹配一个或多个非单词字符，如点号句号等。   
\b    #单词锁定符，如: '\bgrep\b'只匹配grep。 

``` 

**Egrep 扩展正则表达式**
. 以单个字符
[] 指定范围内的任意字符
[^] 指定范围内的非任意字符
* 匹配前边的字符任意次
+ 匹配其前边的字符串至少一次
？ 出现了0次或1次 正则表达式为\?
{m,n}匹配前边字符最少m次，最多n次
**() 分组，用法与grep类似**
**a|b:二选一**
\< 锚定单词词首
\> 锚定单词词尾

----
grep的一些参数命令说明：
1.grep -c option file：显示文件中包含搜索内容行数，比如前面的命令表示显示 file中包含option内容的行数是几
2. grep -n option flie：列出所有的匹配行，并在最前面添加行的序列数
3. grep -v option file:显示文件中不包含所搜索内容的行数，这个和-c的参数正好相反
4. gep -i option file：列出所搜索内容的匹配行，搜索过程中不区分大小写
**5. grep -l option *：列出所有包含option内容的文件的名**
**6. grep -r option :对当前目录和所有的子目录进行搜索**
**7. grep -w option file：精确搜索**，可以说准确性搜索，比如：grep -w b* a.txt：此命令执行时，*不会默认为任何字符，只表示字面意思，就是一个*字符.
8. grep -x option file：完全匹配输出，比如：grep -x hello a.txt，只会输出某一行存在hello字符串，并且此行仅包含hello的内容。假设a.txt中有一行“hello all”，执行上述命令，此行不会被搜索到。
----
5．使用实例：
实例1：查找指定进程
```

[root@localhost ~]# ps -ef|grep svn

```
实例2：查找指定进程个数
```

ps -ef|grep -c svn

```
实例3：从文件中读取关键词进行搜索
```

cat test.txt | grep -f test2.txt #输出test.txt文件中含有从test2.txt文件中读取出的关键词的内容行

```
实例5：从文件中查找关键词
```

grep 'linux' test.txt

```
实例6：从多个文件中查找关键词
```

grep 'linux' test.txt test2.txt

```
实例12：显示包含ed或者at字符的内容行
```

cat test.txt |grep -E "ed|at"

```
参考:http://www.linuxidc.com/Linux/2013-07/87919p2.htm
http://www.cnblogs.com/peida/archive/2012/12/17/2821195.html

##  find 
Linux下查找文件的命令有两个;**locate 和 find**
首先说下locate，locate这个命令是对其生成的数据库进行遍历（生成数据库的命令：**updatedb**）,这一特性决定了用locate查找文件速度很快，但是locate命令只能对文件进行模糊匹配，在精确度上来说差了点，简单介绍下它的两个选项：
#locate 
-i        //查找文件的时候不区分大小写 比如：locate  –i   passwd
 -n       //只显示查找结果的前N行     比如：locate  -n  5   passwd
下面重点说下find，find在不指定查找目录的情况下是对整个系统进行遍历查找

```

find   path   -option   [   -print ]   [ -exec   -ok   command ]   {} \;

```
[查找完执行的action]
```

 -print                                 //默认情况下的动作
 -ls                                     //查找到后用ls 显示出来
-exec   command   {} \;      —–将查到的文件执行command操作,{} 和 \;之间有空格
-ok 和-exec相同，只不过在操作前要询用户

```
![](/data/dokuwiki/linux/pasted/20160621-100713.png)
` 这里要注意{}的使用：替代查找到的文件 `
![](/data/dokuwiki/linux/pasted/20160621-100729.png)
```

find  /tmp  -atime  +30  –exec rm –rf  {}  \； #删除查找到的超过30天没有访问过文件

```
----

**1．使用name选项：**
文件名选项是find命令最常用的选项，要么单独使用该选项，要么和其他选项一起使用。  可以使用某种文件名模式来匹配文件，**记住要用引号将文件名模式引起来。**  不管当前路径是什么，如果想要在自己的根目录$HOME中查找文件名符合*.log的文件，使用~作为 'pathname'参数，波浪号~代表了你的$HOME目录。
find ~  -name "*.log" -print  
想要在当前目录及子目录中查找所有的‘ *.log‘文件，可以用： 
find . -name "*.log" -print  
想要的当前目录及子目录中查找文件名以一个大写字母开头的文件，可以用：  
find . -name "[A-Z]*" -print  
想要在/etc目录中查找文件名以host开头的文件，可以用：  
find /etc -name "host*" -print  
想要查找$HOME目录中的文件，可以用：  
find ~ -name "*" -print 或find . -print  
要想让系统高负荷运行，就从根目录开始查找所有的文件。  
find /  -name "*" -print  
如果想在当前目录查找文件名以一个个小写字母开头，最后是4到9加上.log结束的文件：  
命令：
find . -name "[a-z]*[4-9].log" -print
```

[root@localhost test]# find . -name "[a-z]*[4-9].log" -print
./log2014.log
./log2015.log
./test4/log2014.log

```

----

**2．用perm选项：**
按照**文件权限模式**用-perm选项,按文件权限模式来查找文件的话。最好使用八进制的权限表示法。  
如在当前目录下查找文件权限位为755的文件，即文件属主可以读、写、执行，其他用户可以读、执行的文件，可以用：  
```

find . -perm 755 -print

```

----

**3．忽略某个目录：**
如果在查找文件时希望忽略某个目录，因为你知道那个目录中没有你所要查找的文件，那么可以使用-prune选项来指出需要忽略的目录。
**在使用-prune选项时要当心，因为如果你同时使用了-depth选项，那么-prune选项就会被find命令忽略。**如果希望在test目录下查找文件，但不希望在test/test3目录下查找，可以用：  
命令：
```

find test -path "test/test3" -prune -o -print

```
避开多个文件夹:
```

find test \( -path test/test4 -o -path test/test3 \) -prune -o -print

```
其中 -a 和 -o ,–not都是短路求值，与 shell 的 && 和 ||,! 类似

----

**按文件属主查找文件：**
实例1：在$HOME目录中查找文件属主为peida的文件 
命令：
find ~ -user peida -print  
find /apps -group abc -print  
----
**按照更改时间或访问时间等查找文件：**
如果希望按照更改时间来查找文件，可以使用mtime,atime或ctime选项。如果系统突然没有可用空间了，很有可能某一个文件的长度在此期间增长迅速，这时就可以用mtime选项来查找这样的文件。  
用减号-来限定更改时间在距今n日以内的文件，而用加号+来限定更改时间在距今n日以前的文件。  
希望在系统根目录下查找更改时间在5日以内的文件，可以用：
find / -mtime -5 -print
为了在/var/adm目录下查找更改时间在3日以前的文件，可以用:
find /var/adm -mtime +3 -print

----
**8．查找比某个文件新或旧的文件：**
如果希望查找更改时间比某个文件新但比另一个文件旧的所有文件，可以使用-newer选项。
它的一般形式为：  
newest_file_name ! oldest_file_name  
其中，！是逻辑非符号。  
实例1：查找更改时间比文件log2012.log新但比文件log2017.log旧的文件
命令：
```

find -newer log2012.log ! -newer log2017.log

```

----

**9．使用type选项：**
实例1：在/etc目录下查找所有的目录  
find /etc -type d -print  
实例2：在当前目录下查找除目录以外的所有类型的文件  
find . ! -type d -print  
实例3：在/etc目录下查找所有的符号链接文件
find /etc -type l -print

----
**10．使用size选项：**
可以按照文件长度来查找文件，这里所指的文件长度既可以用块（block）来计量，也可以用字节来计量。**以字节计量文件长度的表达形式为N c**；以块计量文件长度只用数字表示即可。  
在按照文件长度查找文件时，一般使用这种以字节表示的文件长度，在查看文件系统的大小，因为这时使用块来计量更容易转换。  
实例1：在当前目录下查找文件长度大于1 M字节的文件  
find . -size +1000000c -print
实例2：在/home/apache目录下查找文件长度恰好为100字节的文件:  
find /home/apache -size 100c -print  
实例3：在当前目录下查找长度超过10块的文件（一块等于512字节） 
find . -size +10 -print

```

find  /tmp  -size   2M           //查找在/tmp 目录下等于2M的文件
find  /tmp  -size  +2M           //查找在/tmp 目录下大于2M的文件
find  /tmp  -size  -2M           //查找在/tmp 目录下小于2M的文件

```
----
**11．使用depth选项：**
在使用find命令时，可能希望先匹配所有的文件，再在子目录中查找。使用depth选项就可以使find命令这样做。这样做的一个原因就是，当在使用find命令向磁带上备份文件系统时，希望首先备份所有的文件，其次再备份子目录中的文件。  
实例1：find命令从文件系统的根目录开始，查找一个名为CON.FILE的文件。   
命令：
find / -name "CON.FILE" -depth -print
说明：
它将首先匹配所有的文件然后再进入子目录中查找

----
12．使用mount选项： 
在当前的文件系统中查找文件（不进入其他文件系统），可以使用find命令的mount选项。
实例1：从当前目录开始查找位于本文件系统中文件名以XC结尾的文件  
find . -name "*.XC" -mount -print 

参考：http://www.cnblogs.com/peida/archive/2012/11/16/2773289.html
http://www.cnblogs.com/wanqieddy/archive/2011/06/09/2076785.html
http://blog.chinaunix.net/uid-24648486-id-2998767

##  sed 
sed是一个很好的文件处理工具，本身是一个管道命令，**主要是以行为单位进行处理**，可以将数据行进行替换、删除、新增、选取等特定工作，下面先了解一下sed的用法
sed命令行格式为：
sed [-nefri] ‘command’ 输入文本    
常用选项：
```

        -n∶使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN的资料一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。
        -e∶直接在指令列模式上进行 sed 的动作编辑；
        -f∶直接将 sed 的动作写在一个档案内， -f filename 则可以执行 filename 内的sed 动作；
        -r∶sed 的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)
        -i∶直接修改读取的档案内容，而不是由萤幕输出。  

```     
常用命令：
  * a   ∶新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
  * c   ∶取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
  * d   ∶删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
  *  i   ∶插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
  *  p  ∶列印，亦即将某个选择的资料印出。**通常 p 会与参数 sed -n 一起运作～**
  *  s  ∶取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！

举例：（假设我们有一文件名为ab）
删除某行
```

     [root@localhost ruby] # sed '1d' ab              #删除第一行 
     [root@localhost ruby] # sed '$d' ab              #删除最后一行
     [root@localhost ruby] # sed '1,2d' ab           #删除第一行到第二行
     [root@localhost ruby] # sed '2,$d' ab           #删除第二行到最后一行

```
显示某行
```

.    [root@localhost ruby] # sed -n '1p' ab           #显示第一行 
     [root@localhost ruby] # sed -n '$p' ab           #显示最后一行
     [root@localhost ruby] # sed -n '1,2p' ab        #显示第一行到第二行
     [root@localhost ruby] # sed -n '2,$p' ab        #显示第二行到最后一行

```
　使用模式进行查询
```

     [root@localhost ruby] # sed -n '/ruby/p' ab    #查询包括关键字ruby所在所有行
     [root@localhost ruby] # sed -n '/\$/p' ab        #查询包括关键字$所在所有行，使用反斜线\屏蔽特殊含义

```
增加一行或多行字符串
```

sed '1a drink tea' ab  #第一行后增加字符串"drink tea"
 sed '1,3a drink tea' ab #第一行到第三行后增加字符串"drink tea"

```
　代替一行或多行
```

 sed '1c Hi' ab                #第一行代替为Hi
sed '1,2c Hi' ab             #第一行到第二行代替为Hi

```
替换一行中的某部分
　　格式：sed 's/要替换的字符串/新的字符串/g'   （要替换的字符串可以用正则表达式）
```

     [root@localhost ruby] # sed -n '/ruby/p' ab | sed 's/ruby/bird/g'    #替换ruby为bird
　  [root@localhost ruby] # sed -n '/ruby/p' ab | sed 's/ruby//g'        #删除ruby

```

参考：http://www.cnblogs.com/dong008259/archive/2011/12/07/2279897.html
http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2856901.html

##  awk 
**awk是一个强大的文本分析工具**，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。
简单来说**awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。**
awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。
使用方法
```

awk   'pattern {action}' {filenames}

```
**有三种方式调用awk**
1.命令行方式
awk [-F  field-separator]  'commands'  input-file(s)
其中，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。
在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，**默认的域分隔符是空格。**

2.shell脚本方式
将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。
相当于shell脚本首行的：#!/bin/sh
可以换成：#!/bin/awk

3.将所有的awk命令插入一个单独文件，然后调用：
awk -f awk-script-file input-file(s)
其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。

----
假设last -n 5的输出如下
```

[root@www ~]# last -n 5 <==仅取出前五行
root     pts/1   192.168.1.100  Tue Feb 10 11:21   still logged in
root     pts/1   192.168.1.100  Tue Feb 10 00:46 - 02:28  (01:41)
root     pts/1   192.168.1.100  Mon Feb  9 11:41 - 18:30  (06:48)
dmtsai   pts/1   192.168.1.100  Mon Feb  9 11:41 - 11:41  (00:00)
root     tty1                   Fri Sep  5 14:09 - 14:10  (00:01)
如果只是显示最近登录的5个帐号
#last -n 5 | awk  '{print $1}'
root
root
root
dmtsai
root

```
awk工作流程是这样的：读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键",所以$1表示登录用户，$3表示登录用户ip,以此类推。

----
如果只是显示/etc/passwd的账户
```

#cat /etc/passwd |awk  -F ':'  '{print $1}'  
root
daemon
bin
sys

```
这种是awk+action的示例，每行都会执行action{print $1}。
-F指定域分隔符为':'。

如果只是显示/etc/passwd的账户和账户对应的shell,而账户与shell之间以tab键分割
```

#cat /etc/passwd |awk  -F ':'  '{print $1"\t"$7}'
root    /bin/bash
daemon  /bin/sh
bin     /bin/sh
sys     /bin/sh

```
----
如果只是显示/etc/passwd的账户和账户对应的shell,而账户与shell之间以逗号分割,而且在所有行添加列名name,shell,在最后一行添加"blue,/bin/nosh"。
```

cat /etc/passwd |awk  -F ':'  'BEGIN {print "name,shell"}  {print $1","$7} END {print "blue,/bin/nosh"}'
name,shell
root,/bin/bash
daemon,/bin/sh
bin,/bin/sh
sys,/bin/sh
....
blue,/bin/nosh

```
awk工作流程是这样的：先执行BEGING，然后读取文件，读入有/n换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域,随后开始执行模式所对应的动作action。接着开始读入第二条记录······直到所有的记录都读完，最后执行END操作。

----

搜索/etc/passwd有root关键字的所有行
#awk -F ':'  '/root/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
这种是pattern的使用示例，匹配了pattern(这里是root)的行才会执行action(没有指定action，默认输出每行的内容)。
搜索支持正则，例如找root开头的: awk -F ':' '/^root/' /etc/passwd

----
###  awk内置变量 
awk有许多内置变量用来设置环境信息，这些变量可以被改变，下面给出了最常用的一些变量。
ARGC               命令行参数个数
ARGV               命令行参数排列
ENVIRON            支持队列中系统环境变量的使用
**FILENAME           awk浏览的文件名**
FNR                浏览文件的记录数
FS                 设置输入域分隔符，等价于命令行 -F选项
NF                 浏览记录的域的个数
NR                 已读的记录数
OFS                输出域分隔符
ORS                输出记录分隔符
RS                 控制记录分隔符
**此外,$0变量是指整条记录。$1表示当前行的第一个域,$2表示当前行的第二个域,......以此类推。**
统计/etc/passwd:文件名，每行的行号，每行的列数，对应的完整行内容:
```

#awk  -F ':'  '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd
filename:/etc/passwd,linenumber:1,columns:7,linecontent:root:x:0:0:root:/root:/bin/bash
filename:/etc/passwd,linenumber:2,columns:7,linecontent:daemon:x:1:1:daemon:/usr/sbin:/bin/sh
filename:/etc/passwd,linenumber:3,columns:7,linecontent:bin:x:2:2:bin:/bin:/bin/sh
filename:/etc/passwd,linenumber:4,columns:7,linecontent:sys:x:3:3:sys:/dev:/bin/sh

```
使用printf替代print,可以让代码更加简洁，易读
```

 awk  -F ':'  '{printf("filename:%10s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd

```

----
###  awk编程 
 变量和赋值
除了awk的内置变量，awk还可以自定义变量。
下面统计/etc/passwd的账户人数
```

awk '{count++;print $0;} END{print "user count is ", count}' /etc/passwd
root:x:0:0:root:/root:/bin/bash
......
user count is  40

```
count是自定义变量。之前的action{}里都是只有一个print,其实print只是一个语句，而action{}可以有多个语句，以;号隔开。

这里没有初始化count，虽然默认是0，但是妥当的做法还是初始化为0:
```

awk 'BEGIN {count=0;print "[start]user count is ", count} {count=count+1;print $0;} END{print "[end]user count is ", count}' /etc/passwd
[start]user count is  0
root:x:0:0:root:/root:/bin/bash
...
[end]user count is  40

```
统计某个文件夹下的文件占用的字节数
```

ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size}'
[end]size is  8657198

```
如果以M为单位显示:
ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size/1024/1024,"M"}' 
[end]size is  8.25889 M
注意，统计不包括文件夹的子目录。

----
条件语句
 awk中的条件语句是从C语言中借鉴来的，见如下声明方式：
```

if (expression) {
    statement;
    statement;
    ... ...
}

if (expression) {
    statement;
} else {
    statement2;
}

if (expression) {
    statement1;
} else if (expression1) {
    statement2;
} else {
    statement3;
}

```
统计某个文件夹下的文件占用的字节数,过滤4096大小的文件(一般都是文件夹):
```

ls -l |awk 'BEGIN {size=0;print "[start]size is ", size} {if($5!=4096){size=size+$5;}} END{print "[end]size is ", size/1024/1024,"M"}' 
[end]size is  8.22339 M

```

----

**循环语句**
awk中的循环语句同样借鉴于C语言，支持while、do/while、for、break、continue，这些关键字的语义和C语言中的语义完全相同。

----

**数组**
因为awk中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)。值和关键字都存储在内部的一张针对key/value应用hash的表格里。由于hash不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，awk也同样会自动判断其存储的是数字还是字符串。一般而言，awk中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。

显示/etc/passwd的账户
```

awk -F ':' 'BEGIN {count=0;} {name[count] = $1;count++;}; END{for (i = 0; i < NR; i++) print i, name[i]}' /etc/passwd
0 root
1 daemon
2 bin
3 sys
4 sync
5 games
......

```
这里使用for循环遍历数组
awk编程的内容极多，这里只罗列简单常用的用法，更多请参考 http://www.gnu.org/software/gawk/manual/gawk.html

参考:http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html