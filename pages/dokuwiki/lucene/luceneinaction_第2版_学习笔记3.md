title: luceneinaction_第2版_学习笔记3 

#  LuceneInAction(第2版)学习笔记3之搜索 
##  1. 实现简单的搜索功能 
1.1 **对特定项的搜索TermQuery**： 编程实现查询语句
 这种搜索程序员有最终解释权，可以提供灵活的UI
` IndexSearcher `类是用于对索引中文档进行搜索的核心类，它有多个重要的重载方法。
```

 public static Directory getBookIndexDirectory() throws IOException{
  return FSDirectory.open(new File(System.getProperty("index.dir"))); 
  // index.dir的值在build.xml 的ant脚本中，默认值为 "build/index"
 }
 public void testTerm() throws Exception{
  Directory dir = TestUtil.getBookIndexDirectory(); //获取路径信息
  IndexSearcher searcher = new IndexSearcher(dir);  //创建IndexSearcher类
  //最好只创建一个单一的searcher来完成所有查询
  //打开一个新的searcher花费的代价很高，因为需要加载和构建索引的内部数据结构
  Term t = new Term("subject", "ant");
  Query query = new TermQuery(t);
  TopDocs docs = searcher.search(query, 10);
  //docs.totalHits //查到的ant结果数量
  //docs.ScoreDocs对象 //命中结果对象
  searcher.close(); //先关闭searcher
  dir.close();  //才能关闭目录对象
  }

```

**1.2. 解析用户输入的查询表达式： QueryParser类**
QueryParser类将用户输入的文本转换成Query对象
这种搜索更易于使用，且提供了所有用户都熟悉的标准搜索语法。Lucene的搜索方法，需要一个Query对象作为参数。对查询表达式的解析，实际上就是将用户输入的诸如”mock OR junit"的查询表达式，转换成对应的Query实例的过程。Expression ==> QueryParser(Analyzer) ==> (Query Object) ==> IndexSearcher
```

public void testQueryParser() throws Exception{
  Directory dir = TestUtil.getBookIndexDirectory(); //获取路径信息
  IndexSearcher searcher = new IndexSearcher(dir);  //创建IndexSearcher类
  //最好只创建一个单一的searcher来完成所有查询
  //打开一个新的searcher花费的代价很高，因为需要加载和构建索引的内部数据结构
  //新的建立QueryParser对象
  QueryParser parser = new QueryParser(Version.LUCENE_30, "contents", new SimpleAnalyzer());
  Query query = parser.parse("+JUNIT + ANT -MOCK");//JUNIT AND ANT NOT　MOCK
  TopDocs docs = searcher.search(query, 10);
  //docs.totalHits //查到的ant结果数量
  //docs.ScoreDocs对象 //命中结果对象
  searcher.close(); //先关闭searcher
  dir.close();  //才能关闭目录对象
  }

```
 Lucene有一个引人注目的内置特性，就是可以通过QueryParser类对查询表达式进行解析。 它可以把诸如"+JUNIT +ANT -MOCK"和"mock OR junit"这样的两个查询条件解析成一个Query类。
 QueryParser类最主要的目标就是处理人们输入的查询表达式，返回Query对象。QueryParser类需要使用一个分析器把查询语句分割成多个项，它也是搜索过程中用到分析器的唯一类。
 
(1) QueryParser类的使用
QueryParser parser = new QueryParser( Version matchVersion, String field,Analyzer analyzer)
matchVersion参数指示Lucene用它进行默认版本进行操作，目的是为了向后兼容；
field参数是指所有被搜索项所对应的**默认域**
analyzer参数指示分析器对象

public Query parse(String query) throws ParseException名为query的String对象，就是要被解析的表达式。
如果表达式解析不成功，Lucene会抛出一个ParseException异常，程序是可能处理该异常的。
ParseException消息会给出解析失败的合理提示，该提示对普通用户来说可能太过专业了。
Lucene还有很多用于控制QueryParser实例的高级设置。

 
(2)
 ###  用QueryParser处理基本查询表达式 
 查询表达式    匹配文档
```

 java     默认域包含java项的文档
 java junit 或 java OR junit  默认域包含java和junit中一个或两个的文档
 +java +junit 或 java AND junit  默认域中同时包含java和junit的文档
 title:ant    title域中包含ant
 title:extreme -subject:sports  title域中包含extreme且subject域中不包含sports的文档，相当于title:extreme And NOT subject:sports
 (agile OR extreme) AND methodogy 默认域中包含methodology且包含agile和extreme中的一个或两个的文档
 title:"junit in action"   title域为junit in action的文档
 title:"junit action" ~5  title域中junit和action之间距离小于5的文档
 java*     包含由java开头的项的文档，如javaspaces,javaserver,java.net和java本身
 java~     包含与单词java相近的项的文档，如lava
 lastmodified:[1/1/09 TO 12/31/09] lastmodified域值在2009年1月1号和2009年12月31号之间的文档

```

##  2. 使用IndexSearcher类 
2.1. 创建IndexSearcher类
需要一个用于索引的目录
 Directory dir = FSDirectory.open(new File("/path/to/index"));
 1) 通过reader创建searcher实例
```

IndexReader reader = IndexReader.open(dir); //打开reader需要较大的系统开销
IndexSearcher searcher = new IndexSearcher(reader);

```
4.x版本方式为:
创建IndexReader是通过调用DirectoryReader.open()方法
```

IndexReader reader = DirectoryReader.open(FSDirectory.open(new File(indexStorePath)));

```
**创建IndexReader需要很大的开销，因此最好在整个搜索中都重复使用一个IndexReader，只有在必要的时候才创建新的IndexReader**
在创建IndexReader时，它会搜索已有的索引快照，如果你需要搜索索引中的变更信息，那么必须打开一个新的reader。
**DirectoryReader.openIfChanged(DirectoryReader)方法可以判断如果索引信息有变更，可以在耗费较少系统资源的情况下重新创建IndexReader。**

 2) 直接创建searcher实例(这种情况下，系统会在后台立自己私有的reader对象)
IndexSearcher seracher = new IndexSearcher(dir);
采用这个方案，可以通过调用  IndexSearcher 的` getIndexReader `来检索自己的IndexReader。如果此时关闭searcher，则也会关闭自己的reader，因为reader是由searcher打开的。
如果需要搜索索引中的变更信息，必须打开一个新的reader。当然，使用` reader.reopen() `方法也可以获取新的IndexReader对象。
```

  IndexReader newReader = reader.reopen();
  if(reader != newReader){ 
   reader.close();//注意必须关闭旧的reader
   reader=newReader;
   searcher = new IndexSearcher(reader);//使用新的reader重建search
  }

```
**如果索引有所变更，reopen()方法会返回一个新的reader，这时程序必须关闭旧的reader，并创建新的IndexSearcher。**
在实际的应用程序中，**可能有多个线程还在使用旧的reader进行搜索，所以，须要保证上述代码是线程安全的。**
` IndexSearcher实例只能对其初始化时刻所对应的索引进行搜索。如果索引操作是由多线程完成的，则新编入索引的文档不能被该searcher看到。若要看到这些新文档，则必须打开一个新的reader。 `
 
2.2. 实现搜索功能
 一旦获取IndexSearcher实例，就可以调用search()方法来进行搜索。在程序后台，search()方法会快速完成大量工作。它会访问所有候选的搜索匹配文档，并只返回符合每个查询约束条件的结果。
 最后，它会收集最靠前的几个搜索结果并返回给调用程序。
  *  IndexSearcher.search()方法    使用时刻
  *  TopDocs search(Query query, int n)   直接进行搜索，n参数表示返回的评分最高的文档数量
  *  TopDocs search(Query query, Filter filter, int n) 搜索受文档子集约束，约束条件基于过滤策略
  *  TopFieldDocs search(Query query,    搜索受文档子集约束，约束条件基于过滤策略，排序由sort完成
Filter filter, int n, Sort sort)
  *  void search(Query query, Collector results)  当使用自定义文档访问策略时使用，或不想以默认的前N个搜索结果排序策略收集结果时使用
  *  void search(Query query,     同上，区别在于结果文档只有在传入过滤策略时才能被接收
Filter filter, Collector results) 
 搜索结果是**通过相关性进行排序的，**或者说是通过每个结果文档与查询条件的匹配程度进行排序的。

2.3. 使用TopDocs类 
 TopDocs.totalHits属性，返回匹配文档数量；
 TopDocs.scoreDocs属性，返回包含搜索结果的ScoreDoc对象数组；**每个ScoreDoc实例都有一个浮点类型的评分**，该评分是相关性评分；**该实例还包含一个整形的文档号**，它用于标识文档ID，能够用于检索文档中保存的域。具体通过IndexSearcher.doc(doc)方法进行。
 TopDocs.getMaxScore()方法，返回所有匹配文档的最大评分。
 
2.4. 搜索结果分页 
 1) 一次性取所有符合条件的结果，由程序分页.将首次搜索获得的多页搜索结果收集起来，并保存在ScoreDocs和IndexSearcher实例中，并在用户换页浏览时展现这几页结果。

 2) 按页搜索，返回特定页的文档.每次用户换页浏览时，都要重新进行查询操作.说明： **重新查询通常是更好的解决方案，它可以不用存储每个用户的当前浏览状态，**
但对Web应用来说开销太大，尤其是有巨大用户群的应用程序。不过，Lucene的快速处理能力正好可以弥补这个缺陷。此外，现在的操作系统的I/O缓存机制，使得重新查询操作通常会很快完成。
不需要过早使用缓存或存储来对页面导航进行优化，因为，直接按页搜索的方式，可能已经满足你的需求了。
 
 2.5. 近实时搜索：writer.getReader() 
 Lucene2.9的近实时搜索，可以使用一个打开的IndexWriter快速搜索索引的变更内容，而不必首先关闭writer或向该writer提交。近实时搜索，可以对新创建但还未完成提交的段进行搜索。
```

pubic void testNearRealTime() throws Exception {
  Directory dir = new RAMDirectory();
  IndexWriter writer = new IndexWriter(dir, new StandardAnalyzer(Version.LUCENE_30),
       IndexWriter.MaxFiledLength.UNLIMITED);
  for(int i =0; i<10; i++){
   Document doc = new Document();
   doc.add(new Field("id", ""+i, Field.Store.NO,
      Field.Index.NOT_ANALYZED_NO_NORMS));
   doc.add(new Field("text", "aaa", Field.Store.NO, Field.Index.ANALYZED));
   writer.addDocument(doc);
  }
  IndexReader reader = writer.getReader(); //创建近实时reader
  IndexSearcher searcher = new IndexSearcher(reader);//将reader封装至 IndexSearcher
  Query query = new TermQuery(new Term("text", "aaa"));
  TopDocs docs = searcher.search(query, 1);
  //docs.totalHits; //搜索结果
  writer.deleteDocuments(new Term("id", "7"));//删除一个文档
  Document doc = new Document(); //添加一个文档
  doc.add(new Field("id", "11", Field.Store.NO, Field.Index.NOT_ANALYZED_NO_NORMS));
  doc.add(new Field("text", "bbb", FIeld.Store.NO, Field.Index.ANALYZED));
  writer.addDocument(doc);
  IndexReader newReader = reader.reopen();//重新打开一个reader
  //assertFalse(reader == newReader); //确认reader是新建的
  reader.close();   //关闭旧的reader
  search = new IndexSearcher(newReader); 
  TopDocs hits = searcher.search(query, 10);
  //hits.totalHits; //搜索结果
  query = new TermQuery(new Term("text", "bbb"));
  hits = searcher.search(query, 1);
  //hits.totalHits;
  newReader.close();
  writer.close();
 }

```
**IndexWriter.getReader()，它会将缓存中的所有变更刷新到索引目录中，然后创建一个新的包含这些变更的IndexReader。**
如果想通过writer完成更多变更，则可以使用` reader.reopen() `来获取新的reader。**reopen()方法效率非常高：** 对于索引中任何未发生变更的部分来说，新的reader会共享这些部分中打开的文件，并能缓存先前的reader。
只有上次 open() 或 reopen() 操作以来重新创建的文件才会在此轮被打开。这种方式速度很快，翻转时间不到一秒。
 
##  3. Lucene的评分机制 
 每当搜索到匹配文档时，该文档都会被赋予一定的分值，用以反映匹配程度。该分值会计算文档与查询语句之间的相似程度，更高的分值反映了更强的相似程度和匹配程度。
 
3.1. Lucene如何评分
 Lucene相似度评分公式，用来衡量查询语句和对应匹配文档之间的相似程度。
 计算方式为： 查询语句q中每个项t与文档d的匹配分值之和。
 评分归一化处理： 用该查询对应的评分除以最大评分。评分越大，说明文档和查询之间的匹配越好。
 评分因子   描述
 ------------------------------------------------------------------------------------------
  *  tf(t in d)   项频率因子：文档d中出现t的频率
  *  idf(t)    项在倒排文档中出现的频率：衡量项的唯一性.出现多则idf低，出现少则idf高
  *  boost(t.field in d)  域和文档的加权，在索引期间设置.可以用该方法对某个域或文档进行静态单独加权
  *  lengthNorm(t.field in d) 域的归一化值，表示域中包含的项数量，该值在索引期间计算，并保存在norm中
  *  coord(q, d)   协调因子，基于文档中包含查询的项个数
  *  queryNorm(q)   每个查询的归一值，指每个查询项权重的平方和

 在这个评分公式中，**大多数因子的控制和实现都是通过Similarity抽象类的子类完成的。**
 如果没有指定其它实现Similarity的类，则Lucene会使用` DefaultSimilarity `类。
 
3.2. 使用` explain() `理解搜索结果评分
 ```

IndexSearcher类的explain()方法，该方法要传入一个Query对象和一个文档ID，然后返回一个Explanation对象。
 Explanation对象内部包含了所有关于评分计算中各个因子的信息细节。
 Explanation explanation = searcher.explain(query, match.doc); //生成Explanation对象
 Document doc = searcher.doc(match.doc);
 //println(doc.get("title"));
 //println(explanation.toString());//输出Explanation对象
 //explanation.toHtml();

```
 实际上，Explanation是Nutch项目的核心内容，它允许透明排序。
 通过Explanation对象，可以方便看到评分计算的内部运作，但它需要的开销是和做查询操作一样的。
 因此，不要过多使用Explanation对象。
 
##  4. Lucene的多样化查询 
###  4.1. 通过项进行搜索: TermQuery 类【特定项查询】 
 对索引中特定项进行搜索是最基本的搜索方式。
 **Term是最小的索引片段**，每个Term包含了一个域名和一个文本值。
```

 Term t = new Term("contents", "java"); //域名和域值，其中，域值区分大小写
 Query query = new TermQuery(t);

```
 
###  4.2. 在指定的项范围内搜索：TermRangeQuery 类 【特定项范围查询】 
 索引中的各个Term对象会按照字典编排顺序进行排序，允许在Lucene的TermRangeQuery对象提供的范围内进行文本项的直接搜索，搜索时包含或不包含起始项和终止项，如果某个项为空，则意昧着没有边界。
 ```

public void testTermRangeQuery() throws Exception{
  Directory dir = TestUtil.getBookIndexDirectory();
  IndexSearcher searvher = new IndexSearcher(dir);
  TermRangeQuery query = new TermRangeQuery("title2", "d", "j", true, true);//域名、起始域值、结束域值、是否包含起始域值、是否包含结束域值
  TopDocs matches = searcher.search(query, 100);
  /matches.totalHits;
  searcher.close();
  dir.close();
 }

```
 
###  4.3. 在指定的数字范围内搜索：NumericRangeQuery 类【数值项范围查询 
 如果使用` NumericField `对象来索引域，则可以使用NumericRangeQuery在某个特定范围内搜索该域。
 ```

public void testInclusive() throws Exception{
  Directory dir = TestUtil.getBookIndexDirectory();
  IndexSearcher searvher = new IndexSearcher(dir);
  NumericRangeQuery query = new NumericRangeQuery("pubmonth", 200605, 200609, true, true);//域名、起始域值、结束域值、是否包含起始域值、是否包含结束域值
  TopDocs matches = searcher.search(query, 100);
  /matches.totalHits;
  searcher.close();
  dir.close();
 }

```

###  4.4. 通过字符串搜索：PrefixQuery 类  【前缀匹配查询】 
 PrefixQuery 用来搜索包含以指定字符串开头的项的文档。
 ```

public void testPrefix() throws Exception{
  Directory dir = TestUtil.getBookIndexDirectory();
  IndexSearcher searvher = new IndexSearcher(dir);
  
  Term term = new Term("category", "technology/computers/programming");
  TopDocs matches = searcher.search(new PrefixQuery(term), 100); //包含子类
  //matches.totalHits;
   matches = searcher.search(new TermQuery(term), 100); //不包括子类
  //matches.totalHits;
  searcher.close();
  dir.close();
 }

```

###  4.5. 组合查询：BooleanQuery 类【查询子句组合】 
```

 public void testAnd() throws Exception{
  //条件1: 负责查找subject域中包含"search"的所有书
  TermQuery searchingBooks = new TermQuery(new Term("subject", "search"));
  //条件2：负责查找2010年出版的所有书
  NumericRangeQuery books2010 = NumericRangeQuery.newIntRange("pubmonth", 
      201001, 201012, true,true);
  //组合条件: 将各查询子句加入到BooleanQuery对象中
  // 将两个查询合并为一个布尔查询，其中，两个查询子句都是必须的
  BooleanQuery searchingBooks2010 = new BooleanQuery();
  searchingBooks2010.add(searchingBooks, BooleanClause.Occur.MUST); 
  searchingBooks2010.add(books2010, BooleanClause.Occur.MUST); 
  //BooleanClause.Occur.MUST 只有匹配的文档才在考虑之列  AND
  //BooleanClause.Occur.SHOULD 可选的 OR
  //BooleanClause.Occur.MUST_NOT  不会包含匹配该查询的文档，相当于NOT 
  Directory dir = TestUtil.getBookIndexDirectory();
  IndexSearcher searvher = new IndexSearcher(dir);
  
  TopDocs matches = searcher.search(searchingBooks2010, 100);
  //matches.totalHits;
  searcher.close();
  dir.close();
 }

```
两个都为 **` BooleanClause.Occur `**.SHOULD ，才是或 OR。
 BooleanQuery对其中包含的查询子句是有数量限制的，**默认情况下允许包含1024个查询子句**。 这个限制主要是为了防止对应用程序的性能造成影响。可以通过`  ClauseCount(int) `方法修改**查询子句的数量限制。**
 
###  4.6. 通过短语搜索： PhraseQuery类 【关键字位置查询】 
 索引时的 ` omitTermFreqAndPositions ` 选项： 表示**各个 term 之间的顺序和位置。**
 而 PhraseQuery 类，可以根据这些位置信息定位某个距离范围内的项所对应的文档。如查询quick紧邻fox (quick fox)的 或者两者之间只有一个单词 (quick [..] fox)的文档。
在匹配的情况下，**两个项的位置之间所允许的` 最大 `间隔距离称为slop**。这时的距离是指，项若要按顺序组成给定的短语**所需要移动位置的次数**。
如果slop为0，则表示要求查询结果必须和输入的短语完全匹配。
 内容： the quick brown fox jumped over the lazy dog
 搜索A:  quick fox slop=1 成功, 即： fox 向后移1个位置就匹配 quick ... fox
 搜索B:  fox quick slop=3  成功，即： fox 向后移3个位置才匹配 quick ... fox
 
 1) 复合项短语 multiple-term phrases
PhraseQuery 支持复合项短语.无论短语中有多少个项，**slop因子都规定了按顺序移动项位置的总次数的最大值。**
 内容： the quick brown fox jumped over the lazy dog
 搜索A:  quick jump lazy  slop=4 成功, 即： jump 向后移3个位置就匹配 quick ... fox,lazy 向后移4个位置才匹配 jumped ... lazy
 搜索B:  lazy jump quick    slop=8 成功, 即： jump 向后移4个位置就匹配 quick ... jump,lazy 向后移8个位置就匹配 quick ... jump ... lazy

 2) 短语查询评分
短语查询是根据短语匹配所需要的编辑距离来进行评分的。**项之间的距离越小，具有的权重就越大。短语查询的评分因子=1/(distance+1)**。评分与距离成反比关系，距离越大的匹配，其评分就越低。
 
 3) 在QueryParser中的短语查询
在QueryParse的分析表达式中，**双引号里面的若干个项被转换成一个 PhraseQuery 对象。slop因子的默认值为0，但也可以在QueryPhrase的查询表达式中加上~n声明，**以便调整slop因子的值。
如： "quick fox" ~3 的意思是，为fox和quick生成一个slop因子为3的PhraseQuery对象。
 
###  4.7. 通配符查询:  WildcardQuery类 
通配符查询可以让我们使用不完整的、缺少某些字项的项进行查询，仍然可以查到相关的匹配结果。
Lucene使用两个标准的通配符：** *代表0个或者多个字母，?代表0个或者1个字母。**
可以将 WildcardQuery类看做一个更通用的PrefixQuery类，因为通配符是没有两端的。
```

Query query = new WildcardQuery(new Term("contents", "?ild*"));
TopDocs matches = searcher.search(query, 10);

```
 一个Term实例是一个便利的占位符，它代表了一个域名和一个任意字符串。
注意：  **当使用通配符查询时，可能会降低系统性能。较长的前缀可以减少用于查找匹配搜枚举的个数。**如果是以通配符打头，则会强制枚举所有索引中的项，以便用于搜索匹配。
 
###  4.8. 搜索类似项：FuzzyQuery类- 模糊查询 
Lucene的**模糊查询 FuzzyQuery 类，用于匹配与指定项相似的项**。
Levenshtein距离算法用来决定索引文件中的项与指定目标项的相似程序。这种算法又称为编辑距离算法：它是两个字符串之间相似度的一个度量方法。编辑距离是用来计算从一个字符串到另一个字符串所需的最少插入、删除和替换的字母个数。
```

 public void testFuzzy() throws Exception{
  indexSingFieldDocs(new Field[]{
     new Field("contents", "fuzzy", Field.Store.YES, Field.Index.ANALYZED),
     new Field("contents", "wuzzy", Field.Store.YES, Field.Index.ANALYZED)
     });
  IndexSearcher searcher = new IndexSearcher(directory);
  Query query = new FuzzyQuery(new Term("contents", "wuzza"));
   //FuzzyQuery还有两个构造函数，来限制模糊匹配的程度 
// 在FuzzyQuery中，默认的匹配度是0.5，当这个值越小时，通过模糊查找出的文档的匹配程度就 
// 越低，查出的文档量就越多,反之亦然 
FuzzyQuery query1 = new FuzzyQuery(t, 0.1f); 
FuzzyQuery query2 = new FuzzyQuery(t, 0.1f, 1); 
  TopDocs matches = searcher.search(query, 10);
  //matches.totalHits;
  //mathes.scoreDocs[0].score
  Document doc = searcher.doc(matches.socreDocs[0].doc);
  //doc.get("contents");
  searcher.close();
 }

```
 第一个参数当然是词条对象，第二个参数指的是levenshtein算法的最小相似度，第三个参数指的是要有多少个前缀字母完全匹配：

###  4.9. 匹配所有文档： MatchAllDocsQuery类 
 MatchAllDocsQuery，就是匹配索引中所有文档。默认情况下，该类对匹配的文档分配了一个固定的评分，该文档具体的查询加权默认值为1.0。如果将该查询作为顶层查询，则除了使用默认的相关性排序之外，最好通过域来排序。

##  5. 解析查询表达式： QueryParser 
5.1. ` Query.toString() `方法
当查询表达式被QueryParser对象解析后，会产生什么变化呢？要看到这些结果，可以使用Query对象的toString()方法。toString()方法便于可视化调试用构造函数创建的复杂查询，
也是打开QueryParser如何解析查询表达式之门的一把钥匙。**但不要过分依赖Lucene对Query.toString()的输出和QueryParser已解析的表达式进行转换的能力。**由于QueryParser中包含一个分析器，这可能会使情况变得更复杂。
5.2. TermQuery
```

 public void testTermQuery() throws Exception{
  QueryParser parser = new QueryParser(Version.LUCENE_30, "subject", analyzer);
  Query query = parser.parse("computers");
  System.out.println("term: " + query.toString());
 }
 结果为： 
  terms:  subject:computers

```
 很重要的一点，QueryParser使用的分析器与索引期间使用的分析器最好保持一致。

5.3. 项范围查询
 针对文本或日期的范围查询，所采用的是括号形式，并且只需要在**查询范围两端的项之间用TO进行连接即可。`  TO必须都是大写字母 `。
括号的类型：中括号表示包含在内，大括号表示排除在外。**
注意，与NumericRangeQuery或TermRangeQuery不同的是，不能同时进行包含和排除操作。
即： 搜索范围的起点和终点，要么都是包含在内，要么都是排除在外。
```

Query query = new QueryParser(Version.LUCENE_30,"subject", analyzer)
   .parser("title2:[Q TO V]"); //边界在搜索范围内

 Query query = new QueryParser(Version.LUCENE_30,"subject", analyzer)
   .parser("title2:{Q TO \"Tapestry in Action\"}"); //边界在搜索范围外

```
 
5.4. 数值范围搜索和日期范围搜索
 暂无。
 
5.5. 前缀查询和通配符查询
 如果项中包含了一个星号或问号，则该项会被看作是通配符查询对象WildcardQuery。当查询项只在末尾有一个星号时，QueryParser类会净它优化为前缀查询对象PrefixQuery。不管是前缀查询还是通配符查询，其对象都会被转换为小写字母形式。不过，可以控制是否转换。
```

 Query query = new QueryParser(Version.LUCENE_30, "field", analyzer)
   .parser("PrefixQuery*");   
 query.toString("field");//结果为 prefixquery*
 query.setLowercaseExpandedTerms(false); //不转换大小写
 query.toString("field");//结果为 PrefixQuery*

```
 在使用QueryParser类进行通配符查询时，**默认情况是不支持项开端包含通配符的，**若要改变这个默认限制，可以调用 ` setAllowLeadingWildcard `方法，但这个调用会牺牲掉一部分的程序性能。
 
5.6. 布尔操作符
可以使用**AND、OR、NOT**操作符，通过QueryParser建立文本类型的布尔查询。
 注意，**这些布尔操作符必须全部使用大写形式**，列出的项之间，**如果没有指定布尔操作符，则默认使用OR。**可以使用QueryParser对象的` setDefaultOperator() `方法切换隐含的操作符。
 QueryParser parser = new QueryParser(Version.LUCENE_30, "contents", analyzer);
 parser.setDefaultOperator(QueryParser.AND_OPERATOR);
 详细语法  快捷语法
 -----------------------------------------------
 a AND b   +a +b
 a OR b   ab
 a AND NOT b  +a -b
注意：不能单独使用NOT，即不能直接用NOT b, -b必须前面跟一个非否定操作符

5.7. **短语查询 "双引号括"**
**查询语句中用双引号括起来的项，可以用来创建一个 PhraseQuery。**引号之间的文本将被进行分析，作为分析结果，PhraseQuery可能不会跟原始短语一样精确。
  *  1) 双引号内的文本，会促使分析器将它转换为 PhraseQuery。
  *  2) 单项短语(single-term phrase)将被优化成 TermQuery 对象。
```

  Query q = new QueryParser(Version.LUCENE_30, "field", new StandardAnalyzer(Version.LUCENE_30)) .parser("\"This is Some Phrase*\"");
  //q.toString("field"); // "? ? Some phrase"  //即： 停用词对应的位置被问号代替了，默认的slop因子为0。
  q = new QueryParser(Verson.LUCENE_30, "field", analyzer).parser("\"term\"");
  //q instanceof TermQuery // true   //即： 单项短语(single-term phrase)将被优化成 TermQuery 对象。

```
 
 3) 默认的slop因子为0
```

  Query q = new QueryParser(Version.LUCENE_30, "field", analyzer)
   .parse("\"exact phrase\"");
  //q.toString("field");  // "exact phrase" //即： 默认的slop因子为0

```
 
 4) 修改slop因子
```

  QueryParser qp = new QueryParser(Version.LUCENE_30, "field", analyzer);
  qp.setPhraseSlop(5);
  Query q = qp.parse("\"sloppy phrase\"");
  //q.toString("field");  // "sloppy phrase"~5 //即：修改slop因子，波浪号后紧跟因子值

```
 
 
5.8. 模糊查询
** 波浪号(~)会针对正在处理的项来创建模糊的查询(FuzzyQuery)。**注意，**波浪号还可以用于指定松散短语查询。**
双引号表示短语查询，它并不能用于模糊查询。可以选择性地指定一个浮点数，用来表示所需的最小相似程度。
```

 QueryParser parser = QueryParser(Version.LUCENE_30, "subject", analyzer);
 Query query = parser.parse("kountry~");
 //query.toString(); //subject:kountry~0.5
 query = parser.parse("kountry~0.7");
 //query.toString(); //subject:kountry~0.7

```
同样的性能表明，使用通配符WildcardQuery和使用模糊查询是一致的。它们还可以通过自定义方式禁止运行。
 
5.9. MatchAllDocsQuery
 当输入***.***后，QueryParser会生成MatchAllDocsQuery对象。还支持Query子句分组语法，加权子句以及将子句限制在特定域上。
 
5.10. 分组查询
 Lucene的BooleanQuery允许构建复杂的嵌套子句。同样，QueryParser使用分组后的文本类型的查询表达式来支持同样的功能。
**使用小括号来形成子查询**，以建立高组的BooleanQuery。.parse("(agile OR extreme) AND methodology");
 
5.11. 域选择
 默认的域名是在创建QueryParser时提供的，**但查询语句并不公限制在搜索这个默认的域。title:lucene的形式可以将搜索范围限制在title域。**
分组而成的查询子句，可以使用field(a b c)指定域。空格表示OR。
 
5.12. 为子查询设置加权
 在浮点数前面附上一个^符号，可以对查询处理进行加权因子的设置。如: junit^2.0 testing，会将 junit TermQuery的加权系数设置为2.0，并维持 testing 的TermQuery的默认加权系统1.0。可以对任何类型的查询进行加权，包括插入的组。
 
5.13. 是否一定要使用 QueryParser
** QueryParser 可以快速有效地创建强大的查询，但它并不能适合所有的场合。** QueryParser 不能完全提供通过API创建的所有查询类型。
 如果 QueryParser 不能满足你的功能，你可以继承它并进行定制化。**` 当然，可以通过将QueryParser解析后的查询和调用API建立的查询，联合起来作为BooleanQuery中的查询子句，从而获得一个折中效果。 `**
参考：http://blog.csdn.net/liuweitoo/article/details/8137408