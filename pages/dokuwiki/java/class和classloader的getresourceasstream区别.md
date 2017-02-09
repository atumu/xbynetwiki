title: class和classloader的getresourceasstream区别 

#  Class和ClassLoader的getResourceAsStream区别 
参考：http://blog.chinaunix.net/uid-21227800-id-65874.html
这两个方法还是略有区别的， 以前一直不加以区分，直到今天发现要写这样的代码的时候运行 
错误， 才把这个问题澄清了一下。 

基本上，两个都可以用于从 classpath 里面进行资源读取，  classpath包含classpath中的路径 
和classpath中的jar。 
两个方法的区别是资源的定义不同，** 一个主要用于相对与一个object取资源，而另一个用于取相对于classpath的 
资源，用的是绝对路径。** 
  * 在使用` Class.getResourceAsStream ` 时， 资源路径有两种方式， **一种以 / 开头，则这样的路径是指定绝对 
路径， 如果不以 / 开头， 则路径是相对与这个class所在的包的。** 
  * 在使用` ClassLoader.getResourceAsStream `时， 路径直接使用` 相对于classpath `的绝对路径，**但是不要在最前面加"/"** 

举例，下面的三个语句，实际结果是一样的： 
```

com.explorers.Test.class.getResourceAsStream("abc.jpg") 
= com.explorers.Test.class.getResourceAsStream("/com/explorers/abc.jpg") 
= ClassLoader.getResourceAsStream("com/explorers/abc.jpg")

```