title: lucene4学习笔记9 

#  Lucene4.x正则查询 
今天要分享的是关于lucene中另外一种丰富的查询方式-正则查询，lucene内置了许多的查询API，以及更强大的自定义查询方式的QueryParse，大部分情况下我们使用内置的查询API，基本上就可以满足我们的需求了，但是如果你想更灵活的定制自己的查询或者改写自己的查询API那么你完全可以继承QueryParse类来完成这项工作。 
从某种方式上来说，**正则查询（RegexpQuery）**跟通配符查询（WildcardQuery）的功能很相似，因为他们都可以完成一样的工作，但是不同的是正则查询支持更灵活定制细化查询，这一点与通配符的泛化是不一样的，而且正则查询天生支持使用强大的正则表达式的来准确匹配一个或几个term，需要注意的是，**使用正则查询的字段最好是不分词的，因为分词的字段可能会导致边界问题，从而使查询失败**，得不到任何结果，这一点和WildcardQuery效果是一样的。 
下面先来看一下，散仙的测试数据，为了看出分词与不分词给查询造成的影响，散仙的用同样的内容做测试，分词工具使用的是IK的分词器，截图如下： 
![](/data/dokuwiki/lucene/pasted/20160303-095112.png)
在上图中，散仙使用2个字段存储一样的内容，一个是分过词的，一个没分过词的，下面给出使用正则查询的核心代码：
```

egexpQuery query=new RegexpQuery(new Term(field, ".*"+searchStr+".*"));
                  // System.out.println(query.toString());
                  TopDocs s=search.search(query,null, 100);
                //  TopDocs s=search.search(bool,null, 100);
                   System.out.println(s.totalHits);
                  for(ScoreDoc ss:s.scoreDocs){
                          
                         Document docs=search.doc(ss.doc);
                         System.out.println("id=>"+docs.get("id")+"   name==>"+docs.get("bookName")+"   author==>"+docs.get("author"));
                    // System.out.println(docs.get(field));
                     }

```
下面我们先来测，对不分词的字段的做模糊查询，测试的代码如下：
 dao.testRegQuery("bookName","并发");
结果如下： 
命中数据 :2
id=>2   name==>并发数据挑战面临巨大的挑战   author==>并发数据挑战面临巨大的挑战
id=>4   name==>我们的并发数量并秦东亮在不不是很大   author==>我们的并发数量并秦东亮在不不是很大
我们发现它很出色完成了模糊的查询，并且耗时比通配符查询同样的查询条件的耗时要少，

下面我们对分词的字段，进行检索，测试代码如下： 
 dao.testRegQuery("author","并发");
结果如下：
命中数据 :3
id=>2   name==>并发数据挑战面临巨大的挑战   author==>并发数据挑战面临巨大的挑战
id=>3   name==>the food is perfect!   author==>我们的并发数量并不是很大
id=>4   name==>我们的并发数量并秦东亮在不不是很大   author==>我们的并发数量并秦东亮在不不是很大

我们发现对分词字段的模糊匹配，也同样没问题，下面**我们来测下对分词字段的边界查询**。代码如下： 
```

 dao.testRegQuery("bookName","e q");
         dao.testRegQuery("bookName","量并"); //如果进行分词，那么分词后量和并被分配在不同的Tearm中
         System.out.println("# ## 对比界限==");
         dao.testRegQuery("author","e q");
         dao.testRegQuery("author","量并");

```
结果如下：
```

命中数据 :1
id=>1   name==>the quick brown fox jumps over the lazy dog   author==>the quick brown fox jumps over the lazy dog
命中数据 :1
id=>4   name==>我们的并发数量并秦东亮在不不是很大   author==>我们的并发数量并秦东亮在不不是很大
# ## 对比界限==
命中数据 :0
命中数据 :0

```
由以上结果，我们可以发现分词后的字段，如果在某个字之间被切分成两个term，那么无论你用什么样的方式模糊这**两个term边界**之间的数据，都查询不到任何结果，**而不分词的字段，却能查出来，这是因为，不分词的字段都是作为一个单独的term来处理的，来lucene的内部匹配方式，恰恰又是以term作为最小检索单位的，故能检索到结果，这一点需要我们格外注意**，在实现我们的业务时，要根据自己的场景来设计出最优的分词策略。
下面散仙要测的是正则查询的老本行了，**使用正则表达式进行查询**，代码如下：
```

     dao.testRegQuery("bookName","[fb]ox");//利用正则式检索

```
结果如下:
命中数据 :2
id=>1   name==>the quick brown fox jumps over the lazy dog   author==>the quick brown fox jumps over the lazy dog
id=>5   name==>log is small box   author==>log is small box
我们发现含有fox，box的两条数据都被正确的检索出来了，其实检索的条件，在匹配时会被分解成4个条件，分别是，fox，fo，box，bo只要含有这几个term的数据，都会被检索出来，而这一点恰恰省去了，我们在使用其他的查询时使用OR或者AND进行拼接的繁琐，也可以简化成所谓的SQL里面的IN查询，当然使用正则表达式查询方式可以有很多种，在这里只是简单的举了个例子，有兴趣的朋友们，可以自己测测。 

最后在总结一下，
1，**如果是在` 不分词的字段 `里做模糊检索，优先使用正则查询的方式会比其他的模糊方式性能要快**。
2，**在查询的时候，应该` 注意分词字段的边界问题 `**。
3，**在使用OR或AND拼接条件查询时或一些特别复杂的匹配时，也应优先使用正则查询。**
4，大数据检索时，性能尤为重要，**` 注意应避免使用前置模糊的方式 `，无论是正则查询还是通配符查询。** 


参考：http://my.oschina.net/MrMichael/blog/220956