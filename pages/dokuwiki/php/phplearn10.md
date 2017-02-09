title: phplearn10 

#  PHP学习之过滤器 
什么是 PHP 过滤器？
  * PHP 过滤器用于验证和过滤来自非安全来源的数据。
  * 验证和过滤用户输入或自定义数据是任何 Web 应用程序的重要组成部分。
  * 设计 PHP 的过滤器扩展的目的是使数据过滤更轻松快捷。
什么是外部数据？
  * 来自表单的输入数据
  * Cookies
  * 服务器变量
  * 数据库查询结果
##  函数和过滤器 
如需过滤变量，请使用下面的过滤器函数之一：
  * filter_var() - 通过一个指定的过滤器来过滤单一的变量
  * filter_var_array() - 通过相同的或不同的过滤器来过滤多个变量
  * filter_input - 获取一个输入变量，并对它进行过滤
  * filter_input_array - 获取多个输入变量，并通过相同的或不同的过滤器对它们进行过滤
在下面的例子中，我们用 filter_var() 函数验证了一个整数：
```

<?php
$int = 123;
if(!filter_var($int, FILTER_VALIDATE_INT))
 {
 echo("Integer is not valid");
 }
else
 {
 echo("Integer is valid");
 }
?>

```
上面的代码使用了 "FILTER_VALIDATE_INT" 过滤器来过滤变量。由于这个整数是合法的，因此代码的输出是："Integer is valid"。
假如我们尝试使用一个非整数的变量，则输出是："Integer is not valid"。
如需完整的函数和过滤器列表，请访问我们的 [PHP Filter 参考手册](http://www.w3school.com.cn/php/php_ref_filter.asp)。
##  PHP Filter 函数 
PHP：指示支持该函数的最早的 PHP 版本。
函数	描述	PHP
  * filter_has_var()	检查是否存在指定输入类型的变量。	5
  * filter_id()	返回指定过滤器的 ID 号。	5
  * filter_input()	从脚本外部获取输入，并进行过滤。	5
  * filter_input_array()	从脚本外部获取多项输入，并进行过滤。	5
  * filter_list()	返回包含所有得到支持的过滤器的一个数组。	5
  * filter_var_array()	获取多项变量，并进行过滤。	5
  * filter_var()	获取一个变量，并进行过滤。	5
###  PHP Filters 
有两种过滤器：
Validating 过滤器：
  * 用于验证用户输入
  * 严格的格式规则（比如 URL 或 E-Mail 验证）
  * 如果成功则返回预期的类型，如果失败则返回 FALSE
Sanitizing 过滤器：
  * 用于允许或禁止字符串中指定的字符
  * 无数据格式规则
  * 始终返回字符串
ID 名称	描述
  * **FILTER_CALLBACK	调用用户自定义函数来过滤数据。**
  * FILTER_SANITIZE_STRING	去除标签，去除或编码特殊字符。
  * FILTER_SANITIZE_STRIPPED	"string" 过滤器的别名。
  * FILTER_SANITIZE_ENCODED	URL-encode 字符串，去除或编码特殊字符。
  * FILTER_SANITIZE_SPECIAL_CHARS	HTML 转义字符 '"<>& 以及 ASCII 值小于 32 的字符。
  * **FILTER_SANITIZE_EMAIL**	删除所有字符，除了字母、数字以及 !#$%&'*+-/=?^_`{|}~@.[]
  * **FILTER_SANITIZE_URL**	删除所有字符，除了字母、数字以及 $-_.+!*'(),{}|\\^~[]`<>#%";/?:@&=
  * FILTER_SANITIZE_NUMBER_INT	删除所有字符，除了数字和 +-
  * FILTER_SANITIZE_NUMBER_FLOAT	删除所有字符，除了数字、+- 以及 .,eE。
  * FILTER_SANITIZE_MAGIC_QUOTES	应用 addslashes()。
  * FILTER_UNSAFE_RAW	不进行任何过滤，去除或编码特殊字符。
  * FILTER_VALIDATE_INT	在指定的范围以整数验证值。
  * FILTER_VALIDATE_BOOLEAN	如果是 "1", "true", "on" 以及 "yes"，则返回 true，如果是 "0", "false", "off", "no" 以及 ""，则返回 false。否则返回 NULL。
  * FILTER_VALIDATE_FLOAT	以浮点数验证值。
  * FILTER_VALIDATE_REGEXP	根据 regexp，兼容 Perl 的正则表达式来验证值。
  * FILTER_VALIDATE_URL	把值作为 URL 来验证。
  * FILTER_VALIDATE_EMAIL	把值作为 e-mail 来验证。
  * FILTER_VALIDATE_IP	把值作为 IP 地址来验证。

###  选项和标志 
选项和标志用于向指定的过滤器添加额外的过滤选项。不同的过滤器有不同的选项和标志。
在下面的例子中，我们用 ` filter_var() ` 和 "min_range" 以及 "max_range" 选项验证了一个整数：
```

<?php
$var=300;
$int_options = array(
"options"=>array
 (
 "min_range"=>0,
 "max_range"=>256
 )
);

if(!filter_var($var, FILTER_VALIDATE_INT, $int_options))
 {
 echo("Integer is not valid");
 }
else
 {
 echo("Integer is valid");
 }
?>

```
就像上面的代码一样，**选项必须放入一个名为 "options" 的相关数组中**。如果使用标志，则不需在数组内。
由于整数是 "300"，它不在指定的范围内，以上代码的输出将是 "Integer is not valid"。

##  验证输入filter_input() 
filter_input() 函数从脚本外部获取输入，并进行过滤。
本函数用于对来自非安全来源的变量进行验证，比如用户的输入。
本函数可从各种来源获取输入：
  * INPUT_GET
  * INPUT_POST
  * INPUT_COOKIE
  * INPUT_ENV
  * INPUT_SERVER
  * INPUT_SESSION (Not yet implemented)
  * INPUT_REQUEST (Not yet implemented)
如果成功，则返回被过滤的数据，如果失败，则返回 false，如果 variable 参数未设置，则返回 NULL。
语法
**filter_input(input_type, variable, filter, options)**
让我们试着**验证来自表单的输入**。
我们需要作的第一件事情是确认是否存在我们正在查找的输入数据。
然后我们用 ` filter_input() 函数 `过滤输入的数据。
在下面的例子中，输入变量 "email" 被传到 PHP 页面：
```

<?php
if(!filter_has_var(INPUT_GET, "email"))
 {
 echo("Input type does not exist");
 }
else
 {
 if (!filter_input(INPUT_GET, "email", FILTER_VALIDATE_EMAIL))
  {
  echo "E-Mail is not valid";
  }
 else
  {
  echo "E-Mail is valid";
  }
 }
?>

```
##  净化输入 
让我们试着清理一下从表单传来的 URL。
首先，我们要确认是否存在我们正在查找的输入数据。
然后，我们用 filter_input() 函数来净化输入数据。
在下面的例子中，输入变量 "url" 被传到 PHP 页面：
```

<?php
if(!filter_has_var(INPUT_POST, "url"))
 {
 echo("Input type does not exist");
 }
else
 {
 $url = filter_input(INPUT_POST, "url", FILTER_SANITIZE_URL);
 }
?>

```
例子解释：
上面的例子有一个通过 "POST" 方法传送的输入变量 (url)：
检测是否存在 "POST" 类型的 "url" 输入变量
如果存在此输入变量，对其进行净化（删除非法字符），并将其存储在 $url 变量中
假如输入变量类似这样："http://www.W3非o法ol.com.c字符n/"，则净化后的 $url 变量应该是这样的：http://www.W3School.com.cn/

##  过滤多个输入filter_input_array 
表单通常由多个输入字段组成。为了避免对 filter_var 或 filter_input 重复调用，我们可以使用 filter_var_array 或 the filter_input_array 函数。
在本例中，我们使用 filter_input_array() 函数来过滤三个 GET 变量。接收到的 GET 变量是一个名字、一个年龄以及一个邮件地址：
```

<?php
$filters = array
 (
 "name" => array
  (
  "filter"=>FILTER_SANITIZE_STRING
  ),
 "age" => array
  (
  "filter"=>FILTER_VALIDATE_INT,
  "options"=>array
   (
   "min_range"=>1,
   "max_range"=>120
   )
  ),
 "email"=> FILTER_VALIDATE_EMAIL,
 );
$result = filter_input_array(INPUT_GET, $filters);
if (!$result["age"])
 {
 echo("Age must be a number between 1 and 120.<br />");
 }
elseif(!$result["email"])
 {
 echo("E-Mail is not valid.<br />");
 }
else
 {
 echo("User input is valid");
 }
?>

```
##  使用 Filter Callback 
**通过使用 FILTER_CALLBACK 过滤器，可以调用自定义的函数**，把它作为一个过滤器来使用。这样，我们就拥有了数据过滤的完全控制权。
FILTER_CALLBACK 过滤器使用用户自定义函数对值进行过滤。**指定的函数必须存入名为 "options" 的关联数组中。**
在下面的例子中，我们使用了一个自定义的函数把所有 "_" 转换为空格：
```

<?php
function convertSpace($string)
{
return str_replace("_", " ", $string);
}

$string = "Peter_is_a_great_guy!";
echo filter_var($string, FILTER_CALLBACK, array("options"=>"convertSpace"));
?>

```
以上代码的结果是这样的：
Peter is a great guy!
例子解释：
上面的例子把所有 "_" 转换成空格：
创建一个把 "_" 替换为空格的函数
调用 filter_var() 函数，它的参数是 FILTER_CALLBACK 过滤器以及包含我们的函数的数组

##  示例 
1、过滤电子邮件
```

<?php
$email = 'john@example.com';
echo filter_var($email, FILTER_SANITIZE_EMAIL);

```
2、验证电子邮件
```

<?php
$input = 'john@example.com';
$isEmail = filter_var($input, FILTER_VALIDATE_EMAIL);
if ($isEmail !== false) {
    echo "Success";
} else {
    echo "Fail";
}


```
