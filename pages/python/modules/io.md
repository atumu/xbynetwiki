author:xbynet
title:Python模块文档之IO
modifyAt:2016-12-26 12:59:05
location:python/modules/io
createAt:2016-12-26 11:57:30

![9e5e4548-cb1c-11e6-9279-00163e2eed34.png](/data/upload/9e5e4548-cb1c-11e6-9279-00163e2eed34.png)
三种类型IO：text I/O, binary I/O and raw I/O
从3.3开始： 原来的IOError被[OSError](https://docs.python.org/3/library/exceptions.html#OSError)所取代。IOError变成了OSError的别名。
```
#text io
f = open("myfile.txt", "r", encoding="utf-8")
f = io.StringIO("some initial text data")

#bytesio
f = open("myfile.jpg", "rb")
f = io.BytesIO(b"some initial binary data: \x00\x01")

#raw io    buffering=0
f = open("myfile.jpg", "rb", buffering=0)
```
# 高级接口：
**缓存大小**：io.DEFAULT_BUFFER_SIZE。内置函数open()使用文件的blksize(块大小,从os.stat()中可以获取)来作为缓存大小。
io.open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)，内置open()函数的别名
exception io.BlockingIOError，内置BlockingIOError的别名
exception io.UnsupportedOperation

# In-memory streams
**StringIO,BytesIO.还有sys.stdin, sys.stdout, and sys.stderr.**

# 类层次
见上图
 注：BufferedRandom提供了文件随机访问的特性。
 缩略词： abstract base classes (ABCs)
 
 |ABC	|Inherits	|Stub Methods	|Mixin Methods and Properties|
| -------- | -------- | -------- | -------- |
|IOBase	 |	|fileno, seek, and truncate	|close, closed, __enter__, __exit__, flush, isatty, __iter__, __next__, readable, readline, readlines, seekable, tell, writable, and writelines|
|RawIOBase	|IOBase	|readinto and write	|Inherited IOBase methods, read, and readall|
|BufferedIOBase	|IOBase	|detach, read, read1, and write	|Inherited IOBase methods, readinto|
|TextIOBase	|IOBase	|detach, read, readline, and write	|Inherited IOBase methods, encoding, errors, and newlines|

# 线程安全性
`FileIO` objects are **thread-safe** to the extent that the operating system calls (such as read(2) under Unix) they wrap are thread-safe too.

Binary buffered objects (instances of **BufferedReader, BufferedWriter, BufferedRandom and BufferedRWPair**) protect their internal structures using a lock; it is therefore safe to call them from multiple threads at once.

`TextIOWrapper` objects are **not thread-safe.**