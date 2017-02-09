title: luceneinaction_第2版_学习笔记2 

#  LuceneInAction(第2版)学习笔记2之构建索引 
##  1. 文档和域 
1.1.文档和域的关系
文档是Lucene索引和搜索的原子单位。
 **文档为包含一个或多个域的容器，而域则依次包含“真正的”被搜索内容。**每个域都有一个标识名称，该名称为一个文本值或二进制值。将一个文档加入到索引中时，**可以通过一系列选项来控制Lucene的行为**。在对原始数据进行索引时，得先将数据转换成Lucene所能识别的文档和域。在随后的搜索过程中，被搜索对象则为域值。

1.2. Lucene可以针对域进行3种操作
1) 域值可以被索引或不被索引。
要搜索一个域，则要先对该域进行索引。` 被索引的域值必须是文本格式，二进制格式的域值只能被存储而不被索引。 `域值 ==>分析后得到语汇单元 ==> 将语汇单元加入到索引中
2) 域被索引后，可以选择性地存储项向量。项向量可以看做是该域的一个小型反向索引集合，通过该向量能够检索该域的所有语汇单元。这个机制可以实现一些高级功能，如搜索与当前文档相似的文档。
3) 域值可以被单独存储。是指被分析前的域值备份可以写进索引中，以便后续的检索。这机制将使你可以将原始域值展现给用户，如文档的标题或摘要。
有时，可能会需要使用杂项域，即包含所有文本的一个独立域以供搜索。

##  2. Lucene与数据库的区别 
A. Lucene没有一个确定的全局模式
即：
 **加入索引的每个文档都是独立的，它与之前加入的文档完全没有关系；**文档可以包含任意的域，以及任意的索引、存储和项的向量操作选项；**文档也可以不必包含与其它文档相同的域**，甚至跟其它文档可以只有相关操作选项有所区别。这种特性可以随时对文档进行索引，而不必提前设计文档的数据结构表；如果随后想向文档中添加域，则可以完成添加后重新索引该文档或重建索引即可；
 Lucene这种灵活的架构意味着**单一的索引可以包含表示不同实体的多个文档。**
 
B. Lucene要求你在进行索引操作时简单化或反向规格化原始数据
反向规格化Donormalization: 解决有关文档真实结构和Lucene表示能力之间的“不匹配”问题

##  3. Lucene的索引过程 
从原始文档 提取文本并建立文档 --> 分析文档 --> 文档索引(向索引添加文档)
 
A. 提取文本并建立文档
HTML ==> 【 Extract Text 】 ==> 文本
PDF  ==> 【 Extract Text 】 ==> 文本
Word ==> 【 Extract Text 】 ==> 文本
提关提取文本信息的细节可结合**Tika框架详述**，使用该框架能使你很轻易地从各种格式的文件中提取文本信息。一旦提取出预想的文本信息，并建立起对应的包含各个域的文档后，下一步就是对这些文本信息进行分析了。
 
B. 分析文档
分析文档是调用**IndexWriter对象的addDocument**方法将数据传递给Lucene进行索引操作。
索引操作时，Lucene首先分析文本，将文本数据分割成语汇单元，然后对它们执行一些可选操作。
在索引前，对语汇单元的操作：
 1) 统一转换为小写，以使搜索不对大小写敏感；
 2) 调用StopFilter停词过滤类，从输入中去掉一些使用很频繁即没有实际意义的词；
 3) 去掉词干；
 4) 然后调用一系列filter过滤类，以便修正语汇单元；
以上这些操作，构成了分析器。
此外，还可以通过链接Lucene的语汇单元和filter来搭建自己的分析器，或通过其它自定义方式来搭建分析器。
分析过程会产生大批的语汇单元，随后这些语汇单元将被写入索引文件中。
 
C. 文档索引(向索引添加文档)
对输入数据分析完毕后，就可以将分析结果写入索引文件中。Lucene将输入数据以一种**倒排索引(inverted index)的数据结构进行存储。**
在进行关键字快速查找时，这种数据结构能够有效利用磁盘空间。Lucene使用倒排数据结构的原因是：把文档中提取出的语汇单元作为查询关键字，而不是将文档作为中心实体。
倒排索引结构类似于图书的索引与页码的对应关系。倒排索引不是回答【这个文档中包含哪些单词？】，而是经过优化后用来快速回答【哪些文档包含单词x？】”。
现在所有的Web搜索引擎核心都是采用倒排索引技术。
**Lucene的索引文件目录有唯一一个段结构，即索引段**。**可以将一个段看作一个子索引**，尽管每个段都不是一个完全独立的索引。
Lucene的优势之一就是**支持增量索引**(Incremental indexing)，而这个功能主要是靠索引分段实现。
 
##  4. 基本索引操作: 添加、删除、更新索引文档Document 
4.1. 添加索引文档
```

 Directory dir = null;
//取Writer
 private IndexWriter getWriter() throws IOException{
  return new IndexWriter(dir, new WhitespaceAnalyzer(), IndexWriter.MaxFieldLength.UNLIMITED);//IndexWriter构造函数的三个参数：索引存放目录，分析器，域的最大长度(域截取)
 }
 protected void setup() throws Exception{
  dir = new RAMDirectory();
  IndexWriter writer = getWriter();
  for(...){
   Document doc = new Document();
   doc.add(new Field("id", "1", Field.Stores.YES, Field.Index.NOT_ANALYZED));//Field构造函数的四个参数：域名，域值，域的存储状态(是否存储)，域的索引状态(是否分析)
   writer.addDocument(doc);
  }
  writer.commit();//或 writer.close();
 }
 protected int getHitCount(String fieldName, String searchString) throws Exception{
  IndexSearcher searcher = IndexSearcher(dir);
  Term t = new Term(fieldName, searchString);
  Query query = new TermQuery(t);
  int hitCount = searcher.search(query, 1).totalHits;
  searcher.close();
  return hitCount;
 }

``` 
4.2. 删除索引中的文档
```

 protected void DelDoc() throws Exception{
  IndexWriter writer = getWriter();
  writer.deleteDocument(new Term("id", "1"));//删除文档
  //writer.hasDeletions(); 是否包含被标记为已删除的文档
  //writer.maxDoc(); //返回索引中被删除和未被删除的文档总数
  //writer.numDocs(); //返回索引中未被删除的文档总数
  writer.optimize();//优化操作使用删除生效,并强制Lucene在删除一个文档后合并索引段
  writer.commit();
  //或 writer.close();
 }

```
 
4.3. 更新索引中的文档
```

 protected void ModDoc() throws Exception{
  IndexWriter writer = getWriter();
  Document doc = new Document();
  doc.add(new Field("id", "1", Field.Stores.YES, Field.Index.NOT_ANALYZED));
  writer.updateDocument(new Term("id", "1"), doc);//更新文档
  
  //或 writer.close();
 }

``` 
 updateDocument = 先调用 deleteDocument() + 再调用 addDocument();

5. 域选项
Field类是在文档索引期单最重要的类，该类控制着被索引的域值。
域选项有：**索引选项、存储选项、项向量使用选项。**

5.1. 域索引选项** Field.Index.***
 通过倒排索引来控制域文本是否可被搜索。
 Field.Index.ANALYZED  【会分析域值】使用分析器将域值分解成独立的语汇单元流，并使每个语汇单元能被搜索。
 Field.Index.NOT_ANALYZED 【不分析域值】对域进行索引，但不对String值进行分析; 将域值作为单一语汇单元并使之能被搜索; 适用于不能被分解的域值，如：URL、文件路径、日期、人名、社保号码、电话号码等。
 Field.Index.ANALYZED_NO_NORMS   【会分析域值，不存储norms】不在索引中存储**norms信息(加权相关)**
 Field.Index.NOT_ANALYZED_NO_NORMS 【不分析域值，不存储norms】不在索引中存储norms信息
 Field.Index.NO   【使对应的域值不被搜索】
 norms信息记录了索引中的index-time boost 信息，但是当搜索时，可能会比较耗费内存。

5.2. 域存储选项 **Field.Store.***
 **用来确定是否需要存储域的真实值，以便后续搜索时能恢复这个值**。
 Field.Store.YES  【指定存储域值】**建议不要存储太大的域值，因为会消耗掉索引的存储空间**
 Field.Store.NO  【指定不存储域值】
 可以使用Lucene的CompressionTools类中的方法对要存储的域值进行压缩和解压，但该方法会降低索引和搜索速度。
 这其实就是通过消耗更多CPU计算能力来换取更多的磁盘空间，这要仔细权衡。如果域值较小，建议少用压缩。
 
5.3. 域的项向量选项： 【索引域】  【项向量】  【存储域】
 **项向量是介于索引域和存储域的一个中间结构**。
 项向量 ` TermVector `
 
5.4. 域的其它初始化选项： Reader/TokenStream/byte[]
```

 Field(String name, Reader value, TermVector termVector)
  Reader的域值不能被存储 Store.NO，且该域会一直用于分析和索引 Index.ANALYZED
 Field(String name, Reader value)
  TermVector为TermVector.NO
 Field(String name, TokenStream tokenStream, TermVector termVector)
  允许程序对域值进行预分析并生成TokenStream对象。此域不会被存储 Store.NO，会有索引Index.ANALYZED
 Field(String name, TokenStream tokerStream)
  TermVector为TermVector.NO
 Field(String name, byte[] value, Store store)
  采用二进制域，不参与索引 Index.NO，没有项向量 TermVector.NO，会被存储 Store.YES
 Field(String name, byte[] value, int offset, int length, Store store)
  引用二进制的部分片段

```
 
5.5. 域选项组合(常见组合)
索引选项      存储选项 项向量      使用场合
Index.NOT_ANALYZED_**NO_NORMS** Store.NO TermVector.NO     标识符，文件名，主键，URL，日期，用于排序的文本域
Index.ANALYZED      **Store.YES** TermVector.WITH_POSITIONS_OFFSETS 文档标题、摘要
Index.ANALYZED      Store.NO TermVector.WITH_POSITIONS_OFFSETS 文档正文(数据量大，故不存储)
Index.NO      Store.YES TermVector.NO     文档类型、不要搜索的数据库主键
Index.NOT_ANALYZED     Store.NO TermVector.NO     隐藏的关键字
 
5.6. 域排序选项
当Lucene返回匹配搜索条件的文档时，**一般是按照默认评分对文档进行排序的**。当然，也可以自定义排序，前提是要先正确地完成对域的索引。
注意：用于排序的域是必须进行索引的，而且每个对应文档必须包含一个语汇单元。
但为了实现域排序功能，你必须首先正确地完成对域的索引。**如果域是数值类型**，在将它加入文档和进行排序时，要用` NumbericField `类来表示
 
5.7. **多值域**
作者域，当作者数多于一个时，该如何处理？
方法一： 依次处理每个作者名字，将它们加入单个String，然后建立对应的域。
方法二： 向这个域中写入几个不同的值，如：
```

 Document doc = new Document();
 for(String author : authors){
  doc.add(new Field("author", author, Field.Store.YES, Field.Index.ANALYZED));
 }

```
鼓励使用第二种方式，因为这是逻辑上具有多个域值的表达方式。在Lucene中，只要文档出现同名的多值域，倒排索引和项向量都会在逻辑上将这些域的语汇单元附加进去，具体顺序由添加该域的顺序决定。
 
##  6. 对文档和域进行**加权操作** ：  
提升文档或域的评分，默认的加权值为1.0
 加权操作可以在索引期间完成，也可以在搜索期间完成。
6.1. 文档加权操作
 对邮件进行索引和搜索时，如果发送人邮件是在本公司，则要排在更重要的位置，否则，排在不重要的位置。
```

 Document doc = new Document();
 doc.add(new Field("senderEmail", senderEmail, Field.Store.YES, Field.Index.Not_ANALYZED));
 //...
 if(isImportant(senderEmail)){
  doc.setBoost(1.5F);  //重要：加权因子为1.5
 }else if(isUnimportang(senderEmail)){
  doc.setBoost(0.1F);  //不重要：加权因子为0.1
 }
 //其它，采用默认的加权因子： 1.0
 writer.addDocument(doc);

```
 
6.2. 对域加权操作
 当加权一个文档时，Lucene在内部采用同一个**加权因子**来对该文档中的域进行加权。同上节的例子，如何才能使用邮件的主题变得比邮件的作者更重要呢？即：
 搜索时，如何才能让主题域变得比作者域更重要呢？为达到这个目地，可以使用` Field类的setBoost() `方法。Field subjectField = new Field("subject" subject, Field.Store.YES, Field.Index.ANALYZED);
 subjectField.setBoost(1.2F);
注意：当你改变一个域或一个文档的加权因子时，必须完全删除并创建对应的文档，或者使用updateDocument方法来实现。另外：较短的域有一个隐含的加权，这取决于Lucene的评分算法具体实现。
加权操作是一项高级操作，很多搜索程序没有它也能正常运行，所以使用加权的时候要小心。**Lucene的评分机制包含大量的因子，其中就有加权因子。**
**Lucene如何将加权因子写入索引呢？这就是属于` norms `的范畴了。**
 
6.3. 加权基准(Norms)
 **norms经常面临的问题之一就是它在搜索期间的高内存量**。这是因为norms的全部数组要加载到RAM，并且，要对被搜索文档的每个域分配一个字节空间。如果域较多，则加载操作会很快占用大量的RAM空间。
 当然，你也可以关掉norms相关操作，方法是使用Field.Index中的**NO_NORMS索引选项**，或者，在对包含该域的文档进行索引前调用Field.setOmitNorms(true)方法，该操作会影响评分效果。
 因为上面的操作，**使得搜索期间程序不会处理索引时的加权信息。**注意，含Norms选项索引进行到一半时关闭Norms选项，则必须对整个索引进行重建。
 因为，即使只有一个文档域在索引时包含了norms，则在随后的段合并操作中，这个情况会“扩散”， 使得所有文档都会占用一个字节的norms空间，发生这种情况主要是因为Lucene并不针对norms进行松散存储。
 
##  7. 索引数字、日期和时间 
7.1. 索引数字
A.** 数字内嵌在将要索引的文本中**，而想保留这些数字，并将它们作为单独的语汇单元处理，这样搜索过程就可以用到。` 只要选择一个不丢弃数字的分析器即可，如： WhitespaceAnalyzer和StandardAnalyzer ` 。 注意： **SimpleAnalyzer和StopAnalyzer两个类会将语汇单元流中的数字剔除。**
可以使用` Luke `来核实数字是否由分析器保留下来并写入索引。**Luke是一款用于检查Lucene索引细节的优秀工具。**
 
B. 某些域只包含数字，希望将它们作为**数字域**值来索引，并能在搜索和排序中对它们进行精确匹配。
从2.9版本开始，Lucene加入了对数字域的支持，即` NumericField `类。
```

doc.add(new NumericField("price").setDoubleValue(19.99));

```
**NumericField类也能处理日期和时间，方法是将它们转换成等效的int型或long型整数。**
 
7.2. 索引日期和时间
```

 doc.add(new NumericField("timestamp").setLongValue(new Date().getTime()));
 Calendar cal = Calendar.getInstance();
 cal.setTime(date);
 doc.add(new NumericField("timestamp").setIntValue(cal.get(Calendar.DAY_OF_MONTH)));

```
 
##  8. 域截取(Field truncation) 
一些应用程序需要对尺寸未知的文档进行索引，作为一个控制RAM和硬盘空间使用量的安全机制，我们需要对每个域进行索引时对输入的文档尺寸进行限制。如：对每个文档的前面200个单词进行索引。
 **IndexWriter允许你对域进行截取后再索引它们。**
 在实例会IndexWriter后，必须向其传入MaxFieldLength对象，以便传递具体的截取数量。
  * MaxFieldLength.UNLIMITED 不截取
  * MaxFieldLength.LIMITED  只截取域中前1000个项
 当然，在实例化MaxFieldLength对象时，还可以设置自己所需要的截取数。建立IndexWriter后，可以在任意时间调整截取限制：
  * setMaxFieldLength()方法: 设置截取限制
  * getMaxFieldLength()方法: 获取截取限制
 使用MaxFieldLength时，一定要谨慎！因为域截取意味着程序会完全忽略一部分文档文本， 使得这些文本无法被搜索到，从而会让用户发现，有些文档找不到。
 【用户信任是保护业务的最重要条件】
 
##  9. 近实时搜索(Near-real-time Search) 
 Lucene2.9开始新增了一项被称为**近实时搜索**的重要功能，该功能解决了一个长期困扰搜索引擎的问题：文档的即时索引和即时搜索问题。
 实现方式：IndexReader ` IndexWriter.getReader() `方法
 该方法能【` 实时刷新缓冲区中新增或删除的文档 `】，然后创建新的包含这些文档的只读型IndexReader实例。
 请注意：**getReader()方法，会降低索引效率，因为这会使得IndexWriter马上刷新段内容，**而不是等到内存缓冲填满再刷新。
 
10. 优化索引
 优化索引就是将索引的多个段合并成一个或者少量段，同时，优化后的索引还可以在搜索期间少使用一些文件描述符。**优化索引只能提高搜索速度，而不是索引速度。**
 IndexWriter的四个优化方法：
optimize() 将索引压缩至一个段，操作完成再返回
optimize(int maxNumSegments)也称为部分优化，将索引压缩为最多maxNumSegments个段。因为将多个段合并为一个段的开销最大，建议优化至5个段，**它能比优化至一个段更快完成。**
optimize(boolean doWait)跟optimize()类似，doWait为false则不等待而立即执行，但合并工作是在后台完成的。doWait=false只适用于后台线程调用合并程序
optimize(int maxNumSegments, boolean doWait)也是部分优化，灰doWait=false则也在后台运行。
请记住，**索引优化会消耗大量的CPU和I/O资源**，使用时一定要明确这点。这是以一次性大量系统开销来换取更快的搜索速度。**搜索期间，还要注意一项重要开销就是磁盘临时使用空间。**
**合并期间，旧段不会被删除，磁盘临时空间会被用于保存新段对应的文件，这意味着，必须为程序预留大约3倍于优化用量的临时磁盘空间。**
 
11. 其它Directory子类
 Lucene的抽象类Directory主要是为我们提供一个简单的文件类存储API，它隐藏了实现存储的细节信息。当Lucene需要对索引中的文件进行读写操作时，它会调用Directory子类的对应方法来进行。
 五个核心子类如下：
 1) SimpleFSDirecotry
最简单的Directory子类，使用 java.io.* API将文件存入文件系统，不能很好地支持多线程操作。要支持多线程，必须在内部加入锁，而java.io.*并不支持按位置读取。
 
 2) NIOFSDirectory
使用  java.nio.* API将文件保存至文件系统。能很好地支持除Micfosoft Windows之外的多线程操作。它在Windows下的性能比较差，甚至可能比SimpleFSDirectory的性能还要差。
 
 3) MMapDirectory
使用内存映射I/O进行文件访问，对64位JRE来说是一个很好选择，对于32位JRE并且索引相对较小时也可以使用该类。建议最好使用64位的JRE。对于32位的JRE，程序可能由于内存碎片问题而遇到OutOfMemoryError异常MMapDirectory提供setMaxChunkSize()方法来处理该问题。
 
 4) FileSwitchDirectory
使用两个文件目录，根据文件扩展名在两个目录之间切换使用

 5) RAMDirectory
将所有文件都存入RAM

特别：所有的Directory子类在进行写操作时，都共享相同的代码，代码来自于SimpleFSDirectory，使用java.io.*
**那么，到底该采用哪个Directory子类呢？一个很好的方法就是使用静态的` FSDirectory.open() `方法**。该方法会根据当前的操作系统和平台来尝试选择最合适的默认FSDirectory子类，具体选择算法会随着Lucene版本的更新而改进。
 
##  12. 并发、线程安全及锁机制 
 索引文件的并发访问，IndexReader和IndexWriter的线程安全性，Lucene用于实现并发与线程安全的锁机制
12.1. 线程安全和多虚拟机安全
 1) 任意数量的只读属性的IndexReader类都可以同时打开一个索引。
不管这些Reader是否属于同一个JVM，是否属于同一台计算机。记住： **在单个JVM内，利用资源和发挥效率的最好办法是用多线程共享单个的` IndexReader `实例。**如： 多个线程或进程并行搜索同一个索引。

 2)** 对一个索引来说，一次只能打开一个Writer**
Lucene采用**文件锁**来提供保障。一旦建立起IndexWriter对象，系统即会分配一个锁给它。该锁只有当IndexWriter对象被关闭时才会释放。
如果使用IndexReader对象来改变索引的话(如修改norms或删除文档)，这时的IndexReader对象会作为Writer使用。它必须在修改上述内容之前成功地获取Writer锁，并在被关闭时释放该锁。

 3) IndexReader对象甚至可以在IndexWriter对象正在修改索引时打开。
每个IndexReader对象将向索引展示自己被打开的时间点。该对象只有在IndexReader对象提交修改或自己被重新打开后才能获知索引的修改情况。在已经有IndexReader对象被打开的情况下，打开新的IndexReader时采用采数create=true，这样，新的IndexReader会持续检查索引的情况。
 
 4) **任意多个线程都可以共享同一个` IndexReader类或IndexWriter `类。**
**这些类不仅是线程安全的，而且是线程友好的，**即是说他们能够很好地扩展到新增线程。
 
12.2. 通过远程文件系统访问索引
** 使用不同计算机上的不同虚拟机JVM来访问同一个索引，则得提供该索引的远程访问方式。但，最好的方式，是将索引复制到各台计算机自己的文件系统，再进行搜索。Solr支持这种复制策略。**
 四种远程文件系统：
 1) Samba/CIFS1.0 : Windows标准远程文件系统，能很好地共享Lucene索引
 2) Samba/CIFS2.0 : 新版本，由于不连贯的客户端缓存，Lucene不能很好地运行
 3) Networked File System(NFS) : 针对大多数UNIX操作系统的标准远程文件系统由于不连贯的客户端缓存，Lucene不能很好地运行
 4) Apple File Protocal(AFP) : Apple的标准远程文件协议。由于不连贯的客户端缓存，Lucene不能很好地运行
** 针对并发访问唯一的限制是不能同时打开多于一个writer**

12.3. 索引锁机制
 为实现单一的writer，即一个用于删除或修改norms的IndexWriter类或IndexReader类，Lucene采用了基于文件的锁。
 如果锁文件 writer.lock 存在于你的索引所在目录，writer会马上打开该索引。这时，若企图针对同一索引创建其它writer，将产生一个` LockObtainFailedException `异常。这是很重要的保护机制，因为若针对同一索引打开两个 writer 的话，会导致索引损坏。
 **索引允许你修改锁实现方法： 可以通过调用Directory.setLockFactory将任何LockFactory的子类设置为你自己的锁实现。**
 注意，在完成该操作之后，才能在Directory实例中打开IndexWriter类。
 正常情况下，不用担心程序正在使用哪个锁实现，**通常只有那些采用多台电脑或者多虚拟机的搜索程序，才可能需要自定义锁实现，以便能轮流进行索引操作。**
 Lucene提供的锁实现:
 1) NativeFSLockFactory：FSDirectory的默认锁，使用java.nio本地操作系统锁，在JVM还存在的情况下不会释放余下的被锁文件。该锁可能无法与一些共享文件系统很好地协同，特别是NFS文件系统。
 2) SimpleFSLockFactory：使用Java的File.createNewFile API，它比NativeFSLockFactory更易于在不同文件系统间移植。如果JVM崩溃或IndexWriter对象并未在JVM退出之前关尊重，则这会导致遗留一个write.lock文件，要手动删除。
 3) SingleInstanceLockFactory在内存中创建一个完全的锁，它是RAMDirectory默认的锁实现子类。在程序知道所有IndexWriter将在同一个JVM实例化时使用该类。
 4) NoLockFactory完全关闭锁机制，要小心使用。只有在程序确认不需要使用Lucene的锁保护机制时才能使用它。如： 使用带有单个IndexWriter实例的私有RAMDirectory.
** 注意： 这些锁在实现上都是不“公平”的**。如果该锁已被某writer持有，则后续的writer只是简单地重复申请获取该锁，默认情况是申请两次。当该锁被最初持有者释放时，系统并没有提供队列机制让后续writer持有该锁。**如果搜索程序需要采用队列机制的话，最好是自己实现它，同时，要确认该机制能够正确运行。**
` IndexWriter类的isLocked(dir) `方法，可以在创建一个新的writer对象前检查索引是否被锁住。
IndexWriter类的unLock(dir)方法，在任意刻对任意的Lucene索引进行解锁，贸然使用它很危险。在索引正被修改期间对它进行解锁的话，会立即导致该锁索引被毁坏并变得不可使用。最好不要直接操作锁文件，而应该通过Lucene提供的API来进行操作。
 
##  13. 调试索引 
如果需要对Lucene的写索引操作进行调试的话，可以通过调用 ` IndexWriter类的setInfoStream() `方法，通过打印诸如 System.out 等输出流(PrintStream)的方式获取Lucene操作索引的相关输出信息。
```

 IndexWriter writer = new IndexWriter(dir, nanlyzer, IndexWriter.MaxFieldLength.UNLIMITED);
 writer.setInfoStream(System.out);

```
该代码能够揭示有关段刷新和段合并的诊断信息，它可以帮助你调整相关的索引参数。如果在索引期间遇到问题，并估计是Lucene的bug导致，可以将该问题写入Apache的Lucene的用户清单中，然后，通过设置 infoStream 贴出相关系统信息。
 
##  14. 高级索引概念 
14.1. 用IndexReader删除文档
1) IndexReader能够根据文档号删除文档。IndexWriter则不能根据文档号删除文档
2) IndexReader可以通过Term对象删除文档。IndexReader通过Term对象删除文档与IndexWriter通过Term对象删除文档的区别：
**IndexReader会返回被删除的文档号，而IndexWriter则不能**。原因： IndexReader可以**立即决定**删除哪个文档，能对这些文档数量进行计算。IndexWriter则只是将被删除的Term进行缓存，后续再进行实际的删除操作
3) 如果程序使用相同的reader进行搜索，则IndexReader的删除操作会即时生效
而IndexWriter的删除操作必须等到程序打开一个新的Reader时才能被感知
4) IndexWriter可以通过Query对象执行删除操作，但IndexReader则不行
5) IndexReader提供了undeleteAll()方法，能反向操作索引中所有被挂起的删除
注意，只能对还未进行段合并后的文档进行反删除操作。
该方法之所以能实现反删除，是因为IndexWriter只是将被删除文档标记为删除状态，并未真正移除这些文档。最终的删除操作是在该文档对应的段进行合并时才执行。
**如果试图使用IndexReader删除文档，则Lucene只允许一个writer打开一次。但实施删除操作的IndexReader只能算作一作一个writer。这意昧着在使用reader进行删除操作之前，必须关闭已打开的IndexWriter，反之亦然。**
如果程序正在交叉进行文档的添加和删除操作，则会极大地降低索引吞吐量。更好的办法是，将添加和删除操作以批量的形式让IndexWriter完成，这样可以获得更好的性能。**一般而言，最好是只用IndexWriter完成所有删除操作。**
 
14.2. 回收被删除文档所使用过的磁盘空间
 Lucene使用一个简单办法来记录索引中被删除的文档：用bit数组的形式来标识它们，该操作速度很快，但对应的文档数据仍然会占用磁盘空间。该技术是必要的，因为对于一个倒排索引来说，给定的文档项是分散在各处的，
** 因此，在删除文档时试图回收它们占用的磁盘空间是不切实际的。**
1) 段合并操作时回收
** 只有在段合并操作时(正常的合并操作，或显示调用` optimize `方法)，这些磁盘空间才能被回收。**
2) 显式调用` expungeDeletes `方法回收
 还可以显式调用expungeDeletes方法，来回收被删除文档所占用的磁盘空间。该调用会对被挂起的删除操作相关的所有段进行合并。
 **这个操作的开销比优化小，但仍然会导致较大开销，一般只有在完成删除操作较长一段时间后才值得这样做。**
最坏的情况，如果删除操作分散在所有段中，则expungeDeletes所做的工作就与optimize方法一致：将所有段进行合并。
 
14.3. 缓冲和刷新
 当一个新文档被添加至Lucene索引时，或当挂起一个删除操作时，这些操作首先被缓存至内存，而不是立即在磁盘中进行。这种缓冲技术主要是出于降低磁盘I/O操作等性能原因而使用的，
 它们会以新段的形式周期性写入索引的Diretory目录。IndexWriter根据3个可能的标准来触发实际上的刷新，这三个标准是由程序控制的：
 1)  setRAMBufferSizeMB()
当缓存所占用的空间超过预设的RAM比例时进行实施刷新，预设方法为setRAMBufferSizeMB。RAM缓存大小不能被视为最大内存用量，因为还要考虑到影响测量JVM内存容量的其它因素。此外，IndexWriter并不占用所有的RAM使用空间，如段合并操作所占用的内存空间。
 2)  setMaxBufferedDocs()还可以在指定文档号所对应的文档被添加进索引之后通过调用 setMaxBufferedDocs 来完成刷新操作。
 3)  setMaxBufferedDeleteTerm()在删除项和查询语句等操作所占用的缓存总量超过预设值时，可以通过调用 setMaxBufferedDeleteTerm 方法来触发刷新操作。
 这几个触发器，只要其中之一被触发，都会启动刷新操作，与触发事件的顺序没有关系。常量 IndexWriter.DISABLE_AUTO_FLUSH 可以传递给以上任一方法，用以阻止发生刷新操作。
** 在默认情况下，IndexWriter只在RAM用量为16MB时启动刷新操作**。当发生刷新操作时，writer会在Directory目录创建新的段和被删除文件。
`  **但是，这些文件对于新打开的IndexReader来说既不可视也不可用，至到Writer向索引提交更改以及重新打开reader。** `
 **【刷新操作】是用来释放被缓存的更改；【提交操作】是用来让所有的更改在索引中` 保持可视 `。**
`  这意味着，IndexReader所看到的一直是索引的**起始状态**(IndexWriter打开时的索引状态)，直到writer提交更改为止。 `
 
14.4. 索引提交
索引提交的两个方法： ` commit()和close() `
注意：** 新打开或重启的IndexReader或IndexSearcher只能看到上次提交后的索引状态，**而IndexWriter在两次提交之间所完成的所有更改对于reader来说都是不可见的。
唯一的例外是近实时搜索功能，它可以在不用首次向磁盘提交更改的情况下，对IndexWriter所作的更改进行搜索。
 此外：  **提交操作的开销较大**，如果频繁进行该操作，会降低索引吞吐量。如果要取消更改，则在上一次项索引提交更改后调用` rollback() `方法，来删除当前writer上下文中包含的所有更改操作。
 
 1) IndexWriter 的提交步骤
(1) 刷新所有缓存的文档和文档删除操作;
2) 对所有新创建的文件进行同步，包括刷新新的文件，还包括上一次调用commit()方法或者从打开writer后已完成的段合并操作所生成的所有文件。 writer调用Directory.sync()来实现这一目标。该方法会在所有挂起的写操作都通过I/O系统写入稳定存储器之后才返回结果。
(3) 写入和同步下一个segments_N文件。一旦完成操作，reader会立即看到上一次提交后的所有变化。
(4) 调用 IndexDeletionPolicy 删除旧的提交，可以继承该类来实现自定义提交的内容和时间。上一次提交中包含的旧索引文件引用，只有在新的提交完成后才会被删除。

 2) 两阶段提交 two-phrase commit
` prepareCommit() `方法，会完成提交步骤1和步骤2，大多数还会完成步骤3，**但它不能使新的segments_N文件对Reader可视**。调用prepareCommit()方法后，只能调用` rollback() `方法终止提交，或commit()方法来完成提交。**如果已调用prepareCommit()方法，再调用commit()方法则会很快。**
 
 3) 索引删除策略
IndexDeletionPolicy类负责通知IndexWriter何时能够安全删除旧的提交。默认的策略是 KeepOnlyLastCommitDeletionPolicy，该策略会在每次创建完新的提交后删除先前的提交。
 
 4) 管理多个索引提交
通常情况下，Lucene索引只有一个当前提交，它是最近的提交。如实现自定义的提交策略，则可以很容易地在索引中聚集多个提交。IndexReader.listCommits()方法，可检索索引中当前所有的提交。
 
14.5. ACID事务和索引连续性
 Lucene实现了ACID事务模型，其限制是一次只能打开一个事务(writer)。
 (1) Actomic： 原子性
所有针对writer的变更，要么全部提交至索引，要么全不提提，没有中间状态。
 (2) Consistency: 一致性
索引必须是连续的。
 (3) Isolation: 隔离性
当使用writer进行索引变更时，只有进行后续提交时，新打开的reader才能看到上一次提交的索引变化。即使是在新打开writer时传入参数create=true也是如此。IndexReader只能看到上一次成功提交所带来的索引变化。

 (4) Durability: 持久性
如果程序遇到无法处理的异常，如JVM崩溃、操作系统崩溃、计算机掉电等，则索引会保持连续性，并会保留上一次成功提交的所有变更内容。
 
14.6. 合并段
 如果索引包含太多的段，writer会选择其中一些段，并将它们合并成一个单一的、更大的段。
 合并操作会带来两个重要的好处：一是会减少索引中的段数量，能加快搜索速度；二是会减小索引大小。
 1) 段合并策略
writer依赖于抽象基类 ` MergePolicy ` 的子类来决定何时进行段合并。
Lucene提供了两个核心的合并策略，都是 LogMergePolicy 的子类：
A. LogByteSizeMergePolicy该策略由writer使用，它会测量段大小，即：该段所包含的所有文件总字节数。
B. LogDocMergePolicy它完成与上一个子类相同的段合并策略，区别在于：它对段大小的测量是用段中文档数量来表示。
注意：这两个策略都不会执行真正的删除操作。
如果段中文档大小差别较大，**最好是使用 LogByteSizeMergePolicy子类。**
当然，也可以继承 MergePolicy 后，自定义相关策略。
 ***** 控制 LogByteSizeMergePolicy 合并策略的参数
2) ` MergeScheduler `
选取要被合并的段只是第一步。
第二步，是实施实际上的合并。
IndexWriter需要通过一个 MergeScheduler 子类来实成这个工作。
默认使用`  ConcurrentMergeScheduler ` 进行，该类利用后台线程完成段的合并。另外，SerialMergeScheduler 可以由调用它的线程来完成段合并，这意味着可以看到addDocument和deleteDocuments等方法正在进行的段合并操作。
此外，还可以继承MergeScheduler，来自定义自己的合并策略。如果出于某些原因，需要等待所有的段合并操作完成，再进行下一步操作，**则可以调用IndexWriter的waitForMerges方法。**
  
参考：http://blog.csdn.net/liuweitoo/article/details/8137404