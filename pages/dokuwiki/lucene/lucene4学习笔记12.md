title: lucene4学习笔记12 

#  Lucene4.x整体使用流程介绍 
在lucene对文本进行处理的过程中，可以大致分为三大部分：
1、索引文件：提取文档内容并分析，生成索引
2、搜索内容：搜索索引内容，根据搜索关键字得出搜索结果
3、分析内容：对搜索词汇进行分析，生成Quey对象。
```

<properties>
		<lucene.version>4.10.4</lucene.version>
		<tika.version>1.12</tika.version>
	</properties>
<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-analyzers-common</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-analyzers-smartcn</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-core</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-queryparser</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.lucene</groupId>
			<artifactId>lucene-highlighter</artifactId>
			<version>${lucene.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.tika</groupId>
			<artifactId>tika-core</artifactId>
			<version>${tika.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.tika</groupId>
			<artifactId>tika-parsers</artifactId>
			<version>${tika.version}</version>
		</dependency>

		<dependency>
			<groupId>IKAnalyzer</groupId>
			<artifactId>IKAnalyzer</artifactId>
			<version>2012FF_hf1</version>
		</dependency>

```
##  1、建立索引 
基本上创建索引需要三个步骤：
1、创建索引库IndexWriter对象
2、根据文件创建文档Document
3、向索引库中写入文档内容

我这里使了4个文本文件，分别是
a.txt：人民
b.txt：中华人民共和国
c.txt：人民共和国
d.txt：共和国
下面是建立索引的类：
```

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.io.Reader;

import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.IndexWriterConfig.OpenMode;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import org.apache.lucene.util.Version;

public class LuceneIndex {

	// 索引器
	private IndexWriter writer = null;

	public LuceneIndex(String indexStorePath) {
		try {
			// 索引文件的保存位置
			Directory dir = FSDirectory.open(new File(indexStorePath));
			// 分析器
			Analyzer analyzer = new StandardAnalyzer(Version.LUCENE_4_9);
			// 配置类
			IndexWriterConfig iwc = new IndexWriterConfig(Version.LUCENE_4_9, analyzer);
			iwc.setOpenMode(OpenMode.CREATE_OR_APPEND);// 创建模式 OpenMode.CREATE_OR_APPEND 添加模式
			writer = new IndexWriter(dir, iwc);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	// 将要建立索引的文件构造成一个Document对象，并添加一个域"content"
	private Document getDocument(File f) throws Exception {
		Document doc = new Document();

		FileInputStream is = new FileInputStream(f);
		Reader reader = new BufferedReader(new InputStreamReader(is));
		// 字符串 StringField LongField TextField
		Field pathField = new StringField("path", f.getAbsolutePath(), Field.Store.YES);
		Field contenField = new TextField("contents", reader);
		// 添加字段
		doc.add(contenField);
		doc.add(pathField);
		return doc;
	}

	public void writeToIndex(String indexFilePath) throws Exception {
		File folder = new File(indexFilePath);

		if (folder.isDirectory()) {
			String[] files = folder.list();
			for (int i = 0; i &lt; files.length; i++) {
				File file = new File(folder, files[i]);
				Document doc = getDocument(file);
				System.out.println("正在建立索引 : " + file + "");
				writer.addDocument(doc);
			}
		}
	}

	public void close() throws Exception {
		writer.close();
	}
}

```
测试：
```



public final static String INDEX_STORE_PATH = "f:/lucene/ch02/index"; // 索引的存放位置
	public final static String INDEX_FILE_PATH = "f:/lucene/ch02/test"; // 索引的文件的存放路径

	@Test
	public void testIndex() throws Exception {
		// 声明一个对象
		LuceneIndex indexer = new LuceneIndex(INDEX_STORE_PATH);
		// 建立索引
		long start = System.currentTimeMillis();
		indexer.writeToIndex(INDEX_FILE_PATH);
		long end = System.currentTimeMillis();

		System.out.println("建立索引用时" + (end - start) + "毫秒");
		indexer.close();
	}

```
##  2、搜索内容 
搜索的一般步骤：
1、创建IndexReader
2、使用IndexReader创建IndexSearcher
3、根据搜索关键字，使用QueryParser或其他Query子类生成Query对象
4、以Query作为参数调用IndexSearcher.search()，执行搜索，也可以设置Filter进行过滤
5、以TopDocs以及ScoreDocs遍历结果并处理

**创建IndexReader需要很大的开销**，因此最好在整个搜索中都重复使用一个IndexReader，只有在必要的时候才创建新的IndexReader。
在创建IndexReader时，它会搜索已有的索引快照，如果你需要搜索索引中的变更信息，那么必须打开一个新的reader。
**IndexReader的openIfChanged()方法可以判断如果索引信息有变更，可以在耗费较少系统资源的情况下重新创建IndexReader。**


搜索的类的主要代码
```

package me.irfen.lucene.ch02;

import java.io.File;
import java.util.Date;

import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.store.FSDirectory;
import org.apache.lucene.util.Version;

public class LuceneSearch {
	// 声明一个IndexSearcher对象
	private IndexSearcher searcher = null;
	// 声明一个Query对象
	private Query query = null;
	private String field = "contents";

	public LuceneSearch(String indexStorePath) {
		try {
			IndexReader reader = DirectoryReader.open(FSDirectory.open(new File(indexStorePath))); //打开reader
			searcher = new IndexSearcher(reader);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	// 返回查询结果
	public final TopDocs search(String keyword) {
		System.out.println("正在检索关键字 : " + keyword);
		try {
			Analyzer analyzer = new StandardAnalyzer(Version.LUCENE_4_9);
			QueryParser parser = new QueryParser(Version.LUCENE_4_9, field, analyzer);
			// 将关键字包装成Query对象
			query = parser.parse(keyword);
			Date start = new Date();
			TopDocs results = searcher.search(query, 5 * 2);
			Date end = new Date();
			System.out.println("检索完成，用时" + (end.getTime() - start.getTime()) + "毫秒");
			return results;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}

	// 打印结果
	public void printResult(TopDocs results) {
		ScoreDoc[] h = results.scoreDocs;
		if (h.length == 0) {
			System.out.println("对不起，没有找到您要的结果。");
		} else {
			for (int i = 0; i &lt; h.length; i++) {
				try {
					Document doc = searcher.doc(h[i].doc);
					System.out.print("这是第" + i + "个检索到的结果，文件名为：");
					System.out.println(doc.get("path"));
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}
		System.out.println("--------------------------");
	}
}

```
测试：
```

public final static String INDEX_STORE_PATH = "f:/lucene/ch02/index"; // 索引的存放位置

	@Test
	public void testSearch() {
		LuceneSearch test = new LuceneSearch(INDEX_STORE_PATH);
		TopDocs h = null;
		h = test.search("中国");
		test.printResult(h);
		h = test.search("人民");
		test.printResult(h);
		h = test.search("共和国");
		test.printResult(h);
	}

```
参考：http://irfen.me/2-lucene4-9-learning-record-lucene-use/
http://qindongliang.iteye.com/blog/1915056