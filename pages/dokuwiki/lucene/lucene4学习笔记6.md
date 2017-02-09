title: lucene4学习笔记6 

#  Lucene4.x过滤器Filter 
 先介绍下查询与过滤的区别和联系，**其实查询(各种Query)和过滤(各种Filter)之间非常相似，可以这样说只要用Query能完成的事，用过滤也都可以完成，**它们之间可以相互转换，**最大的区别就是使用过滤返回的结果集不带评分操作，而使用Query返回的结果都是带相关性评分的，**所以当我们**如果有一些跟评分操作没有关系的业务，优先使用Filter操作，将会获取更好的性能，**其实这也是Solr里面的q参数跟fq参数的区别。
下面，开始进入正题，在这之前，老生常谈的先来了解一下Lucene里面有关于Filter的整体知识 
![](/data/dokuwiki/lucene/pasted/20160302-150647.png)
下面，我们来看下具体的在代码里怎么实现，先来看下我们的测试数据 
id        score        bookname    ename        type            price        date
1        1        飘渺之旅        pmzl        小说        52.23        201005        
2        1        三国演义        sgyy        小说        36.13        201207        
3        1        数据库实战        sjksz        技术        77.13        200811        
4        1        编程宝典        bcbd        技术        100.3        200501        
5        1        职场关系论        zcgxl        职场        36.59        200501        
6        1        健康生活        jksh        生活        20.47        200008        
7        1        看清本质        kqbz        社会        10.37        201004        
8        1        编程,编程        bcbc        社会        10.37        201004

**TermRangeFilter** 
```

//使用过滤器   最后一个为true时包含边界部分，为false时不包含边界部分
//倒数第二个为true时，包含查询边界，为false时不包含
TermRangeFilter filter=new TermRangeFilter("ename", new BytesRef("h"), new BytesRef("n"), true, true);
TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),filter,10000);//默认无排序方式

```
输出结果 
6        1        健康生活        jksh        生活        20.47        200008        
7        1        看清本质        kqbz        社会        10.37        201004

**NumericRangeFilter**
```

NumericRangeFilter<Double> filter=NumericRangeFilter.newDoubleRange("price", 10D, 40D, true, false);
  TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),filter,10000);//默认无排序方式

```
2        1        三国演义        sgyy        小说        36.13        201207        
5        1        职场关系论        zcgxl        职场        36.59        200501        
6        1        健康生活        jksh        生活        20.47        200008        
7        1        看清本质        kqbz        社会        10.37        201004        
8        1        编程,编程        bcbc        社会        10.37        201004

**使用缓存范围过滤FieldCacheRangeFilter**
```

 //使用缓存过滤
  Filter filter=FieldCacheRangeFilter.newDoubleRange("price", 20D, 50D, true, true);
  TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),filter,10000);//默认无排序方式

```
**缓存域过滤特定的类别FieldCacheTermsFilter**
```

// 缓存域过滤特定的类别
 Filter filter=new FieldCacheTermsFilter("type", new String[]{"技术","社会"});
 TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),filter,10000);//默认无排序方式

```
输出结果：
3        1        数据库实战        sjksz        技术        77.13        200811        
4        1        编程宝典        bcbd        技术        100.3        200501        
7        1        看清本质        kqbz        社会        10.37        201004        
8        1        编程,编程        bcbc        社会        10.37        201004

**使用QueryWrapperFilter类包装一个Query**
```

 //使用QueryWrapperFilter类包装一个Query
 QueryWrapperFilter  filter=new QueryWrapperFilter(new TermQuery(new Term("type", "技术")));
 TopDocs topDocs=searcher.search(new MatchAllDocsQuery(),filter,10000);//默认无排序方式

```

最后我来看下，如何继承Filter基类，来定制我们自己的filter,自定义的Filter，虽然某些时候，功能很强大灵活，但是有几个缺点，我们的了解1，保证是内容不重复的字段，例如主键，如果重复，默认返回第一个作为结果集显示2，保证不能被分词的内容，如果是分词的字段，则可能会出现一些不正确的结果。

##  自定义Filter类 
```

import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.apache.lucene.index.AtomicReaderContext;
import org.apache.lucene.index.DocsEnum;
import org.apache.lucene.index.Term;
import org.apache.lucene.search.DocIdSet;
import org.apache.lucene.search.Filter;
import org.apache.lucene.util.AttributeSource;
import org.apache.lucene.util.Bits;
import org.apache.lucene.util.DocIdBitSet;
import org.apache.lucene.util.FixedBitSet;
import org.apache.lucene.util.OpenBitSet;
 
/***
 *^_^  ^_^  ^_^
 * QQ交流探讨群:324714439
 * 自定义过滤器
 * @author 三劫散仙
 * */
public class MyCustomFilter extends Filter{
     
    public MyCustomFilter() {
        // TODO Auto-generated constructor stub
    }
     
    private String[] terms;//限制返回的数据字典
    public MyCustomFilter(String ...terms) {
        // TODO Auto-generated constructor stub
        this.terms=terms;
    }
    @Override
    public DocIdSet getDocIdSet(AtomicReaderContext arg0, Bits arg1)
            throws IOException {
        FixedBitSet bits=new FixedBitSet(arg0.reader().maxDoc())  ;//获取没有所有的docid包括未删除的
         int base=arg0.docBase;//段的相对基数，保证多个段时相对位置正确
         //int limit=base+arg0.reader().maxDoc();//计算最大限制值
        for(String s:terms){
              DocsEnum doc=arg0.reader().termDocsEnum(new Term("id", s));//必须是唯一的不重复
              //保证是单个不重复的term,如果重复的话，默认会取第一个作为返回结果集,分词后的term也不适用自定义term
              if(doc.nextDoc()!=-1){ 
                bits.set(doc.docID());//对付符合条件约束的docid循环添加到bits里面
                }
              }
        return bits;
    }
}

``` 
##  FilteredQuery类 
使用FilteredQuery对象能够将任何过滤器转换为具体查询，该功能为你提供了某些可能，**比如将某个过滤器添加到BooleanQuery的查询子句中。**
```

public void testFilteredQuery() throws Exception {
    String[] isbns = new String[] {"9780880105118"};                 // #1

    SpecialsAccessor accessor = new TestSpecialsAccessor(isbns);
    Filter filter = new SpecialsFilter(accessor);

    WildcardQuery educationBooks =                                // #2
      new WildcardQuery(new Term("category", "*education*"));     // #2
    FilteredQuery edBooksOnSpecial =                              // #2
        new FilteredQuery(educationBooks, filter);                // #2

    TermQuery logoBooks =                                         // #3
        new TermQuery(new Term("subject", "logo"));               // #3

    BooleanQuery logoOrEdBooks = new BooleanQuery();                  // #4
    logoOrEdBooks.add(logoBooks, BooleanClause.Occur.SHOULD);         // #4
    logoOrEdBooks.add(edBooksOnSpecial, BooleanClause.Occur.SHOULD);  // #4

    TopDocs hits = searcher.search(logoOrEdBooks, 10);
    System.out.println(logoOrEdBooks.toString());
    assertEquals("Papert and Steiner", 2, hits.totalHits);
  }

```
##  特殊的filter之DuplicateFilte去重 
我们在来看下不在Filter家族中的一个特殊的filter，属于Lucene捐赠模块的特殊包中的类**DuplicateFilter**，这个filter的作用是用来**对某个字段进行去重操作的，类似数据库中的Distinct关键字**，可以实现对某个列的结果集去重，**这个去重的字段，一般情况下是不建议分词的，因为分词后，可能去重效果不准确.**
**举个例子，**来说明分词后去重，会造成什么情况，假如我们的索引name一列中有中国，和伟大的中国，那么就对这个name列去重后，就会发现lucene只保留了伟大的中国这个字段，为什么呢？
因为切词后伟大的中国会被分成伟大|的|中国，进行去重时，Lucene认为中国是重复的，而伟大的中国是不重复的，又因为伟大的中国里包含中国，**所以最后的结果就会只保留伟大的中国，而没有中国。**所以无论使` 用这个过滤器去重，还是使用grouping或fact去重，大多数情况下操作的字段是不能分词的， `这一点需要注意！ 
下面我们来具体看下DuplicateFilter这个特殊的过滤器，
```

 String field="type";
  DuplicateFilter filter=new DuplicateFilter(field);//去重过滤
  Query q=new MatchAllDocsQuery();//对所有结果去重
  TopDocs s=search.search(q, filter, 100);

```
可以看出，核心的代码量很少，却可以高效的完成去重工作，去重技术在我们的实际运用中也是一项很常用的技术，有时候我们可能只需要查看不重复的记录，而没有一些类似统计的功能，如果需要去重并统计个数，那么就需要使用分组功能或分面功能了**,当然，如果我们只需要简单的对字段去重，那么就可以使用DuplicateFilter简洁高效的来完成这项任务。**
参考：http://my.oschina.net/MrMichael/blog/220787