title: jar包中读取资源文件 

#  java:jar包中读取资源文件 
参考：http://blog.csdn.net/b_h_l/article/details/7767829
从jar包中读取资源文件，会涉及到` ClassLoader `的知识。
比如：当我们在Eclipse的Java项目中使用和Maven项目中使用，会出现不同的情形。所以我们需要用自己jar包的ClassLoader去去读取资源文件。
以Maven为例：**` 注意Maven中src/main/resource下的资源,在执行mvn process-resources的时候会将此文件夹下的东西全部拷贝到classes文件夹下.也就是说打包成jar包时src/main/resource下的资源文件处于jar包根路径下。 `**
资源文件路径 src/main/resource/component.xml
```

public static final String COMP_XML_CONFIG_PATH="component.xml";
private static final ClassLoader loader=Thread.currentThread().getContextClassLoader();//此处ClassLoader如不正确，则无法读取
InputStream  ins=loader.getResourceAsStream(COMP_XML_CONFIG_PATH);//ClassLoader，此处只能使用相对路径 

```

(1)`  ClassLoader ` 是类加载器的抽象类。它可以在运行时动态的获取加载类的运行信息。 可以这样说，当我们调用ResourceJar.jar中的Resource类时，JVM加载进Resource类，并记录下Resource运行时信息(包括Resource所在jar包的路径信息)。而ClassLoader类中的方法可以帮助我们动态的获取这些信息:
● public URL getResource(String name) 
 查找具有给定名称的资源。资源是可以通过类代码以与代码基无关的方式访问的一些数据(图像、声音、文本等)。并返回资源的URL对象。
● public InputStream ` getResourceAsStream(String name); ` 
 返回读取指定资源的输入流。这个方法很重要，可以直接获得jar包中文件的内容。
(2) ClassLoader是abstract的，不可能实例化对象，更加不可能通过ClassLoader调用上面两个方法。所以我们真正写代码的时候，是通过Class类中的getResource()和getResourceAsStream()方法，这两个方法会委托ClassLoader中的getResource()和getResourceAsStream()方法 
```

       //查找指定资源的URL，其中res.txt仍然开始的bin目录下     
        URL fileURL=this.getClass().getResource("/resource/res.txt");     
        System.out.println(fileURL.getFile());

```  
我们将这段代码打包成ResourceJar.jar ,并将ResourceJar.jar放在其他路径下(比如 c:\ResourceJar.jar)。然后另外创建一个java project并导入ResourceJar.jar，写一段调用jar包中Resource类的测试代码：
```

public class TEST {    
    public static void main(String[] args) throws IOException {    
        Resource res=new Resource();    
        res.getResource();    
    }    
} 

```   
这时的运行结果是：file:/C:/ResourceJar.` jar! `/resource/res.txt。显然不能作为` File f=new File("file:/C:/ResourceJar.jar!/resource/res.txt")将返回null. `
所以我们需要记住第三点。
(3) 我们不能用常规操作文件的方法来读取ResourceJar.jar中的资源文件res.txt，但可以通过Class类的` getResourceAsStream() `方法来获取 ，这种方法是如何读取jar中的资源文件的，这一点对于我们来说是透明的。