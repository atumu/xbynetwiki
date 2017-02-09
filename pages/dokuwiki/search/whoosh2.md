title: whoosh2 

#  Whoosh进阶 
##  时间日期索引与解析 
whoosh.fields.DATETIME 
```

from datetime import datetime, timedelta
from whoosh import fields, index

schema = fields.Schema(title=fields.TEXT, content=fields.TEXT,
                       date=fields.DATETIME)
ix = index.create_in("indexdir", schema)

w = ix.writer()
w.add_document(title="Document 1", content="Rendering images from the command line",
               date=datetime.utcnow())
w.add_document(title="Document 2", content="Creating shaders using a node network",
               date=datetime.utcnow() + timedelta(days=1))
w.commit()

```
解析日期查询：
whoosh.qparser.dateparse.DateParserPlugin
```

from whoosh import index
from whoosh.qparser import QueryParser
from whoosh.qparser.dateparse import DateParserPlugin

ix = index.open_dir("indexdir")
# Instatiate a query parser
qp = QueryParser("content", ix.schema)

# Add the DateParserPlugin to the parser
qp.add_plugin(DateParserPlugin())

```
支持的查询格式：
```

20050912
2005 sept 12th
june 23 1978
23 mar 2005
july 1985
sep 12
today
yesterday
tomorrow
now
next friday
last tuesday
5am
10:25:54
23:12
8 PM
4:46 am oct 31 2010
last tuesday to today
today to next friday
jan 2005 to feb 2008
-1 week to now
now to +2h
-1y6mo to +2 yrs 23d

```
如：
render date:'last tuesday' command
date:['last tuesday' to 'next friday']
```

from whoosh import index
from whoosh.qparser import QueryParser

ix = index.open_dir("indexdir")
qp = QueryParser("content", schema=ix.schema)

# Find all datetimes in 2005
q = qp.parse(u"date:2005")
# Find all datetimes on June 24, 2005
q = qp.parse(u"date:20050624")
# Find all datetimes from 1am-2am on June 24, 2005
q = qp.parse(u"date:2005062401")
# Find all datetimes from Jan 1, 2005 to June 2, 2010
q = qp.parse(u"date:[20050101 to 20100602]")

```

**设置查询的基时间点（base datetime）**
```

qp.add_plugin(DateParserPlugin(basedate=my_datetime))

```

**注册error callback**
```

errors = []
def add_error(msg):
    errors.append(msg)
qp.add_plugin(DateParserPlug(callback=add_error))

q = qp.parse(u"date:blarg")
# errors == [u"blarg"]

```
##  analyzers 
An analyzer是一个函数或者是一个可调用的类(callable class ,a class with a &lt;nowiki&gt;__call__&lt;/nowiki&gt; method)接受一个unicode string作为参数，` yields ` a series of ` analysis.Token ` objects.
Usually a “token” is a word, for example the string “Mary had a little lamb” might **yield** the tokens “Mary”, “had”, “a”, “little”, and “lamb”. 
**An analyzer是一个tokenizer，零个或多个filters的wrapper 。**
比如 whoosh.analysis.RegexTokenizer通过正则来提取单词，并忽略空格和标点
```

>>> from whoosh.analysis import RegexTokenizer
>>> tokenizer = RegexTokenizer()
>>> for token in tokenizer(u"Hello there my friend!"):
...   print repr(token.text)
u'Hello'
u'there'
u'my'
u'friend'

```
**A filter** is a callable that takes a generator of Tokens (either a tokenizer or another filter) and in turn yields a series of Tokens.
比如whoosh.analysis.LowercaseFilter()将字符串转为小写
```

def LowercaseFilter(tokens):
    """Uses lower() to lowercase token text. For example, tokens
    "This","is","a","TEST" become "this","is","a","test".
    """

    for t in tokens:
        t.text = t.text.lower()
        yield t

```
使用Filter过滤tokenizer分词结果
```

>>> from whoosh.analysis import LowercaseFilter
>>> for token in LowercaseFilter(tokenizer(u"These ARE the things I want!")):
...   print repr(token.text)

```

An analyzer只是意味着组合 a tokenizer and some filters into a single package.
实现一个analyzer，你可以创建一个自定义的类，也可以通过管道符"|"来结合tokenizer与多个filter，比如：
```

my_analyzer = RegexTokenizer() | LowercaseFilter() | StopFilter()

```
这个能够工作的前提是Tokenizer是**whoosh.analysis.Composable**的子类。

###  使用analyzers 
```

schema = Schema(content=TEXT(analyzer=StemmingAnalyzer()))

```
###  Advanced Analysis 
Token objects,常见属性如下：
  * str		mode	The mode in which the analyzer is being called, e.g. ‘index’ during indexing or ‘query’ during query parsing‘’
  * bool	positions	Whether term positions are recorded in the token	False
  * bool	chars	Whether term start and end character indices are recorded in the token	False
  * bool	boosts	Whether per-term boosts are recorded in the token	False
  * bool	removestops	Whether stop-words should be removed from the token stream
  * **unicode	text**	The text of the token (this should always be present)
  * unicode	original	The original (pre-filtered) text of the token. The tokenizer may record this, and filters are expected not to modify it.
  * int		pos	The position of the token in the stream, starting at 0 (only set if positions is True)
  * int		startchar	The character index of the start of the token in the original string (only set if chars is True)
  * int		endchar	The character index of the end of the token in the original string (only set if chars is True)
  * float	boost	The boost for this token (only set if boosts is True)
  * bool	stopped	Whether this token is a “stop” word (only set if removestops is False)

###  关于Token需要注意的 
由于Python中创建对象是比较慢的，所以whoose不会为每一个token创建一个Token对象，而是所有token复用一个analysis.Token对象。这种方式在我们使用generator yield时` 并且不保留token对象引用时 `没有什么问题，但是像下面这样就会出现问题：因为它保留了Token引用，而所有的Token都是同一个对象所致。
```

>>> list(tokenizer(u"Hello there my friend"))
[Token(u"friend"), Token(u"friend"), Token(u"friend"), Token(u"friend")]

```
我们应该像下面这样使用：
```

>>> [t.text for t in tokenizer(u"Hello there my friend")]

```

注意：如果你通过定义类的方式实现自己的 tokenizer, filter, or analyzer，你需要实现&lt;nowiki&gt;__eq__&lt;/nowiki&gt;方法.This is important to allow comparison of Schema objects.

##  词根，重音，复数(Stemming, variations, and accent folding)等 
比如搜索render,可能需要包含render,和renders, rendering, rendered等，搜索cafe，也包含café.
分析词根
```

>>> from whoosh.lang.porter import stem
>>> stem("rendering")
'render'

```

whoosh.analysis.StemFilter
```

>>> rext = RegexTokenizer()
>>> stream = rext(u"fundamentally willows")
>>> stemmer = StemFilter()
>>> [token.text for token in stemmer(stream)]
[u"fundament", u"willow"]

```

变化(Variations)
```

>>> from whoosh.lang.morph_en import variations
>>> variations("rendered")
set(['rendered', 'rendernesses', 'render', 'renderless', 'rendering',
'renderness', 'renderes', 'renderer', 'renderements', 'rendereless',
'renderenesses', 'rendere', 'renderment', 'renderest', 'renderement',
'rendereful', 'renderers', 'renderful', 'renderings', 'renders', 'renderly',
'renderely', 'rendereness', 'renderments'])

```
```

from whoosh import qparser, query

qp = qparser.QueryParser("content", termclass=query.Variations)

```
##  N-gram模型 
N-gram indexing is a powerful method for getting fast, “search as you type” functionality like iTunes. It is also useful for quick and effective indexing of languages such as Chinese and Japanese without word breaks.

whoosh.analysis.NgramTokenizer
```

>>> ngt = NgramTokenizer(minsize=2, maxsize=4)
>>> [token.text for token in ngt(u"hi there")]

```
whoosh.analysis.NgramFilter
```

>>> my_analyzer = StandardAnalyzer() | NgramFilter(minsize=2, maxsize=4)
>>> [token.text for token in my_analyzer(u"rendering shaders")]

```
whoosh.fields.NGRAM
whoosh.fields.NGRAMWORDS
NGRAM与NGRAMWORDS区别：前者包含空格与标点，后者会使用tokenizer分词。然后经过NgramFilter

##  Sorting and faceting 
Sorting and faceting search results基于facets,每个facet关联搜索结果文档中的一个值，允许进行排序与分组。
whoosh包含许多facet类型用于排序和分组。

Sorting排序:
默认基于评分机制。
标记field可排序：sortable=True
```

schema = fields.Schema(title=fields.TEXT(sortable=True),
                       content=fields.TEXT,
                       modified=fields.DATETIME(sortable=True)
                       )

```

**column types:**
当你为field指定sortable=True的时候，whoosh会将每个文档对应field的值存在一个column中，column类型决定了存储field的格式。
whoosh.columns模块。
默认text field的column类型为whoosh.columns.VarBytesColumn
numeric fields column类型为 whoosh.columns.NumericColumn
固定长度的whoosh.columns.FixedBytesColumn
whoosh.columns.RefBytesColumn (which can handle both variable and fixed-length values).
```

from whoosh import columns, fields

category_col = columns.RefBytesColumn()
schema = fields.Schema(title=fields.TEXT(sortable=True),
                       category=fields.KEYWORD(sortable=category_col)

```

###  对搜索结果进行排序 
```

from whoosh import fields, index, qparser

schema = fields.Schema(title=fields.TEXT(stored=True),
                       price=fields.NUMERIC(sortable=True))
ix = index.create_in("indexdir", schema)

with ix.writer() as w:
    w.add_document(title="Big Deal", price=20)
    w.add_document(title="Mr. Big", price=10)
    w.add_document(title="Big Top", price=15)

with ix.searcher() as s:
    qp = qparser.QueryParser("big", ix.schema)
    q = qp.parse(user_query_string)

    # Sort search results from lowest to highest price
    results = s.search(q, sortedby="price")
    for hit in results:
        print(hit["title"])

```
sortedby属性：值可以为
  * A FacetType object
  * A field name string
  * A list of FacetType objects and/or field name strings
```

results = searcher.search(myquery, sortedby="size")
facet = sorting.FieldFacet("price", reverse=True)
results = searcher.search(myquery, sortedby=facet)

mf = sorting.MultiFacet()
mf.add_field("size")
mf.add_field("price", reverse=True)
results = searcher.search(myquery, sortedby=mf)

# or...
sizes = sorting.FieldFacet("size")
prices = sorting.FieldFacet("price", reverse=True)
results = searcher.search(myquery, sortedby=[sizes, prices])

cats = sorting.FieldFacet("category")
scores = sorting.ScoreFacet()
results = searcher.search(myquery, sortedby=[cats, scores])

```


###  分组 
groupedby参数，值可以为：
  * A FacetType object
  * A field name string
  * A list or tuple of field name strings
  * A dictionary mapping facet names to FacetType objects
  * A Facets object
```

results = searcher.search(myquery, groupedby="category")
cats = sorting.FieldFacet("category")
tags = sorting.FieldFacet("tags", allow_overlap=True)
results = searcher.search(myquery, groupedby={"category": cats, "tags": tags})
# ...or, using a Facets object has a little less duplication
facets = sorting.Facets()
facets.add_field("category")
facets.add_field("tags", allow_overlap=True)
results = searcher.search(myquery, groupedby=facets)

```

###  Facet types 
FieldFacet
QueryFacet
RangeFacet
DateRangeFacet
ScoreFacet
FunctionFacet
StoredFieldFacet

MultiFacet
