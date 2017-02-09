title: scrapy学习1 

#  Scrapy学习1之入门 
官网:https://scrapy.org/
文档：https://doc.scrapy.org
Demo：https://github.com/scrapy/quotesbot
当前版本:1.2

 pip install scrapy

Architecture overview：
![](/data/dokuwiki/python/pasted/20161104-005856.png)

组件：
Scrapy Engine：用于控制数据在所有组件之间的流转及触发对应的事件动作。
Scheduler：接收requests并将其加入队列
Downloader：下载获取页面。
Spiders：程序开发者的自定义类。定义入口页面，解析响应，提取item或dict，继续提交requests
Item Pipeline：处理item，持久化。
Downloader middlewares：位于Engine与Downloader之间的钩子，在请求由engine到downloader之时，及响应由downloader到engine之时都会进行一系列操作。
  * process a request just before it is sent to the Downloader (i.e. right before Scrapy sends the request to the website);
  * change received response before passing it to a spider;
  * send a new Request instead of passing received response to a spider;
  * pass response to a spider without fetching a web page;
  * silently drop some requests.

Spider middlewares：位于Engine与Spiders之间的钩子，可以处理 spider input (responses) and output (items and requests).
  * post-process output of spider callbacks - change/add/remove requests or items;
  * post-process start_requests;
  * handle spider exceptions;
  * call errback instead of callback for some of the requests based on response content.

Event-driven networking
Scrapy is written with Twisted, a popular event-driven networking framework for Python. Thus, it’s implemented using a non-blocking (aka asynchronous) code for concurrency.


##  快速介绍 
```

import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/tag/humor/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.xpath('span/small/text()').extract_first(),
            }

        next_page = response.css('li.next a::attr("href")').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)

```
运行:
```

scrapy runspider quotes_spider.py -o quotes.json

```
注意几点：
1、调度与执行是异步的。
2、css与xpath选择器


**Scrapy特点**
1、数据抓取支持扩展的css、xpath选择器，正则表达式支持
2、 [interactive shell console](https://doc.scrapy.org/en/1.2/topics/shell.html#topics-shell) 程序来调试css、xpath选择器
3、内建的多格式导出支持：json,xml,csv.及FTP, S3, local filesystem存储支持
4、自动编码检测
5、强大的可扩展性，使用信号，middlewares, extensions, and pipelines
6、很多内置的 extensions and middlewares来处理：
  * cookies and session handling
  * HTTP features like compression, authentication, caching
  * user-agent spoofing
  * robots.txt
  * crawl depth restriction
  * and more
7、[Telnet console](https://doc.scrapy.org/en/1.2/topics/telnetconsole.html#topics-telnetconsole)用于连接运行中的scrapy进程。便于调试和监控。

**安装**
```

pip install Scrapy

```
ubuntu下安装：sudo apt-get install python3 python3-dev python3-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev
依赖：
  * lxml, an efficient XML and HTML parser
  * parsel, an HTML/XML data extraction library written on top of lxml,
  * w3lib, a multi-purpose helper for dealing with URLs and web page encodings
  * twisted, an asynchronous networking framework
  * cryptography and pyOpenSSL, to deal with various network-level security needs

` 提示：w3lib检测网页编码方法：w3lib.encoding.html_body_declared_encoding(html_body_str) `

##  创建Scrapy工程骨架 
```

scrapy startproject tutorial

```
创建的目录结构如下
```

tutorial/
    scrapy.cfg            # 部署配置文件
    tutorial/             # project's Python module, you'll import your code from here
        __init__.py
        items.py          # project items definition file
        pipelines.py      # project pipelines file
        settings.py       # project settings file
        spiders/          # a directory where you'll later put your spiders
            __init__.py

```
创建第一个爬虫，自定义的爬虫类必须是**scrapy.Spider**的子类
```

import scrapy
class QuotesSpider(scrapy.Spider):
    name = "quotes"  #爬虫的名字，在这个工程中必须是唯一的。
    #必须返回一个Scrapy.Request对象的可迭代列表(可以是一个列表，或者是一个生成器).这是爬虫请求的入口。
    def start_requests(self): 
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)
      #处理爬虫下载的一页数据，参数response是TextResponse的实例，包含下载的一页数据。
      #该方法工作如下：提取页面数据到一个字典中，获取新的url来创建新的Request继续抓取数据。
    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log('Saved file %s' % filename)

```
运行爬虫：定位到项目的根目录执行：
```

scrapy crawl quotes

```
简化写法：以start_urls 属性代替start_requests方法：
```

import scrapy
class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]
    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)

```
###  选择器简介
使用scrapy shell测试选择器
**css选择器与re正则过滤**
```

scrapy shell "http://quotes.toscrape.com/page/1/"
>>> response.css('title')  #返回的是SelectorList对象，它是一个list-like对象。包含一个Selector的列表。Selector包含 XML/HTML元素。
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]
>>> response.css('title::text').extract() #::text标识获取直接文本。.extract()获取到的是一个列表，如果需要获取单个值，可以使用.extract_first()
['Quotes to Scrape']
>>> response.css('title::text').extract_first()
'Quotes to Scrape'
>>> quote = response.css("div.quote")[0]

#当然你也可以使用.re()方法来使用正则表达式过滤
>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']

```
**xpath选择器**
```

>>> response.xpath('//title')
[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
>>> response.xpath('//title/text()').extract_first()
'Quotes to Scrape'

```

** 稍微复杂点的例子：**
```

import scrapy


class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        # follow links to author pages
        for href in response.css('.author+a::attr(href)').extract():
            yield scrapy.Request(response.urljoin(href),
                                 callback=self.parse_author)

        # follow pagination links
        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).extract_first().strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }


```
###  使用Spider参数 
通过-a 选项可以向quotes爬虫的构造器传入一个参数，然后爬虫中就可以通过，比如参数名为tag，则可以这样使用tag = getattr(self, 'tag', None)
```

scrapy crawl quotes -o quotes-humor.json -a tag=humor

```
##  命令行工具 
https://doc.scrapy.org/en/1.2/topics/commands.html
配置文件scrapy.cfg
主要配置：
  * SCRAPY_SETTINGS_MODULE
  * SCRAPY_PROJECT
  * SCRAPY_PYTHON_SHELL
命令：
scrapy
scrapyd-deploy
Scrapy commands查看子命令

创建项目：scrapy startproject myproject [project_dir]

子命令列表
Global commands:
  * **startproject**
  * genspider
  * settings
  * **runspider**
  * **shell**
  * fetch
  * view
  * version
Project-only commands:
  * **crawl**
  * check
  * list
  * edit
  * parse
  * bench
##  Spiders 
Spider是你自定义抓取链接规则和提取数据的地方。
在Spider中记录日志：
```

self.logger.info('A response from %s just arrived!', response.url)

```
主要的一些Spider类
  * scrapy.Spider：它是类scrapy.spiders.Spider。是所有Spider的基类。
  * scrapy.spiders.CrawlSpider：它是一个通用的爬虫类。
  * scrapy.spiders.XMLFeedSpider
  * scrapy.spiders.CSVFeedSpider
  * scrapy.spiders.SitemapSpider
###  scrapy.spiders.Spider 
主要属性：
  * name 爬虫名字，必须唯一
  * allowed_domains：一个域名的列表，(默认包含子域名)。如果OffsiteMiddleware被启用的话，链接域名不在列表中的会被禁止抓取。
  * start_urls：入口url列表
  * custom_settings：字典
  * crawler：scrapy.crawler.Crawler实例
  * settings：scrapy.settings.Settings实例
  * logger：以爬虫name属性命名的日志实例

类方法：
  * from_crawler(crawler, *args, * *kwargs)类方法，Scrapy会调用该类方法来创建Spider

实例方法
  * start_requests()
  * make_requests_from_url(url)
  * parse(response)
  * log(message[, level, component])
  * closed(reason)功能和spider_closed信号一样，在spider关闭时调用。

```

import scrapy

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = [
        'http://www.example.com/1.html',
        'http://www.example.com/2.html',
        'http://www.example.com/3.html',
    ]

    def parse(self, response):
      	  self.logger.info('A response from %s just arrived!', response.url)
        for h3 in response.xpath('//h3').extract():
            yield {"title": h3}

        for url in response.xpath('//a/@href').extract():
            yield scrapy.Request(url, callback=self.parse)

```

###  scrapy.spiders.CrawlSpider 
` 注意：使用该类时。不要覆盖parse方法 `
属性：
rules：scrapy.spiders.Rule对象的列表
实例方法：
parse_start_url(response):This method is called for the start_urls responses.允许解析初始的repsonse,必须返回scrapy.item.Item([arg])对象或scrapy.http.Request或者他们的列表。

class scrapy.spiders.**Rule**(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)说明：
参数说明：
  * link_extractor:scrapy.linkextractors.LinkExtractor对象。定义链接提取规则。
  * callback： 可以是 callable or a string。这些回调方法在链接内容获取到之后调用。回调方法接受response参数，必须返回Item或Request的列表。
  * cb_kwargs：会被传递到回调函数的参数。
  * follow：是否继续提取新链接而不进行当前链接回调。默认为False，如果callback=None,则为True
  * process_links： 可以是 callable, or a string。主要用于再次过滤链接
  * process_request： 可以是callable, or a string。必须返回 a request or None 主要用于过滤Request
```

import scrapy
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor

class MySpider(CrawlSpider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = ['http://www.example.com']

    rules = (
        # Extract links matching 'category.php' (but not matching 'subsection.php')
        # and follow links from them (since no callback means follow=True by default).
        Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

        # Extract links matching 'item.php' and parse them with the spider's method parse_item
        Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
    )

    def parse_item(self, response):
        self.logger.info('Hi, this is an item page! %s', response.url)
        item = scrapy.Item()
        item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
        item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
        item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
        return item

```

##  选择器 
支持扩展的CSS选择器，XPtah语法，正则表达式语法
选择器是scrapy.selector.Selector类
```

>>> from scrapy.selector import Selector
>>> from scrapy.http import HtmlResponse
>>> body = '<html><body><span>good</span></body></html>'
>>> Selector(text=body).xpath('//span/text()').extract()
[u'good']
>>> response = HtmlResponse(url='http://example.com', body=body)
>>> Selector(response=response).xpath('//span/text()').extract()
[u'good']


#response也有个selector属性
>>> response.selector.xpath('//span/text()').extract()
[u'good']
#repsonse也直接内置了xpath()和css()方法。
response.xpath('//span/text()').extract()

```
测试：
```

scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html
>>> response.selector.xpath('//title/text()')
[<Selector (text) xpath=//title/text()>]
>>> response.xpath('//title/text()')
>>> response.css('title::text')
>>> response.css('img').xpath('@src').extract()
[u'image1_thumb.jpg']

```
```

>>> response.xpath('//base/@href').extract()
[u'http://example.com/']
>>> response.css('base::attr(href)').extract()
>>> response.xpath('//a[contains(@href, "image")]/@href').extract()
>>> response.css('a[href*=image]::attr(href)').extract()
>>> response.css('a[href*=image] img::attr(src)').extract()

```
使用正则表达式：re(),re_first()
```

>>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
>>> response.xpath('//a[contains(@href, "image")]/text()').re_first(r'Name:\s*(.*)')

```
##  Items 
scrapy.item.Item
抓取数据保存的数据单元,你可以用python dict,但是scrapy提供Item接口。它是一个dict-like对象。
###  自定义Item 
```

import scrapy
class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    stock = scrapy.Field()
    last_updated = scrapy.Field(serializer=str)

```
获取field的值
```

>>> product = Product(name='Desktop PC', price=1000)
>>> print product
Product(name='Desktop PC', price=1000)
>>> product['name']
Desktop PC
>>> product.get('name')
Desktop PC
>>> product['price']
1000
>>> product['last_updated']
Traceback (most recent call last):
    ...
KeyError: 'last_updated'

>>> product.get('last_updated', 'not set')
not set
>>> product['lala'] # getting unknown field
Traceback (most recent call last):
    ...
KeyError: 'lala'

>>> product.get('lala', 'unknown field')
'unknown field'
>>> 'name' in product  # is name field populated?
True
>>> 'last_updated' in product  # is last_updated populated?
False
>>> 'last_updated' in product.fields  # is last_updated a declared field?
True
>>> 'lala' in product.fields  # is lala a declared field?
False

```
设置field的值
```

>>> product['last_updated'] = 'today'
>>> product['last_updated']
today

>>> product['lala'] = 'test' # setting unknown field
Traceback (most recent call last):
    ...
KeyError: 'Product does not support field: lala'

```
访问所有被填充的值
```

>>> product.keys()
['price', 'name']

>>> product.items()
[('price', 1000), ('name', 'Desktop PC')]

```
复制Item
```

>>> product2 = Product(product)
>>> print product2
Product(name='Desktop PC', price=1000)

>>> product3 = product2.copy()
>>> print product3
Product(name='Desktop PC', price=1000)

```
将Item转为dict
```

>>> dict(product) # create a dict from all populated values
{'price': 1000, 'name': 'Desktop PC'}

```
从dict创建item
```

>>> Product({'name': 'Laptop PC', 'price': 1500})
Product(price=1500, name='Laptop PC')

>>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
Traceback (most recent call last):
    ...
KeyError: 'Product does not support field: lala'

```
###  Item填充器(Item Loaders) 
ItemLoader.default_item_class属性？？？
```

from scrapy.loader import ItemLoader
from myproject.items import Product

def parse(self, response):
    l = ItemLoader(item=Product(), response=response)
    l.add_xpath('name', '//div[@class="product_name"]')
    l.add_xpath('name', '//div[@class="product_title"]')
    l.add_xpath('price', '//p[@id="price"]')
    l.add_css('stock', 'p#stock]')
    l.add_value('last_updated', 'today') # you can also use literal values
    return l.load_item()

```
自定义ItemLoader
```

from scrapy.loader import ItemLoader
from scrapy.loader.processors import TakeFirst, MapCompose, Join

class ProductLoader(ItemLoader):

    default_output_processor = TakeFirst()

    name_in = MapCompose(unicode.title)
    name_out = Join()

    price_in = MapCompose(unicode.strip)
    #。。。

```
```

import scrapy
from scrapy.loader.processors import Join, MapCompose, TakeFirst
from w3lib.html import remove_tags
def filter_price(value):
    if value.isdigit():
        return value

class Product(scrapy.Item):
    name = scrapy.Field(
        input_processor=MapCompose(remove_tags),
        output_processor=Join(),
    )
    price = scrapy.Field(
        input_processor=MapCompose(remove_tags, filter_price),
        output_processor=TakeFirst(),
    )

```
注意几点：
  * **Item Loader** field-specific attributes: **field_in** and **field_out** (most precedence)
  * **Field** metadata (**input_processor** and **output_processor** key)
  * **Item Loader** defaults: ItemLoader.**default_input_processor**() and ItemLoader.**default_output_processor**() (least precedence)

Item Loader Context：略。

###  内置ItemLoader处理器processors 
scrapy.loader.processors.**Identity**:原封不动返回值。
scrapy.loader.processors.**TakeFirst**：返回第一个值
scrapy.loader.processors.**Join**(separator=u' ')：合并多个值。
scrapy.loader.processors.**Compose**(*functions, * *default_loader_context):会链式调用所有参数函数，直到所有函数调用完毕返回值。
```

>>> from scrapy.loader.processors import Compose
>>> proc = Compose(lambda v: v[0], str.upper)
>>> proc(['hello', 'world'])
'HELLO'

```
scrapy.loader.processors.**MapCompose**(*functions, * *default_loader_context)：与Compose的区别是，它像相当于map函数的行为
```

>>> def filter_world(x):
...     return None if x == 'world' else x
...
>>> from scrapy.loader.processors import MapCompose
>>> proc = MapCompose(filter_world, unicode.upper)
>>> proc([u'hello', u'world', u'this', u'is', u'scrapy'])
[u'HELLO, u'THIS', u'IS', u'SCRAPY']

```
 scrapy.loader.processors.SelectJmes(json_path)：略。

##  Scrapy Shell 
https://doc.scrapy.org/en/latest/topics/shell.html
##  Item Pipeline 
Item Pipeline接收Item，然后多个Pipeline进行链式处理。在这里你可以将item中的数据保存到文件，数据库等。
定制Pipeline只需要实现一个process_item(self, item, spider)方法即可。
参数：
  * item (**Item** object or a **dict**) – the item scraped
  * spider (Spider object) – the spider which scraped the item
当然还有其他几个方法可以选择覆盖：
  * process_item(self, item, spider)
  * open_spider(self, spider)
  * close_spider(self, spider)
  * from_crawler(cls, crawler)
```

from scrapy.exceptions import DropItem

class PricePipeline(object):
    vat_factor = 1.15
    def process_item(self, item, spider):
        if item['price']:
            if item['price_excludes_vat']:
                item['price'] = item['price'] * self.vat_factor
            return item
        else:
            raise DropItem("Missing price in %s" % item)

```
Write items to a JSON file
```

import json
class JsonWriterPipeline(object):
    def open_spider(self, spider):
        self.file = open('items.jl', 'wb')
    def close_spider(self, spider):
        self.file.close()
    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + "\n"
        self.file.write(line)
        return item

```
Write items to MongoDB
```

import pymongo
class MongoPipeline(object):
    collection_name = 'scrapy_items'
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
        )
    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        self.db[self.collection_name].insert(dict(item))
        return item

```
##  注册Item Pipeline 
在配置文件中添加
```

ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300,
    'myproject.pipelines.JsonWriterPipeline': 800,
}

```
优先级范围： 0-1000
##  Feed exports数据导出与存储 
https://doc.scrapy.org/en/latest/topics/feed-exports.html#std:setting-FEED_EXPORTERS
支持：
JSON
JSON lines
CSV
XML
Local filesystem
FTP
S3
Standard output

导出相关配置：
FEED_URI (mandatory)
FEED_FORMAT
FEED_STORAGES
FEED_EXPORTERS
FEED_STORE_EMPTY
FEED_EXPORT_ENCODING
FEED_EXPORT_FIELDS

##  Requests and Responses 
scrapy.http.Request(url[, callback, method='GET', headers, body, cookies, meta, encoding='utf-8', priority=0, dont_filter=False, errback])
Request子类：
  * scrapy.http.FormRequest(url[, formdata, ...])

scrapy.http.Response(url[, status=200, headers=None, body=b' ', flags=None, request=None])
Response子类：
  * scrapy.http.TextResponse(url[, encoding[, ...]])
  * scrapy.http.HtmlResponse(url[, ...])
  * scrapy.http.XmlResponse(url[, ...])
##  Link Extractors 
scrapy.linkextractors.LinkExtractor,只有一个方法extract_links(Response),返回scrapy.link.Link
scrapy.linkextractors.lxmlhtml.LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), restrict_css=(), tags=('a', 'area'), attrs=('href', ), canonicalize=True, unique=True, process_value=None)

##  配置 
  * Command line options (most precedence)  如-s选项：scrapy crawl myspider -s LOG_FILE=scrapy.log
  * Settings per-spider 如：在你的Spider类中custom_settings属性,值为字典
  * Project settings module	就是你的项目settings.py文件，你也可以使用环境变量指定SCRAPY_SETTINGS_MODULE
  * Default settings per-command 命令类的default_settings属性
  * Default global settings (less precedence) 即scrapy.settings.default_settings

**获取配置**
https://doc.scrapy.org/en/latest/topics/settings.html#topics-settings-ref
self.settings属性
```

class MySpider(scrapy.Spider):
    name = 'myspider'
    start_urls = ['http://example.com']

    def parse(self, response):
        print("Existing settings: %s" % self.settings.attributes.keys())

```
      
**主要配置选项**
BOT_NAME：默认为'scrapybot'，用于构造User-Agent和logging记录，如果使用scrapy startproject命令创建，则为你的项目名。
USER_AGENT="Scrapy/VERSION (+http://scrapy.org)"
DEFAULT_REQUEST_HEADERS：默认为：
```

{
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
}

```
ITEM_PIPELINES：默认为{}
例如：
```

ITEM_PIPELINES = {
    'mybot.pipelines.validate.ValidateMyItem': 300,
    'mybot.pipelines.validate.StoreMyItem': 800,
}

```
ITEM_PIPELINES_BASE：默认为{}

去重机制
DUPEFILTER_CLASS：默认为'scrapy.dupefilters.RFPDupeFilter'，使用scrapy.utils.request.request_fingerprint函数提供的请求指纹策略。
` 想改变这种去重策略，你可以继承RFPDupeFilter，覆盖request_fingerprint方法，该方法接收Request object作为参数，返回 its fingerprint (a string). `
DUPEFILTER_DEBUG：默认False
调度机制
SCHEDULER：默认为 'scrapy.core.scheduler.Scheduler'
SCHEDULER_DEBUG=False
DEPTH_LIMIT：默认为0，爬取深度，0为不限制。
并发
CONCURRENT_REQUESTS：默认16
CONCURRENT_REQUESTS_PER_DOMAIN：默认为8
CONCURRENT_REQUESTS_PER_IP：默认为0，如果非0，则CONCURRENT_REQUESTS_PER_DOMAIN会被忽略。
REACTOR_THREADPOOL_MAXSIZE=10
下载机制
DOWNLOADER:默认为 'scrapy.core.downloader.Downloader'类。
DOWNLOADER_HTTPCLIENTFACTORY：默认为 'scrapy.core.downloader.webclient.ScrapyHTTPClientFactory'
DOWNLOADER_CLIENTCONTEXTFACTORY默认为'scrapy.core.downloader.contextfactory.ScrapyClientContextFactory'。“ContextFactory”在 Twisted term 中用于处理SSL/TLS contexts, 定义 TLS/SSL 协议版本, certificate verification, or even enable client-side authentication 
` Scrapy默认的context factory不执行remote server certificate verification.这样对于爬虫来说是很不错的方式。如果你一定要进行证书校验，你可以使用'scrapy.core.downloader.contextfactory.BrowserLikeContextFactory' `
DOWNLOADER_CLIENT_TLS_METHOD：默认为'TLS'
DOWNLOADER_MIDDLEWARES：默认为{}
DOWNLOADER_MIDDLEWARES_BASE:默认为，你不应该修改它，而是应该修改DOWNLOADER_MIDDLEWARES
```

{
    'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,
    'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,
    'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,
    'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,
    'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,
    'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,
    'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,
    'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,
    'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,
    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,
    'scrapy.downloadermiddlewares.chunked.ChunkedTransferMiddleware': 830,
    'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,
    'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,
}

```
DOWNLOAD_DELAY：默认为0，单位秒
DOWNLOAD_HANDLERS：默认为{}
DOWNLOAD_HANDLERS_BASE:默认为：，你不应该修改DOWNLOAD_HANDLERS_BASE，而是修改DOWNLOAD_HANDLERS
```

{
    'file': 'scrapy.core.downloader.handlers.file.FileDownloadHandler',
    'http': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
    'https': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
    's3': 'scrapy.core.downloader.handlers.s3.S3DownloadHandler',
    'ftp': 'scrapy.core.downloader.handlers.ftp.FTPDownloadHandler',
}

```
DOWNLOAD_TIMEOUT：默认为180，单位秒
DOWNLOAD_MAXSIZE： 1073741824 (1024MB)，单位字节bytes

重试
RETRY_ENABLED=True
RETRY_TIMES=2
RETRY_HTTP_CODES=[500, 502, 503, 504, 408]

AJAXCRAWL_ENABLED=False ,Whether the AjaxCrawlMiddleware will be enabled

日志相关
LOG_ENABLED:默认True
LOG_ENCODING：默认为'utf-8'
LOG_FILE:默认为None，日志名称。
LOG_FORMAT：默认为 '%(asctime)s [%(name)s] %(levelname)s: %(message)s'
LOG_DATEFORMAT默认为'%Y-%m-%d %H:%M:%S'
LOG_LEVEL默认'DEBUG'
LOG_STDOUT默认为 False
重定向
REDIRECT_ENABLED=True
REDIRECT_MAX_TIMES=20
扩展
EXTENSIONS:默认为{}
EXTENSIONS_BASE:默认为
```

{
    'scrapy.extensions.corestats.CoreStats': 0,
    'scrapy.extensions.telnet.TelnetConsole': 0,
    'scrapy.extensions.memusage.MemoryUsage': 0,
    'scrapy.extensions.memdebug.MemoryDebugger': 0,
    'scrapy.extensions.closespider.CloseSpider': 0,
    'scrapy.extensions.feedexport.FeedExporter': 0,
    'scrapy.extensions.logstats.LogStats': 0,
    'scrapy.extensions.spiderstate.SpiderState': 0,
    'scrapy.extensions.throttle.AutoThrottle': 0,
}

```

NEWSPIDER_MODULE，默认为' '

SPIDER_MIDDLEWARES={}
SPIDER_MIDDLEWARES_BASE=
```

{
    'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware': 50,
    'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': 500,
    'scrapy.spidermiddlewares.referer.RefererMiddleware': 700,
    'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware': 800,
    'scrapy.spidermiddlewares.depth.DepthMiddleware': 900,
}

```

SPIDER_MODULES=[] :A list of modules where Scrapy will look for spiders.
ROBOTSTXT_OBEY=False
DEFAULT_ITEM_CLASS：默认为'scrapy.item.Item'，在Scrapy shell使用的默认item类
内存相关：
MEMDEBUG_ENABLED=False
MEMDEBUG_NOTIFY=[],例如：MEMDEBUG_NOTIFY = ['user@example.com']
MEMUSAGE_ENABLED=False 
MEMUSAGE_LIMIT_MB=0
MEMUSAGE_NOTIFY_MAIL=False，例如MEMUSAGE_NOTIFY_MAIL = ['user@example.com']

###  代理配置 
https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware
下面的比较简单：
1.在Scrapy工程下新建“middlewares.py”
```

# Importing base64 library because we'll need it ONLY in case if the proxy we are going to use requires authentication
import base64 
# Start your middleware class
class ProxyMiddleware(object):
    # overwrite process request
    def process_request(self, request, spider):
        # Set the location of the proxy
        request.meta['proxy'] = "http://YOUR_PROXY_IP:PORT"
  
        # Use the following lines if your proxy requires authentication
        proxy_user_pass = "USERNAME:PASSWORD"
        # setup basic authentication for the proxy
        encoded_user_pass = base64.encodestring(proxy_user_pass)
        request.headers['Proxy-Authorization'] = 'Basic ' + encoded_user_pass

```
2、修改settings.py
```

DOWNLOADER_MIDDLEWARES={
    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 110,  
    'tutorial.middlewares.ProxyMiddleware':100,
}

```
更好的配置请参考：https://github.com/kohn/HttpProxyMiddleware/blob/master，这个使用了多个代理，并且不断测试代理是否有效以及失效时重新获取。

##  异常体系 
scrapy.exceptions.DropItem：由item pipeline 抛出。
scrapy.exceptions.CloseSpider(reason='cancelled')
scrapy.exceptions.IgnoreRequest
scrapy.exceptions.NotConfigured
scrapy.exceptions.NotSupported

##  结合BeautifulSoup使用 
```

from bs4 import BeautifulSoup
import scrapy


class ExampleSpider(scrapy.Spider):
    name = "example"
    allowed_domains = ["example.com"]
    start_urls = (
        'http://www.example.com/',
    )

    def parse(self, response):
        # use lxml to get decent HTML parsing speed
        soup = BeautifulSoup(response.text, 'lxml')
        yield {
            "url": response.url,
            "title": soup.h1.string
        }

```
##  使用scrapy-jsonrpc服务器 
https://github.com/scrapy-plugins/scrapy-jsonrpc

$ pip install scrapy-jsonrpc
配置settings.py
```

JSONRPC_ENABLED=True
JSONRPC_PORT=[6080, 7030]
JSONRPC_LOGFILE=None
JSONRPC_HOST='127.0.0.1'
EXTENSIONS = {
    'scrapy_jsonrpc.webservice.WebService': 500,
}

```
访问地址http://localhost:6080/crawler
##  Does Scrapy manage cookies automatically? 
Yes, Scrapy receives and keeps track of cookies sent by servers, and sends them back on subsequent requests, like any regular web browser does.
For more info see Requests and Responses and CookiesMiddleware.
##  代码方式运行Scrapy 
You can use the API to run Scrapy from a script, instead of the typical way of running Scrapy via scrapy crawl.
```

import scrapy
from scrapy.crawler import CrawlerProcess

class MySpider(scrapy.Spider):
    # Your spider definition
    ...

process = CrawlerProcess({
    'USER_AGENT': 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'
})

process.crawl(MySpider)
process.start() # the script will block here until the crawling is finished

```
```

from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings

process = CrawlerProcess(get_project_settings())

# 'followall' is the name of one of the spiders of the project.
process.crawl('followall', domain='scrapinghub.com')
process.start() # the script will block here until the crawling is finished

```
如果是twisted应用，你还可以使用以下方式。
```

from twisted.internet import reactor
import scrapy
from scrapy.crawler import CrawlerRunner
from scrapy.utils.log import configure_logging

class MySpider(scrapy.Spider):
    # Your spider definition
    ...

configure_logging({'LOG_FORMAT': '%(levelname)s: %(message)s'})
runner = CrawlerRunner()

d = runner.crawl(MySpider)
d.addBoth(lambda _: reactor.stop())
reactor.run() # the script will block here until the crawling is finished

```

##  Running multiple spiders in the same process 
By default, Scrapy runs a single spider per process when you run scrapy crawl. However, Scrapy supports running multiple spiders per process using the internal API.
```

import scrapy
from scrapy.crawler import CrawlerProcess

class MySpider1(scrapy.Spider):
    # Your first spider definition
    ...

class MySpider2(scrapy.Spider):
    # Your second spider definition
    ...

process = CrawlerProcess()
process.crawl(MySpider1)
process.crawl(MySpider2)
process.start() # the script will block here until all crawling jobs are finished

```
Same example using CrawlerRunner:
```

import scrapy
from twisted.internet import reactor
from scrapy.crawler import CrawlerRunner
from scrapy.utils.log import configure_logging

class MySpider1(scrapy.Spider):
    # Your first spider definition
    ...

class MySpider2(scrapy.Spider):
    # Your second spider definition
    ...

configure_logging()
runner = CrawlerRunner()
runner.crawl(MySpider1)
runner.crawl(MySpider2)
d = runner.join()
d.addBoth(lambda _: reactor.stop())

reactor.run() # the script will block here until all crawling jobs are finished

```
Same example but running the spiders sequentially by chaining the deferreds:

```

from twisted.internet import reactor, defer
from scrapy.crawler import CrawlerRunner
from scrapy.utils.log import configure_logging

class MySpider1(scrapy.Spider):
    # Your first spider definition
    ...

class MySpider2(scrapy.Spider):
    # Your second spider definition
    ...

configure_logging()
runner = CrawlerRunner()

@defer.inlineCallbacks
def crawl():
    yield runner.crawl(MySpider1)
    yield runner.crawl(MySpider2)
    reactor.stop()

crawl()
reactor.run() # the script will block here until the last crawl call is finished

```
##  文件与图片下载 
```

FILES_STORE = '/path/to/valid/dir'
IMAGES_STORE = '/path/to/valid/dir'
ITEM_PIPELINES = {'scrapy.pipelines.images.ImagesPipeline': 1,'scrapy.pipelines.files.FilesPipeline': 1

```
                  
