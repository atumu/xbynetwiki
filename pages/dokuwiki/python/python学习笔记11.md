title: python学习笔记11 

#  Python学习笔记之数据结构 
##  概述 

**Python列表**
List（列表） 是 Python 中使用最频繁的数据类型。
列表可以完成大多数集合类的数据结构实现。它支持字符，数字，字符串甚至可以包含列表（所谓嵌套）。
列表用[ ]标识。是python最通用的复合数据类型。看这段代码就明白。
列表中的值得分割也可以用到变量**[头下标:尾下标]**，就可以截取相应的列表，从左到右索引默认0开始的，从右到左索引默认-1开始，下标可以为空表示取到头或尾。
**加号（+）是列表连接运算符，星号（*）是重复操作。**如下实例：
```

list = [ 'abcd', 786 , 2.23, 'john', 70.2 ]
tinylist = [123, 'john']

print list # 输出完整列表
print list[0] # 输出列表的第一个元素
print list[1:3] # 输出第二个至第三个的元素 
print list[2:] # 输出从第三个开始至列表末尾的所有元素
print tinylist * 2 # 输出列表两次
print list + tinylist # 打印组合的列表

```
**Python元组**
元组是另一个数据类型，类似于List（列表）。
元组用"()"标识。内部元素用逗号隔开。但是**元素不能二次赋值，相当于只读列表**。
```

tuple = ( 'abcd', 786 , 2.23, 'john', 70.2 )
tinytuple = (123, 'john')

print tuple # 输出完整元组
print tuple[0] # 输出元组的第一个元素
print tuple[1:3] # 输出第二个至第三个的元素 
print tuple[2:] # 输出从第三个开始至列表末尾的所有元素
print tinytuple * 2 # 输出元组两次
print tuple + tinytuple # 打印组合的元组

```
` **以下是元组无效的，因为元组是不允许更新的。而列表是允许更新的**： `
```

#coding=utf-8
#!/usr/bin/python
tuple = ( 'abcd', 786 , 2.23, 'john', 70.2 )
list = [ 'abcd', 786 , 2.23, 'john', 70.2 ]
tuple[2] = 1000 # 元组中是非法应用
list[2] = 1000 # 列表中是合法应用

```
**Python元字典**
字典(dictionary)是除列表以外python之中最灵活的内置数据结构类型。列表是有序的对象结合，字典是无序的对象集合。
两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。
字典用"{ }"标识。字典由索引(key)和它对应的值value组成。
```

dict = {}
dict['one'] = "This is one"
dict[2] = "This is two"

tinydict = {'name': 'john','code':6734, 'dept': 'sales'}


print dict['one'] # 输出键为'one' 的值
print dict[2] # 输出键为 2 的值
print tinydict # 输出完整的字典
print tinydict.keys() # 输出所有键
print tinydict.values() # 输出所有值

```
**Python数据类型转换**
有时候，我们需要对数据内置的类型进行转换，**数据类型的转换，你只需要将数据类型作为函数名即可**。
以下几个内置的函数可以执行数据类型之间的转换。这些函数返回一个新的对象，表示转换的值。
![](/data/dokuwiki/python/pasted/20150824-084722.png)![](/data/dokuwiki/python/pasted/20150824-084743.png)
##  del 语句 

有个方法可以从列表中按给定的索引而不是值来删除一个子项： del 语句。它不同于有返回值的 pop() 方法。语句 del 还可以从列表中删除切片或清空整个列表（我们以前介绍过一个方法是将空列表赋值给列表的切片）。例如:

```

>>> a = [-1, 1, 66.25, 333, 333, 1234.5]
>>> del a[0]
>>> a
[1, 66.25, 333, 333, 1234.5]
>>> del a[2:4]
>>> a
[1, 66.25, 1234.5]
>>> del a[:]
>>> a
[]
del 也可以删除整个变量:

>>> del a

```
此后再引用命名 a 会引发错误（直到另一个值赋给它为止）。我们在后面的内容中可以看到 del 的其它用法。

##  迭代器 
```

for element in [1, 2, 3]:
    print(element)
for element in (1, 2, 3):
    print(element)
for key in {'one':1, 'two':2}:
    print(key)
for char in "123":
    print(char)
for line in open("myfile.txt"):
    print(line, end='')
这种形式的访问清晰、简洁、方便。迭代器的用法在 Python 中普遍而且统一。在后台， for 语句在容器对象中调用 iter() 。该函数返回一个定义了 __next__() 方法的迭代器对象，它在容器中逐一访问元素。没有后续的元素时， __next__() 抛出一个 StopIteration 异常通知 for 语句循环结束。你可以是用内建的 next() 函数调用 __next__() 方法；
给自己的类添加迭代器行为。定义一个 __iter__() 方法，使其返回一个带有 __next__() 方法的对象。如果这个类已经定义了 __next__() ，那么 __iter__() 只需要返回 self:
class Reverse:
    """Iterator for looping over a sequence backwards."""
    def __init__(self, data):
        self.data = data
        self.index = len(data)
    def __iter__(self):
        return self
    def __next__(self):
        if self.index == 0:
            raise StopIteration
        self.index = self.index - 1
        return self.data[self.index]
        
>>> rev = Reverse('spam')
>>> iter(rev)
<__main__.Reverse object at 0x00A1DB50>
>>> for char in rev:
...     print(char)
...
m
a
p
s

```
###   生成器---创建迭代器的简单方法 
Generator 是创建迭代器的简单而强大的工具。它们写起来就像是正规的函数，
需要**返回数据的时候使用 yield** 语句。每次 next() 被调用时，生成器回复它脱离的位置（它记忆语句最后一次执行的位置和所有的数据值）。以下示例演示了生成器可以很简单的创建出来:

```

def reverse(data):
    for index in range(len(data)-1, -1, -1):
        yield data[index]

```
```
      
>>> for char in reverse('golf'):
...     print(char)
...
f
l
o
g

```
前一节中描述了基于类的迭代器，它能作的每一件事生成器也能作到。因为自动创建了 __iter__() 和 __next__() 方法，生成器显得如此简洁。
另一个关键的功能在于两次执行之间，局部变量和执行状态都自动的保存下来。这使函数很容易写，而且比使用 self.index 和 self.data 之类的方式更清晰。
除了创建和保存程序状态的自动方法，当发生器终结时，还会自动抛出 StopIteration 异常。综上所述，这些功能使得编写一个正规函数成为创建迭代器的最简单方法。


##  Python 列表(Lists) 

序列是Python中最基本的数据结构。序列中的每个元素都分配一个数字 - 它的位置，或索引，第一个索引是0，第二个索引是1，依此类推。
Python有6个序列的内置类型，但最常见的是列表和元组。
序列都可以进行的操作包括索引，切片，加，乘，检查成员
此外，Python已经内置确定序列的长度以及确定最大和最小的元素的方法。
**列表的数据项不需要具有相同的类型**
```

list1 = ['physics', 'chemistry', 1997, 2000];
list2 = [1, 2, 3, 4, 5, 6, 7 ];

print "list1[0]: ", list1[0]
print "list2[1:5]: ", list2[1:5]

print "Value available at index 2 : "
print list[2];
list[2] = 2001;
print "New value available at index 2 : "
print list[2];

print list1;
del list1[2];
print "After deleting value at index 2 : "
print list1;

```
**Python列表脚本操作符**
列表对 + 和 * 的操作符与字符串相似。+ 号用于组合列表，* 号用于重复列表。
![](/data/dokuwiki/python/pasted/20150824-134918.png)
**Python列表截取**
Python的列表截取与字符串操作类型，如下所示：
L = ['spam', 'Spam', 'SPAM!']
![](/data/dokuwiki/python/pasted/20150824-134952.png)
**Python列表函数&方法**
![](/data/dokuwiki/python/pasted/20150824-135031.png)
![](/data/dokuwiki/python/pasted/20150824-135051.png)
##  列表推导式 

列表推导式为从序列中创建列表提供了一个简单的方法。

```

squares = [x**2 for x in range(10)]
等价于:
>>> squares = []
>>> for x in range(10):
...     squares.append(x**2)
...
>>> squares
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

```
 列表推导式由包含一个表达式的括号组成，**表达式后面跟随一个 for 子句，之后可以有零或多个 for 或 if 子句**。结果是一个列表，由表达式依据其后面的 for 和 if 子句上下文计算而来的结果构成。
```

例如，如下的列表推导式结合两个列表的元素，如果元素之间不相等的话:

>>> [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
等同于:

>>> combs = []
>>> for x in [1,2,3]:
...     for y in [3,1,4]:
...         if x != y:
...             combs.append((x, y))
...
>>> combs
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]

```
##  Python 元组 
Python的元组与列表类似，不同之处在于` 元组的元素不能修改 `。
元组使用小括号，列表使用方括号。
tup1 = ('physics', 'chemistry', 1997, 2000);
tup2 = (1, 2, 3, 4, 5, 6, 7 );

print "tup1[0]: ", tup1[0]
print "tup2[1:5]: ", tup2[1:5]

元组中的元素**值是不允许修改的，但我们可以对元组进行连接组合**，如下实例:
tup1 = (12, 34.56);
tup2 = ('abc', 'xyz');

# 以下修改元组元素操作是非法的。
# tup1[0] = 100;

# 创建一个新的元组
tup3 = tup1 + tup2;
print tup3;

**元组内置函数**
Python元组包含了以下内置函数
序号	方法及描述
1	cmp(tuple1, tuple2)
比较两个元组元素。
2	len(tuple)
计算元组元素个数。
3	max(tuple)
返回元组中元素最大值。
4	min(tuple)
返回元组中元素最小值。
5	tuple(seq)
将列表转换为元组。

##  集合 
` 注意：集合与字典语法上的区别 `
Python 还包含了一个数据类型 ——`  set ` （集合）。集合是一个**无序不重复元**素的集。基本功能包括关系测试和消除重复元素。
集合对象还支持 union（联合），intersection（交），difference（差）和 sysmmetric difference（对称差集）等数学运算。

大括号或 set() 函数可以用来创建集合。**注意：想要创建空集合，你必须使用 set() 而不是 {}。后者用于创建空字典，**我们在下一节中介绍的一种数据结构。
```

>>> basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
>>> print(basket)                      # show that duplicates have been removed
{'orange', 'banana', 'pear', 'apple'}
>>> 'orange' in basket                 # fast membership testing
True
>>> 'crabgrass' in basket
False

>>> # Demonstrate set operations on unique letters from two words
...
>>> a = set('abracadabra')
>>> b = set('alacazam')
>>> a                                  # unique letters in a
{'a', 'r', 'b', 'c', 'd'}
>>> a - b                              # letters in a but not in b
{'r', 'd', 'b'}
>>> a | b                              # letters in either a or b
{'a', 'c', 'r', 'd', 'b', 'm', 'z', 'l'}
>>> a & b                              # letters in both a and b
{'a', 'c'}
>>> a ^ b                              # letters in a or b but not both
{'r', 'd', 'b', 'm', 'z', 'l'}

```
类似 列表推导式，这里有一**种集合推导式语法:**

```

>>> a = {x for x in 'abracadabra' if x not in 'abc'}
>>> a
{'r', 'd'}

```
##  Python 字典(Dictionary) 
字典是另一种可变容器模型，且可存储任意类型对象，如其他容器模型。
字典由**键和对应值**成对组成。字典也被称作关联数组或哈希表。基本语法如下：
```

dict = {'Name': 'Zara', 'Age': 7, 'Class': 'First'};
 
print "dict['Name']: ", dict['Name'];
print "dict['Age']: ", dict['Age'];
dict['Age'] = 8; # update existing entry
dict['School'] = "DPS School"; # Add new entry
  
del dict['Name']; # 删除键是'Name'的条目
dict.clear();     # 清空词典所有条目
del dict ;        # 删除词典

```
` dict() `函数可以直接从 key-value 对中创建字典:
```

>>> dict([('sape', 4139), ('guido', 4127), ('jack', 4098)])
{'sape': 4139, 'jack': 4098, 'guido': 4127}
如果关键字都是简单的字符串，有时通过关键字参数指定 key-value 对更为方便:

>>> dict(sape=4139, guido=4127, jack=4098)
{'sape': 4139, 'jack': 4098, 'guido': 4127}

```
此外，**字典推导式**可以从任意的键值表达式中创建字典:
```


>>> {x: x**2 for x in (2, 4, 6)}
{2: 4, 4: 16, 6: 36}

```
**字典内置函数&方法**
![](/data/dokuwiki/python/pasted/20150824-135702.png)![](/data/dokuwiki/python/pasted/20150824-135819.png)
##  数据结构迭代循环技巧 
在**字典中循环**时，关键字和对应的值可以使用 ` items() ` 方法同时解读出来:

```

>>> knights = {'gallahad': 'the pure', 'robin': 'the brave'}
>>> for k, v in knights.items():
...     print(k, v)

```
在**序列中循环**时，索引位置和对应值可以使用`  enumerate() ` 函数同时得到:

```

>>> for i, v in enumerate(['tic', 'tac', 'toe']):
...     print(i, v)

```
同时循环两个或更多的序列，可以使用`  zip() ` 整体打包:
```


>>> questions = ['name', 'quest', 'favorite color']
>>> answers = ['lancelot', 'the holy grail', 'blue']
>>> for q, a in zip(questions, answers):
...     print('What is your {0}?  It is {1}.'.format(q, a))
...
What is your name?  It is lancelot.
What is your quest?  It is the holy grail.
What is your favorite color?  It is blue.

```
需要**逆向循环序列**的话，先正向定位序列，然后调用`  reversed() ` 函数:

```

>>> for i in reversed(range(1, 10, 2)):
...     print(i)
...
9
7
5
3
1

```
要按**排序后的顺序循环序列**的话，使用`  sorted() ` 函数，它不改动原序列，而是生成一个新的已排序的序列:
```

>>> basket = ['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
>>> for f in sorted(set(basket)):
...     print(f)

```
  
**若要在循环内部修改正在遍历的序列（**例如复制某些元素），**建议您首先制作副本**。在序列上循环不会隐式地创建副本。**切片表示法使这尤其方便:**

```

>>> words = ['cat', 'window', 'defenestrate']
>>> for w in words[:]:  # Loop over a slice copy of the entire list.   words[:]切片创建副本
...     if len(w) > 6:
...         words.insert(0, w)
...
>>> words
['defenestrate', 'cat', 'window', 'defenestrate']

```
