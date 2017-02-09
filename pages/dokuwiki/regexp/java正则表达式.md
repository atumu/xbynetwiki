title: java正则表达式 

#  Java正则表达式详解 
Java 提供了功能强大的正则表达式API，在` java.util.regex ` 包下。本教程介绍如何使用正则表达式API。
##  java.util.regex.Pattern 
###  Pattern.matches() 
下面是一个简单的Java正则表达式的例子，用于在文本中搜索 http:/ /
```

String text    =
        "This is the text to be searched " +
        "for occurrences of the http:// pattern.";
String pattern = ".*http://.*";
boolean matches = Pattern.matches(pattern, text);Pattern.matches()
System.out.println("matches = " + matches);

```
示例代码实际上没有检测找到的 http:/ / 是否是一个合法超链接的一部分,如包含域名和后缀(.com,.net 等等)。代码只是简单的查找字符串 http:/ / 是否出现。
Java6 中关于正则表达式的API
类 ` java.util.regex.Pattern ` 简称 Pattern, **是Java正则表达式API中的主要入口**，无论何时,需要使用正则表达式,从Pattern 类开始
**Pattern.matches()**检查一个正则表达式的模式是否匹配一段文本的最直接方法是调用静态方法Pattern.matches()
**Pattern.matches() 方法适用于检查 一个模式在一个文本中出现一次的情况**，或适用于Pattern类的默认设置。
###  Pattern.compile() 
如果需要匹配多次出现,甚至输出不同的匹配文本，或者只是需要非默认设置。
需要通过` Pattern.compile() ` 方法得到一个**Pattern 实例**。
```

String text    =
        "This is the text to be searched " +
        "for occurrences of the http:// pattern.";
String patternString = ".*http://.*";
Pattern pattern = Pattern.compile(patternString);

```
可以在Compile 方法中，指定一个**特殊标志**：
```

Pattern pattern = Pattern.compile(patternString, Pattern.CASE_INSENSITIVE);

```
Pattern 类包含多个标志(int 类型),这些标志可以控制Pattern 匹配模式的方式。上面代码中的标志使模式匹配是忽略大小写
###  Pattern.matcher() 
**一旦获得了Pattern对象，接着可以获得Matcher对象。**Matcher 示例用于匹配文本中的模式.示例如下
```

Matcher matcher = pattern.matcher(text);

```
Matcher类有一个matches()方法，可以检查文本是否匹配模式。以下是关于Matcher的一个完整例子
```

String text    =
        "This is the text to be searched " +
        "for occurrences of the http:// pattern.";
String patternString = ".*http://.*";
Pattern pattern = Pattern.compile(patternString, Pattern.CASE_INSENSITIVE);
Matcher matcher = pattern.matcher(text);
boolean matches = matcher.matches();
System.out.println("matches = " + matches);
Pattern.split()

```
**Pattern 类的 split()方法，可以用正则表达式作为分隔符，把文本分割为String类型的数组**。示例：
```

String text = "A sep Text sep With sep Many sep Separators";
String patternString = "sep";
Pattern pattern = Pattern.compile(patternString);
String[] split = pattern.split(text);
System.out.println("split.length = " + split.length);
for(String element : split){
    System.out.println("element = " + element);
}

```
上例中把text 文本分割为一个包含5个字符串的数组。

**Pattern.pattern()**
Pattern 类的 pattern 返回用于创建Pattern 对象的正则表达式,示例：
```

String patternString = "sep";
Pattern pattern = Pattern.compile(patternString);
String pattern2 = pattern.pattern();

```
上面代码中 pattern2 值为sep ，与patternString 变量相同。

##  Matcher (java.util.regex.Matcher) 
java.util.regex.Matcher 类用于匹配一段文本中多次出现一个正则表达式，Matcher 也适用于多文本中匹配同一个正则表达式。
Matcher 有很多有用的方法，详细请参考官方JavaDoc。这里只介绍核心方法。
###  matches() 
以下代码演示如何使用Matcher
```

String text    =
        "This is the text to be searched " +
        "for occurrences of the http:// pattern.";
String patternString = ".*http://.*";
Pattern pattern = Pattern.compile(patternString);
Matcher matcher = pattern.matcher(text);
boolean matches = matcher.matches();

```
首先创建一个Pattern，然后得到Matcher ，调用matches() 方法，返回true 表示模式匹配，返回false表示不匹配。
可以用Matcher 做更多的事。
matches() 方法不能用于查找正则表达式多次出现。如果需要，请使用find(), start() 和 end() 方法。
###  lookingAt() 
**lookingAt() 与matches() 方法类似，最大的不同是，lookingAt()方法对文本的开头匹配正则表达式；而matches() 对整个文本匹配正则表达式**。换句话说，如果正则表达式匹配文本开头而不匹配整个文本,lookingAt() 返回true,而matches() 返回false。 示例：
```

String text    =
        "This is the text to be searched " +
        "for occurrences of the http:// pattern.";
String patternString = "This is the";
Pattern pattern = Pattern.compile(patternString, Pattern.CASE_INSENSITIVE);
Matcher matcher = pattern.matcher(text);
System.out.println("lookingAt = " + matcher.lookingAt());
System.out.println("matches   = " + matcher.matches());

```
上面的例子分别对文本开头和整个文本匹配正则表达式 “this is the”. 匹配文本开头的方法(lookingAt()) 返回true。

###  find() + start() + end() 
find() 方法用于在文本中查找出现的正则表达式，文本是创建Matcher时，通过 Pattern.matcher(text) 方法传入的。**如果在文本中多次匹配，find() 方法返回第一个，之后每次调用 find() 都会返回下一个。**
**start() 和 end() 返回每次匹配的字串在整个文本中的开始和结束位置。**实际上, end() 返回的是字符串末尾的后一位，这样，可以在把 start() 和 end() 的返回值直接用在String.substring() 里。

```

String text    =
        "This is the text which is to be searched " +
        "for occurrences of the word 'is'.";
String patternString = "is";
Pattern pattern = Pattern.compile(patternString);
Matcher matcher = pattern.matcher(text);
int count = 0;
while(matcher.find()) {
    count++;
    System.out.println("found: " + count + " : "  + matcher.start() + " - " + matcher.end());
}

```
这个例子在文本中找到模式 “is” 4次，输出如下:
found: 1 : 2 - 4
 found: 2 : 5 - 7
 found: 3 : 23 - 25
 found: 4 : 70 - 72

###  reset() 
**reset() 方法会重置Matcher 内部的 匹配状态。**当find() 方法开始匹配时,Matcher 内部会记录截至当前查找的距离。调用 reset() 会重新从文本开头查找。
也可以调用 reset(CharSequence) 方法. 这个方法重置Matcher,同时把一个新的字符串作为参数传入，用于代替创建 Matcher 的原始字符串。
###  group() 
假设想在一个文本中查找URL链接，并且想把找到的链接提取出来。当然可以通过 start()和 end()方法完成。但是用group()方法更容易些。
**分组**在正则表达式中用括号表示，例如:(John)
此正则表达式匹配John, 括号不属于要匹配的文本。**括号定义了一个分组。当正则表达式匹配到文本后，可以访问分组内的部分。**
**使用group(int groupNo) 方法访问一个分组。**一个正则表达式可以有多个分组。每个分组由一对括号标记。想要访问正则表达式中某分组匹配的文本，可以把分组编号传入 group(int groupNo)方法。
` group(0) 表示整个正则表达式，要获得一个有括号标记的分组，分组编号应该从1开始计算。 `
```

String text    =  "John writes about this, and John writes about that," +
                        " and John writes about everything. "  ;
String patternString1 = "(John)";
Pattern pattern = Pattern.compile(patternString1);
Matcher matcher = pattern.matcher(text);
while(matcher.find()) {
    System.out.println("found: " + matcher.group(1));
}

```
以上代码在文本中搜索单词John.从每个匹配文本中，提取分组1，就是由括号标记的部分。输出如下
found: John
 found: John 
found: John
###  多分组 
上面提到，一个正则表达式可以有多个分组，例如：(John) (.+?)
这个表达式匹配文本”John” 后跟一个空格,然后跟1个或多个字符，最后跟一个空格。你可能看不到最后的空格。
这个表达式包括一些字符有特别意义。字符 点 . 表示任意字符。 字符 + 表示出现一个或多个，和. 在一起表示 任何字符,出现一次或多次。字符? 表示 匹配尽可能短的文本。
```

String text    =
          "John writes about this, and John Doe writes about that," +
                  " and John Wayne writes about everything."
        ;
String patternString1 = "(John) (.+?) ";
Pattern pattern = Pattern.compile(patternString1);
Matcher matcher = pattern.matcher(text);
while(matcher.find()) {
    System.out.println("found: " + matcher.group(1) +
                       " "       + matcher.group(2));
}

```
注意代码中引用分组的方式。代码输出如下

found: John writes 
found: John Doe 
found: John Wayne
###  嵌套分组 
在正则表达式中分组可以嵌套分组，例如```

((John) (.+?))

```
这是之前的例子，现在放在一个大分组里.(表达式末尾有一个空格)。
` 当遇到嵌套分组时, 分组编号是由左括号的顺序确定的。 `上例中，**分组1 是那个大分组**。分组2 是包括John的分组，分组3 是包括 .+? 的分组。当需要通过groups(int groupNo) 引用分组时，了解这些非常重要。
以下代码演示如何使用嵌套分组
```

String text    =
          "John writes about this, and John Doe writes about that," +
                  " and John Wayne writes about everything."
        ;
String patternString1 = "((John) (.+?)) ";
Pattern pattern = Pattern.compile(patternString1);
Matcher matcher = pattern.matcher(text);
while(matcher.find()) {
    System.out.println("found:   ");
}

```
输出如下
found: 
 found: 
 found:
###  replaceAll() + replaceFirst() 
` replaceAll() 和 replaceFirst() ` 方法可以用于替换Matcher搜索字符串中的一部分。replaceAll() 方法替换全部匹配的正则表达式，replaceFirst() 只替换第一个匹配的。
在处理之前，Matcher 会先重置。所以这里的匹配表达式从文本开头开始计算。
```

String text    =
          "John writes about this, and John Doe writes about that," +
                  " and John Wayne writes about everything."
        ;
String patternString1 = "((John) (.+?)) ";
Pattern pattern = Pattern.compile(patternString1);
Matcher matcher = pattern.matcher(text);

String replaceAll = matcher.replaceAll("Joe Blocks ");
System.out.println("replaceAll   = " + replaceAll);

String replaceFirst = matcher.replaceFirst("Joe Blocks ");
System.out.println("replaceFirst = " + replaceFirst);

```
输出如下
replaceAll = Joe Blocks about this, and Joe Blocks writes about that,
and Joe Blocks writes about everything.

replaceFirst = Joe Blocks about this, and John Doe writes about that,
and John Wayne writes about everything.
注意第1个字符串中所有出现 John 后跟一个单词 的地方，都被替换为 Joe Blocks 。第2个字符串中，只有第一个出现的被替换。
###  appendReplacement() + appendTail() 
appendReplacement() 和 appendTail() 方法用于替换输入文本中的字符串短语，同时把替换后的字符串附加到一个 StringBuffer 中。
当find() 方法找到一个匹配项时，可以调用 appendReplacement() 方法，这会导致输入字符串被增加到StringBuffer 中，而且匹配文本被替换。 从上一个匹配文本结尾处开始，直到本次匹配文本会被拷贝。
appendReplacement() 会记录拷贝StringBuffer 中的内容，可以持续调用find(),直到没有匹配项。
直到最后一个匹配项目，输入文本中剩余一部分没有拷贝到 StringBuffer. 这部分文本是从最后一个匹配项结尾，到文本末尾部分。通过调用 appendTail() 方法，可以把这部分内容拷贝到 StringBuffer 中.
```

String text    =
          "John writes about this, and John Doe writes about that," +
                  " and John Wayne writes about everything.";
String patternString1 = "((John) (.+?)) ";
Pattern      pattern      = Pattern.compile(patternString1);
Matcher      matcher      = pattern.matcher(text);
StringBuffer stringBuffer = new StringBuffer();

while(matcher.find()){
    matcher.appendReplacement(stringBuffer, "Joe Blocks ");
    System.out.println(stringBuffer.toString());
}
matcher.appendTail(stringBuffer);
System.out.println(stringBuffer.toString());

```
**注意我们在while循环中调用appendReplacement() 方法。在循环完毕后调用appendTail()。** 代码输出如下:
Joe Blocks
 Joe Blocks about this, and Joe Blocks
 Joe Blocks about this, and Joe Blocks writes about that, and Joe Blocks
 Joe Blocks about this, and Joe Blocks writes about that, and Joe Blocks
 writes about everything.

参考:http://www.tuicool.com/articles/rQ3mQfJ