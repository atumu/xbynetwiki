title: java_io 

#  Java IO 
本文参考：
http://ifeve.com/java-io/
http://www.cnblogs.com/rollenholt/archive/2011/09/11/2173787.html
http://blog.csdn.net/zhangerqing/article/details/8466532
Java IO 是一套Java用来读写数据（输入和输出）的API。大部分程序都要处理一些输入，并由输入产生一些输出。
Java为此提供了java.io包。此处讲解的主要是java.io包内的BIO，而不包含Java5新增的NIO和java7的AIO

Java.io 包主要涉及文件，网络数据流，内存缓冲等的输入输出。
Java对IO的支持是个不断的演变过程，经过了很多的优化，直到JDK1.4以后，才趋于稳定，在JDK1.4中，加入了nio类，解决了很多性能问题，虽然我们有足够的理由不去了解关于Java IO以前的情况，但是为了学好现在的类，我们还是打算去研究下，通过掌握类的优化情况来彻底理解IO的机制！Java IO主要主要在java.io包下，分为四大块近80个类：
1、基于字节操作的I/O接口：InputStream和OutputStream
2、基于字符操作的I/O接口：Writer和Reader
3、基于磁盘操作的I/O接口：File
4、基于网络操作的I/O接口：Socket（不在java.io包下）
影响IO性能的无非就是两大因素：数据的格式及存储的方式
#  几个概念 

**输入和输出 – 数据源和目标媒介**
术语“输入”和“输出”有时候会有一点让人疑惑。输入流可以理解为向内存输入，输出流可以理解为从内存输出
Java的IO包主要关注的是**从原始数据源的读取以及输出原始数据到目标媒介**。以下是最典型的数据源和目标媒介：
  * 文件
  * 管道
  * 网络连接
  * 内存缓存
  * System.in, System.out, System.error(注：Java标准输入、输出、错误输出)
**流**
在Java IO中，流是一个核心的概念。流从概念上来说是一个连续的数据流。你既可以从流中读取数据，也可以往流中写数据。流与数据源或者数据流向的媒介相关联。
在Java IO中流既可以是**字节流**(以字节为单位进行读写)，也可以是**字符流**(以字符为单位进行读写)。

**接口InputStream, OutputStream, Reader 和Writer**
一个程序需要InputStream或者Reader从数据源读取数据，需要OutputStream或者Writer将数据写入到目标媒介中。
InputStream和Reader与数据源相关联，OutputStream和writer与目标媒介相关联。
![](/data/dokuwiki/java/pasted/20150813-031518.png)

###  文件 

在Java应用程序中，文件是一种常用的数据源或者存储数据的媒介。如果你需要在不同端之间读取文件，你可以根据该文件是二进制文件还是文本文件来选择使用` FileInputStream或者FileReader `。
这两个类允许你从文件开始到文件末尾一次读取一个字节或者字符，或者将读取到的字节写入到字节数组或者字符数组。你不必一次性读取整个文件，相反你可以按顺序地读取文件中的字节和字符。
如果你需要跳跃式地读取文件其中的某些部分，可以使用` RandomAccessFile `。RandomAccessFile可以覆盖一个文件的某些部分、或者追加内容到它的末尾、或者删除它的某些内容，当然它也可以从文件的任何位置开始读取文件。
如果你需要在不同端之间进行文件的写入，你可以根据你要写入的数据是二进制型数据还是字符型数据选用` FileOutputStream或者FileWriter。 `
**通过` File `类可以获取文件和目录的信息而不是内容。**

<WRAP center round important 60%>
注意：所有这些读取操作应该在一个**缓存**中进行，由于java io是采用装饰模式设计，所以对这些底层流需要采用` BufferedInputStream,BufferedOutputStream,BufferedReader,BufferedWriter `来进行包装。
注意：一般字节文本输入采用` BufferedReader或者Scanner,输出采用PrintWriter `进行包装。因为他们可以一次读取一整行
</WRAP>
####  关于文件分隔符 
**File.separator**: String fileName="D:"+File.separator+"hello.txt";

###  管道 

Java IO中的管道为运行**在同一个JVM中的两个线程提供了通信的能力**。所以管道也可以作为数据源以及目标媒介。
` 你不能利用管道与不同的JVM中的线程通信(不同的进程)。 `
可以通过Java IO中的` PipedOutputStream和PipedInputStream `创建管道。一个PipedInputStream流应该和一个PipedOutputStream流相关联。一个线程通过PipedOutputStream写入的数据可以被另一个线程通过相关联的PipedInputStream读取出来。
**Java IO管道示例**
![](/data/dokuwiki/java/pasted/20150813-032756.png)
你也可以使用两个管道共有的connect()方法使之相关联。PipedInputStream和PipedOutputStream都拥有一个可以互相关联的` connect() `方法。

**管道和线程**
请记得，当使用两个相关联的管道流时，**务必将它们分配给不同的线程**。**read()方法和write()方法调用时会导致流阻塞**，这意味着如果你尝试在一个线程中同时进行读和写，可能会导致线程死锁。

**管道的替代**
除了管道之外，一个JVM中不同线程之间还有许多通信的方式。实际上，线程在大多数情况下会传递完整的对象信息而非原始的字节数据。但是，如果你需要在线程之间传递字节数据，Java IO的管道是一个不错的选择。
###  网络 
当两个进程之间建立了网络连接之后，他们通信的方式如同操作文件一样：利用InputStream读取数据，利用OutputStream写入数据。换句话来说，Java网络API用来在不同进程之间建立网络连接，而Java IO则用来在建立了连接之后的进程之间交换数据。
具体暂略。。。
###  字节和字符数组 
内容列表
  * 从InputStream或者Reader中读入数组
  * 从OutputStream或者Writer中写数组
在java中常用字节和字符数组在应用中临时存储数据。
###  System.in, System.out, System.err 
` System.in, System.out, System.err `这3个流同样是常见的数据来源和数据流目的地。使用最多的可能是在控制台程序里利用System.out将输出打印到控制台上。
JVM启动的时候通过Java运行时初始化这3个流，所以你不需要初始化它们(尽管你可以在运行时替换掉它们)。
` System.out.println("File opened..."); `
替换默认：
OutputStream output = new FileOutputStream("c:\\data\\system.out.txt");
PrintStream printOut = new PrintStream(output);
System.` setOut `(printOut);
现在所有的System.out都将重定向到”c:\\data\\system.out.txt”文件中。请记住，务必在JVM关闭之前冲刷System.out(译者注：调用flush())，确保System.out把数据输出到了文件中。
###  Scanner 
` Scanner `类其实我们比较常用的是采用Scanner类来进行数据输入，下面来给一个Scanner的例子吧
import **java.util.Scanner**;
```

 
/**
 * Scanner的小例子，从键盘读数据
 * */
public class ScannerDemo{
    public static void main(String[] args){
        Scanner sca = new Scanner(System.in);
        // 读一个整数
        int temp = sca.nextInt();
        System.out.println(temp);
        //读取浮点数
        float flo=sca.nextFloat();
        System.out.println(flo);
        //读取字符
        //...等等的，都是一些太基础的，就不师范了。
    }
}

```
**其实Scanner可以接受任何的输入流**
` sca.nextLine(); `
下面给一个使用Scanner类从文件中读出内容
```

/**
 * Scanner的小例子，从文件中读内容
 * */
public class ScannerDemo{
    public static void main(String[] args){
 
        File file = new File("d:" + File.separator + "hello.txt");
        Scanner sca = null;
        try{
            sca = new Scanner(file);
        }catch(FileNotFoundException e){
            e.printStackTrace();
        }
        String str = sca.next(); //sca.nextLine();
        System.out.println("从文件中读取的内容是：" + str);
    }
}

```

##  流 
Java IO流是既可以从中读取，也可以写入到其中的数据流。流通常会与数据源、数据流向目的地相关联，比如文件、网络等等。
**流和数组不一样，不能通过索引读写数据。**在流中，你也不能像数组那样前后移动读取数据，除非使用RandomAccessFile 处理文件。**流仅仅只是一个连续的数据流。**
` **流中的数据只能够顺序访问。** `
**InputStream**
通常使用输入流中的` read() `方法读取数据。read()方法返回一个**整数**，代表了读取到的字节的**内容**(译者注：0 ~ 255)。当达到流末尾没有更多数据可以读取的时候，read()方法返回**-1**。
```

InputStream input = new FileInputStream("c:\\data\\input-file.txt");
int data = input.read(); 
while(data != -1){
        data = input.read();
}

```
**OutputStream：**
```

OutputStream output = new FileOutputStream("c:\\data\\output-file.txt");
output.write("Hello World".getBytes());
output.close();

```
**组合流**
你可以将流整合起来以便实现更高级的输入和输出操作。比如，一次读取一个字节是很慢的，所以可以从磁盘中一次读取一大块数据，然后从读到的数据块中获取字节。为了实现缓冲，可以把InputStream包装到BufferedInputStream中。代码示例：
` InputStream input = new BufferedInputStream(new FileInputStream("c:\\data\\input-file.txt")); `
缓冲同样可以应用到OutputStream中。你可以实现将大块数据批量地写入到磁盘(或者相应的流)中，这个功能由` BufferedOutputStream `实现。
##  Reader And Writer 
他们被用于读写文本。
Reader类是Java IO中所有Reader的基类。子类包括BufferedReader，PushbackReader，InputStreamReader，StringReader和其他Reader。
```

Reader reader = new FileReader("c:\\data\\myfile.txt");
int data = reader.read();
while(data != -1){
    char dataChar = (char) data;
    data = reader.read();
}

```
**整合Reader与InputStream**
一个` Reader `可以和一个InputStream相结合。如果你有一个InputStream输入流，并且想从其中读取字符，可以把这个InputStream包装到InputStreamReader中。把InputStream传递到InputStreamReader的构造函数中：
` Reader reader = new InputStreamReader(inputStream); `
在构造函数中可以指定解码方式。更多内容请参阅InputStreamReader。
` Writer `类是Java IO中所有Writer的基类。子类包括BufferedWriter和PrintWriter等等。这是一个Java IO Writer的例子：
```

Writer writer = new FileWriter("c:\\data\\file-output.txt"); 
writer.write("Hello World Writer"); 
writer.close();

```
**整合Writer和OutputStream**
` Writer writer = new OutputStreamWriter(outputStream); `
Reader reader = new ` BufferedReader `(new FileReader(...));
Writer writer = new ` BufferedWriter `(new FileWriter(...));

##  具体接口说明与示例 
###  InputStream 
InputStream类是Java IO API中所有输入流的基类。InputStream子类包括FileInputStream，BufferedInputStream，PushbackInputStream等等。
InputStream用于读取基于字节的数据，一次读取一个字节，这是一个InputStream的例子：
```

InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");
int data = inputstream.read();
while(data != -1) { 
    //do something with data...  
    doSomethingWithData(data);   
    data = inputstream.read();
}
inputstream.close();  //流使用完成后记得及时关闭。。

```
从Java7开始，你可以使用“` try-with-resource `”结构确保InputStream在结束使用之后关闭，这里只是一个简单的例子：
```

try( InputStream inputstream = new FileInputStream("file.txt") ) {

    int data = inputstream.read();

    while(data != -1){

        System.out.print((char) data);

        data = inputstream.read();

    }

}

```
当执行线程退出try语句块的时候，InputStream变量会被关闭。
read()
read(byte[])
InputStream包含了2个从InputStream中读取数据并将数据存储到缓冲数组中的read()方法，他们分别是：
int read(byte[])
int read(byte, int offset, int length)
read(byte[])方法会尝试读取与给定字节数组容量一样大的字节数，返回值说明了已经读取过的字节数。
read(byte, int offset, int length)方法同样将数据读取到字节数组中，不同的是，该方法从数组的offset位置开始，并且最多将length个字节写入到数组中。同样地，read(byte, int offset, int length)方法返回一个int变量，告诉你已经有多少字节已经被写入到字节数组中，所以请记得在读取数据前检查上一次调用read(byte, int offset, int length)的返回值。
**这两个方法都会在读取到达到流末尾时返回-1。**
```

InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");

byte[] data = new byte[1024];

int bytesRead = inputstream.read(data);

while(bytesRead != -1) {

    doSomethingWithData(data, bytesRead);

    bytesRead = inputstream.read(data);

}

inputstream.close();

```
###  OutputStream 
OutputStream类是Java IO API中所有输出流的基类。子类包括BufferedOutputStream，FileOutputStream等等。
write(byte)
write(byte[])
OutputStream同样包含了将字节数据中全部或者部分数据写入到输出流中的方法，分别是write(byte[])和write(byte[], int offset, int length)。
write(byte[])把字节数组中所有数据**写入到输出流中**。
write(byte[], int offset, int length)把字节数据中从offset位置开始，length个字节的数据**写入到输出流**。
**flush()**
OutputStream的flush()方法将所有写入到OutputStream的数据冲刷到相应的目标媒介中。
**close()**
当你结束数据写入时，需要关闭OutputStream。通过调用close()可以达到这一点。
```

OutputStream output = null;
try{
    output = new FileOutputStream("c:\\data\\output-text.txt");
    while(hasMoreData()) {
        int data = getMoreData();
        output.write(data);
    }
} finally {
    if(output != null) {
        output.close();
    }
}

```
###  FileInputStream、FileOutputStream、RandomAccessFile 
RandomAccessFile允许你来回读写文件，也可以替换文件中的某些部分。FileInputStream和FileOutputStream没有这样的功能。
` RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw"); `
在RandomAccessFile的某个位置读写之前，必须把文件指针指向该位置。通过` seek() `方法可以达到这一目标。可以通过调用` getFilePointer() `获得当前文件指针的位置。例子如下：
```

RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");

file.seek(200);

long pointer = file.getFilePointer();

file.close();

```
###  File 
Java IO API中的FIle类可以让你访问底层文件系统，通过File类，你可以做到以下几点：

  * 检测文件是否存在
  * 读取文件长度
  * 重命名或移动文件
  * 删除文件
  * 检测某个路径是文件还是目录
  * 读取目录中的文件列表

请注意：File只能访问文件以及文件系统的元数据。如果你想读写文件内容，需要使用FileInputStream、FileOutputStream或者RandomAccessFile。如果你正在使用Java NIO，并且想使用完整的NIO解决方案，你会使用到java.nio.FileChannel(否则你也可以使用File)
```

File file = new File("c:\\data\\input-file.txt");
boolean fileExists = file.exists();
long length = file.length();
boolean success = file.renameTo(new File("c:\\data\\new-file.txt"));
boolean success = file.delete();
boolean isDirectory = file.isDirectory();

File file = new File("c:\\data");
String[] fileNames = file.list();
File[] files = file.listFiles();

```
###  PrintStream 
PrintStream允许你把格式化数据写入到底层OutputStream中。比如，写入格式化成文本的int，long以及其他原始数据类型到输出流中，而非它们的字节数据。代码如下：
```

PrintStream output = new PrintStream(outputStream);
output.print(true);
output.print((int) 123);
output.println((float) 123.456);
output.printf(Locale.UK, "Text + data: %1$", 123);
output.close();

```
PrintStream包含2个强大的函数，分别是format()和printf()(这两个函数几乎做了一样的事情，但是C程序员会更熟悉printf())。
###  ObjectInput/OutputStream 
```

ObjectInputStream input = new ObjectInputStream(
                            new FileInputStream("object.data"));

MyClass object = (MyClass) input.readObject();
//etc
input.close();   
ObjectOutputStream output = new ObjectOutputStream(
                            new FileOutputStream("object.data"));

MyClass object = new MyClass();

output.writeObject(object); 这个object必须实现java.io.Serializable
//etc.
output.close();  

```
###  StreamTokenizer 
可参考：http://blog.csdn.net/zhouhong1026/article/details/7885114
###  文件压缩 ZipInput/OutputStream类 
```

public class ZipOutputStreamDemo1{
    public static void main(String[] args) throws IOException{
        File file = new File("d:" + File.separator + "hello.txt");
        File zipFile = new File("d:" + File.separator + "hello.zip");
        InputStream input = new FileInputStream(file);
        ZipOutputStream zipOut = new ZipOutputStream(new FileOutputStream(
                zipFile));
        zipOut.putNextEntry(new ZipEntry(file.getName()));
        // 设置注释
        zipOut.setComment("hello");
        int temp = 0;
        while((temp = input.read()) != -1){
            zipOut.write(temp);
        }
        input.close();
        zipOut.close();
    }
}

```
```

/**
 * 一次性压缩多个文件
 * */
public class ZipOutputStreamDemo2{
    public static void main(String[] args) throws IOException{
        // 要被压缩的文件夹
        File file = new File("d:" + File.separator + "temp");
        File zipFile = new File("d:" + File.separator + "zipFile.zip");
        InputStream input = null;
        ZipOutputStream zipOut = new ZipOutputStream(new FileOutputStream(
                zipFile));
        zipOut.setComment("hello");
        if(file.isDirectory()){
            File[] files = file.listFiles();
            for(int i = 0; i < files.length; ++i){
                input = new FileInputStream(files[i]);
                zipOut.putNextEntry(new ZipEntry(file.getName()
                        + File.separator + files[i].getName()));
                int temp = 0;
                while((temp = input.read()) != -1){
                    zipOut.write(temp);
                }
                input.close();
            }
        }
        zipOut.close();
    }
}

```
```

/**
 * 解压缩文件（压缩文件中只有一个文件的情况）
 * */
public class ZipFileDemo2{
    public static void main(String[] args) throws IOException{
        File file = new File("d:" + File.separator + "hello.zip");
        File outFile = new File("d:" + File.separator + "unZipFile.txt");
        ZipFile zipFile = new ZipFile(file);
        ZipEntry entry = zipFile.getEntry("hello.txt");
        InputStream input = zipFile.getInputStream(entry);
        OutputStream output = new FileOutputStream(outFile);
        int temp = 0;
        while((temp = input.read()) != -1){
            output.write(temp);
        }
        input.close();
        output.close();
    }
}

```
```

/**
 * 解压缩一个压缩文件中包含多个文件的情况
 * */
public class ZipFileDemo3{
    public static void main(String[] args) throws IOException{
        File file = new File("d:" + File.separator + "zipFile.zip");
        File outFile = null;
        ZipFile zipFile = new ZipFile(file);
        ZipInputStream zipInput = new ZipInputStream(new FileInputStream(file));
        ZipEntry entry = null;
        InputStream input = null;
        OutputStream output = null;
        while((entry = zipInput.getNextEntry()) != null){
            System.out.println("解压缩" + entry.getName() + "文件");
            outFile = new File("d:" + File.separator + entry.getName());
            if(!outFile.getParentFile().exists()){
                outFile.getParentFile().mkdir();
            }
            if(!outFile.exists()){
                outFile.createNewFile();
            }
            input = zipFile.getInputStream(entry);
            output = new FileOutputStream(outFile);
            int temp = 0;
            while((temp = input.read()) != -1){
                output.write(temp);
            }
            input.close();
            output.close();
        }
    }
}

```
##  经典IO操作示例 
```

import java.io.BufferedReader;
import java.io.FileReader;
public class InputStreamTest {

	public static String read(String filename) throws Exception {
		BufferedReader br = new BufferedReader(new FileReader(filename));
		String s;
		StringBuffer sb = new StringBuffer();
		while ((s = br.readLine()) != null) {
			sb.append(s + "\n");
		}
		br.close();
		return sb.toString();
	}

	public static void main(String[] args) throws Exception {
		System.out.println(read("src/InputStreamTest.java"));
	}
　}

```
```

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.PrintWriter;
import java.io.StringReader;

public class BasicFileOutput {

	static String file = "basie.out";

	public static void main(String[] args) throws Exception {
		BufferedReader in = new BufferedReader(new StringReader(
				InputStreamTest.read("src/BasicFileOutput.java")));
		PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(
				file)));
		int lineCount = 1;
		String s;
		while ((s = in.readLine()) != null) {
			out.println(lineCount++ + ": " + s);
		}
		out.close();
		System.out.println(InputStreamTest.read(file));
	}
}

```