title: python集合 

#  python集合 
[部分集合讲解](http://wiki.xby1993.net/doku.php?id=python:python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B011#集合)
在已经学过的数据类型中：
  * 能够索引的，如list/str，其中的元素可以重复
  * 可变的，如list/dict，即其中的元素/键值对可以原地修改
  * 不可变的，如str/int，即不能进行原地修改
  * 无索引序列的，如dict，即其中的元素（键值对）没有排列顺序

set，翻译过来叫做“集合”。它的特点是：有的可变，有的不可变；元素无次序，不可重复。

tuple算是list和str的杂合(杂交的都有自己的优势,上一节的末后已经显示了),
那么set则可以堪称是list和dict的杂合.
下面通过实验,进一步理解创建set的方法:

```

>>> s1 = set("qiwsir")  
>>> s1                  
set(['q', 'i', 's', 'r', 'w'])
特别注意观察:qiwsir中有两个i，但是在s1中,只有一个i,也就是集合中元素不能重复。
>>> a=set([1,2,3])
>>> a
{1, 2, 3}
除了用set()来创建集合。还可以使用{}的方式，但是这种方式不提倡使用，因为在某些情况下，python搞不清楚是字典还是集合。
>>> a={"1",2}  #通过{}直接创建
>>> a
{'1', 2}

分别用list()和set()能够实现两种数据类型之间的转化。
>>> s1 =set(['q', 'i', 's', 'r', 'w'])
>>> lst = list(s1)
>>> lst
['q', 'i', 's', 'r', 'w']

```
<WRAP center round tip 60%>
利用set()建立起来的集合是可变集合，可变集合都是unhashable类型的。
</WRAP>
##  集合方法 
add, update，pop, remove, discard, clear
set.pop()是从set中任意选一个元素,删除并将这个值返回.但是,不能指定删除某个元素.
set.remove(obj)中的obj,必须是set中的元素,否则就报错
跟remove(obj)类似的还有一个discard(obj):discard(obj)中的obj如果是set中的元素,就删除,如果不是,就什么也不做,
在删除上还有一个绝杀,就是set.clear(),

##  不变的集合 
以set()来建立集合，这种方式所创立的集合都是可原处修改的集合，或者说是可变的，也可以说是unhashable
还有一种集合，不能在原处修改。这种集合的创建方法是用` frozenset() `，顾名思义，这是一个被冻结的集合，当然是不能修改了，那么这种集合就是hashable类型——可哈希。
```

>>> f_set = frozenset("qiwsir")
>>> f_set
frozenset(['q', 'i', 's', 'r', 'w'])
>>> f_set.add("python")             #报错，不能修改，则无此方法
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'frozenset' object has no attribute 'add'

```
##  集合运算 
```

元素与集合的关系
>>> aset
set(['h', 'o', 'n', 'p', 't', 'y'])
>>> "a" in aset
False

集合与集合的关系
>>> a           
set(['q', 'i', 's', 'r', 'w'])
>>> b
set(['a', 'q', 'i', 'l', 'o'])
>>> a == b
False

A是否是B的子集，或者反过来，B是否是A的超集
判断集合A是否是集合B的子集，可以使用A<B，返回true则是子集，否则不是。另外，还可以使用函数A.issubset(B)判断。
>>> a
set(['q', 'i', 's', 'r', 'w'])
>>> c
set(['q', 'i'])
>>> c<a     #c是a的子集
True
>>> c.issubset(a)   #或者用这种方法，判断c是否是a的子集
True
>>> a.issuperset(c) #判断a是否是c的超集
True

A、B的并集，即A、B所有元素
A | B.也可使用函数A.union(B)，得到的结果就是两个集合并集

A、B的交集，即A、B所公有的元素，
A&B, a.intersection(b)

A相对B的差（补），即A相对B不同的部分元素
a - b, a.difference(b)
A-B

```