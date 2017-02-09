title: java2html学习 

#  java2html学习

在写Blog的时候，常常需要粘贴Java源代码，但是从IDE中复制过去的源码为纯文本格式的。IDE中的高亮语法语法全部丢失，贴出去全成黑白的了。看起来很难看，也不易于阅读。java2html是这样一款工具库，可以将java源代码转换成有语法高亮的html以便于阅读。
java2html能够的把java源代码转换为高亮有序的HTML, RTF, TeX 与 XHTML格式。
官方网站：http://www.java2html.de/
Java2Html分两个版本：独立运行版和Eclipse-plugins版。
#  使用 
##  java2html配置文件 
java2html类库会在classpath下查找java2html.properties这个配置文件，如果有，则加载它。
默认配置文件：
```

#Sun Feb 26 17:18:50 CET 2006
defaultStyleName=Eclipse
showFileName=false
showTableBorder=false
showLineNumbers=false
showJava2HtmlLink=false
horizontalAlignment=left
TAB_SIZE=2
Background_COLOR=255,255,255
Background_BOLD=false
Background_ITALIC=false
Line\ numbers_COLOR=128,128,128
Line\ numbers_BOLD=false
Line\ numbers_ITALIC=false
Multi-line\ comments_COLOR=63,127,95
Multi-line\ comments_BOLD=false
Multi-line\ comments_ITALIC=false
Single-line\ comments_COLOR=63,127,95
Single-line\ comments_BOLD=false
Single-line\ comments_ITALIC=false
Keywords_COLOR=127,0,85
Keywords_BOLD=true
Keywords_ITALIC=false
Strings_COLOR=42,0,255
Strings_BOLD=false
Strings_ITALIC=false
Character\ constants_COLOR=153,0,0
Character\ constants_BOLD=false
Character\ constants_ITALIC=false
Numeric\ constants_COLOR=153,0,0
Numeric\ constants_BOLD=false
Numeric\ constants_ITALIC=false
Parenthesis_COLOR=0,0,0
Parenthesis_BOLD=false
Parenthesis_ITALIC=false
Primitive\ Types_COLOR=127,0,85
Primitive\ Types_BOLD=true
Primitive\ Types_ITALIC=false
Others_COLOR=0,0,0
Others_BOLD=false
Others_ITALIC=false
Javadoc\ keywords_COLOR=127,159,191
Javadoc\ keywords_BOLD=false
Javadoc\ keywords_ITALIC=false
Javadoc\ HTML\ tags_COLOR=127,127,159
Javadoc\ HTML\ tags_BOLD=false
Javadoc\ HTML\ tags_ITALIC=false
Javadoc\ links_COLOR=63,63,191
Javadoc\ links_BOLD=false
Javadoc\ links_ITALIC=false
Javadoc\ others_COLOR=63,95,191
Javadoc\ others_BOLD=false
Javadoc\ others_ITALIC=false
Undefined_COLOR=255,97,0
Undefined_BOLD=false
Undefined_ITALIC=false
Annotation_COLOR=100,100,100
Annotation_BOLD=false
Annotation_ITALIC=false

```
当然你也可以**硬编码配置**converter options配置
```

//Get the default options and adjust it to your needs
JavaSourceConversionOptions options =
  JavaSourceConversionOptions.getDefault();
options.setShowLineNumbers(false);
options.getStyleTable().put(
  JavaSourceType.KEYWORD,
  new JavaSourceStyleEntry(Color.orange, true, false));
converter.setConversionOptions(options);

```
##  使用示例 
```

//Create a reader of the raw input text
StringReader stringReader = new StringReader(
  "/** Simple Java2Html Demo */\r\n"+      
  "public static int doThis(String text){ return text.length() + 2; }");

//Parse the raw text to a JavaSource object
JavaSource source = null;
try {
  source = new JavaSourceParser().parse(stringReader);
} catch (IOException e) {
  e.printStackTrace();
  System.exit(1);
}

//Create a converter and write the JavaSource object as Html
JavaSource2HTMLConverter converter = new JavaSource2HTMLConverter();
StringWriter writer = new StringWriter(); 
try {
  JavaSourceConversionOptions options =JavaSourceConversionOptions.getDefault();
  converter.convert(source,options,writer);
} catch (IOException e) {
}
System.out.println(writer.toString());

```
使用Java2Html类
如果你嫌上述过程比较复杂，你也可以直接使用一个类Java2Html，它可以直接将raw string转换个string.无需创建流等。
```

 String rawStr="/** Simple Java2Html Demo */\r\n"+      
  "public static int doThis(String text){ return text.length() + 2; }";
String conStr=Java2Html.convertToHtml(rawStr);//或者convertToHtml(java.lang.String text, JavaSourceConversionOptions options) 
//如果你想转换为完整的html页面，调用下面的。
String conPage=Java2Html.convertToHtmlPage(rawStr);

```
