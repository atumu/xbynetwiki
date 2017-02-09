title: guava学习之i_o 

#  Guava学习之I/O 
大多数Guava流工具一次处理一个完整的流，并且/或者为了效率自己处理缓冲。还要注意到，接受流为参数的Guava方法不会关闭这个流：**关闭流的职责通常属于打开流的代码块。**
术语”字节流”指的是InputStream或OutputStream，”字符流”指的是Reader 或Writer（虽然他们的接口Readable 和Appendable被更多地用于方法参数）。相应的工具方法分别在` ByteStreams 和CharStreams中。 `
##  ByteStreams 和CharStreams 
(作为了解，不推荐使用)
其中的一些工具方法列举如下：
![](/data/dokuwiki/guava/pasted/20150818-052838.png)
**BtyeStreams和CharStreams中很多方法都过时或者被替代了，这两个类在未来的版本会被移除 不推荐使用**
推荐使用CharSource CharSink ByteSource ByteSink代替。

##  源与汇(source and sink) 
源是可读的，汇是可写的。此外，源与汇按照字节和字符划分类型。
 	字节	字符
读	ByteSource	CharSource
写	ByteSink	CharSink
**源与汇API的好处是它们提供了通用的一组操作**。比如，一旦你把数据源包装成了ByteSource，无论它原先的类型是什么，你都得到了一组按字节操作的方法。

**创建源与汇**
Guava提供了若干源与汇的实现
![](/data/dokuwiki/guava/pasted/20150818-053314.png)

注：把已经打开的流（比如InputStream）包装为源或汇听起来是很有诱惑力的，但是应该避免这样做。源与汇的实现应该在每次openStream()方法被调用时都创建一个新的流。始终创建新的流可以让源或汇管理流的整个生命周期，并且让多次调用openStream()返回的流都是可用的。此外，如果你在创建源或汇之前创建了流，你不得不在异常的时候自己保证关闭流，这压根就违背了发挥源与汇API优点的初衷。

**使用源与汇**
一旦有了源与汇的实例，就可以进行若干读写操作。
` ByteSource `有一个方法asCharSource(Charset charset)可以**将ByteSource转为CharSource**，类似与InputStreamReader将InputSream转为Reader
` CharSource	asCharSource(Charset charset) `

**通用操作**

所有源与汇都有一些方法用于打开新的流用于读或写。默认情况下，其他源与汇操作都是先用这些方法打开流，然后做一些读或写，最后保证流被正确地关闭了。这些方法列举如下：
` openStream() `：根据源与汇的类型，返回InputStream、OutputStream、Reader或者Writer。
` openBufferedStream() `：根据源与汇的类型，返回InputStream、OutputStream、BufferedReader或者BufferedWriter。返回的流保证在必要情况下做了缓冲。例如，从字节数组读数据的源就没有必要再在内存中作缓冲，这就是为什么该方法针对字节源不返回BufferedInputStream。字符源属于例外情况，它一定返回BufferedReader，因为BufferedReader中才有readLine()方法。

**源操作**
![](/data/dokuwiki/guava/pasted/20150818-074147.png)
**汇操作**
![](/data/dokuwiki/guava/pasted/20150818-074230.png)

##  文件操作Files 
` Files `类用于对本地文件进行一般的操作，**Resources则可以用于网络资源的操作**
` FileWriteMod ` 这是一个枚举， 用户可以选择写文件时是覆盖写还是将内容追加到文件末尾
` LineProcessor `行处理器，这是一个接口。主要作为` Files.read(File file, Charset charset, LineProcessor<T> callback) `最后一个参数，当读到指定行时返回
` Closer `用于简化关闭各种流，在jdk7.0中可以使用try-with-resources代替。对Closer使用嵌套的try块则可以达到jdk7.0同样的效果。
除了` 创建文件源 `和文件的方法，Files类还包含了若干你可能感兴趣的便利方法。
Guava主要对Reader,Writer,InputStream,OurputStream进行封装，尽量避免用户直接接触这些类

![](/data/dokuwiki/guava/pasted/20150818-074633.png)

示例：
```

//Read the lines of a UTF-8 text file
ImmutableList<String> lines = Files.asCharSource(file, Charsets.UTF_8).readLines();
//Count distinct word occurrences in a file
Multiset<String> wordOccurrences = HashMultiset.create(
        Splitter.on(CharMatcher.WHITESPACE)
            .trimResults()
            .omitEmptyStrings()
            .split(Files.asCharSource(file, Charsets.UTF_8).read()));

//SHA-1 a file
HashCode hash = Files.asByteSource(file).hash(Hashing.sha1());

//Copy the data from a URL to a file
Resources.asByteSource(url).copyTo(Files.asByteSink(file));

//以追加的方式将内容写人文件
CharSink charSink = Files.asCharSink(new File("f:"+File.separator+"a.txt"), Charsets.UTF_8,FileWriteMode.APPEND);
List<String> toWrite = Lists.newArrayList("abc","ddd");
charSink.writeLines(toWrite);


```

参考：
http://daizhenghua.good.blog.163.com/blog/static/10528772620143151555810/
http://ifeve.com/google-guava-io/