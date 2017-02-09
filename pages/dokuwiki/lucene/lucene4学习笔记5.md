title: lucene4学习笔记5 

#  Lucene4.x正则分词器 
参考：http://my.oschina.net/MrMichael/blog/220781
一些特殊的分词需求，在此做个总结。本来的Lucene的内置的分词器，差不多可以完成我们的大部分分词工作了，如果是英文文章那么可以使用StandardAnalyzer标准分词器，WhitespaceAnalyzer空格分词器,
**对于中文我们则可以选择IK分词器，Messeg4j,庖丁等分词器。** 

我们先来看看下面的几个需求 
编号	需求分析
1	按单个字符进行分词无论是数字，字母还是特殊符号
2	按特定的字符进行分词，类似String中spilt()方法
3	按照某个字符或字符串进行分词

在特定的场合下确实是存在这样的需求的，看起来上面的需求很简单，,如果想要满足上面的需求，可能就需要我们自己定制自己的分词器了。 
先来看第一个需求，单个字符切分，这就要不管你是the一个单词还是一个电话号码还是一段话还是其他各种特殊符号都要保留下来，进行单字切分，这种特细粒度的分词，有两种需求情况，可能适应这两种场景 
(-)100%的实现数据库模糊匹配 
(=)对于某个电商网站笔记本的型号Y490,要求用户无论输入Y还是4,9,0都可以找到这款笔记本 

这种单字切分确实可以实现数据库的百分百模糊检索，但是同时也带来了一些问题，如果这个域中是存电话号码，或者身份证之类的与数字的相关的信息，**那么这种单字拆分分词法，会造成这个域的倒排链表非常之长，反映到搜索上，就会出现中文检索很快，而数字的检索确实非常之慢的问题。**原因是因为数字只有0-9个字符，而汉字则远远比这个数量要大的多，所以在选用这种分词时，还是要慎重的考虑下自己的业务场景到底适不适合这种分词，否则就会可能出一些问题。 

再来分析下2和3的需求，这种需求可能存在这么一种情况，就是某个字段里存的内容是按照逗号或者空格，#号，或者是自己定义的一个字符串进行分割存储的，而这种时候我们可能就会想到一些非常简单的做法，直接调用String类的spilt方法进行打散，确实，这种方式是可行的，但是lucene里面的结构某些情况下，就可能不适合用字符串拆分的方法，而是要求我们必须定义一个自己的分词器来完成这种功能，因为涉及到一些参数需要传一个分词器或者索引和检索时都要使用分词器来构造解析，所以有时候就必须得自己定义个专门处理这种情况的分词器了。 

**正则表达式在处理文本字符串上面有其独特的优势。下面我们要做的就是改写自己的正则解析器，代码非常精简，功能却是很强大的，上面的3个需求都可以解决，只需要传入不用的参数即可。** 
```

import java.io.Reader;
import java.util.regex.Pattern;

import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.pattern.PatternTokenizer;
 
/**
 * @author 三劫散仙
 * 自定义单个char字符分词器
 * **/
public class TestRegexpAnalyzer extends Analyzer{
    String regex;//使用的正则拆分式
    public TestRegexpAnalyzer(String regex) {
         this.regex=regex;
    }
 
    @Override
    protected TokenStreamComponents createComponents(String arg0, Reader arg1) {
        return new TokenStreamComponents(new PatternTokenizer(arg1, Pattern.compile(regex),-1));
    }
 
}

```
我们来看下运行效果: 
```

import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
 
/**
 * 测试的demo
 * 
 * **/
public class Test {
     
    public static void main(String[] args)throws Exception {
       //  SplitAnalyzer analyzer=new SplitAnalyzer('#');
         PatternAnalyzer analyzer=new PatternAnalyzer("");
         //空字符串代表单字切分  
        TokenStream ts=    analyzer.tokenStream("field", new StringReader("我#你#他"));
        CharTermAttribute term=ts.addAttribute(CharTermAttribute.class);
        ts.reset();
        while(ts.incrementToken()){
            System.out.println(term.toString());
        }
        ts.end();
        ts.close();
          
    }
 
}

```
输出效果:
我
#
你
#
他

传入#号参数 
PatternAnalyzer analyzer=new PatternAnalyzer("#");
输出效果: 
我
你
他

传入任意长度的字符串参数
 PatternAnalyzer analyzer=new PatternAnalyzer("分割");
TokenStream ts=    analyzer.tokenStream("field", new StringReader("我分割你分割他分割"));
输出效果:
我
你
他