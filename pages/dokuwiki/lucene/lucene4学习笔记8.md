title: lucene4学习笔记8 

#  Lucene4.x高亮显示 
在一个标准的搜索引擎中，高亮的返回命中结果，几乎是必不可少的一项需求，因为通过高亮，我们可以在我们的搜索界面上快速标记出用户的检索关键词，从而减少了用户自己寻找想要的结果，在一定程度上大大提高了用户的体验性和友好度。 
那么，今天就来看下我们在Lucene中，怎么实现高亮，以及高亮的几种实现方式。 
**要使用高亮，首先就得从索引时开始，因为需要高亮的字段，需要准确的获取位置信息，以及一些偏移量，如果信息不准确，那么可能在结果中，就会出现一些莫名其妙的错位**，反映到网页上就是标注了不该标注的字，没有标注该标的内容，所以这一点还是需要注意一下，**在索引的时候，我们需要使用` 项向量 `记录各个token的位置信息**，这很简单，代码如下: 
```

 FieldType type=new FieldType(TextField.TYPE_STORED); 
 type.setStoreTermVectorOffsets(true);//记录相对增量
 type.setStoreTermVectorPositions(true);//记录位置信息
 type.setStoreTermVectors(true);//存储向量信息
 type.freeze();//阻止改动信息
 Field field=new Field("字段名", "值", type);//示例

```
简单说下，TextField的2个枚举变量的意思 
变量名	释义
TYPE_NOT_STORED	索引，分词，不存储
TYPE_STORED	索引，分词，存储
由此看来，需要进行高亮的内容，是一定要存储的，可能有一些比较大的文本，会比较占索引空间，从而影响检索性能，当然我们也可以使用外部存储，关系型数据库，nosql什么的都可以，此时，高亮可能就需要做另一些处理了。 
下面我们来看下，**高亮的需要用到的一些基本的类** 
  * SimpleHTMLFormatter	常用的格式化Html标签器，提供一个构造函数传入高亮颜色标签，默认使用黑色
  * TokenSources	提供静态方法，支持从数据源中获取TokenStream，进行token处理
  * Highlighter	负责获取匹配上的高亮片段
  * QueryScorer	对命中结果进行评分操作
  * Fragmenter	将原始字符串拆分成独立的**片段**
  * NullFragmenter	对较短的域进行整体高亮
  * FastVectorHighlighter	基于快速高亮
  * Encoder	提供一些实现类，对html文本操作，如，去掉一些特殊匹配符号<,>  and so on,及一些其他的非ASCII特殊字符。

下面我们先来看下几条测试数据内容： 
id:1      name:  中国是一个伟大的国家,我们中国人都是好样的哈哈，中国永远是强大的   content:  你好人民
id:2      name:  我们有一个家它的名字是中国   content:  中国的大地，富饶
id:3      name:  我们的中国，我们的大地都是人民的希望的   content:  如果不在片段中生成一些字段的话
id:4      name:  2014年此时此刻你在做什么的啊   content:  哈哈锄禾日当午
id:5      name:  当你孤单时你会想起谁，你想不想找个人来陪   content:  我永远不孤单啊

##  普通高亮 
**1，测试普通高亮的核心代码：** 
```

  String filed="name";
        QueryParser query=new QueryParser(Version.LUCENE_44, filed, new IKAnalyzer(false));
      
        Query q=query.parse("伟大的中国");//测试字段
        TopDocs top=searcher.search(q, 100);
        QueryScorer score=new QueryScorer(q, filed);//传入评分
        SimpleHTMLFormatter fors=new SimpleHTMLFormatter("<span style=\"color:red;\">", "</span>");//定制高亮标签
         
        Highlighter  highlighter=new Highlighter(fors,score);//高亮分析器
        // highlighter.setMaxDocCharsToAnalyze(1);//设置高亮处理的字符个数
        for(ScoreDoc sd:top.scoreDocs){
            Document doc=searcher.doc(sd.doc);
            String name=doc.get(filed);
            TokenStream token=TokenSources.getAnyTokenStream(searcher.getIndexReader(), sd.doc, filed, new IKAnalyzer(true));//获取tokenstream
            Fragmenter  fragment=new SimpleSpanFragmenter(score);
            highlighter.setTextFragmenter(fragment);
            String str=highlighter.getBestFragment(token, name);//获取高亮的片段，可以对其数量进行限制
             
             System.out.println("高亮的片段 =====>"+str);
        }

```
输出结果如下
高亮的片段 =====>中国是一个<span style="color:red;">伟大</span><span style="color:red;">的</span>国家,我们中国人都是好样<span style="color:red;">的</span>哈哈，<span style="color:red;">中国</span>永远是强大<span style="color:red;">的</span>
高亮的片段 =====>我们<span style="color:red;">的</span><span style="color:red;">中国</span>，我们<span style="color:red;">的</span>大地都是人民<span style="color:red;">的</span>希望<span style="color:red;">的</span>
高亮的片段 =====>我们有一个家它<span style="color:red;">的</span>名字是<span style="color:red;">中国</span>
##  快速高亮 
2,快速高亮，**FastVectorHighlighter**，这个类可能会消耗更多的存储空间，来换取更好的性能，当然除了性能上提升外，它还有一个非常炫的功能，支持多种颜色标记，高亮关键字，除此之外还支持Ngram的域，以及智能合并相邻高亮短语. 
```

 Query q=query.parse("伟大的中华民族");
        TopDocs top=searcher.search(q, 100);
        //QueryScorer score=new QueryScorer(q, filed);
        //SimpleHTMLFormatter fors=new SimpleHTMLFormatter("<span style=\"color:red;\">", "</span>");//定制高亮标签
        //Highlighter  highlighter=new Highlighter(fors,score);//高亮分析器
        //FastVectorHighlighter fastHighlighter=new FastVectorHighlighter();
        FragListBuilder fragListBuilder=new SimpleFragListBuilder();
        //注意下面的构造函数里，使用的是颜色数组，用来支持多种颜色高亮
        FragmentsBuilder fragmentsBuilder= new ScoreOrderFragmentsBuilder(BaseFragmentsBuilder.COLORED_PRE_TAGS,BaseFragmentsBuilder.COLORED_POST_TAGS);
         
       
        FastVectorHighlighter fastHighlighter2=new FastVectorHighlighter(true, true, fragListBuilder, fragmentsBuilder);
        FieldQuery querys=fastHighlighter2.getFieldQuery(q);//reader是传入的流
         
        // highlighter.setMaxDocCharsToAnalyze(1);//设置高亮处理的字符个数
        for(ScoreDoc sd:top.scoreDocs){
          
            String snippt=fastHighlighter2.getBestFragment(querys, reader, sd.doc,filed,300);
          
             if(snippt!=null){
                 System.out.println("高亮的片段是:"+snippt);
             }    
        }

```
##  前台JS控制高亮方式 
3.下面来着重说一下，高亮的第三种方式**，前台高亮**。**对于后台高亮，基于高亮的字段，必须的存储，否则无法实现高亮标注，**那么对于大文本情况下，存储到索引里是非常浪费空间的，而且还有可能会影响到检索速度，所以就提出了，第三种方式。 
**在前台进行高亮，然后大文本字段，可以存储在外部其他的数据源里面，需要标记时，可以直接根据ID，或者某个字段，读取数据然后通过JS正则在前端替换检索的关键词即可，在这之前需要做的一步就是，使用ajax把检索的关键词，传入后台进行分词，然后将结果返回前台，进行对分词后的数据，进行匹配替换，再加上颜色标记，就可以在前台实现高亮了，这也是前台高亮的实现原理**，这种做法，在某些业务场景下，可以大大减少服务器压力，通过客户端减压，以及不用再存储一些向量信息，从而对系统的性能的提高，也是有很大帮助的。 
```

$.ajax({
                type :"post",
                url: "getContent",
                data:"str="+str,
                dataType:"json",
                async:false,
                success:function(msg){
                // alert(msg);
                 $("#div").empty();
                     $.each(msg, function(i, n) {
                         var temp=""; 
                  for(var i=0;i<shu.length;i++){
                   if(shu[i]!=""){
                 n.name=n.name.replace(new RegExp(shu[i],'g'), "<span style=\"color:red;\">"+shu[i]+"</span>");
                
                         }
                        }
                          $("#div").append("[*]"+n.name+"");
                          $("#div").append("[*]# # #### =")
                     });
 
                }
            });

```
下面给出一个前台高亮的截图，注意用的是快速高亮的索引。 
参考：http://my.oschina.net/MrMichael/blog/220952