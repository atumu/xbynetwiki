title: guava学习之基本工具_basic_utilities 

#  Guava学习之基本工具 [Basic utilities] 
1.1 使用和避免null：null是模棱两可的，会引起令人困惑的错误，有些时候它让人很不舒服。很多Guava工具类用快速失败拒绝null值，而不是盲目地接受
1.2 前置条件: 让方法中的条件检查更简单
1.3 常见Object方法: 简化Object方法实现，如hashCode()和toString()
1.4 排序: Guava强大的”流畅风格比较器”
1.5 Throwables：简化了异常和错误的传播与检查

#  使用和避免null 
我们认为， 相比默默地接受null，使用快速失败操作拒绝null值对开发者更有帮助。
大多数情况下，开发人员使用null表明的是某种缺失情形：可能是已经有一个默认值，或没有值，或找不到值。例如，Map.get返回null就表示找不到给定键对应的值。
Guava用` Optional<T> `表示可能为null的T类型引用。一个Optional实例可能包含非null的引用（我们称之为**引用存在**），也可能什么也不包括（称之为**引用缺失**）。它从不说包含的是null值，而是用存在或缺失来表示。但Optional从不会包含null值引用。
```

Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5

```
创建Optional实例（以下都是静态方法）：
Optional.of(T)	创建指定引用的Optional实例，若引用为null则快速失败
Optional.absent()	创建引用缺失的Optional实例
Optional.fromNullable(T)	创建指定引用的Optional实例，若引用为null则表示缺失
用Optional实例查询引用（以下都是非静态方法）：
boolean isPresent()	如果Optional包含非null的引用（引用存在），返回true
T get()	返回Optional所包含的引用，若引用缺失，则抛出java.lang.IllegalStateException
T or(T)	返回Optional所包含的引用，若引用缺失，返回指定的值
T orNull()	返回Optional所包含的引用，若引用缺失，返回null
Set<T> asSet()	返回Optional所包含引用的单例不可变集，如果引用存在，返回一个只有单一元素的集合，如果引用缺失，返回一个空集合。
#  前置条件 
前置条件：**让方法调用的前置条件判断更简单。**

Guava在` Preconditions `类中提供了若干**前置条件判断**的实用方法，我们强烈建议在Eclipse中静态导入这些方法。每个方法都有三个变种：比如
checkArgument(boolean expression)
checkArgument(boolean expression, Object errorMessage)
checkArgument(boolean expression, String errorMessageTemplate, Object... errorMessageArgs)
```

checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
checkArgument(i < j, "Expected i < j, but %s > %s", i, j);

```
方法声明（不包括额外参数）	描述	检查失败时抛出的异常
checkArgument(boolean)	检查boolean是否为true，用来检查传递给方法的参数。	IllegalArgumentException
checkNotNull(T)	检查value是否为null，该方法直接返回value，因此可以内嵌使用checkNotNull。	NullPointerException
checkState(boolean)	用来检查对象的某些状态。	IllegalStateException
checkElementIndex(int index, int size)	检查index作为索引值对某个列表、字符串或数组是否有效。index>=0 && index<size *	IndexOutOfBoundsException
checkPositionIndex(int index, int size)	检查index作为位置值对某个列表、字符串或数组是否有效。index>=0 && index<=size *	IndexOutOfBoundsException
checkPositionIndexes(int start, int end, int size)	检查[start, end]表示的位置范围对某个列表、字符串或数组是否有效*	IndexOutOfBoundsException

**在编码时，如果某个值有多重的前置条件，我们建议你把它们放到不同的行**，这样有助于在调试时定位。此外，把每个前置条件放到不同的行，也可以帮助你编写清晰和有用的错误消息。

#  常见Object方法 
**equals**
当一个对象中的字段可以为null时，实现` Object.equals `方法会很痛苦，因为不得不分别对它们进行null检查。使用Objects.equal帮助你执行null敏感的equals判断，从而避免抛出NullPointerException。例如:
```

Objects.equal("a", "a"); // returns true
Objects.equal(null, "a"); // returns false
Objects.equal("a", null); // returns false
Objects.equal(null, null); // returns true

```
注意：JDK7引入的Objects类提供了一样的方法Objects.equals。
**hashCode**
用对象的所有字段作散列[hash]运算应当更简单。Guava的` Objects.hashCode(Object...) `会对传入的字段序列计算出合理的、顺序敏感的散列值。你可以使用Objects.hashCode(field1, field2, …, fieldn)来代替手动计算散列值。
注意：JDK7引入的Objects类提供了一样的方法Objects.hash(Object...)
**toString**
好的toString方法在调试时是无价之宝，但是编写toString方法有时候却很痛苦。使用 Objects.toStringHelper可以轻松编写有用的toString方法。例如：
```

// Returns "ClassName{x=1}"
Objects.toStringHelper(this).add("x", 1).toString();
// Returns "MyObject{x=1}"
Objects.toStringHelper("MyObject").add("x", 1).toString();

```
**compare/compareTo**
实现一个比较器[Comparator]，或者直接实现Comparable接口有时也伤不起。考虑一下这种情况：
```

class Person implements Comparable<Person> {
  private String lastName;
  private String firstName;
  private int zipCode;

  public int compareTo(Person other) {
    int cmp = lastName.compareTo(other.lastName);
    if (cmp != 0) {
      return cmp;
    }
    cmp = firstName.compareTo(other.firstName);
    if (cmp != 0) {
      return cmp;
    }
    return Integer.compare(zipCode, other.zipCode);
  }
}

```
这部分代码太琐碎了，因此很容易搞乱，也很难调试。我们应该能把这种代码变得更优雅，为此，Guava提供了` ComparisonChain `
ComparisonChain执行一种懒比较：**它执行比较操作直至发现非零的结果**，在那之后的比较输入将被忽略。否则返回0
```

public int compareTo(Foo that) {
    return ComparisonChain.start()
            .compare(this.aString, that.aString)
            .compare(this.anInt, that.anInt)
            .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
            .result();
}

```
这种Fluent接口风格的可读性更高，发生错误编码的几率更小，并且能避免做不必要的工作。


##  字符串处理 
###  使用Joiner类字符串连接 
将任意字符串通过分隔符进行连接到一起是大多程序员经常做的事情。他们经常使用array，list，iterable并且循环变量将每一个临时变量添加到StringBuilder当中去，并且中间添加分隔符。这些笨重的处理方式如下:
```

public String buildString(List<String> stringList, String delimiter){
　　StringBuilder builder = new StringBuilder();
　　for (String s : stringList) {
　　　　if(s !=null){
　　　　　　builder.append(s).append(delimiter);
　　　　}
　　}
　　builder.setLength(builder.length() – delimiter.length());
　　return builder.toString();
}

```
注意要删除在最后面的分隔符。不是很难懂，但是使用Joiner类可以得到简单的代码模板。同样的例子使用Joiner类代码如下：
```

public String buildString(List<String> stringList, String delimiter){ 
       return  Joiner.on(delimiter).skipNulls().join(stringList);
}

```
这样更加简明并且不会出错。如果你想将null值替换掉，可以使用如下方法：
```

Joiner.on("|").useForNull("no value").join(stringList);

```
使用Joiner类有几点需要注意。Joiner类不仅仅可以处理字符串的array、list、iterable，他还可以处理任何对象的array、list、iterable。结果就是调用每一个元素的toString()方法。因此，如果没有使用skipNulls或者useForNull，就会抛出空指针异常。Joiner对象一旦被创建就是不可变的，所以他们是线程安全的，可以被当作常亮来看待。然后看看下面的代码片段：
```

Joiner stringJoiner = Joiner.on("|").skipNulls();
　　//使用useForNull方法将会返回一个新的Joiner实例
　　stringJoiner.useForNull("missing");
　　stringJoiner.join("foo","bar",null);

```
在上面的代码实例当中，useForNull方法并没有起作用，null值仍然被忽略了。
Joiner不仅仅能返回字符串，还可以与StringBuilder一起使用：
```

StringBuilder stringBuilder = new StringBuilder();
Joiner joiner = Joiner.on("|").skipNulls();

//返回的StringBuilder实例当中包含连接完成的字符串
joiner.appendTo(stringBuilder,"foo","bar","baz");

```
上面的例子，我们传入一个StringBuilder的参数并且返回一个StringBuilder实例。
Joiner类也可以使用实现了Appendable接口的类。
```

FileWriter fileWriter = new FileWriter(new File("path")):
List<Date> dateList = getDates();
Joiner joiner = Joiner.on("#").useForNulls(" ");
// 返回由字符串拼接后的FileWriter实例 
joiner.appendTo(fileWriter,dateList);

```
这是一个与上一个相似的例子。我们传入一个FileWriter实例和一组数据，就会将这组数据拼接后附加到FileWriter当中并且返回。
我们可以看到，Joiner使一些公共的操作变的非常简单。有一个特殊的方法会实现MapJoiner方法，MapJoiner方法像Joiner一样使用分割符将每组key与value分开，同时key与value之间也有个分隔符。MapJoiner方法的创建如下：
```

MapJoiner mapJoiner = Joiner.on("#").withKeyValueSeparator("=");

```
快速回顾一下上面内容：
Joiner.on("#")方法会创建一个Joiner的实例。
使用返回的Joiner实例调用withKeyValueSeparator方法将会返回MapJoiner对象。
下面是对MapJoiner方法的单元测试代码：
```

@Test
public void testMapJoiner() {
　　String expectedString = "Washington D.C=Redskins#New York City=Giants#Philadelphia=Eagles#Dallas=Cowboys"; 
　　Map<String,String> testMap = Maps.newLinkedHashMap();
　　testMap.put("Washington D.C","Redskins");
　　testMap.put("New York City","Giants");
　　testMap.put("Philadelphia","Eagles");
　　testMap.put("Dallas","Cowboys");
　　String returnedString = Joiner.on("#"). withKeyValueSeparator("=").join(testMap); 
　　assertThat(returnedString,is(expectedString));
}

```
 
###  使用Splitter类分割字符串 
程序员另一个经常处理的问题是对字符串以特定分隔符进行分割并且获取一个字符串数组。如果你需要读取文本文件，你总是会做这样的事情。但是
String.split方法不够完美，看下面的例子：
```

String testString = "Monday,Tuesday,,Thursday,Friday,,";
//parts is [Monday, Tuesday, , Thursday,Friday]
String[] parts = testString.split(",");

```
可以看到，String.split方法省略了最后的2个空串。在有些时候，这个做法是你需要的，但是这些事情是应该由程序员来决定是否省略。Splitter类可以帮助我们实现与Joiner类相反的功能。Splitter可以使用单个字符、固定字符串、正则表达式串、正则表达式对象或者CharMatcher对象（另一个Guava的类，本章会讲到）来分割字符串。可以给定具体分割符来创建Splitter对象然后使用。一旦拥有了Splitter实例后就可以调用split方法，并且会返回包含分割后字符串的迭代器对象。
```

Splitter.on('|').split("foo|bar|baz");
Splitter splitter = Splitter.on("\\d+");

```
在上面的例子当中，我们看到一个Splitter 实例使用了'|'字符分割，另外一个实例使用了正则表达式进行分割。
Splitter有一个可以处理前面空格和后面空格的方法是trimResults。
```

//Splits on '|' and removes any leading or trailing whitespace
Splitter splitter = Splitter.on('|').trimResults();

```
与Joiner类一样Splitter类同样是一个不可变的类，所以在使用的时候应该使用调用trimResults方法后返回的Splitter实例。
```

Splitter splitter = Splitter.on('|');
//Next call returns a new instance, does not
modify the original!
splitter.trimResults();
//Result would still contain empty elements
Iterable<String> parts = splitter.split("1|2|3|||");

```
Splitter 类，像Joiner与MapJoiner一样也有MapSplitter类。MapSplitter类可以将字符串转换成Map实例返回，并且元素的顺序与字符串给定的顺序相同。使用下面方法构造一个MapSplitter实例：
```

//MapSplitter is defined as an inner class of Splitter
Splitter.MapSplitter mapSplitter = Splitter.on("#"). withKeyValueSeparator("=");

```
###  使用Strings类 
Strings类提供一些便利实用的方法处理字符串。你是否写过像下面的代码：
```

StringBuilder builder = new StringBuilder("foo");
　　char c = 'x';
　　for(int i=0; i<3; i++){
　　　　builder.append(c);
　　}
return builder.toString();

```
上面例子当中的6行代码我们可以使用一行代码来替换。
```

Strings.padEnd("foo",6,'x');

```
第二个参数是很重要的，6指定返回的字符串最小长度为6，而不是指定'x'字符在字符串后面追加多少次。如果提供的字符串长度已经大于了6，则不会进行追加。同样也有一个相类似的padStart方法可以在给定字符串的前面追加字符到指定的长度。
在Strings类当中有三个非常有用的方法来**处理空值**的：
  * nullToEmpty：这个方法接受一个字符串参数，如果传入的参数不是null值或者长度大于0则原样返回，否则返回空串("")；
  * emptyToNull：这个方法类似于nullToEmpty，它将返回null值如果传入的参数是空串或者null。
  * isNullOrEmpty：这个方法会检查传入参数是否为null和长度，如果是null和长度为0就返回true。
或许，**总是在任何使用nullToEmpty方法处理传入的字符串参数将是一个不错的主意**