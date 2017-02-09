title: phplearn05 

#  PHP学习之文件系统操作 
  * PHP中目录路径分隔符一律可以使用'/',例如'/opt/www','c:/windows'
  * DIRECTORY_SEPARATOR 路径分隔符，Win下是“\”而*inux下是“/“。
  * PATH_SEPARATOR 多个路劲分隔符，比如使用include多个路劲时候，Win下用”;”，而*inux下为“:”

###  文件系统操作总结 
http://php.net/manual/zh/ref.dir.php
  * basename()：返回路径中的文件
  * dirname()：返回路径中的目录
  * realpath() - 返回规范化的绝对路径名
  * pathinfo — 返回文件路径的信息
mixed pathinfo ( string $path [, int $options = PATHINFO_DIRNAME | PATHINFO_BASENAME | PATHINFO_EXTENSION | PATHINFO_FILENAME ] )
pathinfo() 返回一个关联数组包含有 path 的信息。返回关联数组还是字符串取决于 options。
```

<?php
$path_parts = pathinfo('/www/htdocs/inc/lib.inc.php');
echo $path_parts['dirname'], "\n";
echo $path_parts['basename'], "\n";
echo $path_parts['extension'], "\n";
echo $path_parts['filename'], "\n"; // since PHP 5.2.0
?>
以上例程会输出：
/www/htdocs/inc
lib.inc.php
php
lib.inc

```
0、is_dir() - 判断给定文件名是否是一个目录
1、unlink()删除文件，php中没有delete函数。失败时会产生一个 E_WARNING 级别的错误。
bool unlink ( string $filename [, resource $context ] )

2、rmdir — 删除目录。尝试删除 dirname 所指定的目录。 **该目录必须是空的**，而且要有相应的权限。 失败时会产生一个 E_WARNING 级别的错误。
bool rmdir ( string $dirname [, resource $context ] )

3、mkdir — 新建目录
bool mkdir ( string $pathname [, int $mode = 0777 [, bool $recursive = false [, resource $context ]]] )

4、copy — 拷贝文件，如果要移动文件的话，请使用 rename() 函数。
bool copy ( string $source , string $dest [, resource $context ] )

5、rename — 重命名一个文件或目录
bool rename ( string $oldname , string $newname [, resource $context ] )

6、目录操作补充
chdir — 改变目录
chroot — 改变根目录
**dir — 返回一个 Directory 类实例**
getcwd — 取得当前工作目录
opendir — 打开目录句柄
closedir — 关闭目录句柄
readdir — 从目录句柄中读取条目
rewinddir — 倒回目录句柄
scandir — 列出指定路径中的文件和目录
说明如下：
scandir — 列出指定路径中的文件和目录.但只是列出一级目录和文件，不会列出子文件夹里面的内容。
array scandir ( string $directory [, int $sorting_order [, resource $context ]] )
返回一个 array，包含有 directory 中的文件和目录。
opendir() - 打开目录句柄
打开一个目录句柄，可用于之后的 closedir()，readdir() 和 rewinddir() 调用中。
closedir() - 关闭目录句柄
readdir() - 从目录句柄中读取条目。
rewinddir()
string readdir ([ resource $dir_handle ] )
返回目录中**下一个文件**的文件名。文件名以在文件系统中的排序返回。
```

<?php
if ($handle = opendir('/path/to/files')) {
    echo "Directory handle: $handle\n";
    echo "Files:\n";

    /* 这是正确地遍历目录方法 */
    while (false !== ($file = readdir($handle))) {
        echo "$file\n";
    }

    /* 这是错误地遍历目录的方法 */
    while ($file = readdir($handle)) {
        echo "$file\n";
    }

    closedir($handle);
}
?>

```
```

<?php
$dir = "/etc/php5/";
// Open a known directory, read directory into variable and then close
if (is_dir($dir)) {
    if ($dh = opendir($dir)) {
        $directory = readdir($dh);
        closedir($dh);
    }
}
?>

```
glob() - 寻找与模式匹配的文件路径
 array glob ( string $pattern [, int $flags = 0 ] )
fnmatch — 用模式匹配文件名.目前该函数无法在 Windows 或其它非 POSIX 兼容的系统上使用。
 bool fnmatch ( string $pattern , string $string [, int $flags = 0 ] )  shell 模式其中最简单的形式 '?' 和 '*' 、[]通配符
```

<?php
if (fnmatch("*gr[ae]y", $color)) {
  echo "some form of gray ...";
}
?>

```

##  Directory类 
**Directory 实例是通过调用 dir() 函数创建的，而不是 new 操作符。**
```

Directory {
/* 属性 */
public string $path ;
public resource $handle ;
/* 方法 */
public void close ([ resource $dir_handle ] )
public string read ([ resource $dir_handle ] )
public void rewind ([ resource $dir_handle ] )
}

```
path被打开目录的地址
handle目录句柄。可以被其他的目录操作函数使用，例如 readdir(), rewinddir() and closedir()。
Directory::close — 释放目录句柄
Directory::read — 从目录句柄中读取条目
Directory::rewind — 倒回目录句柄
```

<?php
  $dir = dir("/uploads/");
  while(false !== ($file = $dir->read()))
    //strip out the two entries of . and ..
  	if($file != "." && $file != "..")
  	{
		echo '<a href="filedetails.php?file='.$file.'">'.$file.'</a><br>';
  	}
  echo '</ul>';
  $dir->close();
?>

```
等效于:
```

  $current_dir = '/uploads/';
  $dir = opendir($current_dir);
  while(false !== ($file = readdir($dir)))
    //strip out the two entries of . and ..
  	if($file != "." && $file != "..")
  	{
    	  echo "<li>$file</li>";
  	}
  echo '</ul>';
  closedir($dir);

```
###  遍历目录下所有文件的方法 
```

<?php 
/** 
* array( 
*   'files' => [], 
*   'dirs'  => [], 
* ) 
*/ 
function read_all_files($root = '.'){ 
  $files  = array('files'=>array(), 'dirs'=>array()); 
  $directories  = array(); 
  $last_letter  = $root[strlen($root)-1]; 
  $root  = ($last_letter == '\\' || $last_letter == '/') ? $root : $root.DIRECTORY_SEPARATOR; 
  $directories[]  = $root; 
  
  while (sizeof($directories)) { 
    $dir  = array_pop($directories); 
    if ($handle = opendir($dir)) { 
      while (false !== ($file = readdir($handle))) { 
        if ($file == '.' || $file == '..') { 
          continue; 
        } 
        $file  = $dir.$file; //字符串合并 
        if (is_dir($file)) { 
          $directory_path = $file.DIRECTORY_SEPARATOR; 
          array_push($directories, $directory_path); 
          $files['dirs'][]  = $directory_path; 
        } elseif (is_file($file)) { 
          $files['files'][]  = $file; 
        } 
      } 
      closedir($handle); 
    } 
  } 
  return $files; 
} 
?>

```
##  目录与路径 
PHP 目录处理函数主要包括：
  * mkdir()：创建目录
  * is_dir()：判断给定文件名是否是一个目录
  * rmdir()：删除目录
  * basename()：返回路径中的文件
  * dirname()：返回路径中的目录

**mkdir()**
mkdir() 函数用于创建一个目录，成功返回 TRUE，否则返回 FALSE 。
语法：bool mkdir( string dirname [, int mode ] )
可选参数 mode 表示创建该目录时给定的权限，默认是最大权限 0777 。

**is_dir()**
is_dir() 函数用于检查给定文件名是否是一个目录，成功返回 TRUE ，否则返回 FALSE 。
语法：bool is_dir( string filename )
参考阅读 is_file()：检查给定文件名是否为一个正常的文件。

**rmdir()**
rmdir() 函数删除一个目录，成功返回 TRUE，否则返回 FALSE 。不能删除非空目录
语法：bool rmdir( string dirname )
参考阅读 unlink()：删除文件。

**basename()**
basename() 函数用于返回一个包含全路径的字符串中的**基本文件名**，成功返回字符串，否则返回 FALSE 。
语法：string basename( string path [, string suffix] ) 
可选参数 suffix 表示文件后缀，如果文件名后缀是 suffix ，那这一部分也会被去掉。
假定本地访问该文件 URL 地址为：http://127.0.0.1/html/test.php
```

<?php
echo $PHP_SELF;				//输出：/html/test.php
echo basename( $PHP_SELF );		//输出：test.php
echo basename( $PHP_SELF, '.php');	//输出：test
?>

```
本函数与 dirname() 函数经常用于 URL 处理。

**dirname()**
dirname() 函数用于返回一个包含全路径的字符串中去掉文件名的目录，成功返回字符串，否则返回 FALSE 。
语法：string dirname( string path ) 
例子：
假定本地访问该文件 URL 地址为：http://127.0.0.1/html/test.php
```

<?php
echo $PHP_SELF;		//输出 /html/test.php
echo dirname( $PHP_SELF );	//输出 /html
?>

```
本函数与 basename() 函数经常用于 URL 处理。

##  文件操作(非读写) 
检查文件是否存在
  * file_exists：检查文件或目录是否存在。bool file_exists( string filename )
检查文件是否可读写执行
  * is_readable：检查文件是否可读。
  * is_writable：检查文件是否是否可写入。
  * is_executable：检查文件是否可执行。
文件拷贝
  * copy：拷贝文件。
文件删除
  * unlink：删除文件。
取得文件大小、类型、修改时间信息
  * filesize：取得文件大小。
  * filetype：取得文件类型。
  * filemtime：取得文件修改时间。
其他
  * is_file() 函数用于检查给定文件名是否为一个正常的文件

##  附录：文件系统操作函数大全 
http://php.net/manual/zh/ref.filesystem.php
basename — 返回路径中的文件名部分
chgrp — 改变文件所属的组
chmod — 改变文件模式
chown — 改变文件的所有者
clearstatcache — 清除文件状态缓存
copy — 拷贝文件
unlink-删除文件
dirname — 返回路径中的目录部分
**disk_free_space — 返回目录中的可用空间
disk_total_space — 返回一个目录的磁盘总大小**
diskfreespace — disk_free_space 的别名
fclose — 关闭一个已打开的文件指针
feof — 测试文件指针是否到了文件结束的位置
fflush — 将缓冲内容输出到文件
fgetc — 从文件指针中读取字符
fgetcsv — 从文件指针中读入一行并解析 CSV 字段
fgets — 从文件指针中读取一行
fgetss — 从文件指针中读取一行并过滤掉 HTML 标记
file_exists — 检查文件或目录是否存在
file_get_contents — 将整个文件读入一个字符串
file_put_contents — 将一个字符串写入文件
file — 把整个文件读入一个数组中
fileatime — 取得文件的上次访问时间
filectime — 取得文件的 inode 修改时间
filegroup — 取得文件的组
fileinode — 取得文件的 inode
**filemtime — 取得文件修改时间**
fileowner — 取得文件的所有者
fileperms — 取得文件的权限
filesize — 取得文件大小
**filetype — 取得文件类型
flock — 轻便的咨询文件锁定**
fnmatch — 用模式匹配文件名
fopen — 打开文件或者 URL
fpassthru — 输出文件指针处的所有剩余数据
fputcsv — 将行格式化为 CSV 并写入文件指针
fputs — fwrite 的别名
fread — 读取文件（可安全用于二进制文件）
fscanf — 从文件中格式化输入
fseek — 在文件指针中定位
fstat — 通过已打开的文件指针取得文件信息
ftell — 返回文件指针读/写的位置
ftruncate — 将文件截断到给定的长度
fwrite — 写入文件（可安全用于二进制文件）
glob — 寻找与模式匹配的文件路径
is_dir — 判断给定文件名是否是一个目录
is_executable — 判断给定文件名是否可执行
is_file — 判断给定文件名是否为一个正常的文件
is_link — 判断给定文件名是否为一个符号连接
is_readable — 判断给定文件名是否可读
is_uploaded_file — 判断文件是否是通过 HTTP POST 上传的
is_writable — 判断给定的文件名是否可写
is_writeable — is_writable 的别名
lchgrp — Changes group ownership of symlink
lchown — Changes user ownership of symlink
link — 建立一个硬连接
linkinfo — 获取一个连接的信息
lstat — 给出一个文件或符号连接的信息
mkdir — 新建目录
**move_uploaded_file — 将上传的文件移动到新位置**
parse_ini_file — 解析一个配置文件
parse_ini_string — Parse a configuration string
pathinfo — 返回文件路径的信息
pclose — 关闭进程文件指针
popen — 打开进程文件指针
readfile — 输出文件
readlink — 返回符号连接指向的目标
realpath_cache_get — Get realpath cache entries
realpath_cache_size — Get realpath cache size
**realpath — 返回规范化的绝对路径名**
rename — 重命名一个文件或目录
rewind — 倒回文件指针的位置
rmdir — 删除目录
set_file_buffer — stream_set_write_buffer 的别名
**stat — 给出文件的信息**
symlink — 建立符号连接
tempnam — 建立一个具有唯一文件名的文件
**tmpfile — 建立一个临时文件**
touch — 设定文件的访问和修改时间
umask — 改变当前的 umask
unlink — 删除文件