title: java动态代理与cglib 

#  Java动态代理与cglib 
代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，**代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。**代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，**代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。** 
按照代理的创建时期，代理类可以分为两种。 
  * 　静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。 
  * 　动态代理：在程序运行时，运用反射机制动态创建而成。 

为什么使用动态代理？因为动态代理可以对请求进行任何处理。
哪些地方需要动态代理？不允许直接访问某些类；对访问要做特殊处理等。
目前Java开发包中包含了对动态代理的支持，**但是其实现只支持对接口的的实现**。 其实现主要通过java.lang.reflect.` Proxy `类和java.lang.reflect.` InvocationHandler `接口。 Proxy类主要用来获取动态代理对象，InvocationHandler接口用来约束调用者实现。
` JDK的动态代理用起来非常简单，但它有一个限制，就是使用动态代理的对象必须实现一个或多个接口 `。所以可以用JDK的动态代理。如果想代理没有实现接口的继承的类，该怎么办？ CGLIB就是最好的选择（https://github.com/cglib/cglib，使用apache license 2.0）。其他比较有名的还有从JBoss项目衍生出来的Javassist（https://github.com/jboss-javassist/javassist）。

**CGLib** (Code Generation Library) 是一个强大的,高性能,高质量的Code生成类库。它可以**在运行期扩展Java类与实现Java接口**。Hibernate用它来实现PO字节码的动态生成。
CGLib 比 Java 的 java.lang.reflect.Proxy 类更强的在于它不仅可以接管接口类的方法，还可以接管普通类的方法。
CGLib 的底层是Java字节码操作框架 —— **ASM**。大部分功能实际上是asm所提供的，CGlib只是封装了asm，简化了asm的操作，实现了在运行期动态生成新的class。
**CGlib被许多AOP的框架使用，例如Spring AOP**和dynaop，为他们` 提供方法的interception（拦截） `；最流行的OR Mapping工具**hibernate也使用CGLIB来代理单端single-ended**（多对一和一对一）关联（对集合的**延迟抓取**，是采用其他机制实现的）；EasyMock和jMock是通过使用模仿（moke）对象来测试java代码的包，它们都通过使用CGLIB来为那些没有接口的类创建模仿（moke）对象。

CGLIB包的基本代码很少，但学起来有一定的困难，主要由一下部分组成：
　　（1）net.sf.cglib.core：底层字节码处理类，他们大部分与ASM有关系。
　　（2）net.sf.cglib.transform：编译期或运行期类和类文件的转换。
　　（3）net.sf.cglib.proxy ：实现创建代理和方法拦截器的类。
　　（4）net.sf.cglib.reflect ：实现快速反射和C#风格代理的类。
　　（5）net.sf.cglib.util：集合排序工具类。
　　（6）net.sf.cglib.beans：JavaBean相关的工具类。
CGLIB包是在ASM之上的一个高级别的层。对代理那些没有实现接口的类非常有用。本质上，它是通过动态的生成一个子类去覆盖所要代理类的不是final的方法，并设置好callback，则原有类的每个方法调用就会转变成调用用户定义的拦截方法（interceptors），**这比JDK动态代理方法快多了**。可见，Cglib的原理是对指定的目标类动态生成一个子类，并覆盖其中方法实现增强，**但因为采用的是继承，所以不能对final修饰的类和final方法进行代理**。
GitHub: https://github.com/cglib/cglib
```

<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.0</version>
</dependency>

```

##  class文件简介及加载 
Java编译器编译好Java文件之后，产生.class 文件在磁盘中。这种class文件是二进制文件，内容是只有JVM虚拟机能够识别的机器码。JVM虚拟机读取字节码文件，取出二进制数据，加载到内存中，解析.class 文件内的信息，生成对应的 Class对象:
![](/data/dokuwiki/java/pasted/20151212-202732.png)
关于字节码是如何生成的请参考：http://blog.csdn.net/zhangjg_blog/article/details/21486985
下面通过一段代码演示手动加载 class文件字节码到系统内，转换成class对象，然后再实例化的过程：
```

/** 
 * 程序猿类 
 * @author louluan 
 */  
public class Programmer {  
  
    public void code()  
    {  
        System.out.println("I'm a Programmer,Just Coding.....");  
    }  
}  

```
```

/** 
 * 自定义一个类加载器，用于将字节码转换为class对象 
 * @author louluan 
 */  
public class MyClassLoader extends ClassLoader {  
  
    public Class<?> defineMyClass( byte[] b, int off, int len)   
    {  
        return super.defineClass(b, off, len);  
    }  
      
}  

```
 然后编译成Programmer.class文件，在程序中读取字节码，然后转换成相应的class对象，再实例化：
```

public class MyTest {  
  
    public static void main(String[] args) throws IOException {  
        //读取本地的class文件内的字节码，转换成字节码数组  
        File file = new File(".");  
        InputStream  input = new FileInputStream(file.getCanonicalPath()+"\\bin\\samples\\Programmer.class");  
        byte[] result = new byte[1024];  
          
        int count = input.read(result);  
        // 使用自定义的类加载器将 byte字节码数组转换为对应的class对象  
        MyClassLoader loader = new MyClassLoader();  
        Class clazz = loader.defineMyClass( result, 0, count);  
        //测试加载是否成功，打印class 对象的名称  
        System.out.println(clazz.getCanonicalName());  
                  
               //实例化一个Programmer对象  
               Object o= clazz.newInstance();  
               try {  
                   //调用Programmer的code方法  
                    clazz.getMethod("code", null).invoke(o, null);  
                   } catch (IllegalArgumentException | InvocationTargetException  
                        | NoSuchMethodException | SecurityException e) {  
                     e.printStackTrace();  
                  }  
 }  
} 

```
以上代码演示了，通过字节码加载成class 对象的能力，下面看一下在代码中如何生成class文件的字节码。

##  在运行期的代码中生成二进制字节码 
由于JVM通过字节码的二进制信息加载类的，那么，如果我们在运行期系统中，遵循Java编译系统组织.class文件的格式和结构，**生成相应的二进制数据**，然后再把这个二进制数据加载转换成对应的类，这样，就完成了在代码中，**动态创建一个类的能力**了。
![](/data/dokuwiki/java/pasted/20151212-203144.png)
在运行时期可以按照Java虚拟机规范对class文件的组织规则**生成对应的二进制字节码**。当前有很多开源框架可以完成这些功能，如` ASM，Javassist。 `
##  Java字节码生成开源框架介绍--ASM： 
**ASM 是一个 Java 字节码操控框架。**它能够**以二进制形式**修改已有类或者动态生成类。**ASM 可以直接产生二进制 class 文件，也可以在类被加载入 Java 虚拟机之前动态改变类行为。**ASM 从类文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。
不过ASM在创建class字节码的过程中，**操纵的级别是底层JVM的汇编指令级别**，这要求ASM使用者要对class组织结构和JVM汇编指令有一定的了解。
下面通过ASM 生成下面类Programmer的class字节码：
```

public class Programmer {  
  
    public void code()  
    {  
        System.out.println("I'm a Programmer,Just Coding.....");  
    }  
}  

```
使用ASM框架提供了` ClassWriter ` 接口，通过访问者模式进行动态创建class字节码，看下面的例子：
```

import org.objectweb.asm.ClassWriter;  
import org.objectweb.asm.MethodVisitor;  
import org.objectweb.asm.Opcodes;  
public class MyGenerator {  
  
    public static void main(String[] args) throws IOException {  
  
        System.out.println();  
        ClassWriter classWriter = new ClassWriter(0);  
        // 通过visit方法确定类的头部信息  
        classWriter.visit(Opcodes.V1_7,// java版本  
                Opcodes.ACC_PUBLIC,// 类修饰符  
                "Programmer", // 类的全限定名  
                null, "java/lang/Object", null);  
          
        //创建构造函数  
        MethodVisitor mv = classWriter.visitMethod(Opcodes.ACC_PUBLIC, "<init>", "()V", null, null);  
        mv.visitCode();  
        mv.visitVarInsn(Opcodes.ALOAD, 0);  
        mv.visitMethodInsn(Opcodes.INVOKESPECIAL, "java/lang/Object", "<init>","()V");  
        mv.visitInsn(Opcodes.RETURN);  
        mv.visitMaxs(1, 1);  
        mv.visitEnd();  
          
        // 定义code方法  
        MethodVisitor methodVisitor = classWriter.visitMethod(Opcodes.ACC_PUBLIC, "code", "()V",  
                null, null);  
        methodVisitor.visitCode();  
        methodVisitor.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out",  
                "Ljava/io/PrintStream;");  
        methodVisitor.visitLdcInsn("I'm a Programmer,Just Coding.....");  
        methodVisitor.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/io/PrintStream", "println",  
                "(Ljava/lang/String;)V");  
        methodVisitor.visitInsn(Opcodes.RETURN);  
        methodVisitor.visitMaxs(2, 2);  
        methodVisitor.visitEnd();  
        classWriter.visitEnd();   
        // 使classWriter类已经完成  
        // 将classWriter转换成字节数组写到文件里面去  
        byte[] data = classWriter.toByteArray();  
        File file = new File("D://Programmer.class");  
        FileOutputStream fout = new FileOutputStream(file);  
        fout.write(data);  
        fout.close();  
    }  
}  

```
##  Java字节码生成开源框架介绍--Javassist： 
**Javassist是一个开源的分析、编辑和创建Java字节码的类库。**是由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶 滋）所创建的。它已加入了开放源代码JBoss 应用服务器项目,通过使用Javassist对字节码操作为JBoss实现动态AOP框架。javassist是jboss的一个子项目，**其主要的优点，在于简单，而且快速。直接使用java编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构，或者动态生成类。**
下面通过Javassist创建上述的Programmer类：
```

import javassist.ClassPool;  
import javassist.CtClass;  
import javassist.CtMethod;  
import javassist.CtNewMethod;  
  
public class MyGenerator {  
  
    public static void main(String[] args) throws Exception {  
        ClassPool pool = ClassPool.getDefault();  
        //创建Programmer类       
        CtClass cc= pool.makeClass("com.samples.Programmer");  
        //定义code方法  
        CtMethod method = CtNewMethod.make("public void code(){}", cc);  
        //插入方法代码  
        method.insertBefore("System.out.println(\"I'm a Programmer,Just Coding.....\");");  
        cc.addMethod(method);  
        //保存生成的字节码  
        cc.writeFile("d://temp");  
    }  
}  

```
通过JD-gui反编译工具打开Programmer.class 可以看到以下代码：
![](/data/dokuwiki/java/pasted/20151212-203637.png)
##  代理的基本构成： 
代理模式上，基本上有Subject角色，RealSubject角色，Proxy角色。其中：**Subject角色负责定义RealSubject和Proxy角色应该实现的接口；RealSubject角色用来真正完成业务服务功能；Proxy角色负责将自身的Request请求，调用realsubject 对应的request功能来实现业务功能，自己不真正做业务。**
![](/data/dokuwiki/java/pasted/20151212-203731.png)
上面的这幅代理结构图是典型的静态的代理模式：
当在代码阶段规定这种代理关系，**Proxy类通过编译器编译成class文件，当系统运行时，此class已经存在了。**这种` 静态的代理模式 `固然在访问无法访问的资源，增强现有的接口业务功能方面有很大的优点，**但是大量使用这种静态代理，会使我们系统内的类的规模增大，并且不易维护；**并且由于Proxy和RealSubject的功能 本质上是相同的，Proxy只是起到了中介的作用，这种代理在系统中的存在，**导致系统结构比较臃肿和松散。**
为了解决这个问题，就有了` 动态地创建Proxy `的想法：在运行状态中，需要代理的地方，**根据Subject 和RealSubject，动态地创建一个Proxy，用完之后，就会销毁，这样就可以避免了Proxy 角色的class在系统中冗杂的问题了**。
下面以一个代理模式实例阐述这一问题：
将车站的售票服务抽象出一个接口TicketService,包含问询，卖票，退票功能，车站类Station实现了TicketService接口，车票代售点StationProxy则实现了代理角色的功能，类图如下所示。
![](/data/dokuwiki/java/pasted/20151212-203955.png)
对应的静态的代理模式代码如下所示：
```

/** 
 * 静态代理
 * @author louluan 
 * 
 */  
public class StationProxy implements TicketService {  
	......//省略
}

```
由于我们现在不希望静态地有StationProxy类存在，**希望在代码中，动态生成器二进制代码，加载进来。为此，使用Javassist开源框架，在代码中动态地生成StationProxy的字节码：**
```

import javassist.*;  
public class Test {  
  
    public static void main(String[] args) throws Exception {  
       createProxy();  
    }  
      
    /* 
     * 手动创建字节码 
     */  
    private static void createProxy() throws Exception  
    {  
        ClassPool pool = ClassPool.getDefault();  
  
        CtClass cc = pool.makeClass("com.foo.proxy.StationProxy");  
          
        //设置接口  
        CtClass interface1 = pool.get("com.foo.proxy.TicketService");  
        cc.setInterfaces(new CtClass[]{interface1});  
          
        //设置Field  
        CtField field = CtField.make("private com.foo.proxy.Station station;", cc);  
          
        cc.addField(field);  
          
        CtClass stationClass = pool.get("com.foo.proxy.Station");  
        CtClass[] arrays = new CtClass[]{stationClass};  
        CtConstructor ctc = CtNewConstructor.make(arrays,null,CtNewConstructor.PASS_NONE,null,null, cc);  
        //设置构造函数内部信息  
        ctc.setBody("{this.station=$1;}");  
        cc.addConstructor(ctc);  
  
        //创建收取手续 takeHandlingFee方法  
        CtMethod takeHandlingFee = CtMethod.make("private void takeHandlingFee() {}", cc);  
        takeHandlingFee.setBody("System.out.println(\"收取手续费，打印发票。。。。。\");");  
        cc.addMethod(takeHandlingFee);  
          
        //创建showAlertInfo 方法  
        CtMethod showInfo = CtMethod.make("private void showAlertInfo(String info) {}", cc);  
        showInfo.setBody("System.out.println($1);");  
        cc.addMethod(showInfo);  
          
        //sellTicket  
        CtMethod sellTicket = CtMethod.make("public void sellTicket(){}", cc);  
        sellTicket.setBody("{this.showAlertInfo(\"××××您正在使用车票代售点进行购票，每张票将会收取5元手续费！××××\");"  
                + "station.sellTicket();"  
                + "this.takeHandlingFee();"  
                + "this.showAlertInfo(\"××××欢迎您的光临，再见！××××\");}");  
        cc.addMethod(sellTicket);  
          
        //添加inquire方法  
        CtMethod inquire = CtMethod.make("public void inquire() {}", cc);  
        inquire.setBody("{this.showAlertInfo(\"××××欢迎光临本代售点，问询服务不会收取任何费用，本问询信息仅供参考，具体信息以车站真实数据为准！××××\");"  
        + "station.inquire();"  
        + "this.showAlertInfo(\"××××欢迎您的光临，再见！××××\");}"  
        );  
        cc.addMethod(inquire);  
          
        //添加widthraw方法  
        CtMethod withdraw = CtMethod.make("public void withdraw() {}", cc);  
        withdraw.setBody("{this.showAlertInfo(\"××××欢迎光临本代售点，退票除了扣除票额的20%外，本代理处额外加收2元手续费！××××\");"  
                + "station.withdraw();"  
                + "this.takeHandlingFee();}"  
                );  
        cc.addMethod(withdraw);  
          
        //获取动态生成的class  
        Class c = cc.toClass();  
        //获取构造器  
        Constructor constructor= c.getConstructor(Station.class);  
        //通过构造器实例化  
        TicketService o = (TicketService)constructor.newInstance(new Station());  
        o.inquire();  
          
        cc.writeFile("D://test");  
    }  
      
}

```
通过上面动态生成的代码，我们发现，其实现相当地麻烦在创造的过程中，含有太多的业务代码。我们使用上述创建Proxy代理类的方式的初衷是减少系统代码的冗杂度，` 但是上述做法却增加了在动态创建代理类过程中的复杂度：手动地创建了太多的业务代码，并且封装性也不够，完全不具有可拓展性和通用性。如果某个代理类的一些业务逻辑非常复杂，上述的动态创建代理的方式是非常不可取的！ `
##  InvocationHandler角色的由来 
仔细思考代理模式中的代理Proxy角色。Proxy角色在执行代理业务的时候，无非是在调用真正业务之前或者之后做一些“额外”业务。
![](/data/dokuwiki/java/pasted/20151212-204258.png)
 有上图可以看出，代理类处理的逻辑很简单：在调用某个方法前及方法后做一些额外的业务。换一种思路就是：**在触发（invoke）真实角色的方法之前或者之后做一些额外的业务。**那么，为了构造出具有通用性和简单性的代理类，可以**将所有的触发真实角色动作交给一个触发的管理器**，让这个管理器统一地管理触发。这种管理器就是` Invocation Handler `。
动态代理模式的结构跟上面的静态代理模式稍微有所不同，多引入了一个InvocationHandler角色。
**先解释一下InvocationHandler的作用：**
在静态代理中，代理Proxy中的方法，都指定了调用了特定的realSubject中的对应的方法：
在上面的静态代理模式下，Proxy所做的事情，无非是调用在不同的request时，调用触发realSubject对应的方法；更抽象点看，Proxy所作的事情；在Java中 方法（Method）也是作为一个对象来看待了，
动态代理工作的基本模式就是将自己的方法功能的实现交给 InvocationHandler角色，外界对Proxy角色中的每一个方法的调用，Proxy角色都会交给InvocationHandler来处理，**而InvocationHandler则调用具体对象角色的方法**。如下图所示：
![](/data/dokuwiki/java/pasted/20151212-204612.png)
在这种模式之中：` 代理Proxy 和RealSubject 应该实现相同的功能，这一点相当重要 `。
在面向对象的编程之中，如果我们想要约定Proxy 和RealSubject可以实现相同的功能，有两种方式：
a.一个比较直观的方式，就是**定义一个功能接口**，然后让Proxy 和RealSubject来实现这个接口。
b.还有比较隐晦的方式，就是**通过继承**。因为如果Proxy 继承自RealSubject，这样Proxy则拥有了RealSubject的功能，Proxy还可以通过重写RealSubject中的方法，来实现多态。
` 其中JDK中提供的创建动态代理的机制，是以a 这种思路设计的，而cglib 则是以b思路设计的。 `
##  JDK的动态代理创建机制----通过接口 
 比如现在想为RealSubject这个类创建一个动态代理对象，JDK主要会做以下工作：
    1.   获取 RealSubject上的所有接口列表；
    2.   确定要生成的代理类的类名，默认为：com.sun.proxy.$ProxyXXXX ；
    3.   根据需要实现的接口信息，在代码中动态创建 该Proxy类的字节码；
    4 .  将对应的字节码转换为对应的class 对象；
    5.   创建InvocationHandler 实例handler，用来处理Proxy所有方法调用；
    6.   Proxy 的class对象 以创建的handler对象为参数，实例化一个proxy对象
JDK通过 ` java.lang.reflect.Proxy `包来支持动态代理，一般情况下，我们使用下面的` newProxyInstance `方法
static Object	newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。而对于InvocationHandler，我们需要实现下列的invoke方法：

在调用代理对象中的每一个方法时，在代码内部，都是直接调用了InvocationHandler 的invoke方法，而invoke方法根据代理类传递给自己的method参数来区分是什么方法。
 Object	invoke(Object proxy,Method method,Object[] args)
在代理实例上处理方法调用并返回结果。

**JDK动态代理示例**
现在定义两个接口Vehicle和Rechargable，Vehicle表示交通工具类，有drive()方法；Rechargable接口表示可充电的（工具），有recharge() 方法；
 定义一个实现两个接口的类ElectricCar，类图如下：
![](/data/dokuwiki/java/pasted/20151212-205012.png)
通过下面的代码片段，来为ElectricCar创建动态代理类：
```

import java.lang.reflect.InvocationHandler;  
import java.lang.reflect.Proxy;  
  
public class Test {  
  
    public static void main(String[] args) {  
  
        ElectricCar car = new ElectricCar();  
        // 1.获取对应的ClassLoader  
        ClassLoader classLoader = car.getClass().getClassLoader();  
  
        // 2.获取ElectricCar 所实现的所有接口  
        Class[] interfaces = car.getClass().getInterfaces();  
        // 3.设置一个来自代理传过来的方法调用请求处理器，处理所有的代理对象上的方法调用  
        InvocationHandler handler = new InvocationHandlerImpl(car);  
        /* 
          4.根据上面提供的信息，创建代理对象 在这个过程中，  
                         a.JDK会通过根据传入的参数信息动态地在内存中创建和.class 文件等同的字节码 
                 b.然后根据相应的字节码转换成对应的class，  
                         c.然后调用newInstance()创建实例 
         */  
        Object o = Proxy.newProxyInstance(classLoader, interfaces, handler);  
        Vehicle vehicle = (Vehicle) o;  
        vehicle.drive();  
        Rechargable rechargeable = (Rechargable) o;  
        rechargeable.recharge();  
    }  
}  

```
```

import java.lang.reflect.InvocationHandler;  
import java.lang.reflect.Method;  
  
public class InvocationHandlerImpl implements InvocationHandler {  
  
    private ElectricCar car;  
      
    public InvocationHandlerImpl(ElectricCar car)  
    {  
        this.car=car;  
    }  
      
    @Override  
    public Object invoke(Object paramObject, Method paramMethod,  
            Object[] paramArrayOfObject) throws Throwable {  
        System.out.println("You are going to invoke "+paramMethod.getName()+" ...");  
        paramMethod.invoke(car, null);  
        System.out.println(paramMethod.getName()+" invocation Has Been finished...");  
        return null;  
    }  
  
}  

```
 生成动态代理类的字节码并且保存到硬盘中：  
JDK提供了` sun.misc.ProxyGenerator.generateProxyClass(String proxyName,class[] interfaces) ` 底层方法来**产生动态代理类的字节码**：
```

import java.lang.reflect.Proxy;  
import sun.misc.ProxyGenerator;  
  
public class ProxyUtils {  
  
    /* 
     * 将根据类信息 动态生成的二进制字节码保存到硬盘中， 
     * 默认的是clazz目录下 
         * params :clazz 需要生成动态代理类的类 
         * proxyName : 为动态生成的代理类的名称 
         */  
    public static void generateClassFile(Class clazz,String proxyName)  
    {  
        //根据类信息和提供的代理类名称，生成字节码  
                byte[] classFile = ProxyGenerator.generateProxyClass(proxyName, clazz.getInterfaces());   
        String paths = clazz.getResource(".").getPath();  
        System.out.println(paths);  
        FileOutputStream out = null;    
          
        try {  
            //保留到硬盘中  
            out = new FileOutputStream(paths+proxyName+".class");    
            out.write(classFile);    
            out.flush();    
        } catch (Exception e) {    
            e.printStackTrace();    
        } finally {    
            try {    
                out.close();    
            } catch (IOException e) {    
                e.printStackTrace();    
            }    
        }    
    }  
      
}  

```
现在我们想将生成的代理类起名为“ElectricCarProxy”，并保存在硬盘，应该使用以下语句：
```

ProxyUtils.generateClassFile(car.getClass(), "ElectricCarProxy");

```
这样将在ElectricCar.class 同级目录下产生 ElectricCarProxy.class文件。用反编译工具如jd-gui.exe 打开
仔细观察可以看出生成的动态代理类有以下特点:
1.继承自 java.lang.reflect.Proxy，实现了 Rechargable,Vehicle 这两个ElectricCar实现的接口；
2.类中的所有方法都是final 的；
3.所有的方法功能的实现都统一调用了InvocationHandler的invoke()方法。
![](/data/dokuwiki/java/pasted/20151212-205537.png)
##  cglib 生成动态代理类的机制----通过类继承： 
**JDK中提供的生成动态代理类的机制**有个鲜明的**缺点**是： 某个类必须有实现的接口，而生成的代理类也只能代理某个类接口定义的方法，比如：如果上面例子的ElectricCar实现了继承自两个接口的方法外，另外实现了方法bee() ,则在产生的动态代理类中不会有这个方法了！更极端的情况是：如果某个类没有实现接口，那么这个类就不能同JDK产生动态代理了！
` 幸好我们有cglib。“CGLIB（Code Generation Library），是一个强大的，高性能，高质量的Code生成类库，它可以在运行期扩展Java类与实现Java接口。” `
**cglib 创建某个类A的动态代理类的模式是：**
1.   查找A上的所有` 非final 的public类型的方法 `定义；
2.   将这些方法的定义转换成字节码；
3.   将组成的字节码转换成相应的代理的class对象；
4.   实现 ` MethodInterceptor `接口，用来处理 对代理类上所有方法的请求（这个接口和JDK动态代理InvocationHandler的功能和角色是一样的）
一个有趣的例子：定义一个Programmer类，一个Hacker类
```

/** 
 * 程序猿类 
 * @author louluan 
 */  
public class Programmer {  
  
    public void code()  
    {  
        System.out.println("I'm a Programmer,Just Coding.....");  
    }  
}  

```
```

import net.sf.cglib.proxy.MethodInterceptor;  
import net.sf.cglib.proxy.MethodProxy;  
/* 
 * 实现了方法拦截器接口 
 */  
public class Hacker implements MethodInterceptor {  
    @Override  
    public Object intercept(Object obj, Method method, Object[] args,  
            MethodProxy proxy) throws Throwable {  
        System.out.println("**** I am a hacker,Let's see what the poor programmer is doing Now...");  
        proxy.invokeSuper(obj, args);  
        System.out.println("****  Oh,what a poor programmer.....");  
        return null;  
    }  
  
}  

```
```

import net.sf.cglib.proxy.Enhancer;  
  
public class Test {  
  
    public static void main(String[] args) {  
        Programmer progammer = new Programmer();  
          
        Hacker hacker = new Hacker();  
        //cglib 中加强器，用来创建动态代理  
        Enhancer enhancer = new Enhancer();    
                 //设置要创建动态代理的类  
        enhancer.setSuperclass(progammer.getClass());    
               // 设置回调，这里相当于是对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实行intercept()方法进行拦截  
                enhancer.setCallback(hacker);  
                Programmer proxy =(Programmer)enhancer.create();  
                proxy.code();  
          
    }  
}  

```
##  用Cglib创建动态代理详解 
下图表示Cglib常用到的几类。
![](/data/dokuwiki/java/pasted/20151212-210828.png)
创建一个具体类的代理时，通常要用到的CGLIB包的APIs：
　　net.sf.cglib.proxy.Callback接口：在CGLIB包中是一个很关键的接口，所有被net.sf.cglib.proxy.Enhancer类调用的回调（callback）接口都要继承这个接口。
　　net.sf.cglib.proxy.` MethodInterceptor接口 `：是最通用的回调（callback）类型，**它经常被AOP用来实现拦截（intercept）方法的调用****粗体**。这个接口只定义了一个方法。
public Object intercept(Object object, java.lang.reflect.Method method, Object[] args, MethodProxy proxy) throws Throwable;  
　　当net.sf.cglib.proxy.MethodInterceptor做为所有代理方法的回调 （callback）时，当对基于代理的方法调用时，在调用原对象的方法的之前会调用这个方法，原来的方法可能通过使用java.lang.reflect.Method对象的一般反射调用，或者使用 net.sf.cglib.proxy.MethodProxy对象调用。` net.sf.cglib.proxy.MethodProxy通常被首选使用，因为它更快 `。在这个方法中，我们可以在调用原方法之前或之后注入自己的代码
![](/data/dokuwiki/java/pasted/20151212-211019.png)
net.sf.cglib.proxy.MethodInterceptor能够满足任何的拦截（interception ）需要，当对有些情况下可能过度。**为了简化和提高性能，CGLIB包提供了一些专门的回调（callback）类型。**例如：
　　net.sf.cglib.proxy.FixedValue：为提高性能，FixedValue回调对强制某一特别方法返回固定值是有用的。
　　net.sf.cglib.proxy.` NoOp `：NoOp回调把对方法调用**直接委派到这个方法在父类中（也就是被代理的那个类）的实现。**
　　net.sf.cglib.proxy.LazyLoader：当实际的对象需要**延迟装载**时，可以使用LazyLoader回调。一旦实际对象被装载，它将被每一个调用代理对象的方法使用。
　　net.sf.cglib.proxy.Dispatcher：Dispathcer回调和LazyLoader回调有相同的特点，不同的是，当代理方法被调用时，装载对象的方法也总要被调用。
　　 net.sf.cglib.proxy.ProxyRefDispatcher：ProxyRefDispatcher回调和Dispatcher一样，不同的是，它可以把代理对象作为装载对象方法的一个参数传递。
　　代理类的所以方法经常会用到回调（callback），当然你也可以使用` net.sf.cglib.proxy.CallbackFilter ` **有选择的对一些方法使用回调（callback）**，这种考虑周详的控制特性在JDK的动态代理中是没有的。在JDK代理中，对 java.lang.reflect.InvocationHandler方法的调用对代理类的所有方法都有效。
　　CGLIB的代理包也对` net.sf.cglib.proxy.Mixin `提供支持。基本上，**它允许多个对象被绑定到一个单一的大对象**。在代理中对方法的调用委托到下面相应的对象中。
　　接下来我们看看如何使 用CGLIB代理APIs创建代理。
　　1、创建一个简单的代理
　　CGLIB代理最核心类` net.sf.cglib.proxy.Enhancer `， 为了创建一个代理，最起码你要用到这个类。首先，让我们使用NoOp回调创建一个代理。
```

public Object createProxy(Class targetClass) {   
    Enhancer enhancer = new Enhancer();  
    enhancer.setSuperclass(targetClass);  
    enhancer.setCallback(NoOp.INSTANCE);  
    return enhancer.create();  
} 

```
返回值是target类一个实例的代理。在这个例子中，我们为net.sf.cglib.proxy.Enhancer 配置了一个单一的回调（callback）。我们可以看到很少直接创建一个简单的代理，而是创建一个net.sf.cglib.proxy.Enhancer的实例，在net.sf.cglib.proxy.Enhancer类中你可使用静态帮助方法创建一个简单的代理。一般推荐使用上面例子的方法创建代理，因为它允许你通过配置net.sf.cglib.proxy.Enhancer实例很好的控制代理的创建。
　　要注意的是，` target类是作为产生的代理的父类传进来的。 `不同于JDK的动态代理，它不能在创建代理时传target对象，target对象必须被CGLIB包来创建。在这个例子中，` 默认的无参数构造器时用来创建target实例的。如果你想用CGLIB来创建有参数的实例，用net.sf.cglib.proxy.Enhancer.create(Class[], Object[])方法替代net.sf.cglib.proxy.Enhancer.create()就可以了。 `

###  2、使用MethodInterceptor创建一个代理 
　　为了更好的使用代理，我们可以使用**自己定义的MethodInterceptor类型回调（callback）来代替net.sf.cglib.proxy.NoOp回调。**` 当对代理中所有方法的调用时，都会转向MethodInterceptor类型的拦截（intercept）方法，在拦截方法中再调用底层对象相应的方法。 `下面我们举个例子，假设你想对目标对象的所有方法调用进行权限的检查，如果没有经过授权，就抛出一个运行时的异常AuthorizationException。其中AuthorizationService.java接口的代码如下
```

public interface AuthorizationService {   
    void authorize(Method method);   
}  

```
对net.sf.cglib.proxy.MethodInterceptor接口的实现的类AuthorizationInterceptor.java代码如下：
```

import java.lang.reflect.Method;  
import net.sf.cglib.proxy.MethodInterceptor;  
import net.sf.cglib.proxy.MethodProxy;  
  
import com.lizjason.cglibproxy.AuthorizationService;  
  
/** 
 * A simple MethodInterceptor implementation to 
 * apply authorization checks for proxy method calls. 
 */  
public class AuthorizationInterceptor implements MethodInterceptor {  
  
    private AuthorizationService authorizationService;  
  
    /** 
     * Create a AuthorizationInterceptor with the given AuthorizationService 
     */  
    public AuthorizationInterceptor (AuthorizationService authorizationService) {  
        this.authorizationService = authorizationService;  
    }  
  
    /** 
     * Intercept the proxy method invocations to inject authorization check. * The original 
     * method is invoked through MethodProxy. 
     */  
    public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {  
        if (authorizationService != null) {  
            //may throw an AuthorizationException if authorization failed  
            authorizationService.authorize(method);  
        }  
        return methodProxy.invokeSuper(object, args);  
    }  
}  

```
我们可以看到在拦截方法中，首先进行权限的检查，如果通过权限的检查，拦截方法再调用目标对象的原始方法。由于性能的原因，对原始方法的调用我们使用CGLIB的net.sf.cglib.proxy.MethodProxy对象，而不是反射中一般使用java.lang.reflect.Method对象。
下面是一个完整的使用MethodInterceptor的例子。
```

import java.lang.reflect.Method;  
import net.sf.cglib.proxy.Enhancer;  
import net.sf.cglib.proxy.MethodInterceptor;  
import net.sf.cglib.proxy.MethodProxy;  
  
/** 
 * 定义一个HelloWorld类，没有实现接口 
 * 
 */  
class HelloWorld {  
  
    public void sayHelloWorld() {  
        System.out.println("HelloWorld!");  
    }  
}  
  
/** 
 * 通过Cglib实现在方法调用前后向控制台输出两句字符串 
 * 
 */  
class CglibProxy implements MethodInterceptor {  
  
    //要代理的原始对象  
    private Object obj;  
  
    public Object createProxy(Object target) {  
        this.obj = target;  
        Enhancer enhancer = new Enhancer();  
        // 设置要代理的目标类，以扩展它的功能  
        enhancer.setSuperclass(this.obj.getClass());  
        // 设置单一回调对象，在回调中拦截对目标方法的调用  
        enhancer.setCallback(this);  
        //设置类装载器  
        enhancer.setClassLoader(target.getClass().getClassLoader());  
        //创建代理对象  
        return enhancer.create();  
    }  
  
    /** 
     * 回调方法:在代理实例上拦截并处理目标方法的调用，返回结果 
     * 
     * @param proxy 代理类 
     * @param method 被代理的方法 
     * @param params 该方法的参数数组 
     * @param methodProxy 
     */  
    @Override  
    public Object intercept(Object proxy, Method method, Object[] params,  
            MethodProxy methodProxy) throws Throwable {  
        Object result = null;  
        // 调用之前  
        doBefore();  
        // 调用目标方法，用methodProxy,  
        // 而不是原始的method，以提高性能  
        result = methodProxy.invokeSuper(proxy, params);  
        // 调用之后  
        doAfter();  
        return result;  
    }  
  
    private void doBefore() {  
        System.out.println("before method invoke");  
    }  
  
    private void doAfter() {  
        System.out.println("after method invoke");  
    }  
}  
  
public class TestCglib {  
  
    public static void main(String[] args) {  
        CglibProxy cglibProxy = new CglibProxy();  
        HelloWorld hw = (HelloWorld) cglibProxy.createProxy(new HelloWorld());  
        hw.sayHelloWorld();  
    }  
}  

```
基本流程：需要自己写代理类，它实现MethodInterceptor接口，有一个intercept()回调方法用于拦截对目标方法的调用，里面使用methodProxy来调用目标方法。创建代理对象要用Enhance类，用它设置好代理的目标类、有intercept()回调的代理类实例、最后用create()创建并返回代理实例。
###  3、使用CallbackFilter在方法层设置回调 
` net.sf.cglib.proxy.CallbackFilter `允许我们在方法层设置回调（callback）。假如你有一个PersistenceServiceImpl类，它有两个方法：save和load，其中方法save需要权限检查，而方法load不需要权限检查。
```

public class PersistenceServiceCallbackFilter implements CallbackFilter {   
    //callback index for save method  
    private static final int SAVE = 0;  
    //callback index for load method  
    private static final int LOAD = 1;  
  
    /** 
     * Specify which callback to use for the method being invoked.  
     * @param method the method being invoked. 
     * @return  
     */  
    @Override  
    public int accept(Method method) {  
        //指定各方法的代理回调索引  
        String name = method.getName();  
        if ("save".equals(name)) {  
            return SAVE;  
        }  
        // for other methods, including the load method, use the  
        // second callback  
        return LOAD;  
    }  
}  

```
accept方法中对代理方法和回调进行了匹配，返回的值是某方法在回调数组中的索引。下面是PersistenceServiceImpl类代理的实现。
```

Enhancer enhancer = new Enhancer();  
enhancer.setSuperclass(PersistenceServiceImpl.class);  
//设置回调过滤器  
CallbackFilter callbackFilter = new PersistenceServiceCallbackFilter();  
enhancer.setCallbackFilter(callbackFilter);  
//创建各个目标方法的代理回调  
AuthorizationService authorizationService = ...  
Callback saveCallback = new AuthorizationInterceptor(authorizationService);  
Callback loadCallback = NoOp.INSTANCE;  
//顺序要与指定的回调索引一致  
Callback[] callbacks = new Callback[]{saveCallback, loadCallback };  
enhancer.setCallbacks(callbacks);  //设置回调  
...  
return (PersistenceServiceImpl)enhancer.create();  //创建代理对象  

```
在这个例子中save方法使用了AuthorizationInterceptor实例，load方法使用了NoOp实例。此外，你也可以通过net.sf.cglib.proxy.Enhancer.setInterfaces(Class[])方法指定代理对象所实现的接口。
　　除了为net.sf.cglib.proxy.Enhancer指定回调数组，你还可以通过net.sf.cglib.proxy.Enhancer.setCallbackTypes(Class[]) 方法指定回调类型数组。当创建代理时，如果你没有回调实例的数组，就可以使用回调类型。象使用回调一样，` 你必须使用net.sf.cglib.proxy.CallbackFilter为每一个方法指定一个回调类型索引。 `
###  4、使用Mixin 
Mixin通过代理方式将多种类型的对象绑定到一个大对象上，这样对各个目标类型中的方法调用可以直接在这个大对象上进行。下面是一个例子。
```

interface MyInterfaceA {  
  
    public void methodA();  
}  
  
interface MyInterfaceB {  
  
    public void methodB();  
}  
  
class MyInterfaceAImpl implements MyInterfaceA {  
  
    @Override  
    public void methodA() {  
        System.out.println("MyInterfaceAImpl.methodA()");  
    }  
}  
  
class MyInterfaceBImpl implements MyInterfaceB {  
  
    @Override  
    public void methodB() {  
        System.out.println("MyInterfaceBImpl.methodB()");  
    }  
}  
  
public class Main {  
  
    public static void main(String[] args) {  
        //各个对象对应的类型  
        Class[] interfaces = new Class[]{MyInterfaceA.class, MyInterfaceB.class};  
        //各个对象  
        Object[] delegates = new Object[]{new MyInterfaceAImpl(), new MyInterfaceBImpl()};  
        //将多个对象绑定到一个大对象上  
        Object obj = Mixin.create(interfaces, delegates);  
        //直接在大对象上调用各个目标方法  
        ((MyInterfaceA)obj).methodA();  
        ((MyInterfaceB)obj).methodB();  
    }  
}  

```
##  cglib动态生成Bean 
我们知道，Java Bean包含一组属性字段，用这些属性来存储和获取值。通过指定一组属性名和属性值的类型，我们可以` 使用Cglib的BeanGenerator和BeanMap来动态生成Bean。 `
```

import net.sf.cglib.beans.BeanGenerator;  
import net.sf.cglib.beans.BeanMap;  
  
/** 
 * 动态实体bean 
 * 
 * @author cuiran 
 * @version 1.0 
 */  
class CglibBean {  
  
    //Bean实体Object  
    public Object object = null;  
    //属性map  
    public BeanMap beanMap = null;  
  
    public CglibBean() {  
        super();  
    }  
  
    @SuppressWarnings("unchecked")  
    public CglibBean(Map<String, Class> propertyMap) {  
        //用一组属性生成实体Bean  
        this.object = generateBean(propertyMap);  
        //用实体Bean创建BeanMap，以便可以设置和获取Bean属性的值  
        this.beanMap = BeanMap.create(this.object);  
    }  
  
    /** 
     * 给bean中的属性赋值 
     * 
     * @param property 属性名 
     * @param value 值 
     */  
    public void setValue(String property, Object value) {  
        beanMap.put(property, value);  
    }  
  
    /** 
     * 获取bean中属性的值 
     * 
     * @param property 属性名 
     * @return 值 
     */  
    public Object getValue(String property) {  
        return beanMap.get(property);  
    }  
  
    /** 
     * 得到该实体bean对象 
     * 
     * @return 
     */  
    public Object getObject() {  
        return this.object;  
    }  
  
    @SuppressWarnings("unchecked")  
    private Object generateBean(Map<String, Class> propertyMap) {  
        //根据一组属性名和属性值的类型，动态创建Bean对象  
        BeanGenerator generator = new BeanGenerator();  
        Set keySet = propertyMap.keySet();  
        for (Iterator i = keySet.iterator(); i.hasNext();) {  
            String key = (String) i.next();  
            generator.addProperty(key, (Class) propertyMap.get(key));  
        }  
        return generator.create();  //创建Bean  
    }  
}  
  
/** 
 * Cglib测试类 
 * 
 * @author cuiran 
 * @version 1.0 
 */  
public class CglibTest {  
  
    @SuppressWarnings("unchecked")  
    public static void main(String[] args) throws ClassNotFoundException { // 设置类成员属性  
        HashMap<String, Class> propertyMap = new HashMap<>();  
        propertyMap.put("id", Class.forName("java.lang.Integer"));  
        propertyMap.put("name", Class.forName("java.lang.String"));  
        propertyMap.put("address", Class.forName("java.lang.String")); // 生成动态Bean  
        CglibBean bean = new CglibBean(propertyMap);  
        // 给Bean设置值  
        bean.setValue("id", 123);  //Auto-boxing  
        bean.setValue("name", "454");  
        bean.setValue("address", "789");  
        // 从Bean中获取值，当然获得值的类型是Object  
        System.out.println(" >> id = " + bean.getValue("id"));  
        System.out.println(" >> name = " + bean.getValue("name"));  
        System.out.println(" >> address = " + bean.getValue("address"));  
        // 获得bean的实体  
        Object object = bean.getObject();  
        // 通过反射查看所有方法名  
        Class clazz = object.getClass();  
        Method[] methods = clazz.getDeclaredMethods();  
        for (Method curMethod : methods) {  
            System.out.println(curMethod.getName());  
        }  
    }  
}  

```

```

GlobalConfig a=new GlobalConfig();
GlobalConfig b=new GlobalConfig();
a.setId("111");
a.setIsFirst(0);
    BeanMap map1=BeanMap.create(a);
    BeanMap map2=BeanMap.create(b);
    for(Object key:map1.keySet()){
        if(map2.containsKey(key)){
            map2.put(key, map1.get(key));
        }
    }
System.out.println("id:"+b.getId()); //输出111

```
##  BeanCopier高性能属性拷贝 
```

public class BeanCopierUtils {
    	// 使用多线程安全的Map来缓存BeanCopier，由于读操作远大于写，所以性能影响可以忽略
	public static ConcurrentHashMap<String, BeanCopier> beanCopierMap = new ConcurrentHashMap<String, BeanCopier>();

	public static void copyProperties(Object source, Object target) {
		String beanKey = generateKey(source.getClass(), target.getClass());
		BeanCopier copier = null;
		copier = BeanCopier.create(source.getClass(), target.getClass(), false);
		beanCopierMap.putIfAbsent(beanKey, copier);// putIfAbsent已经实现原子操作了。
		copier = beanCopierMap.get(beanKey);
		copier.copy(source, target, null);
	}

	private static String generateKey(Class<?> class1, Class<?> class2) {
		return class1.toString() + class2.toString();
	}
}

```
关键代码如下
```

copier = BeanCopier.create(source.getClass(), target.getClass(), false);
copier.copy(source, target, null);

```
Create对象过程：产生sourceClass→ TargetClass的拷贝代理类，放入jvm中，` 所以创建的代理类的时候比较耗时。最好保证这个对象的单例模式或者缓存该对象 `。
创建过程：源代码见jdk：net.sf.cglib.beans.BeanCopier.Generator.generateClass(ClassVisitor)
1、 获取sourceClass的所有public get 方法-》PropertyDescriptor[] getters
2、 获取TargetClass 的所有 public set 方法-》PropertyDescriptor[] setters
3、 遍历setters的每一个属性，执行4和5
4、 按setters的name生成sourceClass的所有setter方法-》PropertyDescriptor getter【不符合javabean规范的类将会可能出现空指针异常】
5、 PropertyDescriptor[] setters-》PropertyDescriptor setter
6、 将setter和getter名字和类型 配对，生成代理类的拷贝方法。
` Copy属性过程：调用生成的代理类，代理类的代码和手工操作的代码很类似，效率非常高。 `
##  BulkBean 
相比于BeanCopier，BulkBean将整个Copy的动作拆分为getPropertyValues，setPropertyValues的两个方法，允许自定义处理的属性。
```

public class BulkBeanTest {  
  
    public static void main(String args[]) {  
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "/home/ljh/cglib");  
        String[] getter = new String[] { "getValue" };  
        String[] setter = new String[] { "setValue" };  
        Class[] clazzs = new Class[] { int.class };  
  
        BulkBean bean = BulkBean.create(BulkSource.class, getter, setter, clazzs);  
        BulkSource obj = new BulkSource();  
        obj.setValue(1);  
  
        Object[] objs = bean.getPropertyValues(obj);  
        for (Object tmp : objs) {  
            System.out.println(tmp);  
        }  
    }  
}  
class BulkSource {  
    private int value;  
    .....  
}  
  
// 反编译后的代码：　  
 public void getPropertyValues(Object obj, Object aobj[])  
    {  
        BulkSource bulksource = (BulkSource)obj;  
        aobj[0] = new Integer(bulksource.getValue());  
    } 

```
使用注意
避免每次进行BulkBean.create创建对象，一般建议是通过static BulkBean.create copier = BulkBean.create
应用场景：针对特定属性的get,set操作，一般适用通过xml配置注入和注出的属性，运行时才确定处理的Source,Target类，只需关注属性名即可。
##  CGLIB轻松实现延迟加载 
` 通过使用LazyLoader，可以实现延迟加载 `，即在没有访问对象的字段或方法之前并不加载对象，只有当要访问对象的字段或方法时才进行加载。下面是一个例子。
```

import net.sf.cglib.proxy.Enhancer;  
import net.sf.cglib.proxy.LazyLoader;  
  
class TestBean {  
    private String userName;  
  
    /** 
     * @return the userName 
     */  
    public String getUserName() {  
        return userName;  
    }  
  
    /** 
     * @param userName the userName to set 
     */  
    public void setUserName(String userName) {  
        this.userName = userName;  
    }  
}  
  
//延迟加载代理类  
class LazyProxy implements LazyLoader {  
  
    //拦截Bean的加载，本方法会延迟处理  
    @Override  
    public Object loadObject() throws Exception {  
        System.out.println("开始延迟加载!");  
        TestBean bean = new TestBean(); //创建实体Bean  
        bean.setUserName("test");  //给一个属性赋值  
        return bean;  //返回Bean  
    }  
}  
  
public class BeanTest {  
  
    public static void main(String[] args) {  
        //创建Bean类型的延迟加载代理实例  
        TestBean bean = (TestBean) Enhancer.create(TestBean.class, new LazyProxy());  
        System.out.println("------");  
        System.out.println(bean.getUserName());  
    }  
}  

```
我们创建TestBean类的延迟代理，通过LazyLoader中的loadObject()方法的拦截，实现对TestBean类的对象进行延迟加载。从输出可以看出，当创建延迟代理时，并没有立刻加载目标对象（因为还有输出“开始延迟加载!”），当通过代理访问目标对象的getUserName()方法时，就会加载目标对象。可见loadObject()是延迟执行的。

##  接口生成器InterfaceMaker 
一、作用：
InterfaceMaker会动态生成一个接口，该接口包含指定类定义的所有方法。
二、示例：
比较简单，先定义一个类，仍使用本系列第一篇中的那个ConcreteClassNoInterface类，该类包含3个方法：
```

public class ConcreteClassNoInterface {  
    public String getConcreteMethodA(String str){  
        System.out.println("ConcreteMethod A ... "+str);  
        return str;  
    }  
    public int getConcreteMethodB(int n){  
        System.out.println("ConcreteMethod B ... "+n);  
        return n+10;  
    }  
    public int getConcreteMethodFixedValue(int n){  
        System.out.println("getConcreteMethodFixedValue..."+n);  
        return n+10;  
    }  
}

```
用这个类内定义的方法来生成一个接口：
```

InterfaceMaker im=new InterfaceMaker();  
im.add(ConcreteClassNoInterface.class);  
Class interfaceOjb=im.create();  
System.out.println(interfaceOjb.isInterface());//true  
System.out.println(interfaceOjb.getName());//net.sf.cglib.empty.Object$$InterfaceMakerByCGLIB$$13e205f  

```
```

Method[] methods = interfaceOjb.getMethods();  
for(Method method:methods){  
    System.out.println(method.getName());  
} 
输出结果，与ConcreteClassNoInterface类内定义的方法完全相同：
getConcreteMethodA  
getConcreteMethodB  
getConcreteMethodFixedValue  

``` 
```

Object obj = Enhancer.create(Object.class, new Class[]{ interfaceOjb },   
new MethodInterceptor() {  
   public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {  
      return "intercept!";  
   }  
});  
  
Method method = obj.getClass().getMethod("getConcreteMethodA", new Class[]{String.class});  
System.out.println(method.invoke(obj, new Object[]{"12345"})); 

``` 
此处**让Object生成的代理类实现了由InterfaceMaker生成的接口**，但是由于Object类并没有覆写其中的方法，因此，每当对生成接口内方法进行MethodInterceptor方法拦截时，都返回一个字符串，并在最后打印出来。
```

public class CGLibExample {  
  
    @SuppressWarnings("unchecked")  
    public static void main(String[] args) {  
          
        // 定义一个参数是字符串类型的setCreatedAt方法  
        InterfaceMaker im = new InterfaceMaker();  
        im.add(new Signature("setCreatedAt", Type.VOID_TYPE,   
                new Type[] { Type.getType(String.class) }), null);  
  
        Class myInterface = im.create();  
  
        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(ExampleBean.class);  
        enhancer.setInterfaces(new Class[] { myInterface });  
        enhancer.setCallback(new MethodInterceptor() {  
            public Object intercept(Object obj, Method method, Object[] args,  
                    MethodProxy proxy) throws Throwable {  
                  
                ExampleBean bean = (ExampleBean) obj;  
                  
                // 调用字符串类型的setCreatedAt方法时，转换成Date型后调用Setter  
                if (method.getName().startsWith("setCreatedAt")  
                        && args[0] != null && args[0] instanceof String) {  
  
                    SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");  
                    Date date = null;  
                    try {  
                        date = sdf.parse((String) args[0]);  
                    } catch (final Exception e) { /* nop */ }  
                    bean.setCreatedAt(date);  
                    return null;  
  
                }  
                return proxy.invokeSuper(obj, args);  
            }  
        });  
  
        // 生成一个Bean  
        ExampleBean bean = (ExampleBean) enhancer.create();  
        bean.setId(999);  
  
        try {  
            Method method = bean.getClass().getMethod("setCreatedAt", new Class[] {String.class});  
            method.invoke(bean, new Object[]{"20100531"});  
        } catch (final Exception e) {  
            e.printStackTrace();  
        }  
          
        System.out.printf("id : [%d] createdAt : [%s]\n", bean.getId(), bean.getCreatedAt());  
    }  
}  
  
class ExampleBean implements Serializable {  
    private static final long serialVersionUID = -8121418052209958014L;  
      
    private int id;  
    private Date createdAt;  
  
    public int getId() {  
        return id;  
    }  
  
    public void setId(int id) {  
        this.id = id;  
    }  
  
    public Date getCreatedAt() {  
        return createdAt;  
    }  
  
    public void setCreatedAt(Date createdAt) {  
        this.createdAt = createdAt;  
    }  
}  

```
参考：
http://blog.csdn.net/zhoudaxia/article/details/30591941
http://blog.csdn.net/luanlouis/article/details/24589193?utm_source=tuicool&utm_medium=referral
http://shensy.iteye.com/blog/1887625