title: lucene4学习笔记4 

#  Lucene4.x排序 
排序是对于全文检索来言是一个必不可少的功能。
在默认情况下，Lucene使用的是以关联性降序的方式为默认的排序方式，这样可以使得我们搜索的结果通常是最优的，因为它会尽可能的使得首先出现的几个结果是与我们搜索的内容最相关，而不需要我们翻页寻找我们最想要的内容，这一点是与数据库相比，是全文检索一个很大的优点。**当然，在实际开发中我们也需要根据业务的实际情况来给我们的客户提供多种不同的排序方式。**我们先来看下在Lucene中比较特殊的两种基本的排序方式 
Sort里的属性	SortField里的属性	含义
Sort.INDEXORDER	SortField.FIELD_DOC	按照索引的顺序进行排序
Sort.RELEVANCE	SortField.FIELD_SCORE	按照关联性评分进行排序

我们再来看几个检索时需要用的方法
```

# #### SortField类===
//field是排序字段type是排序类型
public SortField(String field, Type type);
//field是排序字段type是排序类型reverse是指定升序还是降序
//reverse 为true是降序  false为升序
  public SortField(String field, Type type, boolean reverse)
 
  # #### Sort类===
  public Sort();//Sort对象构造方法默认是按文档评分排序
  public Sort(SortField field);//排序的一个SortField
  public Sort(SortField... fields)//排序的多个SortField可以传入一个数组
  
  # ===IndexSearche类r==
//query是查询的Query对象 filter是过滤  n返回的数量  sort是排序
search(Query query, Filter filter, int n, Sort sort) 
//doDocScores 为true情况下每个命中的结果下都会被评分
//doMaxScore  为true情况下对最大分值的搜索结果进行评分
search(Query query, Filter filter, int n, Sort sort, boolean doDocScores, boolean doMaxScore)


```
1，在还没有进行一点排序前我们先来看下索引里的内容，核心代码如下: 
```

TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),10000);

```
![](/data/dokuwiki/lucene/pasted/20160302-110329.png)

2，使用默认的关联性评分后,核心代码和运行效果图如下: 
```

 Sort sort=new Sort();//默认使用关联性评分
TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),10000,sort);

```
![](/data/dokuwiki/lucene/pasted/20160302-110400.png)
关于上图中乱码字符原因是因为**默认排序情况下lucene是不会对搜索结果进行评分操作的，因为评分操作会降低性能，所以关于score的那一列返回的是NAN的字符串，**出于格式的需要，散仙在用DecimalFormat类给其评分结果保留2位小数时，因为是一个特殊字符，所以就出现了上图情况。 

3，按照日期降序排序，,核心代码和运行效果图如下: 
```

 Sort sort=new Sort(new SortField("date", Type.INT,true));//true为降序排列
             TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),10000,sort);

```
![](/data/dokuwiki/lucene/pasted/20160302-110517.png)

4，按照价格升序排序，,核心代码和运行效果图如下: 
```

 Sort sort=new Sort(new SortField("price", Type.DOUBLE,false));//false为降序排列
             TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),10000,sort);

```

5，**多字段排序**，按照日期降序的情况下，因为id为7和8的日期相同，所以我们就新增一个排序字段按ename升序排列，,核心代码和运行效果图如下: 
```

 // Sort sort=new Sort(new SortField("date", Type.INT, true),new SortField("ename", Type.STRING, false));
            //这两段代码效果一样
            Sort sort=new Sort(new SortField[]{new SortField("date", Type.INT, true),new SortField("ename", Type.STRING, false)});
             TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),10000,sort);

```
![](/data/dokuwiki/lucene/pasted/20160302-110647.png)

6，**带评分的排序，注意后面两个布尔类型的变量可以控制是否评分，特别是在没有要求需要打分时，建议别开启，大数量时对性能影响较大**，检索“编程”得到的结果,默认按评分降序排序，核心代码和运行效果图如下: 
```

 Sort sort=Sort.RELEVANCE;
TopDocs topDocs=searcher.search(new TermQuery(new Term("bookname", "编程")),null,100,sort,true,true);

```
![](/data/dokuwiki/lucene/pasted/20160302-110756.png)

7，注意几点 
（1）排序对一个文档里什么域都没存储，使用字符串排序会排在首位 
（2）排序对一个文档里什么域都没存储，使用数字类型排序会默认给其赋值为0进行排序 
（3）我们可以对数字类型的null值的文档进行代码控制，可以将其设置为最大，所以将会排在最后面，代码如下 
```

 SortField sortField = new SortField("value", SortField.Type.INT);
    sortField.setMissingValue(Integer.MAX_VALUE);

```


参考：http://my.oschina.net/MrMichael/blog/220773