title: lucene4学习笔记11 

#  Lucene4.x锁机制 
**Lucene的索引体系是一个写独占，读共享的结构**，这意味着，我们在使用多线程进行添加索引时，性能并不会得到明显的提升，所以**任何时刻只能有一个线程对索引进行写入操作**，而保障这个操作的安全性则是来自于，Lucene独特的锁机制（写入操作进行时，我们可以在Lucene的索引根目录里看到一个命名为**write.lock的锁文件**），**如果同一时刻有多个不同IndexWriter对索引进行写入操作，那么将会引发锁重叠异常，所以Lucene的特殊的索引结构，决定了其只能使用一个IndexWriter对索引进行添加操作。** 

**即使是限定Lucen只能使用一个线程进行写入操作，Lucene的写入性能也是非常高效的，特别是在Lucene4.x之后，更是优异**，我们可以根据自己服务器的硬件环境，来调优一些参数，利用上批处理的特性，可以大大提升写入性能。 

前面说过，Lucene写入时只能用一个线程操作，那么假如我们想使用多线程写入来提速可以吗？ 
答案是肯定的，**虽然Lucene限定只能用一个线程写入，但是这个限制仅仅指的是对一个索引文件的限制，我们可以采取一种折中的方式，利用多个线程写入多个索引文件夹目录，最后在对这几个索引文件合并或者，由此来提升索引速度，Lucene的API也支持多个索引文件的合并，所以采用这种方式来建索引，也能够大大的提升索引性能，**这种方式尤其适用于对数据库的数据建索引，我们可以采用分页读的方式，由某个固定数目的线程来建索引。 

本篇就来介绍下，如何使用LuceneD的API 来对多个索引文件进行合并操作，合并操作大多数时候要求我们的数据结构是要一致的，当然Lucene是一种文档型的松散的存储结构，某个文档里也可以存储自己特有的字段，而其他的文档里，则没有，**不过既然是我们需要合并，那么就要求大多数的结构是要一致的，否则两个完全不同类型的索引，合并到一起也是不符合逻辑的。** 
合并的核心代码如下： 
```

/***
     * 测试多个索引之间
     * 进行合并的方法
     * **/
      public static void combineMoreIndex(){
           
          try{
          Directory d1=FSDirectory.open(new File("E:\\1\\a"));//打开存放索引1的路径
          Directory d2=FSDirectory.open(new File("E:\\2\\a"));//打开存放索引2的路径
           
          Directory d3=FSDirectory.open(new File("E:\\3\\ab"));//合并到索引3里面
           
           IndexWriter writer=new IndexWriter(d3, new IndexWriterConfig(Version.LUCENE_44, new IKAnalyzer()));
           
           writer.addIndexes(d1,d2);//传入各自的Diretory或者IndexReader进行合并
           writer.commit();//提交索引
           writer.close();
           System.out.println("合并索引完毕.........");
           
           
          }catch(Exception e){
              e.printStackTrace();
          }
      }

```

为了演示合并，就建立了2份索引，然后对这两份索引进行合并。
参考：http://my.oschina.net/MrMichael/blog/220966