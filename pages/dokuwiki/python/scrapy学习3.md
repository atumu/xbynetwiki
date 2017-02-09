title: scrapy学习3 

#  Scrapy学习3之常见问题 
##  关于去重 
去重机制配置
DUPEFILTER_CLASS：默认为'scrapy.dupefilters.**RFPDupeFilter**'，使用scrapy.utils.request.request_fingerprint函数提供的请求指纹策略。
想改变这种去重策略，你可以继承RFPDupeFilter，覆盖request_fingerprint方法，该方法接收Request object作为参数，返回 its fingerprint (a string).
scrapy默认有自己的去重机制，默认使用scrapy.dupefilters.**RFPDupeFilter**类进行去重，主要逻辑如下
```

if include_headers:
    include_headers = tuple(to_bytes(h.lower())
                             for h in sorted(include_headers))
cache = _fingerprint_cache.setdefault(request, {})
if include_headers not in cache:
    fp = hashlib.sha1()
    fp.update(to_bytes(request.method))
    fp.update(to_bytes(canonicalize_url(request.url)))
    fp.update(request.body or b'')
    if include_headers:
        for hdr in include_headers:
            if hdr in request.headers:
                fp.update(hdr)
                for v in request.headers.getlist(hdr):
                    fp.update(v)
    cache[include_headers] = fp.hexdigest()
return cache[include_headers]

```
**默认的去重指纹是sha1(method + url + body + header)**.

##  为每个pipeline配置spider 
上面我们是在settings.py里面配置pipeline，这里的配置的pipeline会作用于所有的spider，我们可以为每一个spider配置不同的pipeline，设置Spider的custom_settings对象
```

class LaSpider(CrawlSpider):
    ...
    # 自定义配置
    custom_settings = {
        'ITEM_PIPELINES': {
            'tutorial.pipelines.TestPipeline.TestPipeline': 1,
        }
    }

```
##  缓存 
scrapy默认已经自带了缓存的功能，通常我们只需要配置即可，打开settings.py
```

# 打开缓存
HTTPCACHE_ENABLED = True
# 设置缓存过期时间（单位：秒）
#HTTPCACHE_EXPIRATION_SECS = 0

# 缓存路径(默认为：.scrapy/httpcache)
HTTPCACHE_DIR = 'httpcache'
# 忽略的状态码
HTTPCACHE_IGNORE_HTTP_CODES = []
# 缓存模式(文件缓存)
HTTPCACHE_STORAGE = 'scrapy.extensions.httpcache.FilesystemCacheStorage'

```
更多在[这里](http://scrapy.readthedocs.org/en/latest/topics/downloader-middleware.html#httpcache-middleware-settings)
##  不同的item指定不同pipeline处理 
可以直接在pipeline中用if进行判断
```

if isinstance(item, Aitem):
    db.save(item)


```