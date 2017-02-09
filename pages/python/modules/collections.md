author:xbynet
title:Python之Collections模块 
modifyAt:2016-12-29 14:41:15
location:python/modules/collections
createAt:2016-12-29 11:57:53

collections是Python内建的一个集合模块，提供了许多有用的集合类。

* **namedtuple**()	factory function for creating tuple subclasses with named fields
* **deque**	list-like container with fast appends and pops on either end
* **ChainMap**	dict-like class for creating a single view of multiple mappings
* **Counter**	dict subclass for counting hashable objects
* **OrderedDict**	dict subclass that remembers the order entries were added
* **defaultdict**	dict subclass that calls a factory function to supply missing values
* UserDict	wrapper around dictionary objects for easier dict subclassing
* UserList	wrapper around list objects for easier list subclassing
* UserString	wrapper around string objects for easier string subclassing

Collections Abstract Base Classes to the `collections.abc` module

#  namedtuple 
我们知道tuple可以表示不变集合，例如，一个点的二维坐标就可以表示成：
p = (1, 2)
但是，看到(1, 2)，很难看出这个tuple是用来表示一个坐标的。
定义一个class又小题大做了，这时，namedtuple就派上了用场：
```

>>> from collections import namedtuple
>>> Point = namedtuple('Point', ['x', 'y'])
>>> p = Point(1, 2)
>>> p.x
1
>>> p.y
2

```
**namedtuple是一个函数，它用来创建一个自定义的tuple对象，并且规定了tuple元素的个数，并可以用属性而不是索引来引用tuple的某个元素。**
这样一来，我们用namedtuple可以很方便地定义一种数据类型，**它具备tuple的不变性，又可以根据属性来引用**，使用十分方便。
可以验证创建的Point对象是tuple的一种子类：
```

>>> isinstance(p, Point)
True
>>> isinstance(p, tuple)
True

```

http://xmgu2008.blog.163.com/blog/static/1391223802014226103430480/
#  deque 
**使用list存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为list是线性存储，数据量大的时候，插入和删除效率很低。**
**deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈**：
```
>>> from collections import deque
>>> q = deque(['a', 'b', 'c'])
>>> q.append('x')
>>> q.appendleft('y')
>>> q
deque(['y', 'a', 'b', 'c', 'x'])

```
**deque除了实现list的append()和pop()外，还支持appendleft()和popleft()，这样就可以非常高效地往头部添加或删除元素。**
class collections.deque([iterable[, maxlen]]) **maxlen指定队列的最大长度**。当deque达到最大长度full时，新元素仍然会被添加进来，但是对应会从另一端移除旧元素。

值得一提的是，**`deque是线程安全的，也就是说你可以同时从deque集合的左边和右边进行操作而不会有影响`**

#  defaultdict 
使用dict时，如果引用的Key不存在，就会抛出KeyError。**如果希望key不存在时，返回一个默认值，就可以用defaultdict**：
```

>>> from collections import defaultdict
>>> dd = defaultdict(lambda: 'N/A')
>>> dd['key1'] = 'abc'
>>> dd['key1'] # key1存在
'abc'
>>> dd['key2'] # key2不存在，返回默认值
'N/A'

```
注意默认值是调用函数返回的，而函数在创建defaultdict对象时传入。
除了在Key不存在时返回默认值，defaultdict的其他行为跟dict是完全一样的。
#  OrderedDict 
使用dict时，Key是无序的。在对dict做迭代时，我们无法确定Key的顺序。
**如果要保持Key的顺序，可以用OrderedDict：**
```

>>> from collections import OrderedDict
>>> d = dict([('a', 1), ('b', 2), ('c', 3)])
>>> d # dict的Key是无序的
{'a': 1, 'c': 3, 'b': 2}
>>> od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
>>> od # OrderedDict的Key是有序的
OrderedDict([('a', 1), ('b', 2), ('c', 3)])
注意，OrderedDict的Key会按照插入的顺序排列，不是Key本身排序：
>>> od = OrderedDict()
>>> od['z'] = 1
>>> od['y'] = 2
>>> od['x'] = 3
>>> list(od.keys()) # 按照插入的Key的顺序返回
['z', 'y', 'x']

```

**popitem**(last=True) 如果last=True可以实现LIFO,如果为False可以实现FIFO（先进先出)。
**move_to_end**(key, last=True)  
```
>>> d = OrderedDict.fromkeys('abcde')
>>> d.move_to_end('b')
>>> ''.join(d.keys())
'acdeb'
>>> d.move_to_end('b', last=False)
>>> ''.join(d.keys())
'bacde'
```
**OrderedDict可以实现一个FIFO（先进先出）的dict**，当容量超出限制时，先删除最早添加的Key：
```

from collections import OrderedDict
class LastUpdatedOrderedDict(OrderedDict):

    def __init__(self, capacity):
        super(LastUpdatedOrderedDict, self).__init__()
        self._capacity = capacity

    def __setitem__(self, key, value):
        containsKey = 1 if key in self else 0
        if len(self) - containsKey >= self._capacity:
            last = self.popitem(last=False)
            print('remove:', last)
        if containsKey:
            del self[key]
            print('set:', (key, value))
        else:
            print('add:', (key, value))
        OrderedDict.__setitem__(self, key, value)

```
#  Counter 
**Counter(是dict的一个子类)是一个简单的计数器**，例如，统计字符出现的个数。
```
>>> # Tally occurrences of words in a list
>>> cnt = Counter()
>>> for word in ['red', 'blue', 'red', 'green', 'blue', 'blue']:
...     cnt[word] += 1
>>> cnt
Counter({'blue': 3, 'red': 2, 'green': 1})

>>> # Find the ten most common words in Hamlet
>>> import re
>>> words = re.findall(r'\w+', open('hamlet.txt').read().lower())
>>> Counter(words).most_common(10)
[('the', 1143), ('and', 966), ('to', 762), ('of', 669), ('i', 631),
 ('you', 554),  ('a', 546), ('my', 514), ('hamlet', 471), ('in', 451)]
```
实例化
class collections.Counter([iterable-or-mapping])类是dict的一个子类，key还是key，但是value是一个整数值，用来计数，甚至可以为0，负数等。
```
>>> c = Counter()                           # a new, empty counter
>>> c = Counter('gallahad')                 # a new counter from an iterable
>>> c = Counter({'red': 4, 'blue': 2})      # a new counter from a mapping
>>> c = Counter(cats=4, dogs=8)             # a new counter from keyword args
```

与dict不同之处在于，对于不存在的key不会报KeyError异常，而是返回值为0。
```
>>> c = Counter(['eggs', 'ham'])
>>> c['bacon']                              # count of a missing element is zero
0
```
删除元素还是通过del来删除。

Counter还提供了以下方法：
elements()：返回的是根据计数而重复元素的列表。如下：
```
>>> c = Counter(a=4, b=2, c=0, d=-2)
>>> sorted(c.elements())
['a', 'a', 'a', 'a', 'b', 'b']
```
most_common([n])：用于计算列表中元素出现的次数，最多取前n个，如下：
```
>>> Counter('abracadabra').most_common(3)  
[('a', 5), ('r', 2), ('b', 2)]
```
subtract([iterable-or-mapping])：对两个counter做元素间减法，如下:
```
>>> c = Counter(a=4, b=2, c=0, d=-2)
>>> d = Counter(a=1, b=2, c=3, d=4)
>>> c.subtract(d)
>>> c
Counter({'a': 3, 'b': 0, 'c': -3, 'd': -6})
```
fromkeys(iterable)：没有提供该方法的实现
update([iterable-or-mapping])：与subtract类似，只不过是作加法。

集合间操作：
```
>>> c = Counter(a=3, b=1)
>>> d = Counter(a=1, b=2)
>>> c + d                       # add two counters together:  c[x] + d[x]
Counter({'a': 4, 'b': 3})
>>> c - d                       # subtract (keeping only positive counts)
Counter({'a': 2})
>>> c & d                       # intersection:  min(c[x], d[x]) 
Counter({'a': 1, 'b': 1})
>>> c | d                       # union:  max(c[x], d[x])
Counter({'a': 3, 'b': 2})
```
一元操作
```
>>> c = Counter(a=2, b=-4)
>>> +c
Counter({'a': 2})
>>> -c
Counter({'b': 4})
```

#  queue 
queue.Queue模块提供了一个适用于多线程编程的先进先出数据结构，可以用来安全的传递多线程信息。
我们先来创建一个Queue模块的队列对象：
```

>>> from queue import Queue 
>>> q =Queue(maxsize=10)
>>>

```
上面的代码中，我们先导入了Queue模块，之后创建了一个叫做变量Q的队列对象,相当于创建了一个队列，这个队列有一个可参数masize，可以设置队列有长度，设置为“-1”，就可以让队列达到无限。
我们可以将一个数值放入队列中去：
```

>>> q.put(10)
>>>

```
put()方法可以在队列的尾部插入一个项目，它有2个参数，一个是需要插入的项，第二个默认参数值为1，方法让线程暂停，直到空出一个数据单元项，如果参数为0，会出发Full Python的异常。
有进有出，我们将刚才插入的值，再用get()方法取出来：
```

>>> q.get()
>>>

```
q这个对象的get()方法可以从队列头部删除而且返回一个项目，有一个可选参数，默认值是真，也就是True。**get()就使调用线程暂停，直至有项目可用,如果队列为空且block为False，队列将引发Empty的Python异常。**

# ChainMap
collections.ChainMap可以将多个dict封装成对外的一个单元(但是内部还是分开存储的)
Operations on ChainMap

1. **keys**() :- display all the keys of all the dictionaries in ChainMap.
2. **values**() :- display values of all the dictionaries in ChainMap.
3. **maps** :-  display keys with corresponding values of all the dictionaries in ChainMap.
4. **new_child**() :- adds a new dictionary **in the beginning of** the ChainMap.
5. **reversed**() :- This function reverses the relative ordering of dictionaries in the ChainMap.


```
import collections
 
# initializing dictionaries
dic1 = { 'a' : 1, 'b' : 2 }
dic2 = { 'b' : 3, 'c' : 4 }
 
# initializing ChainMap
chain = collections.ChainMap(dic1, dic2)
 
# printing chainMap using maps
print ("All the ChainMap contents are : ")
print (chain.maps)
 
# printing keys using keys()
print ("All keys of ChainMap are : ")
print (list(chain.keys()))
 
# printing keys using keys()
print ("All values of ChainMap are : ")
print (list(chain.values()))
```
输出如下：
All the ChainMap contents are : 
[{'b': 2, 'a': 1}, {'c': 4, 'b': 3}]
All keys of ChainMap are : 
['a', 'c', 'b']
All values of ChainMap are : 
[1, 4, 2]

**注意：b存在与dic1和dic2中，但是取值时取得是dic1中的b，这与构建ChainMap的顺序有关。**

```
import collections
 
# initializing dictionaries
dic1 = { 'a' : 1, 'b' : 2 }
dic2 = { 'b' : 3, 'c' : 4 }
dic3 = { 'f' : 5 }
 
# initializing ChainMap
chain = collections.ChainMap(dic1, dic2)
 
# printing chainMap using map
print ("All the ChainMap contents are : ")
print (chain.maps)
 
# using new_child() to add new dictionary
chain1 = chain.new_child(dic3)
 
# printing chainMap using map
print ("Displaying new ChainMap : ")
print (chain1.maps)
 
# displaying value associated with b before reversing
print ("Value associated with b before reversing is : ",end="")
print (chain1['b'])
 
# reversing the ChainMap
chain1.maps = reversed(chain1.maps)
 
# displaying value associated with b after reversing
print ("Value associated with b after reversing is : ",end="")
print (chain1['b'])
Run on IDE

```

输出如下：
All the ChainMap contents are : 
[{'b': 2, 'a': 1}, {'b': 3, 'c': 4}]
Displaying new ChainMap : 
[{'f': 5}, {'b': 2, 'a': 1}, {'b': 3, 'c': 4}]
Value associated with b before reversing is : 2
Value associated with b after reversing is : 3

ChainMap在当我们需要保持默认值的时候特别有用。类似于js当中的$.extend({a:1,b:2},{a:12}),如果没有传入b，那么就用默认值b为2.
```
import argparse
import os
 
from collections import ChainMap
 
 
def main():
    app_defaults = {'username':'admin', 'password':'admin'}
 
    parser = argparse.ArgumentParser()
    parser.add_argument('-u', '--username')
    parser.add_argument('-p', '--password')
    args = parser.parse_args()
    command_line_arguments = {key:value for key, value 
                              in vars(args).items() if value}
 
    chain = ChainMap(command_line_arguments, os.environ, 
                     app_defaults)
    print(chain['username'])
 
if __name__ == '__main__':
    main()
    os.environ['username'] = 'test'
    main()
```


参考：http://www.geeksforgeeks.org/chainmap-in-python/
http://www.blog.pythonlibrary.org/2016/03/29/python-201-what-is-a-chainmap/
https://pymotw.com/3/collections/chainmap.html