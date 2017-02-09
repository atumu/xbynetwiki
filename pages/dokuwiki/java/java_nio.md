title: java_nio 

#  Java NIO 
本文参考：
http://ifeve.com/java-nio-all/
http://www.cnblogs.com/zhuYears/p/3184116.html
还可以参考的资料：
http://www.iteye.com/magazines/132-Java-NIO
**不过目前一般Java NIO会采用以下开源库之一：**
  * Netty
  * MINA
关于这两个开源库的说明可以参考如下资料：
http://lippeng.iteye.com/blog/1907279 
http://blog.csdn.net/kobejayandy/article/details/11493717
http://ifeve.com/category/netty/


jdk1.4提供了java.nio包，为从根本上改善I/O的性能提供了可能.俗称新IO
Java NIO 由以下几个核心部分组成：
  * Channels
  * Buffers
  * Selectors
Java NIO 中除此之外还有很多类和组件，如Pipe和FileLock，只不过是与三个核心组件共同使用的工具类.
Channel 和 Buffer基本上，**所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。**
Channel和Buffer有好几种类型。下面是JAVA NIO中的一些主要Channel的实现：
  * FileChannel
  * DatagramChannel
  * SocketChannel
  * ServerSocketChannel
正如你所看到的，这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。与这些类一起的有一些有趣的接口，但为简单起见，我尽量在概述中不提到它们。
以下是Java NIO里关键的Buffer实现：
  * ByteBuffer
  * CharBuffer
  * DoubleBuffer
  * FloatBuffer
  * IntBuffer
  * LongBuffer
  * ShortBuffer
这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int,long, float, double 和 char。
Java NIO 还有个 **Mappedyteuffer**，用于表示内存映射文件， 我也不打算在概述中说明。
**注意：StringBuffer不是一个Buffer**

**Channels and Buffers（通道和缓冲区）**
标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。

**Non-blocking IO（非阻塞IO）**
Java NIO可以让你` 非阻塞 `的使用IO，例如：**当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。**


**Selector**
![](/data/dokuwiki/java/pasted/20150813-060158.png)
**Selector允许单线程处理多个 Channel。单个的线程可以监听多个数据通道。**如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。
这是在一个单线程中使用一个Selector处理3个Channel的图示：要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。
##  Java NIO和IO的主要区别
IO                NIO
面向流            面向缓冲
阻塞IO            非阻塞IO
无                选择器
**我应该何时使用IO，何时使用NIO呢？**在本文中，我会尽量清晰地解析Java NIO和IO的差异、它们的使用场景，以及它们如何影响您的代码设计。
无论您选择IO或NIO工具箱，可能会影响您应用程序设计的以下几个方面：
  * 对NIO或IO类的API调用。
  * 数据处理。
  * 用来处理数据的线程数。


##  Java NIO的使用场景 
**nio缺点：**
如调用read方法之后不能确定是否包含足够数据，但是阻塞IO中BufferedReader中的readLine()可以知道读取了一行。
用一个线程管理多个通道连接，管理复杂度较高。除非高并发情况下

**对于文件I/O， 在我看来使用IO和NIO是区别不大的**，` **Java1.4开始原始IO也根据NIO重新实现过了，提供了对于NIO特性的支持**。即使是流，也会比以前更加高效。 `企业级应用软件中涉及I/O的部分多半是读写文件的功能性需求，很少有在并发上的要求，那么JavaIO包已经很胜任了。
**对于网络I/O**，传统的阻塞式I/O，一个线程对应一个连接，采用线程池的模式在大部分场景下简单高效。**当连接数茫茫多时，并且数据的移动非常频繁，NIO无疑是更好的选择。**
**NIO标榜的是高速、可伸缩的I/O，因为它更亲近操作系统。当需求很平凡，没有太高的效率要求的时候，你看不出它的好，反而觉得NIO代码实现复杂，不易理解。**选择与否全看使用的场景，这点就看使用者的权衡了。
此部分参考：http://www.cnblogs.com/zhuYears/p/3166571.html

##  nio通常需要涉及到三个对象 
1、数据源：从文件中获得的FileInputStream/FileOutputStream、从socket中获得的输入输出流等

2、缓冲区对象(buffer):nio定义了一系列buffer对象，最基本的是ByteBuffer,其他还有各种基本数据类型除了boolean类型外，所对应的buffer类型。使用这些buffer一般的过程是：从数据源中读到ByteBuffer中，然后使用ByteBuffer.asXxxBuffer()转换成特定的基本类型，然后对这个基本类型的buffer进行操作；把基本类型转换成ByteBuffer类型，然后写入数据源中。

3、通道对象(channel):他提供了对数据源的一个连接，可以从文件中获得FileChannel,从Socket中获得SocketChannel,它是一个中介。提供了数据源与缓冲区之间的读写操作。
我们可以同过一句话来解释他们三者之间的关系：**channel对象提供了数据源与缓冲区之间的读写操作，是数据源与缓冲区的一个桥梁，通过这个桥梁，数据在两者之间流动。**
##  Channel 
通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。
下面是一个使用FileChannel读取数据到Buffer中的示例：
```

RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {

System.out.println("Read " + bytesRead);
buf.flip();

while(buf.hasRemaining()){
System.out.print((char) buf.get());
}

buf.clear();
bytesRead = inChannel.read(buf);
}
aFile.close();

```
注意 buf.flip() 的调用，首先读取数据到Buffer，然后反转Buffer,接着再从Buffer中读取数据
###  FileChannel 
Java NIO中的FileChannel是一个连接到文件的通道。可以通过文件通道读写文件。
FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。
我们无法直接打开一个FileChannel，需要通过使用一个InputStream、OutputStream或RandomAccessFile来获取一个FileChannel实例。
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
**从FileChannel读取数据**
调用多个read()方法之一从FileChannel中读取数据。如：
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
read()方法返回的int值表示了有多少字节被读到了Buffer中。**如果返回-1，表示到了文件末尾。**
**向FileChannel写数据**
```

String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
	channel.write(buf);
}

```
用完FileChannel后必须将其关闭。如：` channel.close(); `
**FileChannel的position方法**
有时可能需要在FileChannel的某个特定位置进行数据的读/写操作。可以通过调用` position() `方法获取FileChannel的当前位置。
也可以通过调用position(long pos)方法设置FileChannel的当前位置。
long pos = channel.position();
channel.position(pos +123);
**size方法：**
long fileSize = channel.` size( `);
**FileChannel的truncate方法：**
可以使用FileChannel.` truncate( `)方法截取一个文件。截取文件时，文件将中指定长度后面的部分将被删除。如：
channel.truncate(1024);
这个例子截取文件的前1024个字节。
**FileChannel的force方法**
` FileChannel.force() `方法将通道里尚未写入磁盘的数据强制写到磁盘上。force()方法有一个boolean类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。

###  通道之间的数据传输 
在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel（译者注：channel中文常译作通道）传输到另外一个channel。
####  transferFrom() 

` FileChannel的transferFrom() `方法可以**将数据从源通道传输到FileChannel中**。下面是一个简单的例子：
```

RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

toChannel.transferFrom(position, count, fromChannel);

```
方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。
此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。
####  transferTo() 

` transferTo() `方法**将数据从FileChannel传输到其他的channel中**。下面是一个简单的例子：
```

RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);

```
上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。SocketChannel会一直传输数据直到目标buffer被填满。
##  Buffer 
Java NIO中的Buffer用于和NIO通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的
缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。
**Buffer的基本用法**
使用Buffer读写数据一般遵循以下四个步骤：
  * 写入数据到Buffer
  * 调用flip()方法
  * 从Buffer中读取数据
  * 调用clear()方法或者compact()方法

当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要` **通过flip()方法将Buffer从写模式切换到读模式** `。在读模式下，可以读取之前写入到buffer的所有数据。
一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。
有两种方式能清空缓冲区：调用` clear()或compact() `方法。
  * clear()方法会清空整个缓冲区。
  * compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。
```

RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //read into buffer.
while (bytesRead != -1) {

  buf.flip();  //make buffer ready for read

  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read 1 byte at a time
  }

  buf.clear(); //make buffer ready for writing
  bytesRead = inChannel.read(buf);
}
aFile.close();

```

###  Buffer三个重要的概念以及他们之间的关系 

  * **capacity** 容量
  * **limit** 
  * **position** 

**position和limit的含义取决于Buffer处在读模式还是写模式。**不管Buffer处在什么模式，capacity的含义总是一样的。
![](/data/dokuwiki/java/pasted/20150813-061402.png)
理解这三者的关系是对缓冲区操作的关键：新创建的一个缓冲区，position=0,limit=capacity 我们可以通过position(int newPosition)和limit(int newLimit)来调整position和limit的值。显然0 =< position =< limit =< capacity
**为什么需要limit呢？这是理解缓冲区操作的关键所在。**
一段数据怎么去标示出来呢？两个点决定了一个段啊，而position就是起点，limit就是终点。但这个起点和终点在不同时候，他表示的含义不同：
**当把数据写入buffer时，poision到limit就是可以写的空间**
**当从buffer读数据时，position到limit就是可读取的数据**
capacity
作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。
position
当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.
当读取数据时，也是从某个特定位置读。**当将Buffer从写模式切换到读模式，position会被重置为0.** 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。
limit
在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。** 写模式下，limit等于Buffer的capacity。**
**当切换Buffer到读模式时， limit表示你最多能读到多少数据。**因此，**当切换Buffer到读模式时，limit会被设置成写模式下的position值。**换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，**这个值在写模式下就是position**）
###  flip、rewind、clear、compact的区别 
` flip `：flip方法将Buffer**从写模式切换到读模式**。调用flip()方法会将position设回0，并将limit设置成之前position的值。
` rewind() `：在读模式下调用Buffer.` rewind() `将position设回0，所以你可以**重**读Buffer中的所有数据。limit保持不变
一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。
` clear() `:如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。
` compact(): `如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

**所以我们经常做的事情就是：把数据写入Buffer，,然后使用buf.flip()操作，最后把Buffer写入File中.最后通过clear()清空buffer**

###  mark()与reset()方法 
通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。
更多方法可以参考：http://www.cnblogs.com/zhuYears/p/3184116.html
Scatter/Gathe
r###  Buffer的分配、读写 
读数据到Buffer
```

ByteBuffer buf = ByteBuffer.allocate(48);
CharBuffer buf = CharBuffer.allocate(1024);
int bytesRead = inChannel.read(buf); //read into buffer.
//buf.put(127);

```
**flip()方法**
**flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。**
写数据到Channel:
```

//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
//byte aByte = buf.get();

```
**rewind()方法
Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变.**

##  Scatter/Gather 
Java NIO开始支持scatter/gather，scatter/gather用于描述从Channel（译者注：Channel在中文经常翻译为通道）中读取或者写入到Channel的操作。
分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。
聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。
scatter / gather经常用于需要将传输的**数据分开处理**的场合

**Scattering Reads**
```

ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);

```
注意buffer首先被插入到数组，然后再将数组作为channel.read() 的输入参数。read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。
Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息(译者注：消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。

**Gathering Writes**:
```

ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);

```
buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入。因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。
##   Selector 
Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，**一个单独的线程可以管理多个channel，从而管理多个网络连接。**
Selector的创建
Selector selector = Selector.open();
向Selector注册通道
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,
	Selectionkey.OP_READ);
与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。
注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

Connect
Accept
Read
Write
通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

这四种事件用SelectionKey的四个常量来表示：

SelectionKey.OP_CONNECT
SelectionKey.OP_ACCEPT
SelectionKey.OP_READ
SelectionKey.OP_WRITE
如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
在下面还会继续提到interest集合。
selectionKey.isAcceptable();
selectionKey.isConnectable()
selectionKey.isReadable();
selectionKey.isWritable();

Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
可以将一个对象或者更多信息附着到SelectionKey上，这样就能方便的识别某个给定的通道。具体略。。。。
一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。

下面是select()方法：
int select()
int select(long timeout)
int selectNow()
select()阻塞到至少有一个通道在你注册的事件上就绪了。
select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)。
selectNow()不会阻塞，不管什么通道就绪都立刻返回（译者注：此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）。
select()方法返回的int值表示有多少通道已经就绪。
selectedKeys()
一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：
Set selectedKeys = selector.selectedKeys();
当像Selector注册Channel时，Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。可以通过SelectionKey的selectedKeySet()方法访问这些对象。
可以遍历这个已选择的键集合来访问就绪的通道。如下：
```

Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
}

```
这个循环遍历已选择键集中的每个键，并检测各个键所对应的通道的就绪事件。
**注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。**下次该通道变成就绪时，Selector会再次将其放入已选择键集中。
SelectionKey.channel()方法返回的通道需要转型成你要处理的类型，如ServerSocketChannel或SocketChannel等。
###  SocketChannel 
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
非阻塞模式
可以设置 SocketChannel 为非阻塞模式（non-blocking mode）.设置之后，就可以在异步模式下调用connect(), read() 和write()了。
如果SocketChannel在非阻塞模式下，此时调用connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用finishConnect()的方法。像这样：
```

socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...
}

```
**非阻塞模式与选择器搭配会工作的更好**，通过将一或多个SocketChannel注册到Selector，可以询问选择器哪个通道已经准备好了读取，写入等。Selector与SocketChannel的搭配使用会在后面详讲。
###   ServerSocketChannel 
```

ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}

```
ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的SocketChannel是否是null.如：
```

ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    if(socketChannel != null){
        //do something with socketChannel...
    }
}

```
###  完整的示例 

这里有一个完整的示例，打开一个Selector，注册一个通道注册到这个Selector上(通道的初始化过程略去),然后持续监控这个Selector的四种事件（接受，连接，读，写）是否就绪。
```

Selector selector = Selector.open();
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
while(true) {
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;
  Set selectedKeys = selector.selectedKeys();
  Iterator keyIterator = selectedKeys.iterator();
  while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
  }
}

```
