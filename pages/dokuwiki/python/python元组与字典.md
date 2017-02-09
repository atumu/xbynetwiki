title: python元组与字典 

#  python元组与字典 
[部分Python元组](http://wiki.xby1993.net/doku.php?id=python:python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B011#python_元组)
[部分python字典](http://wiki.xby1993.net/doku.php?id=python:python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B011#python_字典_dictionary)
##  ` tuple `（元组） 

` 它的特点就是其中的元素不能更改 `.它的元素又可以是任何类型的数据，这点上跟list相同
```

#如果这样写，就会是...元组（1,2,3）
>>> a=1,2,3
>>> a
(1, 2, 3)
>>> type(a)
<class 'tuple'>

```
**索引和切片：略**

**tuple用在哪里？**
既然它是list和str的杂合，它有什么用途呢？不是用list和str都可以了吗？
在很多时候，的确是用list和str都可以了。
一般认为,tuple有这类特点,并且也是它使用的情景:
  * Tuple 比 list 操作速度快。如果您定义了一个值的**常量集**，并且唯一要用它做的是不断地遍历它，请使用 tuple 代替 list。
  * 如果对**不需要修改的数据**进行 “写保护”，可以使代码更安全。说明这一数据是常量。如果必须要改变这些值，则需要执行 tuple 到 list 的转换 (需要使用一个特殊的函数)。
  * Tuples 可以在** dictionary（字典，后面要讲述） 中被用做 key，**但是 list 不行。Dictionary key 必须是不可变的。Tuple 本身是不可改变的，但是如果您有一个 list 的 tuple，那就认为是可变的了，用做 dictionary key 就是不安全的。只有字符串、整数或其它对 dictionary 安全的 tuple 才可以用作 dictionary key。
  * Tuples 可以用在**字符串格式化**中。

##  字典dict 
在一个dict中，键是唯一的，不能重复。值则是对应于键，值可以重复。
字典是可变的。需要提醒注意的是，在字典中的**“键”，必须是不可变的数据类型**；“值”可以是任意数据类型。
```

>>> person = {"name":"qiwsir","site":"qiwsir.github.io","language":"python"}

```
  - **基本操作**
字典虽然跟列表有很大的区别，但是依然有不少类似的地方。它的基本操作：
  * len(d)，返回字典(d)中的键值对的数量
  * d[key]，返回字典(d)中的键(key)的值
  * d[key]=value，将值(value)赋给字典(d)中的键(key)
  * del d[key]，删除字典(d)的键(key)项（将该键值对删除）
  * key in d，检查字典(d)中是否含有键为key的项

  - **字符串格式化输出**
```

>>> city_code = {"suzhou":"0512", "tangshan":"0315", "hangzhou":"0571"}
>>> " Suzhou is a beautiful city, its area code is %(suzhou)s" % city_code
' Suzhou is a beautiful city, its area code is 0512'

```

  - **模板**
```

>>> temp = "<html><head><title>%(lang)s<title><body><p>My name is %(name)s.</p></body></head></html>"
>>> my = {"name":"qiwsir", "lang":"python"}
>>> temp % my
'<html><head><title>python<title><body><p>My name is qiwsir.</p></body></head></html>'

```

  - **字典方法**
```

>>> ad = {"name":"qiwsir", "lang":"python"}
>>> id(ad)
3072239652L
>>> cd = ad.copy()
>>> cd
{'lang': 'python', 'name': 'qiwsir'}
>>> id(cd)
3072239788L

在python中，有一个“深拷贝”(deep copy)。不过，要用下一import来导入一个模块。这个东西后面会讲述，前面也遇到过了。
>>> import copy
>>> z = copy.deepcopy(x)
>>> z
{'lang': ['python', 'java'], 'name': 'qiwsir'}
>>> id(x["lang"])
3072243276L
>>> id(z["lang"])
3072245068L

>>> a = {"name":"qiwsir"}
>>> a.clear()
>>> a
{}

>>> d
{'lang': 'python'}
>>> d.get("lang")
'python'

>>> print d.get("name")
None
>>> d["name"]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'name'
这就是dict.get()和dict['key']的区别。

```
` items/iteritems, keys/iterkeys, values/itervalues `
这个标题中列出的是三组dict的函数，并且这三组有相似的地方。
```

>>> dd = {"name":"qiwsir", "lang":"python", "web":"www.itdiffer.com"}
>>> dd_kv = dd.items()
>>> dd_kv
[('lang', 'python'), ('web', 'www.itdiffer.com'), ('name', 'qiwsir')]

>>> dd
{'lang': 'python', 'web': 'www.itdiffer.com', 'name': 'qiwsir'}
>>> dd_iter = dd.iteritems()
>>> type(dd_iter)
<type 'dictionary-itemiterator'>
>>> dd_iter
<dictionary-itemiterator object at 0xb72b9a2c>
>>> list(dd_iter)
[('lang', 'python'), ('web', 'www.itdiffer.com'), ('name', 'qiwsir')]
得到的dd_iter的类型，是一个'dictionary-itemiterator'类型，不过这种迭代器类型的数据不能直接输出，必须用list()转换一下，才能看到里面的真面目。

>>> dd
{'lang': 'python', 'web': 'www.itdiffer.com', 'name': 'qiwsir'}
>>> dd.keys()
['lang', 'web', 'name']
>>> dd.values()
['python', 'www.itdiffer.com', 'qiwsir']


```

` pop, popitem `
```

>>> dd
{'lang': 'python', 'web': 'www.itdiffer.com', 'name': 'qiwsir'}
>>> dd.pop("name")
'qiwsir'

有意思的是D.popitem()倒是跟list.pop()有相似之处，不用写参数（list.pop是可以不写参数），但是，D.popitem()不是删除最后一个，前面已经交代过了，dict没有顺序，也就没有最后和最先了，它是随机删除一个，并将所删除的返回。
>>> dd
{'lang': 'python', 'web': 'www.itdiffer.com'}
>>> dd.popitem()
('lang', 'python')
>>> dd
{'web': 'www.itdiffer.com'}
成功地删除了一对，注意是随机的，不是删除前面显示的最后一个。并且返回了删除的内容，返回的数据格式是tuple


```

` update `
```

>>> d1 = {"lang":"python"}
>>> d2 = {"song":"I dreamed a dream"}
>>> d1.update(d2)
>>> d1
{'lang': 'python', 'song': 'I dreamed a dream'}
>>> d2
{'song': 'I dreamed a dream'}

```

` has_key `判断字典中是否存在某个键
```

>>> d2
{'web': 'itdiffer.com', 'name': 'qiwsir', 'song': 'I dreamed a dream'}
>>> d2.has_key("web")
True
>>> "web" in d2
True

```