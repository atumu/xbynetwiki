title: python学习笔记124 

#  Python总结之框架 
##  flask[微型网络开发框架] 
```

		# http://dormousehole.readthedocs.org/en/latest/
		# html放在 ./templates/   js放在 ./static/
		
		request.args.get('page', 1)          # 获取参数 ?page=1
		request.json                         # 获取传递的整个json数据
		request.form.get("host",'127')       # 获取表单值
			
		简单实例 # 接收数据和展示

			import MySQLdb as mysql
			from flask import Flask, request

			app = Flask(__name__)
			db.autocommit(True)
			c = db.cursor()

			"""
			CREATE TABLE `statusinfo` (
			  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
			  `hostname` varchar(32) NOT NULL,
			  `load` float(10) NOT NULL DEFAULT 0.00,
			  `time` int(15) NOT NULL,
			  `memtotal` int(15) NOT NULL,
			  `memusage` int(15) NOT NULL,
			  `memfree` int(15) NOT NULL,
			  PRIMARY KEY (`id`)
			) ENGINE=InnoDB AUTO_INCREMENT=161 DEFAULT CHARSET=utf8;
			"""

			@app.route("/collect", methods=["GET", "POST"])
			def collect():
				sql = ""
				if request.method == "POST":
					data = request.json                      # 获取传递的json
					hostname = data["Host"]
					load = data["LoadAvg"]
					time = data["Time"]
					memtotal = data["MemTotal"]
					memusage = data["MemUsage"]
					memfree = data["MemFree"]
					
					try:
						sql = "INSERT INTO `statusinfo` (`hostname`,`load`,`time`,`memtotal`,`memusage`,`memfree`) VALUES('%s', %s, %s, %s, %s, %s);" % (hostname, load,time,memtotal,memusage,memfree)
						ret = c.execute(sql)
						return 'ok'
					except mysql.IntegrityError:
						return 'errer'

			@app.route("/show", methods=["GET", "POST"])
			def show():
				try:
					hostname = request.form.get("hostname")     # 获取表单方式的变量值
					sql = "SELECT `load` FROM `statusinfo` WHERE hostname = '%s';" % (hostname)
					c.execute(sql)
					ones = c.fetchall()
					return render_template("sysstatus.html", data=ones, sql = sql)
				except:
					print 'hostname null'

			from flask import render_template
			@app.route("/xxx/<name>")
			def hello_xx(name):
				return render_template("sysstatus.html", name='teach')

			if __name__ == "__main__":
				app.run(host="0.0.0.0", port=50000, debug=True)


```
##  twisted [非阻塞异步服务器框架] 
# 用来进行网络服务和应用程序的编程。虽然 Twisted Matrix 中有大量松散耦合的模块化组件，但该框架的中心概念还是非阻塞异步服务器这一思想。对于习惯于线程技术或分叉服务器的开发人员来说，这是一种新颖的编程风格，但它却能在繁重负载的情况下带来极高的效率。
```

		pip install twisted
		
		from twisted.internet import protocol, reactor, endpoints

		class Echo(protocol.Protocol):
			def dataReceived(self, data):
				self.transport.write(data)
		class EchoFactory(protocol.Factory):
			def buildProtocol(self, addr):
				return Echo()

		endpoints.serverFromString(reactor, "tcp:1234").listen(EchoFactory())
		reactor.run()


```
##  greenlet [微线程/协程框架] 
# 更加原始的微线程的概念,没有调度,或者叫做协程。这在你需要控制你的代码时很有用。你可以自己构造微线程的 调度器；也可以使用"greenlet"实现高级的控制流。例如可以重新创建构造器；不同于Python的构造器，我们的构造器可以嵌套的调用函数，而被嵌套的函数也可以 yield 一个值。
pip install greenlet

##  tornado[极轻量级Web服务器框架] 
```

		# 高可伸缩性和epoll非阻塞IO,响应快速,可处理数千并发连接,特别适用用于实时的Web服务
		# http://www.tornadoweb.cn/documentation
		pip install tornado
		
		import tornado.ioloop
		import tornado.web

		class MainHandler(tornado.web.RequestHandler):
			def get(self):
				self.write("Hello, world")

		application = tornado.web.Application([
			(r"/", MainHandler),
		])

		if __name__ == "__main__":
			application.listen(8888)
			tornado.ioloop.IOLoop.instance().start()

```
##  Scrapy [web抓取框架] 
# Python开发的一个快速,高层次的屏幕抓取和web抓取框架，用于抓取web站点并从页面中提取结构化的数据。Scrapy用途广泛，可以用于数据挖掘、监测和自动化测试。
```

		pip install scrapy
		
		from scrapy import Spider, Item, Field

		class Post(Item):
			title = Field()

		class BlogSpider(Spider):
			name, start_urls = 'blogspider', ['http://blog.scrapinghub.com']

			def parse(self, response):
				return [Post(title=e.extract()) for e in response.css("h2 a::text")]
				
		scrapy runspider myspider.py

```
django   [重量级web框架]
bottle   [轻量级的Web框架]

