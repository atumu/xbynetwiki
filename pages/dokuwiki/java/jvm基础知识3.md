title: jvm基础知识3 

#  class类文件结构 
##  一、Class类文件结构 
![](/data/dokuwiki/java/pasted/20160331-113232.png)
根据Java虚拟机规范的规定，**Class文件格式**采用一种类似于C语言结构体的伪结构来存储，这种伪结构中**只有两种数据类型**：**无符号数和表**。Class类文件严格**按照顺序紧凑的排列**。
  * 无符号数属于基本的数据类型，以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节、8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值，或者按照UTF-8编码构成字符串值。
  * 表是由多个无符号数或者其它表作为数据项构成的复合数据类型，所有表都习惯性地以"_info"结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表。 它由下表所示的数据项构成。
Class类文件格式按如下顺序排列：
![](/data/dokuwiki/java/pasted/20160331-101640.png)
1、**魔数**用来判断该文件是否是Class类文件。很多文件存储标准中都使用魔数来进行身份识别，譬如图片格式，如gif或jpeg等在文件头中都存有魔数。Class文件魔数的值为0xCAFEBABE。如果一个文件不是以0xCAFEBABE开头，那它就肯定不是Java class文件。
**2. 版本号** 
紧接着魔数的4个字节存储的是Class文件的版本号：第5和第6是次版本号（Minior Version），第7个和第8个字节是主版本号(Major Version)。Java的版本号是从45开始的，JDK1.1之后的每个JDK大版本发布主版本号向上加1，高版本的JDK能向下兼容以前版本的Class文件，但不能运行以后版本的Class文件，即使文件格式并未发生变化。JDK1.1能支持版本号为45.0~45.65535的Class文件，JDK1.2则能支持45.0~46.65535的Class文件。JDK1.7可生成的Class文件主版本号的最大值为51.0。 
![](/data/dokuwiki/java/pasted/20160331-105502.png)
###  常量池理解 
3、**常量池的个数从1开始计数**，所以常量池的个数为nstant_pool_count-1。（Class文件结构中只有常量池的容量计数是从1开始的，对于其他集合类型，包括接口索引集合、字段表集合、方法表集合等的容量计数都是从0开始的。）常量池主要存放两大类常量，**字面量以及符号引用**。字面量比较接近于Java语言层面的常量概念，如文本字符串、被声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括了下面三类符号引用常量:
  * ` 类和接口的全限定名 `
  * 字段的名称和描述符
  * 方法的名称和描述符  
常量池的每一项表都是一个表，中共有11中表，具体可以看《深入理解java虚拟机》Page146，上面很详细的介绍而这11中常量，字面量的结构都是一个u1长度的tag，表示这个常量的类型，一个u2长度的length，表示这个常量的长度，以及length个u1长度的bytes（u1，u2，u4，u8分别代表1个字节，2个字节，4个字节，8个字节）。
常量池的11种数据类型:
![](/data/dokuwiki/java/pasted/20160331-134922.png)
每个数据项叫做一个XXX_info项， 比如， 一个常量池中一个CONSTANT_Utf8类型的项， 就是一个CONSTANT_Utf8_info 。除此之外，
**每个info项中都有一个标志值（tag）， 这个标志值表明了这个常量池中的info项的类型是什么**， 从上面的表格中可以看出， 一个CONSTANT_Utf8_info中的tag值为1， 而一个CONSTANT_Fieldref_info中的tag值为9 。
常量池中几乎包含类中的所有信息的描述，** class文件中的很多其他部分都是对常量池中的数据项的引用**，比如后面要讲到的this_class, super_class, field_info, attribute_info等， 另外字节码指令中也存在对常量池的引用， 这个对常量池的引用当做字节码指令的一个操作数。  此外， 常量池中各个项也会相互引用。

**常量池结构如下:**
![](/data/dokuwiki/java/pasted/20160331-113211.png)

**常量池中的数据项是通过索引(从1开始的索引)来引用的， 常量池中的各个数据项之间也会相互引用**。
在这11中常量池数据项类型中， 有两种比较基础， 之所以说它们基础， 是因为这两种类型的数据项会被其他类型的数据项引用。 **这两种数据类型就是CONSTANT_Utf8 和 CONSTANT_NameAndType** ， 其中CONSTANT_NameAndType类型的数据项（CONSTANT_NameAndType_info）也会引用CONSTANT_Utf8类型的数据项（CONSTANT_Utf8_info） 。

**（1） CONSTANT_Utf8_info**
一个CONSTANT_Utf8_info是一个CONSTANT_Utf8类型的常量池数据项， **它存储的是一个常量字符串。 常量池中的所有字面量几乎都是通过CONSTANT_Utf8_info描述的。**下面我们首先讲解CONSTANT_Utf8_info数据项的存储格式。 **常量池中数据项的类型由一个整型的标志值（tag）决定**， 所以所有常量池类型的info中都必须有一个tag信息， 并且这个tag值位于数据项的第一个字节上。 一个11中常量池数据类型， 所以就有11个tag值表示这11中类型。而CONSTANT_Utf8_info的tag值为1， 也就是说如果虚拟机要解析一个常量池数据项， 首先去读这个数据项的第一个字节的tag值， 如果这个tag值为1， 那么就说明这个数据项是一个CONSTANT_Utf8类型的数据项。** 紧挨着tag值的两个字节是存储的字符串的长度length， 剩下的字节就存储着字符串。** 所以， 它的格式是这样的：
![](/data/dokuwiki/java/pasted/20160331-135759.png)
其中tag占一个字节， length占2个字节， bytes代表存储的字符串， 占length字节。所以， 如果这个CONSTANT_Utf8_info存储的是字符串"Hello"， 那么他的存储形式是这样的：
![](/data/dokuwiki/java/pasted/20160331-135825.png)
现在我们知道了CONSTANT_Utf8_info数据项的存储形式， 那么CONSTANT_Utf8_info数据项都存储了什么字符串呢？ CONSTANT_Utf8_info可包括的字符串主要以下这些：
  * 程序中的**字符串常量**
  * 常量池所在当前类（包括接口和枚举）的**全限定名**
  * 常量池所在当前类的直接父类的全限定名
  * 常量池所在当前类型所实现或继承的所有接口的全限定名
  * 常量池所在当前类型中所定义的**字段的名称和描述符**
  * 常量池所在当前类型中所定义的**方法的名称和描述符**
  * 由当前类所引用的类型的全限定名
  * 由当前类所引用的其他类中的字段的名称和描述符
  * 由当前类所引用的其他类中的方法的名称和描述符
  * 与当前class文件中的**属性相关的字符串， 如属性名等**
总结一下，** 其中有这么五类： 程序中的字符串常量， 类型的全限定名， 方法和字段的名称， 方法和字段的描述符， 属性相关字符串。** 
下面我们通过一个例子来进行说明。 示例源码：
```

package com.jg.zhang;  
public class Programer extends Person {  
    static String company = "CompanyA";  
    static{  
        System.out.println("staitc init");  
    }   
    String position;  
    Computer computer;  
    public Programer() {  
        this.position = "engineer";  
        this.computer = new Computer();  
    }  
      
    public void working(){  
        System.out.println("coding...");  
        computer.working();  
    }  
}  

```
别看这个类简单， 但是反编译后， 它的常量池有53项之多。 在这53项常量池数据项中， 各种类型的数据项都有， 当然也包括不少的CONSTANT_Utf8_info 。 
**下面只列出反编译后常量池中的CONSTANT_Utf8_info 数据项**：
```

#2 = Utf8               com/jg/zhang/Programer          //当前类的全限定名  
#4 = Utf8               com/jg/zhang/Person             //父类的全限定名  
#5 = Utf8               company                         //company字段的名称  
#6 = Utf8               Ljava/lang/String;              //company和position字段的描述符  
#7 = Utf8               position                        //position字段的名称  
#8 = Utf8               computer                        //computer字段的名称  
#9 = Utf8               Lcom/jg/zhang/Computer;         //computer字段的描述符  
#10 = Utf8              <clinit>                        //类初始化方法（即静态初始化块）的方法名  
#11 = Utf8              ()V                             //working方法的描述符  
#12 = Utf8              Code                            //Code属性的属性名  
#14 = Utf8              CompanyA                        //程序中的常量字符串  
#19 = Utf8              java/lang/System                //所引用的System类的全限定名  
#21 = Utf8              out                             //所引用的out字段的字段名  
#22 = Utf8              Ljava/io/PrintStream;           //所引用的out字段的描述符  
#24 = Utf8              staitc init                     //程序中的常量字符串  
#27 = Utf8              java/io/PrintStream             //所引用的PrintStream类的全限定名  
#29 = Utf8              println                         //所引用的println方法的方法名  
#30 = Utf8              (Ljava/lang/String;)V           //所引用的println方法的描述符  
#31 = Utf8              LineNumberTable                 //LineNumberTable属性的属性名  
#32 = Utf8              LocalVariableTable              //LocalVariableTable属性的属性名  
#33 = Utf8              <init>                          //当前类的构造方法的方法名  
#41 = Utf8              com/jg/zhang/Computer           //所引用的Computer类的全限定名  
#45 = Utf8              this                            //局部变量this的变量名  
#46 = Utf8              Lcom/jg/zhang/Programer;        //局部变量this的描述符  
#47 = Utf8              working                         //woking方法的方法名  
#49 = Utf8              coding...                       //程序中的字符串常量  
#52 = Utf8              SourceFile                      //SourceFile属性的属性名  
#53 = Utf8              Programer.java                  //当前类所在的源文件的文件名  

```
` 源文件中的几乎所有可见的字符串都存放在CONSTANT_Utf8_info中， 其他类型的常量池项只不过是对CONSTANT_Utf8_info的引用。 其他常量池项， 把引用的CONSTANT_Utf8_info组合起来， 进而可以描述更多的信息。 `

**（2） CONSTANT_NameAndType类型的数据项**
常量池中的一个CONSTANT_NameAndType_info数据项 。 从这个数据项的名称可以看出， **它描述了两种信息，第一种信息是名称（Name）， 第二种信息是类型（Type） 。** 这里的名称是指方法的名称或者字段的名称， **而Type是广义上的类型**， 它**其实描述的是字段的描述符或方法的描述符**。 也就是说， 如果Name部分是一个字段名称， 那么Type部分就是相应字段的描述符； 如果Name部分描述的是一个方法的名称， 那么Type部分就是对应的方法的描述符。** 也就是说， 一个CONSTANT_NameAndType_info就表示了一个方法或一个字段。** 
下面先看一下**CONSTANT_NameAndType_info数据项的存储格式**。 既然是常量池中的一种数据项类型， 那么它的第一个字节也是tag， 它的tag值是12， 也就是说， 当虚拟机读到一个tag为12的常量池数据项， 就可以确定这个数据项是一个CONSTANT_NameAndType_info 。** tag值以下的两个字节叫做name_index， 它指向常量池中的一个CONSTANT_Utf8_info， 这个CONSTANT_Utf8_info中存储的就是方法或字段的名称。 name_index以后的两个字节叫做descriptor_index， 它指向常量池中的一个CONSTANT_Utf8_info， 这个CONSTANT_Utf8_info中存储的就是方法或字段的描述符。** 下图表示它的存储布局：
![](/data/dokuwiki/java/pasted/20160331-140428.png)
下面举一个实例进行说明， 实例的源码为：
```

package com.jg.zhang;  
public class Person {  
  
    int age;  
  
    int getAge(){  
        return age;  
    }  
}  

```
将这段代码使用javap工具反编译之后， 常量池信息如下：
```

 #1 = Class              #2             //  com/jg/zhang/Person  
 #2 = Utf8               com/jg/zhang/Person  
 #3 = Class              #4             //  java/lang/Object  
 #4 = Utf8               java/lang/Object  
 #5 = Utf8               age  
 #6 = Utf8               I  
 #7 = Utf8               <init>  
 #8 = Utf8               ()V  
 #9 = Utf8               Code  
#10 = Methodref          #3.#11         //  java/lang/Object."<init>":()V  
#11 = NameAndType        #7:#8          //  "<init>":()V  
#12 = Utf8               LineNumberTable  
#13 = Utf8               LocalVariableTable  
#14 = Utf8               this  
#15 = Utf8               Lcom/jg/zhang/Person;  
#16 = Utf8               getAge  
#17 = Utf8               ()I  
#18 = Fieldref           #1.#19         //  com/jg/zhang/Person.age:I  
#19 = NameAndType        #5:#6          //  age:I  
#20 = Utf8               SourceFile  
#21 = Utf8               Person.java  

```
常量池一共有21项， 我们可以看到， **一共有两个CONSTANT_NameAndType_info 数据项， 分别是第#11项和第#19项， 其中第#11项的CONSTANT_NameAndType_info**又引用了常量池中的第#7项和第#8项， 被引用的这两项都是CONSTANT_Utf8_info ， **它们中存储的字符串常量值分别是 <init> 和 （）V。 其实他们加起来表示的就是父类Object的构造方法。** 那么这里为什么会是父类Object的构造方法而不是本类的构造方法呢？ **这是因为类中定义的方法如果不被引用（也就是说在当前类中不被调用）， 那么常量池中是不会有相应的 CONSTANT_NameAndType_info 与之对应的， 只有引用了一个方法， 才有相应的CONSTANT_NameAndType_info 与之对应。 这也是为什么说CONSTANT_NameAndType_info 是方法的符号引用的一部分的原因。**
和方法相同， 只定义一个字段而不引用它（在源码中表现为不访问这个变量）， 那么在常量池中也不会存在和该字段相对应的CONSTANT_NameAndType_info 项。这也是为什么说CONSTANT_NameAndType_info作为字段符号引用的一部分的原因。

下面给出这两个CONSTANT_NameAndType_info真实的内存布局图：
和Object构造方法相关的CONSTANT_NameAndType_info的示意图(即#11处的CONSTANT_NameAndType_info)：
![](/data/dokuwiki/java/pasted/20160331-141428.png)
和age字段相关的CONSTANT_NameAndType_info示意图(即#19处的CONSTANT_NameAndType_info)：
![](/data/dokuwiki/java/pasted/20160331-141522.png)
上面主要介绍了常量池中的两种数据项： CONSTANT_NameAndType_info 和 CONSTANT_Utf8_info  。 其中CONSTANT_Utf8_info存储的是源文件中的各种字符串， 而CONSTANT_NameAndType_info表述的是源文件中对一个字段或方法的符号引用的一部分（即 方法名加方法描述符， 或者是 字段名加字段描述符）。

###  其余结构理解 
常量池后面紧接着是类的访问权限控制符，类以及父类的全限定名，以及接口的个数，之后是接口的全限定名，**全限定名都是指向常量池的符号引用。**
再下面就是字段的个数，以及相应个数的表示字段的表，**字段表的结构为：**
![](/data/dokuwiki/java/pasted/20160331-102317.png)
全限定名：com/froest/TestClass;把comm.froest.TestClass中的**"."换成"/"，并且在最后加上";"就成为了全限定名**，简单名称就是域的名称或者方法的名称；比如有方法 int getList(int a,char b,long c)，那么**该方法的描述符为**：(ICJ)I；I为int类型的描述符，C为char类型的描述符，J为long类型的描述符，参数列表用"()"，最后加上返回值的，描述符。

方法表的结构和字段表一样
在Class文件、字段、方法表中都可以携带自己的**属性表**结合，用于描述某些场景专有的信息。**虚拟机预定义的属性如下表所示**：
![](/data/dokuwiki/java/pasted/20160331-102552.png)

**Code属性的表结构如下：**
![](/data/dokuwiki/java/pasted/20160331-102729.png)
**局部变量的顺序，按照this，参数，局部变量。**也就是第一个slot用来存放this(指向常量池中该类的符号引用，是一个地址)，参数在局部变量中从第2个slot开始存放。

4、访问标志
紧接常量池后的两个字节称为access_flags，它展示了文件中定义的类或接口的几段信息。例如，访问标志指明文件中定义的是类还是接口;访问标志还定义的在类或接口的声明中，使用了哪种修饰符oder和接口是抽象的，还是公共的;类的类型可以为final，而final类不可能是抽象的;接口不能为final类型的。这些标志位的定义如下表所示：
![](/data/dokuwiki/java/pasted/20160331-112107.png)
 如一个TestClass类被public关键字修饰但没有被声明为final和abstract,并且它使用了JDK1.2之后的编译器进行编译，因此它的ACC_PUBLIC、ACC_SUPER标志应该为真。因此它的access_flags的值应为:0x0001|0x0020 = 0x0021。
5、类索引
访问标志后面接下来的两个字节是**类索引(this_class)，它是一个对常量池的索引**。在this_class位置的常量池入口必须为**CONSTANT_Class_info表**。该表由两个部分组成——tag和name_index。tag部分是代表其的标志位，name_index位置的常量池入口为**一个包含了类或接口全限定名的CONSTANT_Utf8_info表。** 
6、父类索引
在class文件中，紧接在this_class之后是super_class项，它是一个两个字节的常量池索引。在super_class位置的常量池入口是一个指向该类超类全限定名的CONSTANT_Class_info入口。因为Java程序中所有对象的基类都是java.lang.Object类，除了Object类以外，常量池索引super_class对于所有的类均有效。**对于Object类，super_class的值为0。**对于接口，在常量池入口super_class位置的项为java.lang.Object

7.interfaces_count和interfaces
紧接着super_class的是interfaces_count，此项的含义为：在文件中出该类直接实现或者由接口所扩展的父接口的数量。在这个计数的后面，是名为interfaces的数组，它包含了对每个由该类或者接口直接实现的父接口的常量池索引。每个父接口都使用一个常量池中的CONSTANT_Class_info入口来描述，该CONSTANT_Class_info入口指向接口的全限定名。这个数组只容纳那些直接出现在类声明的implements子句或者接口声明的extends子句中的父接口。超类按照在implements子句和extends子句中出现的顺序在这个数组中显现。 

8. fields_count和fields
在class文件中，紧接在interfaces后面的是对在该类或者接口中所声明的字段的描述。首先是名为fields_count的计数，它是类变量和实例变量的字段的数量总和。在这个计数后面的是不同长度的field_info表的序列(fields_count指出了序列中有多少个field_info表)。只有在文件中由类或者接口声明了的字段才能在fields列表中列出。在fields列表中，不列出从超类或者父接口继承而来的字段。另一方面，fields列表可能会包含在对应的Java源文件中没有叙述的字段，这是因为Java编译器可以会在编译时向类或者接口添加字段。
在Java中，描述字段的信息有：字段的作用域、是实例变量还是类变量（static）、可变性(final)、并发可见性(volatile)、可否序列化(trasient)、字段数据类型、字段名称。这些信息中，各个修饰符都是布尔值，要么有某个修饰符，要么没有，很适合使用标志位来表示。而字段叫什么名字、字段被定义为什么数据类型，这些都是无法固定的，只能引用常量池中的常量来描述。
![](/data/dokuwiki/java/pasted/20160331-112705.png)
 字段修饰符放在access_flags项目中，它与类中的access_flags项目是非常相似的，都是一个u2的数据类型，其中可以设置 的标志位和含义如下表所示
![](/data/dokuwiki/java/pasted/20160331-112720.png)
**跟随access_flags标志的是两项索引值：name_index和descriptor_index。它们都是对常量池的引用，分别代表着字段的简单名称及字段和方法的描述符**。描述符的作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符规则，基本数据类型(byte、char、double、float、int、long、short、boolean)及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符L加对象的全限定名来表示。
![](/data/dokuwiki/java/pasted/20160331-112820.png)
 对于数组类型，每一个维度将使用一个前置的"["字符来描述，如一个定义的"java.lang.String[][]"类型的二维数组，将被记录为:"[ [Ljava/lang/String;",一个整型数组"int[]"将被记录为"[I"
用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号"()"之内。如方法void inc()的描述符为"()V"，方法java.lang.String.toString()的描述符为"()Ljava/lang/String;"。

9.method_count和methods
紧接着field后面的是对在该类或者接口中所声明的方法的描述。其结构与fields一样，不一样的是访问标志。
![](/data/dokuwiki/java/pasted/20160331-113000.png)

 10.attributes_count和attributes
**class文件中最后的部分是属性，它给出了在该文件类或者接口所定义的属性的基本信息。**属性部分由attributes_count开始，attributes_count是指出现在后续attributes列表的attribute_info表的数量总和。每个attribute_info的第一项是指向常量池中CONSTANT_Utf8_info表的引引，该表给出了属性的名称。
**属性有许多种。Java虚拟机规范定义了几种属性**，但任何人都可以创建他们自己的属性种类，并且把它们置于class文件中，Java虚拟机实现必须忽略任何不能识别的属性。
java虚拟机预设的9项虚拟机应当能识别的属性如下表所示。
![](/data/dokuwiki/java/pasted/20160331-113110.png)
##  二、类加载机制 
　　**类加载按加载，连接，初始化这个顺序进行的，其中连接又可以细分为验证，准备，解析三个阶段**，部分解析可以在初始化开始之后再开始，这样可以支持java的运行时绑定。虽然部分解析可以在初始化阶段开始以后再开始，但是这部分的初始化还是需要当前的部分解析以后才可以初始化。
**java虚拟机规范中严格规定了有且之友中情况必须立即对类进行初始化**：
　　1）遇到new创建实例，getstatic获取类的静态字段，putstatic设置静态字段，invokestatic调用类的静态方法
　　2）用java.lang.reflect包方法对类进行反射调用的时候，如果这个类没有初始化过，那么先触发其初始化
　　3）初始化一个类的时候，如果父类没有进行初始化，那么必须先触发其父类的初始化
　　4）当虚拟机启动的时候，需要指定一个执行的主类，虚拟机会先初始化这个主类
用new关键字创建数组不会触发相应的类初始化。**调用一个类的静态常量也不会触发该类的初始化**，因为**调用类在编译阶段就已经把常量转化为对自己的常量池的引用**，例：
```

class ConstClass {
  static {
    System.out.println("ConstClass init");
  }
  public final static String HELLODWORLD = "hello world";
}

public class NotInitialization {
  public static void main(String[] args) {
    //调用静态常量不会触发其初始化案例
    System.out.println(ConstClass.HELLODWORLD);
  }
}

```
**加载阶段**是整个类加载阶段的第一个阶段，在加载阶段主要完成3件事情：
　　1）通过类的全限定名来回去定义此类的二进制流
　　2）将这个二进制流所代表的静态存储结构转化为方法区的运行时数据结构
　　3）在java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口。  
**验证阶段**是连接阶段的第一步，则以不的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段分为4中验证：Class文件格式的验证，元数据的验证，字节码的验证，符号引用验证。

**准备阶段**是正式为类变量（被static修饰的变量）分配初始值。
```

public static int a = 123;//类变量在准备阶段初始化的值为0，而在初始化阶段，在<cinit>构造方法(即类构造方法，实例构造方法为<init>)中会把a的值初始化为123
public static final int a = 123;//用final修饰的类变量在准备阶段，会把a的值初始化为123

```
**解析阶段**就是把虚拟机**在常量池中的符号引用替换为直接引用的过程**，解析动作主要针对类或接口、字段、类方法、接口方法四类符号引用。

**初始化过程**是执行` 类构造器<cinit> `()方法的过程，**<cinit>()会字段收集类中的所有类变量以及静态语句块（static{}）**，在初始化<cinit>()方法的时候，虚拟机会自动调用父类的<cinit>()方法，接口的<cinit>()方法可以到使用的时候在去初始化，虚拟机会保证<cinit>()方法在多线程环境先被正确的加锁和同步。还有一个**<init>()方法，这个方法是实例构造器，在创建实例的时候会被调用并且初始化。**
` 任意一个类，都需要加载它的类加载器和这个类本身一同确定其在java虚拟机中的唯一性。 `.如下述示例：
```

public class ClassLoaderTest {
  public static void main(String[] args) throws Exception {
    ClassLoader myClassLoader = new ClassLoader() {

      @Override
      public Class<?> loadClass(String name) throws ClassNotFoundException {
        try {
          String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
          InputStream is = getClass().getResourceAsStream(fileName);
          if (is == null) {
            return super.loadClass(name);
          }
          byte[] b = new byte[is.available()];
          is.read(b);
          return defineClass(name, b, 0, b.length);
        } catch (Exception e) {
          throw new ClassNotFoundException(name);
        }
      }

    };
    Object obj = myClassLoader.loadClass("com.froest.excel.ClassLoaderTest").newInstance();
    System.out.println(obj.getClass());
    //验证运行时类的唯一性
    System.out.println(obj instanceof com.froest.excel.ClassLoaderTest);//输出false。说明类加载器和这个类全限定类名一起才能确定其在java虚拟机中的唯一性。
  }
}

```
上面代码执行的结果为：
　　class com.froest.excel.ClassLoaderTest
　　false
第一个输出表示obj确实是com.froest.excel.ClassLoaderTest实例化出来的对象，` 但是第二个类型检查确实false，这是因为虚拟机的内存中有两个ClassLoaderTest类，一个是应用程序加载器加载的，另外一个是我们自定义的类加载器加载的，虽然是同一个Class，但是还是独立的两个类。 `
` 类加载器使用双亲委派模型，这样要加载一个类，首先查找这个类是否已经被加载过，如果没有，那么类加载器会把这个类委派给这个加载器的父类去进行加载，如果父类不能加载，那么再自己加载。 `
##  三、字节码执行 
首先看一个数据结构---**栈帧**，**栈帧是用于支持虚拟机进行方法调用和方法执行的数据结构**，他是虚拟机运行时数据区的虚拟机栈的栈元素。**栈帧中从上到下依次存储了局部变量表，操作数栈，动态链接，返回地址等信息，每个方法从调用开始到调用结束，都对应着一个栈帧从入栈到出栈的过程。一个线程中只有栈顶的栈帧才是有效的，称为当前栈帧，这个栈帧所关联的方法就是当前方法。**
操作数栈中的元素的数据类型要与字节码指令的序列完全一致。
经过优化的虚拟机令两个栈帧的局部变量表重叠一部分，公用一部分数据，这样可以减少额外的参数的复制传递了。
**动态链接就是在运行期间把符号引用转化为直接引用的过程**，相对于静态解析（在加载阶段的解析阶段把符号引用转化为直接引用）
**方法的返回地址，方法返回有两种类型，一种是正常完成出口，另一种是异常完成出口，方法退出的过程等同于栈帧出栈**，因此栈帧出栈的时候可能执行的操作有：恢复上层调用方法的局部变量表和操作数栈，把返回值压入调用方法的操作数栈中，调整PC计数器的值以执行方法调用指令的后一条指令等。
java虚拟机中的**调用指令有invokestatic（调用静态方法），invokespecial（调用实例构造器(<init>()方法),私有方法，父类方法），invokevirtual（调用所有的虚方法），invokeinterface（调用接口方法，会在运行时再确定一个实现此接口的对象）。**只要能被invokestatic和invokespecial指令调用的方法，这些方法叫做非虚方法，都可以在解析阶段确定唯一的调用版本，这种方法在类加载的时候就会符号引用解析为直接引用。**相反的被invokevirtual和invokeinterface指令调用的方法叫做` 虚方法 `**（除了final方法，因为final方法不允许被修改，只有一种形式）。
静态分派：所有依赖静态类型来定位方法执行版本的分派动作都称为静态分派，**静态分派的典型应用就是方法重载。**
动态分派：` 在运行期根据实际类型确定方法的执行版本的分派过程称为动态分派 `，**动态分派的典型应用就是方法重写。**
宗量：方法的接受者和方法的参数统称为方法的宗量。
  * 单分派：根据一个宗量对目标方法进行选择
  * 多分派：根据多个宗量对目标方法进行选择
java是一种静态多分派，动态单分派语言。
**类的方法区会保存一张虚方法表，存放方法的实际入口地址**，如果没有重写父类的方法，那么入口与父类的一样，如果重写了父类的方法，那么方法的入口地址指向自己的方法入口地址。方法表一般在类加载的连接阶段进行初始化，准备了类变量的初始值之后，虚拟机会把该类的方法表也初始化完毕，这是java实现动态分派方法。


参考：http://www.cnblogs.com/God-froest/archive/2013/08/31/class_.html
http://www.cnblogs.com/xiaoruoen/archive/2011/11/30/2267309.html
http://www.cnblogs.com/aigongsi/archive/2012/04/15/2450190.html
http://blog.csdn.net/zhangjg_blog/article/details/21557357