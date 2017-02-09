title: phplearn22 

#  PHP学习之服务器变量与环境 
##  与系统环境变量交互 
getenv()与putenv()
运行phpinfo()函数可以获取PHP所有环境变量的列表。
##  动态修改ini配置与include_path 
ini_get — 获取一个配置选项的值
ini_set() - 为一个配置选项设置值
get_cfg_var() - 获取 PHP 配置选项的值
ini_get_all() - 获取所有配置选项
ini_restore() - 恢复配置选项的值

**include和require语句包含并运行指定文件的顺序:**
1、被包含文件先按参数给出的路径寻找，
2、如果没有给出目录（只有文件名）时则按照 ` include_path 指定的目录寻找 `。
3、如果在 include_path 下没找到该文件则 include 最后才在调用脚本文件所在的目录和当前工作目录下寻找。
4、如果最后include仍未找到文件则 include 结构会发出一条警告；这一点和 require 不同，后者会发出一个致命错误。
**如果定义了路径——不管是绝对路径（在 Windows 下以盘符或者 \ 开头，在 Unix/Linux 下以 / 开头）还是当前目录的相对路径（以 . 或者 .. 开头）并且该路径下文件存在。——include_path 都会被完全忽略。**例如一个文件以 ../ 开头，则解析器会在当前目录的父目录下寻找该文件。
当一个文件被包含时，其中所包含的代码继承了 include 所在行的变量范围。从该处开始，调用文件在该行处可用的任何变量在被调用的文件中也都可用。不过所有在包含文件中定义的函数和类都具有全局作用域。

get_include_path — 获取当前的 include_path 配置选项
restore_include_path() - 还原 include_path 配置选项的值
set_include_path() - 设置 include_path 配置选项
##  PHP $_SERVER 变量 
$_SERVER 是一个包含诸如头信息（header）、路径（path）和脚本位置（script locations）的数组。它是 PHP 中一个超级全局变量，我们可以在 PHP 程序的任何地方直接访问它。
$_SERVER 包含着众多的信息，你可以尝试直接打印它：
print_r($_SERVER);

**页面程序相关**
  * ` $_SERVER['PHP_SELF'] `：相对于网站根目录的路径及 PHP 程序名称，与 document root 相关。
  * ` $_SERVER['HTTP_REFERER'] `：链接到当前页面的前一页面的 URL 地址。
  * ` $_SERVER['SCRIPT_NAME'] `：相对于网站根目录的路径及 PHP 程序文件名称 。
  * $_SERVER['REQUEST_URI']：访问此页面所需的 URI 。
  * ` $_SERVER['SCRIPT_FILENAME'] `：当前运行 PHP 程序的绝对路径及文件名。
  * $_SERVER['PATH_TRANSLATED']：当前 PHP 程序所在文件系统（不是文档根目录）的基本路径。
  * $_SERVER['QUERY_STRING']：查询（query）的字符串（URL 中第一个问号 ? 之后的内容但不包括 # 后面的内容）。
  * $_SERVER['argv']：传递给当前 PHP 程序的参数。
  * $_SERVER['argc']：命令行模式下，包含传递给程序的命令行参数的个数。
  * $_SERVER['REQUEST_TIME']：请求开始时的时间戳，从 PHP 5.1.0 起有效。
  * ` $_SERVER['REQUEST_METHOD'] `：访问页面时的请求方法，例如：“GET”、“HEAD”，“POST”或“PUT”。
  * $_SERVER['HTTP_ACCEPT']：当前请求的 Accept: 头信息的内容。
  * $_SERVER['HTTP_ACCEPT_CHARSET']：当前请求的 Accept-Charset: 头信息的内容。例如：“iso-8859-1,*,utf-8”。
  * $_SERVER['HTTP_ACCEPT_ENCODING']：当前请求的 Accept-Encoding: 头信息的内容。例如：“gzip”。
  * $_SERVER['HTTP_ACCEPT_LANGUAGE']：当前请求的 Accept-Language: 头信息的内容。例如：“zh-cn”。
  * $_SERVER['HTTP_CONNECTION']：当前请求的 Connection: 头信息的内容。例如：“Keep-Alive”。
  * $_SERVER['HTTP_HOST']：当前请求的 Host: 头信息的内容。
  * $_SERVER['HTTPS']：如果 PHP 程序是通过 HTTPS 协议被访问，则被设为一个非空的值。
  * $_SERVER['PHP_AUTH_DIGEST']：当作为 Apache 模块运行时，进行 HTTP Digest 认证的过程中，此变量被设置成客户端发送的“Authorization”HTTP 头内容（以便作进一步的认证操作）。
  * $_SERVER['PHP_AUTH_USER']：当 PHP 运行在 Apache 或 IIS（PHP 5 是 ISAPI）模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的用户名。
  * $_SERVER['PHP_AUTH_PW']：当 PHP 运行在 Apache 或 IIS（PHP 5 是 ISAPI）模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的密码。
  * $_SERVER['AUTH_TYPE']：当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是认证的类型。

**服务器端相关**
  * ` $_SERVER['DOCUMENT_ROOT'] `：当前运行 PHP 程序所在的文档根目录，在服务器配置文件中定义。
  * $_SERVER['GATEWAY_INTERFACE']：服务器使用的 CGI 规范的版本，例如：“CGI/1.1”。
  * ` $_SERVER['SERVER_ADDR'] `：当前运行 PHP 程序所在的服务器的 IP 地址。
  * ` $_SERVER['SERVER_NAME'] `：当前运行 PHP 程序所在的服务器的名称。
  * $_SERVER['SERVER_ADMIN']：Apache 服务器配置文件中的 SERVER_ADMIN 参数。
  * ` $_SERVER['SERVER_PORT'] `：服务器所使用的端口。如果使用 SSL 安全连接，则这个值为用户设置的 HTTP 端口。
  * $_SERVER['SERVER_SIGNATURE']：包含服务器版本和虚拟主机名的字符串。
  * $_SERVER['SERVER_SOFTWARE']：服务器标识的字串，在响应请求时的头信息中给出。
  * $_SERVER['SERVER_PROTOCOL']：请求页面时通信协议的名称和版本，例如：“HTTP/1.0”。

**客户端相关**
  * ` $_SERVER['HTTP_USER_AGENT'] `：当前请求的 User-Agent: 头信息的内容，该字符串表明了访问该页面的用户代理的信息。
  * ` $_SERVER['REMOTE_ADDR'] `：正在浏览当前页面用户的 IP 地址。
  * ` $_SERVER['REMOTE_HOST'] `：正在浏览当前页面用户的主机名。
  * ` $_SERVER['REMOTE_PORT'] `：用户连接到服务器时所使用的端口。

##  PHP $_ENV 变量 
**$_ENV 是一个包含服务器端环境变量的数组。**它是 PHP 中一个超级全局变量，我们可以在 PHP 程序的任何地方直接访问它。
$_ENV 只是被动的接受服务器端的环境变量并把它们转换为数组元素，你可以尝试直接打印它：
print_r($_ENV);
限于篇幅，在此不再列出打印的结果，且不同的服务器上，打印出的结果可能是完全不同的。
$_ENV 数组中的元素（数组单元）随服务器环境不同而有较大差异，所以无法像 $_SERVER 那样列出完整的列表。以下是 $_ENV 数组包含的比较通用的元素：
  * $_SERVER['PATH']：环境变量 PATH 路径。
  * $_SERVER['CLASSPATH']：系统 CLASSPATH 路径。
  * $_SERVER['LIB']：系统 LIB 库路径。
  * $_SERVER['INCLUDE']：系统 Include 路径，注意与 PHP 的包含路径是不一样的。
  * $_SERVER['OS']：操作系统类型。
  * $_SERVER['LANG']：系统语言，如 en_US 或 zh_CN。
  * $_SERVER['PWD']：当前工作目录。
  * ` $_SERVER['TEMP'] `：系统 TEMP 路径。
  * $_SERVER['AP_PARENT_PID']：当前进程 ID 号。
  * $_SERVER['NUMBER_OF_PROCESSORS']：系统 CPU 数目。
**$_ENV 为空的原因及解决办法**
如果打印输出 $_ENV 为空，可以检查一下 php.ini 的配置：
variables_order = "EGPCS"
上述配置表示 PHP 接受的外部变量来源及顺序，EGPCS 是 Environment、Get、Post、Cookies 和 Server 的缩写。如果 variables_order 的配置中缺少 E ，则 PHP 无法接受环境变量，那么 $_ENV 也就为空了。
##  使用 $_SERVER['PHP_SELF'] 获取当前页面地址及其安全性问题 
假设我们有如下网址，$_SERVER['PHP_SELF']得到的结果分别为：
http://www.5idev.com/php/ ：/php/index.php
http://www.5idev.com/php/index.php ：/php/index.php
http://www.5idev.com/php/index.php?test=foo ：/php/index.php
http://www.5idev.com/php/index.php/test/foo ：/php/index.php/test/foo
因此，可以使用 $_SERVER['PHP_SELF'] 很方便的获取当前页面的地址：
```

$url = "http://".$_SERVER['HTTP_HOST'].$_SERVER['PHP_SELF'];

```
以上面的地址为例，得到的结果如下：
http://www.5idev.com/php/index.php
上面是简单获取 http 协议的当前页面 URL ，只是要注意该地址是不包含 URL 中请求的参数（?及后面的字串）的。如果希望得到包含请求参数的完整 URL 地址，请使用 $_SERVER['REQUEST_URI'] 。

` PHP $_SERVER['PHP_SELF'] 安全性 `
由于利用 $_SERVER['PHP_SELF'] 可以很方便的获取当前页面地址，因此一些程序员在提交表单数据到当前页面进行处理时，往往喜欢使用如下这种方式：
```

<form method="post" action="<?php echo $_SERVER['PHP_SELF']; ?>">

```
假设该页面地址为：
http://www.5idev.com/php/index.php
访问该页面，得到的表单 html 代码如下：
<form method="post" action="/php/index.php">
显然这段代码不是我们期望的，**攻击者可以在 URL 后面随意加上攻击代码**。要解决该问题，可以：
` 使用 htmlspecialchars($_SERVER['PHP_SELF']) 替代 $_SERVER['PHP_SELF'] `，让 URL 中可能的恶意代码转换为用于显示的 html 代码而无法执行。
**可以的条件下，使用 $_SERVER['SCRIPT_NAME'] 或 $_SERVER['REQUEST_URI'] 替代 $_SERVER['PHP_SELF']**
在公共代码里将 $_SERVER['PHP_SELF'] 进行重写：
```

$phpfile = basename(__FILE__);
$_SERVER['PHP_SELF'] = substr($_SERVER['PHP_SELF'], 0, strpos($_SERVER['PHP_SELF'], $phpfile)).$phpfile;

``` 

##  PHP $_SERVER['PHP_SELF']、$_SERVER['SCRIPT_NAME'] 与 $_SERVER['REQUEST_URI']区别 
` $_SERVER['PHP_SELF']、$_SERVER['SCRIPT_NAME'] 与 $_SERVER['REQUEST_URI'] ` 三者非常相似，返回的都是与当前 URL 或 PHP 程序文件相关的信息：
  * $_SERVER['PHP_SELF']：相对于网站根目录的路径及 PHP 程序名称。
  * $_SERVER['SCRIPT_NAME']：相对于网站根目录的路径及 PHP 程序文件名称。
  * $_SERVER['REQUEST_URI']：访问此页面所需的 URI 。
一个简单的例子可以看出它们的区别。URL 地址如下：
```

http://www.5idev.com/php/index.php/test/foo?username=hbolive
$_SERVER['PHP_SELF'] 得到：/php/index.php/test/foo
$_SERVER['SCRIPT_NAME'] 得到：/php/index.php
$_SERVER['REQUEST_URI'] 得到：/php/index.php/test/foo?username=hbolive

```
从该例子可以看出:
$_SERVER['PHP_SELF'] 则反映的是 PHP 程序本身；
$_SERVER['SCRIPT_NAME'] 反映的是程序文件本身（这在页面需要指向自己时非常有用）；
$_SERVER['REQUEST_URI'] 则反映了完整 URL 地址（不包括主机名）。
其实从各自的命名上，也可以体现出它们之间的细微差别。
特别的，对于如下地址：
```

http://www.5idev.com/
$_SERVER['PHP_SELF'] 得到：/index.php
$_SERVER['SCRIPT_NAME'] 得到：/index.php
$_SERVER['REQUEST_URI'] 得到：/

```

##  $_SERVER['SCRIPT_FILENAME'] 与 __FILE__区别 
通常情况下，PHP $_SERVER['SCRIPT_FILENAME'] 与 __FILE__ 都会返回 PHP 文件的完整路径（绝对路径）与文件名：
```

<?php
echo 'SCRIPT_FILENAME 为：',$_SERVER['SCRIPT_FILENAME'];
echo '<br />';
echo '__FILE__ 为：',__FILE__;
?>

```
上述测试代码拷贝至 test.php 并访问该文件（http://127.0.0.1/php/test.php），得到如下结果：
  * SCRIPT_FILENAME 为：E:/web/html/php/test.php
  * __FILE__ 为：E:\web\html\php\test.php 
提示：在 windows 平台测试，得到结果如上所示可能会出现路径分隔符的细微差别。
**尽管 $_SERVER['SCRIPT_FILENAME'] 与 __FILE__ 非常相似，但在文件被 include 或 require 包含的时候，二者还是有细微区别。**
将上述测试代码拷贝至 E:\web\html\php\common\inc.php ，然后在刚才的 test.php 文件内包含 inc.php ：
```

<?php
include 'common/inc.php';
?>

```
这时候再访问 test.php 文件时，输出结果：
  * SCRIPT_FILENAME 为：E:/web/html/php/test.php
  * __FILE__ 为：E:\web\html\php\common\inc.php 
` 可见二者的差别是：$_SERVER['SCRIPT_FILENAME'] 反映的是当前执行程序的绝对路径及文件名；__FILE__ 反映的是原始文件（被包含文件）的绝对路径及文件名。 `
参考http://www.5idev.com/p-php_server.shtml

##  php中global与$GLOBAL[' ']的区别（重要） 
http://blog.csdn.net/yanhui_wei/article/details/8246083
很多人都认为global和$GLOBALS[]只是写法上面的差别，其实不然。
根据官方的解释是
$GLOBALS['var'] 是外部的全局变量$var本身。
global $var 是外部$var的同名引用。（是个别名引用而已，非指针！！！）
