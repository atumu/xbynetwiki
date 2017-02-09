author:xbynet
title:Python学习笔记之collection模块
modifyAt:2016-12-29 10:01:17
location:dokuwiki/python/python学习笔记24
createAt:2016-12-29 10:01:17

#  Python学习笔记之collection模块 
collections是Python内建的一个集合模块，提供了许多有用的集合类。
##  namedtuple 
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
##  deque 
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
##  defaultdict 
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
##  OrderedDict 
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
##  Counter 
**Counter是一个简单的计数器**，例如，统计字符出现的个数：
```

>>> from collections import Counter
>>> c = Counter()
>>> for ch in 'programming':
...     c[ch] = c[ch] + 1
...
>>> c
Counter({'g': 2, 'm': 2, 'r': 2, 'a': 1, 'i': 1, 'o': 1, 'n': 1, 'p': 1})

```
Counter实际上也是dict的一个子类，上面的结果可以看出，字符'g'、'm'、'r'各出现了两次，其他字符各出现了一次。
小结
collections模块提供了一些有用的集合类，可以根据需要选用。

##  queue 
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