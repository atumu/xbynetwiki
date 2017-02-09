title: tika学习1 

#  tika学习一 
最新版本：1.12
官网：http://tika.apache.org/
APIDOC:http://tika.apache.org/1.12/api/
Tika是一个内容抽取的工具集合(a toolkit for text extracting)。它集成了POI, Pdfbox 并且为文本抽取工作提供了一个统一的界面。
其次，Tika也提供了便利的扩展API，用来丰富其对第三方文件格式的支持。
Tika的功能
  * 文档类型检测
  * 内容提取
  * 元数据提取
  * 语言检测
Tika提供了对如下文件格式的支持:
![](/data/dokuwiki/tika/pasted/20160228-144636.png)
![](/data/dokuwiki/tika/pasted/20160228-144653.png)
<html>
<table class="table table-bordered" style="box-sizing: border-box; border-collapse: collapse; border-spacing: 0px; width: 578px; max-width: 100%; margin-bottom: 20px;  border-color: rgb(221, 221, 221); color: rgb(49, 49, 49); font-family: 'Open Sans', Arial, sans-serif; font-size: 14px; line-height: 22px; background-color: transparent;">
	<tbody style="box-sizing: border-box;">
		<tr style="box-sizing: border-box;">
			<th style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221); background: rgb(238, 238, 238);">
				文件格式</th>
			<th style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221); background: rgb(238, 238, 238);">
				类库</th>
			<th style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221); background: rgb(238, 238, 238);">
				Tika中的类</th>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				XML</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.xml</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				XMLParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				HTML</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.htmll and it uses Tagsoup Library</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				HtmlParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				MS-Office compound document Ole2 till 2007 ooxml 2007 onwards</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				<p style="box-sizing: border-box; color: rgb(0, 0, 0); line-height: 24px; margin: 0em 0.2em 1em; padding: 0px; text-align: justify;">
					org.apache.tika.parser.microsoft</p>
				<p style="box-sizing: border-box; color: rgb(0, 0, 0); line-height: 24px; margin: 0em 0.2em 1em; padding: 0px; text-align: justify;">
					org.apache.tika.parser.microsoft.ooxml and it uses Apache Poi library</p>
			</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				<p style="box-sizing: border-box; color: rgb(0, 0, 0); line-height: 24px; margin: 0em 0.2em 1em; padding: 0px; text-align: justify;">
					OfficeParser(ole2)</p>
				<p style="box-sizing: border-box; color: rgb(0, 0, 0); line-height: 24px; margin: 0em 0.2em 1em; padding: 0px; text-align: justify;">
					OOXMLParser(ooxml)</p>
			</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				OpenDocument Format openoffice</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.odf</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				OpenOfficeParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				portable Document Format(PDF)</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.pdf and this package uses Apache PdfBox library</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				PDFParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Electronic Publication Format (digital books)</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.epub</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				EpubParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Rich Text format</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.rtf</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				RTFParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Compression and packaging formats</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.pkg and this package uses Common compress library</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				PackageParser and CompressorParser and its sub-classes</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Text format</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.txt</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				TXTParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Feed and syndication formats</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.feed</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				FeedParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Audio formats</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.audio and org.apache.tika.parser.mp3</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				AudioParser MidiParser Mp3- for mp3parser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Imageparsers</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.jpeg</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				JpegParser-for jpeg images</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Videoformats</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.mp4 and org.apache.tika.parser.video this parser internally uses Simple Algorithm to parse flash video formats</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Mp4parser FlvParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				java class files and jar files</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.asm</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				ClassParser CompressorParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Mobxformat (email messages)</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.mbox</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				MobXParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				Cad formats</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.dwg</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				DWGParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				FontFormats</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.font</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				TrueTypeParser</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				executable programs and libraries</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				org.apache.tika.parser.executable</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				ExecutableParser</td>
		</tr>
	</tbody>
</table>
</html>
**` 支持格式详情，以及对应的MIME，需要用到的外部解析库请参考http://tika.apache.org/1.12/formats.html `**
组件：
**tika-core-*.jar.包含核心接口和类，但是不包含任何解析器parser的实现。需要Java6以上。
tika-parsers-*.jar。包含基于各种外部解析库的Tika Parser接口的实现，也就相当于解析适配器。**
tika-app-*.jar。Tika application. 包含所有以上组件和所有的外部解析库（all the external parser libraries） 支持 GUI 或命令行方式运行的完整程序。
tika-server-*.jar，Tika JAX-RS REST application. This is a Jetty web server running Tika REST services as described in this page.
tika-bundle-*.jar，Tika bundle. An OSGi bundle that combines tika-parsers with non-OSGified parser libraries to make them easy to deploy in an OSGi environment.

Maven：
```

<dependency>
<!--    <dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-core</artifactId>
    <version>1.12</version>
  </dependency> -->
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-parsers</artifactId>
    <version>1.12</version>
  </dependency>

```
它会自动引入tika-core,和所有需要用到的外部解析依赖库。
**Tika提取的元数据主键**
![](/data/dokuwiki/tika/pasted/20160228-144742.png)

Tika的API十分便捷，` 核心API是Parser ` interface，其中定义了一个parse方法：
```

void parse(
    InputStream stream, ContentHandler handler, Metadata metadata,
    ParseContext context) throws IOException, SAXException, TikaException;

```
用stream参数传递需要解析的文件流， 文本内容会被传入handler进行具体的解析工作，而元数据会更新至metadata(它是双向的：输入数据的metadata，输出数据的metadata)。
可以使用Tika的` ParserUtils `工具来根据文件的mime-type来得到一个适当的Parser来进行解析工作。或者
Tika还提供了一个` AutoDetectParser `根据不同的二进制文件的特殊格式 (比如说Magic Code)，来寻找适合的Parser。
<html>
<table class="table table-bordered" style="box-sizing: border-box; border-collapse: collapse; border-spacing: 0px; width: 578px; max-width: 100%; margin-bottom: 20px;  border-color: rgb(221, 221, 221); color: rgb(49, 49, 49); font-family: 'Open Sans', Arial, sans-serif; font-size: 14px; line-height: 22px; background-color: transparent;">
	<tbody style="box-sizing: border-box;">
		<tr style="box-sizing: border-box;">
			<th style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221); background: rgb(238, 238, 238);">
				S.No.</th>
			<th style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221); background: rgb(238, 238, 238);">
				对象及描述</th>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				1</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				<p style="box-sizing: border-box; color: rgb(0, 0, 0); line-height: 24px; margin: 0em 0.2em 1em; padding: 0px; text-align: justify;">
					<b style="box-sizing: border-box;">InputStream stream</b></p>
				<p>
					包含任何文件的InputStream对象的内容</p>
			</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				2</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				<p style="box-sizing: border-box; color: rgb(0, 0, 0); line-height: 24px; margin: 0em 0.2em 1em; padding: 0px; text-align: justify;">
					<b style="box-sizing: border-box;">ContentHandler handler</b></p>
				<p>
					Tika通过文档作为XHTML内容到此处理，此后该文件正在使用SAX API处理。它提供了在一个文件有效的后处理的内容。</p>
			</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				3</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				<p style="box-sizing: border-box; color: rgb(0, 0, 0); line-height: 24px; margin: 0em 0.2em 1em; padding: 0px; text-align: justify;">
					<b style="box-sizing: border-box;">Metadata metadata</b></p>
				<p>
					元数据对象是用来既作为源和文件的元数据的目标。</p>
			</td>
		</tr>
		<tr style="box-sizing: border-box;">
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				4</td>
			<td style="box-sizing: border-box; padding: 8px; line-height: 1.42857143; vertical-align: top;  border-color: rgb(221, 221, 221);">
				<p style="box-sizing: border-box; color: rgb(0, 0, 0); line-height: 24px; margin: 0em 0.2em 1em; padding: 0px; text-align: justify;">
					<b style="box-sizing: border-box;">ParseContext context</b></p>
				<p>
					此对象使用在如遇有客户端应用程序想要定制解析过程。</p>
			</td>
		</tr>
	</tbody>
</table>
</html>
##  5分钟入门 
###  添加自定义的MIME-Type 
修改` tika-core/src/main/resources/org/apache/tika/mime/tika-mimetypes.xml ` 
**或者create a new file "org/apache/tika/mime/` custom-mimetypes.xml `" in your codebase**. You should add to it something like this:
```

 <?xml version="1.0" encoding="UTF-8"?>
 <mime-info>
   <mime-type type="application/hello">
          <glob pattern="*.hi"/>
   </mime-type>
 </mime-info>

```
##  自定义Parser解析器 
```

import org.apache.tika.exception.TikaException;
import org.apache.tika.metadata.Metadata;
import org.apache.tika.mime.MediaType;
import org.apache.tika.parser.ParseContext;
import org.apache.tika.parser.AbstractParser;
import org.apache.tika.sax.XHTMLContentHandler;
import org.xml.sax.ContentHandler;
import org.xml.sax.SAXException;

public class HelloParser extends AbstractParser {

        private static final Set<MediaType> SUPPORTED_TYPES = Collections.singleton(MediaType.application("hello"));
        public static final String HELLO_MIME_TYPE = "application/hello";
        
        public Set<MediaType> getSupportedTypes(ParseContext context) {
                return SUPPORTED_TYPES;
        }

        public void parse(
                        InputStream stream, ContentHandler handler,
                        Metadata metadata, ParseContext context)
                        throws IOException, SAXException, TikaException {

                metadata.set(Metadata.CONTENT_TYPE, HELLO_MIME_TYPE);
                metadata.set("Hello", "World");

                XHTMLContentHandler xhtml = new XHTMLContentHandler(handler, metadata);
                xhtml.startDocument();
                xhtml.endDocument();
        }
}

```
**让AutoDetectParser包含自定义的parser**
List your new parser in:** tika-parsers/src/main/resources/META-INF/services/org.apache.tika.parser.Parser**

##  Content Detection内容检测 
org.apache.tika.detect.` Detector `
```

MediaType detect(java.io.InputStream input,Metadata metadata) throws java.io.IOException

```
通常，Detector只需要使用Metadata的两个属性： Metadata.RESOURCE_NAME_KEY 代表文件名, and Metadata.CONTENT_TYPE 代表文件类型

几种内置的Detector实现：
Mime Magic Detection： org.apache.tika.detect.` MagicDetector `.根据文件Magic数字识别,遵守文件的原始字节，可以为每个文件找到一些独特的字符模式。一些文件具有特殊的字节前缀称为被专门制成并包含在一个文件中，用于识别文件类型的目的魔术(Magic)字节。
Resource Name Based Detection：org.apache.tika.detect.` NameDetector `.根据扩展名识别
default Mime Types Detector：org.apache.tika.mime.MimeTypes
default Tika Detector：org.apache.tika.detect.DefaultDetector。自动检测所有的Detector，然后选择合适的进行检查文档。**需要配合org.apache.tika.io.TikaInputStream.而不是InputStream使用，否则没有效果。**
进行内容检测：
```

TikaConfig tika = new TikaConfig();

for (File f : myListOfFiles) {
   Metadata metadata = new Metadata();
   metadata.set(Metadata.RESOURCE_NAME_KEY, f.toString());
   String mimetype = tika.getDetector().detect(
        TikaInputStream.get(f), metadata);
   System.out.println("File " + f + " is " + mimetype);
}
for (InputStream is : myListOfStreams) {
   String mimetype = tika.getDetector().detect(
        TikaInputStream.get(is), new Metadata());
   System.out.println("Stream " + is + " is " + mimetype);
}

```
###  Language Detection 
不支持中文
org.apache.tika.language.LanguageIdentifier
```

public String identifyLanguage(String text) {
    LanguageIdentifier identifier = new LanguageIdentifier(text);
    return identifier.getLanguage();
}

```
##  配置Tika 
主要配置:Parsers和Detectors
1、Tika config xml file路径配置：
```

TikaConfig config = new TikaConfig("/path/to/tika-config.xml");
Detector detector = config.getDetector();
Parser autoDetectParser = new AutoDetectParser(config);

```
2、tika-config.xml中配置解析器parsers
```

<?xml version="1.0" encoding="UTF-8"?>
<properties>
  <parsers>
    <!-- Default Parser for most things, except for 2 mime types, and never
         use the Executable Parser -->
    <parser class="org.apache.tika.parser.DefaultParser">
      <mime-exclude>image/jpeg</mime-exclude>
      <mime-exclude>application/pdf</mime-exclude>
      <parser-exclude class="org.apache.tika.parser.executable.ExecutableParser"/>
    </parser>
    <!-- Use a different parser for PDF -->
    <parser class="org.apache.tika.parser.EmptyParser">
      <mime>application/pdf</mime>
    </parser>
  </parsers>
</properties>

```
Java代码配置方式：org.apache.tika.parser.DefaultParser, org.apache.tika.parser.CompositeParser and org.apache.tika.parser.ParserDecorator.

3、tika-config.xml中Detector配置：
```

<?xml version="1.0" encoding="UTF-8"?>
<properties>
  <detectors>
    <!-- All detectors except built-in container ones -->
    <detector class="org.apache.tika.detect.DefaultDetector">
      <detector-exclude class="org.apache.tika.parser.pkg.ZipContainerDetector"/>
      <detector-exclude class="org.apache.tika.parser.microsoft.POIFSContainerDetector"/>
    </detector>
  </detectors>
</properties>

```
编程方式：org.apache.tika.detect.DefaultDetector and org.apache.tika.detect.CompositeDetector.

##  tika获取文件编码 
```

AutoDetectReader dr=new AutoDetectReader(new FileInputStream(file));
System.out.println(dr.getCharset().name());

```
##  API使用示例 
parsing解析操作：
1、Parsing using the Tika Facade 
```

public String parseToStringExample() throws IOException, SAXException, TikaException {
    Tika tika = new Tika();
    try (InputStream stream = ParsingExample.class.getResourceAsStream("test.doc")) {
        return tika.parseToString(stream);
    }
}

```
2、Parsing using the Auto-Detect Parser
```

public String parseExample() throws IOException, SAXException, TikaException {
    //TikaConfig config=new TikaConfig(); //TikaConfig.getDefaultConfig();
    AutoDetectParser parser = new AutoDetectParser();
    BodyContentHandler handler = new BodyContentHandler();
    Metadata metadata = new Metadata();
    try (InputStream stream = ParsingExample.class.getResourceAsStream("test.doc")) {
        parser.parse(stream, handler, metadata);
        return handler.toString();
    }
}

```
3、解析后输出不同的格式
这取决于你使用什么样的ContentHandler实现

```

public String parseTo() throws IOException, SAXException, TikaException {
    BodyContentHandler handler = new BodyContentHandler();//Parsing to Plain Text：
  
    // ContentHandler handler = new ToXMLContentHandler(); Parsing to XHTML
  
   // ContentHandler handler = new BodyContentHandler(new ToXMLContentHandler());Parsing to XHTML但是不包含header部分
  
   // Only get things under html -> body -> div (class=header)，通过Matcher，MatchingContentHandler提取局部内容
   // XPathParser xhtmlParser = new XPathParser("xhtml", XHTMLContentHandler.XHTML);
   // Matcher divContentMatcher = xhtmlParser.parse("/xhtml:html/xhtml:body/xhtml:div/descendant::node()");
    //ContentHandler handler = new MatchingContentHandler(new ToXMLContentHandler(), divContentMatcher)
    
  //PhoneExtractingContentHandler,提取电话号码
    
    AutoDetectParser parser = new AutoDetectParser();
    Metadata metadata = new Metadata();
    try (InputStream stream = ContentHandlerExample.class.getResourceAsStream("test.doc")) {
        parser.parse(stream, handler, metadata);
        return handler.toString();
    }
}

```

**Streaming the plain text in chunks**
```

public List<String> parseToPlainTextChunks() throws IOException, SAXException, TikaException {
    final List<String> chunks = new ArrayList<>();
    chunks.add("");
    ContentHandlerDecorator handler = new ContentHandlerDecorator() {
        @Override
        public void characters(char[] ch, int start, int length) {
            String lastChunk = chunks.get(chunks.size() - 1);
            String thisStr = new String(ch, start, length);
 
            if (lastChunk.length() + length > MAXIMUM_TEXT_CHUNK_SIZE) {
                chunks.add(thisStr);
            } else {
                chunks.set(chunks.size() - 1, lastChunk + thisStr);
            }
        }
    };
 
    AutoDetectParser parser = new AutoDetectParser();
    Metadata metadata = new Metadata();
    try (InputStream stream = ContentHandlerExample.class.getResourceAsStream("test2.doc")) {
        parser.parse(stream, handler, metadata);
        return chunks;
    }
}

```
参考：http://www.yiibai.com/tika/