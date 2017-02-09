title: fastjson 

#  Json处理之fastJson 
项目地址：https://github.com/alibaba/fastjson
安装：
```

<dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>fastjson</artifactId>
     <version>1.2.7</version>
</dependency>

```
fastjson 是一个性能很好的 Java 语言实现的 JSON 解析器和生成器，来自阿里巴巴的工程师开发。
主要特点：
  * 快速FAST (比其它任何基于Java的解析器和生成器更快，包括jackson）
  * 强大（支持普通JDK类包括任意Java Bean Class、Collection、Map、Date或enum）
  * 零依赖（没有依赖其它任何类库除了JDK）

示例代码：
```

import com.alibaba.fastjson.JSON;
Group group = new Group();
group.setId(0L);
group.setName("admin");
 
User guestUser = new User();
guestUser.setId(2L);
guestUser.setName("guest");
 
User rootUser = new User();
rootUser.setId(3L);
rootUser.setName("root");
 
group.getUsers().add(guestUser);
group.getUsers().add(rootUser); 
String jsonString = JSON.toJSONString(group); 
System.out.println(jsonString);

```

##  主要接口 

fastjson入口类是` com.alibaba.fastjson.JSON `，主要的API是**JSON.toJSONString，和parseObject。**
```

        public static final String toJSONString(Object object);
        public static final <T> T  parseObject(String text, Class<T> clazz, Feature... features);
	public static <T> T	   parseObject(String input, Type clazz, Feature... features) 
	public static <T> List<T>	parseArray(String text, Class<T> clazz) 
	public	static List<Object>	parseArray(String text, Type[] types) 
        public static final String   toJSONStringWithDateFormat(Object object, String dateFormat, SerializerFeature... features) 

```
```

序列化：

  String jsonString = JSON.toJSONString(obj);
反序列化：

  VO vo = JSON.parseObject("...", VO.class);
泛型反序列化：

  import com.alibaba.fastjson.TypeReference;

  List<VO> list = JSON.parseObject("...", new TypeReference<List<VO>>() {});

```

##  fastjson-android 

fastjson有专门的for android版本，去掉不常用的功能。jar占的字节数更小。git branch地址是：https://github.com/alibaba/fastjson/tree/android 。
##  使用@JSONField定制序列化 
JSONField 介绍
```

package com.alibaba.fastjson.annotation;

public @interface JSONField {
    // 配置序列化和反序列化的顺序，1.1.42版本之后才支持
    int ordinal() default 0;

     // 指定字段的名称
    String name() default "";

    // 指定字段的格式，对日期格式有用
    String format() default "";

    // 是否序列化
    boolean serialize() default true;

    // 是否反序列化
    boolean deserialize() default true;
}

```
JSONField配置方式：FieldInfo可以配置在getter/setter方法或者字段上。
```

 public class A {
      private int id;

      @JSONField(name="ID")
      public int getId() {return id;}
      @JSONField(name="ID")
      public void setId(int value) {this.id = id;}
 }

```
```

配置在field上

 public class A {
      @JSONField(name="ID")
      private int id;

      public int getId() {return id;}
      public void setId(int value) {this.id = id;}
 }

```
使用format配置日期格式化
```

 public class A {
      // 配置date序列化和反序列使用yyyyMMdd日期格式
      @JSONField(format="yyyyMMdd")
      public Date date;
 }

```
使用serialize/deserialize指定字段不序列化
```

 public class A {
      @JSONField(serialize=false)
      public Date date;
 }

 public class A {
      @JSONField(deserialize=false)
      public Date date;
 }

```

##  fastjson处理日期 
fastjson处理日期的API很简单，例如：
JSON.toJSONStringWithDateFormat(date, "yyyy-MM-dd HH:mm:ss.SSS")

使用ISO-8601日期格式
JSON.toJSONString(obj, SerializerFeature.UseISO8601DateFormat);
  
全局修改日期格式
JSON.DEFFAULT_DATE_FORMAT = "yyyy-MM-dd";
JSON.toJSONString(obj, SerializerFeature.WriteDateUseDateFormat);

反序列化能够自动识别如下日期格式：
  * ISO-8601日期格式
  * yyyy-MM-dd
  * yyyy-MM-dd HH:mm:ss
  * yyyy-MM-dd HH:mm:ss.SSS
  * 毫秒数字
  * 毫秒数字字符串
  * .NET JSON日期格式
  * new Date(198293238)


##  处理超大JSON文本 
请参考：https://github.com/alibaba/fastjson/wiki/Stream-api

##  FastJson SerializerFeature特性配置 
参考:http://blog.csdn.net/zgmzyr/article/details/8478563
SerializerFeature枚举包含的特性：
DisableCheckSpecialChar：一个对象的字符串属性中如果有特殊字符如双引号，将会在转成json时带有反斜杠转移符。如果不需要转义，可以使用这个属性。默认为false 
QuoteFieldNames———-输出key时是否使用双引号,默认为true 
WriteMapNullValue——–是否输出值为null的字段,默认为false 
WriteNullNumberAsZero—-数值字段如果为null,输出为0,而非null 
WriteNullListAsEmpty—–List字段如果为null,输出为[],而非null 
WriteNullStringAsEmpty—字符类型字段如果为null,输出为”“,而非null 
WriteNullBooleanAsFalse–Boolean字段如果为null,输出为false,而非null

##  FastJson循环引用问题 
参考：http://www.tuicool.com/articles/6bUz6fz
https://github.com/alibaba/fastjson/wiki/%E5%BE%AA%E7%8E%AF%E5%BC%95%E7%94%A8
http://www.tuicool.com/articles/vUJBFn
fastjson支持循环引用，并且是缺省打开的。但是当存在实体多对多的情况下，就会导致问题。问题如图，第二个获取不到，而是一个循环引用。
![](/data/dokuwiki/opensourcelearn/pasted/20151220-160029.png)
```

public class Test {
  public static void main(String[] args) {
  Map<String, Student> maps = new HashMap<String, Student>();
  Student s1 = new Student("s1", 16);

  maps.put("s1", s1);
  maps.put("s2", s1);

  byte[] bytes = JSON.toJSONBytes(maps);

  System.out.println(new String(bytes));
  }
}

```
输出：
{"s1":{"age":16,"name":"s1"},"s2":{"$ref":"$.s1"}}
可以看到，这个json如果发到前端是无法使用的，幸好FastJson提供了解决办法，我们来看下，解决办法为禁用循环引用检测，代码如下：
```

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;

public class Test {
  public static void main(String[] args) {
  Map<String, Student> maps = new HashMap<String, Student>();
  Student s1 = new Student("s1", 16);

  maps.put("s1", s1);
  maps.put("s2", s1);
  
  SerializerFeature feature = SerializerFeature.DisableCircularReferenceDetect;

  byte[] bytes = JSON.toJSONBytes(maps,feature);

  System.out.println(new String(bytes));
  }
}

```
输出如下：
{"s1":{"age":16,"name":"s1"},"s2":{"age":16,"name":"s1"}}
全局关闭
```

  JSON.DEFAULT_GENERATE_FEATURE |= SerializerFeature.DisableCircularReferenceDetect.getMask();

```
非全局关闭
```

JSON.toJSONString(obj, SerializerFeature.DisableCircularReferenceDetect);

```
` 但是这样配置之后导致栈溢出，不推荐。解决办法请看后面 `
Spring配置：但是这样配置之后导致栈溢出，不推荐。解决办法请看后面
```

<!-- 增加控制器注解支持 -->
	<mvc:annotation-driven>
		<mvc:message-converters register-defaults="true">
			<!-- 配置Fastjson支持 -->
			<bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
				<property name="supportedMediaTypes">
					<list>
						<value>text/html;charset=UTF-8</value>
						<value>application/json</value>
					</list>
				</property>
				<property name="features">
					<list>
						<value>WriteDateUseDateFormat</value>
						<value>WriteMapNullValue</value>
						<value>QuoteFieldNames</value>
			
					</list>
				</property>
			</bean>
		</mvc:message-converters>
	</mvc:annotation-driven>

```

**推荐解决办法：禁用循环引用字段的JSON序列化**
@JSONField(serialize =false)
```

@Entity
public class ArticleCategory implements Serializable{
	private static final long serialVersionUID = -119533369936136078L;
	@Id
	@GeneratedValue(generator = "system-uuid")
	@GenericGenerator(name = "system-uuid", strategy = "uuid")
	@Column(name = "category_id")
	private String categoryId;
	private String categoryName;
	@ManyToMany(fetch=FetchType.EAGER)
	@JoinTable(name = "article_category_rel", joinColumns = { @JoinColumn(name = "cid") }, inverseJoinColumns = {
			@JoinColumn(name = "aid") })
	@JSONField(serialize =false)
	private Set<Article> articleSet;

```
```

@Entity
public class Article implements Serializable{
	private static final long serialVersionUID = -5893522771824415416L;
	
	@Id
    @GeneratedValue(generator = "system-uuid")
    @GenericGenerator(name = "system-uuid", strategy = "uuid")
	@Column(name="article_id")
	private String articleId;
	@DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss")
	private Timestamp articlePublishTime;
	private String articleTitle;
	private String articleAuthor;
	private String articleSummary; //文章摘要
	private String articleContent;
	private String isTop;//是否置顶
	private String categoryString;
	private String keywordString;
  
	@ManyToMany(mappedBy="articleSet",fetch=FetchType.EAGER)
	private Set<ArticleKeyword> keywordSet;//文章关键字
	@ManyToMany(mappedBy="articleSet",fetch=FetchType.EAGER)

```