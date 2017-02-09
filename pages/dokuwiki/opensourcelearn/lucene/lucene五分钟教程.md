title: lucene五分钟教程 

#  Lucene五分钟教程 
官方文档：http://lucene.apache.org/core/4_6_0/core/overview-summary.html
下面的代码使用Lucene 4.0版本！
Lucene大大简化了在应用中集成全文搜索的功能。但实际上Lucene十分简单，我可以在五分钟之内向你展示如何使用Lucene。
```

<properties>
  <lucene.version>4.10.4</lucene.version>
</properties>
<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-core</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-grouping</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-codecs</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-analyzers-common</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-highlighter</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-join</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-memory</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-misc</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-queries</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-queryparser</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-suggest</artifactId>
			<version>${lucene.version}</version>
		</dependency>


```
##  1. 建立索引 
为了简单起见，我们下面为一些字符串创建内存索引：
```

StandardAnalyzer analyzer = new StandardAnalyzer(Version.LUCENE_40);
Directory indexDir = new RAMDirectory();
IndexWriterConfig config = new IndexWriterConfig(Version.LUCENE_40, analyzer);
IndexWriter w = new IndexWriter(indexDir, config);
addDoc(w, "Lucene in Action", "193398817");
addDoc(w, "Lucene for Dummies", "55320055Z");
addDoc(w, "Managing Gigabytes", "55063554A");
addDoc(w, "The Art of Computer Science", "9900333X");
w.close();

//addDoc()方法把文档（译者注：这里的文档是Lucene中的Document类的实例）添加到索引中。
private static void addDoc(IndexWriter w, String title, String isbn) throws IOException {
  Document doc = new Document();
  doc.add(new TextField("title", title, Field.Store.YES));
  doc.add(new StringField("isbn", isbn, Field.Store.YES));
  w.addDocument(doc);
}

```
注意，对于需要分词的内容我们使用TextField，对于像id这样不需要分词的内容我们使用StringField。

##  2.搜索请求 
我们从标准输入（stdin）中读入搜索请求，然后对它进行解析，最后创建一个Lucene中的` Query `对象。
```

String querystr = args.length > 0 ? args[0] : "lucene";
Query q = new QueryParser(Version.LUCENE_40, "title", analyzer).parse(querystr);

```

##  3.搜索 
我们创建一个` Searcher `对象并且使用上面创建的` Query `对象来进行搜索，匹配到的前10个结果封装在` TopScoreDocCollector `对象里返回。
```

int hitsPerPage = 10;
IndexReader reader = IndexReader.open(indexDir);
IndexSearcher searcher = new IndexSearcher(reader);
TopScoreDocCollector collector = TopScoreDocCollector.create(hitsPerPage, true);
searcher.search(q, collector);
ScoreDoc[] hits = collector.topDocs().scoreDocs;

```
##  4.展示 
现在我们得到了搜索结果，我们需要想用户展示它。
```

System.out.println("Found " + hits.length + " hits.");
for(int i=0;i<hits.length;++i) {
    int docId = hits[i].doc;
    Document d = searcher.doc(docId);
    System.out.println((i + 1) + ". " + d.get("isbn") + "\t" + d.get("title"));
}

```
这里是这个小应用的完整代码
```

import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.queryparser.classic.ParseException;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopScoreDocCollector;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.RAMDirectory;
import org.apache.lucene.util.Version;
 
import java.io.IOException;
 
public class HelloLucene {
  public static void main(String[] args) throws IOException, ParseException {
    // 0. Specify the analyzer for tokenizing text.
    //    The same analyzer should be used for indexing and searching
    StandardAnalyzer analyzer = new StandardAnalyzer(Version.LUCENE_40);
 
    // 1. create the index
    Directory index = new RAMDirectory();
 
    IndexWriterConfig config = new IndexWriterConfig(Version.LUCENE_40, analyzer);
 
    IndexWriter w = new IndexWriter(index, config);
    addDoc(w, "Lucene in Action", "193398817");
    addDoc(w, "Lucene for Dummies", "55320055Z");
    addDoc(w, "Managing Gigabytes", "55063554A");
    addDoc(w, "The Art of Computer Science", "9900333X");
    w.close();
 
    // 2. query
    String querystr = args.length > 0 ? args[0] : "lucene";
 
    // the "title" arg specifies the default field to use
    // when no field is explicitly specified in the query.
    Query q = new QueryParser(Version.LUCENE_40, "title", analyzer).parse(querystr);
 
    // 3. search
    int hitsPerPage = 10;
    IndexReader reader = DirectoryReader.open(index);
    IndexSearcher searcher = new IndexSearcher(reader);
    TopScoreDocCollector collector = TopScoreDocCollector.create(hitsPerPage, true);
    searcher.search(q, collector);
    ScoreDoc[] hits = collector.topDocs().scoreDocs;
     
    // 4. display results
    System.out.println("Found " + hits.length + " hits.");
    for(int i=0;i<hits.length;++i) {
      int docId = hits[i].doc;
      Document d = searcher.doc(docId);
      System.out.println((i + 1) + ". " + d.get("isbn") + "\t" + d.get("title"));
    }
 
    // reader can only be closed when there
    // is no need to access the documents any more.
    reader.close();
  }
 
  private static void addDoc(IndexWriter w, String title, String isbn) throws IOException {
    Document doc = new Document();
    doc.add(new TextField("title", title, Field.Store.YES));
 
    // use a string field for isbn because we don't want it tokenized
    doc.add(new StringField("isbn", isbn, Field.Store.YES));
    w.addDocument(doc);
  }
}

```
可以直接在命令行中使用这个小应用，键入java HelloLucene 。

##  Lucene索引的【增、删、改、查】 
总结以下创建索引的过程：
1 创建Directory，获取索引目录
　　2 创建词法分析器，创建IndexWriter对象
　　3 创建document对象，存储数据
　　4 关闭IndexWriter，提交
```

/**
     * 建立索引
     * 
     * @param args
     */
    public static void index() throws Exception {
        
        String text1 = "hello,man!";
        String text2 = "goodbye,man!";
        String text3 = "hello,woman!";
        String text4 = "goodbye,woman!";
        
        Date date1 = new Date();
        analyzer = new StandardAnalyzer(Version.LUCENE_CURRENT);
        directory = FSDirectory.open(new File(INDEX_DIR));

        IndexWriterConfig config = new IndexWriterConfig(
                Version.LUCENE_CURRENT, analyzer);
        indexWriter = new IndexWriter(directory, config);

        Document doc1 = new Document();
        doc1.add(new TextField("filename", "text1", Store.YES));
        doc1.add(new TextField("content", text1, Store.YES));
        indexWriter.addDocument(doc1);
        
        Document doc2 = new Document();
        doc2.add(new TextField("filename", "text2", Store.YES));
        doc2.add(new TextField("content", text2, Store.YES));
        indexWriter.addDocument(doc2);
        
        Document doc3 = new Document();
        doc3.add(new TextField("filename", "text3", Store.YES));
        doc3.add(new TextField("content", text3, Store.YES));
        indexWriter.addDocument(doc3);
        
        Document doc4 = new Document();
        doc4.add(new TextField("filename", "text4", Store.YES));
        doc4.add(new TextField("content", text4, Store.YES));
        indexWriter.addDocument(doc4);
        
        indexWriter.commit();
        indexWriter.close();

        Date date2 = new Date();
        System.out.println("创建索引耗时：" + (date2.getTime() - date1.getTime()) + "ms\n");
    }

```
增量添加索引
　　Lucene拥有增量添加索引的功能，在不会影响之前的索引情况下，添加索引，它会在何时的时机，自动合并索引文件。
```

/**
     * 增加索引
     * 
     * @throws Exception
     */
    public static void insert() throws Exception {
        String text5 = "hello,goodbye,man,woman";
        Date date1 = new Date();
        analyzer = new StandardAnalyzer(Version.LUCENE_CURRENT);
        directory = FSDirectory.open(new File(INDEX_DIR));

        IndexWriterConfig config = new IndexWriterConfig(
                Version.LUCENE_CURRENT, analyzer);
        indexWriter = new IndexWriter(directory, config);

        Document doc1 = new Document();
        doc1.add(new TextField("filename", "text5", Store.YES));
        doc1.add(new TextField("content", text5, Store.YES));
        indexWriter.addDocument(doc1);

        indexWriter.commit();
        indexWriter.close();

        Date date2 = new Date();
        System.out.println("增加索引耗时：" + (date2.getTime() - date1.getTime()) + "ms\n");
    }

```
删除索引
　　Lucene也是通过IndexWriter调用它的delete方法，来删除索引。我们可以通过关键字，删除与这个关键字有关的所有内容。如果仅仅是想要删除一个文档，那么最好就顶一个唯一的ID域，通过这个ID域，来进行删除操作。
```

/**
     * 删除索引
     * 
     * @param str 删除的关键字
     * @throws Exception
     */
    public static void delete(String str) throws Exception {
        Date date1 = new Date();
        analyzer = new StandardAnalyzer(Version.LUCENE_CURRENT);
        directory = FSDirectory.open(new File(INDEX_DIR));

        IndexWriterConfig config = new IndexWriterConfig(
                Version.LUCENE_CURRENT, analyzer);
        indexWriter = new IndexWriter(directory, config);
        
        indexWriter.deleteDocuments(new Term("filename",str));  
        
        indexWriter.close();
        
        Date date2 = new Date();
        System.out.println("删除索引耗时：" + (date2.getTime() - date1.getTime()) + "ms\n");
    }

```
更新索引
　　Lucene没有真正的更新操作，通过某个fieldname，可以更新这个域对应的索引，但是实质上，它是先删除索引，再重新建立的。
```

/**
     * 更新索引
     * 
     * @throws Exception
     */
    public static void update() throws Exception {
        String text1 = "update,hello,man!";
        Date date1 = new Date();
         analyzer = new StandardAnalyzer(Version.LUCENE_CURRENT);
         directory = FSDirectory.open(new File(INDEX_DIR));

         IndexWriterConfig config = new IndexWriterConfig(
                 Version.LUCENE_CURRENT, analyzer);
         indexWriter = new IndexWriter(directory, config);
         
         Document doc1 = new Document();
        doc1.add(new TextField("filename", "text1", Store.YES));
        doc1.add(new TextField("content", text1, Store.YES));
        
        indexWriter.updateDocument(new Term("filename","text1"), doc1);
        
         indexWriter.close();
         
         Date date2 = new Date();
         System.out.println("更新索引耗时：" + (date2.getTime() - date1.getTime()) + "ms\n");
    }

```
通过索引查询关键字
　　Lucene的查询方式有很多种，这里就不做详细介绍了。它会返回一个ScoreDoc的集合，类似ResultSet的集合，我们可以通过域名获取想要获取的内容。
```

/**
     * 关键字查询
     * 
     * @param str
     * @throws Exception
     */
    public static void search(String str) throws Exception {
        directory = FSDirectory.open(new File(INDEX_DIR));
        analyzer = new StandardAnalyzer(Version.LUCENE_CURRENT);
        DirectoryReader ireader = DirectoryReader.open(directory);
        IndexSearcher isearcher = new IndexSearcher(ireader);

        QueryParser parser = new QueryParser(Version.LUCENE_CURRENT, "content",analyzer);
        Query query = parser.parse(str);

        ScoreDoc[] hits = isearcher.search(query, null, 1000).scoreDocs;
        for (int i = 0; i < hits.length; i++) {
            Document hitDoc = isearcher.doc(hits[i].doc);
            System.out.println(hitDoc.get("filename"));
            System.out.println(hitDoc.get("content"));
        }
        ireader.close();
        directory.close();
    }

```
参考：http://www.importnew.com/12715.html
http://www.lucenetutorial.com/
http://www.cnblogs.com/xing901022/p/3940243.html