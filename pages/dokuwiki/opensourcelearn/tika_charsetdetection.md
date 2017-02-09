title: tika_charsetdetection 

# 使用apache_tika检测文件编码 
在java中检测文件编码方式还是比较难的，平台API没有提供相关功能。**而在很多情况下检测文件编码是比较重要的。**
仔细搜索了一番。发现很多开源库都已经停止开发，而且支持的编码格式不多，或者接口难用。
终于在http://stackoverflow.com/questions/3759356/what-is-the-most-accurate-encoding-detectorfa最下面发现了这样一段话：
**AutoDetectReader** of **Apache Tika** does exactly this - first tries to use HtmlEncodingDetector, then UniversalEncodingDetector(which is based on juniversalchardet), and then tries Icu4jEncodingDetector(based on ICU4J).
也就是说使用Apache Tika项目的AutoDetectReader类可以很精确地检测出文件编码。它本身并不进行具体检测工作，而是在底层会调用多个解析库按顺序进行解析以获取文件编码。调用解析库的顺序为:
  - HtmlEncodingDetector
  - UniversalEncodingDetector(which is based on juniversalchardet), 
  - Icu4jEncodingDetector(based on ICU4J).

#  Aache Tika介绍与安装 
官网:http://tika.apache.org/
Apache Tika 内容抽取的工具集合(a toolkit for text extracting）利用现有的解析类库，从不同格式的文档中（例如HTML, PDF, Doc)，**侦测和提取出元数据和结构化内容**。
功能包括：
  * 侦测文档的类型，字符编码，语言，等其他现有文档的属性。
  * 提取结构化的文字内容。
该项目的**目标使用群体**主要为**搜索引擎以及其他内容索引和分析工具**。编程语言为Java.
Tika支持的解析类库众多，可以解析众多文件形式。
<blockquote>The Apache Tika™ toolkit detects and extracts metadata and text from over a thousand different file types (such as PPT, XLS, and PDF). All of these file types can be parsed through a single interface, making Tika useful for search engine indexing, content analysis, translation, and much more</blockquote>
两个核心接口:Parser与Detector
主要类：Metadata,BodyContentHandler,AutoDetectParser,AutoDetectReader
安装：
官网上可以下载的有三个src.zip,app.jar,server.jar。其中app.jar是一个带有GUI界面的工具软件。
这些都不是我们想要的，我们需要可以从源码构建mvn install或者从maven仓库中获取。主要为两个jar文件tika-core.jar,tika-parsers.jar提供底层解析器
```

<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-core</artifactId>
    <version>...</version>
  </dependency>
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-parsers</artifactId>
    <version>...</version>
  </dependency>

```

#  使用apache tika获取文件编码 
```

AutoDetectReader dr=new AutoDetectReader(new FileInputStream(file));
System.out.println(dr.getCharset().name());

```
