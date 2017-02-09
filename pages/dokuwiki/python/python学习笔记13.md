title: python学习笔记13 

#  Python学习笔记之XML处理 
对xml，python提供了多种模块来处理。
  * xml.dom.* 模块：Document Object Model。适合用于处理 DOM API。
  * xml.sax.* 模块：simple API for XML。由于SAX以流式读取xml文件，从而速度较快，切少占用内存，但是操作上稍复杂，需要用户实现回调函数。
  * xml.parser.expat：是一个直接的，低级一点的基于 C 的 expat 的语法分析器。 expat接口基于事件反馈，有点像 SAX 但又不太像，因为它的接口并不是完全规范于 expat 库的。
  * xml.etree.ElementTree (以下简称 ET)：元素树。它提供了轻量级的python式的API，相对于DOM，ET快了很多 ，而且有很多令人愉悦的API可以使用；相对于SAX，ET也有ET.iterparse提供了 “在空中” 的处理方式，没有必要加载整个文档到内存，节省内存。ET的性能的平均值和SAX差不多，但是API的效率更高一点而且使用起来很方便。
**所以，我用xml.etree.ElementTree**
ElementTree在标准库中有两种实现。一种是纯Python实现：xml.etree.ElementTree ，另外一种是速度快一点：xml.etree.cElementTree 。
只需要一句话import xml.etree.ElementTree as ET即可，然后由模块自动来寻找适合的方式。

##  遍历 
```

<bookstore>
    <book category="COOKING">
        <title lang="en">Everyday Italian</title> 
        <author>Giada De Laurentiis</author> 
        <year>2005</year> 
        <price>30.00</price> 
    </book>
    <book category="CHILDREN">
        <title lang="en">Harry Potter</title> 
        <author>J K. Rowling</author> 
        <year>2005</year> 
        <price>29.99</price> 
    </book>
        <book category="WEB">
        <title lang="en">Learning XML</title> 
        <author>Erik T. Ray</author> 
        <year>2003</year> 
        <price>39.95</price> 
    </book>
</bookstore>

```
![](/data/dokuwiki/python/pasted/20160313-120748.png)
将xml保存为名为22601.xml的文件，然后对其进行如下操作：
```

>>> import xml.etree.cElementTree as ET
为了简化，我用这种方式引入，如果在编程实践中，推荐读者使用try...except...方式。
>>> tree = ET.ElementTree(file="22601.xml")
>>> tree
<ElementTree object at 0xb724cc2c>

```
建立起xml解析树。然后可以通过根节点向下开始读取各个元素（element对象）。
在上述xml文档中，根元素是，它没有属性，或者属性为空。
```

>>> root = tree.getroot()      #获得根
>>> root.tag
'bookstore'
>>> root.attrib
{}
要想将根下面的元素都读出来，可以：

>>> for child in root:
...     print child.tag, child.attrib
... 
book {'category': 'COOKING'}
book {'category': 'CHILDREN'}
book {'category': 'WEB'}
也可以这样读取指定元素的信息：

>>> root[0].tag
'book'
>>> root[0].attrib
{'category': 'COOKING'}
>>> root[0].text        #无内容
'\n        '
再深点，就有感觉了：

>>> root[0][0].tag
'title'
>>> root[0][0].attrib
{'lang': 'en'}
>>> root[0][0].text
'Everyday Italian'

```
**对于ElementTree对象，有一个iter方法可以对指定名称的子节点进行深度优先遍历**。例如：
```

>>> for ele in tree.iter(tag="book"):        #遍历名称为book的节点
...     print ele.tag, ele.attrib
... 
book {'category': 'COOKING'} 
book {'category': 'CHILDREN'} 
book {'category': 'WEB'} 

>>> for ele in tree.iter(tag="title"):        #遍历名称为title的节点
...     print ele.tag, ele.attrib, ele.text
... 
title {'lang': 'en'} Everyday Italian
title {'lang': 'en'} Harry Potter
title {'lang': 'en'} Learning XML

```
如果不指定元素名称，就是将所有的元素遍历一边。
除了上面的方法，还可以通过路径，搜索到指定的元素，读取其内容。这就是xpath。此处对xpath不详解，如果要了解可以到网上搜索有关信息。
```

>>> for ele in tree.iterfind("book/title"):
...     print ele.text
... 
Everyday Italian
Harry Potter
Learning XML

```
利用findall()方法，也可以是实现查找功能：
```

>>> for ele in tree.findall("book"):
...     title = ele.find('title').text
...     price = ele.find('price').text
...     lang = ele.find('title').attrib
...     print title, price, lang
... 
Everyday Italian 30.00 {'lang': 'en'}
Harry Potter 29.99 {'lang': 'en'}
Learning XML 39.95 {'lang': 'en'}

```

##  编辑 
除了读取有关数据之外，还能对xml进行编辑，即增删改查功能。还是以上面的xml文档为例：
```

>>> root[1].tag
'book'
>>> del root[1]
>>> for ele in root:
...     print ele.tag
... 
book
book

```
如此，**成功删除了一个节点。**原来有三个book节点，现在就还剩两个了。打开源文件再看看，是不是正好少了第二个节点呢？一定很让你失望，源文件居然没有变化。
的确如此，源文件没有变化，这就对了。**因为至此的修改动作，还是停留在内存中，还没有将修改结果输出到文件**。不要忘记，我们是在内存中建立的ElementTree对象。再这样做：
```

>>> import os
>>> outpath = os.getcwd()
>>> file = outpath + "/22601.xml"
把当前文件路径拼装好。然后：
>>> tree.write(file)

```
再看源文件，已经变成两个节点了。
除了删除，也能够修改：
```

>>> for price in root.iter("price"):        #原来每本书的价格
...     print price.text
... 
30.00
39.95
>>> for price in root.iter("price"):        #每本上涨7元，并且增加属性标记
...     new_price = float(price.text) + 7
...     price.text = str(new_price)
...     price.set("updated","up")
... 
>>> tree.write(file)

```
查看源文件：
```

<bookstore>
    <book category="COOKING">
        <title lang="en">Everyday Italian</title> 
        <author>Giada De Laurentiis</author> 
        <year>2005</year> 
        <price updated="up">37.0</price> 
    </book>
    <book category="WEB">
        <title lang="en">Learning XML</title> 
        <author>Erik T. Ray</author> 
        <year>2003</year> 
        <price updated="up">46.95</price> 
    </book>
</bookstore>

```
不仅价格修改了，而且在price标签里面增加了属性标记。干得不错。
上面用del来删除某个元素，其实，在编程中，这个用的不多，更喜欢用remove()方法。比如我要删除price > 40的书。可以这么做：
```

>>> for book in root.findall("book"):
...     price = book.find("price").text
...     if float(price) > 40.0:
...         root.remove(book)
... 
>>> tree.write(file)
于是就这样了：

<bookstore>
    <book category="COOKING">
        <title lang="en">Everyday Italian</title> 
        <author>Giada De Laurentiis</author> 
        <year>2005</year> 
        <price updated="up">37.0</price> 
    </book>
</bookstore>

```
接下来就要增加元素了。
```

>>> import xml.etree.cElementTree as ET
>>> tree = ET.ElementTree(file="22601.xml")
>>> root = tree.getroot()
>>> ET.SubElement(root, "book")        #在root里面添加book节点
<Element 'book' at 0xb71c7578>
>>> for ele in root:
...    print ele.tag
... 
book
book
>>> b2 = root[1]                      #得到新增的book节点
>>> b2.text = "python"                #添加内容
>>> tree.write("22601.xml")
查看源文件：

<bookstore>
    <book category="COOKING">
        <title lang="en">Everyday Italian</title> 
        <author>Giada De Laurentiis</author> 
        <year>2005</year> 
        <price updated="up">37.0</price> 
    </book>
    <book>python</book>
</bookstore>

```


##  常用属性和方法总结 
ET里面的属性和方法不少，这里列出常用的，供使用中备查。
**Element对象**
常用属性：
  * tag：string，元素数据种类
  * text：string，元素的内容
  * attrib：dictionary，元素的属性字典
  * tail：string，元素的尾形
针对属性的操作
  * clear()：清空元素的后代、属性、text和tail也设置为None
  * get(key, default=None)：获取key对应的属性值，如该属性不存在则返回default值
  * items()：根据属性字典返回一个列表，列表元素为(key, value）
  * keys()：返回包含所有元素属性键的列表
  * set(key, value)：设置新的属性键与值

针对后代的操作
  * append(subelement)：添加直系子元素
  * extend(subelements)：增加一串元素对象作为子元素
  * find(match)：寻找第一个匹配子元素，匹配对象可以为tag或path
  * findall(match)：寻找所有匹配子元素，匹配对象可以为tag或path
  * findtext(match)：寻找第一个匹配子元素，返回其text值。匹配对象可以为tag或path
  * insert(index, element)：在指定位置插入子元素
  * iter(tag=None)：生成遍历当前元素所有后代或者给定tag的后代的迭代器
  * iterfind(match)：根据tag或path查找所有的后代
  * itertext()：遍历所有后代并返回text值
  * remove(subelement)：删除子元素

**ElementTree对象**
  * find(match)
  * findall(match)
  * findtext(match, default=None)
  * getroot()：获取根节点.
  * iter(tag=None)
  * iterfind(match)
  * parse(source, parser=None)：装载xml对象，source可以为文件名或文件类型对象.
  * write(file, encoding="us-ascii", xml_declaration=None, default_namespace=None,method="xml")　

