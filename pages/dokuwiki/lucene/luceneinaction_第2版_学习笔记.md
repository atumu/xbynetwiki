title: luceneinaction_第2版_学习笔记 

#  LuceneInAction(第2版)学习笔记—第一章 初识Lucene 
书中Lucene版本3.0
4.x版本教程：http://lucene.apache.org/core/4_10_4/index.html
4.x API变动说明:http://lucene.apache.org/core/4_10_4/MIGRATE.html
Lucene: 是一个搜索类库，而不是完整的程序
Lucene除了核心JAR包之外，还有位于contrib区域的大量的扩展模块，如spellchecker,highlighter等
![](/data/dokuwiki/lucene/pasted/20160227-114523.png)
maven:
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
##  A. 索引组件 

根据原始内容创建索引
Raw Content : Acquire Content    --> Build Document --> Analyze Document(*) --> Index Document(*) ==> Index(*)
原始内容    : 获取内容(提取文本) --> 建立文档     --> 分析文档     --> 文档索引   ==> 索引

**索引过程**中的核心类
` Document `类:  一个Lucene文档可以包含多个域` Field `
` Analyzer `类: 分析器 分析文档，如果被索引内容不是纯文本文件，则需要先将其转换为文本文档
分析器是一个抽象类，Lucene提供了几个类实现它
分析器的分析对象是文档，该文档包含一些分离的能被索引的域
` IndexWriter `类: 索引写入器，它负责创建新索引或打开已有索引，以及向索引中添加、删除、更新被索引文档的信息.IndexWrite 提供对索引文件的写入操作，但不能用于读取或搜索索引
` Directory `类: 索引文件的存储路径。一般通过` FSDirectory.open(dir) `自动选择并创建最佳子类实例。
```

//以Lucene4.x的示例代码
import org.apache.commons.io.FileUtils;
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field.Store;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import org.apache.lucene.util.Version;

public class Indexer {
	private IndexWriter writer;
	public Indexer(String indexDir) throws IOException {
		Directory dir=FSDirectory.open(FileUtil.getDirFromString(indexDir));
		Analyzer analyzer=new StandardAnalyzer();
		IndexWriterConfig config=new IndexWriterConfig(Version.LUCENE_CURRENT, analyzer);
		writer=new IndexWriter(dir, config);
		
	}
	public void close() throws IOException{
		writer.close();//关闭IndexWriter
	}
	public int index(String dataDir,FileFilter filter){
		File[] files=new File(dataDir).listFiles();
		for(File f:files){
			if(!f.isDirectory()&&f.exists()&&filter.accept(f)){
				try {
					indexFile(f);
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
		return writer.numDocs();//返回被索引的文档数
	}
	private static class TextFilesFilter implements FileFilter{
		public boolean accept(File f) {
			return f.getName().toLowerCase().endsWith(".txt");
		}
	}
	protected Document getDocument(File f) throws IOException{
		Document doc=new Document();
		doc.add(new TextField("fileName",f.getName(),Store.YES));
		doc.add(new TextField("filePath",f.getAbsolutePath(),Store.YES));
		doc.add(new TextField("content",FileUtils.readFileToString(f),Store.YES));
		return doc;
	}
	private void indexFile(File f) throws IOException{
		System.out.println("索引文件:"+f.getAbsolutePath());
		Document doc=getDocument(f);
		writer.addDocument(doc);
	}
	public static void main(String[] args) throws IOException{
		String indexDir="E:\\demo\\test\\search";
		String dataDir="E:\\workspace\\elasticsearch-2.1.1";
		Indexer indexer=new Indexer(indexDir);
		long start=System.currentTimeMillis();
		int indexed=0;
		try{
			indexed=indexer.index(dataDir, new TextFilesFilter());
		}finally{
			indexer.close();
		}
		long end=System.currentTimeMillis();
		System.out.println("use time:"+(end-start));
	}
}


``` 
在创建索引的过程中比较重要的就是创建不同的Field，看看api就可以知道Field有哪些实现：BinaryDocValuesField, DoubleField, FloatField, IntField, LongField, NumericDocValuesField, SortedDocValuesField, SortedNumericDocValuesField, SortedSetDocValuesField, StoredField, StringField, TextField。

比较常用的有LongField、StoredField、StringField、TextField。具体有哪些构造方法建议大家自己查阅api文档。
每种Field都有具体的介绍，我这里重点介绍一下常用的几个：
LongField：索引但是不分词，适用于全部搜索，一般用于文件创建、修改时间的秒数等。
StoredField：只存不索引
**StringField：索引但是不分词，所以适用于全部搜索的内容**，比如国家等，要么不对，要么就是全对。另外提一句，**这个有个长度限制是32766**，大家使用的时候自己斟酌一下，一般不会超过。
**TextField：索引并分词，所以这个应该是我们做全文检索的时候应该最常用到的一个Field。**

官方索引示列：http://lucene.apache.org/core/4_10_4/demo/src-html/org/apache/lucene/demo/IndexFiles.html

##  B. 搜索组件 

在索引中搜索单词，找到包含该单词的文档
Search UI   : Build Query(*)  --> Run Query(*) --> Render Results(*)
用户搜索界面: 建立查询       --> 运行查询     --> 展现结果

**搜索过程**中的核心类
` IndexSearcher `类: 搜索器，用于搜索由IndexWriter类创建的索引，可以将其看作一个以只读方式打开索引的类。它需要利用Directory实例来掌控前期创建的索引，然后才能提供大量的搜索方法。搜索器中有一些方法在它的抽象父类Searcher中实现。最简单的搜索方法是将单个Query对象和int topN计数作为该方法的参数search(Query q,int topN)，并返回一个` TopDocs `对象。
` IndexReader `类：用于打开索引完成索引读取工作。IndexSearcher底层就是通过它来读取索引的。DirectoryReader是其子类之一
` Term `抽象父类:** Term对象是搜索功能的基本单元**；与Field对象类似，Term对象包含一对字符串元素，域名和单词(或域文本值).Term对象还与索引操作有关。但Term对象是由Lucene内部创建的，在索引阶段不需要了解它们。
在比较简单的搜索过程中，可以创建Term对象，并和` TermQuery `对象一起使用。
```

public class Searcher {
	public static void search(String indexDir,String q) throws IOException, ParseException{
		Directory dir=FSDirectory.open(new File(indexDir));
		Analyzer analyzer=new StandardAnalyzer();
		DirectoryReader reader=DirectoryReader.open(dir);//创建IndexReader
		IndexSearcher search=new IndexSearcher(reader);
		QueryParser parser =new QueryParser("content",analyzer);
		Query query=parser.parse(q);
		long start=System.currentTimeMillis();
		TopDocs hits=search.search(query, 10); //返回结果的指针容器TopDocs结果集
		long end=System.currentTimeMillis();
		System.out.println("search use time:"+(end-start));
		for(ScoreDoc sdoc:hits.scoreDocs){
			Document doc=search.doc(sdoc.doc);//真正加载被搜索到的文档 sdoc.doc是得到文档的ID
			System.out.println(doc.get("filePath"));
		}
		
          	reader.close(); //完成后记得关闭，不过IndexReader打开时比较耗资源的，所以一般建议全局公用一个IndexReader，需要更新变更时调用它的reopen()方法即可。
		dir.close(); //Directory也需要被关闭
          	search.close();//关闭
		
	}
	public static void main(String[] args) throws IOException, ParseException {
		String dir="E:\\demo\\test\\search";
		search(dir, "2004");
	}
}

```
` Query `抽象父类：有很多查询子类，TermQuery/BooleanQuery/PhraseQuery/PrefixQuery/PhrasePrefixQuery/TermRangeQuery/NumericRangeQuery/FilteredQuery/SpanQuery
TermQuery类： 是Lucene提供的最基本的查询类型，也是简单查询类型之一，它用来匹配指定域中包含特定项的文档
` TopDocs `类： **就一个简单的指针容器，一般指向前N个排名的搜索结果**，搜索结果即匹配查询条件的文档

C. 其它模块
管理界面: Administration Interface
分析界面: Analytics Interface
搜索范围: Scaling 分布式索引和分布式查询时，各节点的范围

官方搜索示列：http://lucene.apache.org/core/4_10_4/demo/src-html/org/apache/lucene/demo/SearchFiles.html
##  D. 附加组件 

  * 中文分词组件。 网上有几个比较好的开源中文分词库IKAnalyzer，mmseg
  * 中文短语查询组件

**分词: 索引时对域进行分词，搜索时对关键字进行分词，分词后得到语汇单元。**
总的来说，搜索比索引更重要，因为索引文件只被创建一次，却要被搜索多次。

Lucene只是一款核心搜索库，目前有很多的搜索引擎：
  * Solr:支持从关系数据库和XML文档中提取原始数据，以及能够通过集成` Tika `来处理复杂文档。
  * Nutch：爬虫工具
  * Grub:比较流行的爬虫工具

##  Lucene索引文件结构速览 
**Lucene的索引结构是有层次结构。**
每个层次都保存了本层次的信息以及下一层次的元信息。
1) 索引Index
 在Lucene中，**一个索引是放在一个文件夹中的**
2) 段Segment
** 一个索引可以包含多个段，段与段之间是独立的。**添加新文档可以生成新的段，不同的段可以合并。
3) 文档Doucument
文档是我们建索引的基本单位,不同的是保存在不同的段中的,一个段可以包含多篇文档,新加的文档是单独保存在一个新生成的段中的,随着段的合并，不同的文档合并到同一个段中
4) 域Field
一篇文档包含不同类型的信息，可以分开索引,不同域的索引方式可以不同
5) 词Term
** 词是索引的最小单位,是经过词法分析和语言处理后的字符串**

''Lucene的索引结构，即保存了正向信息，也保存了反向信息。
正向信息：索引Index --> 段Segment --> 文档Document --> 域Field --> 词Term
反向信息：词Term --> 文档Document''
 
1. 属于整个索引index的文件
1.1. Generation of index【segments.gen】
segments.gen
1.2. 段的元数据 Segment metadata【segments_N】
segments_N
保存的是段segment的元数据
然后分多个segment保存数据信息
同一个segment有相同的前缀文件名
 
1.3. 写锁定文件 Write lock file【Write.lock】
Write.lock
1.4. 段数据 Segment data下节细讲

2. 属于某个段segment的文件
对于每个段，包含域信息、词信息，以及其它信息，如标准化因子、删除文档
2.1. 域信息 Field Information (是正向信息)
2.1.1.域的元数据信息 Field metadata 【.fnm】
.fnm 保存了此段包含了多少个域，每个域的名称及索引方式。
2.1.2.域的数据信息 Field data 【.fdt/.fdx】
.fdt、.fdx 保存了此段包含的所有文档 
.fdt是域数据文件: 保存存储域信息
.fdx是域索引文件: 保存索引域信息
 
2.1.3. 词向量Term Vector的数据信息【.tvx/.tvd/.tvf】
.tvx词向量索引文件
.tvd词向量文档文件
.tvf词向量域文件
 
2.2. 词信息 Term Information (是反向信息)

2.2.1. 词典 Term Dictionary 【.tis/.tii】
.tis词典文件
.tii词典索引文件
 
2.2.2. 文档号及词频信息 Term Frequency 【.frq】
.frq: 文档号及词频倒排表
 
2.2.3. 词位置信息 Term Postion 【.prx】
.prx: 词位置倒排表 Term Postion
 
2.3. 其它信
2.3.1. 标准化因子文件 Normalization Info 【.nrm】
.nrm
 
2.3.2. 删除文档文件 Deleted Documents 【.del】
.del
参考：http://blog.csdn.net/liuweitoo/article/details/8137401