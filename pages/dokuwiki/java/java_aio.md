title: java_aio 

#  Java AIO(NIO.2) 
本文为完工。。。。。
本文参考：
http://www.ibm.com/developerworks/cn/java/j-lo-nio2/
http://yunhaifeiwu.iteye.com/blog/1714664
http://ningandjiao.iteye.com/blog/1508043 系列AIO文章

**不过目前一般Java NIO AIO会采用以下开源库之一：**

  * Netty
  * MINA
关于这两个开源库的说明可以参考如下资料：
http://lippeng.iteye.com/blog/1907279
http://blog.csdn.net/kobejayandy/article/details/11493717
http://ifeve.com/category/netty/

简单介绍 Asynchronous I/O
JDK7的发布带来了 AIO(NIO.2)，主要包括新的：
  * 异步 I/O（简称 AIO）；
  * Multicase 多播；
  * Stream Control Transport Protocol（SCTP）；
  * 文件系统 API；
  * 以及一些 I/O API 的更新，例如：java.io.File.toPath，NetworkChannel 的完整抽象，等等。

AIO 包括 Sockets 和 Files 两部分的异步通道接口及其实现，并尽量使用操作系统提供的原生本地 I/O 功能进行实现。
**AIO 的核心概念：发起非阻塞方式的 I/O 操作。当 I/O 操作完成时通知。
应用程序的责任就是：什么时候发起操作？ I/O 操作完成时通知谁？**
AIO 的 I/O 操作，有两种方式的 API 可以进行：
  * Future 方式；
  * Callback 方式。

###  Future 方式 

Future 方式：即提交一个 I/O 操作请求，返回一个 Future。然后您可以对 Future 进行检查，确定它是否完成，或者阻塞 IO 操作直到操作正常完成或者超时异常。使用 Future 方式很简单，比较典型的代码通常像清单 1 所示。
```

AsynchronousSocketChannel ch = AsynchronousSocketChannel.open(); 

 // 连接远程服务器，等待连接完成或者失败
 Future<Void> result = ch.connect(remote); 
 // 进行其他工作，例如，连接后的准备环境，f.e. 
 //prepareForConnection(); 
 //Future 返回 null 表示连接成功
 if(result.get()!=null){ 
    // 连接失败，清理刚才准备好的环境，f.e. 
    //clearPreparation(); 
    return; 
 } 

 // 网络连接正常建立
 ... 

 ByteBuffer buffer = ByteBuffer.allocateDirect(8192); 

 // 进行读操作
 Future<Integer> result = ch.read(buffer); 

 // 此时可以进行其他工作，f.e. 
 //prepareLocalFile(); 
 // 然后等待读操作完成
 try { 
    int bytesRead = result.get(); 
    if(bytesRead==-1){ 
    // 返回 -1 表示没有数据了而且通道已经结束，即远程服务器正常关闭连接。
        //clear(); 
        return; 
    } 
    
    // 处理读到的内容，例如，写入本地文件，f.e. 
    //writeToLocolFile(buffer); 
 } catch (ExecutionExecption x) { 
    //failed 
 }

```
需要注意的是，因为** Future.get()是同步的**，所以如果不仔细考虑使用场合，使用 Future 方式可能很容易进入完全同步的编程模式，
###  Callback 方式 

Callback 方式：即提交一个 I/O 操作请求，并且指定一个 CompletionHandler。当异步 I/O 操作完成时，便发送一个通知，此时这个 CompletionHandler 对象的 completed 或者 failed 方法将会被调用，样例代码如清单 2 所示。
完成处理接口
```

 public interface CompletionHandler<V,A> { 
    // 当操作完成后被调用，result 参数表示操作结果，
    //attachment 参数表示提交操作请求时的参数。
    void completed(V result, A attachment); 

    // 当操作失败是调用，exc 参数表示失败原因。attachment 参数同上。
    void failed(Throwable exc, A attachment); 
 }

```
V表示结果值的类型。对于异步网络通道的读写操作而言，这个结果值 V 都是整数类型，表示已经操作的卦数，如果是 -1，NIO.2 内核实现保证传递的 ByteBuffer参数不会有变化。
A表示关联到 I/O 操作的对象的类型。用于传递操作环境。通常会封装一个连接环境。
如果成功则 completed 方法被调用。如果失败则 failed 方法被调用。
准备好 CompletionHandler 之后，如何使用 CompletionHandler 呢？ AIO 提供四种类型的异步通道以及不同的 I/O 操作接受一个 CompletionHandler 对象，它们分别是：
AsynchronousSocketChannel：connect，read，write
AsynchronousFileChannel：lock，read，write
AsynchronousServerSocketChannel：accept
AsynchronousDatagramChannel：read，write，send，receive