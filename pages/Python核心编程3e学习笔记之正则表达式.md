author:xbynet
title:Python核心编程3e学习笔记之正则表达式
modifyAt:2016-12-27 01:22:04
location:Python核心编程3e学习笔记之正则表达式
createAt:2016-12-27 01:10:03

可供参考
[Python学习笔记之正则表达式](/pages/dokuwiki/python/python学习笔记5)
[正则表达式](/pages/dokuwiki/web/正则表达式)
正则表达式的主要功能：文本模式匹配、抽取、搜索和替换
python中的`re模块`来支持正则表达式(更为强大的Perl风格 )。

搜索和匹配的区别：
匹配指的是模式匹配(pattern-matching).
搜索指在字符串**任意位置**中搜索匹配的模式；
匹配是判断一个字符串能否**从起始处**全部或者部分地匹配某个模式。

除了传统表示法之外，还支持的**扩展表示法**有：

* (?iaaa)： (?标记)，嵌入一个或多个特殊的标记参数如i,x,m,L等。也可以通过函数参数来达到同样的效果，类似于js中的/aaa/i
* (?:...)：表示一个匹配不用保存的分组.(?:\w+\.)
* (?P< name>...):分组命名匹配
* (?P=name)
* (?#...)注释
* (?=...) 如果...出现在之后的位置。
* (?!..) 如果...不出现在之后的位置
* (?<=...) 如果...出现在之前的位置
* (?<!...)如果...不出现在之前的位置
* (?(id/name)Y|N) 如果分组所提供的id或者name存在，就返回正则表达式的条件匹配Y，如果不存在，就返回N，其中|N是可选项。如(?(1)y|x)，意思是如果分组1 (\1)存在，那么接下来就匹配y，否则匹配x

分组与别名
默认数字命名分组1,2,3 引用\1 \2 \3
命名分组(?P< name>...),引用\g< name>

# re模块
核心方法和函数，还有一个正则表达式对象、正则表达式匹配对象。

* compile(pattern,flags=0)
* match(pattern,string,flags=0)
* search()
* findall()
* finditer()
* split(pattern,string,max=0)
* 
* sub(pattern,repl,string,count=0)
* subn(),除了返回替换后的字符串，还返回替换的总数
* purge()
* 
* group(num=0) 为0时返回整个匹配对象，传入几就返回第几个分组
* groups(default=None) 返回一个包含所有匹配子组的元组.
* groupsdict(default=None) 返回一个包含所有匹配的命名子组的字典。

模块属性：

* re.I,忽略大小写
* re.L 使用本地语言环境匹配
* re.M 多行模式
* re.S "."点号默认匹配除了\n换行符之外的所有单个字符，该标记可以让点号支持换行符匹配。
* re.X 提供可读性的选项。
这些标记可以通过"|"符合来组合使用

建议使用预编译re.compile()以提高性能。
**关于findall()**,总是返回一个列表，当存在子组，对于一个成功的匹配，每个子组匹配是由findall()返回的结果列表的单一元素；对于多个成功的匹配，每个子组匹配是返回的一个元组中的单一元素，而且每个元组(每个元组都对应一个成功的匹配)是结果列表中的元素。

例子：
```
bool(re.search(r'(?:(x)|y)(?(1)y|x)','xy')) #True
bool(re.search(r'(?:(x)|y)(?(1)y|x)','xx')) # False
```
```
s='This and that'
re.findall(r'th(\w+) and (th\w+)',s,re.I) #[('This','that')]
re.findall(r'th(\w+)',s,re.I) #['This','that']

[g.groups() for g in re.finditer(r'th(\w+) and (th\w+)'),s,re.I)]
[g.groups(1) for g in re.finditer(r'th(\w+)',s,re.I),s,re.I)]
```
处理日期格式转换
```
re.sub(r'(\d{1,2})/(\d{1,2})/(\d{2}|\d{4})',r'\2/\1/\3','2/20/91') # 20/2/91
```

处理DOS环境下tasklist命令的输出:
```
import os
import re

f = os.popen('tasklist /nh', 'r')
for eachLine in f:
    print re.findall(
        '([\w.]+(?: [\w.]+)*)\s\s+(\d+) \w+\s\s+\d+\s\s+([\d,]+ K)',
        eachLine.rstrip())
f.close()
```