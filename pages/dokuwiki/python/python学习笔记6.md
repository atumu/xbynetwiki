title: python学习笔记6 

#  Python学习笔记之序列化与JSON 
#  使用 json 存储结构化数据 
Python 允许你使用常用的数据交换格式 JSON（JavaScript Object Notation）。` 标准模块 json ` 可以接受 Python 数据结构，并将它们转换为字符串表示形式；此过程称为** 序列化**。从字符串表示形式重新构建数据结构称为** 反序列化。**序列化和反序列化的过程中，表示该对象的字符串可以存储在文件或数据中，也可以通过网络连接传送给远程的机器。
如果你有一个对象 x，你可以用简单的一行代码查看其 JSON 字符串表示形式:
` json.dumps `([1, 'simple', 'list'])
'[1, "simple", "list"]'
dumps() 函数的另外一个变体`  dump() `，直接将对象序列化到一个文件。所以如果 f 是为写入而打开的一个 文件对象，我们可以这样做:
` json.dump(x, f) `
为了重新解码对象，如果 f 是为读取而打开的 文件对象:
` x = json.load(f) `
这种简单的序列化技术可以处理列表和字典，但序列化任意类实例为 JSON 需要一点额外的努力。 json 模块的手册对此有详细的解释。

###  JSON进阶 
Python的dict对象可以直接序列化为JSON的{}，不过，很多时候，我们更喜欢用class表示对象，比如定义Student类，然后序列化：
```

import json
class Student(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

s = Student('Bob', 20, 88)
print(json.dumps(s))

```
运行代码，**毫不留情地得到一个TypeError：**
Traceback (most recent call last):
TypeError: <__main__.Student object at 0x10603cc50> is not JSON serializable
错误的原因是**Student对象不是一个可序列化为JSON的对象。**
如果连class的实例对象都无法序列化为JSON，这肯定不合理！
别急，我们仔细看看dumps()方法的参数列表，可以发现，除了第一个必须的obj参数外，dumps()方法还提供了一大堆的可选参数：
https://docs.python.org/3/library/json.html#json.dumps
**这些可选参数就是让我们来定制JSON序列化。**前面的代码之所以无法把Student类实例序列化为JSON，是因为默认情况下，dumps()方法不知道如何将Student实例变为一个JSON的{}对象。
**可选参数default就是把任意一个对象变成一个可序列为JSON的对象**，我们只需要为Student` 专门写一个转换函数 `，再把函数传进去即可：
```

def student2dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }

```
**这样，Student实例首先被student2dict()函数转换成dict，然后再被顺利序列化为JSON：**
```

>>> print(json.dumps(s, default=student2dict))
{"age": 20, "name": "Bob", "score": 88}

```
不过，下次如果遇到一个Teacher类的实例，照样无法序列化为JSON。我们可以偷个懒，把任意class的实例变为dict：
```

print(json.dumps(s, default=lambda obj: obj.__dict__))

```
因为通常class的实例都有一个__dict__属性，它就是一个dict，用来存储实例变量。也有少数例外，比如定义了__slots__的class。
同样的道理，如果我们要把JSON反序列化为一个Student对象实例，loads()方法首先转换出一个dict对象，然后，我们传入的object_hook函数负责把dict转换为Student实例：
```

def dict2student(d):
    return Student(d['name'], d['age'], d['score'])
运行结果如下：
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> print(json.loads(json_str, object_hook=dict2student))
<__main__.Student object at 0x10cd3c190>

```
打印出的是反序列化的Student实例对象。

##  序列化 
pickle模块是能够让我们直接在文件中存储几乎任何Python对象的高级工具。
```

D={'a':1,'b':2}
F=open('datafile.pk','wb')
import pickle
pickle.dump(D,F)
F.close()
解析：
F=open('datafile.pk','rb')
e=pickle.load(F)
e

```

##  把对象存储到shelve数据库中 
对象持久化通过3个标准的库模块来实现，这3个模块在Python中都可用：
  * pickle 任意Python对象和字符串之间的序列化
  * dbm 实现一个可通过键访问的文件系统，以存储字符串
  * shelve：使用dbm和pickle模块按照键把Python对象存储到一个文件中。

pickle模块是一种非常通用的对象格式化和解格式化工具。
而shelve模块提供了一个额外的层结构，允许按照键来存储pickle处理后的对象。
```

a=person('x')
import shelve
db=shelve.open('aaamdb')
db[a.__name__]=a
db.close() #记得关闭

db=shelve.open('aaamdb')
len(db)
list(db.keys())

a=db['x']
a.name='b'
db['x']=a
db.close()

```

