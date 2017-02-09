title: phplearn9 

#  PHP学习之错误处理与异常 
PHP调试手册下载http://vdisk.weibo.com/s/9Q0zX
##  错误抑制符@ 
```

$handler=@ fopen("http://wiki.xby1993.net")
@ $handler=fopen("http://wiki.xby1993.net")

```
##  PHP 错误处理 
**在 PHP 中，默认的错误处理很简单。一条消息会被发送到浏览器，这条消息带有文件名、行号以及一条描述错误的消息。**

错误配置php.ini:
```

;显示错误
display_startup_errors=On
display_errors=On
;报告所有错误
error_reporting=-1
;记录错误
log_errors=On

```

生产环境配置如下:
```

;不显示错误
display_startup_errors=Off
display_errors=Off
;除了注意事项之外，报告所有其他错误
error_reporting=E_ALL & ~E_NOTICE
;记录错误
log_errors=On

```
###  简单的 "die()" 语句 
基本的错误处理：使用 ` die() 函数 `
第一个例子展示了一个打开文本文件的简单脚本：
```

<?php
$file=fopen("welcome.txt","r");
?>

```
如果文件不存在，您会获得类似这样的错误：
Warning: fopen(welcome.txt) [function.fopen]: failed to open stream: 
No such file or directory in C:\webfolder\test.php on line 2
为了避免用户获得类似上面的错误消息，我们在访问文件之前检测该文件是否存在：
```

<?php
if(!file_exists("welcome.txt"))
 {
 die("File not found");
 }
else
 {
 $file=fopen("welcome.txt","r");
 }
?>

```
现在，假如文件不存在，您会得到类似这样的错误消息：
File not found
比起之前的代码，上面的代码更有效，**这是由于它采用了一个简单的错误处理机制在错误之后终止了脚本。**
不过，**简单地终止脚本并不总是恰当的方式。**让我们研究一下用于处理错误的备选的 PHP 函数。

###  error_reporting 
` error_reporting() 函数 `能够在运行时设置 error_reporting 指令。 PHP 有诸多错误级别，使用该函数可以设置在脚本运行时的级别。 如果没有设置可选参数 level， error_reporting() 仅会返回当前的错误报告级别。
```

<?php

// 关闭所有PHP错误报告
error_reporting(0);

// Report simple running errors
error_reporting(E_ERROR | E_WARNING | E_PARSE);

// 报告 E_NOTICE也挺好 (报告未初始化的变量
// 或者捕获变量名的错误拼写)
error_reporting(E_ERROR | E_WARNING | E_PARSE | E_NOTICE);

// 除了 E_NOTICE，报告其他所有错误
// 这是在 php.ini 里的默认设置
error_reporting(E_ALL ^ E_NOTICE);

// 报告所有 PHP 错误 (参见 changelog)
error_reporting(E_ALL);

// 报告所有 PHP 错误
error_reporting(-1);

// 和 error_reporting(E_ALL); 一样
ini_set('error_reporting', E_ALL);

?>

```
###  创建自定义错误处理器 
创建一个自定义的错误处理器非常简单。我们很简单地创建了一个专用函数，可以在 PHP 中发生错误时调用该函数。
**该函数必须有能力处理至少两个参数 (error level 和 error message)，但是可以接受最多五个参数（可选的：file, line-number 以及 error context）**：
**error_function(error_level,error_message,error_file,error_line,error_context)**
现在，让我们创建一个处理错误的函数：
```

function customError($errno, $errstr,$errfile,$errline)
 { 
 echo "<b>Error:</b> [$errno] $errstr<br />";
 echo "Ending Script";
 die();
 }

```
上面的代码是一个简单的错误处理函数。当它被触发时，它会取得错误级别和错误消息。然后它会输出错误级别和消息，并终止脚本。
现在，我们已经创建了一个错误处理函数，我们需要确定在何时触发该函数。
**Set Error Handler**
PHP 的默认错误处理程序是内建的错误处理程序。我们打算把上面的函数改造为脚本运行期间的默认错误处理程序。
可以修改错误处理程序，使其仅应用到某些错误，这样脚本就可以不同的方式来处理不同的错误。不过，在本例中，我们打算针对所有错误来使用我们的自定义错误处理程序：
**set_error_handler("customError");**
由于我们希望我们的自定义函数来处理所有错误，set_error_handler() 仅需要一个参数，**可以添加第二个参数来规定错误级别。**
通过尝试输出不存在的变量，来测试这个错误处理程序：
```

<?php
set_error_handler(function customError($errno, $errstr,$errfile,$errline)
 { 
 echo "<b>Error:</b> [$errno] $errstr";
 });

//trigger error
echo($test);
?>

```
以上代码的输出应该类似这样：
Error: [8] Undefined variable: test
###  触发错误 
在脚本中用户输入数据的位置，当用户的输入无效时触发错误的很有用的。在 PHP 中，这个任务由 ` trigger_error() ` 完成。
在本例中，如果 "test" 变量大于 "1"，就会发生错误：
```

<?php
$test=2;
if ($test>1)
{
trigger_error("Value must be 1 or below");
}
?>

```
以上代码的输出应该类似这样：
Notice: Value must be 1 or below
in C:\webfolder\test.php on line 6
您可以在脚本中任何位置触发错误，通过添加的第二个参数，您能够规定所触发的错误级别。
可能的错误类型：
  * E_USER_ERROR - 致命的用户生成的 run-time 错误。错误无法恢复。脚本执行被中断。
  * E_USER_WARNING - 非致命的用户生成的 run-time 警告。脚本执行不被中断。
  * E_USER_NOTICE - 默认。用户生成的 run-time 通知。脚本发现了可能的错误，也有可能在脚本运行正常时发生。

###  错误记录 
**默认地，根据在 php.ini 中的 error_log 配置，PHP 向服务器的错误记录系统或文件发送错误记录。**通过使用 ` error_log() 函数 `，您可以向指定的文件或远程目的地发送错误记录。
通过电子邮件向您自己发送错误消息，是一种获得指定错误的通知的好办法。
**通过 E-Mail 发送错误消息**
在下面的例子中，如果特定的错误发生，我们将发送带有错误消息的电子邮件，并结束脚本：
```

<?php
//error handler function
function customError($errno, $errstr)
 { 
 echo "<b>Error:</b> [$errno] $errstr<br />";
 echo "Webmaster has been notified";
 error_log("Error: [$errno] $errstr",1,"someone@example.com","From: webmaster@example.com");
}

//set error handler
set_error_handler("customError",E_USER_WARNING);

//trigger error
$test=2;
if ($test>1)
 {
 trigger_error("Value must be 1 or below",E_USER_WARNING);
 }
?>

```
以上代码的输出应该类似这样：
Error: [512] Value must be 1 or below
Webmaster has been notified
接收自以上代码的邮件类似这样：
Error: [512] Value must be 1 or below
这个方法不适合所有的错误。常规错误应当通过使用默认的 PHP 记录系统在服务器上进行记录。

##  异常处理 
http://www.php.net/manual/zh/language.exceptions.php
PHP 5 提供了一种新的面向对象的错误处理方法。
**异常处理用于在指定的错误（异常）情况发生时改变脚本的正常流程。这种情况称为异常。**
当异常被触发时，通常会发生：
  * 当前代码状态被保存
  * 代码执行被切换到预定义的异常处理器函数
  * 根据情况，处理器也许会从保存的代码状态重新开始执行代码，终止脚本执行，或从代码中另外的位置继续执行脚本
我们将展示不同的错误处理方法：
  * 异常的基本使用
  * 创建自定义的异常处理器
  * 多个异常
  * 重新抛出异常
  * 设置顶层异常处理器
当异常被抛出时，其后的代码不会继续执行，PHP 会尝试查找匹配的 "catch" 代码块。
如果异常没有被捕获，而且又没用使用 set_exception_handler() 作相应的处理的话，那么将发生一个严重的错误（致命错误），并且输出 "Uncaught Exception" （未捕获异常）的错误消息。
**try, throw 和 catch、finally**
要避免上面例子出现的错误，我们需要创建适当的代码来处理异常。
正确的处理程序应当包括：
  * try - 使用异常的函数应该位于 "try" 代码块内。如果没有触发异常，则代码将照常继续执行。但是如果异常被触发，会抛出一个异常。
  * throw - 这里规定如何触发异常。每一个 "throw" 必须对应至少一个 "catch"
  * catch - "catch" 代码块会捕获异常，并创建一个包含异常信息的对象
  * finally - PHP5.5之后版本可用。
```

<?php
//创建可抛出一个异常的函数
function checkNum($number)
 {
 if($number>1)
  {
  throw new Exception("Value must be 1 or below");
  }
 return true;
 }

//在 "try" 代码块中触发异常
try
 {
 checkNum(2);
 //If the exception is thrown, this text will not be shown
 echo 'If you see this, the number is 1 or below';
 }finally{

 }

//捕获异常
catch(Exception $e)
 {
 echo 'Message: ' .$e->getMessage();
 }
?>

```
不过，为了遵循“每个 throw 必须对应一个 catch”的原则，可以**设置一个顶层的异常处理器来处理漏掉的错误**。
###  创建一个自定义的 Exception 类 
http://php.net/manual/zh/language.exceptions.extending.php
内置Exception类代码如下:
```

<?php
class Exception
{
    protected $message = 'Unknown exception';   // 异常信息
    private   $string;                          // __toString cache
    protected $code = 0;                        // 用户自定义异常代码
    protected $file;                            // 发生异常的文件名
    protected $line;                            // 发生异常的代码行号
    private   $trace;                           // backtrace
    private   $previous;                        // previous exception if nested exception

    public function __construct($message = null, $code = 0, Exception $previous = null);

    final private function __clone();           // Inhibits cloning of exceptions.

    final public  function getMessage();        // 返回异常信息
    final public  function getCode();           // 返回异常代码
    final public  function getFile();           // 返回发生异常的文件名
    final public  function getLine();           // 返回发生异常的代码行号
    final public  function getTrace();          // backtrace() 数组
    final public  function getPrevious();       // 之前的 exception
    final public  function getTraceAsString();  // 已格成化成字符串的 getTrace() 信息

    // Overrideable
    public function __toString();               // 可输出的字符串
}
?>

```
这个自定义的 exception 类继承了 PHP 的 Exception 类的所有属性，您可向其添加自定义的函数。
我们开始创建 exception 类：` 只有_ _toString()方法是可以被覆盖的。 `
如果使用自定义的类来扩展内置异常处理类，并且要重新定义构造函数的话，建议同时调用 ` parent::_ _construct() ` 来检查所有的变量是否已被赋值。当对象要输出字符串的时候，可以重写`  _ _toString() ` 并自定义输出的样式。
```

<?php
/**
 * 自定义一个异常处理类
 */
class MyException extends Exception
{
    // 重定义构造器使 message 变为必须被指定的属性
    public function __construct($message, $code = 0, Exception $previous = null) {
        // 自定义的代码

        // 确保所有变量都被正确赋值
        parent::__construct($message, $code, $previous);
    }

    // 自定义字符串输出的样式
    public function __toString() {
        return __CLASS__ . ": [{$this->code}]: {$this->message}\n";
    }

    public function customFunction() {
        echo "A custom function for this type of exception\n";
    }
}

/**
 * 创建一个用于测试异常处理机制的类
 */
class TestException
{
    public $var;

    const THROW_NONE    = 0;
    const THROW_CUSTOM  = 1;
    const THROW_DEFAULT = 2;

    function __construct($avalue = self::THROW_NONE) {

        switch ($avalue) {
            case self::THROW_CUSTOM:
                // 抛出自定义异常
                throw new MyException('1 is an invalid parameter', 5);
                break;

            case self::THROW_DEFAULT:
                // 抛出默认的异常
                throw new Exception('2 is not allowed as a parameter', 6);
                break;

            default: 
                // 没有异常的情况下，创建一个对象
                $this->var = $avalue;
                break;
        }
    }
}


// 例子 1
try {
    $o = new TestException(TestException::THROW_CUSTOM);
} catch (MyException $e) {      // 捕获异常
    echo "Caught my exception\n", $e;
    $e->customFunction();
} catch (Exception $e) {        // 被忽略
    echo "Caught Default Exception\n", $e;
}

```

###  设置顶层异常处理器 （Top Level Exception Handler） 
` set_exception_handler() ` 函数可设置处理所有未捕获异常的用户定义函数。
```

<?php
function myException($exception)
{
echo "<b>Exception:</b> " , $exception->getMessage();
}
set_exception_handler('myException');
throw new Exception('Uncaught Exception occurred');
?>

```
以上代码的输出应该类似这样：
Exception: Uncaught Exception occurred
在上面的代码中，不存在 "catch" 代码块，而是触发顶层的异常处理程序。应该使用此函数来捕获所有未被捕获的异常。
###  异常的规则 
  * 需要进行异常处理的代码应该放入 try 代码块内，以便捕获潜在的异常。
  * 每个 try 或 throw 代码块必须至少拥有一个对应的 catch 代码块。
  * 使用多个 catch 代码块可以捕获不同种类的异常。
  * 可以在 try 代码块内的 catch 代码块中再次抛出（re-thrown）异常。
**简而言之：如果抛出了异常，就必须捕获它。**

##  开发环境友好显示错误和异常 
可以使用Whoops（https://github.com/filp/whoops）组件。
使用Composer安装:
```

composer require filp/whoops

```
注册Whoops提供的处理程序
```

<?php
// Use composer autoloader
require 'vendor/autoload.php';

// Setup Whoops error and exception handlers
$whoops = new \Whoops\Run;
$whoops->pushHandler(new \Whoops\Handler\PrettyPageHandler);
$whoops->register();

throw new \Exception('This is an exception!');


```
##  PHP7中的异常与错误处理改进 
**一、现在有两个异常类：Exception and Error.**
PHP7现在有两个异常类，` Exception and Error `。这两个类都实现了一个新的接口：` Throwable `。在您的异常处理代码中，类型暗示可能需要调整下。

二、一些致命错误和不可恢复致命错误改为抛出**Error对象。**
有一些致命错误和可恢复致命错误现在改为报出Error对象。Error对象是和Exception独立的，它们无法被常规的try/catch扑获。编者按：需要注册错误处理函数，请参考下面的RFC。
对于这些已经转为异常的可恢复致命错误，已经无法通过error handler静默的忽略掉。尤其是无法忽略类型暗示错误。

三、语法错误会抛出一个**ParseError对象**
语法错误会抛出一个ParseError对象，该对象继承自Error对象。之前处理eval()的时候，对于潜在可能错误的代码除了检查返回值或者error_get_last()之外，还应该捕获ParseError对象。

四、内部对象的构造方法如果失败的时候总会抛出异常
内部对象的构造方法如果失败的时候总会报出异常。之前的有一些构造方法会返回NULL或者一个无法使用的对象。
五、一些E_STRICT错误的级别调整了。
六、参考资料
https://wiki.php.net/rfc/engine_exceptions_for_php7
https://wiki.php.net/rfc/throwable-interface
https://wiki.php.net/rfc/internal_constructor_behaviour
https://wiki.php.net/rfc/reclassify_e_strict