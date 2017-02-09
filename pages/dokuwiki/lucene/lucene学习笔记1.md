title: lucene学习笔记1 

#  Lucene4.x Field与FieldType说明 
![](/data/dokuwiki/lucene/pasted/20160302-095829.png)
![](/data/dokuwiki/lucene/pasted/20160302-102109.png)

##  二、Field类 
1、  类的说明
在一般情况下为Document对象创建一个Field对象会使用它的子类，比如：
IntField,LongField, FloatField, DoubleField, BinaryDocValuesField, StringField, TextField, NumericDocValuesField, SortedDocValuesField, StoredField。而不是使用它自己。
一个Field是Document的一部分，每一个Field有三部分组成，分别是：名 称、类型和值。值可以是文本（String类型，Reader类型或者是预分享的TokenStream）,二进制（byet[]）,或者是数字（一个 Number类型）Field是可以存储在索引中的，以便日后返回这个文档。
需要注意的是：**这个Field 是一个实现了IndexableFieldType接口的类，**修改IndexableFieldType的状态将影响字段的使用。**强烈建议不要在实例化Field对象后修改IndexableFieldType实例**。**可以通过在创建Field的配置类FieldType时调用FieldType的方法freeze().该方法的解释是：阻止未来改变，推荐在自定义FieldType时调用，去预防无意的状态改变。**
` 在Field子类中设置FieldType时都调用了freeze()方法 `，代码如下：
```

        TYPE_NOT_STORED.setIndexed(true);
        TYPE_NOT_STORED.setOmitNorms(true);
        TYPE_NOT_STORED.setIndexOptions(IndexOptions.DOCS_ONLY);
        TYPE_NOT_STORED.setTokenized(false);
        TYPE_NOT_STORED.freeze(); //阻止查创建Field对象之后的未来改变。

```
上面是StringField中的一段代码，代码的最后调用了freeze()方法。

2、  构造函数
public Field(String name, Reader reader, FieldType type)
public Field(String name, TokenStream tokenStream,FieldType type)
public Field(String name, byte[] value, FieldType type)
public Field(String name, byte[] value, int offset, int length, FieldType type)
public Field(String name, BytesRef bytes, FieldType type)
public Field(String name, String value, FieldType type)
其中传byte[]参数的构造函数需要主要的是：byte[]参数使用不是复制(也就是使用的是对象引用)，所以在Field完成前，不要修改它。
**Field还有一些构造函数，在4.4版本中是不赞成使用的。**
在上面所列的Field构造函数的最后一个参数都是**FieldType类型，这个是Field重要的辅助类**，稍后会记录。
3、  内部类
Field类里面还包含三个枚举类型的内部类，分别是：Store、Index和TermVector，**其中Index和TermVector在4.4版本中也是不赞成使用的，**那我们就光看Store吧，它包含两个枚举值，YES和NO,就是表示Field是否存储时使用。

##  三、StringField类 
1、  类的概述：
**StringField是Field类的一个子类**，其实就是在原有Field类的基础上 添加了FieldType字段，官方说明是：**一个索引但不分词的Field**，通过构造传过来的String值会是一个单独的Token，也就是会把这个字 符串当成一个完整的词来进行索引，官方还举了一些例子，如：一个地名或ID还有path(路径)都不需要分词。你打算使用排序或访问通过字段缓存。
2、  StringField类的源码
通过看这个类的源码你会一目了然
StringField定义了两个FieldType对象
一个为索引存储但不分词。另一个是索引不存储不分词，
这两个都是通过一个静态代码块儿来完成初始化的，**通过看这个FieldType对象的设置，我们以后要定义FieldType时就可以参考这个了。**
最后一个构造函数来调用父类的构造函数。
![](/data/dokuwiki/lucene/pasted/20160302-103728.png)


###  四、TextField类 
1、  类的概述
跟StringField一样，**TextField类**也是一个Field类的子类，也是包含了多个FieldType对象，官方说明是：**这个Field是一个索引分词**，不包含term vectors，例如将被用到“body”属性，包含大量文本的Document中。
##  五、StoredField类 
同StringField类一样，官方说明是：**一个字段的值被存储，但不分词，不索引**，所以可以通过
IndexSearcher.doc和IndexReader.document获得这个Field的值。

##  六、IntField类 
官方说明：int类型的Field在一个有效的范围内过滤和排序，下面是一个例子：
document.add(new IntField(name, 6, Field.Store.NO));
为了获得最佳性能，使用一个IntField和Document实例来保存多个文档，例子如下：

七、LongField类
基本与IntField一样，例子也一样，就是把IntField改成LongField。
八、DoubleField类
与IntField和Long一样。
九、FloatField类
十、NumericDocValuesField类
1、 类概述
官方概述：这个Field用于每个Document添加一个long类型的值用于评分、排序或者索引值，例子如下：
document.add(new NumericDocValuesField(name, 22L));
如果你需要去存储这个值，你应该添加一个单独的StoredField实例。

##  十一、FieldType类 
1、  类的概述
该类主要是配置Field类来使用的，例如：**是否储存，是否索引，是否分词**等。
还有一个重要的方法**freeze()，用来阻止Field在实例化后完成前修改。**
2、  内部类
**FieldType类包含一个内部的枚举类型NumericType来表示数字类**型，值分别是，INT,LONG,FlOAT,DOUBLE。
3、  **FieldType还用到了一个IndexOptions枚举类型**，他有四个值，分别是：
DOCS_ONLY文档只包含索引，位置和词频将忽略，查询短语和位置信息将引起异常
DOCS_AND_FREQS文档只包含索引和词频，位置将忽略，这个可以正常评分，查询短语和位置也将引发异常
DOCS_AND_FREQS_AND_POSITIONS文档包含索引位置和词频，这是一个典型的默认为全文搜索:全部启用评分和位置查询的支持。
DOCS_AND_FREQS_AND_POSITIONS_AND_OFFSETS文档包含索引、位置、词频和偏移量。
至此Field类的情况就基本上介绍完了。有什么不对或者不完善的地方欢迎指正。


参考：http://my.oschina.net/MrMichael/blog/220681