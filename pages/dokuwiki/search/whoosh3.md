title: whoosh3 

#  Whoosh高亮搜索结果 
四个组件：
  * Fragments : 基于匹配的term在文档中的位置，把原始的文档砍成__fragments__
  * Scorers: 给每个fragment赋一个分值，允许系统根据分值排出最好的fragment.
  * Order functions: 控制展示给用户的fragment，是展示先出现在文档里的fragment,还是展示score最高的fragment.
  * Formatters: 把对应的fragment格式化为用户可读格式，如html
其他参考：http://m.doc00.com/doc/100100dk7
定制高亮:

**whoosh默认仅从文本的前32k字符中抽取fragment. 可以通过调整参数来增加这个值。**
```

results = mysearcher.search(myquery)
results.fragmenter.charlimit = 100000或者
results.fragmenter.charlimit = None 直接关闭限制

```
也可以初始化一个定制的framenter, 直接在fragment上设置字符限制:
```

sf = highlight.SentenceFragmenter(charlimit=100000)
results.fragmenter = s

```

fragment的数量:可以使用top参数控制每个文档返回的fragment的数量
```

print hit.highlights("content", top=5)

```
fragment的size: fragmenter的maxchar属性控制fragment的最大长度，默认是200. surround属性控制fragment里关键字前后截取的背景文字长度，默认是20
　　

Fragmenter:
Fragmenter控制如何从原始文本中提取摘录文字. Whoosh.hightlight包里已有以下预制的fragmenter:
  * whoosh.highlight.ContextFragmenter (默认)这是一个默认的智能fragment,可以发现匹配的term, 并抽取周围的文字生成 fragment. 这个fragmenter只产生包含匹配term的fragment.
  * whoosh.highlight.SentenceFragmenter依标点符号，按句子抽取
  * whoosh.highlight.WholeFragmenter返回整个文本作为fragment,高效（不做处理，当然高效）适合小文本
不同的fragmenter有不同的选项参数

Scorer:
Scorer是一个可调用的对象，使用fragment作为参数，返回一个可排序的值（值越大，代表该fragment质量越好）。　高亮系统使用这个值去挑选出展示给用户的最佳fragment.

Order:
order是一个函数，该函数以fragment为参数，返回一个可排序的值，通过该值对fragment进行排序。
whoosh.highlight模块有一下的排序函数:
FIRST（默认的）按出现顺序
SCORE：按得分顺序
还有其他不常用的: LONGER长fragment优先,SHORTER短fragment优先

Formatter:
formatter负责控制分值最高的fragment如何以格式化的方式呈现给用户。通过设置不同的formatter可以返回任何格式，包括:html,genshi事件流，SAX事件生成器，等。
highlight模块有一下预置实现
  * whoosh.highlight.HtmlFormatter以带class属性的html标签包围的方式返回匹配的term
  * whoosh.highlight.UppercaseFormatter把匹配的term以大写字母呈现
  * whoosh.highlight.GenshiFormatter输出一个genshi事件流
替换formatter的方式和上面替换其他三个元件的方式一致，更改results.formatter属性即可

```

results = mysearcher.search(myquery)
for hit in results:
    print(hit["title"])

```
可以使用whoosh.searching.Hit对象的highlights()方法来获取highlighted snippets 
```

results = mysearcher.search(myquery)
for hit in results:
    print(hit["title"])
    # Assume "content" field is stored
    print(hit.highlights("content"))

```
**highlights()第一个参数为field名，如果 field is stored，那么仅接受一个参数。如果不是存储的，那么接受一个text参数，值为你从别的地方(如数据库，文件等)中获取到的内容。**
```

results = mysearcher.search(myquery)
for hit in results:
    print(hit["title"])
    # Assume the "path" stored field contains a path to the original file
    with open(hit["path"]) as fileobj:
        filecontents = fileobj.read()
    print(hit.highlights("content", text=filecontents))

```

##  字符限制  
默认只提取32k,Whoosh only pulls fragments from the first 32K characters of the text.改变：
```

results = mysearcher.search(myquery)
results.fragmenter.charlimit = 100000
#或者
results.fragmenter.charlimit = None

```
如果自己实例化自定义Fragmenter
```

sf = highlight.SentenceFragmenter(charlimit=100000)
results.fragmenter = sf

```
##  Customizing the highlights 
Number of fragments
```

# Show a maximum of 5 fragments from the document
print hit.highlights("content", top=5)

```
Fragment size
```

# Allow larger fragments
results.fragmenter.maxchars = 300
# Show more context before and after
results.fragmenter.surround = 50

```
##  Fragmenter 
A fragmenter controls how to extract excerpts from the original text.
  * whoosh.highlight.ContextFragmenter (the default)
  * whoosh.highlight.SentenceFragmenter
  * whoosh.highlight.WholeFragmenter
##  Scorer 
A scorer is a **callable** that takes a whoosh.highlight.Fragment object and returns a sortable value 
```

def StandardDeviationScorer(fragment):
    """Gives higher scores to fragments where the matched terms are close
    together.
    """

    # Since lower values are better in this case, we need to negate the
    # value
    return 0 - stddev([t.pos for t in fragment.matched])

```
```

results.scorer = StandardDeviationScorer

```
##  Order 
The order is a function that takes a fragment and returns a sortable value used to sort the highest-scoring fragments before presenting them to the user (where fragments with lower values appear before fragments with higher values).
  * FIRST (the default)
  * SCORE
results.order = highlight.SCORE
##  Formatter 
A formatter contols how the highest scoring fragments are turned into a formatted bit of text for display to the user. It can return anything (e.g. plain text, HTML,)
  * whoosh.highlight.HtmlFormatter
  * whoosh.highlight.UppercaseFormatter
  * whoosh.highlight.GenshiFormatter
```

class BracketFormatter(highlight.Formatter):
    """Puts square brackets around the matched terms.
    """

    def format_token(self, text, token, replace=False):
        # Use the get_text function to get the text corresponding to the
        # token
        tokentext = highlight.get_text(text, token, replace)

        # Return the text as you want it to appear in the highlighted
        # string
        return "[%s]" % tokentext
      
brf = BracketFormatter()
results.formatter = brf

```
##  Highlighter object 
Rather than setting attributes on the results object, you can create a reusable **whoosh.highlight.Highlighter** object. Keyword arguments let you change the fragmenter, scorer, order, and/or formatter:
```

hi = highlight.Highlighter(fragmenter=my_cf, scorer=sds)

```
You can then use the whoosh.highlight.Highlighter.**highlight_hit**() method to get highlights for a Hit object:
```

for hit in results:
    print(hit["title"])
    print(hi.highlight_hit(hit))

```
##  Speeding up highlighting 
```

# Record per-document term matches
results = searcher.search(myquery, terms=True)

```
###  PinpointFragmenter 
使用场景：If you have long documents and have increased/disabled the character limit, and/or if the field has a very complex analyzer, re-tokenizing may be slow.

whoosh.highlight.PinpointFragmenter
```

# Index the start and end chars of each term
schema = fields.Schema(content=fields.TEXT(stored=True, chars=True))
# Record per-document term matches
results = searcher.search(myquery, terms=True)
results.fragmenter = highlight.PinpointFragmenter()

```
##  HtmlFormatter 
whoosh.highlight.HtmlFormatter(tagname='strong', between='...', classname='match', termclass='term', maxclasses=5, attrquote='"')
```

>>> hf = HtmlFormatter(tagname="span", classname="match", termclass="term")
>>> hf(mytext, myfragments)
"The <span class="match term0">template</span> <span class="match term1">geometry</span> is..."

```
