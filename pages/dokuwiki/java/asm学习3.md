title: asm学习3 

#  ASM学习之泛型与注解 
##  泛型 
出于后向兼容的原因，有关泛型的信息没有存储在类型或方法描述符中（它们的定义远早于Java 5 中对泛型的引入），而是保存在称为类型、方法和类**签名**的类似构造中。在涉及泛型时， 除了描述符之外，这些**签名**也会存储在类、字段和方法声明中（泛型不会影响方法的字节代码： 编译器用它们执行静态类型检查，但会在必要时重新引入类型转换，就像这些方法未被使用一样进行编译）。
与类型和方法描述符不同，**类型签名的语法非常复杂，这也是因为泛型的递归本质造成的**（一个泛型可以将另一泛型作为参数——例如，考虑 List<List<E>>）。其语法由以下规则给出（有 关这些规则的完整描述，请参阅《Java 虚拟机规范》）：
  * TypeSignature: Z | C | B | S | I | F | J | D | FieldTypeSignature FieldTypeSignature: 
  * ClassTypeSignature | [ TypeSignature | TypeVar ClassTypeSignature: L Id ( / Id )* TypeArgs? ( . Id 
  * TypeArgs? )* ; TypeArgs: < TypeArg+ >
  * TypeArg: * | ( + | - )? FieldTypeSignature TypeVar: T Id ;
第一条规则表明，类型签名或者是一个基元类型描述符，或者是一个字段类型签名。
第二条规则将一个字段类型签名定义为一个类类型签名、数组类型签名或类型变量。
第三条规则定义类类型签名：它们是类类型描述符，在主类名之后或者内部类名之后的尖括号中可能带有类型参数（以点为前缀）。
其他规则定义了类型参数和类型变量。注意，一个类型参数可能是一个完整的字段类型签名，带有它自己的类型参数：因此，类型签名可能非常复杂（见图 4.1）。
Java
类型和相应的类型签名
```

List<E> Ljava/util/List<TE;>; 
List<?> Ljava/util/List<*>;
List<? extends Number> Ljava/util/List<+Ljava/lang/Number;>;
List<? super Integer> Ljava/util/List<-Ljava/lang/Integer;>; 
List<List<String>[]> Ljava/util/List<[Ljava/util/List<Ljava/lang/String;>;>;
HashMap<K, V>.HashIterator<K> Ljava/util/HashMap<TK;TV;>.HashIterator<TK;>;

```                                      
比如以下泛型静态方法的方法签名，它以类型变量 T 为参数：
static <T> Class<? extends T> m (int n)
它是以下方法签名：
<T:Ljava/lang/Object;>(I)Ljava/lang/Class<+TT;>;
最后要说的是类签名，不要将它与类类型签名相混淆，它被定义为其超类的类型签名，后面跟有所实现接口的类型签名，以及可选的形式类型参数：
ClassSignature: TypeParams? ClassTypeSignature ClassTypeSignature*
例 如 ， 一 个 被 声 明 为  C<E> extends List<E> 的 类 的 类 签 名 就 是
<E:Ljava/lang/Object;>Ljava/util/List<TE;>;。

##  接口与组件 
和描述符的情况一样，也出于相同的效果原因（见 2.3.1 节），ASM API 公开签名的形式与 它们在编译类中的存储形式相同（签名主要出现在 ClassVisitor 类的 visit、visitField 和 visitMethod 方法中，分别作为可选类、类型或方法签名参数 ）。幸好它还在 ` org.objectweb.asm.signature ` 包中提供了一些基于** SignatureVisitor** 抽象类的工具，**用于生成和转换签名**
```

public abstract class SignatureVisitor {
public final static char EXTENDS = ’+’; 
public final static char SUPER = ’-’; 
public final static char INSTANCEOF = ’=’; 
public SignatureVisitor(int api);
public void visitFormalTypeParameter(String name); 
public SignatureVisitor visitClassBound();
public SignatureVisitor visitInterfaceBound();
public SignatureVisitor visitSuperclass(); 
public SignatureVisitor visitInterface(); 
public SignatureVisitor visitParameterType(); 
public SignatureVisitor visitReturnType(); 
public SignatureVisitor visitExceptionType(); 
public void visitBaseType(char descriptor); 
public void visitTypeVariable(String name); 
public SignatureVisitor visitArrayType(); 
public void visitClassType(String name);
public void visitInnerClassType(String name); 
public void visitTypeArgument();
public SignatureVisitor visitTypeArgument(char wildcard);
public void visitEnd();
}

```
这个抽象类用于访问**类型签名、方法签名和类签名**。
用于**类型签名**的方法，必须按以下顺序调用，它反映了前面的语法规则（注意，其中两个返回了 SignatureVisitor：这 是因**为类型签名的递归定义**导致的）：
visitBaseType | visitArrayType | visitTypeVariable | ( visitClassType visitTypeArgument*( visitInnerClassType visitTypeArgument* )* visitEnd ) )

用于访问**方法签名**的方法如下：
( visitFormalTypeParameter visitClassBound? visitInterfaceBound* )*visitParameterType* visitReturnType visitExceptionType*

最后，用于访问**类签名**的方法为：
( visitFormalTypeParameter visitClassBound? visitInterfaceBound* )*visitSuperClass visitInterface*

这些方法大多返回一个 SignatureVisitor：它是准备用来访问类型签名的。注意，不同 于 ClassVisitor 返 回 的 MethodVisitors ， 
**SignatureVisitor 返 回 的 SignatureVisitors 不得为 null，而且必须顺序使用**：事实上，在完全访问一个嵌套签名 之前，不得访问父访问器的任何方法。
和类的情况一样，
**ASM API 基于这个 API 提供了两个组件：SignatureReader 组件分析 一个签名，并针对一个给定的签名访问器调用适当的访问方法；SignatureWriter组件基于它接收到的方法调用生成一个签名。**


工具
2.3 节给出的 TraceClassVisitor 和 ASMifier 类以内部形式打印类文件中包含的签名。 利用它们，可以通过以下方式找出与一个给定泛型相对应的签名：编写一个具有某一泛型的 Java类，编译它，并用这些命令行工具来找出对应的签名。

##  注解 
类、字段、方法和方法参数注解，比如@Deprecated 或@Override，只要它们的保留策 略不是 RetentionPolicy.SOURCE，它们就会被存储在编译后的类中。这一信息不是在运行 时供字节代码指令使用，但是，如果保留策略是 RetentionPolicy.RUNTIME，则可以通过 反射 API 访问它。它还可以供编译器使用。

4.2.1   结构
源 代 码 中 的 注 解 可 以 具 有 各 种 不 同 形 式 ， 比 如 @Deprecated 、@Retention(RetentionPolicy.CLASS)或@Task(desc="refactor", id=1)。但在内
部，所有注解的形式都是相同的，由一种注解类型和一组名称/值对规定，其中的取值仅限于如 下几种：
  * 口  基元，String 或 Class 值
  * 口   枚举值
  * 口   注解值
  * 口   上述值的数组
注意，一个注解中可以包含其他注解，甚至可以包含注解数组。因此，注解可能非常复杂。

4.2.2   接口与组件
用于生成和转换注释的 ASM API 是基于 **AnnotationVisitor** 抽象类的（见图 4.3）。
```

public abstract class AnnotationVisitor { public AnnotationVisitor(int api);
public AnnotationVisitor(int api, AnnotationVisitor av);
public void visit(String name, Object value);
public void visitEnum(String name, String desc, String value); public AnnotationVisitor 
visitAnnotation(String name, String desc); public AnnotationVisitor visitArray(String name);
public void visitEnd();

```

工具
2.3 节介绍的 TraceClassVisitor,  CheckClassAdapter 和 ASMifier 类也支持注释（ 就 像 对 于 方 法 一 样 ， 还 可 能 使 用  TraceAnnotationVisitor  或CheckAnnotationAdapter，在各个注释的级别工作，而不是在类级别工作）。


