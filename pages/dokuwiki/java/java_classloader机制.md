title: java_classloader机制 

#  java classLoader机制 
参考：
http://longdick.iteye.com/blog/442213/
http://blog.chinaunix.net/uid-21227800-id-65885.html
http://www.cnblogs.com/xiaoruoen/archive/2011/12/01/2262788.html
http://www.importnew.com/6579.html
建议后续参考：
Java 类的热替换 http://www.ibm.com/developerworks/cn/java/j-lo-hotswapcls/index.html
自定义ClassLoader，用于加载用户JAR包 http://obullxl.iteye.com/blog/651128/
Java Classloader机制解析 http://my.oschina.net/aminqiao/blog/262601
Java的ClassLoader机制解析 http://developer.51cto.com/art/201111/303470.htm

类从被加载到虚拟机内存中开始，到卸装出内存为止，它的整个生命周期包括了:加载，连接(验证，准备，解析)，初始化，使用和卸载七个阶段。其中验证、准备和解析三个部分称为连接，也就是说，**一个Java类从字节代码到能够在JVM中被使用，需要经过加载、链接和初始化这三个步骤** 。我们看一看Java虚拟机的体系结构。
Java虚拟机的体系结构如下图所示:
![](/data/dokuwiki/java/pasted/20160331-143123.png)
Java类加载的全过程，是加载、验证、准备、解析和初始化这五个阶段的过程。而加载阶段是类加载过程的一个阶段。在加载阶段，虚拟机需要完成以下三件事情：
  * 通过一个类的全限定名来获取定义此类的二进制字节流。
  * 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
  * 在Java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口。 
在这三件事情中，通过一个类的全限定名来获取定义此类的二进制字节流这个动作是在Java虚拟机外部来实现的，以便让应用程序自己决定如何去获取所需要的类。实现这个动作的代码模块被称为“类加载器”。
**系统提供的类装载器主要由下面三个：**
  * 启动(引导)类加载器（bootstrap classloader）：它用来加载 Java 的核心库，是用原生代码(本地代码，与平台有关)来实现的。这个类加载器负责将存放在<JAVA_HOME>\lib目录中的，或者被-Xbootclasspath参数所指定的路径中的，并且是虚拟机识加的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用。
  * 扩展类加载器（extensions classloader）：扩展类加载器是由 Sun 的 ExtClassLoader（sun.misc.Launcher$ExtClassLoader） 实现的。它负责将 < Java_Runtime_Home >/lib/ext 或者由系统变量java.ext.dir 指定位置中的类库加载到内存中
  * 应用程序类加载器（application classloader）：系统类加载器是由 Sun 的 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的，**由于这个类加载器是ClassLoader中getSystemClassLoader()方法的返回值，所以一般也称它为系统类加载器。**它根据 Java 应用的类路径（` CLASSPATH） `来加载 Java 类。开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序默认的类加载器。(一般在JavaSE独立程序中通过main函数启动的情况)

**用户自定义的类装载器 ** 
用户自定义的类装载器是普通的Java对象，它的类必须` 派生自java.lang.ClassLoader类 `。ClassLoader中定义的方法为程序为程序提供了访问类装载器机制的接口。此外，对于每一个被装载的类型，Java虚拟机都会为它创建一个` java.lang.Class `类的实例来代表该类型。和所有其它对象一样，用户自定义的类装载器以有**Class类的实例都放在内存中的堆区，而装载的类型信息则都放在方法区。**
下面介绍一下java.lang.ClassLoader类：
为了完成加载类的这个职责， java.lang.ClassLoader类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个 Java 类，即 java.lang.Class类的一个实例。除此之外，ClassLoader还负责加载 Java 应用所需的资源，如图像文件和配置文件等。ClassLoader提供了一系列的方法，比较重要的方法如下表所示：
  * getParent()	返回该类加载器的父类加载器。
  * loadClass(String name)	加载名称为 name的类，返回的结果是 java.lang.Class类的实例。
  * findClass(String name)	查找名称为 name的类，返回的结果是 java.lang.Class类的实例。
  * findLoadedClass(String name)	查找名称为 name的已经被加载过的类，返回的结果是 java.lang.Class类的实例。
  * defineClass(String name, byte[] b, int off, int len)	把字节数组 b中的内容转换成 Java 类，返回的结果是 java.lang.Class类的实例。**这个方法被声明为 final的。**
  * resolveClass(Class<?> c)	链接指定的 Java 类。

类装载器的特征 
**Java类加载器有两个比较重要的特征：层次组织结构和代理模式。**这两个特征也就是我们平时说的**类加载器的双亲委派模型**。层次组织结构指的是除了顶层为启动类加载器之外，其余的类加载器都有一个父类加载器，通过` getParent()方法 `可以获取到。类加载器通过这种父亲-后代的方式组织在一起，**形成树状层次结构**。这里类加载器之间的父子关系一般不会以继承的关系来实现，**而是都使用组合关系来复用父加载器的代码。**代理模式则指的是一个类加载器既可以自己完成Java类的定义工作，也可以代理给其它的类加载器来完成。

类加载器的树状组织结构
在系统提供的类加载器中，除了启动类加载器之外，所有的类加载器都有一个父类加载器。通过getParent()方法可以得到。对于系统提供的类加载器来说，系统类加载器的父类加载器是扩展类加载器，而扩展类加载器的父类加载器是启动类加载器；对于开发人员编写的类加载器来说，其父类加载器是加载此类加载器 Java 类的类加载器。因为类加载器 Java 类如同其它的 Java 类一样，也是要由类加载器来加载的。一般来说，开发人员编写的类加载器的父类加载器是系统类加载器。类加载器通过这种方式组织起来，形成树状结构。树的根节点就是引导类加载器。
![](/data/dokuwiki/java/pasted/20160331-143847.png)
```

public class ClassloaderTree {
    public static void main(String[] args) {
        ClassLoader loader = ClassloaderTree.class.getClassLoader();
        while(loader!=null){
            System.out.println(loader);
            loader = loader.getParent();
        }

    }

}

```
打印出来的结果为：
sun.misc.Launcher$AppClassLoader@1372a1a 
sun.misc.Launcher$ExtClassLoader@ad3ba4
并没有输出引导类加载器，这是由于有些 JDK 的实现对于父类加载器是引导类加载器的情况，getParent()方法返回 null 

代理模式
**代理模式说的是双亲委派模型的工作过程：**如果一个类加载器收到了类加载的请求，**它首先不会自己尝试加载这个类，而是把这个请求委派给父类加载器去完成**，每一个层次的类加载器都是如此，**因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围没有找到所需的类）时，子加载器才会尝试自己去加载。** 
那么为什么要使用代理模式呢，每个类加载器都由自己加载不是很好吗，搞得这么复杂干吗？要解释这个，**就得首先说明Java虚拟机是如何判定两个类是相同的。在Java虚拟机中，对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性，通俗来说，Java 虚拟机不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。即便是同样的字节代码，被不同的类加载器加载之后所得到的类，也是不同的。**这个说的相同，包括代表类的Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法的返回结果，也包括了使用instanceof关键字做对象所属关系判定等情况。 
在 上表中列出的 java.lang.ClassLoader类的常用方法中，一般来说，**自己开发的类加载器只需要覆写 findClass(String name)方法即可。java.lang.ClassLoader类的方法loadClass()封装了前面提到的代理模式的实现（loadClass的具体实现如下所示）。该方法会首先调用 findLoadedClass()方法来检查该类是否已经被加载过；如果没有加载过的话，会调用父类加载器的 loadClass()方法来尝试加载该类；如果父类加载器无法加载该类的话，就调用 findClass()方法来查找该类。
因此，` 为了保证类加载器都正确实现代理模式，在开发自己的类加载器时，最好不要覆写 loadClass()方法，而是覆写findClass()方法。 `**
**代理模式是为了保证 Java 核心库的类型安全。**所有 Java 应用都至少需要引用 java.lang.Object类，也就是说在运行的时候，java.lang.Object这个类需要被加载到 Java 虚拟机中。如果这个加载过程由 Java 应用自己的类加载器来完成的话，很可能就存在多个版本的 java.lang.Object类（由上面的例子就可以看出来），而且这些类之间是不兼容的。通过代理模式，对于 Java 核心库的类的加载工作由引导类加载器来统一完成，保证了 Java 应用所使用的都是同一个版本的 Java 核心库的类，是互相兼容的。

线程上下文类加载器
**线程上下文类加载器（context class loader）**是从 JDK 1.2 开始引入的。**类 java.lang.Thread中的方法 getContextClassLoader()和setContextClassLoader(ClassLoader cl)用来获取和设置线程的上下文类加载器。**如果没有通过setContextClassLoader(ClassLoader cl)方法进行设置的话，**线程将继承其父线程的上下文类加载器。**Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。
前面提到的类加载器的代理模式并不能解决 Java 应用开发中会遇到的类加载器的全部问题。Java 提供了很多服务提供者接口（Service Provider Interface，SPI），允许第三方为这些接口提供实现。常见的 SPI 有 JDBC、JCE、JNDI、JAXP 和 JBI 等。这些 SPI 的接口由 Java 核心库来提供，如 JAXP 的 SPI 接口定义包含在 javax.xml.parsers包中。这些 SPI 的实现代码很可能是作为 Java 应用所依赖的 jar 包被包含进来，可以通过类路径（CLASSPATH）来找到，如实现了 JAXP SPI 的
Apache  Xerces所包含的 jar 包。SPI 接口中的代码经常需要加载具体的实现类。如 JAXP 中的 javax.xml.parsers.DocumentBuilderFactory类中的newInstance()方法用来生成一个新的 DocumentBuilderFactory的实例。这里的实例的真正的类是继承自javax.xml.parsers.DocumentBuilderFactory，由 SPI 的实现所提供的。如在 Apache Xerces 中，实现的类是org.apache.xerces.jaxp.DocumentBuilderFactoryImpl。而问题在于，SPI 的接口是 Java 核心库的一部分，是由引导类加载器来加载的；SPI 实现的 Java 类一般是由系统类加载器来加载的。引导类加载器是无法找到 SPI 的实现类的，因为它只加载 Java 的核心库。它也不能代理给系统类加载器，因为它是系统类加载器的祖先类加载器。也就是说，类加载器的代理模式无法解决这个问题。
线程上下文类加载器正好解决了这个问题**。如果不做任何的设置，Java 应用的线程的上下文类加载器默认就是系统上下文类加载器。**在 SPI 接口的代码中使用线程上下文类加载器，就可以成功的加载到 SPI 实现的类。**线程上下文类加载器在很多 SPI 的实现中都会用到。**

##  ClassLoader流程 

java应用环境中不同的class分别由不同的ClassLoader负责加载。
一个jvm中默认的classloader有` Bootstrap ClassLoader、Extension ClassLoader、App ClassLoader `，分别各司其职：
  * Bootstrap ClassLoader    	 负责加载java基础类，主要是 %JRE_HOME/lib/ 目录下的rt.jar、resources.jar、charsets.jar和class等
  * Extension ClassLoader   	   负责加载java扩展类，主要是 %JRE_HOME/lib/ext 目录下的jar和class
  * App ClassLoader           负责加载当前java应用的classpath中的所有类。 
  * 
其中Bootstrap ClassLoader是JVM级别的，由C++撰写；
Extension ClassLoader、App ClassLoader都是java类，**都继承自` URLClassLoader `超类。**
Bootstrap ClassLoader由JVM启动，然后初始化sun.misc.Launcher ，sun.misc.Launcher初始化Extension ClassLoader、App ClassLoader。
下图是ClassLoader的加载类流程图，以加载一个类的过程类示例说明整个ClassLoader的过程。
![](/data/dokuwiki/java/pasted/20150911-043149.png)![](/data/dokuwiki/java/pasted/20150911-043138.png)

 Bootstrap ClassLoader、Extension ClassLoader、App ClassLoader三者的关系如下：
Bootstrap ClassLoader是Extension ClassLoader的parent，Extension ClassLoader是App ClassLoader的parent。
**但是这并不是继承关系，只是语义上的定义，**基本上，**每一个ClassLoader实现，都有一个Parent ClassLoader。**
可以通过ClassLoader的getParent方法得到当前ClassLoader的parent。Bootstrap ClassLoader比较特殊，因为它不是java class所以Extension ClassLoader的getParent方法返回的是NULL。

##  双亲委托模型 
Java中ClassLoader的加载采用了**双亲委托机制**，采用双亲委托机制加载类的时候采用如下的几个步骤：

1、当前ClassLoader首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类。
2、每个类加载器都有自己的加载缓存，当一个类被加载了以后就会放入缓存，等下次加载的时候就可以直接返回了。
3、当前classLoader的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到bootstrp ClassLoader.
4、当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回。否则抛出ClassNotFoundException

说到这里大家可能会想，**Java为什么要采用这样的委托机制？**理解这个问题，我们引入另外一个关于**Classloader的概念“命名空间”**， 它是指**要确定某一个类，需要类的全限定名以及加载此类的ClassLoader来共同确定。**也就是说**即使两个类的全限定名是相同的，但是因为不同的 ClassLoader加载了此类，那么在JVM中它是不同的类。**明白了命名空间以后，我们再来看看委托模型。采用了委托模型以后加大了不同的 ClassLoader的交互能力，**比如上面说的，我们JDK本生提供的类库**，比如hashmap,linkedlist等等，这些类由bootstrp 类加载器加载了以后，无论你程序中有多少个类加载器，那么**这些类其实都是可以共享的，**这样就**避免了不同的类加载器加载了同样名字的不同类以后造成混乱。**
##  ClassLoader与其加载路径 
**每个ClassLoader定义的加载路径可能不同。我们的ClassLoader加载资源时是对于自己的加载路径而言的。**

Spring中的org.springframework.util.ClassUtils类中的获取默认加载器的方法
```

public static ClassLoader getDefaultClassLoader() {
		ClassLoader cl = null;
		try {
			cl = Thread.currentThread().getContextClassLoader();
		} catch (Throwable localThrowable) {
		}
		if (cl == null) {
			cl = ClassUtils.class.getClassLoader();
			if (cl == null) {
				try {
					cl = ClassLoader.getSystemClassLoader();
				} catch (Throwable localThrowable1) {
				}
			}
		}
		return cl;
	}

```
##  自定义ClassLoader： 
由于一些特殊的需求，我们可能需要定制ClassLoader的加载行为，这时候就需要自定义ClassLoader了.
自定义ClassLoader需要继承ClassLoader抽象类，重写findClass方法，这个方法定义了ClassLoader查找class的方式。

  * findLoadedClass：**每个类加载器都维护有自己的一份已加载类名字空间**，其中不能出现两个同名的类。凡是通过该类加载器加载的类，**无论是直接的还是间接的，都保存在自己的名字空间中**，该方法就是在该名字空间中寻找指定的类是否已存在，如果存在就返回给类的引用，否则就返回 null。这里的直接是指，存在于**该类加载器的加载路径**上并由该加载器完成加载，间接是指，由该类加载器把类的加载工作委托给其他类加载器完成类的实际加载。
  * getSystemClassLoader：Java2 中新增的方法。该方法返回系统使用的 ClassLoader。可以在自己定制的类加载器中通过该方法把一部分工作转交给系统类加载器去处理。
  * defineClass：该方法是 ClassLoader 中非常重要的一个方法，**它接收以字节数组表示的类字节码，并把它转换成 Class 实例，该方法转换一个类的同时，会先要求装载该类的父类以及实现的接口类。**
  * loadClass：**加载类的入口方法，调用该方法完成类的显式加载。通过对该方法的重新实现，我们可以完全控制和管理类的加载过程。**
  * resolveClass：链接一个指定的类。这是一个在某些情况下确保类可用的必要方法，详见 Java 语言规范中“执行”一章对该方法的描述。

主要可以扩展的方法有：
  * findClass          定义查找Class的方式
  * defineClass       将类文件字节码加载为jvm中的class
  * findResource    定义查找资源的方式
 
如果嫌麻烦的话，我们可以直接使用或继承已有的ClassLoader实现，比如
  *  java.net.URLClassLoader
  *  java.security.SecureClassLoader
  *  java.rmi.server.RMIClassLoader
  *  sun.applet.AppletClassLoader
Extension ClassLoader 和 App ClassLoader都是` java.net.URLClassLoader `的子类。
这个是URLClassLoader的构造方法：
  * public URLClassLoader(URL[] urls, ClassLoader parent)
  * public URLClassLoader(URL[] urls)
urls参数是需要加载的ClassPath url数组，可以指定parent ClassLoader，不指定的话默认以当前调用类的ClassLoader为parent。
```

ClassLoader classLoader = new URLClassLoader(urls);  
Thread.currentThread().setContextClassLoader(classLoader);  
Class clazz=classLoader.loadClass("com.company.MyClass");//使用loadClass方法加载class,这个class是在urls参数指定的classpath下边。  
  
Method taskMethod = clazz.getMethod("doTask", String.class, String.class);//然后我们就可以用反射做些事情了  
taskMethod.invoke(clazz.newInstance(),"hello","world");  

```
由于classloader 加载类用的是**全盘负责委托机制**。所谓全盘负责，即是当一个classloader加载一个Class的时候，这个Class所依赖的和引用的所有 Class也由这个classloader负责载入，除非是显式的使用另外一个classloader载入。
所以，当我们自定义的classloader加载成功了com.company.MyClass以后，MyClass里所有依赖的class都由这个classLoader来加载完成。
自定义ClassLoader在某些应用场景还是比较适用，特别是需要灵活地动态加载class的时候。


##  定制的类加载器的实现代码 
实现一个定制的类加载器来完成这样的加载流程：我们为该类加载器指定一些必须由该类加载器直接加载的类集合，在该类加载器进行类的加载时，如果要加载的类属于必须由该类加载器加载的集合，那么就由它直接来完成类的加载，否则就把类加载的工作委托给系统的类加载器完成。
在给出示例代码前，有两点内容需要说明一下：1、要想实现同一个类的不同版本的共存，那么这些不同版本必须由不同的类加载器进行加载，因此就不能把这些类的加载工作委托给系统加载器来完成，因为它们只有一份。2、为了做到这一点，就不能采用系统默认的类加载器委托规则，也就是说我们定制的类加载器的父加载器必须设置为 null。该定制的类加载器的实现代码如下：
```

class CustomCL extends ClassLoader { 

	private String basedir; // 需要该类加载器直接加载的类文件的基目录
    private HashSet dynaclazns; // 需要由该类加载器直接加载的类名

    public CustomCL(String basedir, String[] clazns) { 
        super(null); // 指定父类加载器为 null 
        this.basedir = basedir; 
        dynaclazns = new HashSet(); 
        loadClassByMe(clazns); 
    } 

    private void loadClassByMe(String[] clazns) { 
        for (int i = 0; i < clazns.length; i++) { 
            loadDirectly(clazns[i]); 
            dynaclazns.add(clazns[i]); 
        } 
    } 

    private Class loadDirectly(String name) { 
        Class cls = null; 
        StringBuffer sb = new StringBuffer(basedir); 
        String classname = name.replace('.', File.separatorChar) + ".class";
        sb.append(File.separator + classname); 
        File classF = new File(sb.toString()); 
        cls = instantiateClass(name,new FileInputStream(classF),
            classF.length()); 
        return cls; 
    }   		

    private Class instantiateClass(String name,InputStream fin,long len){ 
        byte[] raw = new byte[(int) len]; 
        fin.read(raw); 
        fin.close(); 
        return defineClass(name,raw,0,raw.length); 
    } 
    
	protected Class loadClass(String name, boolean resolve) 
            throws ClassNotFoundException { 
        Class cls = null; 
        cls = findLoadedClass(name); 
        if(!this.dynaclazns.contains(name) && cls == null) 
            cls = getSystemClassLoader().loadClass(name); 
        if (cls == null) 
            throw new ClassNotFoundException(name); 
        if (resolve) 
            resolveClass(cls); 
        return cls; 
    } 

}

```
在该类加载器的实现中，所有指定必须由它直接加载的类都在该加载器实例化时进行了加载，当通过 loadClass 进行类的加载时，如果该类没有加载过，并且不属于必须由该类加载器加载之列都委托给系统加载器进行加载。理解了这个实现，距离实现类的热替换就只有一步之遥了。

##  实现 Java 类的热替换 
在本小节中，我们将结合前面讲述的类加载器的特性，并在上小节实现的自定义类加载器的基础上实现 Java 类的热替换。首先我们把上小节中实现的类加载器的类名 CustomCL 更改为 HotswapCL，以明确表达我们的意图。
为了简单起见，我们的包为默认包，没有层次，并且省去了所有错误处理。要替换的类为 Foo，实现很简单，仅包含一个方法 sayHello：
清单 2. 待替换的示例类
```

public class Foo{ 
    public void sayHello() { 
        System.out.println("hello world! (version one)"); 
    } 
}

```
在当前工作目录下建立一个新的目录 swap，把编译好的 Foo.class 文件放在该目录中。接下来要使用我们前面编写的 HotswapCL 来实现该类的热替换。具体的做法为：我们编写一个定时器任务，每隔 2 秒钟执行一次。其中，我们会创建新的类加载器实例加载 Foo 类，生成实例，并调用 sayHello 方法。接下来，我们会修改 Foo 类中 sayHello 方法的打印内容，重新编译，并在系统运行的情况下替换掉原来的 Foo.class，我们会看到系统会打印出更改后的内容。定时任务的实现如下（其它代码省略，请读者自行补齐）：
清单 3. 实现定时任务的部分代码
```

public void run(){ 
    try { 
        // 每次都创建出一个新的类加载器
        HowswapCL cl = new HowswapCL("../swap", new String[]{"Foo"}); 
        Class cls = cl.loadClass("Foo"); 
        Object foo = cls.newInstance(); 

        Method m = foo.getClass().getMethod("sayHello", new Class[]{}); 
        m.invoke(foo, new Object[]{}); 
    
    }  catch(Exception ex) { 
        ex.printStackTrace(); 
    } 
}

```
编译、运行我们的系统，会出现如下的打印：
图 3. 热替换前的运行结果
![](/data/dokuwiki/java/pasted/20150911-044713.png)

好，现在我们把 Foo 类的 sayHello 方法更改为：
```

public void sayHello() { 
    System.out.println("hello world! (version two)"); 
}

```
在系统仍在运行的情况下，编译，并替换掉 swap 目录下原来的 Foo.class 文件，我们再看看屏幕的打印，奇妙的事情发生了，新更改的类在线即时生效了，我们已经实现了 Foo 类的热替换。屏幕打印如下：
![](/data/dokuwiki/java/pasted/20150911-044719.png)

**敏锐的读者可能会问，为何不用把 foo 转型为 Foo，直接调用其 sayHello 方法呢？这样不是更清晰明了吗？**下面我们来解释一下原因，并给出一种更好的方法。
如果我们采用转型的方法，代码会变成这样：Foo foo = (Foo)cls.newInstance(); 读者如果跟随本文进行试验的话，**会发现这句话会抛出 ClassCastException 异常，为什么吗？**
因为在 Java 中，即使是同一个类文件，如果是由不同的类加载器实例加载的，**那么它们的类型是不相同的**。在上面的例子中 cls 是由 HowswapCL 加载的，而 foo 变量类型声名和转型里的 Foo 类却是由 run 方法所属的类的加载器（默认为 AppClassLoader）加载的，因此是完全不同的类型，所以会抛出转型异常。

那么通过接口调用是不是就行了呢？我们可以定义一个 IFoo 接口，其中声名 sayHello 方法，Foo 实现该接口。也就是这样：IFoo foo = (IFoo)cls.newInstance(); 本来该方法也会有同样的问题的，因为外部声名和转型部分的 IFoo 是由 run 方法所属的类加载器加载的，而 Foo 类定义中 implements IFoo 中的 IFoo 是由 HotswapCL 加载的，因此属于不同的类型转型还是会抛出异常的，但是由于我们在实例化 HotswapCL 时是这样的：
HowswapCL cl = new HowswapCL("../swap", new String[]{"Foo"});
其中仅仅指定 Foo 类由 HotswapCL 加载，而其实现的 IFoo 接口文件会委托给系统类加载器加载，因此转型成功，采用接口调用的代码如下：
清单 4. 采用接口调用的代码
```

public void run(){ 
    try { 
        HowswapCL cl = new HowswapCL("../swap", new String[]{"Foo"}); 
        Class cls = cl.loadClass("Foo"); 
        IFoo foo = (IFoo)cls.newInstance(); 
        foo.sayHello(); 
    } catch(Exception ex) { 
        ex.printStackTrace(); 
    } 
}

```
确实，简洁明了了很多。**在我们的实验中，每当定时器调度到 run 方法时，我们都会创建一个新的 HotswapCL 实例，在产品代码中，无需如此，仅当需要升级替换时才去创建一个新的类加载器实例。**
更多请参考http://www.ibm.com/developerworks/cn/java/j-lo-hotswapcls/index.html

##   线程中的ClassLoader 
 线程中的ClassLoader每个运行中的线程都有一个成员` contextClassLoader， `用来在运行时动态地载入其它类，可以使用方法Thread.currentThread().setContextClassLoader(...);更改当前线程的contextClassLoader，来改变其载入类的行为；也可以通过方法` Thread.currentThread().getContextClassLoader() `来获得当前线程的ClassLoader。  
实际上，在Java应用中所有程序都运行在线程里，如果在程序中没有手工设置过ClassLoader，对于一般的java类如下两种方法获得的ClassLoader通常都是同一个  
  * this.getClass.getClassLoader()；  
  * Thread.currentThread().getContextClassLoader()；  
**方法一得到的Classloader是静态的，表明类的载入者是谁；方法二得到的Classloader是动态的，谁执行（某个线程）**，就是那个执行者的Classloader。
<note tip>对于单例模式的类，静态类等，载入一次后，这个实例会被很多程序（线程）调用，对于这些类，载入的Classloader和执行线程的Classloader通常都不同。</note>

**1.6 获得ClassLoader的几种方法可以通过如下3种方法得到ClassLoader ** 
```

this.getClass.getClassLoader(); // 使用当前类的ClassLoader  
Thread.currentThread().getContextClassLoader(); // 使用当前线程的ClassLoader  
ClassLoader.getSystemClassLoader(); // 使用系统ClassLoader，即系统的入口点所使用的ClassLoader。（JVM下system ClassLoader通常为App ClassLoader）

```  

##  ClassLoader配置文件载入 
使用的路径是相对于这个ClassLoader的那个点的` 相对路径 `
```

/** 
 * 因为有3种方法得到ClassLoader，对应有如下3种方法读取文件 
 * 使用的路径是相对于这个ClassLoader的那个点的相对路径，此处只能使用相对路径 
 */ 
InputStream is = null; 
is = this.getClass().getClassLoader().getResourceAsStream( 
       "com/rain/config/sys.properties"); //方法1 
//is = Thread.currentThread().getContextClassLoader().getResourceAsStream( 
       "com/rain/config/sys.properties"); //方法2 
//is = ClassLoader.getSystemResourceAsStream("com/rain/config/sys.properties"); //方法3 

```
如果是配置文件，可以通过java.util.Properties.load(is)将内容读到Properties里，这里要注意编码问题。  

##  在web应用里载入资源 
在web应用里当然也可以使用ClassLoader来载入资源，**但更常用的情况是使用ServletContext**，如下是web目录结构  
    ContextRoot 
       |- JSP、HTML、Image等各种文件 
        |- [WEB-INF] 
              |- web.xml 
              |- [lib] Web用到的JAR文件 
                |- [classes] 类文件 
用户程序通常在classes目录下，如果想读取classes目录里的文件，可以使用ClassLoader，如果想读取其他的文件，一般使用` ServletContext.getResource()  ` 
如果**使用ServletContext.getResource(path)方法，路径必须以"/"开始，路径被解释成相对于ContextRoot的路径，（而ClassLoader是以相对这个点的classpath的相对路径）**
此处载入文件的方法和ClassLoader不同，举例"/WEB-INF/web.xml","/download/WebExAgent.rar"
##  资源加载顺序 
一般先是线程上下文的classloader加载，然后是装载当前类的classLoader加载，而当前类的classLoader加载时遵循双亲委托的原则，先委托父ClassLoader进行加载，一直到BootstrapClassLoader，最后才是当前类自己进行尝试加载。

##  自定义ClassLoader，用于加载用户JAR包 
**方式一、比较简洁，继承URLClassLoader:**
```

public class StartClassLoader extends URLClassLoader {

	private static final String JAR = ".jar";
	private static final String ZIP = ".zip";
  
	public StartClassLoader() {  
        	this(getSystemClassLoader());  //设置parent为SystemClassLoader.
   	 }  
  
   	 public StartClassLoader(ClassLoader parent) {  
       		 super(new URL[] {}, parent);  
    	}  

	/**
	 * @param libHome
	 * @return
	 */
	public void  loadLib(String libHome) {
		// lib目录下包的URL
		List<URL> urlList = new ArrayList<URL>();
		// 递归
		loopLibFiles(new File(libHome), urlList);
		// 转换
		URL[] urlArray = new URL[urlList.size()];
		for (int i = 0; i < urlList.size(); i++) {
			urlArray[i] = urlList.get(i);
		}
		super(urlArray);
	}

	/**
	 * 递归
	 * 
	 * @param file
	 * @param urlList
	 */
	private final void loopLibFiles(File file, List<URL> urlList) {
		if (file.isDirectory()) {
			File[] tmps = file.listFiles();
			for (File tmp : tmps) {
				loopLibFiles(tmp, urlList);
			}
		} else {
			String p = file.getAbsolutePath();
			if (p.endsWith(JAR) || p.endsWith(ZIP)) {
				try {
					urlList.add(file.toURI().toURL());
				} catch (MalformedURLException e) {
					e.printStackTrace();
				}
			}
		}
	}

}

```
测试以下：
```

public class ClassLoaderTest {  
  
    public static final String ZIP_PATH = "G:/Lib/Quaqua 3.9.4/lib/quaqua.jar";  
    public static void main(String[] args) throws Throwable {  
        StartClassLoader loader1 = new StartClassLoader();  
        loader1.addlib(ZIP_PATH);  
        Class<?> clazz1 = loader1.loadClass("ch.randelshofer.quaqua.AnimatedBorder");  
  
        StartClassLoader loader2 = new StartClassLoader();  
        loader2.addlib(ZIP_PATH);  
        Class<?> clazz2 = loader2.loadClass("ch.randelshofer.quaqua.AnimatedBorder");  
  
        System.out.println(clazz1 == clazz2);  
    }  
}  

```

**方式二、稍微繁琐点.**
参考：http://obullxl.iteye.com/blog/651128/
自动升级过程中，升级文件的JAR包是专门加载到程序中去的，因此，自定义一个ClassLoader，用于加载用户JAR包，就非常的重要了。
应用程序ClassLoader只提供了一个public Class<?> loadClass(String name) throws ClassNotFoundException 方法，没有提供加载JAR的方法。
URLClassLoader提供了一个protected void addURL(URL url)的方法，倒是可以加载JAR包，但苦于非public的。
AppClassLoader是URLClassLoader的子类。因此，我们完全可以利用URLClassLoader了哦。
```

URLClassLoader system = (URLClassLoader) ClassLoader.getSystemClassLoader();

```
这样，我们可以通过反射得到addURL方法，在程序中加载我们自己的JAR包了。
```

/** 
 * @author obullxl 
 * 
 * email: obullxl@163.com  MSN: obullxl@hotmail.com  QQ: 303630027 
 * 
 * Blog: http://obullxl.iteye.com 
 */  
public final class ClassLoaderUtil {  
    /** URLClassLoader的addURL方法 */  
    private static Method addURL = initAddMethod();  
      
    /** 初始化方法 */  
    private static final Method initAddMethod() {  
        try {  
            Method add = URLClassLoader.class  
                .getDeclaredMethod("addURL", new Class[] { URL.class });  
            add.setAccessible(true);  
            return add;  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
        return null;  
    }  
  
    private static URLClassLoader system = (URLClassLoader) ClassLoader.getSystemClassLoader();  
  
    /** 
     * 循环遍历目录，找出所有的JAR包 
     */  
    private static final void loopFiles(File file, List<File> files) {  
        if (file.isDirectory()) {  
            File[] tmps = file.listFiles();  
            for (File tmp : tmps) {  
                loopFiles(tmp, files);  
            }  
        } else {  
            if (file.getAbsolutePath().endsWith(".jar") || file.getAbsolutePath().endsWith(".zip")) {  
                files.add(file);  
            }  
        }  
    }  
  
    /** 
     * <pre> 
     * 加载JAR文件 
     * </pre> 
     * 
     * @param file 
     */  
    public static final void loadJarFile(File file) {  
        try {  
            addURL.invoke(system, new Object[] { file.toURI().toURL() });  
            System.out.println("加载JAR包：" + file.getAbsolutePath());  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
  
    /** 
     * <pre> 
     * 从一个目录加载所有JAR文件 
     * </pre> 
     * 
     * @param path 
     */  
    public static final void loadJarPath(String path) {  
        List<File> files = new ArrayList<File>();  
        File lib = new File(path);  
        loopFiles(lib, files);  
        for (File file : files) {  
            loadJarFile(file);  
        }  
    }  
}  

```
在程序中，只要使用上面最后两个方法，就可以加载自定义JAR包和一个目录中的所有JAR包了。

##  类什么时候初始化 
**类的初始化意味着它会初始化所有类静态成员。**
` 通常通过Class.forName()加载的类，加载完成之后会立即进行初始化。但是通过ClassLoader.loadClass()加载完类后，类的初始化不会发生，它只是负责加载，验证和解析。 `
**以下情况一个类被初始化：**
  * 实例通过使用new()关键字创建或者使用class.forName()反射，但它有可能导致ClassNotFoundException。
  * 类的静态方法被调用
  * 类的静态域被赋值
  * 静态域被访问，而且它不是常量
  * 在顶层类中执行assert语句
  * 反射同样可以使类初始化，比如java.lang.reflect包下面的某些方法，JLS严格的说明**：一个类不会被任何除以上之外的原因初始化。**

**类是如何被初始化的**
现在我们知道什么时候触发类的初始化了，他精确地写在Java语言规范中。但了解清楚 域（fields，静态的还是非静态的）、块（block静态的还是非静态的）、不同类（子类和超类）和不同的接口（子接口，实现类和超接口）的初始化顺序也很重要类。事实上很多核心Java面试题和SCJP问题都是基于这些概念，**下面是类初始化的一些规则：**
  * 类从顶至底的顺序初始化，所以声明在顶部的字段的早于底部的字段初始化
  * 超类早于子类和衍生类的初始化
  * 如果类的初始化是由于访问静态域而触发，那么只有声明静态域的类才被初始化，而不会触发超类的初始化或者子类的初始化即使静态域被子类或子接口或者它的实现类所引用。
  * 接口初始化不会导致父接口的初始化。
  * 静态域的初始化是在类的静态初始化期间，非静态域的初始化时在类的实例创建期间。这意味这静态域初始化在非静态域之前。
  * 非静态域通过构造器初始化，子类在做任何初始化之前构造器会隐含地调用父类的构造器，他保证了非静态或实例变量（父类）初始化早于子类
初始化例子
这是一个有关类被初始化的例子，你可以看到哪个类被初始化
```

/**
 * Java program to demonstrate class loading and initialization in Java.
 */
public class ClassInitializationTest {
 
    public static void main(String args[]) throws InterruptedException {
 
        NotUsed o = null; //this class is not used, should not be initialized
        Child t = new Child(); //initializing sub class, should trigger super class initialization
        System.out.println((Object)o == (Object)t);
    }
}
 
/**
 * Super class to demonstrate that Super class is loaded and initialized before Subclass.
 */
class Parent {
    static { System.out.println("static block of Super class is initialized"); }
    {System.out.println("non static blocks in super class is initialized");}
}
 
/**
 * Java class which is not used in this program, consequently not loaded by JVM
 */
class NotUsed {
    static { System.out.println("NotUsed Class is initialized "); }
}
 
/**
 * Sub class of Parent, demonstrate when exactly sub class loading and initialization occurs.
 */
class Child extends Parent {
    static { System.out.println("static block of Sub class is initialized in Java "); }
    {System.out.println("non static blocks in sub class is initialized");}
}
 
Output:
static block of Super class is initialized
static block of Sub class is initialized in Java
non static blocks in super class is initialized
non static blocks in sub class is initialized
false

```
从上面结果可以看出：
  * 超类初始化早于子类
  * 静态变量或代码块初始化早于非静态块和域
  * 没使用的类根本不会被初始化，因为他没有被使用
再来看一个例子：
```

/**
 * Another Java program example to demonstrate class initialization and loading in Java.
 */
 
public class ClassInitializationTest {
 
    public static void main(String args[]) throws InterruptedException {
 
       //accessing static field of Parent through child, should only initialize Parent
       System.out.println(Child.familyName);
    }
}
 
class Parent {
    //compile time constant, accessing this will not trigger class initialization
    //protected static final String familyName = "Lawson";
 
    protected static String familyName = "Lawson";
 
    static { System.out.println("static block of Super class is initialized"); }
    {System.out.println("non static blocks in super class is initialized");}
}
 
Output:
static block of Super class is initialized
Lawson

```
分析：
**这里的初始化发生是因为有静态域被访问，而且不一个编译时常量。**如果声明的”familyName”是**使用final关键字修饰的编译时常量使用（就是上面的注释代码块部分）超类的初始化就不会发生**。
` 尽管静态与被子类所引用但是也仅仅是超类被初始化 `
还有另外一个例子与接口相关的，JLS清晰地解释**子接口的初始化不会触发父接口的初始化**。强烈推荐阅读JLS14.4理解类加载和初始化细节。以上所有就是有关类被初始化和加载的全部内容。

##  Class.forName()与ClassLoader的区别 
**Java的体系结构允许动态扩展Java程序，这个过程包括运行时决定所使用的类型，装载它们，使用它们。**
**通过传递类型的名字到java.lang.Class的forName()方法，或者用户自定义的类装载器的loadClass()方法，可以动态扩展Java程序。**两种方法都可以使运行中的程序去调用在源代码中未曾提及的，而是在程序运行中决定的类型。动态扩展的例子如支持Java的Web浏览器，它跨网络装载applet的class文件。当浏览器启动的时候，它不知道将要从网络上装载什么class文件，当它遇到包含这些applet的网页的时候才知道每个applet所需的类和接口的名字。
**动态扩展Java程序最直接的方式就是使用java.lang.Class的forName()方法**，它有两种重载形式。
```

 public static Class<?> forName(String className) 
                throws ClassNotFoundException {
        return forName0(className, true, ClassLoader.getCallerClassLoader());
    }

public static Class<?> forName(String name, boolean initialize,
				   ClassLoader loader)
        throws ClassNotFoundException
    {
	if (loader == null) {
	    SecurityManager sm = System.getSecurityManager();
	    if (sm != null) {
		ClassLoader ccl = ClassLoader.getCallerClassLoader();
		if (ccl != null) {
		    sm.checkPermission(
			SecurityConstants.GET_CLASSLOADER_PERMISSION);
		}
	    }
	}
	return forName0(name, initialize, loader);
    }

```
forName()的三参数形式是在1.2版中加入的，将类型的全限定名装入String类型的className参数。` 如果boolean类型的initialize参数为true,类型会在forName()方法返回之前连接并初使始化;如果initialize参数为false,类型会被装载，可能会被连接但是不会被forName()方法明确地初始化。 `然而，如果该类型在调用forName()之前已经被初始化了，即使将false作为第二个参数传递到forName()，返回的类型也已经被初始化了。第三个参数为ClassLoader，传递一个用户定制的类装载器的引用给forName(),让其使用这个类装载器来请求类型。也可以指定forName()用默认的启动类装载器来请求类型，只需传递null作为ClassLoader参数。forName()还有一个只采用一个参数的版本，它总是使用当前的类装载器（就是装载执行forName()请求的类的类装载器），并且总是初始化该类型。两个版本的forName()方法都返回Class实例的引用，它代表被装载的类 型。如果类型无法被装载，会抛出ClassNotFoundException异常。
**动态扩展Java程序的另外一种方式就是使用用户自定义的类装载器的loadClass()方法。如果需要用自定义的类装载器请求类型` ，只需调用那个类装载器的loadClass()方法 `。类ClassLoader包含两个名为loadClass()的重载方法，其形式如下：**
```

 public Class<?> loadClass(String name) throws ClassNotFoundException {
	return loadClass(name, false);
    }
protected synchronized Class<?> loadClass(String name, boolean resolve)
	throws ClassNotFoundException
    {
	// First, check if the class has already been loaded
	Class c = findLoadedClass(name);
	if (c == null) {
	    try {
		if (parent != null) {
		    c = parent.loadClass(name, false);
		} else {
		    c = findBootstrapClass0(name);
		}
	    } catch (ClassNotFoundException e) {
	        // If still not found, then invoke findClass in order
	        // to find the class.
	        c = findClass(name);
	    }
	}
	if (resolve) {
	    resolveClass(c);
	}
	return c;
    }

```
两个loadClass()方法都接受装载类型的全限定名装入String类型的name参数。loadClass()的语义和forName()是一样的。如果loadClass()方法已经用String类型的name参数传递的全限定名装载了类型，它会返回这个已经被装载的类型的Class实例。否则，该方法会试图用某种用户定制的方式来装载请求的类型。如果类装载器用定制的方式成功地装载了类型。loadClass()应该返回一个Class的实例，表示新装载的类型。否则方法将抛出ClassNotFoundException异常。
双参数版本的loadClass()中，boolean类型的resolve参数表示是否在装载时执行该类型的连接。连接包含三个步骤：校验、准备、解析。如果resolve参数为true,loadClass()方法会确保在方法返回某个类型的Class实例之前已经装载并连接了该类型。如果resolve参数是false，loadClass()方法仅仅去试图装载请求的类型，而不关心类型是否被连接了。` 双参数版本的loadClass()是一个过时的方法，实际上从Java1.1开始，resolve参数就没有作用了。通常，应该调用单参数版本的loadClass()时，它会试图装载类型并返回而把连接和初始化类型的进行留给虚拟机去掌握。 `
loadClass实例:
```

import java.io.InputStream;
public class MyClassLoader extends ClassLoader {
  	//不建议覆盖loadClass方法，建议重写findClass方法
	@Override
	public Class<?> loadClass(String name) throws ClassNotFoundException {
		// TODO Auto-generated method stub
		String fileName = name.substring(name.lastIndexOf(".")+1)+".class";
		InputStream is = getClass().getResourceAsStream(fileName);
		if( is == null){
			return super.loadClass(name);
		}
		try {
			byte[] b = new byte[is.available()];
			is.read(b);
			return defineClass(name,b,0,b.length);
		} catch (IOException e) {
			e.printStackTrace();
		}
		return super.loadClass(name);
	}


}

```
```

public class ClassLoaderTest {
	public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException, SecurityException, NoSuchMethodException, IllegalArgumentException, InvocationTargetException {
		MyClassLoader loader = new MyClassLoader();
		Class c = loader.loadClass("com.xiaoruoen.test.Greet");
		Object obj = c.newInstance();
		Method method = c.getMethod("greet");
		method.invoke(obj);
	}

}

```
使用forName()还是调用用户自定义的类装载器的laodClass()方法取决于用户的需要。如果没有特别的类装载器的要求，或许应该用forName(),因为forName()是动态扩展最直接的方法。另外，` 如果需要请求的类型在装载时就初始化（并且连接）的等方面，则不得不使用forName()。当loadClass()方法返回类型的时候，类型有可能没有被连接，但谳用单参数版本的forName()方法或者调用它的三参数版本并且传递true作为initialize参灵敏的值 时，返回的类型一事实上是已经被连接、初始化过了。 `
` 初始化有时很重要的。比如JDBC驱动程序通常用forName()调用装载的。因为每一个JDBC驱动程序类的静态方法都用DriverManager注册驱动程序，这样才能被应用程序所使用，驱动程序类必须被初始化，而不是仅仅被加载。如果一个驱动程序被装载了，但是没有初始化，那么类的静态初始化方法就无法被执行，驱动程序就没有在DriverManager中被注册，驱动程序就无法被应用程序使用。 `
**类装载器可以满足一些forName()无法满足的需求。如果需要一些特定的装载类型的方法，比如从网络上下载，从数据库中取出，从加密文件是提取，甚至动态地创建它们，这时就需要一个类装载器。**
