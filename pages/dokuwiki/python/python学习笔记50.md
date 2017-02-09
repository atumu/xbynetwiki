title: python学习笔记50 

#  Python学习笔记之字符串数字提取等操作实战 
##  字符串数字提取 
有如下几种方式：
使用filter函数
```

item = 123and456
if item.rfind('123') >= 0:
   number = filter(str.isdigit, item)

```
使用正则表达式
```

>>> import re
>>> str1 = 'balance-rr 0'
>>> mode = re.compile(r'\d+')
>>> 
>>> mode.findall(str1)
['0']
>>> str1 = '12j33jk12 ksdjfkj23jk4h1k23h'
>>> mode.findall(str1)
['12', '33', '12', '23', '4', '1', '23']
>>>

import re
s = 'speed=210,angle=150'
m = re.findall(r'([0-9]+)',s)
print（m）
输出
['210', '150']

```
使用lambda表达式
```

s = "发表于：2013-06-04" 
filter(lambda x:x.isdigit(),s)

```
##  连接字符串，列表转字符串 
连接字符串
```

delimiter = ','
mylist = ['Brazil', 'Russia', 'India', 'China']
print delimiter.join(mylist)

```
##  模拟lastIndexOf 
```

s = "C:/Python27/1/3.py"
pos = s.rfind("/")
s[:pos] # "C:/Python27/1"

```
参考:http://www.cnblogs.com/huangcong/archive/2011/08/29/2158268.html

##  正则示例 
```

import re
# Some sample text
text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'

# (a) Find all matching dates
datepat = re.compile(r'\d+/\d+/\d+')
print(datepat.findall(text))

# (b) Find all matching dates with capture groups
datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
for month, day, year in datepat.findall(text):
    print('{}-{}-{}'.format(year, month, day))

# (c) Iterative search
for m in datepat.finditer(text):
    print(m.groups())

input("")

```
输出:
['11/27/2012', '3/13/2013']
2012-11-27
2013-3-13
('11', '27', '2012')
('3', '13', '2013')
```

import re
# Some sample text
text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
# (a) Simple substitution
print(datepat.sub(r'\3-\1-\2', text))

```
输出Today is 2012-11-27. PyCon starts 2013-3-13.
