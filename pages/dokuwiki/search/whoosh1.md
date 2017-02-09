title: whoosh1 

#  Whoosh入门 
官网：http://whoosh.readthedocs.io/en/latest/index.html
当前版本:2.7.4
Whoosh 是一个纯python实现的全文搜索组件。Whoosh不但功能完善，还非常的快。
主要特性：
    * 敏捷的API（Pythonic API）。
    * 纯python实现，无二进制包。程序不会莫名其妙的崩溃。
    * 按字段进行索引。
    * 索引和搜索都非常的快 -- 是目前最快的纯python全文搜索引擎。
    * 良好的构架，评分模块/分词模块/存储模块等各个模块都是可插拔的。
    * 功能强大的查询语言（通过pyparsing实现功能）。
    * 纯python实现的拼写检查（目前唯一的纯python拼写检查实现）

为啥选择Whoosh 
    * 纯python实现，省了编译二进制包的繁琐过程。
    * python代码比java更容易读懂，而且用起来也更方便。（翻者注：这个容易引发口水）
    * 在很多时候易用性比单纯的最求速度更重要。

Whoosh从其他的开源搜索引擎中获取了大量的灵感。 基础构建参考Lucene，使用KinoSearch的索引算法，部分评分算法来自Terrier，英文的词语态变化来自Minion.

```

>>> from whoosh.index import create_in
>>> from whoosh.fields import *
>>> schema = Schema(title=TEXT(stored=True), path=ID(stored=True), content=TEXT)
>>> ix = create_in("indexdir", schema)
>>> writer = ix.writer()
>>> writer.add_document(title=u"First document", path=u"/a",
...                     content=u"This is the first document we've added!")
>>> writer.add_document(title=u"Second document", path=u"/b",
...                     content=u"The second one is even more interesting!")
>>> writer.commit()
>>> from whoosh.qparser import QueryParser
>>> with ix.searcher() as searcher:
...     query = QueryParser("content", ix.schema).parse("first")
...     results = searcher.search(query)
...     results[0]
...
{"title": u"First document", "path": u"/a"}

```
##  几个关键类 
###  Index和Schema 
```

from whoosh.fields import Schema, STORED, ID, KEYWORD, TEXT

schema = Schema(title=TEXT(stored=True), content=TEXT,
                path=ID(stored=True), tags=KEYWORD, icon=STORED)

import os.path
from whoosh.index import create_in

if not os.path.exists("index"):
    os.mkdir("index")
ix = create_in("index", schema)  

```
Schema定义需要被索引数据的模式。每个Schema包含多个Field。每个Field又分为是否被索引，是否存储，是否分词德等特性(STORED, ID, KEYWORD, TEXT,etc.)
Index对象用于创建索引.通过create_in函数来创建一个Index.后续可以通过open_dir函数来打开一个Index

感觉这些概念都与lucene似曾相识。
**Schema Field类型**
  * whoosh.fields.ID 会被索引，可选存储，但是不会被分词，作为一个独立单元。不会存储frequency information。
  * whoosh.fields.STORED 会被存储，但是不会被索引。
  * whoosh.fields.KEYWORD 会被索引,可以选择存储。用于空格或逗号分隔的关键字列表。几个有用属性：` stored=True `，lowercase=True.默认情况下是以空格分隔关键字，如果是逗号，设置commas=True.还有打分机制scorable=True.
  * whoosh.fields.TEXT 会被索引(可选存储)，同时会存储项位置用于phrase searching.默认使用whoosh.analysis.**StandardAnalyzer**来进行分词.如果你想换一个，可以指定analyzer属性，例如TEXT(analyzer=analysis.StemmingAnalyzer()).默认情况下，该Text Field会存储索引项(term)的位置信息，这是为了确保phrases短语搜索.你也可以禁用该特性TEXT(phrase=False)。默认情况下，它也不会被存储，但是你可以改变这一行为TEXT(stored=True)
  * whoosh.fields.NUMERIC 用于数字
  * whoosh.fields.BOOLEAN 用于bool类型,支持搜索关键字yes, no, true, false, 1, 0, t or f.
  * whoosh.fields.DATETIME 用于datetime类型
  * whoosh.fields.NGRAM与whoosh.fields.NGRAMWORDS 

创建物理索引，物理索引只需要创建一次即可。
```

import os.path
from whoosh.index import create_in

if not os.path.exists("index"):
    os.mkdir("index")
ix = create_in("index", schema)

```
在底层，这个会创建一个Storage对象来存储索引，一个Storage对象代表索引存储的媒介。一般是FileStorage，它在一个目录中创建所有文件。
以后每次需要Index时可以通过open_dir函数来打开
```

from whoosh.index import open_dir

ix = open_dir("index")

```

除了通过直接实例化Schema类，你还可以通过继承SchemaClass的方式来获取Schema：
```

from whoosh.fields import SchemaClass, TEXT, KEYWORD, ID, STORED

class MySchema(SchemaClass):
    path = ID(stored=True)
    title = TEXT(stored=True)
    content = TEXT
    tags = KEYWORD

```

####  Schema被创建并索引后的修改 
` Schema被创建并索引后的修改 `
```

writer = ix.writer()
writer.add_field("fieldname", fields.TEXT(stored=True))
writer.remove_field("content")
writer.commit()

```
如果后端是采用filedb作为存储，那么当你移除field时，只是简单的从schema中移除，而与其相关的数据还是会被保留直到你执行 optimize操作(Optimizing will compact the index, removing references to the deleted field as it goes:)。
```

writer = ix.writer()
writer.add_field("uuid", fields.ID(stored=True))
writer.remove_field("path")
writer.commit(optimize=True)

```
####  Dynamic fields 
允许你采用glob，模式来模糊化field的名字.支持*, ?, and/or [abc]通配符,使用glob=True属性打开该特性
```

#schema.add("*_d", fields.DATETIME(stored=True), glob=True)
schema = fields.Schema(path=fields.ID)
schema.add("*_id", fields.ID, glob=True)

ix = index.create_in("myindex", schema)

w = ix.writer()
w.add_document(path=u"/a", test_id=u"alfa")
w.add_document(path=u"/b", class_id=u"MyClass")
# ...
w.commit()

qp = qparser.QueryParser("path", schema=schema)
q = qp.parse(u"test_id:alfa")
with ix.searcher() as s:
    results = s.search(q)

```
####  Field倍数因子 
Field boosts
```

schema = Schema(title=TEXT(field_boost=2.0), body=TEXT)

```
####  Field types 
所有field type都是fields.FieldType的子类,如ID,FieldType的主要属性如下：
  * format	fields.Format	Defines what kind of information a field records about each term, and how the information is stored on disk.
  * vector	fields.Format	Optional: if defined, the format in which to store per-document forward-index information for this field.
  * scorable	bool	If True, the length of (number of terms in) the field in each document is stored in the index. Slightly misnamed, since field lengths are not required for all scoring. However, field lengths are required to get proper results from BM25F.
  * stored	bool	If True, the value of this field is stored in the index.
  * unique	bool	If True, the value of this field may be used to replace documents with the same value when the user calls document_update() on an IndexWriter.
####  Formats 
fields.Format对象调用analyzer进行分词，然后编码分词信息到token。
预置以下几种formats:
  * Stored	A “null” format for fields that are stored but not indexed.
  * Existence	Records only whether a term is in a document or not, i.e. it does not store term frequency. Useful for identifier fields (e.g. path or id) and “tag”-type fields, where the frequency is expected to always be 0 or 1.
  * Frequency	Stores the number of times each term appears in each document.
  * Positions	Stores the number of times each term appears in each document, and at what positions.

STORED field使用Stored format
ID field使用Existence format
KEYWORD field使用Frequency format
TEXT field(默认phrase=True，否则不使用)使用Positions format 


**Vector :略。**

###  IndexWriter 
获取Index对象之后，我们需要获取IndexWriter，通过Index的writer()方法可以获取IndexWriter对象。然后通过IndexWriter的add_document(* *kwargs)方法来添加文档。
```

writer = ix.writer()
writer.add_document(title=u"My document", content=u"This is my document!",
                    path=u"/a", tags=u"first short", icon=u"/icons/star.png")
writer.add_document(title=u"Second try", content=u"This is the second example.",
                    path=u"/b", tags=u"second short", icon=u"/icons/sheep.png")
writer.add_document(title=u"Third time's the charm", content=u"Examples are many.",
                    path=u"/c", tags=u"short", icon=u"/icons/book.png")
writer.commit()

```
当然，你也没必要输入给所有的field赋值。
**如果你想索引的值和存储的值不同的话**，可以像下面这样：
```

writer.add_document(title=u"Title to be indexed", _stored_title=u"Stored title")

```
最后提交
```

writer.commit()

```
###  Searcher 
searcher = index.searcher()
searcher对象打开之后需要关闭，因为他关联这一系列文件。有两种方式：手动关闭和with
```

try:
    searcher = ix.searcher()
    ...
finally:
    searcher.close() 

```
```

with ix.searcher() as searcher:
    ...

``` 
接下来就是调用searcher对象的search()方法，它接受一个Query对象作为参数。Query对象的获取可以通过直接构建，也可以通过QueryParser来解析查询字符串获取。
```

# Construct query objects directly

from whoosh.query import *
myquery = And([Term("content", u"apple"), Term("content", "bear")])

```
```

# Parse a query string

from whoosh.qparser import QueryParser
parser = QueryParser("content", ix.schema)
myquery = parser.parse(querystring)

```
QueryParser构造器接受两个参数，一个需要检索的字段名，一个索引的模式.
接下来调用search()方法来获取Results：
```

>>> results = searcher.search(myquery)
>>> print(len(results))
1
>>> print(results[0])
{"title": "Second try", "path": "/b", "icon": "/icons/sheep.png"}

```

注意：
**QueryParser**实现的查询语言和Lucene是类似的。它允许你使用AND or OR来连接terms，使用NOT，使用括号来组织terms到一个结果，还提供了范围，前缀，wildcard查询。
```

>>> print(parser.parse(u"render shade animate"))
And([Term("content", "render"), Term("content", "shade"), Term("content", "animate")])

>>> print(parser.parse(u"render OR (title:shade keyword:animate)"))
Or([Term("content", "render"), And([Term("title", "shade"), Term("keyword", "animate")])])

>>> print(parser.parse(u"rend*"))
Prefix("content", "rend")

```

whoose还包含了额外特性哟娜来处理搜索结果results，比如：
  * 结果排序
  * 高亮搜索关键字
  * 结果分页
  * Expanding the query terms based on the top few documents found.

##  索引文档(index documents) 
创建索引：index.create_in:
```

import os, os.path
from whoosh import index

if not os.path.exists("indexdir"):
    os.mkdir("indexdir")

ix = index.create_in("indexdir", schema)

```
打开索引：index.open_dir:
```

import whoosh.index as index

ix = index.open_dir("indexdir")

```

使用` FileStorage `创建或打开索引
```

from whoosh.filedb.filestore import FileStorage
storage = FileStorage("indexdir")

# Create an index
ix = storage.create_index(schema)

# Open an existing index
storage.open_index()

```

在同一目录创建多个索引对象：使用**indexname属性**
```

# Using the convenience functions
ix = index.create_in("indexdir", schema=schema, indexname="usages")
ix = index.open_dir("indexdir", indexname="usages")

# Using the Storage object
ix = storage.create_index(schema, indexname="usages")
ix = storage.open_index(indexname="usages")

```

**检测目录是否包含有效索引**
```

exists = index.exists_in("indexdir")
usages_exists = index.exists_in("indexdir", indexname="usages")

```

**索引文档**：注意：IndexWriter对象不是线程安全的。
```

ix = index.open_dir("index")
writer = ix.writer()
writer.add_document(title=u"My document", content=u"This is my document!",
                    path=u"/a", tags=u"first short", icon=u"/icons/star.png")
writer.commit()

```

###  更新文档： 
add_document只会不断地插入文档，而不会更新。如需更新请使用update_document
update_document遇到` unique field `时会替换该文档。
```

from whoosh.fields import Schema, ID, TEXT

schema = Schema(path = ID(unique=True), content=TEXT)

ix = index.create_in("index")
writer = ix.writer()
writer.add_document(path=u"/a", content=u"The first document")
writer.add_document(path=u"/b", content=u"The second document")
writer.commit()

writer = ix.writer()
# Because "path" is marked as unique, calling update_document with path="/a"
# will delete any existing documents where the "path" field contains "/a".
writer.update_document(path=u"/a", content="Replacement for the first document")
writer.commit()

```
**The “unique” field(s) must be indexed.**
如果不存在需要被更新的文档，update_document就会表现为add_document的行为



**提交与取消：**
writer.commit()
writer.cancel()

**Merging segments**：
一个索引可以有很多片段，片段可以被合并.

###  删除文档： 
  * delete_document(docnum)
  * is_deleted(docnum)
  * delete_by_term(fieldname, termtext)
  * delete_by_query(query)
例如:
```

# Delete document by its path -- this field must be indexed
ix.delete_by_term('path', u'/a/b/c')
# Save the deletion to disk
ix.commit()

```
注意：在filedb后端存储中，删除只是简单地将该docnum添加到一个删除文档列表中。文档还是保留的，只是不会出现在搜索结果中。
如果想要彻底删除，需要你手动merge.
writer.commit(optimize=True)或者Index.optimize()

##  检索文档 
whoosh.searching.Searcher，通过Index的searcher()方法获取
```

#searcher = myindex.searcher()
with ix.searcher() as searcher:
    ...

```
由于Searcher代表着打开文件的集合。所以使用完毕需要关闭它。使用with关键字。
```

from whoosh.qparser import QueryParser

qp = QueryParser("content", schema=myindex.schema)
q = qp.parse(u"hello world")

with myindex.searcher() as s:
    results = s.search(q)

```
默认情况下results仅包含最多10条匹配文档，如果需要更多，使用limit参数
```

results = s.search(q, limit=20) #前20条
results = s.search(q, limit=None) #所有

```
检索一页,参考[这里](http://whoosh.readthedocs.io/en/latest/api/searching.html#whoosh.searching.Searcher.search_page)
```

results = s.search_page(q, 1) #默认一页10条,1表示需要获取的页
results = s.search_page(q, 5, pagelen=20)

querystring = request.get("q")
query = queryparser.parse("content", querystring)

pagenum = int(request.get("page", 1))
pagelen = int(request.get("perpage", 10))

results = searcher.search_page(query, pagenum, pagelen=pagelen)
print("Page %d of %d" % (results.pagenum, results.pagecount))
print("Showing results %d-%d of %d"
      % (results.offset + 1, results.offset + results.pagelen + 1,
         len(results)))
for hit in results:
    print("%d: %s" % (hit.rank + 1, hit["title"]))

```
###  Results 
Results对象类似一个列表，可以进行分片
```

>>> # Show the best hit's stored fields
>>> results[0]
{"title": u"Hello World in Python", "path": u"/a/b/c"}
>>> results[0:2]
[{"title": u"Hello World in Python", "path": u"/a/b/c"},
{"title": u"Foo", "path": u"/bar"}]

```
**判断结果长度：**
```

>>> len(results)
>>> results.scored_length()
if results.has_exact_length():
    print("Scored", found, "of exactly", len(results), "documents")
else:
    low = results.estimated_min_length()
    high = results.estimated_length()
    print("Scored", found, "of between", low, "and", high, "documents")

```
**打分和排序：**
whoosh.scoring模块包含很多打分算法的实现，默认为BM25F.你可以通过weighting参数改变。
```

from whoosh import scoring

with myindex.searcher(weighting=scoring.TF_IDF()) as s:
    ...

```
###  过滤Results 
通过filter参数可以设置**允许**出现在结果集的项结果。
通过mask参数可以设置**不允许**出现在结果集的项结果。
```

with myindex.searcher() as s:
    qp = qparser.QueryParser("content", myindex.schema)
    user_q = qp.parse(query_string)

    # Only show documents in the "rendering" chapter
    allow_q = query.Term("chapter", "rendering")
    # Don't show any documents where the "tag" field contains "todo"
    restrict_q = query.Term("tag", "todo")

    results = s.search(user_q, filter=allow_q, mask=restrict_q)

```
**使用 results.filtered_count (or resultspage.results.filtered_count)来获取被过滤的文档数目**
```

with myindex.searcher() as s:
    qp = qparser.QueryParser("content", myindex.schema)
    user_q = qp.parse(query_string)
 # Filter documents older than 7 days
    old_q = query.DateRange("created", None, datetime.now() - timedelta(days=7))
    results = s.search(user_q, mask=old_q)

    print("Filtered out %d older documents" % results.filtered_count)

```
Which terms from my query matched?
设置terms=True参数来从whoosh.searching.Results and whoosh.searching.Hit获取这些信息
```

with myindex.searcher() as s:
    results = s.seach(myquery, terms=True)
# Was this results object created with terms=True?
if results.has_matched_terms():
    # What terms matched in the results?
    print(results.matched_terms())

    # What terms matched in each hit?
    for hit in results:
        print(hit.matched_terms())

```
###  Collapsing results 
Whether a document should be collapsed is determined by the value of a “collapse facet”. If a document has an empty collapse key, it will never be collapsed, but otherwise** only the top N documents with the same collapse key will appear in the results. **   
```

with myindex.searcher() as s:
    # Set the facet to collapse on and the maximum number of documents per
    # facet value (default is 1)
    results = s.collector(collapse="hostname", collapse_limit=3)

    # Dictionary mapping collapse keys to the number of documents that
    # were filtered out by collapsing on that key
    print(results.collapsed_counts)

```
```

from whoosh import sorting

with myindex.searcher() as s:
    price_facet = sorting.FieldFacet("price", reverse=True)
    type_facet = sorting.FieldFacet("type")
    rating_facet = sorting.FieldFacet("rating", reverse=True)

    results = s.collector(sortedby=price_facet,  # Sort by reverse price
                          collapse=type_facet,  # Collapse on product type
                          collapse_order=rating_facet  # Collapse to highest rated
                          )

```
###  Time limited searches 
Time limited searches时间限制的检索
```

from whoosh.collectors import TimeLimitCollector, TimeLimit

with myindex.searcher() as s:
    # Get a collector object
    c = s.collector(limit=None, sortedby="title_exact")
    # Wrap it in a TimeLimitedCollector and set the time limit to 10 seconds
    tlc = TimeLimitedCollector(c, timelimit=10.0)

    # Try searching
    try:
        s.search_with_collector(myquery, tlc)
    except TimeLimit:
        print("Search took too long, aborting!")

    # You can still get partial results from the collector
    results = tlc.results()

```
###  非分词field的方便检索方法 
```

>>> list(searcher.documents(indexeddate=u"20051225"))
[{"title": u"Christmas presents"}, {"title": u"Turkey dinner report"}]
>>> print searcher.document(path=u"/a/b/c")
{"title": "Document C"}

```
###  组合多个检索结果 
```

# Parse the user query
userquery = queryparser.parse(querystring)

# Get the terms searched for
termset = set()
userquery.existing_terms(termset)

# Formulate a "best bet" query for the terms the user
# searched for in the "content" field
bbq = Or([Term("bestbet", text) for fieldname, text
          in termset if fieldname == "content"])

# Find documents matching the searched for terms
results = s.search(bbq, limit=5)

# Find documents that match the original query
allresults = s.search(userquery, limit=10)

# Add the user query results on to the end of the "best bet"
# results. If documents appear in both result sets, push them
# to the top of the combined results.
results.upgrade_and_extend(allresults)

```
有以下用于组合结果的方法
  * Results.extend(results)，将参数results的文档添加到当前results的末尾
  * Results.filter(results)，将参数results中存在的文档从当前results移除
  * Results.upgrade(results) 将参数results和当前results中都存在的文档移至结果列表的最前面
  * Results.upgrade_and_extend(results)
##  解析查询QueryParser 
默认查询语言：http://whoosh.readthedocs.io/en/latest/querylang.html
whoosh.qparser与whoosh.query模块
查询字符串rendering shading会被解析为如下：
And([Term("content", u"rendering"), Term("content", u"shading")])

whoosh.qparser.QueryParser,whoosh.qparser.MultifieldParser(), whoosh.qparser.SimpleParser() or whoosh.qparser.DisMaxParser()


```

from whoosh.qparser import QueryParser
parser = QueryParser("content", schema=myindex.schema)
#如果是测试，可以直接使用>>> parser.parse(u"alpha OR beta gamma")

```

默认情况下空格分隔的查询字符串，会解析为AND，如果想改变这一行为，比如改成OR，(让physically based rendering解析为physically OR based OR rendering)
```

parser = qparser.QueryParser(fieldname, schema=myindex.schema,
                             group=qparser.OrGroup)
og = qparser.OrGroup.factory(0.9)
parser = qparser.QueryParser(fieldname, schema, group=og)

```
###  多域field检索 
 whoosh.qparser.MultifieldParser
```

from whoosh.qparser import MultifieldParser

mparser = MultifieldParser(["title", "content"], schema=myschema)

```
比如three blind mice会被解析为
(title:three OR content:three) (title:blind OR content:blind) (title:mice OR content:mice）

###  查询语言定制 
精简/添加/替换
  * whoosh.qparser.QueryParser.add_plugin()
  *  whoosh.qparser.QueryParser.remove_plugin_class()
  *  whoosh.qparser.QueryParser.replace_plugin()
```

parser.remove_plugin_class(qparser.FieldsPlugin)
parser.remove_plugin_class(qparser.WildcardPlugin)

```

**Changing the AND, OR, ANDNOT, ANDMAYBE, and NOT syntax：**
whoosh.qparser.OperatorsPlugin 
```

# Use Spanish equivalents instead of AND and OR
op = qparser.OperatorsPlugin(And=" Y ", Or=" O ")
parser.replace_plugin(op)

```

**Adding less-than, greater-than, etc.：**
使用 >, <, >=, < =, = >, or =<
field:{apple to]
field:>apple

date:>='31 march 2001'
date:[31 march 2001 to]

**Adding fuzzy term queries**
语法使用~n,比如cat~,cat~2,johannson~2/3,johannson~/3
```

parser = qparser.QueryParser("fieldname", my_index.schema)
parser.add_plugin(qparser.FuzzyTermPlugin())

```

**Allowing complex phrase queries:whoosh.qparser.SequencePlugin略。**

**Creating custom operators**：PrefixOperator, PostfixOperator, or InfixOperator

