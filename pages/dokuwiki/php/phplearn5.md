title: phplearn5 

#  PHP学习之文件IO操作 
##  文件基础 
###  文件读取总结: 
1、readfile():**将文件内容一次性读取，并输出到标准输出(浏览器)中**，这就意味着，我们无法临时保存到变量。**readfile() 自身不会导致任何内存问题（已优化很好了）。** 如果出现内存不足的错误，使用 ob_get_level() 确保输出缓存已经关闭。
int readfile ( string $filename [, bool $use_include_path = false [, resource $context ]] )

2、file():把整个文件**读入一个数组中**。每行一个到一个数组元素。
array file ( string $filename [, int $flags = 0 [, resource $context ]] )

3、fread — 读取文件**（可安全用于二进制文件）**
string fread ( resource $handle , int $length )

4、file_get_contents():将整个文件读入一个字符串,**file_get_contents() 函数是用来将文件的内容读入到一个字符串中的首选方法。如果操作系统支持还会使用内存映射技术来增强性能。**
string file_get_contents ( string $filename [, bool $use_include_path = false [, resource $context [, int $offset = -1 [, int $maxlen ]]]] )

5、fgets()、fgetss()：**从文件指针中读取一行**.fgetss()是fgets的变体，就可以**过滤PHP与HTML标记**
string fgets ( resource $handle [, int $length ] )

6、fgetc():一次读取一个字节
string fgetc ( resource $handle )

7、fopen()支持HTTP、FTP、和本地打开、fclose()、feof()

8、filesize()、file_exist()

10、**文件定位**:rewind()将文件指针返回到开始，fseek()定位文件指针到某处、ftell()返回当前文件指针位置.
bool rewind ( resource $handle )
int ftell ( resource $handle )
int fseek ( resource $handle , int $offset [, int $whence = SEEK_SET ] )。第三个参数可取值
  * SEEK_SET - 设定位置等于 offset 字节。表示文件开始处开始定位。默认值。
  * SEEK_CUR - 设定位置为当前位置加上 offset。
  * SEEK_END - 设定位置为文件尾加上 offset。

11、**flock()**:轻便的咨询文件锁定.默认情况下，这个函数会阻塞到获取锁.
bool flock ( resource $handle , int $operation [, int &$wouldblock ] )
operation 可以是以下值之一：
  * LOCK_SH取得共享锁定（读取的程序）。
  * LOCK_EX 取得独占锁定（写入的程序。
  * LOCK_UN 释放锁定（无论共享或独占）。
如果不希望 flock() 在锁定时堵塞，则是 LOCK_NB（Windows 上还不支持）。
```

<?php

$fp = fopen("/tmp/lock.txt", "r+");

if (flock($fp, LOCK_EX)) {  // 进行排它型锁定
    ftruncate($fp, 0);      // truncate file
    fwrite($fp, "Write something here\n");
    fflush($fp);            // flush output before releasing the lock
    flock($fp, LOCK_UN);    // 释放锁定
} else {
    echo "Couldn't get the lock!";
}
fclose($fp);
?>

```

###  文件写入总结 
1、fwrite()和fputs:fputs()是fwrite的别名。 写入文件**（可安全用于二进制文件）**
int fwrite ( resource $handle , string $string [, int $length ] )

2、file_put_contents — 将一个字符串写入文件
int file_put_contents ( string $filename , mixed $data [, int $flags = 0 [, resource $context ]] ).和依次调用 fopen()，fwrite() 以及 fclose() 功能一样。
flags 的值可以是 以下 flag 使用 OR (|) 运算符进行的组合。
  * FILE_USE_INCLUDE_PATH	在 include 目录里搜索 filename。 更多信息可参见 include_path。
  * FILE_APPEND	如果文件 filename 已经存在，追加数据而不是覆盖。
  * LOCK_EX	在写入时获得一个独占锁。



###  读/写具体说明 
** 1.函数 file('目标文件')**
把文件以数组的形式读出来，用循环方式遍历数组输
```

<?  
header('Content-Type:text/html; charset=utf-8');  
$file=file("test.txt");  
if($file)  
{  
    foreach ($file as $num=>$content)  
    {  
        echo "行数为 ".$num." 内容为 ".$content."<br>";  
    }  
}  
else 
{  
    echo "文件读取失败";  
}  
?> 

```
**readfile() 函数**读取文件，并把它写入标准输出(浏览器)中。这个函数底层实现很高效，而且尽可能少地占有php的内存
与file不同的是直接读取文件的内容。
```

<?php
echo readfile("webdictionary.txt");
?>

```
 **fopen()**
函数fopen('要打开的文件','用何种方式') 打开一个文件，方式有R R+ W W+ A B等等，如果使用W，那么没有该文件会自动创建该文件
打开文件的更好的方法是通过 fopen() 函数。此函数为您提供比 readfile() 函数更多的选项。
fopen() 的第一个参数包含被打开的文件名，第二个参数规定打开文件的模式。
![](/data/dokuwiki/php/pasted/20160406-111833.png)
```

<?php
$myfile = fopen("webdictionary.txt", "r") or die("Unable to open file!");
echo fread($myfile,filesize("webdictionary.txt"));
fclose($myfile);
?>

```
PHP 读取文件 - **fread()**
fread() 的第一个参数包含待读取文件的文件名，第二个参数规定待读取的最大字节数。
```

fread($myfile,filesize("webdictionary.txt"));

```
PHP 关闭文件 - **fclose()**
fclose() 函数用于关闭打开的文件。
注释：用完文件后把它们全部关闭是一个良好的编程习惯。您并不想打开的文件占用您的服务器资源。
fclose() 需要待关闭文件的名称（或者存有文件名的变量）：
```

<?php
$myfile = fopen("webdictionary.txt", "r");
// some code to be executed....
fclose($myfile);
?>

```
PHP 读取**单行**文件 - **fgets()**
fgets() 函数用于从文件读取单行。fgets()有一个变体fgetss()它可以过滤字符串中包含的PHP和HTML标记。
```

<?php
$myfile = fopen("webdictionary.txt", "r") or die("Unable to open file!");
echo fgets($myfile);//注释：调用 fgets() 函数之后，文件指针会移动到下一行。
fclose($myfile);
?>

```
PHP 检查 End-Of-File - **feof()**
feof() 函数检查是否已到达 "end-of-file" (EOF)。
feof() 对于遍历未知长度的数据很有用。
下例逐行读取 "webdictionary.txt" 文件，直到 end-of-file：
```

<?php
$myfile = fopen("webdictionary.txt", "r") or die("Unable to open file!");
// 输出单行直到 end-of-file
while(!feof($myfile)) {
  echo fgets($myfile) . "<br>";
}
fclose($myfile);
?>

```
PHP 读取单字符 - **fgetc()**
fgetc() 函数用于从文件中读取**单个字符**。
下例逐字符读取 "webdictionary.txt" 文件，直到 end-of-file：
```

<?php
$myfile = fopen("webdictionary.txt", "r") or die("Unable to open file!");
// 输出单字符直到 end-of-file
while(!feof($myfile)) {
  echo fgetc($myfile);
}
fclose($myfile);
?>

```
**filesize('目标文件') 读出目标文件的大小**
```

<?  
$filename=("t1.php");  
$size=filesize($filename);  
echo $size."byts.";  
?> 

```
**7.unlink('目标文件') 删除目标文件**(php中没有delete函数)

**8.copy('目标文件','复制的位置')** 复制的位置可以是当前目录，也可以指定精确位置

**9.mkdir('文件夹名')创建一个文件夹**
```

<?  
header('Content-Type:text/html; charset=utf-8');  
 
function mk($dir)  
{  
      
if (file_exists($dir) && is_dir($dir))  
{  
    echo "该文件夹名存在";  
}  
else 
{  
    if (mkdir($dir,0777));  
    echo $dir."创建成功";  
}  
}  
mk(date("Y-m-d"));  
?> 


```
10.**rmdir('目标文件夹') 删除文件夹，注，无法删除非空文件夹**，如果要删除非空文件夹，需要遍历文件夹里所有内容，然后先删除文件，再删除文件夹

文件写入
PHP 创建文件 - **fopen()**
**fopen() 函数也用于创建文件**。也许有点混乱，**但是在 PHP 中，创建文件所用的函数与打开文件的相同。**
如果您用 fopen() 打开并不存在的文件，此函数会创建文件，假定文件被打开为写入（w）或增加（a）。
下面的例子创建名为 "testfile.txt" 的新文件。此文件将被创建于 PHP 代码所在的相同目录中：
```

$myfile = fopen("testfile.txt", "w")

```
PHP 文件权限
如果您试图运行这段代码时发生错误，请检查您是否有向硬盘写入信息的 PHP 文件访问权限。

PHP 写入文件 - **fwrite()**
fwrite() 函数用于写入文件。
函数fwrite('目标文件','写入的内容')，给目标文件写入内容
下面的例子把姓名写入名为 "newfile.txt" 的新文件中：
```

<?php
$myfile = fopen("newfile.txt", "w") or die("Unable to open file!");
$txt = "Bill Gates\n";
fwrite($myfile, $txt);
$txt = "Steve Jobs\n";
fwrite($myfile, $txt);
fclose($myfile);
?>

```
PHP 覆盖（Overwriting）
如果现在 "newfile.txt" 包含了一些数据，我们可以展示在写入已有文件时发生的的事情。所有已存在的数据会被擦除并以一个新文件开始。

##  文件进阶PHP 中的 Streams 
http://www.oschina.net/translate/understanding-streams-in-php
http://php.net/manual/zh/book.stream.php
Streams 是在PHP 4.3.0版本被引入的，**它被用于统一文件、网络、数据压缩等类文件的操作方式**，为这些类文件操作提供了一组通用的函数接口。简而言之，**一个stream就是一个具有流式行为的资源对象**。也就是说，我们可以用线性的方式来对stream进行读取和写入。并且可以用使用fseek()来跳转到stream内的任意位置。

每个Streams对象都有一个包装类，在包装中可以添加处理特殊协议和编码的相关代码。PHP中已经内置了一些常用的包装类，我们也可以创建和注册自定义的包装类。**我们甚至能够使用现有的context和filter对包装类进行修改和增强。**

**Stream 可以通过<scheme>:/ /<target>方式来引用。**其中<scheme>是包装类的名字，<target>中的内容是由包装类的语法指定，不同的包装类的语法会有所不同。
**PHP默认的包装类是file:/ /，**也就是说我们在访问文件系统的时候，其实就是在使用一个stream。我们可以通过下面两种方式来读取文件中的内容，**readfile('/path/to/somefile.txt')或者readfile('file:/ / /path/to/somefile.txt')，这两种方式是等效的。如果你是使用readfile('http:/ /google.com/')，那么PHP会选取HTTP stream包装类来进行操作。**
正如上文所述，PHP提供了不少内建的包转类，protocol以及filter。 按照下文所述的方式，**可以查询到本机所支持的包装类：**
```

<?php
print_r(stream_get_transports());
print_r(stream_get_wrappers());
print_r(stream_get_filters());

```
在我机器上的输出结果为：
```

Array
(
    [0] => tcp
    [1] => udp
    [2] => unix
    [3] => udg
    [4] => ssl
    [5] => sslv3
    [6] => sslv2
    [7] => tls
)
Array
(
    [0] => https
    [1] => ftps
    [2] => compress.zlib
    [3] => compress.bzip2
    [4] => php
    [5] => file
    [6] => glob
    [7] => data
    [8] => http
    [9] => ftp
    [10] => zip
    [11] => phar
)
Array
(
    [0] => zlib.*
    [1] => bzip2.*
    [2] => convert.iconv.*
    [3] => string.rot13
    [4] => string.toupper
    [5] => string.tolower
    [6] => string.strip_tags
    [7] => convert.*
    [8] => consumed
    [9] => dechunk
    [10] => mcrypt.*
    [11] => mdecrypt.*
)

```
###  php:/ / 包装类 
PHP中内建了本语言用于处理I/O stream的包装类。可以分为几类，**基础的有php:/ /stdin,php:/ /stdout, 以及php:/ /stderr，**这3个stream分别映射到默认 的I/O资源。**同时PHP还提供了php:/ /input，通过这个包装类可以使用只读的方式访问POST请求中的raw body。** 这是一项非常有用的功能，特别是在处理那些将数据负载嵌入到POST请求中的远程服务时。
下面我们使用cURL工具来做一个简单的测试：
curl -d "Hello World" -d "foo=bar&#038;name=John" http://localhost/dev/streams/php_input.php
在PHP脚本中使用**print_r($_POST)的测试结果**如下所示：
```

Array
(
    [foo] => bar
    [name] => John
)

```
我们注意$_POST array中是无法访问到第一项数据的。**但是如果我们使用readfile('php:/ /input')，结果就不同了：**
```

Hello World&#038;foo=bar&#038;name=John

```
**PHP 5.1又增加了php:/ /memory和php:/ /tempstream这两个包转类，用于读写临时数据。**正如包装类命名中所暗示的，这些数据被存储在底层系统中的内存或者临时文件中。 
###  php:/ /filter元包装类 
**php:/ /filter是一个元包装类，用于为stream增加filter功能。在使用fopen()、readfile()或者file_get_contents()/stream_get_contents()等打开stream时，filter将被使能。**下面是一个例子：
```

<?php
// Write encoded data
file_put_contents("php://filter/write=string.rot13/resource=file:///path/to/somefile.txt","Hello World");
 
// Read data and encode/decode
readfile("php://filter/read=string.toupper|string.rot13/resource=http://www.google.com");

```
在第一个例子中使用了一个filter来对保存到磁盘中的数据进行编码处理，在二个例子中，使用两个级联的filter来从远端的URL读取数据。使用filter能为你的应用带来极为强大的功能。

你也可以使用filter_append()函数附加过滤器到fopen()打开的流上。示例如下:
使用DateTime类和流过滤器迭代bzip压缩的日志文件:
```

<?php
$dateStart = new \DateTime();
$dateInterval = \DateInterval::createFromDateString('-1 day');
$datePeriod = new \DatePeriod($dateStart, $dateInterval, 30);
foreach ($datePeriod as $date) {
    $file = 'sftp://USER:PASS@rsync.net/' . $date->format('Y-m-d') . '.log.bz2';
    if (file_exists($file)) {
        $handle = fopen($file, 'rb');
        stream_filter_append($handle, 'bzip2.decompress');
        while (feof($handle) !== true) {
            $line = fgets($handle);
            if (strpos($line, 'www.example.com') !== false) {
                fwrite(STDOUT, $line);
            }
        }
        fclose($handle);
    }
}


```
####  自定义流过滤器 
```

<?php
// Create custom filter
class DirtyWordsFilter extends php_user_filter
{
    /**
     * @param resource $in       Incoming bucket brigade
     * @param resource $out      Outgoing bucket brigade
     * @param int      $consumed Number of bytes consumed
     * @param bool     $closing  Last bucket brigade in stream?
     */
    public function filter($in, $out, &$consumed, $closing)
    {
        $words = array('grime', 'dirt', 'grease');
        $wordData = array();
        foreach ($words as $word) {
            $replacement = array_fill(0, mb_strlen($word), '*');
            $wordData[$word] = implode('', $replacement);
        }
        $bad = array_keys($wordData);
        $good = array_values($wordData);

        // Iterate each bucket from incoming bucket brigade
        while ($bucket = stream_bucket_make_writeable($in)) {
            // Censor dirty words in bucket data
            $bucket->data = str_replace($bad, $good, $bucket->data);

            // Increment total data consumed
            $consumed += $bucket->datalen;

            // Send bucket to downstream brigade
            stream_bucket_append($out, $bucket);
        }

        return PSFS_PASS_ON;
    }
}
stream_filter_register('dirty_words_filter', 'DirtyWordsFilter');

// Use custom filter
$handle = fopen('data.txt', 'rb');
stream_filter_append($handle, 'dirty_words_filter');
while (feof($handle) !== true) {
    echo fgets($handle);
}
fclose($handle);


```
###  Stream上下文-context 
**context是一组stream相关的参数或选项，使用context可以修改或增强包装类的行为。例如使用context来修改HTTP包装器是一个常用到的使用场景。** 这样我们就可以不使用cURL工具，就能完成一些简单的网络操作。下面是一个例子：
```

<?php
$opts = array(
  'http'=>array(
    'method'=>"POST",
    'header'=> "Auth: SecretAuthTokenrn" .
        "Content-type: application/x-www-form-urlencodedrn" .
              "Content-length: " . strlen("Hello World"),
    'content' => 'Hello World'
  )
);
$default = stream_context_get_default($opts);
readfile('http://localhost/dev/streams/php_input.php');

```
首先要定义一个options array,这是个二位数组，可以通过$array['wrapper']['option_name']的形式来访问其中的参数。（**注意每个包装类中context的options是不同的**）。**然后调用stream_context_get_default()来设置这些option,stream_context_get_default()同时还会将默认的context作为结果返回回来。（不建议这么使用，这么使用记得之后要恢复default context。）**设置完成后，接下来调用readfile()，就会应用刚才设置好的context来抓取内容。

**在上面的例子中是修改默认context的参数，当然我们也可以创建一个新的context(推荐方式)**,进行交替使用。
```

<?php
$alternative = stream_context_create($other_opts);
readfile('http://localhost/dev/streams/php_input.php', false, $alternative);

```


```

<?php
$requestBody = '{"username":"josh"}';
$context = stream_context_create(array(
    'http' => array(
        'method' => 'POST',
        'header' => "Content-Type: application/json;charset=utf-8;\r\n" .
                    "Content-Length: " . mb_strlen($requestBody),
        'content' => $requestBody
    )
));
$response = file_get_contents('https://my-api.com/users', false, $context);


```
##  例子 
```

<?php
/* Read local file from /home/bar */
$localfile = file_get_contents("/home/bar/foo.txt");

/* Identical to above, explicitly naming FILE scheme */
$localfile = file_get_contents("file:///home/bar/foo.txt");

/* Read remote file from www.example.com using HTTP */
$httpfile  = file_get_contents("http://www.example.com/foo.txt");

/* Read remote file from www.example.com using HTTPS */
$httpsfile = file_get_contents("https://www.example.com/foo.txt");

/* Read remote file from ftp.example.com using FTP */
$ftpfile   = file_get_contents("ftp://user:pass@ftp.example.com/foo.txt");

/* Read remote file from ftp.example.com using FTPS */
$ftpsfile  = file_get_contents("ftps://user:pass@ftp.example.com/foo.txt");
?>

```
```

Writing data to a compressed file
<?php
/* Create a compressed file containing an arbitrarty string
* File can be read back using compress.zlib stream or just
* decompressed from the command line using 'gzip -d foo-bar.txt.gz'
*/
$fp = fopen("compress.zlib://foo-bar.txt.gz", "wb");
if (!$fp) die("Unable to create file.");

fwrite($fp, "This is a test.\n");

fclose($fp);
?>

```
###  示例1-删除含有文件的文件夹
1.用` realpath函数 `得到真实的地址，如果地址等于空，等于/，或者地址等于:\\ 那么证明是根目录，不能删除，返回假
2.如果不等于1中的内容，那么使用` opendir函数打开目录句柄 `，返回一个**目录流**
` 用while (readdir())遍历目录 `，并且赋值给$file
3.如果$file不等于假的时候，**如果等于.或者..的时候，continue**
4.给$path赋值为 $dir（就是真实地址）连接接` DIRECTORY_SEPARATOR（系统分隔符） `连接$file. 这个地址为文件夹中文件的地址
5.当$path 为目录且` rmdir($path) `函数不为假时 就是当$path是目录但是不能删除时
` unlink($path); ` 删除这个文件。关闭句柄。再删除文件夹。

**但此方法只适用于文件夹中有文件的，文件夹的嵌套将不起作用**
```

<?  
header('Content-Type:text/html; charset=utf-8');  
 
function mrdirs($dir)  
{  
      
    $dir = realpath($dir);  
    if($dir=='' || $dir=='/' || (strlen($dir)==3 && substr($dir,1)==':\\'))  
    {  
        return false;  
    }  
    else 
    {  
        if(false != ($dh=opendir($dir)))  
        {  
            while(false != ($file=readdir($dh)))  
            {  
                if($file=='.' || $file=='..') {continue;}  
                echo $path=$dir .DIRECTORY_SEPARATOR . $file;  
                 if (is_dir($path))  
                 {  
                     if(!rmdir($path)){return false;}
                 }  
                 else 
                 {  
                         unlink($path);  
                         echo $path."文件以删除";  
                 }  
           }  
	    closedir($dh);  
	    rmdir($dir);  
	    echo "删除文件夹成功";  
	    return true;  
        }  
              
        else 
        {  
            return false;  
        }     
        }   
    }  
$dir=date('Y-m-d');  
if(file_exists($dir))  
{  
mrdirs($dir);  
}  
else 
{  
    echo "文件夹不存在，无法删除";  
}  
?> 

```

##  源代码加亮输出 
string show_source(you_file_path)
string highlight_file(you_file_path)
