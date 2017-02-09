author:xbynet
title:Python学习笔记之正则表达式 
modifyAt:2016-12-27 01:11:12
location:dokuwiki/python/python学习笔记5
createAt:2016-12-27 01:10:48

#  Python学习笔记之正则表达式 
` re 模块 `使 Python 语言拥有全部的正则表达式功能。
compile 函数根据一个模式字符串和可选的标志参数生成一个正则表达式对象。该对象拥有一系列方法用于正则表达式匹配和替换。
##  简单的正则表达式语法 
```

1、匹配字符
.   匹配任意除换行符，也就是“\n”以外的任何字符。
\  转义符，改变原来符号含义，后面会有演示。
[ ]  中括号用来创建一个字符集，第一个出现字符如果是^，表示反向匹配。
2、预定义字符集
\d    匹配数字，如：[0-9]
\D   与上面正好相反，匹配所有非数字字符。
\s     空白字符，如：空格，\t\r\n\f\v等。
\S    非空白字符。
\w    单词字符，如：大写A~Z，小写a~z，数字0~9。
\W   非上面这些字符。
3、可选项与重复子模式
*   匹配前一个字符0次或无限次数。
+  匹配前一个字符1次或无限次数。
?   匹配前一个字符0次或1次。
{m} 匹配前一个字符m次。
{m,n} 匹配前一个字符m至n次。

```
python re模块重要函数变量
1 )、compile() 根据正则表达式字符串，创建模式的对象。
2 )、search() 在字符串中寻找模式。
3 )、match() 在字符串开始处匹配模式。
4 )、split() 根据模式的匹配项来分割字符串。
5 )、findall() 显示出字符串中模式的所有匹配项。
6 )、sub(old,new) 方法的功能是，用将所有old的匹配项用new替换掉。
7 )、escape() 将字符串中所有特殊正则表达式字符转义。
##  函数介绍 
**re.match函数**
re.match 尝试从字符串的开始匹配一个模式。
函数语法：
re.match(pattern, string, flags=0)
flags	标志位，用于控制正则表达式的匹配方式，如：是否区分大小写，多行匹配等等。
匹配成功re.match方法返回一个匹配的对象，否则返回None。
我们可以使用` group(num) 或 groups() ` 匹配对象函数来获取匹配表达式。
group(num=0)	匹配的整个表达式的字符串，group() 可以一次输入多个组号，在这种情况下它将返回一个包含那些组所对应值的元组。
groups()	返回一个包含所有小组字符串的元组，从 1 到 所含的小组号。
```

import re

line = "Cats are smarter than dogs"

matchObj = re.match( r'(.*) are (.*?) .*', line, re.M|re.I)

if matchObj:
   print "matchObj.group() : ", matchObj.group()
   print "matchObj.group(1) : ", matchObj.group(1)
   print "matchObj.group(2) : ", matchObj.group(2)
else:
   print "No match!!"
以上实例执行结果如下：

matchObj.group() :  Cats are smarter than dogs
matchObj.group(1) :  Cats
matchObj.group(2) :  smarter

```
**re.search方法**
re.search 会在字符串内查找模式匹配，直到找到第一个匹配。
函数语法：
re.search(pattern, string, flags=0)
**re.match与re.search的区别**
` re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；而re.search匹配整个字符串，直到找到一个匹配。 `
```

import re

line = "Cats are smarter than dogs";

matchObj = re.match( r'dogs', line, re.M|re.I)
if matchObj:
   print "match --> matchObj.group() : ", matchObj.group()
else:
   print "No match!!"

matchObj = re.search( r'dogs', line, re.M|re.I)
if matchObj:
   print "search --> matchObj.group() : ", matchObj.group()
else:
   print "No match!!" 以上实例运行结果如下：
No match!!
search --> matchObj.group() :  dogs

```
###  检索和替换 
Python 的re模块提供了re.sub用于替换字符串中的匹配项。
语法：
re.sub(pattern, repl, string, max=0)
```

import re

phone = "2004-959-559 # This is Phone Number"

# Delete Python-style comments
num = re.sub(r'#.*$', "", phone)
print "Phone Num : ", num

# Remove anything other than digits
num = re.sub(r'\D', "", phone)    
print "Phone Num : ", num 以上实例执行结果如下：
Phone Num :  2004-959-559
Phone Num :  2004959559

```
**正则表达式修饰符 - 可选标志**
正则表达式可以包含一些可选标志修饰符来控制匹配的模式。修饰符被指定为一个可选的标志。
多个标志可以通过` 按位 OR(|) ` 它们来指定。如 re.I | re.M 被设置成 I 和 M 标志：
修饰符	描述
re.I	使匹配对大小写不敏感
re.L	做本地化识别（locale-aware）匹配
re.M	多行匹配，影响 ^ 和 $
re.S	使 . 匹配包括换行在内的所有字符
re.U	根据Unicode字符集解析字符。这个标志影响 \w, \W, \b, \B.
re.X	该标志通过给予你更灵活的格式以便你将正则表达式写得更易于理解。
**正则表达式模式**
反斜杠开头。。。。
略。。。

###  re.sub的各个参数的详细解释 
re.sub共有五个参数。
其中三个必选参数：pattern, repl, string
两个可选参数：count, flags

第一个参数：pattern
pattern，表示正则中的模式字符串，这个没太多要解释的。
需要知道的是：
反斜杠加数字（\N），则对应着匹配的组（matched group）.` 比如\6，表示匹配前面pattern中的第6个group `意味着，pattern中，前面肯定是存在对应的，第6个group，然后你后面也才能去引用
比如，想要处理：hello crifan, nihao crifan且此处的，前后的crifan，肯定是一样的。而想要把整个这样的字符串，换成crifanli
则就可以这样的re.sub实现替换：
```

inputStr = "hello crifan, nihao crifan";
replacedStr = re.sub(r"hello (\w+), nihao \1", "crifanli", inputStr);
print "replacedStr=",replacedStr; #crifanli

```
第二个参数：repl
repl，就是replacement，被替换的字符串的意思。**repl可以是字符串，也可以是函数。**
**如果repl是字符串的话**，其中的任何反斜杠转义字符，都会被处理的。
  * \n：会被处理为对应的换行符；
  * \r：会被处理为回车符；
  * 其他不能识别的转移字符，则只是被识别为普通的字符：
  * ` 反斜杠加g以及中括号内一个名字，即：\g<name>，对应着命了名的组，named group `
接着上面的举例：想要把对应的：hello crifan, nihao crifan中的crifan提取出来，只剩：crifan
就可以写成：
```

inputStr = "hello crifan, nihao crifan";
replacedStr = re.sub(r"hello (\w+), nihao \1", "\g<1>", inputStr);
print "replacedStr=",replacedStr; #crifan

```
对应的**带命名的组**（named group）的版本是：
```

inputStr = "hello crifan, nihao crifan";
replacedStr = re.sub(r"hello (?P<name>\w+), nihao (?P=name)", "\g<name>", inputStr);
print "replacedStr=",replacedStr; #crifan

```
**如果repl是函数**
举例说明：比如输入内容是：hello 123 world 456想要把其中的数字部分，都加上111，变成：hello 234 world 567
那么就可以写成：
```

import re;
    inputStr = "hello 123 world 456";
    def _add111(matched):
        intStr = matched.group("number"); #123
        intValue = int(intStr);
        addedValue = intValue + 111; #234
        addedValueStr = str(addedValue);
        return addedValueStr;
         
    replacedStr = re.sub("(?P<number>\d+)", _add111, inputStr);
    print "replacedStr=",replacedStr; #hello 234 world 567

```
第三个参数：string即表示要被处理，要被替换的那个string字符串。
第四个参数：count
举例说明：继续之前的例子，假如对于匹配到的内容，只处理其中一部分。
比如对于：hello 123 world 456 只是像要处理前面一个数字：123，给他加111，而不处理456，
replacedStr = re.sub("(?P<number>\d+)", _add111, inputStr, 1);

第五个参数：flags. re.I|re.M之类的。

其余资料：
http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html
http://www.iplaypython.com/module/re.html
http://www.crifan.com/python_re_sub_detailed_introduction/