title: indexsearcher中检索方法 

#  IndexSearcher中检索方法 

原文：http://blog.csdn.net/xiaojimanman/article/details/43052829
查看Lucene4.3.1中IndexSearcher的API请点击这里，关于搜索的方法如下图：
![](/data/dokuwiki/opensourcelearn/lucene/pasted/20150513-030134.png)
从上图几个方法可以看出，有几个重点类需要介绍下：Query（上篇博客已介绍）、Filter、Sort、ScoreDoc、Collector
#  IndexSearcher API介绍 

Collector
Collector主要用来对搜索结果做收集、自定义排序、过滤等，在Lucene4.3.1的API中，有两个搜索方法用到了Collector，但是其下面都有一句Lower-level search API（低级别的搜索API），如果没有非用不可的需求，尽量还是使用其他方法。

Filter
Filter主要是做筛选条件的，用于指定哪些文档可以在搜索结果中，这个自己使用的并不是太多，查询了一些资料，介绍说有Filter的检索过程是先对数据源做筛选预处理（Filter中指定的），然后将筛选的结果交给查询语句，如果是这样的话，使用Filter的代价将会很大，他的查询耗时可能会提高数倍。个人认为也没有必要使用Filter，如果真的需要对结果做筛选，可以把这些筛选条件合并到Query中，而没有必要创建一个Filter对象。

Sort
Sort在检索方法中指定排序方式，相当于数据库中的order by，创建方式如Sort sort = new Sort(new SortField("time", Type.LONG, true))，这里的SortField构造方法中的三个参数分别代表域名、域数据类型、排序方式（true降序/false升序），这里的例子只是按照一个域进行排序，如果多个域可以直接在构造方法中添加，如sort = new Sort(new SortField("time", Type.LONG, true), new SortField("star", Type.INT, false))。

ScoreDoc
从上图方法中，searchAfter方法中使用到ScoreDoc，该方法主要用在分页查询中，当然也可以用search方法替代，但是有一种情况是无法替代的，比如查询第一页10条数据，但由于推广或广告等需求，需要在其中添加几条（具体未知）其他记录，但是前端只能展示10条数据，这样该页的最后几条数据就没有办法显示，但在下一页中又想显示这几条数据，这样使用seach方法实现就有点困难，事例需求图形描述如下：
![](/data/dokuwiki/opensourcelearn/lucene/pasted/20150513-030354.png)
searchAfter在实现上述的需求时，在取下一页数据时，只需要将上次查询的最后一个ScoreDoc告诉它即可，它可以直接从该条数据开始查询下一页的数据。
#  IndexWriter删改操作 
```

//更改
public boolean updateDocument(Term term, Document doc){
	try {
		indexWriter.updateDocument(term, doc);
		return true;
	} catch (IOException e) {
		e.printStackTrace();
		return false;
	}
}
//删除
public boolean deleteDocument(Query query){
	try {
		indexWriter.deleteDocuments(query);
		return true;
	} catch (IOException e) {
		e.printStackTrace();
		return false;
	}
}
//删除All
public boolean deleteAll(){
	try {
		indexWriter.deleteAll();
		return true;
	} catch (IOException e) {
		e.printStackTrace();
		return false;
	}
}

```
<note>` 上述的这些操作，都需要执行indexWriter.commit()之后才会保存，否则是不会有效的。 `</note>
