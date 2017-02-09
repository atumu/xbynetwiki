title: commons_lang3 

#  commons组件学习系列记录之lang3 

#  一.org.apache.commons.lang3包 
1. ArrayUtils
2.BooleanUtils:能优雅地处理null值等方便转换。
3.RandomUtils自从commons lang3.3发布之后便有了这个类：用于产生一个范围内的随机数。
4.RandomStringUtils,可以用于产生数字，字符及其组合等随机数
5.SerializationUtils增强的功能：深拷贝，方便的反序列化，序列化等。
7.SystemUtils:如果无法取得，不会抛出异常，返回null：用于取得系统环境的相关特殊目录路径。
8.NumberUtils:它的所有操作都不会抛出异常，如果转换不成功返回0,0.0d,0.0f等形式。转换操作也可以指定默认值     

  * 用于从字符串解析并创建对应包装类
  * 用于判断字符串是否为数字形式
  * 用于转换字符串为对应基本类型。
  * 从数组或一组值中判断最小值最大值min(),max()                                  
9.ClassUtils:可以不用反射就可以操作java类。
10:StringEscapeUtils:字符串转义工具： xml,html特殊字符转义等

##  1. ArrayUtils 

ArrayUtils能优雅地处理数组中的null值输入。它不会抛出异常。
内有很多静态方法用于创建数组，及数组操作。当插入null值时也不会抛出异常
比如向数组添加元素，元素为null则不会抛出异常，而是什么都不做：
ArrayUtils.add([true,false],null)=[true,false]
##  2.BooleanUtils:能优雅地处理null值等方便转换。 

static Boolean toBooleanObject(String str)
static boolean toBoolean(String str)
static boolean isTrue(Boolean bool)
static boolean isFalse(Boolean bool)
static boolean isNotTrue(Boolean bool)
注意区别：
BooleanUtils.isNotTrue(null)          = true
 BooleanUtils.isFalse(null)          = false
##  3.RandomUtils 
自从commons lang3.3发布之后便有了这个类：用于产生一个范围内的随机数。
RandomUtils帮助我们产生随机数，不止是整数类型。
 注意这里传入的参数不是随机种子,而是在0~1000之间产生一位随机数
RandomUtils.nextInt(0，1000);
此外还有nextBytes/Double/Float/Long等
##  4.RandomStringUtils, 
可以用于产生数字，字符及其组合等随机数
/产生5位长度的随机字符串，中文环境下是乱码
RandomStringUtils.random(5);
//使用指定的字符生成5位长度的随机字符串
RandomStringUtils.random(5, new char[]{'a','b','c','d','e','f', '1', '2', '3'});
//生成指定长度的字母和数字的随机组合字符串
RandomStringUtils.randomAlphanumeric(5);
//生成随机数字字符串
RandomStringUtils.randomNumeric(5);
//生成随机[a-z]字符串，包含大小写
RandomStringUtils.randomAlphabetic(5);
生成从ASCII 32到126组成的随机字符串
RandomStringUtils.randomAscii(4)

##  5.SerializationUtils 
```

增强的功能：深拷贝，方便的反序列化，序列化等。
   Deep clone using serialization
   Serialize managing finally and IOException
   Deserialize managing finally and IOException
clone(T obj): deep clone
deseralize(byte[] objData);
deserialize(InputStream inputStream，[OutputStream out])反序列化

```
##  6.StringUtils: 
包含判断是否为空，trim及查找字符，分割，联合，子集，取得索引，切换大小写，替换，删除等功能
及其判断是否为数字，是否为字符等
特性：
```

   IsEmpty/IsBlank - checks if a String contains text
   Trim/Strip - removes leading and trailing whitespace
   Equals - compares two strings null-safe
   startsWith - check if a String starts with a prefix null-safe
   endsWith - check if a String ends with a suffix null-safe
   IndexOf/LastIndexOf/Contains - null-safe index-of checks
       Substring/Left/Right/Mid - null-safe substring extractions
       SubstringBefore/SubstringAfter/SubstringBetween - substring extraction relative to other strings
       Split/Join - splits a String into an array of substrings and vice versa
       Remove/Delete - removes part of a String
       Replace/Overlay - Searches a String and replaces one String with another
   UpperCase/LowerCase/SwapCase/Capitalize/Uncapitalize - changes the case of a String          
   DefaultString - protects against a null input String
   Reverse/ReverseDelimited - reverses a String
最常用：
static boolean isEmpty(CharSequence cs)
static boolean isNumeric(CharSequence cs)

```

##  7.SystemUtils: 
如果无法取得，不会抛出异常，返回null：用于取得系统环境的相关特殊目录路径。

getJavaHome(),getJavaIoTmpDir;getUserDir(); getUserHome(),

##  8.NumberUtils: 
它的所有操作都不会抛出异常，如果转换不成功返回0,0.0d,0.0f等形式。转换操作也可以指定默认值

用于从字符串解析并创建对应包装类
用于判断字符串是否为数字形式
用于转换字符串为对应基本类型。
从数组或一组值中判断最小值最大值min(),max()
```

NumberUtils.toDouble(null)   = 0.0d
  NumberUtils.toDouble("")     = 0.0d
  NumberUtils.toDouble("1.5")  = 1.5d

NumberUtils.toLong("", 1L)   = 1L
  NumberUtils.toLong("1", 0L)  = 1L
注意isNUmber与isDigits的区别。一般使用isDigits
/*1.NumberUtils.isNumber():判断字符串是否是数字*/
NumberUtils.isNumber("5.96");//结果是true
NumberUtils.isNumber("s5");//结果是false
NumberUtils.isNumber("0000000000596");//结果是true
/*2.NumberUtils.isDigits():判断字符串中是否全为数字*/
NumberUtils.isDigits("0000000000.596");//false
NumberUtils.isDigits("0000000000596");//true
/*5.NumberUtils.min():找出最小的一个*/
NumberUtils.min(newint[]{3,5,6});//结果是6
NumberUtils.min(3,1,7);结果是7

```

##  9.ClassUtils: 
可以不用反射就可以操作java类。
##  10:StringEscapeUtils 
:字符串转义工具： xml,html特殊字符转义等

escapeHtml4(String input)；
escapeXml(String input)；
unescapeHtml4(String input)


#  二、org.apache.commons.lang3.builder 

Assists in creating consistentequals(Object),toString(),hashCode(), andcompareTo(Object)methods.
##  1.CompareToBuilder: 

使用举例：
Assists in implementing Comparable.compareTo(Object) methods. I
```

public int compareTo(Object o) {
    MyClass myClass = (MyClass) o;
    return new CompareToBuilder()
      .appendSuper(super.compareTo(o)
      .append(this.field1, myClass.field1)
      .append(this.field2, myClass.field2)
      .append(this.field3, myClass.field3)
      .toComparison();
  }
  </code>
##  2.EqualsBuilder 

Assists in implementing Object.equals(Object) methods.
Typical use for the code is as follows:
<code>
public boolean equals(Object obj) {
  if (obj == null) { return false; }
  if (obj == this) { return true; }
  if (obj.getClass() != getClass()) {
    return false;
  }
  MyClass rhs = (MyClass) obj;
  return new EqualsBuilder()
                .appendSuper(super.equals(obj))
                .append(field1, rhs.field1)
                .append(field2, rhs.field2)
                .append(field3, rhs.field3)
                .isEquals();
 }

```
##  3.HashCodeBuilder 

Assists in implementing Object.hashCode() methods.
To use this class write code as follows:
```

public class Person {
  String name;
  int age;
  boolean smoker;
  ...
  public int hashCode() {
    // you pick a hard-coded, randomly chosen, non-zero, odd number
    // ideally different for each class
    return new HashCodeBuilder(17, 37).
      append(name).
      append(age).
      append(smoker).
      toHashCode();
  }
}

```
##  4.ToStringBuilder 

Assists in implementing Object.toString() methods.
To use this class write code as follows:
```

public class Person {
  String name;
  int age;
  boolean smoker;
  ...
  public String toString() {
    return new ToStringBuilder(this).
      append("name", name).
      append("age", age).
      append("smoker", smoker).
      toString();
  }
}

```

#  三、org.apache.commons.lang3.concurrent 


Provides support classes for multi-threaded programming.
TimedSemaphore

#  四、org.apache.commons.lang3.event 


Provides some useful event-based utilities.
 EventListenerSupport<L>:可以管理监听器列表，并可以以特定事件触发fire()所有监听器。
EvenUtils：可用于添加监听器到指定事件源。也可以绑定特定事件到相关方法。
1.EventListener:
```

public class MyActionEventSource
{
  private EventListenerSupport actionListeners =
      EventListenerSupport.create(ActionListener.class); //创建针对于特定监听器的对象。
  public void someMethodThatFiresAction()
  {//创建一个Event对象。
    ActionEvent e = new ActionEvent(this, ActionEvent.ACTION_PERFORMED, "somethingCool");
    actionListeners.fire().actionPerformed(e);//触发事件通知所有的listeners
  }
}

```
注意序列化EventListenerSupport并不会序列化它维护的listener list

2.EventUtils:
```

addEventListener(Object eventSource, Class<L> listenerType,
                                L listener)
bindEventsToMethod(Object target, String methodName, Object eventSource, Class<L> listenerType, String... eventTypes)
          Binds an event listener to a specific method on a specific object.

```
七、org.apache.commons.lang3.reflect 
#  九、org.apache.commons.lang3.time 

时间或日期模式：
character	duration element
y	years
M	months
d	days
H	hours
m	minutes
s	seconds
S	milliseconds
DateFormatUtils:用于时间日期格式化。或将毫秒格式化为时间。format
DateUtils:用于操作时间或日期的加减。
1.DateFormatUtils:
static String format(long millis, String pattern)
static String format(Date date, String pattern)
2.DateUtils:
static Date addDays(Date date, int amount)
addYears(Date date, int amount)
#  十、org.apache.commons.lang3.tuple 


用于处理一对键值的对象pair类似于Map.entry
commons lang3增加了可以处理3个值的Triple基类
此包下定义了Pair<L,R>抽象基类，及MutablePair,MutableTriple,ImmutablePair,ImmutableTriple子类
一个线程非安全，另一个线程安全
接口：
Pair：封装一对键值对。
实现类：
  * 可变：MutablePair<L,R>
  * 不可变：ImmutablePair
接口：
Triple：封装3个值的类。
实现类：
ImmutableTriple; MuttableTriple<L,M,R>