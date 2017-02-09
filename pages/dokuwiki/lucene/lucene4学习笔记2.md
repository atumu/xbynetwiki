title: lucene4学习笔记2 

#  Lucene4.x索引与搜索 
在java工程中创建两个文件夹用来分别放原文件和索引文件（content、index）在content文件夹 中放一些txt文件，完成后的工程图：
![](/data/dokuwiki/lucene/pasted/20160302-104441.png)
第三步：创建一个java类命名为ReaderAndWriterIndex.java并创建两个方法，createindex()用来创建索引，searcher()用来查询，
代码如下：
```

import org.apache.lucene.analysis.standard.StandardAnalyzer; 
import org.apache.lucene.document.Document; 
import org.apache.lucene.document.Field; 
import org.apache.lucene.document.FieldType; 
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.DirectoryReader; 
import org.apache.lucene.index.FieldInfo.IndexOptions;
import org.apache.lucene.index.IndexReader; 
import org.apache.lucene.index.IndexWriter; 
import org.apache.lucene.index.IndexWriterConfig; 
import org.apache.lucene.queryparser.classic.ParseException; 
import org.apache.lucene.queryparser.classic.QueryParser; 
import org.apache.lucene.search.IndexSearcher; 
import org.apache.lucene.search.Query; 
import org.apache.lucene.search.ScoreDoc; 
import org.apache.lucene.search.TopDocs; 
import org.apache.lucene.store.Directory; 
import org.apache.lucene.store.FSDirectory; 
import org.apache.lucene.util.Version;
public class ReaderAndWriterIndex { 
 public static void createindex(){ 
  IndexWriter writer = null;
        try { 
            //创建Directory对象有多种方式，使用FSDirectory.open()创建它会选择最优的方式获得 
            //Directory对象，Directory实现类包括SimpleFSDirectory和RAMDirectory两个类 
            //其中RAMDirectory是在内存中创建索引对象 
            //1、创建Directory对象来保存索引，index为刚才创建的index目录 
            Directory dir = FSDirectory.open(new File("index"));
            //2、创建IndexWriter对象需要IndexWriterConfig和Analyzer对象，这里使用标准分词器 
            IndexWriterConfig config = new IndexWriterConfig(Version.LUCENE_47, new StandardAnalyzer(Version.LUCENE_47)); 
            writer = new IndexWriter(dir, config);
            //3、创建Document对象 
            Document doc = null;
            File file = new File("content"); 
            //创建一个索引存储分词的FieldType 
            //在这里FieldType对象只是为了说明，下面并没有使用。 
            FieldType index_store_tokeniz = new FieldType(); 
            index_store_tokeniz.setIndexed(true); 
            index_store_tokeniz.setIndexOptions(IndexOptions.DOCS_ONLY); 
            index_store_tokeniz.setStored(true); 
            index_store_tokeniz.setTokenized(true); 
            index_store_tokeniz.freeze();
            for(File f : file.listFiles()){ 
                //4、根据索引内容创建Field对象并添加到Document中 
                doc = new Document();
                //为Documnet对象添加需要索引或保存的合适的Field对象 
                //TextField是一个索引分词可以存储的Field实现
                doc.add(new TextField("filename", f.getName(), Field.Store.YES));
                //StringField是一个索引可以存储但不分词的Field实现 
                doc.add(new StringField("filepath",f.getPath(),Field.Store.YES));
                //TextField是一个索引分词可以存储的Field实现
                //使用filename+Reader的构造是不存储的 
                doc.add(new TextField("content",new FileReader(f)));
                //6、把Document对象添加到IndexWriter对象中 
                writer.addDocument(doc); 
             }
        } catch (IOException e) { 
            e.printStackTrace(); 
        } finally{ 
            try { 
                if(writer != null){ 
                    //6、关闭IndexWriter对象 
                    writer.close(); 
                } 
            } catch (IOException e) { 
                e.printStackTrace(); 
            } 
        } 
    }
    public static void searcher(){ 
        try { 
         //1、创建Directory 
            Directory dir = FSDirectory.open(new File("index"));
            //2、创建IndexReader 
            IndexReader reader = DirectoryReader.open(dir); 
            //3、根据IndexReader创建IndexSearcher 
            IndexSearcher searcher = new IndexSearcher(reader); 
            //4、创建QueryParser 
            QueryParser parser = new QueryParser(Version.LUCENE_47, "content", new StandardAnalyzer(Version.LUCENE_47)); 
            //5、根据QueryParser创建Query,并传递需要查询的词 
            Query query = parser.parse("Lucene"); 
            //6、根据query使用IndexSearcher创建TopDocs 
            TopDocs tds = searcher.search(query, 10); 
            //7、通过TopDocs获得ScoreDoc对象 
            ScoreDoc[] sds = tds.scoreDocs; 
            for(ScoreDoc sd : sds){ 
                //8、通过IndexSearcher和ScoreDoc.doc获得Document对象 
                Document d = searcher.doc(sd.doc); 
                //9、关键Document对象获得值 
                System.out.println(d.get("filename") + "===" + d.get("filepath")); 
            } 
            //10、关闭IndexReader 
            reader.close(); 
        } catch (IOException e) { 
             e.printStackTrace(); 
        } catch (ParseException e) { 
             e.printStackTrace(); 
        } 
    } 
}

```
第四步：为工程添加JUnit并编写测试类测试上面的两个方法：
```

import org.junit.Test;
public class IndexReaderMethodTest { 
    /** 
     * 测试创建索引方法 
     * */ 
    @Test  
    public void testIndex(){ 
        ReaderAndWriterIndex.createindex(); 
    } 
    /** 
     * 测试查询索引方法 
     * */ 
    @Test  
    public void testReader(){ 
        ReaderAndWriterIndex.searcher(); 
    } 
}

```
参考：http://my.oschina.net/MrMichael/blog/220684