title: java7变化1 

#  java7变化之语法变化 
Java程序员修炼之道源码:https://www.manning.com/books/the-well-grounded-java-developer
  * 语法糖-数字下划线 如100_000
  * switch支持String,再加上以前的int,char,byte,short
  * 泛型钻石语法.如Map<String,Object> map=new HashMap<>()
  * try-with-resources语句.如try(BufferedReader reader=Files.newBufferedReader(file,StandardCharsets.UTF_8,StandardOpenOption.READ)){}
  * 提升的异常处理能力.:multicatch和final重抛。
  * JVM新特性-动态调用
##  try-with-resources语句(TWR) 
**TWR与AutoCloseable接口(还有一个父接口Closeable)。**TWR特性依赖于AutoCloseable接口的思想。所有需要用到try-with-resources语句的资源类必须实现这个接口。
```

try(BufferedReader reader=Files.newBufferedReader(file,StandardCharsets.UTF_8,StandardOpenOption.READ)){

}

```
**多个资源的情形**
```

try(BufferedReader reader=Files.newBufferedReader(file,StandardCharsets.UTF_8,StandardOpenOption.READ); //加个分号即可
	BufferedWriter writer=Files.newBufferedWriter(file2,StandardCharsets.UTF_8,StandardOpenOption.WRITE)
	){

}

```
##  提升的异常处理能力.:multicatch和final重抛 
```

try{

}catch(FileNotFoundException |ParseException e){

}catch(IOException e){

}

```
```

try{

}catch(final Exception e){
	throw e //这就是final重抛，抛出的e会是实际的异常类型(如IOException)，而不是笼统的Exception。比如外层捕获实际的IOException
}

```

