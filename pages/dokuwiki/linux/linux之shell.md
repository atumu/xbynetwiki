title: linux之shell 

#  linux之shell 
![](/data/dokuwiki/linux/pasted/20150628-213704.png)
![](/data/dokuwiki/linux/pasted/20150628-213721.png)
![](/data/dokuwiki/linux/pasted/20150628-213733.png)
##  一． Linux基本命令 
常用命令:cp,mv,rm,mkdir,rmdir,cd,ls
1.7. su命令
这个命令非常重要。它可以让一个普通用户拥有超级用户或其他用户的权限，也可以让超级用户以普通用户的身份做一些事情。普通用户使用这个命令时必须有超级用户或其他用户的口令。
该命令的一般形式为： su [选项] [使用者帐号]
 
1.8. ps命令
显示系统中执行的程序。
语法：ps [选项]
 
1.9. kill命令
删除执行中的程序
语法：kill [选项] PID
 
1.10. grep命令
搜寻输出的特定文字
语法：grep 字符串
例：
ps aux | grep matlab
kill PID
 
1.11. echo命令
echo命令的功能是在显示器上显示一段文字，一般起到一个提示的作用。
该命令的一般格式为： echo [ -n ] 字符串
 
1.12. clear命令
clear命令的功能是清除屏幕上的信息，它类似于DOS中的 cls命令。清屏后，提示符移动到屏幕左上角。
##  Hello World 
#!/bin/sh  
echo hello,world  
【注】
- 第1行是必须的，用以表示本脚本由哪个程序来执行，此处是用 /bin/sh 程序执行
- 文本文件要使用unix/linux格式，即换行符为\n；与此对照的是，Windows下建立的文件文件是以 \r\n结尾。可以用三种方法确保这一点：
(1) 对于新手来说，可以在Linux下用vi或gedit来建立此文件，可以保证是unix格式
(2) 或者在windows下建立此文件，然后在linux使用dos2unix命令来改格式
(3) 在Windows下，用Notepad++软件进行编辑, 在菜单的"编辑 | 档案格式转换 | 转为unix格式"
##  2. 执行脚本 
脚本在书写好了之后，有几种执行方法。
(1)  sh  test.sh
这里用/bin/sh这种程序来解释执行test.sh
(2)  ./test.sh
这是把test.sh当作一个可执行文件来执行。要求：
     - test.sh有可执行属性   chmod +x test.sh
     - test.sh第一行是  #!/bin/sh
(3)  .   test.sh
点号也是可以执行脚本的。和前面的区别是，用点号执行时脚本的变量将自动输出到当前环境中。而用前面2种方法执行时，脚本中的变量不会注入到当前环境（除非显示地export）
##   shell程序的调试 
```

 sh [-nvx] scripts.sh
选项与参数：
-n  ：不要运行 script，仅查询语法的问题；
-v  ：再运行 sccript 前，先将 scripts 的内容输出到萤幕上；
-x  ：将使用到的 script 内容显示到萤幕上，这是很有用的参数！

```
范例一：测试 sh16.sh 有无语法的问题？
[root@www ~]# sh -n sh16.sh 
# 若语法没有问题，则不会显示任何资讯！

范例二：将 sh15.sh 的运行过程全部列出来～
[root@www ~]# sh -x sh15.sh 
##  script 的运行方式差异 (source, sh script, ./script) 
**利用直接运行的方式来运行 script: sh script.sh 在子bash中运行**
```

[root@www scripts]# echo $firstname $lastname
    <==确认了，这两个变量并不存在喔！
[root@www scripts]# sh sh02.sh
Please input your first name: VBird <==这个名字是鸟哥自己输入的
Please input your last name:  Tsai 

Your full name is: VBird Tsai      <==看吧！在 script 运行中，这两个变量有生效
[root@www scripts]# echo $firstname $lastname
    <==事实上，这两个变量在父程序的 bash 中还是不存在的！

```
![](/data/dokuwiki/linux/pasted/20151026-145615.png)

**利用 source 来运行脚本：在父程序中运行**
如果你使用 source 来运行命令那就不一样了！同样的脚本我们来运行看看：
```

[root@www scripts]# source sh02.sh
Please input your first name: VBird
Please input your last name:  Tsai

Your full name is: VBird Tsai
[root@www scripts]# echo $firstname $lastname
VBird Tsai  <==嘿嘿！有数据产生喔！

```
![](/data/dokuwiki/linux/pasted/20151026-145732.png)
##  3. 变量 
SHELL里的变量都是字符串
(1) 变量定义
AUTHOR_NAME=shaofa
USER_COUNT=12
【注】
- **等号两边不可以用空**
- **变量的值会被看作字符串，不会被看作数字** 【这可能有点难以理解，通常用expr函数来得到一个数字】
- **语句无需以分号结尾**
- **值不需要用引号括起来**

(2) 变量使用
$AUTHOR_NAME
或
${AUTHOR_NAME}
**用$表示取变量的值**
(3) 变量导出
export AUTHOR_NAME=shaofa
或
AUTHOR_NAME=shaofa
export AUTHOR_NAME
(4) 取消变量
unset AUTHOR_NAME
可以从当前环境变量里取消一个变量
###  变量符号意义： 
　　$0 这个程序的执行名字
　　$n 　这个程序的第n个参数值，n=1...9
　　$*　 这个程序的所有参数
　　$# 这个程序的参数个数
　　$$ 这个程序的PID
　　$! 执行上一个背景指令的PID
` $? 上一个指令的返回值 `
##  双引号及单引号 
```

$echo "$HOME $PATH"  -- 显示变量值
/home/hbwork opt/kde/bin:/usr/local/bin:
 $echo '$HOME $PATH'  -- 显示单引号里的内容
$HOME $PATH

```
**反引号：**
```

docker kill `docker ps -q`

```
##   从键盘输入变量值 

使用read命令
read var1 var2 … varn
##  4. 函数 
SHELL中也是支持函数的定义的。例如:

```

#!/bin/sh  
function my_test()  
{  
    _ARG1=$1;  
    _ARG2=$2;  
    echo "Got Argument: ${_ARG1}, ${_ARG2}"  
        return 0;  
} 
my_test  aaa  bbb  

```   
注：
- **函数的参数不会显式的列在括号里，但可以在代码里用 $1, $2 ... 引用**
- 参数的个数貌似是有限制的，应该是从1到9
- 函数调用时，把参数列在后面，以空格分开，**末尾不用加分号**
- **函数可以return一个` 整数 `，作为返回码**。也可以直接return退出函数
##  5. 条件测试 
建议参考http://www.cnblogs.com/liuling/p/2013-8-4-01.html
在if ... else, while等控制语句，必须有条件测试。
```

#!/bin/sh  
if [ -f a.txt ]; then  
    echo "File Exist."  
else  
    echo "File Not Exist."  
fi 

``` 
注意方括号内[ ]，这里就是测试条件。其中 -f a.txt表示判断a.txt是否存在。【注】` 方括号内左右都要有空格 `，不能把各部分连在一起写
**文件条件测试**
  * -d  是否为目录
  * -f   是否为文件
  * -L 是否为链接
  * -r 是否可读
  * -w 是否可写
  * -s 是否为空(长度为0)
  * -x 是否可执行
  * -u 是否有suid标志

**字符串条件测试**
  * =  字符串相同
  * != 字符串不等
  * -z 字符串为空
  * -n 字符串非空
` 字符串测试时，要把变量放在引号里 `，下面是一个例子
```

NAME=a  
if [ -z "$NAME" ]; then  
    echo "String Is Null."  
else  
    echo "String Is Not Null."  
fi 

``` 

**数值测试**
  * -eq   即=
  * -ne   即!=
  * -gt    即>
  * -lt     即<
  * -ge  即>=
  * -le   即<=
数值比较时，可以把变量放在引号，也可以不用引号

**多个条件的与或关系**
''条件与:  -a 或者 &&
条件或: -o 或者 || 
非!''
```

[ "$yn" == "Y" -o "$yn" == "y" ]
上式可替换为
[ "$yn" == "Y" ] || [ "$yn" == "y" ]

```
**&lt;nowiki&gt;[[]] 表达式&lt;/nowiki&gt;**

```

[root@localhost ~]# [ 1 -eq 1 ] && echo 'ok'           
ok
[root@localhost ~]$ [[ 2 < 3 ]] && echo 'ok' 
ok
[root@localhost ~]$ [[ 2 < 3 && 4 > 5 ]] && echo 'ok' 
ok

```
**&lt;nowiki&gt;注意：[[]] 运算符只是[]运算符的扩充。能够支持&lt;,&gt;符号运算不需要转义符，它还是以字符串比较大小。里面支持逻辑运算符：|| &amp;&amp;&lt;/nowiki&gt;**
###  6. 流程控制 
**(1)  if ... else**
```

if  [ 条件  ];  then
   ....
fi
或
if [ 条件 ]; then
   ....
elif [ 条件 ] ; then
   ....
else
   ....
fi

```

**2、case语句**
```


case  $变量名称 in   <==关键字为 case ，还有变量前有钱字号
  "第一个变量内容")   <==每个变量内容建议用双引号括起来，关键字则为小括号 )
	程序段
	;;            <==每个类别结尾使用两个连续的分号来处理！
  "第二个变量内容")
	程序段
	;;
  *)                  <==最后一个变量内容都会用 * 来代表所有其他值
	不包含第一个变量内容与第二个变量内容的其他程序运行段
	exit 1
	;;
esac                  <==最终的 case 结尾！『反过来写』思考一下！

```

**3、循环**
1) for命令
```

for var in con1 con2 con3 ...
do
	程序段
done

```
```

for...do...done 的数值处理

for (( 初始值; 限制值; 运行步阶 ))
do
	程序段
done

for (( i=1; i<=$nu; i=i+1 ))
do
	s=$(($s+$i))
done

```
2） While 命令
```

while [ condition ]
 do
循环体，只要命令返回状态为真便继续进行
 done

```
 
**3）until 命令**
```

until [ condition ]
do
	程序段落
done

```
4）break命令：退出循环，取自C语言。
##  数字运算 
&lt;nowiki&gt;利用『 $((计算式)) 』来进行数值运算的&lt;/nowiki&gt;
```

echo $(( 13 % 3 ))

```
##  逻辑运算 
test进行逻辑运算，` 用[ ]括起来就是test运算 `
int1 -eq int2 相等?
int1 -ne int2 不等?
int1 -gt int2 int1 > int2 ?
int1 -ge int2 int1 >= int2 ?
int1 -lt int2 int1 < int2 ?
int1 -le int2 int1 <= int2

 

##  语法 
**注释**
Shell编程中的注释以#开头
**1、  set命令**
当没有参数的时候，列出系统中所有的自定义变量值；当有参数的时候，重置基本参数如$1、$2等。如set `date` 将date命令的输出当作输入参数；
 
**4、shift命令**
 将参数表向左移动一个位置，$2变成$1,...，依次类推。
 
5、shell中可以嵌套命令，使用\`来保护内层命令，如`cd \`pwd\``。
 
**10、 shell模式匹配规则**
  * *      匹配任意字符串，包括空字符串
  * ?      匹配任意单字符串
  * [ABC]  匹配ABC中任意字符
  * “…”   完全与…匹配，引号保护特殊字符，也可以写成’…’
  * \C     匹配C

##  常用grep\awk\sed语法 
###  1. grep 常用方法与参数 

```

grep "关键字" 文件名
grep "关键字" 文件名1 文件名2 ..... //在指定的多个文件中查找关键字
grep "关键字" * //表示在当前目录下的所有文件中查找
grep "关键字" * -R //表示在当前目录下查找，如果有子目录则进入到子目录中查找
grep "正则表达式" * //在文件中按正则表达式查找关键字
grep -n "关键字" * //显示出关键字在文件中的行号
grep -c "关键字" * //只打印匹配的行数，不显示匹配的内容。
grep -v "关键字" 文件名//选择那些不匹配搜索条件的行
grep -i "关键字" 文件名 //忽略关键字的大小写
grep -l "关键字" * //只显示查找到的文件，不显示关键字

```
###  2. sed 

（1）基本知识
```

sed -e '编辑指令1' -e '编辑指令2' ... 文件档 //基本格式
-e 表式后边跟的是编辑命令
编辑命令由两部分组成. [address1[,address2]]function[argument] 其中, 地址参数 address1 、address2 为行数或 regular expression 字符串 , 表示所执行编辑的资料行 ; 函数参数 function[argument] 为 sed 的内定函数 , 表示执行的编辑动作。
（2）sed选项
-n 不打印；s e d不写编辑行到标准输出，缺省为打印所有行（编辑和未编辑）。p命令可以用来打印编辑行。
-f 如果正在调用s e d脚本文件，使用此选项。此选项通知s e d一个脚本文件支持所有的s e d命令，例如：sed -f myscript.sed input_file，这里m y s c r i p t . s e d即为支持s e d命令的文件。
-c 下一命令是编辑命令。使用多项编辑时加入此选项。如果只用到一条s e d命令，此选项无用，但指定它也没有关系。
-i 编辑原文件(此选项慎用,如果使用则原文件就会被修改,无法恢复)。

```
(3)地址
```

sed -e '10d' filename //删除档内第 10 行资料 , 则指令为 10d。
sed -e '/man/d filename //删除含有 "man" 字符串的资料行时 , 则指令为 /man/d。
sed -e '1,3d' //删除档内第 1 行到第 3 行资料, 则指令为 1,3d。
sed -e '1,/man/d' filename //删除档内第 1 行到含 "man" 字符串的资料行
sed -e '/man/, 3d' filename //删除档内含 "man"行到第 3 的资料行
sed -e '/man1/,/man2/d' filename //删除档内第含"man1"行到含“man2" 的资料行

```
（4）编辑命令
  * p 打印匹配行
  * = 显示文件行号
  * a\ 在定位行号后附加新文本信息
  * i\ 在定位行号后插入新文本信息
  * d 删除定位行
  * c\ 用新文本替换定位文本
  * s 使用替换模式替换相应模式
  * r 从另一个文件中读文本
  * w 写文本到一个文件
  * q 第一个模式匹配完成后推出或立即推出
  * l 显示与八进制A S C I I代码等价的控制字符
  * { } 在定位行执行的命令组
  * n 从另一个文件中读文本下一行，并附加在下一行
  * g 将模式2粘贴到/pattern n/
  * y 传送字符
  * n 延续到下一输入行；允许跨行的模式匹配语句
```

sed -e '/machine/s/phi/beta/g' filename //在filename中搜索包含machine的行，然后用beta替换phi。
sed -e '5c\ 
Those must often wipe a bloody nose. 
' filename 
//将第5行替换为 Those must often wipe a bloody nose. ，其中c后边的"\"是连字符。
sed -e '1,100c\ 
How are you?\ 
data be deleted! 
' filename
//将文件中 1 至 100 行的资料区 , 替换成输入的两行。
sed -e '/man/w filename2' filename1 //搜索man所在行，写到 filename2中
sed -e '/man/r filename2' filename1 //将filename2中的内容读到man所在行
sed -i "s/查找的关键字/替换的词/g" 文件名将文件名中所有关键字替换成指定字符串

```
###  3. awk 

awk 用法：awk ‘ pattern {action} ‘
变量名含义
  * ARGC 命令行变元个数
  * ARGV 命令行变元数组
  * FILENAME 当前输入文件名
  * FNR 当前文件中的记录号
  * FS 输入域分隔符，默认为一个空格
  * RS 输入记录分隔符
  * NF 当前记录里域个数
  * NR 到目前为止记录数
  * OFS 输出域分隔符
  * ORS 输出记录分隔符
1、
```

awk ‘/101/’ file 显示文件file中包含101的匹配行。
awk ‘/101/,/105/’ file
awk ‘$1 == 5′ file
awk ‘$1 == “CT”‘ file 注意必须带双引号
awk ‘$1 * $2 >100 ‘ file
awk ‘$2 >5 && $2<=15' file

```
2、
```

awk '{print NR,NF,$1,$NF,}' file 显示文件file的当前记录号、域数和每一行的第一个和最后一个域。
awk '/101/ {print $1,$2 + 10}' file 显示文件file的匹配行的第一、二个域加10。
awk '/101/ {print $1$2}' file
awk '/101/ {print $1 $2}' file 显示文件file的匹配行的第一、二个域，但显示时域中间没有分隔符。

```
3、
```

df | awk '$4>1000000 ‘ 通过管道符获得输入，如：显示第4个域满足条件的行。

```
4、
```

awk -F “|” ‘{print $1}’ file 按照新的分隔符”|”进行操作。
awk ‘BEGIN { FS=“[: \t|]” } {print $1,$2,$3}’ file 通过设置输入分隔符（FS=“[: \t|]”）修改输入分隔符。
Sep=“|”
awk -F $Sep ‘{print $1}’ file 按照环境变量Sep的值做为分隔符。
awk -F ‘[ :\t|]’ ‘{print $1}’ file 按照正则表达式的值做为分隔符，这里代表空格、:、TAB、|同时做为分隔符。
awk -F ‘[][]’ ‘{print $1}’ file 按照正则表达式的值做为分隔符，这里代表[、]

```
5、
```

awk -f awkfile file 通过文件awkfile的内容依次进行控制。
cat awkfile /101/{print “\047 Hello! \047″} –遇到匹配行以后打印‘ Hello! ‘.\047代表单引号。
{print $1,$2} –因为没有模式控制，打印每一行的前两个域。

```
```

6、
awk ‘$1 ~ /101/ {print $1}’ file 显示文件中第一个域匹配101的行（记录）。
7、
awk ‘BEGIN { OFS=“%”} {print $1,$2}’ file 通过设置输出分隔符（OFS=“%”）修改输出格式。
8、
awk ‘BEGIN { max=100 ;print “max=“ max} BEGIN 表示在处理任意行之前进行的操作。{max=($1 > max ?$1:max); print $1,”Now max is “max}’ file 取得文件第一个域的最大值。
（表达式1?表达式2:表达式3 相当于：
if (表达式1)
表达式2
else
表达式3
awk ‘{print ($1>4 ? “high “$1: “low “$1)}’ file

```
```

9、
awk ‘$1 * $2 >100 {print $1}’ file 显示文件中第一个域匹配101的行（记录）。
10、
awk ‘{$1 == ‘Chi’ {$3 = ‘China’; print}’ file 找到匹配行后先将第3个域替换后再显示该行（记录）。
awk ‘{$7 %= 3; print $7}’ file 将第7域被3除，并将余数赋给第7域再打印。
11、
awk ‘/tom/ {wage=$2+$3; printf wage}’ file 找到匹配行后为变量wage赋值并打印该变量。
12、
awk ‘/tom/ {count++;}
END {print “tom was found “count” times”}’ file END表示在所有输入行处理完后进行处理。

```
```

13、
awk ‘gsub(/\$/,”");gsub(/,/,”"); cost+=$4; END {print “The total is $” cost>“filename”}’ file gsub函数用空串替换$和,再将结果输出到filename中。
1 2 3 $1,200.00
1 2 3 $2,300.00
1 2 3 $4,000.00
awk ‘{gsub(/\$/,”");gsub(/,/,”");
if ($4>1000&&$4<2000) c1+=$4;
else if ($4>2000&&$4<3000) c2+=$4;
else if ($4>3000&&$4<4000) c3+=$4;
else c4+=$4; }
END {printf "c1=[%d];c2=[%d];c3=[%d];c4=[%d]\n",c1,c2,c3,c4}"' file
通过if和else if完成条件语句
awk '{gsub(/\$/,"");gsub(/,/,"");
if ($4>3000&&$4<4000) exit;
else c4+=$4; }
END {printf "c1=[%d];c2=[%d];c3=[%d];c4=[%d]\n",c1,c2,c3,c4}"' file
通过exit在某条件时退出，但是仍执行END操作。
awk '{gsub(/\$/,"");gsub(/,/,"");
if ($4>3000) next;
else c4+=$4; }
END {printf “c4=[%d]\n”,c4}”‘ file
通过next在某条件时跳过该行，对下一行执行操作。

```
```

14、
awk ‘{ print FILENAME,$0 }’ file1 file2 file3>fileall 把file1、file2、file3的文件内容全部写到fileall中，格式为打印文件并前置文件名。
15、
awk ‘ $1!=previous { close(previous); previous=$1 }
{print substr($0,index($0,” “) +1)>$1}’ fileall 把合并后的文件重新分拆为3个文件。并与原文件一致。
16、
awk ‘BEGIN {“date”|getline d; print d}’ 通过管道把date的执行结果送给getline，并赋给变量d，然后打印。
17、
awk ‘BEGIN {system(“echo \”Input your name:\\c\”"); getline d;print “\nYour name is”,d,”\b!\n”}’
通过getline命令交互输入name，并显示出来。
awk ‘BEGIN {FS=“:”; while(getline< "/etc/passwd" >0) { if($1~”050[0-9]_”) print $1}}’
打印/etc/passwd文件中用户名包含050x_的用户名。
18、
awk '{ i=1;while(i<NF) {print NF,$i;i++}}' /etc/passwd

```
```

19、
在awk中调用系统变量必须用单引号，如果是双引号，则表示字符串
Flag=abcd
awk ‘{print ‘$Flag’}’ 结果为abcd
awk ‘{print “$Flag”}’ 结果为$Flag

```

##  Shell实战部分 
###  交互式脚本：变量内容由使用者决定 
```

#!/bin/bash
# Program:
#	User inputs his first name and last name.  Program shows his full name.
# History:
# 2005/08/23	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input your first name: " firstname  # 提示使用者输入
read -p "Please input your last name:  " lastname   # 提示使用者输入
echo -e "\nYour full name is: $firstname $lastname" # 结果由萤幕输出

```
###  随日期变化：利用 date 进行文件的创建 
```

#!/bin/bash
# Program:
#	Program creates three files, which named by user's input 
#	and date command.
# History:
# 2005/08/23	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

# 1. 让使用者输入文件名称，并取得 fileuser 这个变量；
echo -e "I will use 'touch' command to create 3 files." # 纯粹显示资讯
read -p "Please input your filename: " fileuser         # 提示使用者输入

# 2. 为了避免使用者随意按 Enter ，利用变量功能分析档名是否有配置？
filename=${fileuser:-"filename"}           # 开始判断有否配置档名

# 3. 开始利用 date 命令来取得所需要的档名了；
date1=$(date --date='2 days ago' +%Y%m%d)  # 前两天的日期
date2=$(date --date='1 days ago' +%Y%m%d)  # 前一天的日期
date3=$(date +%Y%m%d)                      # 今天的日期
file1=${filename}${date1}                  # 底下三行在配置档名
file2=${filename}${date2}
file3=${filename}${date3}

# 4. 将档名创建吧！
touch "$file1"                             # 底下三行在创建文件
touch "$file2"
touch "$file3"

```
###  数值运算：简单的加减乘除 
&lt;nowiki&gt;利用『 $((计算式)) 』来进行数值运算的&lt;/nowiki&gt;
```

#!/bin/bash
# Program:
#	User inputs 2 integer numbers; program will cross these two numbers.
# History:
# 2005/08/23	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "You SHOULD input 2 numbers, I will cross them! \n"
read -p "first number:  " firstnu
read -p "second number: " secnu
total=$(($firstnu*$secnu))
echo -e "\nThe result of $firstnu x $secnu is ==> $total"

```
###  避免定时任务脚本的常见问题 
很多脚本在实际使用的时候往往是**以定时任务的方式运行，而非手工运行**。
以定时任务方式运行的脚本往往会遇到以下几个问题。
  * ` 路径问题 `：**当前目录往往不是脚本文件所在目录**。因此，脚本在引用其使用的外部文件，如配置文件和其它脚本文件时**，无法方便得使用相对路径**。
  * ` 命令找不到问题 `：脚本中使用到的一些外部命令，在手工执行脚本的时候可以正常调用。但是在定时任务下运行则可能出现脚本解析器找不到相关命令的问题。
  * ` 脚本重复运行问题 `：**一次脚本的执行未结束，而下一次脚本的运行已经开始。导致系统中有多个进程在同时运行同一个脚本。**
下面分享定时任务脚本开发中上述几个常见问题的处理方法。
**路径问题**
定时任务下当前路径往往不是脚本文件所在目录。因此我们**需要用绝对路径来引用**。即先获取脚本所在目录，然后以该目录为基础采用绝对路径的方式去引用脚本所需的外部文件。方法如下面代码所示。
清单 1. 获取脚本文件所在路径
```

#!/usr/bin/ksh
echo "Current path is: `pwd`"
scriptPath=`dirname $0` #获取脚本所在路径
echo "The script is located at: $scriptPath"
cat "$scriptPath/readme" #使用绝对路径引用外部文件

```
将清单 1 中的脚本置于目录/opt/demo/scripts/auto-task 下，并在 cron 中添加该脚本。定时任务运行输出如下。
Current path is: /home/viscent
The script is located at: /opt/demo/scripts/auto-task

**命令找不到问题**
定时任务下运行的脚本可能出现脚本解析器找不到相关命令的问题。这是因为**脚本在定时任务下执行时脚本是由非登录式 Shell 来执行的**，并且执行脚本的父 Shell 并非 Oracle 用户的 Shell。因此，此时 Oracle 用户的.profile 文件并没有被调用。故解决的方法是在脚本的开头添加以下代码：
清单 2. 解决找不到外部命令问题
source /home/oracle/.profile
也就说，对于外部命令找不到的问题，可以通过在脚本的开头加一个 source 用户的.profile 文件的语句来解决。

**脚本重复运行问题**
定时任务脚本的另外一个常见问题是脚本重复运行的问题。比如，一个脚本被设置为每 5 分钟运行一次。若某一次该脚本的运行无法在 5 分钟内结束的话，定时任务服务仍然会新启一个进程来执行该脚本。这时就出现了运行同一个脚本的多个进程。而这可能导致脚本功能紊乱。并且浪费了系统资源。 **避免脚本重复运行的方法通常有两种。一是在脚本执行时先检查系统是否存在运行该脚本的其它进程。若存在，则终止当前脚本的运行。二是，脚本运行时检查系统中是否存在其它进程运行该脚本。若存在，则结束那个进程（此方法有一定风险，慎用！）**。这两种方法均需要在脚本的开头检查系统是否已经存在运行当前脚本的进程，若存在这样的进程则获取该进程的 PID。示例代码如下清单 3 所示。
清单 3. 防止脚本重复运行方法 1
```

#!/usr/bin/ksh
main(){
selfPID="$$"
scriptFile="$0"

typeset existingPid
existingPid=`getExistingPIDs $selfPID "$scriptFile"`

if [ ! -z "$existingPid" ]; then
  echo "The script already running, exiting..."
  exit -1
fi

doItsTask

}


#获取除本身进程以外其它运行当前脚本的进程的 PID
getExistingPIDs(){
selfPID="$1"
scriptFile="$2"

ps -ef | grep "/usr/bin/ksh ${scriptFile}" | grep -v "grep" | awk "{ if(\$2!=$selfPID) print \$2 }"
}

doItsTask(){
echo "Task is now being executed..."
sleep 20  #睡眠 20s，以模拟脚本在执行需要长时间完成的任务
}

main $*

```
清单 4. 防止脚本重复运行方法 2
```

#!/usr/bin/ksh

main(){
selfPID="$$"
scriptFile="$0"

typeset existingPid
existingPid=`getExistingPIDs $selfPID "$scriptFile"`

if [ ! -z "$existingPid" ]; then
  echo "The script already running, killing it..."
  kill -9 "$existingPid" #此方法有一定风险，慎用！
fi

doItsTask

}

#获取除本身进程以外其它运行当前脚本的进程的 PID
getExistingPIDs(){
selfPID="$1"
scriptFile="$2"
ps -ef | grep "/usr/bin/ksh ${scriptFile}" | grep -v "grep" | awk "{ if(\$2!=$selfPID) print \$2 }"
}

doItsTask(){
echo "Task is now being executed..."
sleep 20  #睡眠 20s，以模拟脚本在执行需要长时间完成的任务
}

main $*

```
参考：
http://blog.csdn.net/iamshaofa/article/details/7925938
http://blog.sina.com.cn/s/blog_5b1acf750101g9gn.html
http://energykey.iteye.com/blog/997695
http://www.linuxidc.com/Linux/2013-07/87047.htm
http://www.ibm.com/developerworks/cn/linux/1309_huangwh_linuxshell/