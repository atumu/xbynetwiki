title: class.getresource和classloader.getresource的路径问题 

#  关于Class.getResource和ClassLoader.getResource的路径问题 
Java中取资源时，经常用到Class.getResource和ClassLoader.getResource，这里来看看他们在取资源文件时候的路径问题。
##  Class.getResource(String path) 
**Class.getResource(String path)
path不以’/'开头时，默认是从此类所在的包下取资源；
path  以’/'开头时，则是从ClassPath根下获取；**
什么意思呢？看下面这段代码的输出结果就明白了：
```

package testpackage;
public class TestMain {
    public static void main(String[] args) {
        System.out.println(TestMain.class.getResource(""));
        System.out.println(TestMain.class.getResource("/"));
    }
}

```
输出结果：
file:/E:/workspace/Test/bin/testpackage/
file:/E:/workspace/Test/bin/
上面说到的【path以’/'开头时，则是从ClassPath根下获取；】在这里就是相当于bin目录(Eclipse环境下)。
再来一个实例，假设有如下Project结构：
![](/data/dokuwiki/java/pasted/20160224-110653.png)

如果我们想在TestMain.java中分别取到1~3.properties文件，该怎么写路径呢？代码如下：
```

package testpackage;
public class TestMain {

    public static void main(String[] args) {
        // 当前类(class)所在的包目录
        System.out.println(TestMain.class.getResource(""));
        // class path根目录
        System.out.println(TestMain.class.getResource("/"));
        
        // TestMain.class在<bin>/testpackage包中
        // 2.properties  在<bin>/testpackage包中
        System.out.println(TestMain.class.getResource("2.properties"));
        
        // TestMain.class在<bin>/testpackage包中
        // 3.properties  在<bin>/testpackage.subpackage包中
        System.out.println(TestMain.class.getResource("subpackage/3.properties"));
        
        // TestMain.class在<bin>/testpackage包中
        // 1.properties  在bin目录（class根目录）
        System.out.println(TestMain.class.getResource("/1.properties"));
    }
}

```

※Class.getResource和Class.getResourceAsStream在使用时，路径选择上是一样的。

##  Class.getClassLoader().getResource(String path) 

**Class.getClassLoader（）.getResource(String path)
path不能以’/'开头时；
path是从ClassPath根下获取；**
还是先看一下下面这段代码的输出：
```

package testpackage;
public class TestMain {
    public static void main(String[] args) {
        TestMain t = new TestMain();
        System.out.println(t.getClass());
        System.out.println(t.getClass().getClassLoader());
        System.out.println(t.getClass().getClassLoader().getResource(""));
        System.out.println(t.getClass().getClassLoader().getResource("/"));//null
    }
}

```
输出结果：
class testpackage.TestMain
sun.misc.Launcher$AppClassLoader@1fb8ee3
file:/E:/workspace/Test/bin/
null
从结果来看【` TestMain.class.getResource("/") == t.getClass().getClassLoader().getResource("") `】
如果有同样的Project结构
![](/data/dokuwiki/java/pasted/20160224-110831.png)
```

使用Class.getClassLoader（）.getResource(String path)可以这么写：
package testpackage;
public class TestMain {
    public static void main(String[] args) {
        TestMain t = new TestMain();
        System.out.println(t.getClass().getClassLoader().getResource(""));
        
        System.out.println(t.getClass().getClassLoader().getResource("1.properties"));
        System.out.println(t.getClass().getClassLoader().getResource("testpackage/2.properties"));
        System.out.println(t.getClass().getClassLoader().getResource("testpackage/subpackage/3.properties"));
    }
}

```

##  WEB-INF/lib/xxx.jar里面的某个类的ClassLoader.getResource/.class.getResource() 
**WEB-INF/lib/xxx.jar里面的某个类的ClassLoader.getResource/.class.getResource()里面获取到的classpath和外WEB-INF/classes里面类获取到的classpath根目录是不同的。**
所以在使用的时候需要特别注意这一点。

##  关于ClassLoader.getResourceAsStream额外说明 
class.getResourceAsStream最终调用是ClassLoader.getResourceAsStream
只是在这之前对参数进行了调整。如果参数已/开头，则去除/，否则把当前类的包名加在参数的前面。
在使用ClassLoader.getResourceAsStream时，路径直接使用相对于classpath的绝对路径,并且不能已 / 开头。
InputStream resourceAsStream = ClassLoader.getSystemResourceAsStream("com/github/demo/1.txt");

建议：
```

ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
InputStream input = classLoader.getSystemResourceAsStream(txtFile);

```

##  Spring的PropertiesLoaderUtils参考 
org.springframework.core.io.support.PropertiesLoaderUtils。
org.springframework.util.ClassUtils
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
参考：http://www.cnblogs.com/yejg1212/p/3270152.html
http://stackoverflow.com/search?q=ClassLoader.getSystemResourceAsStream+jar