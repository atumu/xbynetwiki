author:xbynet
title:python压缩与解压
modifyAt:2016-12-27 12:56:30
location:python/modules/python压缩与解压
createAt:2016-12-27 09:46:36

# bz2
bz2模块主要接口:

* open() 函数和BZ2File类用于读写压缩文件.
* BZ2Compressor and BZ2Decompressor类用于增量式压缩和解压。
* compress() and decompress() 函数用于一次性压缩和解压.

bz2.open(filename, mode='r', compresslevel=9, encoding=None, errors=None, newline=None) 以二进制或文本模式打开bz2压缩文件，并返回file object
参数说明：

* filename，支持路径字符串、已存在的文件对象。
* mode选项支持'r', 'rb', 'w', 'wb', 'x', 'xb', 'a' or 'ab' for binary mode, or 'rt', 'wt', 'xt', or 'at' for text mode. The default is 'rb'
* compresslevel支持1-9,越高压缩级别越高。

class bz2.BZ2File(filename, mode='r', buffering=None, compresslevel=9) 以二进制方式打开bz2压缩文件。BZ2File支持迭代与with语句
BZ2File内部对 io.BufferedIOBase进行了包装，除此之外还提供以下方法：peek([n])

```
import bz2
import io
import os

data = 'Contents of the example file go here.\n'

with bz2.BZ2File('example.bz2', 'wb') as output:
    with io.TextIOWrapper(output, encoding='utf-8') as enc:
        enc.write(data)

os.system('file example.bz2')
#example.bz2: bzip2 compressed data, block size = 900k
```
```
import bz2
import io

with bz2.BZ2File('example.bz2', 'r') as input:
    with io.TextIOWrapper(input, encoding='utf-8') as dec:
        print(dec.read())
```
# 增量式压缩解压
class bz2.BZ2Compressor(compresslevel=9)，主要方法：compress(data)，flush() 一定要在最后调用flush()来完成压缩写入
class bz2.BZ2Decompressor，主要方法decompress(data, max_length=-1)，标记变量eof(True if the end-of-stream marker has been reached.)

```
import bz2
import binascii
import io

compressor = bz2.BZ2Compressor()

with open('lorem.txt', 'rb') as input:
    while True:
        block = input.read(64)
        if not block:
            break
        compressed = compressor.compress(block)
        if compressed:
            print('Compressed: {}'.format(
                binascii.hexlify(compressed)))
        else:
            print('buffering...')
    remaining = compressor.flush()
    print('Flushed: {}'.format(binascii.hexlify(remaining)))
```
```
import bz2

lorem = open('lorem.txt', 'rt').read().encode('utf-8')
compressed = bz2.compress(lorem)
combined = compressed + lorem

decompressor = bz2.BZ2Decompressor()
decompressed = decompressor.decompress(combined)

decompressed_matches = decompressed == lorem
print('Decompressed matches lorem:', decompressed_matches)

unused_matches = decompressor.unused_data == lorem
print('Unused data matches lorem :', unused_matches)
```
# 一次性压缩解压
bz2.compress(data, compresslevel=9)
bz2.decompress(data)

```
import bz2
import binascii

original_data = b'This is the original text.'
print('Original     : {} bytes'.format(len(original_data)))
print(original_data)

print()
compressed = bz2.compress(original_data)
print('Compressed   : {} bytes'.format(len(compressed)))
hex_version = binascii.hexlify(compressed)
for i in range(len(hex_version) // 40 + 1):
    print(hex_version[i * 40:(i + 1) * 40])

print()
decompressed = bz2.decompress(compressed)
print('Decompressed : {} bytes'.format(len(decompressed)))
print(decompressed)
```

```
import bz2

original_data = b'This is the original text.'

fmt = '{:>15}  {:>15}'
print(fmt.format('len(data)', 'len(compressed)'))
print(fmt.format('-' * 15, '-' * 15))

for i in range(5):
    data = original_data * i
    compressed = bz2.compress(data)
    print(fmt.format(len(data), len(compressed)), end='')
    print('*' if len(data) < len(compressed) else '')
```

# 压缩网络数据
```
import bz2
import logging
import socketserver
import binascii

BLOCK_SIZE = 32


class Bz2RequestHandler(socketserver.BaseRequestHandler):

    logger = logging.getLogger('Server')

    def handle(self):
        compressor = bz2.BZ2Compressor()

        # Find out what file the client wants
        filename = self.request.recv(1024).decode('utf-8')
        self.logger.debug('client asked for: "%s"', filename)

        # Send chunks of the file as they are compressed
        with open(filename, 'rb') as input:
            while True:
                block = input.read(BLOCK_SIZE)
                if not block:
                    break
                self.logger.debug('RAW %r', block)
                compressed = compressor.compress(block)
                if compressed:
                    self.logger.debug(
                        'SENDING %r',
                        binascii.hexlify(compressed))
                    self.request.send(compressed)
                else:
                    self.logger.debug('BUFFERING')

        # Send any data being buffered by the compressor
        remaining = compressor.flush()
        while remaining:
            to_send = remaining[:BLOCK_SIZE]
            remaining = remaining[BLOCK_SIZE:]
            self.logger.debug('FLUSHING %r',
                              binascii.hexlify(to_send))
            self.request.send(to_send)
        return
```
```
if __name__ == '__main__':
    import socket
    import sys
    from io import StringIO
    import threading

    logging.basicConfig(level=logging.DEBUG,
                        format='%(name)s: %(message)s',
                        )

    # Set up a server, running in a separate thread
    address = ('localhost', 0)  # let the kernel assign a port
    server = socketserver.TCPServer(address, Bz2RequestHandler)
    ip, port = server.server_address  # what port was assigned?

    t = threading.Thread(target=server.serve_forever)
    t.setDaemon(True)
    t.start()

    logger = logging.getLogger('Client')

    # Connect to the server
    logger.info('Contacting server on %s:%s', ip, port)
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((ip, port))

    # Ask for a file
    requested_file = (sys.argv[0]
                      if len(sys.argv) > 1
                      else 'lorem.txt')
    logger.debug('sending filename: "%s"', requested_file)
    len_sent = s.send(requested_file.encode('utf-8'))

    # Receive a response
    buffer = StringIO()
    decompressor = bz2.BZ2Decompressor()
    while True:
        response = s.recv(BLOCK_SIZE)
        if not response:
            break
        logger.debug('READ %r', binascii.hexlify(response))

        # Include any unconsumed data when feeding the
        # decompressor.
        decompressed = decompressor.decompress(response)
        if decompressed:
            logger.debug('DECOMPRESSED %r', decompressed)
            buffer.write(decompressed.decode('utf-8'))
        else:
            logger.debug('BUFFERING')

    full_response = buffer.getvalue()
    lorem = open(requested_file, 'rt').read()
    logger.debug('response matches file contents: %s',
                 full_response == lorem)

    # Clean up
    server.shutdown()
    server.socket.close()
    s.close()
```

# gzip
gzip模块由zlib模块提供底层支持
gzip模块提供 GzipFile类及open(), compress() and decompress()函数
gzip.open(filename, mode='rb', compresslevel=9, encoding=None, errors=None, newline=None)
class gzip.GzipFile(filename=None, mode=None, compresslevel=9, fileobj=None, mtime=None)
gzip.compress(data, compresslevel=9)
gzip.decompress(data)
```
import gzip
with gzip.open('/home/joe/file.txt.gz', 'rb') as f:
    file_content = f.read()
```
```
import gzip
content = b"Lots of content here"
with gzip.open('/home/joe/file.txt.gz', 'wb') as f:
    f.write(content)
```
压缩已存在文件
```
import gzip
import shutil
with open('/home/joe/file.txt', 'rb') as f_in:
    with gzip.open('/home/joe/file.txt.gz', 'wb') as f_out:
        shutil.copyfileobj(f_in, f_out)
```
```
import gzip
s_in = b"Lots of content here"
s_out = gzip.compress(s_in)
```

# zipfile
异常:
exception zipfile.BadZipFile
exception zipfile.LargeZipFile
类和函数
class zipfile.ZipFile
class zipfile.PyZipFile
class zipfile.ZipInfo(filename='NoName', date_time=(1980, 1, 1, 0, 0, 0))
zipfile.is_zipfile(filename)
压缩方式常量：
zipfile.ZIP_STORED 仅存储而不压缩
zipfile.ZIP_DEFLATED zip方式压缩
zipfile.ZIP_BZIP2  bzip2方式压缩
zipfile.ZIP_LZMA lzma方式压缩

class zipfile.ZipFile(file, mode='r', compression=ZIP_STORED, allowZip64=True) 支持迭代及with语句。 r,w,a,x
```
with ZipFile('spam.zip', 'w') as myzip:
    myzip.write('eggs.txt')
```

ZipFile.close()
ZipFile.getinfo(name)
ZipFile.infolist()
ZipFile.namelist()

ZipFile.open(name, mode='r', pwd=None, *, force_zip64=False)
如果压缩文件大于2G，需要开启zip64支持
```
with ZipFile('spam.zip') as myzip:
    with myzip.open('eggs.txt') as myfile:
        print(myfile.read())
```

ZipFile.extract(member, path=None, pwd=None)
ZipFile.extractall(path=None, members=None, pwd=None)
ZipFile.printdir()
ZipFile.setpassword(pwd)
ZipFile.read(name, pwd=None)
ZipFile.testzip()
ZipFile.write(filename, arcname=None, compress_type=None)
ZipFile.writestr(zinfo_or_arcname, data[, compress_type])
ZipFile.debug指定日志级别，0-3

class zipfile.PyZipFile(file, mode='r', compression=ZIP_STORED, allowZip64=True, optimize=-1)

ZipInfo类
Instances of the ZipInfo class are returned by the **getinfo**() and **infolist**() methods of ZipFile objects. Each object stores information about a single member of the ZIP archive.
classmethod ZipInfo.from_file(filename, arcname=None)
ZipInfo.is_dir()
ZipInfo.filename
ZipInfo.date_time
ZipInfo.compress_type
ZipInfo.compress_size：Size of the compressed data.
ZipInfo.file_size：Size of the uncompressed file.

命令行
选项-c 创建压缩文件, -e解压，-l 列出压缩文件条目
```
$ python -m zipfile -c monty.zip spam.txt eggs.txt
$ python -m zipfile -c monty.zip life-of-brian_1979/
$ python -m zipfile -e monty.zip target-dir/
$ python -m zipfile -l monty.zip
```

# tarfile
tarfile模块支持在tar归档的基础之上进行gzip, bz2 and lzma压缩
tarfile.open(name=None, mode='r', fileobj=None, bufsize=10240, **kwargs) 返回 TarFile对象
参数mode形式为 'filemode[:compression]' ,支持`'r' , 'r:*','r:gz','r:bz2','r:xz','x','x:gz','x:bz2','x:xz','a','w','w:gz','w:bz2','w:xz'`

class tarfile.TarFile(name=None, mode='r', fileobj=None, format=DEFAULT_FORMAT, tarinfo=TarInfo, dereference=False, ignore_zeros=False, encoding=ENCODING, errors='surrogateescape', pax_headers=None, debug=0, errorlevel=0)
classmethod TarFile.open(...)： tarfile.open() 的别名
TarFile.extractall(path=".", members=None, *, numeric_owner=False)
TarFile.extract(member, path="", set_attrs=True, *, numeric_owner=False)
TarFile.extractfile(member)
TarFile.add(name, arcname=None, recursive=True, exclude=None, *, filter=None)
TarFile.addfile(tarinfo, fileobj=None)
其余见文档

TarInfo Objects
命令行:略。

```
def pack(excludes=[]):

    def ecludefiles(path):
        for name in ['venv','celerybeat-schedule','node_modules','websrc','__pycache__','.git','.idea','dist','.log']+excludes:
            if path.find(os.sep+name)>0:
                return True
        return False

    if os.path.exists(distFile):
        os.remove(distFile)
    with tarfile.open(distFile,'w:gz') as f:
        f.add(srcPath,arcname='mdwiki',exclude=ecludefiles)

```

参考：https://pymotw.com/3/bz2/
https://pymotw.com/3/gzip/