title: java正则exception1 

#  Java replaceAll()方法报错Illegal group reference 
replaceAll(regex, replacement)函数，由于第一个参数支持正则表达式，**replacement中出现“$”,会按照$1$2的分组模式进行匹配。
当编译器发现“$”后跟的不是整数的时候，就会抛出“非法的组引用”的异常。**
 
例如，如下代码会报错：
```

public class Test {  
    public static void main(String[] args) {  
        String str = "123ABC456";  
        String re = "#7T$/#";  
        System.out.println(str.replaceAll("ABC", re));  
    }  
}

```  

报错内容：
```

Exception in thread "main" java.lang.IllegalArgumentException: Illegal group reference  
    at java.util.regex.Matcher.appendReplacement(Unknown Source)  
    at java.util.regex.Matcher.replaceAll(Unknown Source)  
    at java.lang.String.replaceAll(Unknown Source)  
    at cn.com.vogue.Test.main(Test.java:6) 

``` 
 
**解决办法：**
方法一、一个是JDK提供的方法，对特殊字符进行处理：
对要替换的字符做处理代码如下：
```

re = java.util.regex.Matcher.quoteReplacement(re); 

``` 
 
方法二、把特殊字符转为特定字符，然后交给接收方处理：
```

public static void main(String[] args) {  
        String text = "123456";  
        String replacement = "two$two";  
        replacement = replacement.replaceAll("\\$", "RDS_CHAR_DOLLAR");// encode replacement;  
        String resultString = text.replaceAll("2", replacement);  
        resultString = resultString.replaceAll("RDS_CHAR_DOLLAR", "\\$");// decode replacement;  
        System.out.println(resultString);  
    }

``` 

参考：http://cuisuqiang.iteye.com/blog/2062173
http://stackoverflow.com/questions/11913709/why-does-replaceall-fail-with-illegal-group-reference
http://stackoverflow.com/questions/14152811/java-string-replaceall-and-replacefirst-fails-at-symbol-at-replacement-text