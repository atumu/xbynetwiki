title: python学习笔记27 

#  Python学习笔记之HTML处理 
##  只带的HTMLParser模块 
Python提供了` HTMLParser `来非常方便地解析HTML.除此之外还有强大的Beautiful Soup等三方库
如果我们要编写一个搜索引擎，第一步是用爬虫把目标网站的页面抓下来，第二步就是解析该HTML页面，看看里面的内容到底是新闻、图片还是视频。
假设第一步已经完成了，第二步应该如何解析HTML呢？
HTML本质上是XML的子集，但是HTML的语法没有XML那么严格，所以不能用标准的DOM或SAX来解析HTML。
好在Python提供了` HTMLParser `来非常方便地解析HTML，只需简单几行代码：
```

from html.parser import HTMLParser
from html.entities import name2codepoint
class MyHTMLParser(HTMLParser):
    def handle_starttag(self, tag, attrs):
        print('<%s>' % tag)
    def handle_endtag(self, tag):
        print('</%s>' % tag)
    def handle_startendtag(self, tag, attrs):
        print('<%s/>' % tag)
    def handle_data(self, data):
        print(data)
    def handle_comment(self, data):
        print('<!--', data, '-->')
    def handle_entityref(self, name):
        print('&%s;' % name)
    def handle_charref(self, name):
        print('&#%s;' % name)
parser = MyHTMLParser()
parser.feed('''<html>
<head></head>
<body>
<!-- test html parser -->
    <p>Some <a href=\"#\">html</a> HTML&nbsp;tutorial...<br>END</p>
</body></html>''')

```
**feed()方法可以多次调用**，也就是不一定一次把整个HTML字符串都塞进去，可以一部分一部分塞进去。
特殊字符有两种，一种是英文表示的&nbsp;，一种是数字表示的&#1234;，这两种字符都可以通过Parser解析出来。
小结
利用HTMLParser，可以把网页中的文本、图像等解析出来。

HTMLParser采用的是一种**事件驱动的模式**，当HTMLParser找到一个特定的标记时，它会去调用一个用户定义的函数，以此来通知程序处理。它主要的用户回调函数的命名都是以handler_开头的，都是HTMLParser的成员函数。当我们使用时，就从HTMLParser派生出新的类，然后重新定义这几个以handler_开头的函数即可。这几个函数包括：
  * handle_startendtag  处理开始标签和结束标签
  * handle_starttag     处理开始标签，比如<xx>   tag不区分大小写
  * handle_endtag       处理结束标签，比如</xx>
  * handle_charref      处理特殊字符串，就是以&#开头的，一般是内码表示的字符
  * handle_entityref    处理一些特殊字符，以&开头的，比如 &nbsp;
  * handle_data         处理数据，就是<xx>data</xx>中间的那些数据
  * handle_comment      处理注释
  * handle_decl         处理<!开头的，比如<!DOCTYPE html PUBLIC “-/ /W3C/ /DTD HTML 4.01 Transitional/ /EN”
  * handle_pi           处理形如<?instruction>的东西
```

def handle_starttag(self,tag,attr):
        #注意：tag不区分大小写，此时也可以解析 <A 标签
        # HTMLParser 会在创建attrs 时将属性名转化为小写。
        if tag=='a':
            for href,link in attr:
                if href.lower()=="href":
                        pass


```
**tag是的html标签，attrs是 (属性，值)元组(tuple)的列表(list). 
HTMLParser自动将tag和attrs都转为小写。**

###  解析html的超链接和链接显示的内容 
```

from html.parser import HTMLParser
class myHtmlParser(HTMLParser):  
  
    def __init__(self):  
        HTMLParser.__init__(self)  
        self.flag=None  
  
    # 这里重新定义了处理开始标签的函数  
    def handle_starttag(self,tag,attrs):  
         # 判断标签<a>的属性  
        if tag=='a' and len(attrs) != 0:  
            self.flag='a'  
            for href,link in attrs:  
                if href=='href':  
                    print "href:",link  
  
    def handle_data(self,data):  
        if self.flag=='a':  
            print "data:",data.decode('utf-8')  
  
if __name__ == '__main__':  
    a = '<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">\  
    <html><head><!--insert javaScript here!--><title>test</title><body><a href="http: //www.163.com">链接到163</a></body></html>'  
    m=myHtmlParser()  
    m.feed(a)  
    m.close()  
  
输出结果：  
  
href: http: //www.163.com  
data: 链接到163</span>  

```
##  强大的Beautiful Soup库 
文档:http://beautifulsoup.readthedocs.org/zh_CN/latest/#
Beautiful Soup 是一个可以从HTML或XML文件中提取数据的Python库.它能够通过你喜欢的转换器实现惯用的文档导航,查找,修改文档的方式.Beautiful Soup会帮你节省数小时甚至数天的工作时间.
###  安装: 
**1、安装库**
```

pip install beautifulsoup4

```
**2、安装解析器(可选)**
安装解析器
Beautiful Soup支持Python标准库中的HTML解析器,还支持一些第三方的解析器,其中一个是 lxml .根据操作系统不同,可以选择下列方法来安装lxml:
$ pip install lxml
另一个可供选择的解析器是纯Python实现的 html5lib , **html5lib的解析方式与浏览器相同**,可以选择下列方法来安装html5lib:
$ pip install html5lib
下表列出了主要的解析器,以及它们的优缺点:
![](/data/dokuwiki/python/pasted/20160313-180807.png)
推荐使用lxml作为解析器,因为效率更高. 在Python2.7.3之前的版本和Python3中3.2.2之前的版本,必须安装lxml或html5lib, 因为那些Python版本的标准库中内置的HTML解析方法不够稳定.
###  快速入门 
```

html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

```
使用BeautifulSoup解析这段代码,能够得到一个 BeautifulSoup 的对象,并能按照标准的缩进格式的结构输出:
```

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc)
print(soup.prettify())

```
几个简单的浏览结构化数据的方法:
```

soup.title
# <title>The Dormouse's story</title>
soup.title.name
# u'title'
soup.title.string
# u'The Dormouse's story'
soup.title.parent.name
# u'head'
soup.p
# <p class="title"><b>The Dormouse's story</b></p>
soup.p['class']
# u'title'
soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find(id="link3")
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

```
**从文档中找到所有<a >标签的链接:**
```

for link in soup.find_all('a'):
    print(link.get('href'))
    # http://example.com/elsie
    # http://example.com/lacie
    # http://example.com/tillie

```
从文档中获取所有文字内容:
```

print(soup.get_text())
# The Dormouse's story
#
# The Dormouse's story
#
# Once upon a time there were three little sisters; and their names were
# Elsie,
# Lacie and
# Tillie;
# and they lived at the bottom of a well.
#
# ...

```
###  如何使用 
将一段文档传入BeautifulSoup 的构造方法,就能得到一个文档的对象, 可以传入一段字符串或一个文件句柄.
```

from bs4 import BeautifulSoup
soup = BeautifulSoup(open("index.html"))
soup = BeautifulSoup("<html>data</html>")

```
首先,文档被转换成Unicode,并且HTML的实例都被转换成Unicode编码
然后,Beautiful Soup选择最合适的解析器来解析这段文档,如果手动指定解析器那么Beautiful Soup会选择指定的解析器来解析文档.
###  对象的种类 
Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,**所有对象可以归纳为4种: Tag , NavigableString , BeautifulSoup , Comment .**
####  Tag 
Tag 对象与XML或HTML原生文档中的tag相同:
```

soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>

```
Tag有很多方法和属性,在 遍历文档树 和 搜索文档树 中有详细解释.现在介绍一下**tag中最重要的属性: name和attributes**
Name每个tag都有自己的名字,通过 .name 来获取:
```

tag.name
# u'b'

```
如果改变了tag的name,那将影响所有通过当前Beautiful Soup对象生成的HTML文档:
####  Attributes 
一个tag可能有很多个属性. tag <b class="boldest" > 有一个 “class” 的属性,值为 “boldest” . tag的属性的操作方法与字典相同:
```

tag['class']
# u'boldest'
也可以直接”点”取属性, 比如: .attrs :
tag.attrs
# {u'class': u'boldest'}

```
tag的属性可以被添加,删除或修改. 再说一次,** tag的属性操作方法与字典一样**
```

tag['class'] = 'verybold'
tag['id'] = 1
tag
# <blockquote class="verybold" id="1">Extremely bold</blockquote>
del tag['class']
del tag['id']
tag
# <blockquote>Extremely bold</blockquote>
tag['class']
# KeyError: 'class'
print(tag.get('class'))
# None

```
###  多值属性 
HTML 4定义了一系列可以包含多个值的属性.在HTML5中移除了一些,却增加更多.最常见的多值的属性是 class (一个tag可以有多个CSS的class). 还有一些属性 rel , rev , accept-charset , headers , accesskey .
**在Beautiful Soup中多值属性的返回类型是list**:
```

css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body", "strikeout"]
css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ["body"]

```
如果某个属性看起来好像有多个值,但在任何版本的HTML定义中都没有被定义为多值属性,那么Beautiful Soup会将这个属性作为字符串返回
id_soup = BeautifulSoup('<p id="my id"></p>')
id_soup.p['id']
# 'my id'
将tag转换成字符串时,多值属性会合并为一个值
```

rel_soup = BeautifulSoup('<p>Back to the <a rel="index">homepage</a></p>')
rel_soup.a['rel']
# ['index']
rel_soup.a['rel'] = ['index', 'contents']
print(rel_soup.p)
# <p>Back to the <a rel="index contents">homepage</a></p>

```
如果转换的文档是XML格式,那么tag中不包含多值属性
###  可以遍历的字符串 
字符串常被包含在tag内.Beautiful Soup用 **NavigableString 类来包装tag中的字符串**:
```

tag.string
# u'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>

```
一个 NavigableString 字符串与Python中的Unicode字符串相同,并且还支持包含在 遍历文档树 和 搜索文档树 中的一些特性. **通过 unicode() 方法可以直接将 NavigableString 对象转换成Unicode字符串:**
```

unicode_string = unicode(tag.string)
unicode_string
# u'Extremely bold'
type(unicode_string)
# <type 'unicode'>

```
tag中包含的字符串不能编辑,但是可以被替换成其它的字符串,用 replace_with() 方法:
```

tag.string.replace_with("No longer bold")
tag
# <blockquote>No longer bold</blockquote>

```
NavigableString 对象支持 遍历文档树 和 搜索文档树 中定义的大部分属性, 并非全部.尤其是,一个字符串不能包含其它内容(tag能够包含字符串或是其它tag),字符串不支持 .contents 或 .string 属性或 find() 方法.
如果想在Beautiful Soup之外使用 NavigableString 对象,需要调用 unicode() 方法,将该对象转换成普通的Unicode字符串,否则就算Beautiful Soup已方法已经执行结束,该对象的输出也会带有对象的引用地址.这样会浪费内存.
###  BeautifulSoup 
**BeautifulSoup 对象表示的是一个文档的全部内容.**大部分时候,可以把它当作 Tag 对象,它支持 遍历文档树 和 搜索文档树 中描述的大部分的方法.
因为 BeautifulSoup 对象并不是真正的HTML或XML的tag,所以它没有name和attribute属性.但有时查看它的 .name 属性是很方便的,所以 **BeautifulSoup 对象包含了一个值为 “[document]” 的特殊属性** .name
soup.name
# u'[document]'

###  注释及特殊字符串 
Tag , NavigableString , BeautifulSoup 几乎覆盖了html和xml中的所有内容,但是还有一些特殊对象.容易让人担心的内容是文档的注释部分:
```

markup = "<b><!--Hey, buddy. Want to buy a used parser?--></b>"
soup = BeautifulSoup(markup)
comment = soup.b.string
type(comment)
# <class 'bs4.element.Comment'>

```
Comment 对象是一个特殊类型的 NavigableString 对象:
comment
# u'Hey, buddy. Want to buy a used parser'
但是当它出现在HTML文档中时, Comment 对象会使用特殊的格式输出:
print(soup.b.prettify())
# <b>
#  <!--Hey, buddy. Want to buy a used parser?-->
# </b>
Beautiful Soup中定义的其它类型都可能会出现在XML的文档中: CData , ProcessingInstruction , Declaration , Doctype .与 Comment 对象类似,这些类都是 NavigableString 的子类,只是添加了一些额外的方法的字符串独享.下面是用CDATA来替代注释的例子:
```

from bs4 import CData
cdata = CData("A CDATA block")
comment.replace_with(cdata)
print(soup.b.prettify())
# <b>
#  <![CDATA[A CDATA block]]>
# </b>

```
##  find及find_all查找 
###  find_all 
**find_all( name , attrs , recursive , text , * *kwargs )**
find_all() 方法搜索当前tag的所有tag子节点,并判断是否符合过滤器的条件.
```

soup.find_all("p",id="link2")
# [<p class="sister" href="http://example.com/lacie" id="link2">Lacie</p>]

```
###  find_all参数解析 
  * name 参数(仅有一个)：可以查找所有名字为 name 的tag,字符串对象会被自动忽略掉.**还支持正则表达式**如soup.find_all(re.compile("^b")):找出所有以b开头的标签,这表示<body>和<b>标签都应该被找到
**也支持列表**如：soup.find_all(["a", "b"])找到文档中所有<a>标签和<b>标签
  * keyword 参数(可以有多个键值对，也可以用attrs={"data-foo": "value"}来代替键值对):如果一个指定名字的参数不是搜索内置的参数名,搜索时会把该参数当作指定名字tag的属性来搜索,如果包含一个名字为 id 的参数,Beautiful Soup会搜索每个tag的”id”属性.
  * 按CSS搜索：如soup.find_all("a", class_="sister")，其中class_带下划线
  * text 参数可以搜搜文档中的字符串内容。text 参数接受 字符串 , 正则表达式 , 列表, True . 例如soup.find_all(text=re.compile("Dormouse"))
  * limit 参数：限制返回结果的数量
  * recursive 参数：调用tag的 find_all() 方法时,Beautiful Soup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 recursive=False .
###  像调用 find_all() 一样调用tag 
find_all() 几乎是Beautiful Soup中最常用的搜索方法,**所以我们定义了它的简写方法**. 
**BeautifulSoup 对象和 tag 对象可以被当作一个方法来使用,这个方法的执行结果与调用这个对象的 find_all() 方法相同,下面两行代码是等价的:**
```

soup.find_all("a")
soup("a")
这两行代码也是等价的:
soup.title.find_all(text=True)
soup.title(text=True)

```
###  find/find_parents() 和 find_parent()/find_next_siblings() 合 find_next_sibling()/find_previous_siblings() 和 find_previous_sibling()/find_all_next() 和 find_next()/find_all_previous() 和 find_previous() 
**find( name , attrs , recursive , text , * *kwargs )**
find_all() 方法将返回文档中符合条件的所有tag,尽管有时候我们只想得到一个结果.比如文档中只有一个<body>标签,那么使用 find_all() 方法来查找<body>标签就不太合适, 使用 find_all 方法并设置 limit=1 参数不如直接使用 find() 方法.下面两行代码是等价的:
```

soup.find_all('title', limit=1)
# [<title>The Dormouse's story</title>]

soup.find('title')
# <title>The Dormouse's story</title>

```
唯一的区别是 find_all() 方法的返回结果是值包含一个元素的列表,而 find() 方法直接返回结果.
```

soup.find("p", class="title")
#<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;

```

##  CSS选择器 
Beautiful Soup支持大部分的CSS选择器 [6] ,在 Tag 或 BeautifulSoup 对象的** .select() 方法**中传入字符串参数,即可使用CSS选择器的语法找到tag:
```

soup.select(".sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
soup.select("[class~=sister]")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

其余请参考：http://beautifulsoup.readthedocs.org/zh_CN/latest/#id5
http://python.jobbole.com/84774/