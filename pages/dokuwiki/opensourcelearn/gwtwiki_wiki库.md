title: gwtwiki_wiki库 

#  概述 
GWTWiki是一个wiki 语法转换器
gwtwiki - The Java Wikipedia API (Bliki engine)，是一个 Wikipedia/Mediawiki 语法解析器，可以把 wiki 的文本转换成 HTML。它支持 wiki 标签，例如 bold, italic, headers, nowiki, source, table of contents, tables, lists,categories, footnotes, images 等等。
项目地址：https://bitbucket.org/axelclk/info.bliki.wiki/wiki/Home
文档地址：https://bitbucket.org/axelclk/info.bliki.wiki/wiki/browse/
下载地址：https://code.google.com/p/gwtwiki/downloads/list
#  使用介绍 
#  Mediawiki2HTML 
convert Mediawiki text to HTML
依赖添加：
bliki-core-3.0.xx.jar (version >= 3.0.19)-核心库
bliki-addon-3.0.xx.jar (optional)
bliki-pdf-3.0.xx.jar	(optional)
其他依赖包：打包下载地址： http://gwtwiki.googlecode.com/files/bliki.core.libs.001.zip:
  * commons-logging-1.0.4.jar
  * commons-lang-2.4.jar
  * commons-codec-1.2.jar
  * commons-httpclient-3.1.jar for the [MediaWikiAPISupport MediaWiki api.php support]
  * commons-compress-1.0.jar for the [MediaWikiDumpSupport MediaWiki XML dump support]
  * junit-4.5.jar (only for JUnit tests)
` 保证IDE文件编码为UTF-8 `
###  基本使用 
```

//转换为<p>This is a simple <a href="/Hello_World" title="Hello World">Hello World</a> wiki tag</p>
String htmlText = WikiModel.toHtml("This is a simple [[Hello World]] wiki tag");
//也可以直接写入到流中：
java.io.StringWriter writer = new java.io.StringWriter();
  try {
    WikiModel.toHtml("This is a simple [[Hello World]] wiki tag", writer);
    writer.flush();
    ...
    writer.close();
  } catch (IOException e) {
    e.printStackTrace();
  }

```
#  HTML2Mediawiki 
```

 HTML2WikiConverter conv = new HTML2WikiConverter();
conv.setInputHTML("<b>hello<em>world</em></b>");
String result = conv.toWiki(new ToWikipedia());

```
#  SourceCode2HTML 
```

 String javaCode = "public class HelloWorld {\n" + 
                "   public static void main(String[] args) {\n" + 
                "       System.out.println(\"Hello World\");\n" + 
                "   }\n" + 
                "}\n" + 
                "";
        SourceCodeFormatter f = new JavaCodeFilter();
        String result;
        String coding1 = "<pre class=\"java\" style=\"border: 1px solid #b4d0dc; background-color: #ecf8ff;\">";
        String coding3 = "</pre>";
        result = f.filter(javaCode);
        result = coding1 + result + coding3;
        System.out.println(result);

```

还有很多其他特性，具体查看官方文档：
https://bitbucket.org/axelclk/info.bliki.wiki/wiki/browse/