title: luceneinaction_第2版_学习笔记5 

#  LuceneInAction(第2版)学习笔记5之Lucene基本扩展 
##  Luke:Lucene的索引查询工具箱 
github:https://github.com/DmitryKey/luke
基本使用教程:
http://www.656463.com/article/juI7vm.htm

##  RegexQuery 
```

public void testRegexQuery() throws Exception {
    Directory directory = TestUtil.getBookIndexDirectory();
    IndexSearcher searcher = new IndexSearcher(directory);
    RegexpQuery q = new RegexpQuery(new Term("title", ".*st.*"));
    TopDocs hits = searcher.search(q, 10);
    assertEquals(2, hits.totalHits);
    assertTrue(TestUtil.hitsIncludeTitle(searcher, hits,
                                         "Tapestry in Action"));
    assertTrue(TestUtil.hitsIncludeTitle(searcher, hits,
                                         "Mindstorms: Children, Computers, And Powerful Ideas"));
    searcher.close();
    directory.close();
  }

```

##  在结果中查询 
2.1.两种方案
**在检索结果中再次进行检索，是一个很常见的需求**，一般有两种方案可以选择：
①使用` QueryFilter `把第一个查询当作一个过滤器处理；
②用` BooleanQuery `把前后两个查询结合起来，并且使用` BooleanClause.Occur.MUST `。
针对第一种方法，我需要解释一下。QueryFilter在Lucene的2.x版本中是存在的，但是在3.x中，lucene的API中这个类已经被废弃了，无法再找到。如果你的项目使用的是lucene是3.x，但是你又一定要使用QueryFilter，那么你必须自己创建一个QueryFilter类，然后将2.x中QueryFilter的源代码复制过来。

**BooleanQuery方案** 
使用BooleanQuery来实现在结果中检索的过程是这样的，首先通过关键字keyword1正常检索，当用户需要在检索结果中再通过关键字keyword2检索的时候，通过构建BooleanQuery，来实现对在结果中检索的效果。这里要注意，这两个关键字都要使用BooleanClause.Occur.MUST。
```

//创建BooleanQuery  
BooleanQuery booleanQuery = new BooleanQuery();  
//多域检索，在这四个域中检索  
String[] fields = { "phoneType", "name", "category","free" };  
Query multiFieldQuery = new MultiFieldQueryParser(Version.LUCENE_36, fields, analyzer).parse(keyword);  
//将multiFieldQuery添加到BooleanQuery中  
booleanQuery.add(multiFieldQuery, BooleanClause.Occur.MUST);  
//如果osKeyword不为空  
if(osKeyword != null && !osKeyword.equals("") && !osKeyword.equals("null")){  
    TermQuery osQuery = new TermQuery(new Term("phoneType",osKeyword));   
    //将osQuery添加到BooleanQuery中  
    booleanQuery.add(osQuery, BooleanClause.Occur.MUST);  
}  

```
##  针对多个域的一次性查询 
1.1.三种方案    
使用lucene构造搜索引擎的时候，如果要针对多个域进行一次性查询，一般来说有三种方法：
第一种实现方法是创建多值的全包含域的文本进行索引，这个方案最简单。但是这个防范有个缺点：你不能直接对每个域的加权进行控制。
第二种方法是使用` MultiFieldQueryParser `，它是QueryParser的子类，它会在后台程序中实例化一个QueryParser对象，用来针对每个域进行查询表达式的解析，然后使用BooleanQuery将查询结果合并起来。当程序向BooleanQuery添加查询子句时，默认操作符OR被用于最简单的解析方法中。为了实现更好的控制，布尔操作符可以使用BooleanClause的常量指定给每个域。如果需要指定的话可以使用BooleanClause.Occur.MUST，如果禁止指定可以使用BooleanClause.Occur.MUST_NOT，或者普通情况为BooleanClause.Occur.SHOULD。下面的程序展示的是如何创建MultiFieldQueryParser类的方法：
```

// 在这四个域中检索  
String[] fields = { "phoneType", "name", "category", "price" };  
Query query = new MultiFieldQueryParser(Version.LUCENE_36, fields, analyzer).parse(keyword); 

``` 
第三种方法就是使用高级DisjunctionMaxQuery类，它会封装一个或者多个任意的查询，将匹配的文档进行OR操作。
1.2.方案选择
以上三种方案中，并不是第三种方案最好，也不是第一种方案就最差。哪种实现方式更适合你的应用程序呢？答案是“看情况”，因为这里存在一些取舍。全包含域是一个简单的解决方案——但这个方案只能对搜索结果进行简单的排序并且可能浪费磁盘空间（程序可能对同样的文本索引两次），但这个方案可能会获得最好的搜索性能。
MultiFieldQueryParser生成的BooleanQuery会计算所有查询所匹配的文档评分的总和（DisjunctionMaxQuery则只选取最大评分），然后它能够实现针对每个域的加权。你必须对以上3中解决方案都进行测试，同时需要一起考虑搜索性能和搜索相关性，然后再找出最佳方案。

参考：http://www.cnblogs.com/zhwl/p/3484804.html