title: python字符串 

#  python字符串 
一部分在[python基础语法](http://wiki.xby1993.net/doku.php?id=python:python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B01#python_字符串)
python字符串str可以用以下三种表示法:
```

a="abc"
a='abc'
a="""abc
我是字符串"""

```
##  字符串连接：+ 
```

>>> a=25
>>> pring(a+"haha")
Traceback (most recent call last):
  File "<pyshell#14>", line 1, in <module>
    print(a+"haha")
TypeError: unsupported operand type(s) for +: 'int' and 'str'

>>> print(str(a)+"ahah_")
25ahah_

```
原来a对应的对象是一个int类型的，不能将它和str对象连接起来。怎么办？**这一点和Java不同**
原来，** +拼接起来的两个对象，必须是同一种类型的**。如果两个都是数字，毫无疑问是正确的，就是求和；如果都是字符串，那么就得到一个新的字符串。

**print()默认是以\n结尾的**，所以，会看到每个输出语句之后，输出内容后面自动带上了\n，于是就换行了
print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)

##  原始字符串： 
两种方式：方式一：用\转义，方式二： 由r开头引起字符串
```

>>> dos = "c:\\news"
>>> print dos
c:\news
>>> dos = r"c:\news"
>>> print dos
c:\news

```
##  索引和切片 

```

>>> lang = "study python"
>>> lang[0]
's'
>>> lang.index("p")
6
>>> lang
'study python'    #在前面“切”了若干的字符之后，再看一下该字符串，还是完整的。
>>> lang[2:9]
'udy pyt'

```
**字符串是一种序列，所有序列都有如下基本操作：**
  * len()：求序列长度
  * + ：连接2个序列
  * * : 重复序列元素
  * in :判断元素是否存在于序列中
  * max() :返回最大值
  * min() :返回最小值
  * cmp(str1,str2) :比较2个序列值是否相同

##  格式化字符串 
三种方式
```

>>> "I like %s"
'I like %s'
>>> "I like %s" % "python"
'I like python'

```
占位符	说明
%s	字符串(采用str()的显示)
%r	字符串(采用repr()的显示)
%c	单个字符
%b	二进制整数
%d	十进制整数
%i	十进制整数
%o	八进制整数
%x	十六进制整数
%e	指数 (基底写为e)
%E	指数 (基底写为E)
%f	浮点数
%F	浮点数，与上相同
%g	指数(e)或浮点数 (根据显示长度)
%G	指数(E)或浮点数 (根据显示长度)

当然，还可以在一个字符串中**设置多个占位符**，就像下面一样
```

>>> print("Suzhou is more than %d years. %s lives in here." % (2500, "qiwsir"))
Suzhou is more than 2500 years. qiwsir lives in here.
>>> s2 = "Suzhou is more than {} years. {} lives in here.".format(2500, "qiwsir") 
>>> s2
'Suzhou is more than 2500 years. qiwsir lives in here.'
这就是python非常提倡的string.format()的格式化方法，其中{}作为占位符。
这种方法真的是非常好，而且非常简单，只需要将对应的东西，按照顺序在format后面的括号中排列好，分别对应占位符{}即可。我喜欢的方法。
如果你觉得还不明确，还可以这样来做。
>>> print "Suzhou is more than {year} years. {name} lives in here.".format(year=2500, name="qiwsir") 
Suzhou is more than 2500 years. qiwsir lives in here.
真的很简洁，看成优雅。
其实，还有一种格式化的方法，被称为“字典格式化”，这里仅仅列一个例子，
>>> lang = "python"
>>> print "I love %(program)s"%{"program":lang}
I love python
列举了三种基本格式化的方法，你喜欢那种？我推荐：string.format()

```
##  常用字符串操作函数 
字符串的方法很多。可以通过dir来查看：
 ` dir(str) `
这么多，不会一一介绍，要了解某个具体的含义和使用方法，最好是使用help查看。举例：
help(str.isalpha)

  - **split**
这个函数的作用是将字符串根据某个分割符进行分割。
```

>>> a = "I LOVE PYTHON"
>>> a.split(" ")
['I', 'LOVE', 'PYTHON']

```
这是用空格作为分割，得到了一个名字叫做列表（list）的返回值

  - **去掉字符串两头的空格**
方法是：
  * S.` strip() ` 去掉字符串的左右空格
  * S.lstrip() 去掉字符串的左边空格
  * S.rstrip() 去掉字符串的右边空格
```

>>> b=" hello "    #两边有空格
>>> b.strip()
'hello'

```

  - **字符大小写的转换**
对于英文，有时候要用到大小写转换。最有名驼峰命名，里面就有一些大写和小写的参合。
在python中有下面一堆内建函数，用来实现各种类型的大小写转化
  * S.upper() #S中的字母大写
  * S.lower() #S中的字母小写
  * S.capitalize() #首字母大写
  * S.isupper() #S中的字母是否全是大写
  * S.islower() #S中的字母是否全是小写
  * S.istitle()#把S中的所有单词的第一个字母转化为大写
```

>>> a="border-bottom-color"
>>> a.title()
'Border-Bottom-Color'

```
##  字符串转成驼峰 
例如：border-bottom-color -> borderBottomColor
```

def convert(one_string,space_character):    #one_string:输入的字符串；space_character:字符串的间隔符，以其做为分隔标志
    string_list = str(one_string).split(space_character)    #将字符串转化为list
    first = string_list[0].lower()
    others = string_list[1:] 
    others_capital = [word.capitalize() for word in others]      #str.capitalize():将字符串的首字母转化为大写
    #others_capital[0:0] = [first]
     others_capital= [first]+others_capital
    hump_string = ''.join(others_capital)     #将list组合成为字符串，中间无连接符。
    return hump_string

if __name__=='__main__':
    print "the string is:ab-cd-ef"
    print "convert to hump:"
    print convert("ab-cd-ef","-")

```
##  join拼接字符串 
用“+”能够拼接字符串，但不是什么情况下都能够如愿的。比如，将列表（关于列表，后续详细说，它是另外一种类型）中的每个字符（串）元素拼接成一个字符串，并且用某个符号连接，如果用“+”，就比较麻烦了（是能够实现的，麻烦）。
用字符串的` join `就比较容易实现。
```

>>> b
'www.itdiffer.com'
>>> c = b.split(".")
>>> c
['www', 'itdiffer', 'com']
>>> ".".join(c)
'www.itdiffer.com'
>>> "*".join(c)
'www*itdiffer*com'

```