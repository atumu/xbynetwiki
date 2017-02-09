title: java_classloader机制2 

#  java classLoader机制与类扫描初步 
##  获取所有classpath路径和其下jar文件 
**方式一、通过java.class.path属性获取**
```

String[] classpath=System.getProperty("java.class.path").trim().split(";");
for(String str:classpath){
	System.out.println(str);
}

```
在eclipse下运行web项目输出:
D:\work\workspace\eclipse_java_work\spring-test\target\test-classes
D:\work\workspace\eclipse_java_work\spring-test\target\classes
D:\work\tomcat7\lib\annotations-api.jar
D:\work\tomcat7\lib\catalina-ant.jar
......
D:\work\tomcat7\lib\websocket-api.jar
D:\work\workspace\eclipse_java_work\spring-test\src\main\webapp\WEB-INF\lib\json.jar
D:\work\workspace\eclipse_java_work\spring-test\src\main\webapp\WEB-INF\lib\ueditor-1.1.2.jar
C:\Users\Administrator\.m2\repository\org\springframework\spring-context\4.2.3.RELEASE\spring-context-4.2.3.RELEASE.jar
.......

**方式二、通过URLClassLoader.getURLs获取**
```

public static File[] getDefaultClasspath(ClassLoader classLoader) {
		Set<File> classpaths = new TreeSet<>();
		while (classLoader != null) {
			if (classLoader instanceof URLClassLoader) {
				URL[] urls = ((URLClassLoader) classLoader).getURLs();
				for (URL u : urls) {
					File f = FileUtil.toFile(u);
					if ((f != null) && f.exists()) {
						try {
							f = f.getCanonicalFile();
							boolean newElement = classpaths.add(f);
						} catch (IOException ignore) {
						}
					}
				}
			}
			classLoader = classLoader.getParent();
		}
		return classpaths.toArray(new File[classpaths.size()]);
	}

```

##  获取方法的调用者Class 
分析：
```

public class Test1 {
	public static void main(String[] args) {
		Test2.show();
	}
}

```
```

public class Test2 {
	public static void show() {
		Test3.show();
	}
}

```
```

public class Test3 {
	public static void show(){
		StackTraceElement[] eles=new Throwable().getStackTrace();//StackTraceElement[] eles=Thread.currentThread().getStackTrace();【1】
		for(StackTraceElement ele:eles){
			System.out.println(ele.getClassName());
			System.out.println("----------");
		}

	}
}

```
输出:
```

net.xby1993.springmvc.entity.Test3
----------
net.xby1993.springmvc.entity.Test2
----------
net.xby1993.springmvc.entity.Test1
----------
如果将1处的代码换成注释掉的那句StackTraceElement[] eles=Thread.currentThread().getStackTrace();那么输出结果为:
java.lang.Thread
---
net.xby1993.springmvc.entity.Test3
----------
net.xby1993.springmvc.entity.Test2
----------
net.xby1993.springmvc.entity.Test1
----------

```
我们发现多了一行。第一行总是java.lang.Thread.所以我们通常推荐使用` StackTraceElement[] eles=new Throwable().getStackTrace() `;来进行分析

**总结，最终写法：**
```

	/**
	 * 获取方法调用者的Class
	 * @param level 方法调用层次，以1开始，一般取2或3，在使用Throwable().getStackTrace()处理时取2，在使用Thread.currentThread().getStackTrace()进行处理时取3
	 * @return
	 */
	public static Class getCallerClass(int level) {
		StackTraceElement[] eles=new Throwable().getStackTrace();
		if(level<=0||level>eles.length){
			level=eles.length;
		}
		String className=eles[level-1].getClassName();
		try {
			return Thread.currentThread().getContextClassLoader().loadClass(className);
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
		
	}

```
参考http://stackoverflow.com/questions/11306811/how-to-get-the-caller-class-in-java

##  获取默认的ClassLoader 
```

	public static ClassLoader getDefaultClassLoader() {
		ClassLoader cl = null;
		try {
			cl = Thread.currentThread().getContextClassLoader();
		} catch (Throwable localThrowable) {
		}
          	if(cl==null){
			cl=getCallerClass(2).getClassLoader();
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
	/**
	 * 获取方法调用者的Class
	 * @param level 方法调用层次，以1开始，一般取2或3
	 * @return
	 */
	public static Class getCallerClass(int level) {
		StackTraceElement[] eles=new Throwable().getStackTrace();
		if(level<=0||level>eles.length){
			level=eles.length;
		}
		String className=eles[level-1].getClassName();
		try {
			return Thread.currentThread().getContextClassLoader().loadClass(className);
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
		
	}

```


##  类扫描时过滤含有某个注解的class文件 
优势：无需加载Class实例，然后通过反射判断。直接以文件流的形式判断。
例如Test.class是否含有@TestAnn注解
思路通过获取@TestAnn注解的类型描述符转换成字节数组，然后判断由Test.class转换的InputStream中是否含有那个字节数组，如果有则包含，否则不包含。
```

	protected boolean isTypeSignatureInUse(InputStream inputStream, byte[] bytes) {
		try {
			byte[] data = new byte[inputStream.available()];
			IOUtils.read(inputStream, data);
			int index = ArrayUtils.indexOf(data, bytes);
			return index != -1;
		} catch (IOException ioex) {
			ioex.printStackTrace();
		}
		return false;
	}
	protected byte[] getTypeSignatureBytes(Class type) {
		String name = 'L' + type.getName().replace('.', '/') + ';';
		return name.getBytes();
	}

```
使用：
```

private final byte[] testAnnotationBytes=getTypeSignatureBytes(TestAnn.class);
if (isTypeSignatureInUse(inputStream, testAnnotationBytes) == false) {
		.....
}

```
附工具类:
```

public class ArrayUtils {
	/**
	 * Finds the first occurrence in an array.
	 */
	public static int indexOf(byte[] array, byte[] sub) {
		return indexOf(array, sub, 0, array.length);
	}
	public static boolean contains(byte[] array, byte[] sub) {
		return indexOf(array, sub) != -1;
	}
	/**
	 * Finds the first occurrence in an array from specified given position.
	 */
	public static int indexOf(byte[] array, byte[] sub, int startIndex) {
		return indexOf(array, sub, startIndex, array.length);
	}

	/**
	 * Finds the first occurrence in an array from specified given position and upto given length.
	 */
	public static int indexOf(byte[] array, byte[] sub, int startIndex, int endIndex) {
		int sublen = sub.length;
		if (sublen == 0) {
			return startIndex;
		}
		int total = endIndex - sublen + 1;
		byte c = sub[0];
	mainloop:
		for (int i = startIndex; i < total; i++) {
			if (array[i] != c) {
				continue;
			}
			int j = 1;
			int k = i + 1;
			while (j < sublen) {
				if (sub[j] != array[k]) {
					continue mainloop;
				}
				j++; k++;
			}
			return i;
		}
		return -1;
	}
}

```

##  全局类扫描工具开发过程 
定义匹配classpath中jar文件的规则:
```

public class Rule<R> {
	private R rule; //规则
	private boolean included;//是否包含，true规则应该被包含，false规则不应该被包含
	public Rule(){
	}
	/**
	 * @param rule
	 * @param included
	 */
	public Rule(R rule, boolean included) {
		super();
		this.rule = rule;
		this.included = included;
	}
	
	/**
	 * @return the rule
	 */
	public R getRule() {
		return rule;
	}
	/**
	 * @param rule the rule to set
	 */
	public void setRule(R rule) {
		this.rule = rule;
	}
	/**
	 * @return the included
	 */
	public boolean isIncluded() {
		return included;
	}
	/**
	 * @param included the included to set
	 */
	public void setIncluded(boolean included) {
		this.included = included;
	}
}

```
定义扫描jar匹配规则集的接口：
```

public interface ScanRules<R> {
	/**
	 * Adds include rule.
	 */
	public void include(R rule);

	/**
	 * Adds exclude rule.
	 */
	public void exclude(R rule);

	/**
	 * Adds a rule. Duplicates are not allowed and will be ignored.
	 */
	public void addRule(R rule, boolean include);
	
	/**判断是否包含此规则
	 * @param rule
	 * @return
	 */
	public boolean accept(R rule);
}


```
创建规则管理集的实现：
```

public class InCludeExcludeRule implements ScanRules<String>{
	protected List<Rule<String>> rules=new ArrayList<>();
	private int includesCount=0;
	private int excludesCount=0;
	/* (non-Javadoc)
	 * @see com.css.sword.cms.service.ClassScanRule#include(java.lang.Object)
	 */
	@Override
	public void include(String rule) {
		rules.add(new Rule<String>(rule,true));
		includesCount++;
	}
	/* (non-Javadoc)
	 * @see com.css.sword.cms.service.ClassScanRule#exclude(java.lang.Object)
	 */
	@Override
	public void exclude(String rule) {
		// TODO Auto-generated method stub
		rules.add(new Rule<String>(rule,false));
		excludesCount++;
	}
	/* (non-Javadoc)
	 * @see com.css.sword.cms.service.ClassScanRule#addRule(java.lang.Object, boolean)
	 */
	@Override
	public void addRule(String rule, boolean include) {
		// TODO Auto-generated method stub
		rules.add(new Rule<String>(rule,include));
		if(include){
			includesCount++;
		}else{
			excludesCount++;
		}
	}
	/**
	 * @return the includesCount
	 */
	public int getIncludesCount() {
		return includesCount;
	}
	/**
	 * @return the excludesCount
	 */
	public int getExcludesCount() {
		return excludesCount;
	}
	/* (non-Javadoc)
	 * @see com.css.sword.cms.service.ClassScanRules#isInclude(java.lang.Object)
	 */
	@Override
	public boolean accept(String filename) {
		boolean isInclude=true;
		for(Rule<String> r:rules){
			boolean match=FilenameUtils.wildcardMatch(filename, r.getRule());
			if(match&&!r.isIncluded()){
				isInclude= false;
				break;
			}
		}
		return isInclude;
	}
}

```
定义ClassScanner抽象类:
```

public abstract class ClassScanner {
	private static final String CLASS_FILE_EXT = ".class";
	private static final String JAR_FILE_EXT = ".jar";
	/**
	 * Array of system jars that are excluded from the search.
	 * By default, these paths are common for linux, windows and mac.
	 */
	protected static String[] systemJars = new String[] {
			"*/jre/lib/*.jar",
			"*/jre/lib/ext/*.jar",
			"*/Java/Extensions/*.jar",
			"*/Classes/*.jar"
	};
	protected final InCludeExcludeRule rulesJars=createRulesJars();
	
	/**
	 * 排除扫描系统jars
	 * @return
	 */
	private InCludeExcludeRule createRulesJars(){
		InCludeExcludeRule rulesJarsTmp=new InCludeExcludeRule();
		for (String systemJar : systemJars) {
			rulesJarsTmp.exclude(systemJar);
		}
		return rulesJarsTmp;
	}
	/**
	 * Returns system jars.
	 */
	public static String[] getSystemJars() {
		return systemJars;
	}
	/**
	 * Specify excluded jars.
	 */
	public void setExcludedJars(String... excludedJars) {
		for (String excludedJar : excludedJars) {
			rulesJars.exclude(excludedJar);
		}
	}
	/**
	 * Specify included jars.
	 */
	public void setIncludedJars(String... includedJars) {
		for (String includedJar : includedJars) {
			rulesJars.include(includedJar);
		}
	}
	/**
	 * Scans several URLs. If (#ignoreExceptions} is set, exceptions
	 * per one URL will be ignored and loops continues. 
	 */
	protected void scanUrls(URL... urls) {
		for (URL path : urls) {
			scanUrl(path);
		}
	}
	
	/**
	 * Scans single URL for classes and jar files.
	 * Callback {@link #onEntry(EntryData)} is called on
	 * each class name.
	 */
	protected void scanUrl(URL url) {
		File file = FileUtil.toFile(url);
		scanPath(file);
	}


	protected void scanPaths(File... paths) {
		for (File path : paths) {
			scanPath(path);
		}
	}

	protected void scanPaths(String... paths) {
		for (String path : paths) {
			scanPath(path);
		}
	}
	
	protected void scanPath(String path) {
		scanPath(new File(path));
	}
	/**
	 * Scans single path.
	 */
	protected void scanPath(File file) {
		String path = file.getAbsolutePath();

		if (StringUtil.endsWithIgnoreCase(path, JAR_FILE_EXT) == true) {

			if (rulesJars.accept(path) == false) {
				return;
			}
			scanJarFile(file);
		} else if (file.isDirectory() == true) {
			scanClassPath(file);
		}
	}
	/**
	 * Scans classes inside single JAR archive. Archive is scanned as a zip file.
	 * @see #onEntry(EntryData)
	 */
	protected void scanJarFile(File file) {
		ZipFile zipFile;
		try {
			zipFile = new ZipFile(file);
		} catch (IOException ioex) {
			ioex.printStackTrace();
			return;
		}
		Enumeration entries = zipFile.entries();
		while (entries.hasMoreElements()) {
			ZipEntry zipEntry = (ZipEntry) entries.nextElement();
			String zipEntryName = zipEntry.getName();
			try {
				if (StringUtil.endsWithIgnoreCase(zipEntryName, CLASS_FILE_EXT)) {
					String entryName = prepareEntryName(zipEntryName, true);
					EntryData entryData = new EntryData(entryName, zipFile, zipEntry);
					try {
						scanEntry(entryData);
					} finally {
						entryData.closeInputStreamIfOpen();
					}
				} 
			} catch (RuntimeException rex) {
				IOUtils.closeQuietly(zipFile);
			}
		}
		IOUtils.closeQuietly(zipFile);
	}

	/**
	 * Scans single classpath directory.
	 * @see #onEntry(EntryData)
	 */
	protected void scanClassPath(File root) {
		String rootPath = root.getAbsolutePath();
		if (rootPath.endsWith(File.separator) == false) {
			rootPath += File.separatorChar;
		}

		Collection<File> fileList=FileUtils.listFiles(new File(rootPath),new String[]{"class"},true);
		for(File file:fileList) {
			String filePath = file.getAbsolutePath();
			try {
				if (StringUtil.endsWithIgnoreCase(filePath, CLASS_FILE_EXT)) {
					scanClassFile(filePath, rootPath, file, true);
				} else{
					scanClassFile(filePath, rootPath, file, false);
				}
			} catch (RuntimeException rex) {
				rex.printStackTrace();
			}
		}
	}

	protected void scanClassFile(String filePath, String rootPath, File file, boolean isClass) {
		if (StringUtil.startsWithIgnoreCase(filePath, rootPath) == true) {
			String entryName = prepareEntryName(filePath.substring(rootPath.length()), isClass);
			EntryData entryData = new EntryData(entryName, file);
			try {
				scanEntry(entryData);
			} finally {
				entryData.closeInputStreamIfOpen();
			}
		}
	}

	/**
	 * Prepares resource and class names. For classes, it strips '.class' from the end and converts
	 * all (back)slashes to dots. For resources, it replaces all backslashes to slashes.
	 */
	protected String prepareEntryName(String name, boolean isClass) {
		String entryName = name;
		if (isClass) {
			entryName = name.substring(0, name.length() - 6);		// 6 == ".class".length()
			entryName = StringUtil.replaceChar(entryName, '/', '.');
			entryName = StringUtil.replaceChar(entryName, '\\', '.');
		} else {
			entryName = '/' + StringUtil.replaceChar(entryName, '\\', '/');
		}
		return entryName;
	}
	// ---------------------------------------------------------------- callback

	/**
	 * Called during classpath scanning when class or resource is found.
	 * <ul>
	 * <li>Class name is java-alike class name (pk1.pk2.class) that may be immediately used
	 * for dynamic loading.</li>
	 * <li>Resource name starts with '\' and represents either jar path (\pk1/pk2/res) or relative file path (\pk1\pk2\res).</li>
	 * </ul>
	 *
     * InputStream is provided by InputStreamProvider and opened lazy.
	 * Once opened, input stream doesn't have to be closed - this is done by this class anyway.
	 */
	protected abstract void onEntry(EntryData entryData) throws Exception;
	
	/**
	 * If entry name is {@link #acceptEntry(String) accepted} invokes {@link #onEntry(EntryData)} a callback}.
	 */
	protected void scanEntry(EntryData entryData) {
		if (rulesJars.accept(entryData.getName()) == false) {
			return;
		}
		try {
			onEntry(entryData);
		} catch (Exception ex) {
			ex.printStackTrace();
		}
	}

	// ---------------------------------------------------------------- utilities

	/**
	 * Returns type signature bytes used for searching in class file.
	 */
	protected byte[] getTypeSignatureBytes(Class type) {
		String name = 'L' + type.getName().replace('.', '/') + ';';
		return name.getBytes();
	}

	/**
	 * Returns true if class contains {@link #getTypeSignatureBytes(Class) type signature}.
	 * It searches the class content for bytecode signature. This is the fastest way of finding if come
	 * class uses some type. Please note that if signature exists it still doesn't means that class uses
	 * it in expected way, therefore, class should be loaded to complete the scan.
	 */
	protected boolean isTypeSignatureInUse(InputStream inputStream, byte[] bytes) {
		try {
			byte[] data = new byte[inputStream.available()];
			IOUtils.read(inputStream, data);
			int index = ArrayUtils.indexOf(data, bytes);
			return index != -1;
		} catch (IOException ioex) {
			ioex.printStackTrace();
		}
		return false;
	}

	// ---------------------------------------------------------------- class loading

	/**
	 * Loads class by its name. If {@link #ignoreException} is set,
	 * no exception is thrown, but null is returned.
	 */
	protected Class loadClass(String className) throws ClassNotFoundException {
		try {
			return ClassUtils.getDefaultClassLoader().loadClass(className);
		} catch (ClassNotFoundException cnfex) {
			cnfex.printStackTrace();
		} catch (Error error) {
			error.printStackTrace();
		}
		return null;
	}
}


```
```

public class DefaultClassScanner  extends ClassScanner{
	
	private final byte[] interceptAnnotationBytes=getTypeSignatureBytes(TestAnn.class);
	private final List<Class<?>> clazzList=new ArrayList<>();
	/**
	 * Scans all classes and registers only those annotated with {@link jodd.petite.meta.PetiteBean}.
	 * Because of performance purposes, classes are not dynamically loaded; instead, their
	 * file content is examined. 
	 */
	@Override
	protected void onEntry(EntryData entryData) {
		String entryName = entryData.getName();
		InputStream inputStream = entryData.openInputStream();
		//判断类中是否存在注解Intercept
		if (isTypeSignatureInUse(inputStream, interceptAnnotationBytes) == false) {
			return;
		}
		Class<?> beanClass=null;

		try {
			beanClass = loadClass(entryName);
		} catch (ClassNotFoundException cnfex) {
			cnfex.printStackTrace();
		}

		if (beanClass == null) {
			return;
		}

		clazzList.add(beanClass);
	}

}


```
EntryData:
```

public  class EntryData {

	private final File file;
	private final ZipFile zipFile;
	private final ZipEntry zipEntry;
	private final String name;

	EntryData(String name, ZipFile zipFile, ZipEntry zipEntry) {
		this.name = name;
		this.zipFile = zipFile;
		this.zipEntry = zipEntry;
		this.file = null;
		inputStream = null;
	}
	EntryData(String name, File file) {
		this.name = name;
		this.file = file;
		this.zipEntry = null;
		this.zipFile = null;
		inputStream = null;
	}

	private InputStream inputStream;

	/**
	 * Returns entry name.
	 */
	public String getName() {
		return name;
	}

	/**
	 * Returns true if archive.
	 */
	public boolean isArchive() {
		return zipFile != null;
	}

	/**
	 * Returns archive name or null if entry is not inside archived file.
	 */
	public String getArchiveName() {
		if (zipFile != null) {
			return zipFile.getName(); 
		}
		return null;
	}

	/**
	 * Opens zip entry or plain file and returns its input stream.
	 */
	public InputStream openInputStream() {
		if (zipFile != null) {
			try {
				inputStream = zipFile.getInputStream(zipEntry);
				return inputStream;
			} catch (IOException ioex) {
				ioex.printStackTrace();
			}
		}
		try {
			inputStream = new FileInputStream(file);
			return inputStream;
		} catch (FileNotFoundException fnfex) {
			fnfex.printStackTrace();
		}
		return null;
	}

	/**
	 * Closes input stream if opened.
	 */
	void closeInputStreamIfOpen() {
		if (inputStream == null) {
			return;
		}
		IOUtils.closeQuietly(inputStream);
		inputStream = null;
	}

	@Override
	public String toString() {
		return "EntryData{" + name + '\'' +'}';
	}
}

```
ClassUtils
```

public class ClassUtils {
	private static final String[] MANIFESTS = {"Manifest.mf", "manifest.mf", "MANIFEST.MF"};
	public static ClassLoader getDefaultClassLoader() {
		ClassLoader cl = null;
		try {
			cl = Thread.currentThread().getContextClassLoader();
		} catch (Throwable localThrowable) {
		}
		if(cl==null){
			cl=getCallerClass(2).getClassLoader();
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
	public static File[] getDefaultClasspath(ClassLoader classLoader) {
		Set<File> classpaths = new TreeSet<>();

		while (classLoader != null) {
			if (classLoader instanceof URLClassLoader) {
				URL[] urls = ((URLClassLoader) classLoader).getURLs();
				for (URL u : urls) {
					File f = FileUtil.toFile(u);
					if ((f != null) && f.exists()) {
						try {
							f = f.getCanonicalFile();

							boolean newElement = classpaths.add(f);
							if (newElement) {
								addInnerClasspathItems(classpaths, f);
							}
						} catch (IOException ignore) {
						}
					}
				}
			}
			classLoader = classLoader.getParent();
		}
		return classpaths.toArray(new File[classpaths.size()]);
	}
	private static void addInnerClasspathItems(Set<File> classpaths, File item) {

		Manifest manifest = getClasspathItemManifest(item);
		if (manifest == null) {
			return;
		}

		Attributes attributes = manifest.getMainAttributes();
		if (attributes == null) {
			return;
		}

		String s = attributes.getValue(Attributes.Name.CLASS_PATH);
		if (s == null) {
			return;
		}

		String base = getClasspathItemBaseDir(item);

		String[] tokens = s.split(" ");
		for (String t : tokens) {
			t=t.trim();
			File file;

			// try file with the base path
			try {
				file = new File(base, t);
				file = file.getCanonicalFile();
				if (file.exists() == false) {
					file = null;
				}
			} catch (Exception ignore) {
				file = null;
			}

			if (file == null) {
				// try file with absolute path
				try {
					file = new File(t);
					file = file.getCanonicalFile();
					if (file.exists() == false) {
						file = null;
					}
				} catch (Exception ignore) {
					file = null;
				}
			}

			if (file == null) {
				// try the URL
				try {
					URL url = new URL(t);

					file = new File(url.getFile());
					file = file.getCanonicalFile();
					if (file.exists() == false) {
						file = null;
					}
				} catch (Exception ignore) {
					file = null;
				}
			}

			if (file != null && file.exists() == true) {
				classpaths.add(file);
			}
		}
	}
	
	/**
	 * Returns classpath item manifest or null if not found.
	 */
	public static Manifest getClasspathItemManifest(File classpathItem) {
		Manifest manifest = null;

		if (classpathItem.isFile()) {
			FileInputStream fis = null;
			try {
				fis = new FileInputStream(classpathItem);
				JarFile jar = new JarFile(classpathItem);
				manifest = jar.getManifest();
			} catch (IOException ignore) {
			}
			finally {
				IOUtils.closeQuietly(fis);
			}
		} 
		return manifest;
	}

	/**
	 * Returns base folder for classpath item. If item is a (jar) file,
	 * its parent is returned. If item is a directory, its name is returned.
	 */
	public static String getClasspathItemBaseDir(File classpathItem) {
		String base;
		if (classpathItem.isFile()) {
			base = classpathItem.getParent();
		} else {
			base = classpathItem.toString();
		}
		return base;
	}

	/**
	 * Returns default classpath using
	 * {@link #getDefaultClassLoader() default classloader}.
	 */
	public static File[] getDefaultClasspath() {
		return getDefaultClasspath(getDefaultClassLoader());
	}
	
	/**
	 * 获取方法调用者的Class
	 * @param level 方法调用层次，以1开始，一般取2或3
	 * @return
	 */
	public static Class getCallerClass(int level) {
		StackTraceElement[] eles=new Throwable().getStackTrace();
		if(level<=0||level>eles.length){
			level=eles.length;
		}
		String className=eles[level-1].getClassName();
		try {
			return Thread.currentThread().getContextClassLoader().loadClass(className);
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
		
	}

}

```
FileUtil:
```

public class FileUtil {
	private static final String DEFAULT_ENCODING="utf-8";
	/**
	 * Converts file URLs to file. Ignores other schemes and returns null
	 */
	public static File toFile(URL url) {
		String fileName = toFileName(url);
		if (fileName == null) {
			return null;
		}
		return file(fileName);
	}
	/**
	 * Converts file to URL in a correct way.
	 * Returns null in case of error.
	 */
	public static URL toURL(File file) throws MalformedURLException {
		return file.toURI().toURL();
	}

	/**
	 * Converts file URLs to file name. Accepts only URLs with 'file' protocol.
	 * Otherwise, for other schemes returns null
	 */
	public static String toFileName(URL url) {
		if ((url == null) || (url.getProtocol().equals("file") == false)) {
			return null;
		}
		String filename = url.getFile().replace('/', File.separatorChar);

		try {
			return URLDecoder.decode(filename, DEFAULT_ENCODING);
		} catch (UnsupportedEncodingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return filename;
	}
	/**
	 * Simple factory for File objects.
	 */
	private static File file(String fileName) {
		return new File(fileName);
	}
}

```
StringUtil
```

public class StringUtil {
	/**
	 * Tests if this string ends with the specified suffix.
	 *
	 * @param src    String to test
	 * @param subS   suffix
	 *
	 * @return true if the character sequence represented by the argument is
	 *         a suffix of the character sequence represented by this object;
	 *        false otherwise.
	 */
	public static boolean endsWithIgnoreCase(String src, String subS) {
		String sub = subS.toLowerCase();
		int sublen = sub.length();
		int j = 0;
		int i = src.length() - sublen;
		if (i < 0) {
			return false;
		}
		while (j < sublen) {
			char source = Character.toLowerCase(src.charAt(i));
			if (sub.charAt(j) != source) {
				return false;
			}
			j++; i++;
		}
		return true;
	}

	/**
	 * Returns if string ends with provided character.
	 */
	public static boolean endsWithChar(String s, char c) {
		if (s.length() == 0) {
			return false;
		}
		return s.charAt(s.length() - 1) == c;
	}
	/**
	 * Tests if this string starts with the specified prefix with ignored case.
	 *
	 * @param src    source string to test
	 * @param subS   starting substring
	 *
	 * @return true if the character sequence represented by the argument is
	 *         a prefix of the character sequence represented by this string;
	 *         false otherwise.
	 */
	public static boolean startsWithIgnoreCase(String src, String subS) {
		return startsWithIgnoreCase(src, subS, 0);
	}

	/**
	 * Tests if this string starts with the specified prefix with ignored case
	 * and with the specified prefix beginning a specified index.
	 *
	 * @param src        source string to test
	 * @param subS       starting substring
	 * @param startIndex index from where to test
	 *
	 * @return true if the character sequence represented by the argument is
	 *         a prefix of the character sequence represented by this string;
	 *         false otherwise.
	 */
	public static boolean startsWithIgnoreCase(String src, String subS, int startIndex) {
		String sub = subS.toLowerCase();
		int sublen = sub.length();
		if (startIndex + sublen > src.length()) {
			return false;
		}
		int j = 0;
		int i = startIndex;
		while (j < sublen) {
			char source = Character.toLowerCase(src.charAt(i));
			if (sub.charAt(j) != source) {
				return false;
			}
			j++; i++;
		}
		return true;
	}
	/**
	 * Replaces all occurrences of a character in a string.
	 *
	 * @param s      input string
	 * @param sub    character to replace
	 * @param with   character to replace with
	 */
	public static String replaceChar(String s, char sub, char with) {
		int startIndex = s.indexOf(sub);
		if (startIndex == -1) {
			return s;
		}
		char[] str = s.toCharArray();
		for (int i = startIndex; i < str.length; i++) {
			if (str[i] == sub) {
				str[i] = with;
			}
		}
		return new String(str);
	}
}


```