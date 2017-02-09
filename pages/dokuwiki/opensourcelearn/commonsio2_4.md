title: commonsio2_4 

#  apache commons学习系列记录之IO组件version2.4 

**基于commons-IO-2.4版本** 
#  一、org.apache.commons.io 

IOUtils
    简化InputStream,OutputStream,Writer,Reader等的IO操作
   内部自动缓存buffer,友好的关闭流closeQuetly,以LineIterator行迭代器的形式读取文件，流，File与流，String的转换。直接从流到String的转换
   跳过读取等。
DirectoryWalker:  
     DirectoryWolker<T>抽象类，用于层次遍历目录及子目录。同时给它的子类提供hooks钩子方法以便处理相关事件行为。这些钩子方法以handle开头，由DirecotryWalker抽象类的机制去调用，实现者无需关心。
一般与IO组件的FileFilter结合使用，可以指定遍历深度。这些FIleFilter实现位于IO组件的filefilter包中。
由于遍历过程比较耗时，所以这个遍历过程还提供取消功能。
FileUtils:
    包含处理文件与目录的操作，如移动，复制，读取，写入，比较，复制，大小，遍历，删除,生成文件校验码等。
FilenameUtils:便于路径的一致性
SystemUtils:获取卷的剩余容量
LineIterator:行迭代器。一般用于迭代从文件或流中读取的数据。

##  1.IOUtils: 

 closeQuietly - these methods close a stream ignoring nulls and exceptions
  toXxx/read - these methods read data from a stream
  write - these methods write data to a stream
  copy - these methods copy all the data from one stream to another
  contentEquals - these methods compare the content of two streams
有用的Field: DIR_SEPARATOR,LINE_SEPARATOR
static void closeQuietly(Closeable closeable)用于安静地关闭流stream,reader,writer或socket。
copy方法 可以将数据直接从一个流copy到另外一个流，同时还可以指定字符编码。但是需要注意的是这些数据都会直接通过内存，
所以在copy大文件的时候需要特别注意这一点。
copy(Reader input, OutputStream output, Charset/String encoding)
         Copy chars from aReaderto bytes on anOutputStreamusing the specified character encoding, and
calling flush.
copy大文件可以是用copyLarge方法，它可以指定缓存char[],byte[]的大小,指定拷贝的范围，长度等。
copyLarge(InputStream input, OutputStream output,
        byte[] buffer)
以行为单位迭代读取数据：数据来源-流,举例：
static LineIterator lineIterator(InputStream input, String encoding)会得到LineIterator迭代器来进行迭代，
不过最后要记得手动关闭这个迭代器以释放资源。
 LineIterator it = IOUtils.lineIterator(stream, charset);
while (it.hasNext()) {
  String line = it.nextLine();
  /// do something with line
}
} finally {
IOUtils.closeQuietly(stream);
}
以行为单位读取得带列表而非迭代器。readLines方法可以得到行列表。
跳过读取：skip.
skip(Reader input,long toSkip)
写入write：类似
write(byte[]/char[]/String/Collection, Writer/OutputStream)可以指定编码，以及行结尾符。
与其他流或者形式的转换：
toBufferedInputStream, toByteArray, toCharArray, toInputStream, toString
IOUtils使用举例：
传统方式：
```

InputStream in = new URL( "http://commons.apache.org" ).openStream();
try {
  InputStreamReader inR = new InputStreamReader( in );
  BufferedReader buf = new BufferedReader( inR );
  String line;
  while ( ( line = buf.readLine() ) != null ) {
    System.out.println( line );
  }
} finally {
  in.close();
}
使用IOUtils方式：
InputStream in = new URL( "http://commons.apache.org" ).openStream();
try {
  System.out.println( IOUtils.toString( in ) );
} finally {
  IOUtils.closeQuietly(in);
}

```
从上面可以看到使用IOUtils的优势了。
##  2. DirectoryWalker<T>抽象类 
```

public class FileCleaner extends DirectoryWalker {
   public FileCleaner() {
     super();
   }
   public List clean(File startDirectory) {
     List results = new ArrayList();
     walk(startDirectory, results);// startDirecotry代表要遍历的目录，results代表想要保存的结果集。
     return results;
   }
   protected boolean handleDirectory(File directory, int depth, Collection results) {
     // delete svn directories and then skip
     if (".svn".equals(directory.getName())) {
       directory.delete();
       return false;
     } else {
       return true;
     }
   }
   protected void handleFile(File file, int depth, Collection results) {
     // delete file and add to list of deleted
     file.delete();
     results.add(file);
   }
 }

```
选择哪些目录和文件来进行处理对于WalkDirectory来说很重要。我们可以通过三种方式来确定。  
第一种遍历所有那么就无需使用FileFilter
第二种指定针对目录和文件的单个Filter。这个Filter对于所有文件和目录都起作用。
第三种方式为分别为目录和文件制定各自的FileFilter.推荐。
第二种
```

public class FooDirectoryWalker extends DirectoryWalker {
   public FooDirectoryWalker(FileFilter filter) {
     super(filter, -1);//其中-1的位置代表深度，如果<0则代表没有限制。
   }
 }
示例：创建一个FileFilter:
创建一个过滤隐藏目录的Filter
// Build up the filters and create the walker
   // Create a filter for Non-hidden directories
   IOFileFilter fooDirFilter =
       FileFilterUtils.andFileFilter(FileFilterUtils.directoryFileFilter,
                                     HiddenFileFilter.VISIBLE);
创建一个过滤.txt结尾文件的FIlter
// Create a filter for Files ending in ".txt"
   IOFileFilter fooFileFilter =
       FileFilterUtils.andFileFilter(FileFilterUtils.fileFileFilter,
                                     FileFilterUtils.suffixFileFilter(".txt"));
组合者两个Filter为一个FIlter
 // Combine the directory and file filters using an OR condition
   java.io.FileFilter fooFilter =
       FileFilterUtils.orFileFilter(fooDirFilter, fooFileFilter);
使用这个组合FileFilter来构造WalkDirectory
  // Use the filter to construct a DirectoryWalker implementation
   FooDirectoryWalker walker = new FooDirectoryWalker(fooFilter);
第三种方式为分别为目录和文件制定各自的FileFilter.推荐。
For example, if you wanted all directories which are not hidden and files which end in ".txt":
 public class FooDirectoryWalker extends DirectoryWalker {
   public FooDirectoryWalker(IOFileFilter dirFilter, IOFileFilter fileFilter) {
     super(dirFilter, fileFilter, -1);
   } 
 }
 // Use the filters to construct the walker
 FooDirectoryWalker walker = new FooDirectoryWalker(
   HiddenFileFilter.VISIBLE,
   FileFilterUtils.suffixFileFilter(".txt"),
 );

```
可取消的遍历进程(Cancellation)：由于遍历需要耗时，我们需要定义其为可以取消的。子类必须实现这一点。
原理为：
在子类内部抛出DirectoryWalker.CancelException，这个异常会被walk()方法所捕获，然后walk()方法会触发调用handleCancelled()来处理取消任务，这个方法供子类定制。

控制子类在内部抛出DirectoryWalker.CancelException异常的方式有两种：
一中是外部/多线程方式。不过需要配和覆盖
一中是内部直接抛出DirectoryWalker.CancelException。
protected  boolean handleIsCancelled(File file, int depth, Collection<T> results)
方法，这个方法返回true则它会抛出异常告诉walk()方法来终止遍历。
对外则提供一个用于控制这个返回值标志的方法cancel()以公共外届调用。。
```

public class FooDirectoryWalker extends DirectoryWalker {
   private volatile boolean cancelled = false;//注意多线程环境下flag的线程安全性，此处设置为volatile，但是要注意者也不是很安全，但对于目前足够了。
   public void cancel() {  //对外提供cancel方法用于设置flag，
       cancelled = true;
   }
   protected boolean handleIsCancelled(File file, int depth, Collection results) {  //关键需要覆盖这个方法，返回值确定是否立即stop这个任务。
       return cancelled;
   }
   protected void handleCancelled(File startDirectory, Collection results, CancelException cancel) {//这是确定停止后最后调用的用于处理停止的方法。。
       // implement processing required when a cancellation occurs
   }
 }
内部方式示例：通过在内部抛出DirectoryWalker.CancelException控制终止，这个Exception会被walk()方法捕获。
public class BarDirectoryWalker extends DirectoryWalker {
   protected boolean handleDirectory(File directory, int depth, Collection results) throws IOException {
       // cancel if hidden directory
       if (directory.isHidden()) {
           throw new CancelException(file, depth);
       }
       return true;
   }
   protected void handleFile(File file, int depth, Collection results) throws IOException {
       // cancel if read-only file
       if (!file.canWrite()) {
           throw new CancelException(file, depth);
       }
       results.add(file);
   }
   protected void handleCancelled(File startDirectory, Collection results, CancelException cancel) {
       // implement processing required when a cancellation occurs
   }
 } 

```
##  3.FileUtils 
:包含处理文件与目录的操作，如移动，复制，读取，写入，比较，复制，大小，遍历，删除,生成文件校验码等。
示例；读取文件以行的形式为列表、
File file = new File("/commons/io/project.properties");  
List lines = FileUtils.readLines(file, "UTF-8");
将文件大小以易读的形式转换方法：
static String byteCountToDisplaySize(BigInteger/long size)
如1024000:变成“1M”
生成文件的校验码：配合使用java.util.zip.Checksum接口和java.util.zip.CRC32/Adler32算法。
checksum(File file, Checksum checksum)
checksumCRC32(File file)
复制目录：移动，通过FileFilter过滤
copyDirectory(File srcDir, File destDir, [FileFilter filter])
复制文件：移动，到输出流，到目录
copyFile(File input, OutputStream output)
copyFile(File srcFile, File destFile)
copyFileToDirectory(File srcFile, File destDir)
删除文件和目录
deleteDirectory(File directory)
deleteQuietly(File file)
检查目录是否包含某个文件：
directoryContains(File directory, File child)
创建File对象：
getFile([File directory], String... names)
获取系统临时目录和用户目录路径：
getTempDirectory[Path]()
getUserDirectory[Path]()
遍历目录：
 Iterator<File> iterateFiles(File directory, IOFileFilter fileFilter, IOFileFilter dirFilter)最后的Filter参数决定是否递归遍历
使用TrueFileFilter.TRUE作为第二个filter则可以递归遍历。
文件的读取：获取LineIterator:lineIterator(File file)还可以指定编码
还有移动方法move一系列
还有读取文件为不同的形式：
readFileToByteArray/String ; readLines; 
还有将不同形式的数据写入到文件。
文件与目录大小
sizeOf(File f)； sizeOfDirectory;
##  4.FilenameUtils 
:便于在各种平台间移植的操作工具。
对文件路径的操作。
String filename = "C:/commons/io/../lang/project.xml";  
String normalized = FilenameUtils.normalize(filename);
result is "C:/commons/lang/project.xml"

含义：
 the prefix - C:\
 the path - dev\project\
 the full path - C:\dev\project\
 the name - file.txt
 the base name - file
 the extension - txt

##  6.FileSystemUtils 
取得卷或分区的剩余大小
FileSystemUtils.freeSpaceKb("C:");       // Windows
FileSystemUtils.freeSpaceKb("/volume");  // *nix

##  7.LineIterator: 

An Iterator over the lines in aReader. 注意对其关闭以释放资源。
```

LineIterator it = FileUtils.lineIterator(file, "UTF-8");
try {
 while (it.hasNext()) {
   String line = it.nextLine();
   /// do something with line
 }
} finally {
 LineIterator.closeQuietly(iterator);
}

```

#  纲要： 

org.apache.commons.io.filefilter 文件过滤器
org.apache.commons.io.comparator比较器
LockableFileWriter
org.apache.commons.io.monitor侦测文件或目录的变化情况。

#  二、org.apache.commons.io.filefilter 文件过滤器： 

组合过滤器，预定义的各种过滤器，以及正则表达式过滤，文件大小过滤，扩展名过滤等等。
IOFileFilter接口继承了jdk中的FileFilter,FilenameFilter接口
FileFilter的组合:
    封装多个Filter从而组合成一个新的FileFilter:如：AndFileFilter,OrFileFilter类似
具体的Filter实现类：
CanReadFileFilter:枚举：CAN_READ,CONNOT_READ,READ_ONLY等
CanWriteFileFilter枚举：CAN_WRITE,CANNOT_WRITE
HiddenFileFilter:HIDDEN,VISIBLE
DirectoryFileFIlter枚举：DIRECTORY
EmptyFileFilter枚举：EMPTY,NOT_EMPTY
FalseFileFilter枚举:FALSE,总是返回false的过滤器;还有一个TrueFileFilter
FileFileFilter枚举：FILE

RegexFileFilter:根据正则表达式过滤：
SizeFileFilter:根据文件大小过滤：
PrefixFileFilter,
SuffixFileFilter:根据后缀过滤：


##  1. 过滤器组合AndFileFilter、OrFileFilter 

```

AndFileFilter(IOFileFilter filter1, IOFileFilter filter2)
         Constructs a new file filter that ANDs the result of two other filters.    
AndFileFilter(List<IOFileFilter> fileFilters)
         Constructs a new instance ofAndFileFilterwith the specified list of filters.    
  </code>
##  2.RegexFileFilter:根据正则表达式过滤： 
<code java>
File dir = new File(".");
FileFilter fileFilter = new RegexFileFilter("^.*[tT]est(-\\d+)?\\.java$");
File[] files = dir.listFiles(fileFilter);
for (int i = 0; i < files.length; i++) {
  System.out.println(files[i]);
}

```
##  3.SizeFileFilter:根据文件大小过滤： 
```

File dir = new File(".");
String[] files = dir.list( new SizeFileFilter(1024 * 1024) );
for ( int i = 0; i < files.length; i++ ) {
    System.out.println(files[i]);
}
SizeFileFilter(long size)
          Constructs a new size file filter for files equal to or
larger than a certain size.SizeFileFilter(long size,boolean acceptLarger)
          Constructs a new size file filter for files based on a certain size threshold.
  </code>
对于acceptLarger:acceptLarger- if true, files equal to or larger are accepted,
otherwise smaller ones (but not equal to)
##  4.SuffixFileFilter 
<code java>
SuffixFileFilter:根据后缀过滤：
File dir = new File(".");
String[] files = dir.list( new SuffixFileFilter(".java") );
for (int i = 0; i < files.length; i++) {
    System.out.println(files[i]);
}
SuffixFileFilter(List<String> suffixes);
SuffixFileFilter(String[] suffixes)
SuffixFileFilter(List<String> suffixes, IOCase caseSensitivity)其中IOCase枚举SENSITIVE,INSENSITIVE

```
#  三、org.apache.commons.io.comparator 

文件比较器与过滤器的用法类似。具体看API文档。
#  四其他包中的有用方法： 

LockableFileWriter确保只有一个线程在写入文件。给文件加锁写入，在调用close（）方法时释放。确保多线程写入安全性。除非文件特别重要时采用此方法。
#  五、org.apache.commons.io.monitor侦测文件或目录的变化情况。 


FileAlterationMonitor, FileAlterationObserver和FileAlterationListener接口与FileAlterationListenerAdapter:开启一个线程来实时监听文件的变化。
如文件或目录改变，删除，创建等事件
```

File directory = new File(new File("."), "src");
    FileAlterationObserver observer = new FileAlterationObserver(directory);
    observer.addListener(...);
    observer.addListener(...);

```
创建观察者后有两种选择，1.手动运行观察和停止；2.通过FileAlterationMonitor来启动一个新的线程进行统一的管理。
```

第一种方式如下
 // intialize
    observer.init();
    ...
    // invoke as required
    observer.checkAndNotify();
    ...
    // finished
    observer.finish();
第二种方式：会启动一个新的线程来进行管理所有的观察者。同时可以指定时间间隔。以避免频繁检查。
long interval = ...
    FileAlterationMonitor monitor = new FileAlterationMonitor(interval);
    monitor.addObserver(observer);//注册观察者到监听器
    monitor.start();
    ...
    monitor.stop();

```


更多参考：http://www.importnew.com/13715.html