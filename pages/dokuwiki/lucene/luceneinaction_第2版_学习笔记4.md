title: luceneinaction_第2版_学习笔记4 

#  LuceneInAction(第2版)学习笔记4之Analyzer分析过程 
**分析Analysis**，在Lucene中指的是**将域(Field)文本转换成最基本的索引表示单元————项(term)**的过程。
 在搜索过程中，这些项用于决定什么样的文档能够匹配查询条件。分析器对分析操作进行了封装，它通过执行若干操作，**将文本转换成语汇单元。**
 这些操作有：
  * 提取单词、去除标点符号、去掉字母上的音调符号、
  * 将字母转换成小写(也称规范化)、去除常用词、
  * 将单词还原为词干形式(词干还原)、将单词转换成基本形式(词形归并lemmatization)。

 分析器的处理过程，也称为**【语汇单元化过程】————tokenization。**
 通过【语汇单元化过程】，从文本流中提取文本块，这些文本块称为 【语汇单元token】。
 
 在建立索引时，**通过分析过程得到的【语汇单元token】就是被索引的项。**
 最重要的是，**只有被索引的项才能被搜索到。(除非该域没有进行分析，则整个域值都被作为一个语汇单元。)**
 ` 【语汇单元token】与它的【域名】结合合，就形成了【项term】。 `
 **【项term】= 【域名】 + 【语汇单元token】**

 使用Lucene时，选择一个合适的分析器是非常关键的。 对分析器的选择没有唯一标准，待分析的语种是影响选择的因素之一。 影响选择的另一因素是被分析的文本所属的领域，不同的行业有不同的术语、缩写词和缩略语。
 注意：不存在能适用于所有情况的分析器。
 
** 【分析器Analyzer】　＝　【分词器Tokenizer】＋【N个过滤器TokenFilter】**
通常情况下，过滤器的工作比切词器简单，**过滤器拿着每个token，决定是继续流转下去或者替换或者抛弃。**
链上的每个过滤器按顺序执行，因此注明**过滤器的顺序显得很重要**。通常情况下，先执行普通过滤器，然后执行专业过滤器。
 
##  1. 使用分析器 
1.1. 分析器说明
 1) 什么时候要用分析器
分析操作会出现在任何需要将文本转换成项的时候。对于Lucene核心来说，**分析操作会出现在两个时间点： 建立索引期间和使用QueryParser对象进行搜索时。**
此外，如果搜索结果要高亮显示被搜索内容，也可能要用于分析操作。当然，高亮显示功能要调用Lucene的两个软件捐赠模块。
 
 2) 分析结果中的语汇单元取决于对应的分析器
不同的分析器分析同样的域值，得到的语汇单元可能是不一样的。

 3) 四个常用分析器说明
A. WhitespaceAnalyzer通过**空格**来分割文本信息，不做其它规范化处理。
B. SimpleAnalyzer通过**非字母字符**来分割文本信息，然后**将语汇单元统一为小写形式**。
` 注意，这两者会去掉数字类型的字符，但会保留其它字符。 `

C. StopAnalyzer
与SimpleAnalyzer类似，区别在于StopAnalyzer会去掉常用单词。默认会去掉常用单词(the a等)，但也可以自己设置常用单词。

D. StandardAnalyzer       是Lucene最复杂的核心分析器。
它包含大量的逻辑操作来识别某些种类的语汇单元，如公司名称、Email地址、主机名称等。它还会将语汇单元转换成小写形式，并去除停用词和标点符号。
 Lucene的分析结果对于搜索用户来说是不可见的。从原始文本中提取的项会被立即编入索引，并且在搜索时用于匹配对应文档。
 当使用QueryParser进行搜索时，Lucene会再次进行分析操作， 这次分析的内容是用户输入的查询语句的文本信息，这样就能确保最佳匹配效果。
 
1.2. 索引过程中的分析
** 在索引期间，文档域值所包含的文本信息需要被转换成语汇单元。**
 程序首先要实例化一个Analyzer对象，然后将之传递给IndexWriter对象。
```

  Analyzer analyzer = new StandardAnalyzer(Version.LUCENE_30);
  IndexWriter writer = new IndexWriter(dir, analyzer, IndexWriter.MaxFieldLength.UNLIMITED);

```
 如果某个域不被分析，则该域值被整个索引成一个单独的语汇单元。
 在用IndexWriter实例索引文档时，一般通过默认的分析器来分析文档中的每个域。
 **如某个文档需要用特殊分析器处理的话，可以针对这个文档指定分析器：**
IndexWriter的addDocument()和updateDocument()都允许为某个文档选择对应的分析器。
 Field.Index.ANALYZED和Field.Index.ANALYXED_NO_NORMS两个参数，保证该域会被分析。
 Field.NOT_Index.ANALYZED和Field.Index.NOT_ANALYXED_NO_NORMS两个参数，保证该域不会被分析，而是将该域值作为一个语汇单元处理。

1.3. QueryParser分析
 QueryParser能很好地为搜索用户提供形式自由的查询。
 为完成这个任务，QueryParser使用分析器将文本信息分割成各个项用于搜索。在实例化QueryParser对象时，同样需要传入一个分析器对象：
```

  QueryParser parser = new QueryParser(Version.LUCENE_30, "contents", analyzer);
  Query query = parser.parse(expression);

```
分析器会接收表达式中连续的独立的文本片段，但不会接收整个表达式。这个表达式可能包含操作符、圆括号或其它表示范围、通配符以及模糊查询在内的特殊表达式语法。
查询时，QueryParser所使用的分析器必须和索引期间使用的分析器相同吗？不一定。
1.4. 解析 VS 分析 : 分析器何时不再适用
 在Lucene内部，分析器用于将域的文本内容单元化，这是分析器的一个重点任务。
 分析器可以用于每次分析一个特定的域，并且将该域内容分解为语汇单元，但是在分析器内不可以创建新的域。
 分析器不能用于从文档中分离和提取域，因为分析器的职责范围只是每次处理一个域。
 为了对域进行分离，要在分析之前预先解析这些文档。
 【解析】： 从文档中分离和提取域，可能会产生多个域
 【分析】： 每次分析一个特定的域，并且将该域内容分解为语汇单元，在分析器内不可以创建新的域
 
##  2. 剖析分析器 
2.1. 分析器的工作原理
 (1) Analyzer类
**` Analyzer类 `是一个抽象类，是所有分析器的基类。它通过` TokenStream类 `，以一种很好的方式将文本逐字转换为` 语汇单元流 `。**
分析器实现TokenStream对象的唯一声明方法是：
public TokenStream tokenStream(String fieldName, Reader reader) 
**返回的TokenStream对象，就用来递归处理所有的语汇单元。**
 
 (2) SimpleAnalyzer类
SimpleAnalyzer分析器是如何工作的？
```

  public final class SimpleAnalyzer extends Analyzer{
  public TokenStream tokenStream(String fieldName, Reader reader){
   return new LowerCaseTokenizer(reader);
   //LowerCaseTokenizer对象依据文本中的非字母字符来分割文本，去掉非字母字符，
   //并且，将字母转换成小写形式
   //非字母字符通过 Character.isLetter()方法来判断
  }
  public TokenStream reusableTokenStream(String fieldName, Reader reader) 
   throws IOException {
   Tokenizer tokenizer = (Tokenizer) getPreviousTokenStream();
   if(tokenizer == null){
    tokenizer = new LowerCaseTokenizer(reader);
    setPreviousTokenStream(tokenizer);
   }else{
    tokenizer.reset(reader);
   }
   return tokenizer;
  }
  //reusableTokenStream()方法是个可选方法，分析器可以实现这个方法，并通过它得到更好地索引效率
  //该方法，可重复利用之前向对应线程返回的同一个TokenStream对象。
  //这种方式，可以节约大量的磁盘分配空间和垃圾搜集开销
  //如果不这样处理，则每个文档的每个域都要重新初始化一个TokenStream对象

  }
 </code>
(3) Analyzer基类的工具类方法
**Analyzer基类实现了两个工具类方法，即setPreviousTokenStream()和getPreviousTokenStream()。**
它们为本机存储线程提供了**针对TokenStream对象的存储和回收功能**。 
Lucene的所有内置分析器都要实现如下方法：
该方法首次被某线程调用时，必须建立并存储一个新的TokenStream对象。
在将该分析器指派给新的Reader对象后，后续调用该分析器时就会返回先前建立的TokenStream对象。
 
2.2. 语汇单元的组成
![](/data/dokuwiki/lucene/pasted/20160228-100443.png)
 (1) 语汇单元内容
 【语汇单元流TokenStream】是分析过程所产生的基本单元。在索引时，Lucene使用特定的分析器来处理需要被语汇单元化的域，而每个语汇单元相关的重要属性随即被编入索引中。每个语汇单元表示一个独立的单词。
** 一个语汇单元携带了一个文本值(单词本身)和其它一些元数据，如：原始文本，从起点到终点的偏移量、语汇单元的类型、以及位置增量。**
 语汇单元可以选择性包括一些由程序定义的标志位和任意字节数的有效负载，这样程序就能根据具体需要来处理这些语汇单元。
 起点偏移量是指语汇单元文本的起始字符在原始文本中的位置， 终点偏移量则表示语汇单元文本终止字符的下一个位置。
 **偏移量对于在搜索结果中用高亮显示匹配的语汇单元非常有用**。
 ` 语汇单元的类型是用String对象表示的，默认值是"word"。 `
 如果需要，可以在语汇单元的过滤过程中控制和利用到其类型属性。
 文本被语汇单元化之后，**相对于前一个语汇单元的位置信息，是以位置增量值保存的。**
 **大多数内置的语汇单元化模块都` 将位置增量的默认值设为1 `，**表示所有语汇单元都是**连续的，在位置上是一个接一个的。同义词用0表示**
 每个语汇单元还有多个选择标志，一个标志即32bit。Lucene的内置分析器不能使用这些标志，但自己设计的搜索程序是可以使用它们的。此外，每个语汇单元都能以byte[]形式记录在索引中，用以指向有效负载。
 
 (2) 语汇单元转换成项
 当文本在索引过程中进行分析后，每个语汇单元都作为一个项被传递给索引。位置增量、起点和终点偏移量和有效负载是语汇单元携带到索引中的唯一附加元数据。**而语汇单元的类型和标志位都被抛弃了**————它们只在分析过程中使用。
 
 (3) 位置增量
 位置增量使得当前语汇单元和前一个语汇单元在位置上关联起来。**一般来说位置增量为1，表示每个单词存在于域中唯一且连续的位置上。**
 **位置增量因子会直接影响短语查询和跨度查询，因为这些查询需要知道域中各个项之间的距离。**
 如果**位置增量大于1，**则允许语汇单元之间**有空隙**，可以用这个空隙来表示被删除的单词。
 **位置增量为0**的语汇单元表示将该语汇单元放置在前一个语汇单元的位置上。**同义词**分析器可以通过0增量来表示插入的同义词。
 这个做法使得Lucene在进行短语查询时，输入任意一个同义词都能匹配到同一结果。
 
2.3. 语汇单元流揭秘
2.3.1. 语汇单元流组成
 ` TokenStream `是一个能在被调用后产生**语汇单元序列**的类。
 它有两种不同的类型：
  * Tokenizer类 : 通过java.io.Reader对象读取字符并**创建语汇单元** 
  * TokenFilter类 : 封装了另外一个TokenStream抽象类的组合模式(该抽象可以是另外一个TokenFilter类).TokenFilter负责处理输入的语汇单元，然后通过增、删、改属性的方式来产生新的语汇单元
 这两个类都是从TokenStream继承而来。
 **当分析器从它的tokenStream()方法或reusableTokenStream()方法返回tokenStream对象后，它就开始用一个tokenizer对象创建初始语汇单元序列，然后再链接任意数量的tokenFilter对象来修改这些语汇单元。**
 这些称为**分析器链(analyzer chain)**。 如：
Reader --> Tokenizer --> TokenFilter --> TokenFilter --> TokenFilter --> Tokens
分析器链以一个Tokenizer对象开始，通过Reader对象读取字符并产生初始语汇单元，然后用任意数量的链接TokenFilter对象修改这些语汇单元。
【语汇单元流TokenStream】 = 【分词器Tokenizer】 + N个【语汇过滤器TokenFilter】
分析器Tokenizer类和语汇过滤器TokenFilter类，两个类都是从TokenStream继承而来
 
2.3.2. Lucene的核心Tokenizer和TokenFilter
 1) TokenStream : 抽象Tokenizer基类
 2) Tokenizer : 输入参数为Reader对象的TokenStream子类
 3) 【CharTokenizer】: 基于字符的Tokenizer父类，包含抽象方法isTokenChar()
**当isTokenChar()为true时，输出连续的语汇单元块，它还能将字符规范化处理，如转为小写形式**，输出的语汇单元所包含的最大字符数为255
 4) WhitespaceTokenizer :isTokenizer()为true时的CharTokenizer类，用于处理所有非空格字符
 5) **【KeywordTokenizer】 : 将输入的整个字符串转换为一个语汇单元**
 6) LetterTokenizer  : isTokenChar()为true，且Charcter.isLetter为true时的CharTokenizer类
 7) LowerCaseTokenizer : 将所有字符规范化处理为小写形式的LetterTokenizer类
 8) SinkTokenizer : Tokenizer子类，用于吸收语汇单元，能将语汇单元缓存至一个私有列表中，并能递归访问该列表中的语汇单元.该类与TeeTokenizer联合使用时，用于"拆分"TokenStream对象。
** 9) 【Standard` To `kenizer】: 复杂而基于语法的语汇单元产生器，用于输出高级别类型的语汇单元。如：Email地址。每个输出的语汇单元标记为一个特殊类型。**
 10) TokenFilter : 输入参数为另一个TokenStream子类的TokenStream子类。 
 11) LowerCaseFilter : 将语汇单元转成小写形式
 12) StopFilter : 移除指定集合中的停用词
 13) PorterStemFilter : 利用Poster词干提取算法将语汇单元还原为词干
 14) TeeTokenFilter : 将语汇单元写入SinkTokenizer对象，完成对TokenStream对象的拆分，该类还会返回未被修改的语汇单元
 15) ASCIIFoldingFilter: 将带音调的字符映射为不带音调的字符
 16) CachingTokenFilter: 存储从输入字符流中提取的所有语汇单元，调用该类的reset()方法后能重复以上处理
 17) LengthFilter : 支持特定长度的语汇单元
** 18) StandardFilter :  接收一个StandardTokenizer对象作为参数， 用于去掉缩略词中的点号，或者在带有单引号的单词中去掉's**
 

2.3.3. 分析器链示例
<code java>
 public TokenStream tokenStream(String fieldName, Reader reader) {
  return new StopFilter(true, new LowerCaseTokenizer(reader), stopWords);
 }

```
在该分析器中，LowerCaseTokenizer对象会通过Reader对象输出原始语汇单元集，然后将这个语汇单元集传递给StopFilter对象的初始化方未能。LowerCaseTokenizer对象所输出的语汇单元对应于原始文本中的邻接字母，同时还会将这些字母转为小写形式。语汇单元的边界以非字母字符来区分，同时，这些非字母字符不会出现在语汇单元中。
 **经过语汇单元转换为小写形式后，StopFilter会调用positionIncrement()方法，该方法查询停用词列表，移除语汇单元中的停用词。**
 
2.3.4. 语汇单元流其它功能
 在实现TokenStream抽象类时，加入缓存功能是必要的。底层的Tokenizer对象使用缓存来存储字符，以便在遇到空格和非字母等边界时生成语汇单元。TokenFilter对象在向数据流中输出附加的语汇单元时，需要将当前语汇单元和附加语汇单元进行排列，并且，一次只能输出一个语汇单元。大多数Lucene内置的TokenFilter类都对输入的语汇单元流进行了某种程度的修改，而其中一个SeeSinkTokenFilter类更是如此。
任意该过滤器会将输入的语汇单元流复制成任意数量的拷贝发送到sink输出流和标准输出流中， 每个sink输出流都可以用于进一步处理。该功能在一些情况下很好用： 当两个或多个域共享同一个初始化分析步骤，但相互之间在语汇单元的最终处理过程中又存在差异。
 
2.4. 观察分析器
 通常情况下，分析过程所产生的语汇单元会在毫无提示的情况下用于索引操作。然后，如能跟踪到语汇单元的生成，则能对分析过程有一个具体的了解。

2.4.1. AnalyzerDemo : 分析操作实例
```

 public class AnalyzerDemo {
  private static final String[] examples = {
   "The quick brown fox jumped over the lazy dog",
   "XY&Z Corporation - xyz@example.com"
  };
  private static final Analyzer[] analyzers = new Analyzer[] {
   new WhitespaceAnalyzer(),
   new SimpleAnalyzer(),
   new StopAnalyzer(Version.LUCENE_30),
   new StandardAnalyzer(Version.LUCENE_30)
  };
  public static void main(String[] args) throws IOException {
   String[] strings = examples;
   if(args.length >0){
    strings=args;//分析处理的命令行参数
   }
  }
  private static void analyze(String text) throws IOException {
   System.out.println("Analyzing \"" + text + "\"");
   for(Analyzer analyzer : analyzers) {
    String name = analyzer.getClass().getSimpleName();
    System.out.println(" " + name + ":");
    System.out.println("    ");
    AnalyzerUtils.displayTokens(analyzer, text); //执行实际操作
    //AnalyzerUtils中，分析器用于处理提取出来的文本和语汇单元
    System.out.println("\n");
   }
  }
 }

```
2.4.2. AnalyzerUtils : 深入研究分析器
```

 public static void displayTokens(Analyzer analyzer, 
     String text) throws IOException {
  displayTokens(analyzer.tokenStream("contents", new StringReader(text)));//调用分析过程
 }
 public static void displayTokens(TokenStream stream) throws IOException {
  TermAttribute term = stream.addAttribute(TermAttribute.class);
  white(stream.incrementToken()){
   System.out.println("[" + term.term() + "]");   //输出带括号的语汇单元
  }
 }

```
 
2.4.3. 深入分析语汇单元
** TokenFilter对象会访问和修改输入的语汇单元流，但语汇单元流是由哪些属性组成的呢？**为更好的说明，我为在AnalyzerUtils类中加了一个名为displayTokensWithFullDetails()的方法。
 该方法，**可以观察各个语汇单元的项、偏移量、类型和位置增量。**
```

 public static void displayTokensWithFullDetails(Analyzer analyzer, String text) throws IOException{
  TokenStream stream = analyzer.tokenStream("contents", new StringReader(text));//执行分析
  //获取有用属性
  TermAttribute term = stream.addAttribute(TermAttribute.class);
  PositionIncrementAttribute posIncr = stream.addAttribute(PositionIncrementAttribute.class);
  OffsetAttribute offset = stream.addAttribute(OffsetAttribute.class);
  TypeAttribute type = stream.addAttribute(TypeAttribute.class);
  //递归处理所有语汇单元
  int position = 0;
  while(stream.incrementToken()){
   //计算位置信息并打印
   int increment = posIncr.getPositionIncrement();
   if(increment>0){
    position = position + increment;
    System.out.println();
    System.out.print(position + ":");
   }
   //打印所有语汇单元细节信息
   System.out.print("[" +
     term.term() + ":" + 
     offset.startOffset() + "->" + 
     offset.endOffset() + ":" +
     type.type() + "]");

  }
  System.out.println();
 }

```
            
2.4.4. 语汇单元的属性
** 注意，TokenStream永远不会显式创建包含所有语汇单元属性的单个对象。它会分别与语汇单元每个元素(项、偏移量、位置增量)对应的可重用属性接口进行交互。**
` TokenStream继承类AttributeSource并生成子类，AttributeSource是一个实用方法。 `
它能在程序非运行期间提供增强类型并且能够完全扩展的属性操作，这给我们带来了很好的运行性能。Lucene在分析期间使用某些预定义的属性，但还是可以自由加入自己的属性，方法是创建实现Attribute接口的具体类。
注意：Lucene在索引期间不会对这些新属性作任何操作。
Lucene内置的语汇单元属性：
  * TermAttribute 语汇单元对应的文本
  * PositionIncrementAttribute 位置增量，默认值为1
  * OffsetAttribute 起始字符和终止字符的偏移量
  * TypeAttribute 语汇单元类型，默认为单词
  * FlagsAttribute 自定义标志位
  * PayloadAttribute 每个语汇单元的byte[]类型有效负载

2.4.5. 起始和结束位置偏移量有什么好处？
它们记录的是初始字符在语汇单元中的偏移量，它们并不属于Lucene的核心功能。并且，它们对于各个语汇单元来说，只是不透明的整数，你可以在这里设置任意的整数。
 
2.4.6. 语汇单元类型的作用
你可以通过语汇单元的类型值将语汇单元指明为某种特定类型 的词汇。
**StandardAnalyzer是唯一一个能影响语汇单元类型的内置分析器。**默认情况下，**Lucene并不会将语汇单元的类型编入索引，因此，该类型只会在分析时使用。**
 
 
2.5. 语汇单元过滤器：过滤顺序的重要性
对于某些TokenFilter子类来说，在分析过程中对事件的**处理顺序**是非常重要的。
每个步骤也许都必须一览前一个步骤才能完成。移除停用词的处理就是一个很好的例子。StopFilter类在停用词集合中区分大小写地查对每一个语汇单元，这个步骤就依赖于输入小写形式的语汇单元。

到目前为止，你应该对分析过程的内部机制有了一个坚实的掌握。分析器会简单地定义一个语汇单元链，该链的开端 是新语汇单元tokenStream的初始数据源。其后跟随任意数量的用于修改语汇单元的tokenFilter子类。
语汇单元包含一个属性集，Lucene会通过各种不同的方法来存储该集合。
 
##  3. 使用内置分析器 
3.1. 常见内置分析器的功能
Lucene包含了一些内置分析器，它们由某些内置的语汇单元 Tokenizer 和 TokenFilter 联合组成。
  * WhitespaceAnalyzer 根据空格拆分语汇单元
  * SimpleAnalyzer       根据非字母字符拆分文本，并将其转换为小写形式
  * **StopAnalyzer           根据非字母字符拆分文本，然后小写化，再移除停用词**
  * **KeywordAnalyzer    将整个文本作为一个单一语汇单元处理**
  * **StandardAnalyzer   基于复杂的语法来生成语汇单元，该语法能识别email地址、首字母缩写词、汉语、日语、韩语字符、字母数字等。还能完成小写转换和移除停用词。**
这些内置分析器，几乎可以用于分析所有的西方(主要是欧洲)的语言。
 
3.2. StopAnalyzer
StopAnalyzer分析器除了完成基本的单词拆分和小写化功能之外，还负责移除一些称之为停用词(stop words)的特殊单词。
**停用词是指较为通用的词，如the，它对于搜索来说，词义单一，几乎每个文档都会包含这样的词。**StopAnalyzer类内置了一个常用英文停用词集合，该集合为` ENGLISH_STOP_WORDS_SET。 `
StopAnalyzer还有一个可重载的构造方法，允许你通过它传入自己的停用词集合。初始化StopAnalyzer时，系统会在后台创建一个StopFilter对象以完成过滤功能。

3.3. StandardAnalyzer
StandardAnalyzer是公认的最实用的Lucene内置分析器。该分析器基于JFlex-based语法操作，能够将相关类型的词汇准确地转化为语汇单元。
StandardAnalyzer还能通过以StopAnalyzer相同的机制移除停用词。它们使用同一个默认的英文停用词集合，也可以通过重载方法选择自定义的停用词集合。因此，StandardAnalyzer是你不二的选择。

3.4. 应当采用哪种核心分析器
大多数应用程序都不使用任意一种内置分析器，而是选择创建自己的分析器链。对于使用核心分析器的应用程序来说，StandardAnalyzer可能是最常用的选择。某个包含部分数字列表的域可能会使用 WhitespaceAnalyzer 。
事实上，Solr能够轻松创建自定义的分析链，方法是在solrconfig.xml文件中，以XML格式直接定义分析链表达式。
 
##  4. 近音词查询，自定义近音词Analyzer 

**近音词算法： Metaphone或Soundex**
通过Apache Commons Codec项目中的Metaphone算法实现。
```

 public void testKoolKat() throws Exception {
    RAMDirectory directory = new RAMDirectory();
    Analyzer analyzer = new MetaphoneReplacementAnalyzer();
    IndexWriter writer = new IndexWriter(directory, analyzer, true,
                                         IndexWriter.MaxFieldLength.UNLIMITED);
    Document doc = new Document();
    doc.add(new Field("contents", //#A
                      "cool cat",
                      Field.Store.YES,
                      Field.Index.ANALYZED));
    writer.addDocument(doc);
    writer.close();
    IndexSearcher searcher = new IndexSearcher(directory);
    Query query = new QueryParser(Version.LUCENE_30,  //#B
                                  "contents", analyzer)    //#B
                              .parse("kool kat");          //#B
    TopDocs hits = searcher.search(query, 1);
    assertEquals(1, hits.totalHits);   //#C
    int docID = hits.scoreDocs[0].doc;
    doc = searcher.doc(docID);
    assertEquals("cool cat", doc.get("contents"));   //#D
    searcher.close();
  }
  public static void main(String[] args) throws IOException {
    MetaphoneReplacementAnalyzer analyzer =
                                 new MetaphoneReplacementAnalyzer();
    AnalyzerUtils.displayTokens(analyzer,
                   "The quick brown fox jumped over the lazy dog");
    System.out.println("");
    AnalyzerUtils.displayTokens(analyzer,
                   "Tha quik brown phox jumpd ovvar tha lazi dag");//两者完全一致，这就是近音词算法。
    
  }

```
```

public class MetaphoneReplacementAnalyzer extends Analyzer {
  public TokenStream tokenStream(String fieldName, Reader reader) {
    return new MetaphoneReplacementFilter(
                   new LetterTokenizer(reader));
  }
}

```
```

public class MetaphoneReplacementFilter extends TokenFilter {
  public static final String METAPHONE = "metaphone";

  private Metaphone metaphoner = new Metaphone();
  private TermAttribute termAttr;
  private TypeAttribute typeAttr;

  public MetaphoneReplacementFilter(TokenStream input) {
    super(input);
    termAttr = addAttribute(TermAttribute.class);
    typeAttr = addAttribute(TypeAttribute.class);
  }

  public boolean incrementToken() throws IOException {
    if (!input.incrementToken())                    //#A  
      return false;                                 //#A  

    String encoded;
    encoded = metaphoner.encode(termAttr.term());   //#B转换为Metaphone编码
    termAttr.setTermBuffer(encoded);                //#C使用编码文本覆盖
    typeAttr.setType(METAPHONE);                    //#D设置语汇单元类型，默认为"word"
    return true;
  }
}

```
##  5. 同义词、别名和其它表示相同意义的词 
同义词分析器，主要是位置增量为0的同义词。同义词是双向的。只有在索引期间或搜索期间才需要扩展同义词，如在两个处理阶段内都进行同义词扩展则是对系统资源的浪费。
如果在索引期间进行同义词的扩展，则索引所占用的磁盘空间会稍大一些，但这能加快搜索速度。因为这样使得被访问的搜索项更少。如果同义词被编入索引，则不能随意快速地更改它们，也不能在搜索期间随意观察这些改变所带来的效果。如果在搜索期间进行同义词的扩展，则能在测试时很快观察到这种改变带来的效果。
```

 private IndexSearcher searcher;
  private static SynonymAnalyzer synonymAnalyzer =
                      new SynonymAnalyzer(new TestSynonymEngine());

  public void setUp() throws Exception {
    RAMDirectory directory = new RAMDirectory();

    IndexWriter writer = new IndexWriter(directory,
                                         synonymAnalyzer,  //#1  
                                         IndexWriter.MaxFieldLength.UNLIMITED);
    Document doc = new Document();
    doc.add(new Field("content",
                      "The quick brown fox jumps over the lazy dog",
                      Field.Store.YES,
                      Field.Index.ANALYZED));  //#2
    writer.addDocument(doc);
                                  
    writer.close();

    searcher = new IndexSearcher(directory, true);
  }

  public void tearDown() throws Exception {
    searcher.close();
  }

  public void testJumps() throws Exception {
    TokenStream stream =
      synonymAnalyzer.tokenStream("contents",                   // #A
                                  new StringReader("jumps"));   // #A
    TermAttribute term = stream.addAttribute(TermAttribute.class);
    PositionIncrementAttribute posIncr = stream.addAttribute(PositionIncrementAttribute.class);

    int i = 0;
    String[] expected = new String[]{"jumps",              // #B
                                     "hops",               // #B
                                     "leaps"};             // #B
    while(stream.incrementToken()) {
      assertEquals(expected[i], term.term());

      int expectedPos;      // #C
      if (i == 0) {         // #C
        expectedPos = 1;    // #C
      } else {              // #C
        expectedPos = 0;    // #C
      }                     // #C
      assertEquals(expectedPos,                      // #C
                   posIncr.getPositionIncrement());  // #C
      i++;
    }
    assertEquals(3, i);
  }

```
```

public class SynonymAnalyzer extends Analyzer {
  private SynonymEngine engine;

  public SynonymAnalyzer(SynonymEngine engine) {
    this.engine = engine;
  }

  public TokenStream tokenStream(String fieldName, Reader reader) {
    TokenStream result = new SynonymFilter(
                          new StopFilter(true,
                            new LowerCaseFilter(
                              new StandardFilter(
                                new StandardTokenizer(
                                 Version.LUCENE_30, reader))),
                            StopAnalyzer.ENGLISH_STOP_WORDS_SET),
                          engine
                         );
    return result;
  }
}


```
```

public class SynonymFilter extends TokenFilter {
  public static final String TOKEN_TYPE_SYNONYM = "SYNONYM";

  private Stack<String> synonymStack; //同义词临时缓存栈
  private SynonymEngine engine;
  private AttributeSource.State current;

  private final TermAttribute termAtt;
  private final PositionIncrementAttribute posIncrAtt;

  public SynonymFilter(TokenStream in, SynonymEngine engine) {
    super(in);
    synonymStack = new Stack<String>();                     //#1 
    this.engine = engine;

    this.termAtt = addAttribute(TermAttribute.class);
    this.posIncrAtt = addAttribute(PositionIncrementAttribute.class);
  }

  public boolean incrementToken() throws IOException {
    if (synonymStack.size() > 0) {                          //#2
      String syn = synonymStack.pop();                      //#2
      restoreState(current);                                //#2
      termAtt.setTermBuffer(syn);
      posIncrAtt.setPositionIncrement(0);                   //#3
      return true;
    }

    if (!input.incrementToken())                            //#4  
      return false;

    if (addAliasesToStack()) {                              //#5 
      current = captureState();                             //#6
    }

    return true;                                            //#7
  }

  private boolean addAliasesToStack() throws IOException {
    String[] synonyms = engine.getSynonyms(termAtt.term()); //#8
    if (synonyms == null) {
      return false;
    }
    for (String synonym : synonyms) {                       //#9
      synonymStack.push(synonym);
    }
    return true;
  }
}

/*
#1 Define synonym buffer
#2 Pop buffered synonyms
#3 Set position increment to 0
#4 Read next token
#5 Push synonyms onto stack
#6 Save current token
#7 Return current token
#8 Retrieve synonyms
#9 Push synonyms onto stack
*/


```
```

public class TestSynonymEngine implements SynonymEngine {
  private static HashMap<String, String[]> map = new HashMap<String, String[]>();

  static {
    map.put("quick", new String[] {"fast", "speedy"});
    map.put("jumps", new String[] {"leaps", "hops"});
    map.put("over", new String[] {"above"});
    map.put("lazy", new String[] {"apathetic", "sluggish"});
    map.put("dog", new String[] {"canine", "pooch"});
  }

  public String[] getSynonyms(String s) {
    return map.get(s);
  }
}

```
##  6. 词干分析 
 词干算法，也称词干提取器，是对英文单词中较常见的，因时态、语态、复数格等原因引起的词尾变化进行移除的处理过程。
即：经过词干算法的处理，**每个单词的各种形式都会被还原为其词干形式。**
**词干算法有 Porter算法、Snowball算法等。**

##  7. 域分析 
7.1.** 多值域分析**
多值域：一个域有多个值，存储时，每个值都存放在相同名称的域实例中。文档可能包含同名的多个Field实例，而Lucene在索引过程中，在逻辑上将这些域的语汇单元按照顺序附加在一起。
**默认情况下，getPositionIncrementGap()方法返回0，表示无间隙，则它在运行时，会认为各个域值是连接在一起的。**如果该值加大，则位置查询就不会错误地在各个域值边界进行匹配了。
还有一点很重要，那就是，要确保正常计算多值域的语汇单元偏移量。即：如果程序不需要对各个域值连接处的文本进行匹配操作，则可以设置位置增量。
 
###  7.2. 特定域分析--不同的域要使用不同的分析器来分析 
在索引操作期间，对于分析器选择的粒度为 IndexWriter 或文档级别。如果使用QueryParser，则程序会只用一个分析器来处理文本。
**但有时，不同的域要使用不同的分析器来分析。**在程序内部，分析器能轻易地处理正在被处理的域名，因为这个域名是作为参数传入其tokenStream方法的。
Lucene内置的分析器却不能扩展这种能力，因为它被设计用来处理一般情况，而域名则是与应用程序有关的。不过，可以很容易创建一个自定义分析器来处理这种情况。
作为可选，**Lucene的内置工具类 ` PerFieldAnalyzerWrapper `，这个类，可以针对每个域使用不同的分析器。**
```

//在创建 PerFieldAnalyzerWapper 时，要提供默认的分析器。
PerFieldAnalyzerWrapper analyzer = new PerFieldAnalyzerWrapper(new SimpleAnalyzer());
//要使用不同的分析器处理域时，可以调用其addAnalyzer()方法
//其它没有指定的域，则使用默认的分析器
analyzer.addAnalyzer("body", new StandardAnalyzer(Version.LUCENE_30));

```
7.3. 搜索未被分析的域
**有多种解决方案，其中最简单的方案就是使用 PerFieldAnalyzerWapper 。**

##  8. 语言分析 
8.1. 分析不同语言的文本时要处理的问题
在分析不同语言的文本时，必须面临以下几个问题：
**首先要正确设置字符的编码方式，**从而使Java能够顺利地读取诸如文件等外部数据。在分析过程中，**每种语言都有其不同的停用词集合，以及特有的词干分析算法。**而且，根据每种语言的不同需要，可能还要把一些声调符号从字符中去掉。最后，如果事先不知道是何种语言，则程序还要对此判断。
至于，如何利用Lucene提供的基本模块处理上述问题，最终要由开发者决定。此外，在Lucene的contrib目录和互联网上，还有诸如Tokenizer和TokenStream等大量的分析器和附加模块可用。
 
8.2. Unicode与字符编码
在Lucene内部，所有的字符都是以标准的UTF-8编码存储的。Java会在字符串对象内对Unicode编码进行自动处理，用UTF-16格式表示字符。但，外部文本传递给Java和Lucene时的编码处理，Lucene是不负责的，而必须由自己负责。
 
8.3. 非英语语种分析
在处理非英文文本时，原来分析过程中用到的各种细节仍然适用。分析文本的目的就是从中提取出所有项。对于西方语言来说，分割单词使用的是空格和标点。
所以，必须把西方语言中的停用词列表以及词干还原算法，调整为针对特定语种的文本，以便对之进行分析。
 
8.4. 字符规范化处理
Lucene提供了一套字符过滤类，用来表示基于语汇单元的相关功能。Lucene提供的单一的CharFilter核心子类，称为MappingCharFilter，该类允许你接收输入和输出字符串。Lucene的核心分析器都没有字符过滤功能，必须创建自己的分析器。该分析器以一个CharReader作为开端，后跟任意数量的CharFilter，之后，再建立Tokenizer和TokenFilter链。
 
8.5. 亚洲语种分析
对于汉日韩(CJK)等亚洲语种来说，一般使用表意文字而不是使用由字母组成的单词。这些象形字符不一定通过空格来分隔，所以需要使用一种完全不同的分析方法来识别和分隔语汇单元。
**StandardAnalyzer是Lucene内置的唯一能够处理亚洲语种的分析器。**该分析器可以将一定范围内的Unicode编码识别为CJK字符，并将它们拆分为独立的语汇单元。
**Lucene的contrib目录中有3个分析器适合处理亚洲语种：CJKAnalyzer、ChineseAnalyzer和` Smart ChineseAnalyzer `。**
![](/data/dokuwiki/lucene/pasted/20160228-104031.png) 
```

public class ChineseDemo {
  private static String[] strings = {"道德經"};  //A

  private static Analyzer[] analyzers = {
    new SimpleAnalyzer(),
    new StandardAnalyzer(Version.LUCENE_30),
    new ChineseAnalyzer (),    //B
    new CJKAnalyzer (Version.LUCENE_30),
    new SmartChineseAnalyzer (Version.LUCENE_30)   
  };

  public static void main(String args[]) throws Exception {

    for (String string : strings) {
      for (Analyzer analyzer : analyzers) {
        analyze(string, analyzer);
      }
    }

  }

  private static void analyze(String string, Analyzer analyzer)
         throws IOException {
    StringBuffer buffer = new StringBuffer();

    TokenStream stream = analyzer.tokenStream("contents",
                                              new StringReader(string));
    TermAttribute term = stream.addAttribute(TermAttribute.class);

    while(stream.incrementToken()) {   //C
      buffer.append("[");
      buffer.append(term.term());
      buffer.append("] ");
    }

    String output = buffer.toString();

    Frame f = new Frame();
    f.setTitle(analyzer.getClass().getSimpleName() + " : " + string);
    f.setResizable(true);

    Font font = new Font(null, Font.PLAIN, 36);
    int width = getWidth(f.getFontMetrics(font), output);

    f.setSize((width < 250) ? 250 : width + 50, 75);

    // NOTE: if Label doesn't render the Chinese characters
    // properly, try using javax.swing.JLabel instead
    Label label = new Label(output);   //D
    label.setSize(width, 75);
    label.setAlignment(Label.CENTER);
    label.setFont(font);
    f.add(label);

    f.setVisible(true);
  }

  private static int getWidth(FontMetrics metrics, String s) {
    int size = 0;
    int length = s.length();
		for (int i = 0; i < length; i++) {
      size += metrics.charWidth(s.charAt(i));
    }

    return size;
  }
}

/* 	
#A Analyze this text
#B Test these analyzers
#C Retrieve tokens
#D Display analysis
*/

```
8.6. 有关非英语语种分析的其它问题
当在同一索引中处理各种不同语种时，所碰到的主要障碍是：如何处理文本编码。这时，StandardAnalyzer仍然是Lucene中最好的通用内置分析器，它甚至还可以用于处理CJK字符；
不过，**捐赠的SmartChineseAnalyzer类似乎更适用于对中文进行分析。**当希望把由多种语言构成的文档编入同一索引时，最好为每个文档都指定相应的分析器。
最后要介绍的是语种检测，和字符编码一样，它也超出了Lucene所涉及的范围。但它对搜索程序来说很重要，而它也是当前搜索技术中较为活跃的研究领域之一。
 
9. Nutch分析
我们不可能得到Google的源代码，但可以得到与之相似的开源项目Nutch的源代码。它是由Lucene的开发者Doug Cutting开发的。Nutch对文本的分析方式很有趣，它对停用词做了一些特殊处理，在Nutch中将停用词称为常用项common terms。如果在查询中使用了常用项，并且该项不是包含在带有其它修饰词或引号的某个短语中，则程序将忽略这些常用词。Nutch把【索引期间】进行分析所使用的【二元语法技术】与【查询期间】【对短语的优化技术】结合在一起。这种结合可以大大减小搜索时需要考虑的文档范围。

##  自定义AnalyzerUtils工具类3.x版本 
```

public class AnalyzerUtils {
  public static void displayTokens(Analyzer analyzer,
                                   String text) throws IOException {
    displayTokens(analyzer.tokenStream("contents", new StringReader(text)));  //A
  }

  public static void displayTokens(TokenStream stream)
    throws IOException {

    TermAttribute term = stream.addAttribute(TermAttribute.class);
    while(stream.incrementToken()) {
      System.out.print("[" + term.term() + "] ");    //B
    }
  }
  /*
    #A Invoke analysis process
    #B Print token text surrounded by brackets
  */

  public static int getPositionIncrement(AttributeSource source) {
    PositionIncrementAttribute attr = source.addAttribute(PositionIncrementAttribute.class);
    return attr.getPositionIncrement();
  }

  public static String getTerm(AttributeSource source) {
    TermAttribute attr = source.addAttribute(TermAttribute.class);
    return attr.term();
  }

  public static String getType(AttributeSource source) {
    TypeAttribute attr = source.addAttribute(TypeAttribute.class);
    return attr.type();
  }

  public static void setPositionIncrement(AttributeSource source, int posIncr) {
    PositionIncrementAttribute attr = source.addAttribute(PositionIncrementAttribute.class);
    attr.setPositionIncrement(posIncr);
  }

  public static void setTerm(AttributeSource source, String term) {
    TermAttribute attr = source.addAttribute(TermAttribute.class);
    attr.setTermBuffer(term);
  }

  public static void setType(AttributeSource source, String type) {
    TypeAttribute attr = source.addAttribute(TypeAttribute.class);
    attr.setType(type);
  }

  public static void displayTokensWithPositions
    (Analyzer analyzer, String text) throws IOException {

    TokenStream stream = analyzer.tokenStream("contents",
                                              new StringReader(text));
    TermAttribute term = stream.addAttribute(TermAttribute.class);
    PositionIncrementAttribute posIncr = stream.addAttribute(PositionIncrementAttribute.class);

    int position = 0;
    while(stream.incrementToken()) {
      int increment = posIncr.getPositionIncrement();
      if (increment > 0) {
        position = position + increment;
        System.out.println();
        System.out.print(position + ": ");
      }

      System.out.print("[" + term.term() + "] ");
    }
    System.out.println();
  }

  public static void displayTokensWithFullDetails(Analyzer analyzer,
                                                  String text) throws IOException {

    TokenStream stream = analyzer.tokenStream("contents",                        // #A
                                              new StringReader(text));

    TermAttribute term = stream.addAttribute(TermAttribute.class);        // #B
    PositionIncrementAttribute posIncr =                                  // #B 
    	stream.addAttribute(PositionIncrementAttribute.class);              // #B
    OffsetAttribute offset = stream.addAttribute(OffsetAttribute.class);  // #B
    TypeAttribute type = stream.addAttribute(TypeAttribute.class);        // #B

    int position = 0;
    while(stream.incrementToken()) {                                  // #C

      int increment = posIncr.getPositionIncrement();                 // #D
      if (increment > 0) {                                            // #D
        position = position + increment;                              // #D
        System.out.println();                                         // #D
        System.out.print(position + ": ");                            // #D
      }

      System.out.print("[" +                                 // #E
                       term.term() + ":" +                   // #E
                       offset.startOffset() + "->" +         // #E
                       offset.endOffset() + ":" +            // #E
                       type.type() + "] ");                  // #E
    }
    System.out.println();
  }
  /*
    #A Perform analysis
    #B Obtain attributes of interest
    #C Iterate through all tokens
    #D Compute position and print
    #E Print all token details
   */

  public static void assertAnalyzesTo(Analyzer analyzer, String input,
                                      String[] output) throws Exception {
    TokenStream stream = analyzer.tokenStream("field", new StringReader(input));

    TermAttribute termAttr = stream.addAttribute(TermAttribute.class);
    for (String expected : output) {
      Assert.assertTrue(stream.incrementToken());
      Assert.assertEquals(expected, termAttr.term());
    }
    Assert.assertFalse(stream.incrementToken());
    stream.close();
  }

  public static void displayPositionIncrements(Analyzer analyzer, String text)
    throws IOException {
    TokenStream stream = analyzer.tokenStream("contents", new StringReader(text));
    PositionIncrementAttribute posIncr = stream.addAttribute(PositionIncrementAttribute.class);
    while (stream.incrementToken()) {
      System.out.println("posIncr=" + posIncr.getPositionIncrement());
    }   
  }

  public static void main(String[] args) throws IOException {
    System.out.println("SimpleAnalyzer");
    displayTokensWithFullDetails(new SimpleAnalyzer(),
        "The quick brown fox....");

    System.out.println("\n----");
    System.out.println("StandardAnalyzer");
    displayTokensWithFullDetails(new StandardAnalyzer(Version.LUCENE_30),
        "I'll email you at xyz@example.com");
  }
}

/*
#1 Invoke analysis process
#2 Output token text surrounded by brackets
*/

```
lucene4.x参考：
```

Version matchVersion = Version.LUCENE_XY; // Substitute desired Lucene version for XY
    Analyzer analyzer = new StandardAnalyzer(matchVersion); // or any other analyzer
    TokenStream ts = analyzer.tokenStream("myfield", new StringReader("some text goes here"));
    OffsetAttribute offsetAtt = ts.addAttribute(OffsetAttribute.class);
    
    try {
      ts.reset(); // Resets this stream to the beginning. (Required)
      while (ts.incrementToken()) {
        // Use AttributeSource.reflectAsString(boolean)
        // for token stream debugging.
        System.out.println("token: " + ts.reflectAsString(true));

        System.out.println("token start offset: " + offsetAtt.startOffset());
        System.out.println("  token end offset: " + offsetAtt.endOffset());
      }
      ts.end();   // Perform end-of-stream operations, e.g. set the final offset.
    } finally {
      ts.close(); // Release resources associated with this stream.
    }

```
Lucene provides seven Attributes out of the box:
  * CharTermAttribute 	The term text of a token. Implements CharSequence (providing methods length() and charAt(), and allowing e.g. for direct use with regular expression Matchers) andAppendable (allowing the term text to be appended to.)
  * OffsetAttribute	The start and end offset of a token in characters.
  * PositionIncrementAttribute	See above for detailed information about position increment.
  * PositionLengthAttribute	The number of positions occupied by a token.
  * PayloadAttribute	The payload that a Token can optionally have.
  * TypeAttribute	The type of the token. Default is 'word'.
  * FlagsAttribute	Optional flags a token can have.
  * KeywordAttribute	Keyword-aware TokenStreams/-Filters skip modification of tokens that return true from this attribute's isKeyword() method.
参考：http://blog.csdn.net/liuweitoo/article/details/8137415