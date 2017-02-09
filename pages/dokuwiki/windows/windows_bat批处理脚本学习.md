title: windows_bat批处理脚本学习 

#  windows_bat批处理脚本学习 

#  部分实用命令列举 

##  文件和目录操作： 

ASSOC          显示或修改文件扩展名关联。
ATTRIB         显示或更改文件属性。
CD             显示当前目录的名称或将其更改。
CHDIR          显示当前目录的名称或将其更改。
COMP           比较两个或两套文件的内容。
COPY           将至少一个文件复制到另一个位置。
DEL            删除至少一个文件。
DIR            显示一个目录中的文件和子目录。
ERASE          删除一个或多个文件。
FC             比较两个文件或两个文件集并显示它们之间的不同。
FIND           在一个或多个文件中搜索一个文本字符串。
FINDSTR        在多个文件中搜索字符串。
FTYPE          显示或修改用在文件扩展名关联的文件类型。
TREE           以图形显示启动器或路径的目录结构。
TYPE           显示文本文件的内容。
XCOPY          复制文件和目录树。
MD(MKDIR  )             创建一个目录。
MOVE           将一个或多个文件从一个目录移动到另一个目录。
PRINT          打印一个文本文件。
REN（RENAME）            重新命名文件。
REPLACE        替换文件。
(RD)RMDIR          删除目录。
SORT           将输入排序。
MORE           逐屏显示输出
常用命令
BREAK          设置或清除扩展式 CTRL+C 检查。
CALL           从另一个批处理程序调用这一个。
COLOR          设置默认控制台前景和背景颜色。
DATE           显示或设置日期。
ECHO           显示消息，或将命令回显打开或关上。
ENDLOCAL       结束批文件中环境更改的本地化。
FOR            为一套文件中的每个文件运行一个指定的命令。
GOTO           将 Windows 命令解释程序指向批处理程序 中某个带标签的行。
HELP           提供 Windows 命令的帮助信息。
IF             在批处理程序中执行有条件的处理过程。
PATH           为可执行文件显示或设置搜索路径。
PAUSE          停止批处理文件的处理并显示信息。
POPD           还原由 PUSHD 保存的当前目录上一次的值。
PROMPT         改变 Windows 命令提示。
PUSHD          保存当前目录，然后对其进行更改。
REM            记录批处理文件或 CONFIG.SYS 中的注释。
SET            显示、设置或删除 Windows 环境变量。
SETLOCAL       开始用批文件改变环境的本地化。
SHIFT          调整批处理文件中可替换参数的位置。
START          打开单独视窗运行指定程序或命令。
TIME           显示或设置系统时间。
磁盘操作
CONVERT        将 FAT 卷转换成 NTFS。您不能转换 当前驱动器。
FORMAT         格式化磁盘，以便跟 Windows 使用。
LABEL          创建、更改或删除磁盘的卷标。
MKLINK         创建符号链接和硬链接
MODE           配置系统设备。
RECOVER        从损坏的磁盘中恢复可读取的信息。

##  网络操作 

netstat -a 检测端口
nslookup IP地址检测
ping 检测IP及可到达。
route print 查看路由表

##  其他 

CLS            清除屏幕。
SHUTDOWN       让机器在本地或远程正确关闭。
EXIT           退出 CMD.EXE 程序(命令解释程序)。
CMD            打开另一个 Windows 命令解释程序窗口。
TASKLIST       显示包括服务的所有当前运行的任务。
TASKKILL       终止正在运行的进程或应用程序。
TITLE          设置 CMD.EXE 会话的窗口标题。
VER            显示 Windows 的版本。
logoff 注销
net start [servicename] 启动服务
net stop [servicename] 停止服务
regedit 注册 表编辑

##  具体命令简介 

Echo 命令 
打开回显或关闭请求回显功能，或显示消息。如果没有任何参数，echo 命令将显示当前回显 设置。 
语法 
echo [{on|off}] [message] 
比如：@echo off / echo hello world 

@ 命令 
表示不显示@后面的命令。 
比如：@echo Now initializing the program,please wait a minite... 

Goto 命令 
指定跳转到标签，找到标签后，程序将处理从下一行开始的命令。 
语法：goto label （label是参数，指定所要转向的批处理程序中的行。） 

Rem 命令 
注释命令，在C语言中相当与/*--------*/,它并不会被执行，只是起一个注释的作用，便于 别人阅读和你自己日后修改。 
语法：Rem Message 
比如：@Rem Here is the description. 

Pause 命令 
运行 Pause 命令时，将显示下面的消息： Press any key to continue . . . 
比如： 
@echo off 
:begin 
copy a:*.* d：\back 
echo Please put a new disk into driver A 
pause 
goto begin 
在这个例子中，驱动器 A 中磁盘上的所有文件均复制到d:\back中。显示的注释提示您将另 一张磁盘放入驱动器 A 时，pause 命令会使程序挂起，以便您更换磁盘，然后按任意键继续
处理。 

Call 命令 
它用于调用调用批处理文件或批处理函数。如果在脚本或批处理文件外使用 Call，它将不会在命令行起作用。 
语法 
call [[Drive:][Path] FileName [BatchParameters]] [:label [arguments]] 
参数 
[Drive:}[Path] FileName 
指定要调用的批处理程序的位置和名称。filename 参数必须具有 .bat 或 .cmd 扩展名。 

start 命令 
调用外部程序，所有的DOS命令和命令行程序都可以由start命令来调用。 
常用参数： 
MIN 开始时窗口最小化 
WAIT 启动应用程序并等候它结束 
parameters 这些为传送到命令/程序的参数 

#  语法介绍 

##  系统动态变量 

动态变量，顾名思义，变量是动态的，会跟据环境的不同，在你使用的时候他的值也是不同的。
%CD% - 当前目录。
%DATE% - 当前日期。
%TIME% - 当前时间。
%RANDOM% - 得到一个十进制数字的随机数 （0 和 32767 之间的任意）
%ERRORLEVEL% - 当前 ERRORLEVEL 数值。
%USERPROFILE%-当前用户的home目录:如C:\Users\Administrator.USER-20141002FV
%TEMP%和%TMP%:当前用户的临时文件存放路径。如:C:\Users\ADMINI~1.USE\AppData\Local\Temp
%PATH%
%windir%:windows的安装目录。
##  读取用户输入 

set input=
set /p input=please input:

##  使用参数 

批处理中可以使用参数，一般从1%到 9%这九个，当有多个参数时需要用shift来移动，这 种情况并不多见，我们就不考虑它了。 另外%0表示当前批处理的文件名（全路径并含后缀）
@echo off 
if "%1"=="a" format a: 

##  set与变量设置表达式运算 

set a=100;
set /p input=prompt
set /a m=1+1
##  setlocal与endlocal命令与局部变量: 

@echo off
setlocal path=g:\programs\superapp;%path%
call superapp>c:\superapp.out
endlocal
start notepad c:\superapp.out
##  choice 命令 
 
choice 使用此命令可以让用户输入一个字符，从而运行不同的命令。使用时应该加/c:参数， 
c:后应写提示可输入的字符，之间无空格。它的返回码为1234…… 
比如: choice /c:dme defrag,mem,end 将显示 
defrag,mem,end[D,M,E]? 
示例1： 
@echo off 
choice /c:dme defrag,mem,end 
if errorlevel 3 goto defrag （应先判断数值最高的错误码） 
if errorlevel 2 goto mem 
if errotlevel 1 goto end 
:defrag 
c:\dos\defrag 
goto end 
:mem 
mem 
goto end 
:end 
echo good bye 
此文件运行后，将显示 defrag,mem,end[D,M,E]? 用户可选择d m e ，然后if语句将作出 判断，d表示执行标号为defrag的程序段，m表示执行标号为mem的程序段，e表示执行标号为end的程序段，每个程序段最后都以goto end将程序跳到end标号处，然后程序将显示good bye，文件结束。

##  If 命令 
 
if 表示将判断是否符合规定的条件，从而决定执行不同的命令。 有三种格式: 
1、if "参数" == "字符串" 待执行的命令 
2、if exist 文件名 待执行的命令 
如果有指定的文件，则条件成立，运行命令，否则运行下一句。 
如if exist config.sys edit config.sys 
3、if errorlevel / if not errorlevel 数字 待执行的命令 
如果返回码等于指定的数字，则条件成立，运行命令，否则运行下一句。 
如if errorlevel 2 goto x2 
DOS程序运行时都会返回一个数字给DOS，称为错误码errorlevel或称返回码，常见的返回 码为0、1。
4、if 和else的组合
实例1：
if exist test.ini (
echo 存在 test.ini 文件
) else (
echo 不存在 test.ini 文件
)

##  5、 if defined 存在判断 


if defined与if exist用法基本一样，但是if defined比if exist多一个用法，就是用来判断环境变量是否存在。

实例2：

if defined name (
echo name is %name%
) else (
echo name is not initial
)
set name=robin
@if defined name (
echo name is %name%
) else (
echo name is not initial
)
Pause

##  find命令 

FIND [/V] [/C] [/N] [/I] [/OFF[LINE]] "string" [[drive:][path]filename[
/V 显示所有未包含指定字符串的行。
/C 仅显示包含字符串的行数。
/N 显示行号。
/I 搜索字符串时忽略大小写。
/OFF[LINE] 不要跳过具有脱机属性集的文件。
"string" 指定要搜索的文字串，
[drive:][path]filename
指定要搜索的文件。
如果没有指定路径，FIND 将搜索键入的或者由另一命令产生的文字。 
实例1：
find /n "hubin" *.txt

##  for命令 
```

基础用法
在批处理文件中使用 FOR 命令时，指定变量请使用 %%variable 
FOR %variable IN (set) DO command [command-parameters]

  %variable  指定一个单一字母可替换的参数。
  (set)      指定一个或一组文件。可以使用通配符。
  command    指定对每个文件执行的命令。
  command-parameters
             为特定命令指定参数或命令行开关。
在批处理程序中使用 FOR 命令时，指定变量请使用 %%variable
而不要用 %variable。变量名称是区分大小写的，所以 %i 不同于 %I.
@echo off
for %%i in (c d e f) do (
cd /d %%i:
for /f "delims=" %%a in ('dir /s/b *.mp3') do (@echo %%a)
)
exit
搜索CDEF盘上所有的MP3。什么盘？cdef。什么文件？mp3。经过两层过滤检索到了硬盘上所有的MP3。
注意1：在批处理文件中使用 FOR 命令时，指定变量请使用 %%variable 
而不要用 %variable。变量名称是区分大小写的。

实例3：列出当前目录下都有哪些文件
for %%i in (*.*) do echo "%%i" 

实例4：列出当前目录下所有的文本文件
for %%i in (*.txt) do echo "%%i" 

实例5：出只用两个字符作为文件名的文本文件
for %%i in (??.txt) do echo "%%i" 

注意1：列出当前目录下各种文件的方法中，最简单的还是用dir命令，但是，从以上代码中，各位可以加深对for语句执行流程的理解（用到了通配符*和?）； 
注意2：以上代码不能列出含有隐藏或系统属性的文件。
注意3：上面列出当前目录下文件的命令，并不会列出子目录下的文件。

for /D 用法
FOR /D %variable IN (set) DO command [command-parameters]
    如果集中包含通配符，则指定与目录名匹配，而不与文件名匹配。
for /R 用法
FOR /R [[drive:]path] %variable IN (set) DO command [command- parameters] 
检查以 [drive:]path 为根的目录树，指向每个目录中的 FOR 语句。如果在 /R 后没有指定目录，则使用当前 目录。如果集仅为一个单点(.)字符，则枚举该目录树。 
实例6：列出G:\projects\work目录下所有的文本文件
for /L 用法
FOR /L %variable IN (start,step,end) DO command [command-para ]
该集表示以增量形式从开始到结束的一个数字序列。 因此，(1,1,5) 将产生序列 1 2 3 4 5，(5,-1,1) 将产生 序列 (5 4 3 2 1)。 
实例7：
for /L %%i in (1 1 5) DO (@echo NO:%%i)

for /F用法
FOR /F ["options"] %variable IN (file-set) DO command 
FOR /F ["options"] %variable IN ("string") DO command 
FOR /F ["options"] %variable IN ('command') DO command 
参数替代默认解析操作。这个带引号的字符串包括一个或多个指定不同解析选项的关键字。这些关键字为: 
eol=c -指读取文件时, 忽略以c打头的那些行
skip=n - 指读取文件时，忽略开始时的n行。 
delims=xxx - 指分隔符集。这个替换了空格和跳格键的默认分隔符集。 可以一次性指定多个分隔符号,比如：for /f "delims=.，" %%i in (test.txt) do echo %%i
tokens=x,y,m-n - 指每行的哪一项被传递到for的迭代变量 。如果要提取的内容是连续的多“节”的话，那么，连续的数字可以只写最小值和最大值，中间用短横连接起来即可，比如 tokens=1,2,3,4,5 可以简写为 tokens=1-5 。 还可以把这个表达式写得更复杂一点：tokens=1,2-5，tokens=1-3,4,5，tokens=1-4,5……
tokens=后面所接的星号具备这样的功能：字符串从左往右被切分成紧跟在*之前的数值所表示的节数之后，字符串的其余部分保持不变，整体被*所表示的一个变量接收。 
比如： tokens=1,* 中，星号前面紧跟的是数字1；第一节字符串被切分完之后，其余部分字符串不做任何切分，整体作为第二节字符串，这样，[txt2]就被切分成了两节，分别被变量%%i和变量%%j接收。 
FOR /F "eol=; tokens=2,3* delims=, " %i in (myfile.txt) do @echo %i %j %k

```

##  组合命令 

1.& ；2.&& ；3.||
##  管道命令 

1.| 命令 
Usage：第一条命令 | 第二条命令 [| 第三条命令...] 
将第一条命令的结果作为第二条命令的参数来使用，记得在unix中这种方式很常见。 
比如： 
time /t>>D:\IP.log 
netstat -n -p tcp|find ":3389">>D:\IP.log 

2.>、>>输出重定向命令 
将一条命令或某个程序输出结果的重定向到特定文件中, > 与 >>的区别在于，>会清除调原有文件中的内容后写入指定文件，而>>只会追加内容到指定文件中，而不会改动其中的内容。
3.< 、>& 、<& 
< 从文件中而不是从键盘中读入命令输入。 
>& 将一个句柄的输出写入到另一个句柄的输入中。 
<& 从一个句柄读取输入并将其写入到另一个句柄输出中。 

#  字符串处理 

基本连接操作
在DOS中，对字符串的处理其最简单是就字符串的连接,连接符为"_"：
set out_dir=.\out
set out_file_name=Appstore
set version=001
set out_file_name=%out_file_name%_%version%
echo %out_file_name%
最后变量out_file_name的值就是Appstore_001
在DOS中，对字符串的负责处理（替换和截取）其实是通过操作环境变量的字符串值来进行。

%PATH:str1=str2%
这个是替换变量值的内容 %PATH:str1=str2%这个操作就是把变量%PATH%的里的str1全部用str2替换
@echo off
set a=bbs.verybat.cn 
echo 替换前的值: "%a%" 
set var=%a:.=伤脑筋% 
echo 替换后的值: "%var%" 
pause 
执行后就会把变量%a%里面的"."全部替换为"伤脑筋"

%PATH:~10,5%

取变量PATH从第10位开始（字符从0开始），5个字符的值z做为新值。

实例3：

@echo off 
set a=0123456789
set var=%a:~1,2%  #输出12
echo %var% 
set var=%a:~3,5% #输出34567
echo %var% 
%PATH:~0,-2%
取变量PATH第0字符和倒数第2个的所值做为新值。

实例5：
@echo off 
set a=0123456789
set var=%a:~0,-2%
echo %var% 
pause
结果：01234567

字符串比较
EQU - 等于NEQ - 不等于LSS - 小于LEQ - 小于或等于GTR - 大于GEQ - 大于或等于
if后面加上/i的开关，在字母的比较中就不会区分大小写了，即，a与A是相等的。

实例7
echo %date:~0,10% //提取年月日信息 
echo %date:~-3% //提取星期几信息 
echo %time:~0,5% //提取时间中的时和分 
echo %time:~0,-3% //提取时和分和秒信息 