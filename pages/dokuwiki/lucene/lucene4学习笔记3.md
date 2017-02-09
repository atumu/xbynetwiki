title: lucene4学习笔记3 

#  Lucene4.x分词器测试 
**比较推荐的中文分词器是IK分词器**，在进入正式的讲解之前，我们首先对Lucene里面内置的几个分析器做个了解. 
分析器类型	基本介绍
WhitespaceAnalyzer	以空格作为切词标准，不对语汇单元进行其他规范化处理
SimpleAnalyzer	以非字母符来分割文本信息，并将语汇单元统一为小写形式，并去掉数字类型的字符
StopAnalyzer	该分析器会去除一些常有a,the,an等等，也可以自定义禁用词
StandardAnalyzer	Lucene内置的标准分析器,会将语汇单元转成小写形式，并去除停用词及标点符号
CJKAnalyzer	能对中，日，韩语言进行分析的分词器，对中文支持效果一般。
SmartChineseAnalyzer	对中文支持稍好，但扩展性差

下面先看第一个纯分词的测试
```

import java.io.StringReader;
 
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
 
 
public class Test {
     
      
    public static void main(String[] args)throws Exception {
                      //下面这个分词器，是经过修改支持同义词的分词器
          IKSynonymsAnalyzer analyzer=new IKSynonymsAnalyzer();
           String text="三劫散仙是一个菜鸟";
           TokenStream ts=analyzer.tokenStream("field", new StringReader(text));
            CharTermAttribute term=ts.addAttribute(CharTermAttribute.class);
            ts.reset();//重置做准备
            while(ts.incrementToken()){
                System.out.println(term.toString());
            }
            ts.end();//
            ts.close();//关闭流
         
          
    }
 
}

```
运行结果:
三
劫
散
仙
是
一个
菜鸟

下面给出扩展同义词部分的源码
```

import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.synonym.SynonymFilterFactory;
import org.apache.solr.core.SolrResourceLoader;
import org.wltea.analyzer.lucene.IKTokenizer;
/**
 * 可以加载同义词库的Lucene
 * 专用IK分词器
 * 
 * 
 * */
public class IKSynonymsAnalyzer extends Analyzer {
 
      
    @Override
    protected TokenStreamComponents createComponents(String arg0, Reader arg1) {
         
        Tokenizer token=new IKTokenizer(arg1, true);//开启智能切词
         
        Map<String, String> paramsMap=new HashMap<String, String>();
        paramsMap.put("luceneMatchVersion", "LUCENE_43");
        paramsMap.put("synonyms", "E:\\同义词\\synonyms.txt");
        SynonymFilterFactory factory=new SynonymFilterFactory(paramsMap);
         SolrResourceLoader loader=    new SolrResourceLoader("");
        try {
            factory.inform(loader);
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
      
        return new TokenStreamComponents(token, factory.create(token));
    }
}

```
参考：http://my.oschina.net/MrMichael/blog/220771