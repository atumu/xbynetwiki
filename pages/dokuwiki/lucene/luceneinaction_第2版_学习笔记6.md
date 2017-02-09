title: luceneinaction_第2版_学习笔记6 

#  Lucene4.x相比3.x的变动 
在Lucene 3.x时代，《Lucene In Action》是一本相当不错的参考书，书中详细介绍了Lucene各种高级使用技术，对于开发者来说非常实用。但是近期Lucene升级到了4.x版本，在性能等各方面有了很大的提高，值得在新项目中使用。然而Lucene 4.x中的API相比3.x来说有了很大的改变，《Lucene In Action》中的很多内容都已经过时了
由于现在网络搜索都希望达到实时搜索的效果，用户上传文章后，希望立即在搜索结果中可见，这就要求我们必须使用Lucene的准实时搜索功能，使我们在不影响性能的情况下达到近实时搜索的效果。
**然而准实时搜索API在4.x版本中已经与3.x版本完全不同了。**
首先来看怎样获取准实时搜索的Reader实例，大家都知道，由于性能等方面原因，基于Lucene的应用一般都采用共享Lucene的Writer和Reader及Searcher的方案，我们这里也不例外：
```

	indexPathname = "D:/aproject/xincaigu/work/index";  
        analyzer = new MMSegAnalyzer();  
        IndexWriterConfig iwc = new IndexWriterConfig(Version.LUCENE_41, analyzer);  
        iwc.setOpenMode(OpenMode.CREATE_OR_APPEND);  
        try {  
            indexDir = FSDirectory.open(new File(indexPathname));   
            writer = new IndexWriter(indexDir, iwc);  // writer和reader整个程序共用  
            reader = DirectoryReader.open(writer, true);  
            //reader = writer.getReader();  
        } catch (CorruptIndexException e) {  
        } catch (LockObtainFailedException e) {  
        } catch (IOException e) {  
        }  

```
熟悉Lucene 3.x的朋友一定注意到了，**获取准实时搜索所用的Reader已经改用DirectoryReader.open方法，而不是3.x当中的writer.getReader()方法了。**

同样，在3.x中，为了可以看到刚刚添加的新文章，Reader需要进行reopen操作，这是一种节省资源的方式，可以获取新加入索引的文章，而不需要将改动保存到磁盘上，然后重新打开索引的方式来进行了。**但是reopne在4.x也被新API所取代，具体的用法如下所示：**
```

try {  
            IndexReader newReader = DirectoryReader.openIfChanged((DirectoryReader)reader, writer, false);//reader.reopen();      // 读入新增加的增量索引内容，满足实时索引需求  
            if (newReader != null) {  
                reader.close();  
                reader = newReader;  
            }  
            searcher = new IndexSearcher(reader);  
        } catch (CorruptIndexException e) {  
        } catch (IOException e) {  
        }  

```
**这里首先利用新APIDirctoryReader.openIfChanged来获取Reader，如果有新内容，则返回新的Reader，这时我们需要关闭老的Reader。**
**通过以上代码，我们就可以利用Lucene 4.x的准实时搜索功能了。**
但是Lucene 4.x中API的变动远不止这些，**在进行索引时，原来定义Field的方式已经过时，取而代之的是更加灵活的FieldType机制**，下篇文章中我们将详细探讨如何在文本索引中使用这一新的机制。
以前在增加索引的时候给document增加字段都是 
Field FieldPath = new Field("path", textFiles[i].getPath(),Field.Store.YES, Field.Index.NO);
可以指定该字段是否存储，是否索引，**但4.0的版本里面Field.Index这个属性已经Deprecated。** 
而且构造方法Field(String name, String value, Field.Store store, Field.Index index)也显示Deprecated，**提示使用StringField和TextField来代替**。
这里StringField是默认不分词的，而TextField是默认分词的。所以上面的代码可以使用StringField来代替的。 
比较常用的有LongField、StoredField、StringField、TextField。具体有哪些构造方法建议大家自己查阅api文档。
每种Field都有具体的介绍，我这里重点介绍一下常用的几个：
  * LongField：索引但是不分词，适用于全部搜索，一般用于文件创建、修改时间的秒数等。
  * StoredField：只存不索引
  * **StringField：索引但是不分词，所以适用于全部搜索的内容**，比如国家等，要么不对，要么就是全对。另外提一句，**这个有个长度限制是32766**，大家使用的时候自己斟酌一下，一般不会超过。
  * **TextField：索引并分词，所以这个应该是我们做全文检索的时候应该最常用到的一个Field。**

