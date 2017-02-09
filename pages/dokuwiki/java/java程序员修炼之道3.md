title: java程序员修炼之道3 

#  Java程序员修炼之道之类文件和字节码 
  * 类加载机制与自定义ClassLoader
  * javap工具分析类文件
  * Java7新增的方法句柄
  * JVM字节码介绍
##  类加载和类对象 
类是JVM平台能加载的最小程序代码单元。
JVM必须以字节数据流的方式取出类文件中的内容，并将其转换成可用的格式加入运行态中。分为两步:加载和连接.连接又分为验证，准备和解析
**Class对象**
###  类加载器 
类加载器负责加载Class对象和资源。
Java平台里自带的类加载器:
  * 根(或引导)类加载器(BootstrapClassLoader)——通常在VM启动后不久实例化，一般用本地代码实现。负责加载系统的基础JAR(主要是rt.jar)，而且它不做验证工作。
  * 扩展类加载器——用来加载自带的标准扩展。
  * 应用(或系统)类加载器(application classloader或system classloader)——它负责加载应用类。` 在JSE标准版环境中，主要工作是由它来完成 `。(main)
  * ` 定制类加载器——在JavaWeb或JEE环境或比较复杂的框架中，通常会定制类加载器(比如tomcat加载webapp时使用的就是定制类加载器。) `
更多参考[[java:java_classloader机制]]
##  方法句柄 
Java 7增加了一个新的特性，使得Java语言对动态更好的支持，而且性能也有很大的提升，那就是方法句柄。
方法句柄是对Java中方法、构造器、字段的一个强类型的可执行的引用。通过方法句柄可以直接调用该句柄所引用的底层方法。
从作用上来说，方法句柄的作用类似于反射中的Method类，但方法句柄的功能更强大，使用更灵活，性能更好。
方法句柄的类型
方法句柄是由` java.lang.invoke.MethodHandle `类表示的。**方法句柄的类型(` MethodType `)完全由它的返回类型和参数类型来确定的，和它所引用的底层方法的名称和所在的类没有关系。**比如String类的length方法和Integer类的intValue方法的方法句柄类型是一样的。MethodType类的对象实例只能通过MethodType类中的静态工厂方法创建。这样的工厂方法工有三类。
创建方法句柄的类型
第一类是通过指定返回值和参数类型来创建MethodType，有多种重载，如下：
static MethodType	methodType(Class<?> rtype)
static MethodType	methodType(Class<?> rtype, Class<?> ptype0)
static MethodType	methodType(Class<?> rtype, Class<?>[] ptypes)
static MethodType	methodType(Class<?> rtype, Class<?> ptype0, Class<?>... ptypes)
static MethodType	methodType(Class<?> rtype, List<Class<?>> ptypes)
static MethodType	methodType(Class<?> rtype, MethodType ptypes)
**第一个参数rtype是返回类型，返回值类型是必须有的，如果返回值是void类型，可以用java.lang.Void.class或void.class来声明。可以有0或多个参数类型。**
其余两类忽略
创建MethodType的Demo如下：
```

//第一类
MethodType mt1 = MethodType.methodType(int.class);  
MethodType mt2 = MethodType.methodType(String.class, String.class);
//第二类
MethodType mt3 = MethodType.genericMethodType(3);  
MethodType mt4 = MethodType.genericMethodType(2, true);  
//第三类
ClassLoader cl = this.getClass().getClassLoader();  
String descriptor = "(Ljava/lang/String;)Ljava/lang/String;";  
MethodType mt5 = MethodType.fromMethodDescriptorString(descriptor, cl);

```
###  获取方法句柄 
**方法句柄的查找**是通过` java.lang.invoke.MethodHandles.Lookup `类来完成的。**首先调用MethodHandles.lookup方法获取MethodHandles.Lookup对象**，.MethodHandles.Lookup类提供了一些方法根据不同的条件进行查找：
```

//分别是字段Get和Set方法句柄，第一个参数为类的class对象，第二个参数为字段名，第三个参数为字段数据类型；
lookup.findGetter(C.class,"f",FT.class)
lookup.findSetter(C.class,"f",FT.class)
//分别是静态字段Get和Set方法句柄，第一个参数为类的class对象，第二个参数为字段名，第三个参数为字段数据类型；
lookup.findStaticGetter(C.class,"f",FT.class)
lookup.findStaticSetter(C.class,"f",FT.class)
//分别是一般方法和静态方法的方法句柄，第一个参数为类的class对象，第二个参数为方法名，第三个参数为MethodType对象；
lookup.findVirtual(C.class,"m",MT)
lookup.findStatic(C.class,"m",MT)
//构造器的方法句柄，第一个参数为类的class对象，第二个为MethodType对象；
lookup.findConstructor(C.class,MT)
//查找类中的特殊方法，主要是类中的私有方法。
lookup.findSpecial(C.class,"m",MT,this.class)

```
**其中findSpecial方法比之前的findVirtual和findStatic等方法多了一个参数。这个额外的参数用来指定私有方法被调用时所使用的类。**提供这个类的原因是为了满足对私有方法的访问控制的要求。当方法句柄被调用时，指定的调用类必须具备访问私有方法的权限，否则会出现无法访问的错误。
更多可以参考API：http://docs.oracle.com/javase/7/docs/api/
![](/data/dokuwiki/java/pasted/20160327-234749.png)

###  方法句柄的调用 
**在获取一个方法句柄后，最直接的使用方法就是调用它所引用的底层方法。比较常用的方法是invokeExact**:
Object	invokeExact(Object... args)
invokeExact的参数是可变的，参数一次是作为方法接受者的对象和调用时候的实际参数列表。但如果是静态方法，则不要指定接收对象，参数都是实际参数列表。
Demo如下：
```

import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;
public class ReflectTest {
	
	public static void main(String[] args) throws Throwable{ 
		User user=new User("kevin");
		MethodHandles.Lookup lookkup=MethodHandles.lookup();
		MethodHandle mth=lookkup.findVirtual(User.class, "SayHello", MethodType.methodType(void.class,String.class));
		mth.invokeExact(user,"oseye");
	}
}
class User{
	private String name;
	public User(String name){this.name=name;}
	public void SayHello(String name){
		System.out.println(this.name+" Say Hello,"+name);
	}
}

```

使得Java语言对动态更好的支持，而且性能也有很大的提升。就让我们实际测试来看看结果对比吧.

###  访问私有属性或方法 
主要思路是
  * 先通过反射的getDeclaredXxx获取私有属性或方法。
  * 然后调用setAccessible(true)设置可见性。
  * 然后调用MethodHandle.Lookup.unreflectXxx方法得到方法句柄对象MethodHandle.之后便可使用方法句柄进行操作。

1、访问私有方法：
```

public static void main(String[] args) {
    Lookup lookup = MethodHandles.lookup();
    NestedTestClass ntc = new Program().new NestedTestClass();
    try {
        // Grab method using normal reflection and make it accessible
        Method pm = NestedTestClass.class.getDeclaredMethod("gimmeTheAnswer");
        pm.setAccessible(true);
        // Now convert reflected method into method handle
        MethodHandle pmh = lookup.unreflect(pm);
        System.out.println("reflection:" + pm.invoke(ntc));
        // We can now revoke access to original method
        pm.setAccessible(false);
        // And yet the method handle still works!
        System.out.println("handle:" + pmh.invoke(ntc));
        // While reflection is now denied again (throws exception)
        System.out.println("reflection:" + pm.invoke(ntc));

    } catch (Throwable e) {
        e.printStackTrace();
    }
}

```
2、访问私有属性:
```

	public static int numAddMethodHandle(int loops) throws Throwable{
		User user=new User();
		MethodHandles.Lookup lookkup=MethodHandles.lookup();
		Field field=user.getClass().getDeclaredField("num");
		field.setAccessible(true);
		MethodHandle mthGet=lookkup.unreflectGetter(field);
		MethodHandle mthSet=lookkup.unreflectSetter(field);
		long startTime=0;
		for(int i=0;i<loops;i++){
			if(i==0){startTime=System.nanoTime();}
			mthSet.invokeExact(user,(int)mthGet.invokeExact(user)+i);
		}
		long totalTime=System.nanoTime()-startTime;
		System.out.println("方法句柄调用字段总的纳秒时间：\t"+totalTime);
		return user.num;
	}

```
参考:http://stackoverflow.com/questions/19135218/invoke-private-method-with-java-lang-invoke-methodhandle
###  性能对比测试代码 
```

import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.reflect.Field;
 
public class ReflectTest {
	//直接调用
	public static int numAdd(int loops){
		int val=0;
		long startTime=0;
		for(int i=0;i<loops;i++){
			if(i==0){startTime=System.nanoTime();}
			val+=i;
		}
		long totalTime=System.nanoTime()-startTime;
		System.out.println("直接调用总的纳秒时间：\t\t"+totalTime);
		return val;
	}
	//引用调用字段
	public static int numAddReference(int loops){
		User user=new User();
		long startTime=0;
		for(int i=0;i<loops;i++){
			if(i==0){startTime=System.nanoTime();}
			user.num+=i;
		}
		long totalTime=System.nanoTime()-startTime;
		System.out.println("引用调用字段总的纳秒时间：\t"+totalTime);
		return user.num;
	}
	//反射调用字段
	public static int numAddReflection(int loops) throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException{
		User user=new User();
		Class<?> cUser=user.getClass();
		long startTime=0;
		for(int i=0;i<loops;i++){
			if(i==0){startTime=System.nanoTime();}
			Field field=cUser.getField("num");
			field.set(user, field.getInt(user)+i);
		}
		long totalTime=System.nanoTime()-startTime;
		System.out.println("反射调用字段总的纳秒时间：\t"+totalTime);
		return user.num;
	}
	//方法句柄
	public static int numAddMethodHandle(int loops) throws Throwable{
		User user=new User();
		MethodHandles.Lookup lookkup=MethodHandles.lookup();
		MethodHandle mthGet=lookkup.findGetter(User.class, "num", int.class);
		MethodHandle mthSet=lookkup.findSetter(User.class, "num", int.class);
		long startTime=0;
		for(int i=0;i<loops;i++){
			if(i==0){startTime=System.nanoTime();}
			mthSet.invokeExact(user,(int)mthGet.invokeExact(user)+i);
		}
		long totalTime=System.nanoTime()-startTime;
		System.out.println("方法句柄调用字段总的纳秒时间：\t"+totalTime);
		return user.num;
	}
	
	public static void main(String[] args) throws Throwable{ 
		int loops=1000000;
		numAdd(loops);
		numAddReference(loops);
		numAddReflection(loops);
		numAddMethodHandle(loops);
	}
}
class User{
	public int num;
}

```
输出：
直接调用总的纳秒时间：		4869897
引用调用字段总的纳秒时间：	3907301
反射调用字段总的纳秒时间：	733545592
方法句柄调用字段总的纳秒时间：	18029318

###  日志调用中使用MethodHandle 
以前写法:
Logger log=LoggerFactory.getLogger(MyClass.class)
使用方法句柄方式:
Logger log=LoggerFactory.getLogger(MethodHandles.lookup().looupClass());


##  检查类文件 
###  javap工具 
JDK自带了一个叫javap的工具可以用来探视类文件内部信息和反汇编类文件。
  * javap MyClass.class输出public、protected和默认级别的方法
  * javap -p MyClass.class可以输出private方法或域
  * javap -s MyClass.class相比不带选项参数时可以输出签名的类型描述符，比较常用
  * javap –v MyClass.class，查看字节码完整内容,包括常量池
  * javap –c MyClass.class，对类进行反编译为一些操作码组成的东西。
最简单情形:
```

public class WorkUnit<T> {
  private final T workUnit;

  public T getWork() {
    return workUnit;
  }

  public WorkUnit(T workUnit_) {
    workUnit = workUnit_;
  }
}

```
```

//javap WorkUnit.class
chapter5>javap WorkUnit.class
Compiled from "WorkUnit.java"
public class com.java7developer.chapter5.WorkUnit<T> {
  public T getWork();
  public com.java7developer.chapter5.WorkUnit(T);
}
//javap -s WorkUnit.class
chapter5>javap -s WorkUnit.class
Compiled from "WorkUnit.java"
public class com.java7developer.chapter5.WorkUnit<T> {
  public T getWork();
    Signature: ()Ljava/lang/Object;

  public com.java7developer.chapter5.WorkUnit(T);
    Signature: (Ljava/lang/Object;)V
}
//javap -p WorkUnit.class
chapter5>javap -p WorkUnit.class
Compiled from "WorkUnit.java"
public class com.java7developer.chapter5.WorkUnit<T> {
  private final T workUnit;
  public T getWork();
  public com.java7developer.chapter5.WorkUnit(T);
}


```
默认情况下，javap会显示访问权限为public、protected和默认级别的方法。` 加上-p选项后还可以显示private方法和域 `
###  方法签名的内部形式 
方法签名的内部形式和javap显示出来供人阅读的形式不太一样。
内部采用紧凑型是。类型名是经过压缩的。
字节码：
Class文件是8位字节流，按字节对齐。之所以称为字节码，是因为每条指令都只占据一个字节，所有的操作码和操作数都是按字节对齐的。如：0×03表示iconst_0
Class文件的头4个字节称为魔数（Magic Number），它的唯一作用是用于确认该文件是否是能被JVM接受的Class文件。魔数值为：0xCAFEBABE。
紧接着魔数的4个字节是Class文件的版本号：第5和第6字节是次版本号（Minor Version），第7和第8字节是主版本号（Major Version）。Java的版本号从45开始的，JDK6的版本号是50。
**javap –v MyClass.class，查看字节码完整内容
javap -s MyClass.class相比不带选项参数时可以输出签名的类型描述符，比较常用
**
**全限定名：把类全名中的“.”替换成“/”最后加入一个“;”表示结束。如com/test/TestClass;**
**类型描述符：基本类型及void用大写字符表示，对象类型用字符L加对象的全限定名表示。**
标识字符	含义
  * B	基本类型byte
  * C	char
  * D	double
  * F	float
  * I	int
  * J	long
  * S	short
  * Z	boolean
  * V	void
  * L	对象类型，如Ljava/lang/Object;
  * [	数组，如[Ljava/lang/String;代表String[]
对于数组类型，每一维度将使用一个前置的“[”字符来描述，如定义一个“java.lang.String[][]”类型的二维数组，将被记录为：“[ [Ljava/lang/String;”，一个整型数组“int[]”将被记录为“[I”.

**用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，**
参数列表按照参数的严格顺序放在一组小括号“( )”之内，如方法void inc()的描述符为“( )V”,方法java.lang.String toString() 的描述符合为“( )Ljava/lang/String;”，方法int indexOf(char[] source, int sourceOffset,int sourceCount, char[] target, int targetOffset, int targetCount, int fromIndex)的描述符为“([CII[CIII)I”
**类构造器“<clinit>”方法，实例构造器“<init>”**

###  操作码介绍 
字节码 助记符 指令含义   
0x00 nop 什么都不做。   
0x01 aconst_null 将null推送至栈顶。   
0x02 iconst_m1 将int型-1推送至栈顶。   
0x03 iconst_0 将int型0推送至栈顶。   
0x04 iconst_1 将int型1推送至栈顶。   
0x05 iconst_2 将int型2推送至栈顶。   
0x06 iconst_3 将int型3推送至栈顶。   
0x07 iconst_4 将int型4推送至栈顶。   
0x08 iconst_5 将int型5推送至栈顶。   
0x09 lconst_0 将long型0推送至栈顶。   
0x0a lconst_1 将long型1推送至栈顶。   
0x0b fconst_0 将float型0推送至栈顶。   
0x0c fconst_1 将float型1推送至栈顶。   
0x0d fconst_2 将float型2推送至栈顶。   
0x0e dconst_0 将double型0推送至栈顶。   
0x0f dconst_1 将double型1推送至栈顶。   
0x10 bipush 将单字节的常量值（-128~127）推送至栈顶。   
0x11 sipush 将一个短整型常量值（-32768~32767）推送至栈顶。   
0x12 ldc 将int，float或String型常量值从常量池中推送至栈顶。   
0x13 ldc_w 将int，float或String型常量值从常量池中推送至栈顶（宽索引）。  
0x14 ldc2_w 将long或double型常量值从常量池中推送至栈顶（宽索引）。   
0x15 iload 将指定的int型局部变量推送至栈顶。   
0x16 lload 将指定的long型局部变量推送至栈顶。   
0x17 fload 将指定的float型局部变量推送至栈顶。   
0x18 dload 将指定的double型局部变量推送至栈顶。   
0x19 aload 将指定的引用类型局部变量推送至栈顶。   
0x1a iload_0 将第一个int型局部变量推送至栈顶。   
0x1b iload_1 将第二个int型局部变量推送至栈顶。   
0x1c iload_2 将第三个int型局部变量推送至栈顶。   
0x1d iload_3 将第四个int型局部变量推送至栈顶。   
0x1e lload_0 将第一个long型局部变量推送至栈顶。   
0x1f lload_1 将第二个long型局部变量推送至栈顶。   
0x20 lload_2 将第三个long型局部变量推送至栈顶。   
0x21 lload_3 将第四个long型局部变量推送至栈顶。   
0x22 fload_0 将第一个float型局部变量推送至栈顶。   
0x23 fload_1 将第二个float型局部变量推送至栈顶。   
0x24 fload_2 将第三个float型局部变量推送至栈顶   
0x25 fload_3 将第四个float型局部变量推送至栈顶。   
0x26 dload_0 将第一个double型局部变量推送至栈顶。   
0x27 dload_1 将第二个double型局部变量推送至栈顶。   
0x28 dload_2 将第三个double型局部变量推送至栈顶。   
0x29 dload_3 将第四个double型局部变量推送至栈顶。   
0x2a aload_0 将第一个引用类型局部变量推送至栈顶。   
0x2b aload_1 将第二个引用类型局部变量推送至栈顶。   
0x2c aload_2 将第三个引用类型局部变量推送至栈顶。   
0x2d aload_3 将第四个引用类型局部变量推送至栈顶。   
0x2e iaload 将int型数组指定索引的值推送至栈顶。   
0x2f laload 将long型数组指定索引的值推送至栈顶。  
0x30 faload 将float型数组指定索引的值推送至栈顶。   
0x31 daload 将double型数组指定索引的值推送至栈顶。   
0x32 aaload 将引用型数组指定索引的值推送至栈顶。   
0x33 baload 将boolean或byte型数组指定索引的值推送至栈顶。   
0x34 caload 将char型数组指定索引的值推送至栈顶。   
0x35 saload 将short型数组指定索引的值推送至栈顶。   
0x36 istore 将栈顶int型数值存入指定局部变量。   
0x37 lstore 将栈顶long型数值存入指定局部变量。   
0x38 fstore 将栈顶float型数值存入指定局部变量。   
0x39 dstore 将栈顶double型数值存入指定局部变量。   
0x3a astore 将栈顶引用型数值存入指定局部变量。   
0x3b istore_0 将栈顶int型数值存入第一个局部变量。   
0x3c istore_1 将栈顶int型数值存入第二个局部变量。   
0x3d istore_2 将栈顶int型数值存入第三个局部变量。   
0x3e istore_3 将栈顶int型数值存入第四个局部变量。   
0x3f lstore_0 将栈顶long型数值存入第一个局部变量。   
0x40 lstore_1 将栈顶long型数值存入第二个局部变量。   
0x41 lstore_2 将栈顶long型数值存入第三个局部变量。   
0x42 lstore_3 将栈顶long型数值存入第四个局部变量。   
0x43 fstore_0 将栈顶float型数值存入第一个局部变量。   
0x44 fstore_1 将栈顶float型数值存入第二个局部变量。   
0x45 fstore_2 将栈顶float型数值存入第三个局部变量。   
0x46 fstore_3 将栈顶float型数值存入第四个局部变量。   
0x47 dstore_0 将栈顶double型数值存入第一个局部变量。   
0x48 dstore_1 将栈顶double型数值存入第二个局部变量。   
0x49 dstore_2 将栈顶double型数值存入第三个局部变量。   
0x4a dstore_3 将栈顶double型数值存入第四个局部变量。   
0x4b astore_0 将栈顶引用型数值存入第一个局部变量。  
0x4c astore_1 将栈顶引用型数值存入第二个局部变量。   
0x4d astore_2 将栈顶引用型数值存入第三个局部变量   
0x4e astore_3 将栈顶引用型数值存入第四个局部变量。   
0x4f iastore 将栈顶int型数值存入指定数组的指定索引位置   
0x50 lastore 将栈顶long型数值存入指定数组的指定索引位置。   
0x51 fastore 将栈顶float型数值存入指定数组的指定索引位置。   
0x52 dastore 将栈顶double型数值存入指定数组的指定索引位置。   
0x53 aastore 将栈顶引用型数值存入指定数组的指定索引位置。   
0x54 bastore 将栈顶boolean或byte型数值存入指定数组的指定索引位置。   
0x55 castore 将栈顶char型数值存入指定数组的指定索引位置   
0x56 sastore 将栈顶short型数值存入指定数组的指定索引位置。   
0x57 pop 将栈顶数值弹出（数值不能是long或double类型的）。   
0x58 pop2 将栈顶的一个（long或double类型的）或两个数值弹出（其它）。   
0x59 dup 复制栈顶数值并将复制值压入栈顶。   
0x5a dup_x1 复制栈顶数值并将两个复制值压入栈顶。   
0x5b dup_x2 复制栈顶数值并将三个（或两个）复制值压入栈顶。   
0x5c dup2 复制栈顶一个（long或double类型的)或两个（其它）数值并将复制值压入栈顶。   
0x5d dup2_x1 dup_x1指令的双倍版本。   
0x5e dup2_x2 dup_x2指令的双倍版本。   
0x5f swap 将栈最顶端的两个数值互换（数值不能是long或double类型的）。   
0x60 iadd 将栈顶两int型数值相加并将结果压入栈顶。   
0x61 ladd 将栈顶两long型数值相加并将结果压入栈顶。   
0x62 fadd 将栈顶两float型数值相加并将结果压入栈顶。   
0x63 dadd 将栈顶两double型数值相加并将结果压入栈顶。   
0x64 isub 将栈顶两int型数值相减并将结果压入栈顶。  
0x65 lsub 将栈顶两long型数值相减并将结果压入栈顶。   
0x66 fsub 将栈顶两float型数值相减并将结果压入栈顶。   
0x67 dsub 将栈顶两double型数值相减并将结果压入栈顶。   
0x68 imul 将栈顶两int型数值相乘并将结果压入栈顶。。   
0x69 lmul 将栈顶两long型数值相乘并将结果压入栈顶。   
0x6a fmul 将栈顶两float型数值相乘并将结果压入栈顶。   
0x6b dmul 将栈顶两double型数值相乘并将结果压入栈顶。   
0x6c idiv 将栈顶两int型数值相除并将结果压入栈顶。   
0x6d ldiv 将栈顶两long型数值相除并将结果压入栈顶。   
0x6e fdiv 将栈顶两float型数值相除并将结果压入栈顶。   
0x6f ddiv 将栈顶两double型数值相除并将结果压入栈顶。   
0x70 irem 将栈顶两int型数值作取模运算并将结果压入栈顶。   
0x71 lrem 将栈顶两long型数值作取模运算并将结果压入栈顶。   
0x72 frem 将栈顶两float型数值作取模运算并将结果压入栈顶。   
0x73 drem 将栈顶两double型数值作取模运算并将结果压入栈顶。   
0x74 ineg 将栈顶int型数值取负并将结果压入栈顶。   
0x75 lneg 将栈顶long型数值取负并将结果压入栈顶。   
0x76 fneg 将栈顶float型数值取负并将结果压入栈顶。   
0x77 dneg 将栈顶double型数值取负并将结果压入栈顶。   
0x78 ishl 将int型数值左移位指定位数并将结果压入栈顶。   
0x79 lshl 将long型数值左移位指定位数并将结果压入栈顶。   
0x7a ishr 将int型数值右（有符号）移位指定位数并将结果压入栈顶。   
0x7b lshr 将long型数值右（有符号）移位指定位数并将结果压入栈顶。   
0x7c iushr 将int型数值右（无符号）移位指定位数并将结果压入栈顶。   
0x7d lushr 将long型数值右（无符号）移位指定位数并将结果压入栈顶。   
0x7e iand 将栈顶两int型数值作“按位与”并将结果压入栈顶。   
0x7f land 将栈顶两long型数值作“按位与”并将结果压入栈顶。   
0x80 ior 将栈顶两int型数值作“按位或”并将结果压入栈顶。  
0x81 lor 将栈顶两long型数值作“按位或”并将结果压入栈顶。   
0x82 ixor 将栈顶两int型数值作“按位异或”并将结果压入栈顶。   
0x83 lxor 将栈顶两long型数值作“按位异或”并将结果压入栈顶。   
0x84 iinc 将指定int型变量增加指定值。   
0x85 i2l 将栈顶int型数值强制转换成long型数值并将结果压入栈顶。   
0x86 i2f 将栈顶int型数值强制转换成float型数值并将结果压入栈顶。   
0x87 i2d 将栈顶int型数值强制转换成double型数值并将结果压入栈顶。   
0x88 l2i 将栈顶long型数值强制转换成int型数值并将结果压入栈顶。   
0x89 l2f 将栈顶long型数值强制转换成float型数值并将结果压入栈顶。   
0x8a l2d 将栈顶long型数值强制转换成double型数值并将结果压入栈顶。   
0x8b f2i 将栈顶float型数值强制转换成int型数值并将结果压入栈顶。   
0x8c f2l 将栈顶float型数值强制转换成long型数值并将结果压入栈顶。   
0x8d f2d 将栈顶float型数值强制转换成double型数值并将结果压入栈顶。   
0x8e d2i 将栈顶double型数值强制转换成int型数值并将结果压入栈顶。   
0x8f d2l 将栈顶double型数值强制转换成long型数值并将结果压入栈顶。   
0x90 d2f 将栈顶double型数值强制转换成float型数值并将结果压入栈顶。   
0x91 i2b 将栈顶int型数值强制转换成byte型数值并将结果压入栈顶。   
0x92 i2c 将栈顶int型数值强制转换成char型数值并将结果压入栈顶。   
0x93 i2s 将栈顶int型数值强制转换成short型数值并将结果压入栈顶。   
0x94 lcmp 比较栈顶两long型数值大小，并将结果（1，0，-1）压入栈顶。  
0x95 fcmpl 比较栈顶两float型数值大小，并将结果（1，0，-1）压入栈顶；当其中一个数值为“NaN”时，将-1压入栈顶。   
0x96 fcmpg 比较栈顶两float型数值大小，并将结果（1，0，-1）压入栈顶；当其中一个数值为“NaN”时，将1压入栈顶。   
0x97 dcmpl 比较栈顶两double型数值大小，并将结果（1，0，-1）压入栈顶；当其中一个数值为“NaN”时，将-1压入栈顶。   
0x98 dcmpg 比较栈顶两double型数值大小，并将结果（1，0，-1）压入栈顶；当其中一个数值为“NaN”时，将1压入栈顶。   
0x99 ifeq 当栈顶int型数值等于0时跳转。   
0x9a ifne 当栈顶int型数值不等于0时跳转。   
0x9b iflt 当栈顶int型数值小于0时跳转。   
0x9c ifge 当栈顶int型数值大于等于0时跳转。   
0x9d ifgt 当栈顶int型数值大于0时跳转。   
0x9e ifle 当栈顶int型数值小于等于0时跳转。   
0x9f if_icmpeq 比较栈顶两int型数值大小，当结果等于0时跳转。   
0xa0 if_icmpne 比较栈顶两int型数值大小，当结果不等于0时跳转。   
0xa1 if_icmplt 比较栈顶两int型数值大小，当结果小于0时跳转。   
0xa2 if_icmpge 比较栈顶两int型数值大小，当结果大于等于0时跳转。   
0xa3 if_icmpgt 比较栈顶两int型数值大小，当结果大于0时跳转   
0xa4 if_icmple 比较栈顶两int型数值大小，当结果小于等于0时跳转。   
0xa5 if_acmpeq 比较栈顶两引用型数值，当结果相等时跳转。   
0xa6 if_acmpne 比较栈顶两引用型数值，当结果不相等时跳转。   
0xa7 goto 无条件跳转。   
0xa8 jsr 跳转至指定16位offset位置，并将jsr下一条指令地址压入栈顶。   
0xa9 ret 返回至局部变量指定的index的指令位置（一般与jsr，jsr_w联合使用）。   
0xaa tableswitch 用于switch条件跳转，case值连续（可变长度指令）。  
0xab lookupswitch 用于switch条件跳转，case值不连续（可变长度指令）。   
0xac ireturn 从当前方法返回int。   
0xad lreturn 从当前方法返回long。   
0xae freturn 从当前方法返回float。   
0xaf dreturn 从当前方法返回double。   
0xb0 areturn 从当前方法返回对象引用。   
0xb1 return 从当前方法返回void。   
0xb2 getstatic 获取指定类的静态域，并将其值压入栈顶。   
0xb3 putstatic 为指定的类的静态域赋值。   
0xb4 getfield 获取指定类的实例域，并将其值压入栈顶。   
0xb5 putfield 为指定的类的实例域赋值。   
0xb6 invokevirtual 调用实例方法。   
0xb7 invokespecial 调用超类构造方法，实例初始化方法，私有方法。   
0xb8 invokestatic 调用静态方法。   
0xb9 invokeinterface 调用接口方法。   
0xba invokedynamic 调用动态链接方法①。   
0xbb new 创建一个对象，并将其引用值压入栈顶。   
0xbc newarray 创建一个指定原始类型（如int、float、char??）的数组，并将其引用值压入栈顶。   
0xbd anewarray 创建一个引用型（如类，接口，数组）的数组，并将其引用值压入栈顶。   
0xbe arraylength 获得数组的长度值并压入栈顶。   
0xbf athrow 将栈顶的异常抛出。   
0xc0 checkcast 检验类型转换，检验未通过将抛出ClassCastException。   
0xc1 instanceof 检验对象是否是指定的类的实例，如果是将1压入栈顶，否则将0压入栈顶。  
0xc2 monitorenter 获得对象的monitor，用于同步方法或同步块。   
0xc3 monitorexit 释放对象的monitor，用于同步方法或同步块。   
0xc4 wide 扩展访问局部变量表的索引宽度。   
0xc5 multianewarray 创建指定类型和指定维度的多维数组（执行该指令时，操作栈中必须包含各维度的长度值），并将其引用值压入栈顶。   
0xc6 ifnull 为null时跳转。   
0xc7 ifnonnull 不为null时跳转。   
0xc8 goto_w 无条件跳转（宽索引）。   
0xc9 jsr_w 跳转至指定32位地址偏移量位置，并将jsr_w下一条指令地址压入栈顶。  
  
保留指令   
0xca breakpoint 调试时的断点标志。   
0xfe impdep1 用于在特定硬件中使用的语言后门。   
0xff impdep1 用于在特定硬件中使用的语言后门。  


参考：http://www.cnblogs.com/royi123/p/3570003.html
http://www.oseye.net/user/kevin/blog/160
http://budairenqin.iteye.com/blog/1565750