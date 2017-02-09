title: scrapy学习2 

#  Scrapy学习2之定制Middleware 
##  Downloader Middleware 
https://doc.scrapy.org/en/latest/topics/downloader-middleware.html
scrapy.downloadermiddlewares.DownloaderMiddleware基类
方法：
  * process_request(request, spider)，should either: return None, return a Response object, return a Request object, or raise IgnoreRequest.
  * process_response(request, response, spider) should either: return a Response object, return a Request object or raise a IgnoreRequest exception.
  * process_exception(request, exception, spider)，should return: either None, a Response object, or a Request object.

##  Spider Middleware 
https://doc.scrapy.org/en/latest/topics/spider-middleware.html
scrapy.spidermiddlewares.SpiderMiddleware基类
  * process_spider_input(response, spider) should return None or raise an exception.
  * process_spider_output(response, result, spider)，must return an iterable of Request, dict or Item objects.
  * process_spider_exception(response, exception, spider),should return either None or an iterable of Response, dict or Item objects.
  * process_start_requests(start_requests, spider),must return only requests (not items).
