title: android官方文档学习之解析xml 

#  android官方文档学习之解析XML 
参考：
官方文档
http://blog.csdn.net/liuhe688/article/details/6415593

PULL解析器的运行方式和SAX类似，都是基于事件的模式。不同的是，在PULL解析过程中，我们需要自己获取产生的事件然后做相应的操作，而不像SAX那样由处理器触发一种事件的方法，执行我们的代码。PULL解析器小巧轻便，解析速度快，简单易用，非常适合在Android移动设备中使用，Android系统内部在解析各种XML时也是用PULL解析器。
我们推荐` XmlPullParser `，其在android中对XML的解析是高效且可维护的。从过去来看，android已经拥有该接口的两个实现：
  * ` KXmlParser，通过XmlPullParserFactory.newPullParser()创建 `。
  * ` ExpatPullParser，通过Xml.newPullParser()创建。 `

我会在项目的assets目录中放置一个XML文档books.xml，内容如下：
```

<?xml version="1.0" encoding="utf-8"?>  
<books>  
    <book>  
        <id>1001</id>  
        <name>Thinking In Java</name>  
        <price>80.00</price>  
    </book>  
    <book>  
        <id>1002</id>  
        <name>Core Java</name>  
        <price>90.00</price>  
    </book>  
    <book>  
        <id>1003</id>  
        <name>Hello, Andriod</name>  
        <price>100.00</price>  
    </book>  
</books>  

```
解析技术解析文档，得到一个List<Book>的对象，先来看一下Book.java的代码：
```

public class Book {  
    private int id;  
    private String name;  
    private float price;  
      
    public int getId() {  
        return id;  
    }  
  
    public void setId(int id) {  
        this.id = id;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public float getPrice() {  
        return price;  
    }  
  
    public void setPrice(float price) {  
        this.price = price;  
    }  
  
    @Override  
    public String toString() {  
        return "id:" + id + ", name:" + name + ", price:" + price;  
    }  
}

```
最后，我们还要把这个集合对象中的数据生成一个新的XML文档：
生成的XML结构跟原始文档略有不同，是下面这种格式：
```

<?xml version="1.0" encoding="UTF-8"?>  
<books>  
  <book id="1001">  
    <name>Thinking In Java</name>  
    <price>80.0</price>  
  </book>  
  <book id="1002">  
    <name>Core Java</name>  
    <price>90.0</price>  
  </book>  
  <book id="1003">  
    <name>Hello, Andriod</name>  
    <price>100.0</price>  
  </book>  
</books>  

```
接下来，就该介绍操作过程了，**我们先为解析器定义一个BookParser接口，每种类型的解析器需要实现此接口**。BookParser.java代码如下：
```

public interface BookParser {  
    /** 
     * 解析输入流 得到Book对象集合 
     * @param is 
     * @return 
     * @throws Exception 
     */  
    public List<Book> parse(InputStream is) throws Exception;  
      
    /** 
     * 序列化Book对象集合 得到XML形式的字符串 
     * @param books 
     * @return 
     * @throws Exception 
     */  
    public String serialize(List<Book> books) throws Exception;  
}  

```
使用PULL解析器：
PullBookParser.java代码如下：
```

public class PullBookParser implements BookParser {  
      
    @Override  
    public List<Book> parse(InputStream is) throws Exception {  
        List<Book> books = null;  
        Book book = null;  
          
//      XmlPullParserFactory factory = XmlPullParserFactory.newInstance();  
//      XmlPullParser parser = factory.newPullParser();  
          
        XmlPullParser parser = Xml.newPullParser(); //由android.util.Xml创建一个XmlPullParser实例  
        parser.setInput(is, "UTF-8");               //设置输入流 并指明编码方式  
  
        int eventType = parser.getEventType();  
        while (eventType != XmlPullParser.END_DOCUMENT) {  
            switch (eventType) {  
            case XmlPullParser.START_DOCUMENT:  
                books = new ArrayList<Book>();  
                break;  
            case XmlPullParser.START_TAG:  
                if (parser.getName().equals("book")) {  
                    book = new Book();  
                } else if (parser.getName().equals("id")) {  
                    eventType = parser.next();  
                    book.setId(Integer.parseInt(parser.getText()));  
                } else if (parser.getName().equals("name")) {  
                    eventType = parser.next();  
                    book.setName(parser.getText());  
                } else if (parser.getName().equals("price")) {  
                    eventType = parser.next();  
                    book.setPrice(Float.parseFloat(parser.getText()));  
                }  
                break;  
            case XmlPullParser.END_TAG:  
                if (parser.getName().equals("book")) {  
                    books.add(book);  
                    book = null;      
                }  
                break;  
            }  
            eventType = parser.next();  
        }  
        return books;  
    }  
      
    @Override  
    public String serialize(List<Book> books) throws Exception {  
//      XmlPullParserFactory factory = XmlPullParserFactory.newInstance();  
//      XmlSerializer serializer = factory.newSerializer();  
          
        XmlSerializer serializer = Xml.newSerializer(); //由android.util.Xml创建一个XmlSerializer实例  
        StringWriter writer = new StringWriter();  
        serializer.setOutput(writer);   //设置输出方向为writer  
        serializer.startDocument("UTF-8", true);  
        serializer.startTag("", "books");  
        for (Book book : books) {  
            serializer.startTag("", "book");  
            serializer.attribute("", "id", book.getId() + "");  
              
            serializer.startTag("", "name");  
            serializer.text(book.getName());  
            serializer.endTag("", "name");  
              
            serializer.startTag("", "price");  
            serializer.text(book.getPrice() + "");  
            serializer.endTag("", "price");  
              
            serializer.endTag("", "book");  
        }  
        serializer.endTag("", "books");  
        serializer.endDocument();  
          
        return writer.toString();  
    }  
}  

```
PULL轻巧灵活，速度快，占用内存小，使用非常顺手。读者也可以根据自己的喜好选择相应的解析技术.