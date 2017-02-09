title: java7变化2 

#  java7变化之NIO.2 
Java7的新IO API(即NIO.2) java.nio
基础工具类
  * Path-完全替代java.io.File的基于文件和目录的基础类、处理相对路径。以及获取路径信息。Path除了便是传统文件系统外，还可以表示zip或jar这样的文件系统
  * Paths-用于创建路径的工具类。如get(String first,String... mors)、get(URI uri)
  * FileSystem。与文件系统交互类
  * FileSystems。工具类。比如FileSystems.getDefault()
  * Files-文件操作(新建，删除，移动，复制)等工具类，快速地流创建方法(Files.newBufferedReader(...)).文件属性支持Files.readAttributes().目录递归遍历Files.walkFileTree。配合FileVisitor接口(及其实现SimpleFileVisitor)、FileVisitResult枚举
  * FileAttribute文件属性、PosixPermissions。
  * WatchService-监视文件或目录变化.

**异步IO类**
java.nio.channels.SeekableByteChannel接口及其实现java.nio.channels.FileChannel.寻址能力.如FileChannel.read(ByteBuffer,startPos)
NetworkChannel
  * AsynchronousFileChannel
  * AsynchronousSocketChannel
  * AsynchronousServerSocketChannel
使用异步IO API时主要有两种形式：Future,Callback

##  Path类 
Path-完全替代java.io.File的基于文件和目录的基础类、处理相对路径。以及获取路径信息。Path除了便是传统文件系统外，还可以表示zip或jar这样的文件系统
Path不一定代表真实的文件或目录。这意味着你可以随心所欲地操作Path。
**创建Path :** 
Path path=Paths.get("/usr/bin/aa.zip")或者
Path path=FileSystem.getDefault().getPath("/usr/bin/aa.zip")

我们还可以从Path中**获取信息**，比如其父目录、文件名等
  * path.getFileName() 如aa.zip
  * path.getNameCount()如3
  * path.getPath()如/usr/bin
  * path.getRoot()如/

**格式化路径**
  * Path path=Paths.get("../aa.zip").normalize()
  * Path path=Paths.get("../aa.zip").toAbsolutePath()
  * Path path=Paths.get("../aa.link").toRealPath()将符合链接指向真实文件路径

java.nio.file.Path与java.io.File相互转化:
  * file.toPath()
  * path.toFile()

##  处理目录和目录树 
  * java.nio.file.DirectoryStream<T>可以用于处理目录(只能用于处理一级)并可以进行glob模式匹配。通过Files.newDirectoryStram(Path,"*.java")获取
  * Files.walkFileTree()可以用于处理目录树
在目录中查找文件
```

import java.nio.file.DirectoryStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
public class Listing_2_2 {
  public static void main(String[] args) {
    Path dir = Paths.get("C:\\workspace\\java7developer");
    try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir,
        "*.properties")) {
      for (Path entry : stream) {
        System.out.println(entry.getFileName());
      }
    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
  }
}

```
**遍历目录树**
Files.walkFileTree(Path dir,FileVisitor<? super Path> visitor)
FileVisitor接口有个实现SimpleFileVisitor
```

import java.nio.file.FileVisitResult;
import java.nio.file.SimpleFileVisitor;
import java.nio.file.attribute.BasicFileAttributes;
public class Listing_2_3 {
  public static void main(String[] args) throws IOException {
    Path startingDir = Paths
        .get("/Users/karianna/Documents/workspace/java7developer_code_trunk");
    Files.walkFileTree(startingDir, new FindJavaVisitor());
  }
  private static class FindJavaVisitor extends SimpleFileVisitor<Path> {
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
      if (file.toString().endsWith(".java")) {
        System.out.println(file.getFileName());
      }
      return FileVisitResult.CONTINUE;
    }
  }
}

```
##  创建、删除、移动、复制文件 
**创建**
```

Path target=Paths.get("D:\\backup.11.txt")
Path file=Files.createFile(target)
//OSIX文件系统上检测操作权限:
Path target=Paths.get("D:\\backup.11.txt")
Set<PosixPermission> perms=PosixFilePermissions.fromString("rw-rw-rw-")
FileAttribute<Set<PosixPermission>> attr=PosixFilePermissions.asFileAttribute(perms)
Files.create(target,attr)

```

**复制和移动**
```

Path src=Paths.get("D:\\11.java")
Path dest=Paths.get("D:\\22.java")
Files.copy(src,dest)
可以附带一些设置选项位于java.nio.file.StandardCopyOption
Files.copy(src,dest,StandardCopyOption.REPLACE_EXISTING)
选项包括ATOMIC_MOVE、COPY_ATTRIBUTES、REPLACE_EXISTING
Files.move(src,dest,REPLACE_EXISTING，COPY_ATTRIBUTES)

```

**删除**
Files.delete(Path)

##  文件的属性 
BasicFileAttributes接口
FileAttribute接口
获取属性集static Map<String,Object> readAttributes(Path path, String attributes, LinkOption... options)
获取单个属性
```

  public static void main(String[] args) {
    try {
      Path zip = Paths.get("/usr/bin/zip");
      System.out.println(zip.toAbsolutePath().toString());
      System.out.println(Files.getLastModifiedTime(zip));
      System.out.println(Files.size(zip));
      System.out.println(Files.isSymbolicLink(zip));
      System.out.println(Files.isDirectory(zip));
      System.out.println(Files.readAttributes(zip, "*"));
    } catch (IOException ex) {
      System.out.println("Exception" + ex.getMessage());
    }
  }

```
**Java7对文件属性的支持**
```

import java.nio.file.attribute.PosixFileAttributes;
import java.nio.file.attribute.PosixFilePermission;
import java.nio.file.attribute.PosixFilePermissions;
import static java.nio.file.attribute.PosixFilePermission.*;
public class Listing_2_5 {

  public static void main(String[] args) {
    try {
      Path profile = Paths.get("/user/Admin/.profile");

      PosixFileAttributes attrs = Files.readAttributes(profile,
          PosixFileAttributes.class);

      Set<PosixFilePermission> posixPermissions = attrs.permissions();
      posixPermissions.clear();

      String owner = attrs.owner().getName();
      String perms = PosixFilePermissions.toString(posixPermissions);
      System.out.format("%s %s%n", owner, perms);

      posixPermissions.add(OWNER_READ);
      posixPermissions.add(GROUP_READ);
      posixPermissions.add(OWNER_READ);
      posixPermissions.add(OWNER_WRITE);
      Files.setPosixFilePermissions(profile, posixPermissions);

    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
  }
}

```

##  快速读写数据、打开文件流 
```

Path logFile=Paths.get("D:\\aaa.log")
Path logFile2=Paths.get("D:\\aaa.log")
try(BufferedReader reader=Files.newBufferedReader(logFile,StandardCharsets.UTF_8);
	BufferedWriter writer=Files.newBufferedWriter(logFile2,StandardCharsets.UTF_8,StandardOpenOption.WRITE)
	){

}

```
StandardOpenOption常用的有如下几项:
WRITE、READ、APPEND、CREATE

**简化读写：**
  * Files.readAllLines(logFile,StandardCharsets.UTF_8)
  * Files.readALlBytes(logFile)
##  检测文件或目录的变化 
java.nio.file.WatchService
```

import static java.nio.file.StandardWatchEventKinds.*;
import java.nio.file.WatchEvent;
import java.nio.file.WatchKey;
import java.nio.file.WatchService;
public class Listing_2_7 {
  private static boolean shutdown = false;
  public static void main(String[] args) {
    try {
      WatchService watcher = FileSystems.getDefault().newWatchService();
      Path dir = FileSystems.getDefault().getPath("/usr/karianna");
      //注册watcher服务，并附带StandardWatchEventKinds选项:ENTRY_CREATE、ENTRY_DELETE、ENTRY_MODIFY、OVERFLOW
      WatchKey key = dir.register(watcher, ENTRY_MODIFY);
      while (!shutdown) {
        key = watcher.take();//获取key
        for (WatchEvent<?> event : key.pollEvents()) { //迭代key事件
          if (event.kind() == ENTRY_MODIFY) {
            System.out.println("Home dir changed!");
          }
        }
        key.reset();//重置key
      }
    } catch (IOException | InterruptedException e) {
      System.out.println(e.getMessage());
    }
  }
}

```
##  异步IO类 
  * AsynchronousFileChannel
  * AsynchronousSocketChannel
  * AsynchronousServerSocketChannel
使用异步IO API时主要有两种形式：Future,Callback。异步IO不会阻塞主线程。AsynchronousFileChannel底层会关联线程池来执行IO操作
Future方式
```

import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousFileChannel;
import java.util.concurrent.Future;
public class Listing_2_8 {
  public static void main(String[] args) {
    try {
      Path file = Paths.get("/usr/karianna/foobar.txt");
      AsynchronousFileChannel channel = AsynchronousFileChannel.open(file);
      ByteBuffer buffer = ByteBuffer.allocate(100_000);
      Future<Integer> result = channel.read(buffer, 0);
      while (!result.isDone()) {
        ProfitCalculator.calculateTax();
      }
      Integer bytesRead = result.get();
      System.out.println("Bytes read [" + bytesRead + "]");
    } catch (IOException | ExecutionException | InterruptedException e) {
      System.out.println(e.getMessage());
    }
  }
  private static class ProfitCalculator {
    public ProfitCalculator() {
    }
    public static void calculateTax() {
    }
  }
}

```
**回调式：**
java.nio.channels.CompletionHandler<V,A>。当异步IO结束后，该接口中的某个方法会被调用。V是结果类型，A是结果的附着对象。
```

public class Listing_2_9 {

  public static void main(String[] args) {
    try {
      Path file = Paths.get("/usr/karianna/foobar.txt");
      AsynchronousFileChannel channel = AsynchronousFileChannel.open(file);

      ByteBuffer buffer = ByteBuffer.allocate(100_000);

      channel.read(buffer, 0, buffer,
          new CompletionHandler<Integer, ByteBuffer>() {

            public void completed(Integer result, ByteBuffer attachment) {
              System.out.println("Bytes read [" + result + "]");
            }

            public void failed(Throwable exception, ByteBuffer attachment) {
              System.out.println(exception.getMessage());
            }
          });
    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
  }
}

```
##  Socket和Channel的整合 
NetworkChannel
```

public class Listing_2_10 {

  public static void main(String[] args) {
    SelectorProvider provider = SelectorProvider.provider();
    try {
      NetworkChannel socketChannel = provider.openSocketChannel();
      SocketAddress address = new InetSocketAddress(3080);
      socketChannel = socketChannel.bind(address);

      Set<SocketOption<?>> socketOptions = socketChannel.supportedOptions();

      System.out.println(socketOptions.toString());
      socketChannel.setOption(StandardSocketOptions.IP_TOS, 3);
      Boolean keepAlive = socketChannel
          .getOption(StandardSocketOptions.SO_KEEPALIVE);
    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
  }
}


```
MulticastChannel及其实现DatagramChannel:
```

public class Listing_2_11 {

  public static void main(String[] args) {
    try {
      NetworkInterface networkInterface = NetworkInterface.getByName("net1"); //选择网络接口

      DatagramChannel dc = DatagramChannel.open(StandardProtocolFamily.INET);

      dc.setOption(StandardSocketOptions.SO_REUSEADDR, true);
      dc.bind(new InetSocketAddress(8080));
      dc.setOption(StandardSocketOptions.IP_MULTICAST_IF, networkInterface);//将Channel设置为多播

      InetAddress group = InetAddress.getByName("180.90.4.12");  //多播组地址
      MembershipKey key = dc.join(group, networkInterface); //加入多播组
    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
  }
}


```