title: python学习笔记66 

#  Python爬虫技术(JavaScript渲染)动态页面抓取 
当我们进行网页爬虫时，我们会利用一定的规则从返回的 HTML 数据中提取出有效的信息。**但是如果网页中含有 JavaScript 代码，我们必须经过渲染处理才能获得原始数据。此时，如果我们仍采用常规方法从中抓取数据，那么我们将一无所获。**浏览器知道如何处理这些代码并将其展现出来，但是我们的程序该如何处理这些代码呢？接下来，我将介绍一个简单粗暴的方法来抓取含有 JavaScript 代码的网页信息。
大多数人利用 ` lxml 和 BeautifulSoup ` 这两个包来提取数据。本文中我将不会介绍任何爬虫框架的内容，因为我只利用最基础的 lxml 包来处理数据。本文中将介绍一个非常有趣的案例——之前我突然发现我的文章出现在最近的 Pycoders weekly issue 147中，因此我想爬取 Pycoders weekly 中所有档案的链接。
很明显，**这是一个含有 JavaScript 渲染的网页。我想要抓取网页中所有的档案信息和相应的链接信息。那么我该怎么处理呢？首先，我们利用 HTTP 方法无法获得任何信息。**
```

import requests
from lxml import html
# storing response
response = requests.get('http://pycoders.com/archive')
# creating lxml tree from response body
tree = html.fromstring(response.text)
# Finding all anchor tags in response
print tree.xpath('//div[@class="campaign"]/a/@href')

```
**当我们运行上述代码时，我们无法获得任何信息。**这怎么可能呢？网页中明明显示那么多档案的信息。接下来我们需要考虑如何解决这个问题？

**如何获取内容信息？**
接下来我将介绍**如何利用 Webkit 从 JS 渲染网页中获取数据**。什么是 Web kit呢？Web kit 可以实现浏览器所能处理的任何事情。对于某些浏览器来说，Web kit就是其底层的网页渲染工具。Web kit 是 QT 库的一部分，因此如果你已经安装 QT 和 PyQT4 库，那么你可以直接运行之。
你可以利用命令行来安装该软件库：
sudoapt-get install python-qt4
现在所有的准备工作已经完成，接下来我们将使用一个全新的方法来提取信息。
解决方案
**我们首先通过 Webkit 发送请求信息，然后等待网页被完全加载后将其赋值到某个变量中。接下来我们利用 lxml 从 HTML 数据中提取出有效的信息。这个过程需要一点时间，不过你会惊奇地发现整个网页被完整地加载下来了。**
```

import sys
from PyQt4.QtGui import *
from PyQt4.Qtcore import *
from PyQt4.QtWebKit import *
class Render(QWebPage):
def __init__(self, url):
self.app = QApplication(sys.argv)
QWebPage.__init__(self)
self.loadFinished.connect(self._loadFinished)
self.mainFrame().load(QUrl(url))
self.app.exec_()
def _loadFinished(self, result):
self.frame = self.mainFrame()
self.app.quit()

```
类 Render 可以用来渲染网页，当我们新建一个 Render 类时，它可以将 url 中的所有信息加载下来并存到一个新的框架中。
```

url = 'http://pycoders.com/archive/'
# This does the magic.Loads everything
r = Render(url)
# Result is a QString.
result = r.frame.toHtml()

```
利用以上的代码我们将 HTML 结果储存到变量 result 中，由于 lxml 无法直接处理该特殊的字符串数据，因此我们需要转换数据格式。
```

# QString should be converted to string before processed by lxml
formatted_result = str(result.toAscii())
# Next build lxml tree from formatted_result
tree = html.fromstring(formatted_result)
# Now using correct Xpath we are fetching URL of archives
archive_links = tree.xpath('//div[@class="campaign"]/a/@href')
print archive_links

```
利用上述代码我们可以获得所有的档案链接信息，接下来我们可以利用这些 Render 和这些URL链接来提取文本内容信息。Web kit 提供了一个强大的网页渲染工具，我们可以利用这个工具从 JS 渲染的网页中抓取出有效的信息。
总结
本文中我介绍了一个如何从 JS 渲染的网页中抓取信息的有效方法，这个工具虽然速度比较慢，但是却非常简单粗暴。我希望你会喜欢这篇文章。现在你可以将该方法运用到任何你觉得难以处理的网页中。
参考:http://python.jobbole.com/84600/?utm_source=tuicool&utm_medium=referral
QWebPage进一步使用：http://www.cnblogs.com/misoag/archive/2013/01/09/2853515.html