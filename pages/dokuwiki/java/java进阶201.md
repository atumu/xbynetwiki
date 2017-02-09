title: java进阶201 

#  java Synthetic Method 
翻看guava eventbus源码时看到这一段:
```

for (Method method : supertype.getDeclaredMethods()) {
        if (method.isAnnotationPresent(Subscribe.class) && !method.isSynthetic()) {

```
**method.isSynthetic()为什么还需要这样判断,Synthetic方法是什么方法？**
The Java Language Specification (section 13.1)Java语言规范13章写道 ：` 由编译器产生的任何构建，如果在源码中没有对应的构建存在，那么这个构建就必须被标记为synthetic(除了默认构造器和类初始化方法。) `原话如下:
"Any constructs introduced by the compiler that do not have a corresponding construct in the source code must be marked as synthetic, except for default constructors and the class initialization method." 
JavaDoc文档中Member.isSynthetic()对此也有说明: 当某个成员是由编译器自己生成的时，返回true。
其实说来说去，也就一句话："synthetic"是由编辑器产生的一个java构建(a Java construct introduced by the compiler.)

当存在内嵌class定义，而且需要在外包class和内嵌class之间访问对方的private修饰的属性的时候，java编译器必须创建synthetic method.
(请记住:对于java编译器而言，内部类也会被单独编译成一个class文件。那么原有代码中的相关属性可见性就难以维持，synthetic method也就是为了这个目的而生成的。生成的synthetic方法是包访问性的static方法.)
请看如下实例:
**1、嵌套类访问外包类的私有属性**
```

public class Foo {
  private Object baz = "Hello";
  private class Bar {
    private Bar() {
      System.out.println(baz);
    }
  }
}

```
然后Foo生成的签名。以人类友好形式说明类似于如下:(通过javap工具)
```

public class Foo extends java.lang.Object{
    public Foo();
    static java.lang.Object access$000(Foo);
}

```
access$000方法是自动生成的:它是为了用于Bar类访问Foo类中私有属性baz而由编译器自动创建的，它会被标记为Synthetic方法. 
接下来我们试试私有方法: 下面的get()是个私有方法.
```

public class Foo {
  private Object baz = "Hello";
  private int get(){
  	return 1;
  }
  private class Bar {
    private Bar() {
      System.out.println(get());
    }
  }
}

```
我们使用javap  -private  Foo打印看一下:
```

public class Foo {
  private java.lang.Object baz;
  public Foo();
  private int get();
  static int access$000(Foo); //多出来的sythetic方法，为了在B中的这段代码System.out.println(get());
}

```
另外一个例子：
**2、外包类访问嵌套类私有属性**
```

import java.lang.String;

public class A{
	private static class B{
		private String b1="b111";
		private String b2="b2222";
	}
	public static void main(String[] args){
		A.B b=new A.B();
		String tmp=b.b1;
          	String tmp1=b.b2;
	}
}

```
运行javap -private A.B输出如下:
```

class A$B {
  private java.lang.String b1;
  private java.lang.String b2;
  private A$B();
  A$B(A$1);
  static java.lang.String access$100(A$B);
  static java.lang.String access$200(A$B);
}

```
生成了两个synthetic方法，分别对应于String tmp=b.b1;String tmp1=b.b2;访问两个私有属性。

**附录：javap工具使用**
javap是JDK自带的反汇编器，可以查看java编译器为我们生成的字节码。通过它，可以对照源代码和字节码，从而了解很多编译器内部的工作。
可以在命令行窗口先用javap -help看下javap工具支持的选项：C:\>javap -help 
```

Usage: javap <options> <classes>...
where options include:
   -c                      输出类中各方法的未解析的代码，即构成java字节码的指令
   -classpath <pathlist>       指定javap用来查找类的路径。目录用：分隔
   -extdirs <dirs>             覆盖搜索安装方式扩展的位置，扩展的缺省位置为jre/lib/ext
   -help                    输出帮助信息
   -J<flag>                  直接将flag传给运行时系统
   -l                       输出行及局部变量表
   -public                   只显示public类及成员
   -protected                只显示protected和public类及成员。
   -package                 只显示包、protected和public类及成员，，这是缺省设置
   -private                  显示所有的类和成员
   -s                        输出内部类型签名
   -bootclasspath <pathlist>    指定加载自举类所用的路径，如jre/lib/rt.jar或i18n.jar
   -verbose                 打印堆栈大小、各方法的locals及args参数，以及class文件的编译版本

```

参考：
http://www.javaworld.com/article/2073578/java-s-synthetic-methods.html
http://stackoverflow.com/questions/5557955/whats-the-penalty-for-synthetic-methods
http://aruld.info/synthetic-methods-in-java/
http://docs.oracle.com/javase/6/docs/technotes/tools/windows/javap.html
http://blog.csdn.net/zhaozheng7758/article/details/8623526