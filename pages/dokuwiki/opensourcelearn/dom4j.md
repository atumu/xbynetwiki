title: dom4j 

#  dom4j解析xml 
dom4j是一个非常流行的java的xml解析库
官网：https://github.com/dom4j/dom4j/
官方教程：https://github.com/dom4j/dom4j/wiki/Cookbook
API ：http://www.oschina.net/uploads/doc/dom4j-1.6.1/index.html
可供参考的资料：
http://blog.csdn.net/redarmy_chen/article/details/12969219
http://www.blogjava.net/i369/articles/154264.html

##  安装 
```

<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6</version>
</dependency>
<!--用于支持XPath表达式 -->
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>1.1.6</version>
</dependency>

```

可以针对雅虎天气API写了解析XML的例子：
http://weather.yahooapis.com/forecastrss?p=邮编(如http://weather.yahooapis.com/forecastrss?p=60202)

##  使用 
```

InputStream  ins=loader.getResourceAsStream(COMP_XML_CONFIG_PATH);
		 SAXReader reader = new SAXReader();             
	       Document   document = reader.read(ins);
	       Element root = document.getRootElement();
	       List<Element> components=root.elements();
	       for(Element e:components){
	    	   
	    	   Attribute attribute=e.attribute("name");
//	    	   System.out.println(e.getText()+":"+attribute.getText());
	    	   map.put(attribute.getText(), e.getText());
	    	  
	    	   
	       }

```
` SAXReader的read() `方法支持:
  * ` java.lang.String ` - a SystemId is a String that contains a URI e.g. a URL to a XML file
  * ` java.net.URL ` - represents a Uniform Resource Loader or a Uniform Resource Identifier. Encapsulates a URL.
  * ` java.io.InputStream ` - an open input stream that transports xml data
  * ` java.io.Reader ` - more compatible. Has abilitiy to specify encoding scheme
  * ` org.sax.InputSource ` - a single input source for a XML entity.
它的主要接口都在org.dom4j这个包里定义：
![](/data/dokuwiki/opensourcelearn/pasted/20150828-084005.png)
要想弄懂这套接口，关键的是要明白**接口的继承关系**：
![](/data/dokuwiki/opensourcelearn/pasted/20150828-084043.png)
##  １．读取并解析XML文档： 

读写XML文档主要依赖于org.dom4j.io包，其中提供` DOMReader和SAXReader `两类不同方式，而调用方式是一样的。这就是依靠接口的好处。
```

    // 从文件读取XML，输入文件名，返回XML文档
    public Document read(String fileName) throws MalformedURLException, DocumentException {
       SAXReader reader = new SAXReader();
       Document document = reader.read(new File(fileName));
       return document;
    }

``` 
其中，**reader的read方法是重载的，可以从InputStream, File, Url等多种不同的源来读取**。得到的Document对象就带表了整个XML。
根据本人自己的经验，读取的字符编码是按照XML文件头定义的编码来转换。如果遇到乱码问题，注意要把各处的编码名称保持一致即可。
**２． 取得Root节点**
读取后的第二步，就是得到Root节点。熟悉XML的人都知道，一切XML分析都是从Root元素开始的。
```

　  public Element getRootElement(Document doc){
       return doc.getRootElement();
    }

```
###  节点对象操作的方法 

```

1.获取文档的根节点.  
      Element root = document.getRootElement();  
    2.取得某个节点的子节点.  
      Element element=node.element(“四大名著");  
    3.取得节点的文字  
        String text=node.getText();  
    4.取得某节点下所有名为“csdn”的子节点，并进行遍历.  
       List nodes = rootElm.elements("csdn");   
         for (Iterator it = nodes.iterator(); it.hasNext();) {     
      Element elm = (Element) it.next();    
    // do something  
 }  
     5.对某节点下的所有子节点进行遍历.      
      for(Iterator it=root.elementIterator();it.hasNext();){        
        Element element = (Element) it.next();        
       // do something   
 }  
    6.在某节点下添加子节点  
      Element elm = newElm.addElement("朝代");  
    7.设置节点文字.  elm.setText("明朝");  
    8.删除某节点.//childElement是待删除的节点,parentElement是其父节点  parentElement.remove(childElment);  
    9.添加一个CDATA节点.Element contentElm = infoElm.addElement("content");contentElm.addCDATA(“cdata区域”);  

```
###  节点对象的属性方法操作 
```

1.取得某节点下的某属性    Element root=document.getRootElement();        //属性名name  
         Attribute attribute=root.attribute("id");  
    2.取得属性的文字  
    String text=attribute.getText();  
    3.删除某属性 Attribute attribute=root.attribute("size"); root.remove(attribute);  
    4.遍历某节点的所有属性     
      Element root=document.getRootElement();        
       for(Iterator it=root.attributeIterator();it.hasNext();){          
           Attribute attribute = (Attribute) it.next();           
           String text=attribute.getText();          
           System.out.println(text);    
  }  
    5.设置某节点的属性和文字.   newMemberElm.addAttribute("name", "sitinspring");  
    6.设置属性的文字   Attribute attribute=root.attribute("name");   attribute.setText("csdn");

```
###  遍历XML树 
DOM4J提供至少3种遍历节点的方法：
**1) 枚举(Iterator)**
```

    // 枚举所有子节点
    for ( Iterator i = root.elementIterator(); i.hasNext(); ) {
       Element element = (Element) i.next();
       // do something
    }
    // 枚举名称为foo的节点
    for ( Iterator i = root.elementIterator(foo); i.hasNext();) {
       Element foo = (Element) i.next();
       // do something
    }
    // 枚举属性
    for ( Iterator i = root.attributeIterator(); i.hasNext(); ) {
       Attribute attribute = (Attribute) i.next();
       // do something
    }

```
**2)递归所有节点**
递归也可以采用Iterator作为枚举手段，但文档中提供了另外的做法
```

    public void treeWalk() {
       treeWalk(getRootElement());
    }
    public void treeWalk(Element element) {
       for (int i = 0, size = element.nodeCount(); i < size; i++)     {
           Node node = element.node(i);
           if (node instanceof Element) {
              treeWalk((Element) node);
           } else { // do something....
           }
       }
}

```

**3) Visitor模式**
最令人兴奋的是` DOM4J对Visitor的支持 `，这样可以大大缩减代码量，并且清楚易懂。了解设计模式的人都知道，Visitor是GOF设计模式之一。其主要原理就是**两种类互相保有对方的引用，并且一种作为Visitor去访问许多Visitable。**我们来看DOM4J中的Visitor模式(快速文档中没有提供)
只需要自定一个类` 实现Visitor接口 `即可。
```
 
　       public class MyVisitor extends VisitorSupport {
           public void visit(Element element){
               System.out.println(element.getName());
           }
           public void visit(Attribute attr){
               System.out.println(attr.getName());
           }
        }
        调用：  root.accept(new MyVisitor())

```
Visitor接口提供多种Visit()的重载，根据XML不同的对象，将采用不同的方式来访问。上面是给出的Element和Attribute的简单实现，一般比较常用的就是这两个。VisitorSupport是DOM4J提供的默认适配器
** 注意，这个Visitor是自动遍历所有子节点的。如果是root.accept(MyVisitor)，将遍历子节点。**

##  XPath支持 
DOM4J对XPath有良好的支持，如访问一个节点，可直接用XPath选择。
采用xpath查找需要引入jaxen-xx-xx.jar，否则会报java.lang.NoClassDefFoundError: org/jaxen/JaxenException异常。
```

   public void bar(Document document) {
        List list = document.selectNodes( "//foo/bar" );
        Node node = document.selectSingleNode("/foo/bar/author");
        String name = node.valueOf( "/foo/bar/author/@name" );
     }
 
    例如，如果你想查找XHTML文档中所有的超链接，下面的代码可以实现：
 
    public void findLinks(Document document) throws DocumentException {
        List list = document.selectNodes( //a/@href );
        for (Iterator iter = list.iterator(); iter.hasNext(); ) {
            Attribute attribute = (Attribute) iter.next();
            String url = attribute.getValue();
        }
     }

```
** xpath语法**

**1、选取节点**
XPath 使用路径表达式在 XML 文档中选取节点，节点是沿着路径或者 step 来选取的。
常见的路径表达式：
```

nodename选取当前节点的所有子节点。 
例如bookstore 选取 bookstore 元素的所有子节点。bookstore/book选取bookstore 下名字为 book的所有子元素。

/从根节点选取 
例如/bookstore 选取根元素 bookstore

//从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置 
例如//book选取所有 book 子元素，而不管它们在文档中的位置。bookstore//book选取bookstore 下名字为 book的所有后代元素，而不管它们位于 bookstore 之下的什么位置。//@lang选取所有名为 lang 的属性。

.选取当前节点

..选取当前节点的父节点

```
```

/bookstore/book[1]
选取属于 bookstore 子元素的第一个 book 元素。

/bookstore/book[last()]
选取属于 bookstore 子元素的最后一个 book 元素。

/bookstore/book[last()-1]
选取属于 bookstore 子元素的倒数第二个 book 元素。

/bookstore/book[position()<3]
选取最前面的两个属于 bookstore 元素的子元素的 book 元素。

//title[@lang]
选取所有拥有名为 lang 的属性的 title 元素。

//title[@lang='eng']
选取所有 title 元素，要求这些元素拥有值为 eng 的 lang 属性。

/bookstore/book[price>35.00]
选取所有 bookstore 元素的 book 元素，要求book元素的子元素 price 元素的值须大于 35.00。

/bookstore/book[price>35.00]/title
选取所有 bookstore 元素中的 book 元素的 title 元素，要求book元素的子元素 price 元素的值须大于 35.00

```
**选取未知节点**
XPath 通配符可用来选取未知的 XML 元素。
```

*
匹配任何元素节点

@*
匹配任何属性节点

node()
匹配任何类型的节点

```

实例
```

/bookstore/*
选取 bookstore 元素的所有子节点

//*
选取文档中的所有元素

//title[@*]
选取所有带有属性的 title 元素。

```
**选取若干路径**
通过在路径表达式中使用“|”运算符，您可以选取若干个路径。
```

//book/title | //book/price
选取所有 book 元素的 title 和 price 元素。

//title | //price
选取所有文档中的 title 和 price 元素。

/bookstore/book/title|//price
选取所有属于 bookstore 元素的 book 元素的 title 元素，以及文档中所有的 price 元素。

```

参考：http://www.cnblogs.com/forlina/archive/2011/06/09/2076534.html

针对雅虎天气的实例:
http://weather.yahooapis.com/forecastrss?p=60202
```

weather.setCity( doc.valueOf("/rss/channel/y:location/@city") );
weather.setRegion( doc.valueOf("/rss/channel/y:location/@region") );
weather.setCountry( doc.valueOf("/rss/channel/y:location/@country") );
weather.setCondition( doc.valueOf("/rss/channel/item/y:condition/@text") );
weather.setTemp( doc.valueOf("/rss/channel/item/y:condition/@temp") );
weather.setChill( doc.valueOf("/rss/channel/y:wind/@chill") );
weather.setHumidity( doc.valueOf("/rss/channel/y:atmosphere/@humidity") );

```
###  字符串与XML的转换 
有时候经常要用到字符串转换为XML或反之，
 
```

    // XML转字符串
　 Document document = ...;
    String text = document.asXML();
// 字符串转XML
    String text = James ;
    Document document = DocumentHelper.parseText(text);

```
###  创建XML 
一般创建XML是写文件前的工作，这就像StringBuffer一样容易。
```

    public Document createDocument() {
       Document document = DocumentHelper.createDocument();
       Element root = document.addElement(root);
       Element author1 =
           root
              .addElement(author)
              .addAttribute(name, James)
              .addAttribute(location, UK)
              .addText(James Strachan);
       Element author2 =
           root
              .addElement(author)
              .addAttribute(name, Bob)
              .addAttribute(location, US)
              .addText(Bob McWhirter);
       return document;
    }

```
8. 文件输出
一个简单的输出方法是将一个Document或任何的Node通过write方法输出
```

1.文档中全为英文,不设置编码,直接写入的形式.    
       XMLWriter writer = new XMLWriter(new  FileWriter("ot.xml"));   
       writer.write(document);    
       writer.close();  
2.文档中含有中文,设置编码格式写入的形式.  
       OutputFormat format = OutputFormat.createPrettyPrint();// 创建文件输出的时候，自动缩进的格式                    
       format.setEncoding("UTF-8");//设置编码  
       XMLWriter writer = new XMLWriter(newFileWriter("output.xml"),format);  
       writer.write(document);  
       writer.close();  

```
##  案例 
1.sida.xml描述四大名著的操作，文件内容如下
```

<?xml version="1.0" encoding="UTF-8"?>  
<四大名著>  
    <西游记 id="x001">  
        <作者>吴承恩1</作者>  
        <作者>吴承恩2</作者>  
        <朝代>明朝</朝代>  
    </西游记>  
    <红楼梦 id="x002">  
        <作者>曹雪芹</作者>  
    </红楼梦>  
</四大名著> 

``` 
2.解析类测试操作
```

public class Demo01 {    
    @Test  
    public void test() throws Exception {  
  
        // 创建saxReader对象  
        SAXReader reader = new SAXReader();  
        // 通过read方法读取一个文件 转换成Document对象  
        Document document = reader.read(new File("src/dom4j/sida.xml"));  
        //获取根节点元素对象  
        Element node = document.getRootElement();  
        //遍历所有的元素节点  
        listNodes(node);  
  
        // 获取四大名著元素节点中，子节点名称为红楼梦元素节点。  
        Element element = node.element("红楼梦");  
        //获取element的id属性节点对象  
        Attribute attr = element.attribute("id");  
        //删除属性  
        element.remove(attr);  
        //添加新的属性  
        element.addAttribute("name", "作者");  
        // 在红楼梦元素节点中添加朝代元素的节点  
        Element newElement = element.addElement("朝代");  
        newElement.setText("清朝");  
        //获取element中的作者元素节点对象  
        Element author = element.element("作者");  
        //删除元素节点  
        boolean flag = element.remove(author);  
        //返回true代码删除成功，否则失败  
        System.out.println(flag);  
        //添加CDATA区域  
        element.addCDATA("红楼梦，是一部爱情小说.");  
        // 写入到一个新的文件中  
        writer(document);  
  
    }  
  
    /** 
     * 把document对象写入新的文件 
     *  
     * @param document 
     * @throws Exception 
     */  
    public void writer(Document document) throws Exception {  
        // 紧凑的格式  
        // OutputFormat format = OutputFormat.createCompactFormat();  
        // 排版缩进的格式  
        OutputFormat format = OutputFormat.createPrettyPrint();  
        // 设置编码  
        format.setEncoding("UTF-8");  
        // 创建XMLWriter对象,指定了写出文件及编码格式  
        // XMLWriter writer = new XMLWriter(new FileWriter(new  
        // File("src//a.xml")),format);  
        XMLWriter writer = new XMLWriter(new OutputStreamWriter(  
                new FileOutputStream(new File("src//a.xml")), "UTF-8"), format);  
        // 写入  
        writer.write(document);  
        // 立即写入  
        writer.flush();  
        // 关闭操作  
        writer.close();  
    }  
  
    /** 
     * 遍历当前节点元素下面的所有(元素的)子节点 
     *  
     * @param node 
     */  
    public void listNodes(Element node) {  
        System.out.println("当前节点的名称：：" + node.getName());  
        // 获取当前节点的所有属性节点  
        List<Attribute> list = node.attributes();  
        // 遍历属性节点  
        for (Attribute attr : list) {  
            System.out.println(attr.getText() + "-----" + attr.getName()  
                    + "---" + attr.getValue());  
        }  
  
        if (!(node.getTextTrim().equals(""))) {  
            System.out.println("文本内容：：：：" + node.getText());  
        }  
  
        // 当前节点下面子节点迭代器  
        Iterator<Element> it = node.elementIterator();  
        // 遍历  
        while (it.hasNext()) {  
            // 获取某个子节点对象  
            Element e = it.next();  
            // 对子节点进行遍历  
            listNodes(e);  
        }  
    }  
  
    /** 
     * 介绍Element中的element方法和elements方法的使用 
     *  
     * @param node 
     */  
    public void elementMethod(Element node) {  
        // 获取node节点中，子节点的元素名称为西游记的元素节点。  
        Element e = node.element("西游记");  
        // 获取西游记元素节点中，子节点为作者的元素节点(可以看到只能获取第一个作者元素节点)  
        Element author = e.element("作者");  
  
        System.out.println(e.getName() + "----" + author.getText());  
  
        // 获取西游记这个元素节点 中，所有子节点名称为作者元素的节点 。  
  
        List<Element> authors = e.elements("作者");  
        for (Element aut : authors) {  
            System.out.println(aut.getText());  
        }  
  
        // 获取西游记这个元素节点 所有元素的子节点。  
        List<Element> elements = e.elements();  
  
        for (Element el : elements) {  
            System.out.println(el.getText());  
        }  
  
    }  
  
} 

``` 