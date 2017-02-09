title: python学习笔记059 

#  python中 class 或对象属性转化成dict 、dict转换成对象 
**class和对象的属性有所区别,如下：**
```

class A(object):  
...   def __init__(self):  
...     self.b = 1  
...     self.c = 2  
>>> a.__dict__  
{'c': 2, 'b': 1}

```
```

class A(object):  
	b=1
	c=2
>>> a.__dict__  
{}

>>>A.__dict__
{'__module__': '__main__', '__dict__': <attribute '__dict__' of 'A' objects>, '__weakref__': <attribute '__weakref__' of 'A' objects>, 'c': 2, 'b': 1, '__doc__': None}

>>>dir(a)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'b', 'c']

```

 一、对象属性转化成dict 
```

>>> class A(object):  
...   def __init__(self):  
...     self.b = 1  
...     self.c = 2  
...   def do_nothing(self):  
...     pass  
...  
>>> a = A()  
>>> a.__dict__  
{'c': 2, 'b': 1}  

```

二、把类属性(包括方法)转化为dict：
```

>>> class Foo(object):  
...     bar = 'hello'  
...     baz = 'world'  
...  
>>> f = Foo()  
>>> [name for name in dir(f) if not name.startswith('__')]  
[ 'bar', 'baz' ]  
>>> dict((name, getattr(f, name)) for name in dir(f) if not name.startswith('__'))   
{ 'bar': 'hello', 'baz': 'world' }  

```
可以改进上述方法，只返回数据属性，不包括方法：
```

dict((name, getattr(f, name)) for name in dir(f)   
       if not name.startswith('__')  and not callable(value)) 

```

三、dict 转化成object
```

>>> d  
{'a': 1, 'b': {'c': 2}, 'd': ['hi', {'foo': 'bar'}]}  
>>> def obj_dic(d):  
    top = type('new', (object,), d)  
    seqs = tuple, list, set, frozenset  
    for i, j in d.items():  
        if isinstance(j, dict):  
            setattr(top, i, obj_dic(j))  
        elif isinstance(j, seqs):  
            setattr(top, i,   
                type(j)(obj_dic(sj) if isinstance(sj, dict) else sj for sj in j))  
        else:  
            setattr(top, i, j)  
    return top  
  
>>> x = obj_dic(d)  
>>> x.a  
1  
>>> x.b.c  
2  
>>> x.d[1].foo  
'bar'  

```

http://blog.csdn.net/chenyulancn/article/details/8203763
http://stackoverflow.com/questions/1305532/convert-Python-dict-to-object
http://stackoverflow.com/questions/61517/python-dictionary-from-an-objects-fields