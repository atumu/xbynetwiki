title: luceneinaction_第2版_学习笔记8 

#  LuceneInAction2rd学习笔记之高亮显示查询项 
 高亮显示模块需要两个独立的输入：完整的原始文本以用来提供操作数据，以及来源于该文本的一个` TokenStream `。为了创建TokenStream，你必须对文本进行重新分析，此时需要使用与索引期间相同的分析器进行。
Highlighter依赖于词汇单元流中每个词汇单元的起始和结束位置偏移量来将原始输入文本中的字符片段进行精确定位，来用于高亮显示。
高亮处理两个部分：文本拆分、高亮显示
高亮处理的相关概念：
##  Fragmenter 
作用是将原始字符串拆分成独立的**片段**。
NullFragmenter 是该接口的一个具体实现类，**它将整个字符串作为单个片段返回，这适合于处理title域和前台文本较短的域**，而对于这些域来说，我们是希望在搜索结果中全部展示。
SimpleFragmenter 是负责**将文本拆分封固定字符长度的片段**，但它并处理子边界。**你可以指定每个片段的字符长度（默认情况100**）但这类片段有点过于简单，在创建片段时，他并不限制查询语句的位置，因此对于跨度的匹配操作会轻易被拆分到两个片段中；
SimpleSpanFragmenter 是尝试将**让片段永远包含跨度匹配的文档。**

**如果不Highlighter实例中设置Fragmenter，那么它会默认的SimpleFragmenter。**

##  Scorer 
**Fragmenter输出的是文本片段序列，而Highlighter必须从中挑选出最适合的一个或多个片段呈现给客户**，为了做到这点，Highlighter会要求Java接口**Scorer来对每个片段进行评分**。Highlighter提供了两个Scorer具体实现类：QueryTermScorer和QueryScorer
QueryTermScorer 基于片段中对应Query的项数进行评分。
QueryScorer只对促成文档匹配的实际项进行评分。
 
**QueryScorer和SimpleSpanFragmenter联合使用通常是最好的选择**
##  Encoder 
他的目的很简单，将初始文本编码成外部格式。该接口实现有两个：
DefaultEncoder：默认情况下供Hightlighter使用，它并不对文本进行任何操作。
SimpleHTMLEncoder：负责将文本编码成HTML，并忽略一些如< 、>以及其它非ASCII等特殊字符。一旦完成编码，最后一步就是对片段进行格式化处理向用户展现。
 
##  Formatter 
它负责将片段转换成String形式，以及将被高亮显示的项一起用于搜索结果展示以及高亮显示。常用` SimpleHTMLFormatter `
   
3.x版本:
```

public class HighlightIt {
  private static final String text =
    "In this section we'll show you how to make the simplest " +
    "programmatic query, searching for a single term, and then " +
    "we'll see how to use QueryParser to accept textual queries. " +
    "In the sections that follow, we’ll take this simple example " +
    "further by detailing all the query types built into Lucene. " +
    "We begin with the simplest search of all: searching for all " +
    "documents that contain a single term.";

  public static void main(String[] args) throws Exception {
    String filename = "d:\\11111.html";
    String searchText = "term";                               // #1
    QueryParser parser = new QueryParser(Version.LUCENE_30,      // #1
                                         "f",                         // #1
                                         new StandardAnalyzer(Version.LUCENE_30));// #1
    Query query = parser.parse(searchText);                           // #1

    SimpleHTMLFormatter formatter =                                   // #2
      new SimpleHTMLFormatter("<span class=\"highlight\">",           // #2
                              "</span>");                             // #2

    TokenStream tokens = new StandardAnalyzer(Version.LUCENE_30)  // #3
        .tokenStream("f", new StringReader(text));                    // #3

    QueryScorer scorer = new QueryScorer(query, "f");                 // #4

    Highlighter highlighter = new Highlighter(formatter, scorer);     // #5
    highlighter.setTextFragmenter(                                    // #6
                  new SimpleSpanFragmenter(scorer));                  // #6

    String result =                                                   // #7
        highlighter.getBestFragments(tokens, text, 3, "...");         // #7

    FileWriter writer = new FileWriter(filename);                     // #8
    writer.write("<html>");                                           // #8
    writer.write("<style>\n" +                                        // #8
        ".highlight {\n" +                                            // #8
        " background: yellow;\n" +                                    // #8
        "}\n" +                                                       // #8
        "</style>");                                                  // #8
    writer.write("<body>");                                           // #8
    writer.write(result);                                             // #8
    writer.write("</body></html>");                                   // #8
    writer.close();                                                   // #8
  }
}

/*
#1 Create the query
#2 Customize surrounding tags
#3 Tokenize text
#4 Create QueryScorer
#5 Create highlighter
#6 Use SimpleSpanFragmenter to fragment
#7 Highlight best 3 fragments
#8 Write highlighted HTML
*/

```
**4.x版本**
```

//... Above, create documents with two fields, one with term vectors (tv) and one without (notv)
  IndexSearcher searcher = new IndexSearcher(directory);
  QueryParser parser = new QueryParser("notv", analyzer);
  Query query = parser.parse("million");

  TopDocs hits = searcher.search(query, 10);

  SimpleHTMLFormatter htmlFormatter = new SimpleHTMLFormatter();
  Highlighter highlighter = new Highlighter(htmlFormatter, new QueryScorer(query));
  for (int i = 0; i < 10; i++) {
    int id = hits.scoreDocs[i].doc;
    Document doc = searcher.doc(id);
    String text = doc.get("notv");
    TokenStream tokenStream = TokenSources.getAnyTokenStream(searcher.getIndexReader(), id, "notv", analyzer);
    TextFragment[] frag = highlighter.getBestTextFragments(tokenStream, text, false, 10);//highlighter.getBestFragments(tokenStream, text, 3, "...");
    for (int j = 0; j < frag.length; j++) {
      if ((frag[j] != null) && (frag[j].getScore() > 0)) {
        System.out.println((frag[j].toString()));
      }
    }
    //Term vector
    text = doc.get("tv");
    tokenStream = TokenSources.getAnyTokenStream(searcher.getIndexReader(), hits.scoreDocs[i].doc, "tv", analyzer);
    frag = highlighter.getBestTextFragments(tokenStream, text, false, 10);
    for (int j = 0; j < frag.length; j++) {
      if ((frag[j] != null) && (frag[j].getScore() > 0)) {
        System.out.println((frag[j].toString()));
      }
    }
    System.out.println("-------------");
  }

```

示例：
```

public void search(String fieldName, String keyword)throws CorruptIndexException, IOException, ParseException {  
    searcher = new IndexSearcher(indexPath);  
    QueryParser queryParse = new QueryParser(fieldName, analyzer); // 构造QueryParser，解析用户输入的检索关键字  
    Query query = queryParse.parse(keyword);  
    Hits hits = searcher.search(query);  
    for (int i = 0; i < hits.length(); i++) {  
        Document doc = hits.doc(i);  
        String text = doc.get(fieldName);  
        SimpleHTMLFormatter simpleHTMLFormatter = new SimpleHTMLFormatter("<font color='red'>", "</font>");  
        Highlighter highlighter = new Highlighter(simpleHTMLFormatter, new QueryScorer(query));  
        highlighter.setTextFragmenter(new SimpleFragmenter(text.length()));  
        if (text != null) {  
            TokenStream tokenStream = analyzer.tokenStream(fieldName,new StringReader(text));  
            String highLightText = highlighter.getBestFragment(tokenStream,text);  
            System.out.println("高亮显示第 " + (i + 1) + " 条检索结果如下所示：");  
            System.out.println(highLightText);  
        }  
    }  
    searcher.close();  
}  

```
参考：
http://qindongliang.iteye.com/blog/1953409
http://www.cnblogs.com/zhwl/p/3484804.html
http://blog.csdn.net/seven_zhao/article/details/42709171
