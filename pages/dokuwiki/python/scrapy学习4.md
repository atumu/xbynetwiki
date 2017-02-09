title: scrapy学习4 

#  Scrapy+Selenium+Phantomjs Demo 
http://www.tuicool.com/articles/z63yYn2
使用Scrapy框架爬取京东的商品信息。商品详情页的价格是由js生成的，而通过Scrapy直接爬取的源文件中无价格信息。
通过Selenium、Phantomjs便能实现。

Scrapy使用Selenium
需要弄懂一个概念， 下载中间件 ，可以阅读下官方文档 http://scrapy-chs.readthedocs.io/zh_CN/1.0/topics/downloader-middleware.html
在settings.py加入
```

DOWNLOADER_MIDDLEWARES = {
'jdSpider.middlewares.middleware.JavaScriptMiddleware':543,#键为中间件类的路径，值为中间件的顺序
'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware':None,#禁止内置的中间件
}

```
```

from selenium import webdriver
from scrapy.http import HtmlResponse
import time

classJavaScriptMiddleware(object):
	def process_request(self, request, spider):
		if spider.name =="jd":
			print"PhantomJS is starting..."
 			driver = webdriver.PhantomJS() #指定使用的浏览器
			# driver = webdriver.Firefox()
 			driver.get(request.url)
 			time.sleep(1)
 			js = "var q=document.documentElement.scrollTop=10000"
 			driver.execute_script(js) #可执行js，模仿用户操作。此处为将页面拉至最底端。
 			time.sleep(3)
 			body = driver.page_source
			print("访问"+request.url)
			return HtmlResponse(driver.current_url, body=body, encoding='utf-8', request=request)
		else:
			return

```
在spiders/jd.py中parse()方法接收到的response则是我们自定义中间件返回的结果。我们得到的便是js生成后的界面。
```

# -*- coding: utf-8 -*-
importscrapy
class JdSpider(scrapy.Spider):
 name = "jd"
 allowed_domains = ["jd.com"]
 start_urls = (
'http://search.jd.com/Search?keyword=三星s7&enc=utf-8&wq=三星s7&pvid=tj0sfuri.v70avo',
 )

def parse(self, response):
	print response.body

```