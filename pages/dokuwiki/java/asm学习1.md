title: asm学习1 

#  ASM学习概述 
本系列文章以ASM4.0为基础，参考官方文档及对应的中文翻译版。不过截止目前最新版本为ASM 5.0.不过出入应该不会太大。
ASM官网：http://asm.objectweb.org, http://asm.ow2.org/
ASM 5.0 APIDOC: http://asm.ow2.org/asm50/javadoc/user/index.html
Java字节码查看工具JClassLib:https://github.com/ingokegel/jclasslib ,下载：https://bintray.com/ingokegel/generic/jclasslib/view
Maven：
```

<dependency>
  <groupId>org.ow2.asm</groupId>
  <artifactId>asm-all</artifactId>
  <version>5.0.4</version>
</dependency>

```
**程序分析、程序生成和程序转换**都是非常有用的技术，可在许多应用环境下使用：
  * 程序分析，既可能只是简单的语法分析（syntaxic  parsing），也可能是完整的语义分析（sematic analysis），可用于查找应用程序中的潜在 bug、检测未被用到的代码、对代码 实施逆向工程，等等。
  * 程序生成，在**编译器**中使用。这些编译器不仅包括传统编译器，还包括用于分布式程序 设计的 stub 编译器或 skeleton 编译器，以及 JIT（即时）编译器，等等。
  * 程序转换可，用于**优化或混淆**（obfuscate）程序、向应用程序中插入调试或性能监视代码，用于面向方面的程序设计，等等。

由于程序分析、生成和转换技术的用途众多，所以人们针对许多语言实现了许多用于分析、生成和转换程序的工具，这些语言中就包括 Java 在内。**ASM** 就是为 Java 语言设计的工具之一，用于进行**运行时类生成与转换**
用于进行运行时（也是脱机的）类生成与转换。于是，人们设计了 ASM①库，用于处理经过编译 的 Java 类。这个库的设计使其尽可能保持快速和小型化。对于那些在运行时使用 ASM 进行动态

##  ASM Scope： 
ASM 库的目的是**生成、转换和分析**以字节数组表示的已编译 Java 类（它们在磁盘中的存储 和在 Java 虚拟机中的加载都采用这种**字节数组**形式）。为此，ASM 提供了一些工具，使用高于字节级别的概念来读写和转换这种字节数组，这些概念包括数值常数、字符串、Java 标识符、Java 类型、Java 类结构元素，等等。注意，**ASM 库的范围严格限制于类的读、写、转换和分析。具体来说，类的加载过程就超出了它的范围之外。**
##  API模型 
ASM 库提供了两个用于生成和转换已编译类的 API，一个是**核心 API**，以**基于事件**的形式 来表示类，另一个是**树 API**，以**基于对象**的形式来表示类。
**在采用基于事件的模型**时，类是用一系列事件来表示的，**每个事件表示类的一个元素**，比如 它的一个标头、一个字段、一个方法声明、一条指令，等等。基于事件的 API **定义了一组可能事件，以及这些事件必须遵循的发生顺序**，还提供了一个类分析器，为每个被分析元素生成一个 事件，还提供一个类写入器，由这些事件的序列生成经过编译的类。

而在采用**基于对象的模型**时，**类用一个对象树表示，每个对象表示类的一部分**，比如类本身、 一个字段、一个方法、一条指令，等等，每个对象都有一些引用，指向表示其组成部分的对象。 基于对象的 API 
提供了一种方法，可以将表示一个类的事件序列转换为表示同一个类的对象树， 也可以反过来，将对象树表示为等价的事件序列。换言之，**基于对象的 API 构建在基于事件的 API 之上。**

这两个 API 可以与“用于 XML 的简单 API”（Simple API for XML，SAX）和用于 XML 文 档的“文档对象模型（Document Object Model，DOM）API”相比较：基于事件的 API 类似于 SAX，而基于对象的 API 类似于 DOM。基于对象的 API 构建在基于事件的 API 之上，类似于 DOM 可在 SAX 的上层提供。ASM 之所以要提供两个 API，是因为没有哪种 API 是最佳的。实际上，每个 API 都有自己的优缺点：

` 注意，这两个 API 都是仅能同时维护一个类，而且独立于其他类，也就是说，它们不会维护有关类层级结构的信息，如果类的转换影响到其他类，那其他这些类的修改应当由用户负责完成。 `

##  组织形式 
ASM 库划分为几个包，以几个 jar 文件的形式进行分发：
org.objectweb.asm 和 org.objectweb.asm.signature 包定义了**基于事件的API，并提供了类分析器和写入器组件**。它们包含在 asm.jar 存档文件中。
org.objectweb.asm.util 包，位于 asm-util.jar 存档文件中，提供各种基于 核心 API 的工具，可以在开发和调试 ASM 应用程序时使用。
org.objectweb.asm.commons 包提供了几个很有用的**预定义类转换器**，它们大多是基于核心 API 的。这个包包含在 asm-commons.jar 存档文件中。
org.objectweb.asm.tree 包，位于 asm-tree.jar 存档文件中，定义了**基于对 象的 API**，并提供了一些工具，用于在基于事件和基于对象的表示方法之间进行转换。
org.objectweb.asm.tree.analysis 包提供了一个类分析框架和几个预定义的类分析器，它们以树 API 为基础。这个包包含在 asm-analysis.jar 存档文件中。





