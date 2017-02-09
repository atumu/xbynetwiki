title: python列表 

#  python列表 
[关于列表部分在这里讲到](http://wiki.xby1993.net/doku.php?id=python:python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B011#python_列表_lists)
list的一大特点：可以无限大，就是说list里面所能容纳的元素数量无限
```

>>> a=['2',3,'qiwsir.github.io']
>>> a
['2', 3, 'qiwsir.github.io']
>>> type(a)
<type 'list'>
>>> bool(a)
True

```
##  索引和切片 
```

>>> a
['2', 3, 'qiwsir.github.io']
>>> a[0]    #索引序号也是从0开始
'2'
>>> a[:2]   #跟str中的类似，切片的范围是：包含开始位置，到结束位置之前
['2', 3]    #不包含结束位置
>>> a[1:]
[3, 'qiwsir.github.io']

>>> lst = ['python','java','c++']
>>> lst.index('java')
1

```
在前面讲述字符串索引和切片的时候，以及前面的演示，所有的索引都是从左边开始编号，第一个是0，然后依次增加1。此外，还有一种编号方式，
就是从右边开始，右边第一个可以编号为-1，然后向左依次是：-2,-3,...，依次类推下来。这对字符串、列表等各种序列类型都是用。
```

>>> lang
'python'
>>> lang[-1]
'n'
>>> lst
['python', 'java', 'c++']
>>> lst[-1]
'c++'

```
##  反转 
```

alst = [1,2,3,4,5,6]
>>> list(reversed(alst))
[6, 5, 4, 3, 2, 1]

```
##  对list的操作 
```

>>> lst
['python', 'java', 'c++']
>>> len(lst)
3

>>> lst
['python', 'java', 'c++']
>>> alst
[1, 2, 3, 4, 5, 6]
>>> lst + alst
['python', 'java', 'c++', 1, 2, 3, 4, 5, 6]

>>> lst
['python', 'java', 'c++']
>>> lst * 3
['python', 'java', 'c++', 'python', 'java', 'c++', 'python', 'java', 'c++']

>>> "python" in lst
True
>>> "c#" in lst
False

>>> max(alst)
6
>>> min(alst)
1

>>> lsta = [2,3]
>>> lstb = [2,4]
>>> cmp(lsta,lstb)
-1

>>> a = ["good","python","I"]      
>>> a.append("like")        #向list中添加str类型"like"
>>> a
['good', 'python', 'I', 'like']


>>> a[len(a):]=[3]      #len(a),即得到list的长度，这个长度是指list中的元素个数。
>>> a
['good', 'python', 'I', 'like', 3]

>>> a[0:0]=[2]
>>> a
[2,'good', 'python', 'I', 'like', 3]

```
##  list函数 
'append', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort'
```

>>> la
[1, 2, 3]
>>> lb
['qiwsir', 'python']
>>> la.extend(lb)
>>> la
[1, 2, 3, 'qiwsir', 'python']
list.extend(L) 等效于 list[len(list):] = L，L是待并入的list

>>> la = [1,2,1,1,3]
>>> la.count(1)
3

与list.append(x)类似，list.insert(i,x)也是对list元素的增加。只不过是可以在任何位置增加一个元素。
>>> all_users
['qiwsir', 'github', 'io']
>>> all_users.insert(0,"python")
>>> all_users
['python', 'qiwsir', 'github', 'io']

>>> lst = ["python","java","python","c"]
>>> lst.remove("python") #当删除后，发现结果只删除了第一个'python'字符串，第二个还在。
>>> lst
['java', 'python', 'c']

>>> if "python" in all_users:
...     all_users.remove("python")
...     print all_users
... else:
...     print "'python' is not in all_users"

>>> all_users
['qiwsir', 'github', 'io', 'algorithm']
>>> all_users.pop()     #list.pop([i]),圆括号里面是[i]，表示这个序号是可选的
'algorithm'             #如果不写，就如同这个操作，默认删除最后一个，并且将该结果返回
>>> all_users
['qiwsir', 'github', 'io']
>>> all_users.pop(1)        #指定删除编号为1的元素"github"
'github'
>>> all_users
['qiwsir', 'io']
简单总结一下，list.remove(x)中的参数是列表中元素，即删除某个元素；list.pop([i])中的i是列表中元素的索引值，这个i用方括号包裹起来，意味着还可以不写任何索引值，如上面操作结果，就是删除列表的最后一个。

reverse比较简单，就是把列表的元素顺序反过来。
>>> a = [3,5,1,6]
>>> a.reverse()
>>> a
[6, 1, 5, 3]
注意，是原地反过来，不是另外生成一个新的列表。所以，它没有返回值。跟这个类似的有一个内建函数reversed
因为list.reverse()不返回值，所以不能实现对列表的反向迭代，如果要这么做，可以使用reversed函数。

>>> a = [6, 1, 5, 3]
>>> a.sort()
>>> a
[1, 3, 5, 6]

在前面的函数说明中，还有一个参数key，这个怎么用呢？不知道看官是否用过电子表格，里面就是能够设置按照哪个关键字进行排序。这里也是如此。
>>> lst = ["python","java","c","pascal","basic"]
>>> lst.sort(key=len)
>>> lst
['c', 'java', 'basic', 'python', 'pascal']
对于排序，也有一个更为常用的内建函数sorted。

```
##  list和str 
list和str的最大区别是：**list是可以改变的，str不可变。**
**list和str转化**
以下涉及到的` split()和join() `
str.split():这个内置函数实现的是将str转化为list。
```

>>> s = "I am, writing\npython\tbook on line"   #这个字符串中有空格，逗号，换行\n，tab缩进\t 符号
>>> print s         #输出之后的样式
I am, writing
python  book on line
>>> s.split()       #用split(),但是括号中不输入任何参数
['I', 'am,', 'writing', 'python', 'book', 'on', 'line']
#如果split()不输入任何参数，显示就是见到任何分割符号，就用其分割了。

```
"[sep]".join(list):join可以说是split的逆运算，举例：
```

>>> name
['Albert', 'Ainstain']
>>> "".join(name)       #将list中的元素连接起来，但是没有连接符，表示一个一个紧邻着
'AlbertAinstain'
>>> ".".join(name)      #以英文的句点做为连接分隔符
'Albert.Ainstain'

```
