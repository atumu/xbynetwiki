title: java7_aio 

#  Java7 AIO 

随着JDK7的发布，Java的AIO正式支持版本也出炉了，就像当年发布NIO特性支持时，基本上所有的Java服务器都重写了自己的网络框架以通过NIO来提高服务器的性能。AIO的发布势必也会引起Java界的一次重写风潮，现在很多的网络框架（如Mina），大型软件（如Oracle DB）都宣布自己已经在新版本中支持了AIO的特性以提高性能。下面就来看一下aio的基本原理，以及如何使用JDK7的AIO特性。

所谓AIO，异步IO，其主要是针对进程在调用IO获取外部数据时，是否阻塞调用进程而言的。一个进程的IO调用步骤大致如下：
1、进程向操作系统请求数据
2、操作系统把外部数据加载到内核的缓冲区中，
3、操作系统把内核的缓冲区拷贝到进程的缓冲区
4、进程获得数据完成自己的功能
<note tip>当操作系统在把外部数据放到进程缓冲区的这段时间（即上述的第二，三步），如果应用进程是挂起等待的，那么就是同步IO，反之，就是异步IO，也就是AIO</note>
JDK对于IO支持基本上都是基于操作系统的封装，**而IO操作的核心就是如何有效的管理Channel（数据通道）**，JDK7对AIO的支持主要提供如下的一些封装类：
  * **AsynchronousChannel**：所有AIO Channel的父类。
  * **AsynchronousByteChannel**：支持Byte读写的Channel
  * **AsynchronousFileChannel**：支持文件读写的Channel
  * **AsynchronousDatagramChannel**：支持数据包（datagram）读写的Channel
  * **AsynchronousServerSocketChannel**：支持数据流读写的服务器端Channel
  * **AsynchronousSocketChannel**：支持数据流读写的客户端Channel
  * **AsynchronousChannelGroup**：支持资源共享的Channel分组

对于AIO的Channel，JDK定义了2种类型的操作，
1、**Future operation（….)**:即通过Future判断是操作是否完成。
2、**void operation(Object attachment, CompletionHandler handler)**:即通过CompletionHandler来通知异步IO完成。

下面就是一个AsynchronousFileChannel实现的完整异步读写文件的例子
```

4. import java.io.IOException;
5. import java.nio.ByteBuffer;
6. import java.nio.channels.AsynchronousFileChannel;
7. import java.nio.channels.CompletionHandler;
8. import java.nio.file.Path;
9. import java.nio.file.Paths;
10. import java.util.concurrent.ExecutionException;
11. import java.util.concurrent.Future;

20. public class AFCDemo {
21. static Thread current;
22.
23. public static void main(String[] args) throws IOException, InterruptedException, ExecutionException {
24. if (args == null || args.length == 0) {
25. System.out.println("Please input file path");
26. return;
27. }
28. Path filePath = Paths.get(args[0]);
29. AsynchronousFileChannel afc = AsynchronousFileChannel.open(filePath);
30. ByteBuffer byteBuffer = ByteBuffer.allocate(16 * 1024);
31. //使用FutureDemo时，请注释掉completionHandlerDemo，反之
亦然
32. futureDemo(afc, byteBuffer);
33. completionHandlerDemo(afc, byteBuffer);
34. }
35.
36. private static void completionHandlerDemo(AsynchronousFileChannel afc, ByteBuffer byteBuffer) throws IOException{
37. current = Thread.currentThread();
38. afc.read(byteBuffer, 0, null, new CompletionHandler<Integer, Object>() {
39. @Override
40. public void completed(Integer result, Object attachme
nt) {

41. System.out.println("Bytes Read = " + result);
42. current.interrupt();
43. }
44.
45. @Override
46. public void failed(Throwable exc, Object attachment) {
47. System.out.println(exc.getCause());
48. current.interrupt();
49. }
50. });
51. System.out.println("Waiting for completion…");
52. try {
53. current.join();
54. } catch (InterruptedException e) {
55. }
56. System.out.println("End");
57. afc.close();
58. }
59.
60. private static void futureDemo(AsynchronousFileChannel afc, ByteBuffer byteBuffer) throws InterruptedException, ExecutionException, IOException {
61. Future<Integer> result = afc.read(byteBuffer, 0);
62. while (!result.isDone()) {
63. System.out.println("Waiting file channel finished….");
64. Thread.sleep(1);
65. }
66. System.out.println("Finished? = " + result.isDone());
67. System.out.println("byteBuffer = " + result.get());
68. System.out.println(byteBuffer);
69. afc.close();
70. }
71. }

```