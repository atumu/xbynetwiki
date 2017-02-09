title: gson 

#  Json处理之Gson 

项目地址:https://github.com/google/gson
Gson：java的json解析库。（其他类似的有json-lib,Jackson,fastson）
核心类：Gson或者GsonBuilder
#  使用jsonschema2pojo来创建POJO 

1、通过网站http://www.jsonschema2pojo.org/在线创建:选择源代码类型为Json，注解类型是Gson,然后点击preview.
2、通过命令行工具:jsonschema2pojo.bat -a GSON -T JSON --source jsonaddress --target java-gen

#  使用Gson 

你可以使用new Gson()或者通过GsonBuilder来构建Gson对象。
```

(Serialization)  
Gson gson = new Gson();  
gson.toJson(1);            ==> prints 1  
gson.toJson("abcd");       ==> prints "abcd"  
gson.toJson(new Long(10)); ==> prints 10  
int[] values = { 1 };  
gson.toJson(values);       ==> prints [1]  
  
(Deserialization)  
int one = gson.fromJson("1", int.class);  
Integer one = gson.fromJson("1", Integer.class);  
Long one = gson.fromJson("1", Long.class);  
Boolean false = gson.fromJson("false", Boolean.class);  
String str = gson.fromJson("\"abc\"", String.class);  
String anotherStr = gson.fromJson("[\"abc\"]", String.class);  
[java] view plaincopy
class BagOfPrimitives {  
  private int value1 = 1;  
  private String value2 = "abc";  
  private transient int value3 = 3;  
  BagOfPrimitives() {  
    // no-args constructor  
  }  
}  
  
(Serialization)  
BagOfPrimitives obj = new BagOfPrimitives();  
Gson gson = new Gson();  
String json = gson.toJson(obj);    
==> json is {"value1":1,"value2":"abc"}  
  
Note that you can not serialize objects with circular references since that will result in infinite recursion.   
  
(Deserialization)  
BagOfPrimitives obj2 = gson.fromJson(json, BagOfPrimitives.class);     
==> obj2 is just like obj  

```
#  Nested Classes (including Inner Classes) 

Gson可以很容易地序列化静态static的内部类（但是非静态的这不能）
##  Array Examples 
```

Gson gson = new Gson();  
int[] ints = {1, 2, 3, 4, 5};  
String[] strings = {"abc", "def", "ghi"};  
  
(Serialization)  
gson.toJson(ints);     ==> prints [1,2,3,4,5]  
gson.toJson(strings);  ==> prints ["abc", "def", "ghi"]  
  
(Deserialization)  
int[] ints2 = gson.fromJson("[1,2,3,4,5]", int[].class);

```
##  Collections Examples 
```

Gson gson = new Gson();  
Collection<Integer> ints = Lists.immutableList(1,2,3,4,5);  
  
(Serialization)  
String json = gson.toJson(ints); ==> json is [1,2,3,4,5]  
  
(Deserialization)  
Type collectionType = new TypeToken<Collection<Integer>>(){}.getType();  
Collection<Integer> ints2 = gson.fromJson(json, collectionType);  

```
##  Serializing and Deserializing Generic Types 
```

Type fooType = new TypeToken<Foo<Bar>>() {}.getType();  
gson.toJson(foo, fooType);  
gson.fromJson(json, fooType);  

```
##  Serializing and Deserializing Collection with Objects of Arbitrary Types 

Sometimes you are dealing with JSON array that contains mixed types. For example:
<blockquote>['hello',5,{name:'GREETINGS',source:'guest'}]</blockquote>
```

public class RawCollectionsExample {  
  static class Event {  
    private String name;  
    private String source;  
    private Event(String name, String source) {  
      this.name = name;  
      this.source = source;  
    }  
    @Override  
    public String toString() {  
      return String.format("(name=%s, source=%s)", name, source);  
    }  
  }  
  
  @SuppressWarnings({ "unchecked", "rawtypes" })  
  public static void main(String[] args) {  
    Gson gson = new Gson();  
    Collection collection = new ArrayList();  
    collection.add("hello");  
    collection.add(5);  
    collection.add(new Event("GREETINGS", "guest"));  
    String json = gson.toJson(collection);  
    System.out.println("Using Gson.toJson() on a raw collection: " + json);  
    JsonParser parser = new JsonParser();  
    JsonArray array = parser.parse(json).getAsJsonArray();  
    String message = gson.fromJson(array.get(0), String.class);  
    int number = gson.fromJson(array.get(1), int.class);  
    Event event = gson.fromJson(array.get(2), Event.class);  
    System.out.printf("Using Gson.fromJson() to get: %s, %d, %s", message, number, event);  
  }  
}  

```
#  Compact Vs. Pretty Printing for JSON Output Format 

默认情况下Gson的json输出是没有空格与忽略null值得，所以不是很友好。我们可以采用pretty输出,同时输出null
```

Gson gson = new GsonBuilder().setPrettyPrinting().serialzeNulls().create(); 

```
NOTE: when serializing nulls with Gson, it will add a JsonNull element to the JsonElement structure. Therefore, this object can be used in custom serialization/deserialization.
###  Versioning Support 

本文不做介绍
##  Excluding Fields From Serialization and Deserialization 

默认情况下transient 和static field是被忽略的。但是如果你想包含某些transient域可以采取如下形式：
```

import java.lang.reflect.Modifier;  
  
Gson gson = new GsonBuilder()  
    .excludeFieldsWithModifiers(Modifier.STATIC)  
    .create();  
  
NOTE: you can use any number of the Modifier constants to "excludeFieldsWithModifiers" method.  For example:  
Gson gson = new GsonBuilder()  
    .excludeFieldsWithModifiers(Modifier.STATIC, Modifier.TRANSIENT, Modifier.VOLATILE)  
    .create();  

```
不过有更好的形式可以**采取@Expose注解**：然后调用 new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()创建Gson` 凡是被@Expose注解的域都会被包括，没有被注解的都被忽略。 `
##  JSON Field Naming Support  @SerializedName 
```

private class SomeObject {  
  @SerializedName("custom_naming") private final String someField;  
  private final String someOtherField;  
  
  public SomeObject(String a, String b) {  
    this.someField = a;  
    this.someOtherField = b;  
  }  
}  
  
SomeObject someObject = new SomeObject("first", "second");  
Gson gson = new GsonBuilder().setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE).create();  
String jsonRepresentation = gson.toJson(someObject);  
System.out.println(jsonRepresentation);  
  
# == OUTPUT ==  
{"custom_naming":"first","SomeOtherField":"second"}  

```
#  Gson2.3新的功能 

  * New Methods in JsonArray
  * @TypeAdapter Annotation
  * JsonPath Support

参考：
https://sites.google.com/site/gson/gson-user-guide
http://www.studytrails.com/java/json/java-google-json-new-2.3.jsp