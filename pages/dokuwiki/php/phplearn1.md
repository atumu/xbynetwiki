title: phplearn1 

#  PHP学习之语法1 
官网:http://php.net/
PHP手册:http://php.net/manual/zh/
APIDOC:http://www.php.net/manual/zh/indexes.functions.php
函数归类参考:http://php.net/manual/zh/funcref.php
扩展库归类参考:http://php.net/manual/zh/extensions.membership.php http://cn2.php.net/manual/zh/refs.basic.other.php
example:http://php.net/manual/zh/indexes.examples.php
PSR:http://www.php-fig.org/ https://github.com/php-fig/fig-standards
学习版本:5.6,最新版本7.0.本系列针对5.6进行学习
```

<!DOCTYPE html>
<html>
<body>

<?php
echo "我的第一段 PHP 脚本！";
?>

</body>
</html>

```
PHP 支持三种**注释**：
```

<!DOCTYPE html>
<html>
<body>
<?php
// 这是单行注释

# 这也是单行注释

/*
这是多行注释块
它横跨了
多行
*/
?>

</body>
</html>

```


##  PHP变量 
变量以 $ 符号开头，其后是变量的名称，变量名称对大小写敏感（$y 与 $Y 是两个不同的变量）
```

<?php
$txt="Hello world!";
$x=5;
$y=10.5;
?>

```
###  PHP 变量作用域 
在 PHP 中，可以在脚本的任意位置对变量进行声明。
变量的作用域指的是变量能够被引用/使用的那部分脚本。
PHP 有三种不同的变量作用域：
  * local（局部）
  * global（全局）
  * static（静态）
PHP 同时在名为 $GLOBALS数组中存储了所有的全局变量。下标存有变量名。这个数组在函数内也可以访问，并能够用于直接更新全局变量。

###  PHP static 关键词 
通常，当函数完成/执行后，会删除所有变量。不过，有时我需要不删除某个局部变量。实现这一点需要更进一步的工作。
要完成这一点，请在您首次声明变量时使用 static 关键词：
```

<?php
function myTest() {
  static $x=0;
  echo $x;
  $x++;
}

myTest();
myTest();
myTest();

?>

```
然后，每当函数被调用时，这个变量所存储的信息都是函数最后一次被调用时所包含的信息。
注释：` 该变量仍然是函数的局部变量。 `
##  PHP echo 和 print、print_r 语句 
echo 和 print 之间的差异：
echo - 能够输出一个以上的字符串
print - 只能输出一个字符串，并始终返回 1
print_r 打印数组的关键字：语法print_r(要打印数组名)，主要用于调试
提示：echo 比 print 稍快，因为它不返回任何值。
echo 是一个语言结构，有无括号均可使用：echo 或 echo()。
**显示变量**
下面的例子展示如何用 echo 命令来显示字符串和变量：
```

<?php
$txt1="Learn PHP";
$txt2="W3School.com.cn";
$cars=array("Volvo","BMW","SAAB");

echo $txt1;
echo "<br>";
echo "Study PHP at $txt2";
echo "My car is a {$cars[0]}";
?>

```
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$stu=array('stu_no'=>'10010','stu_name'=>'老赵');  
print_r($stu);  
?> 
输出结果Array ( [stu_no] => 10010 [stu_name] => 老赵 )

```
##  PHP数据类型 
PHP 支持八种原始类型（type）。
四种标量类型：
  * string（字符串）
  * integer（整型）
  * float（浮点型，也作 double ）
  * boolean（布尔型）
两种复合类型：
  * array（数组）
  * object（对象）
两种特殊类型：
  * resource（资源）
  * NULL（空）
**PHP 字符串**
字符串是字符序列，比如 "Hello world!"。
字符串可以是引号内的任何文本。您**可以使用单引号或双引号**，区别如下：
  * 使用单引号，出现的所有特殊字符都会被当作字符串处理，无需显示转义。
  * **使用双引号，php会尝试解释其中的特殊字符，变量**。也就是尝试进行表达式求值。
定义字符串有 3 种方法：
  * 单引号（'）
  * 双引号（"）
  * 定界符（<<<）
```

<?php
$str = <<<EOD
我是用定界符定义字符串的例子
这是其他更多字符
……
EOD;

//双引号字符串最重要的一点是其中的变量名会被变量值解析替代：
$var_char = "这是一些文字";
echo "请打印这些文字：$var_char";	//输出：请打印这些文字：这是一些文字
?>

```

**PHP 逻辑**
逻辑是 true (1)或 false (0)。
` 一般对false进行判断时为防止歧义，需要注意使用&lt;nowiki&gt;!==或者===&lt;/nowiki&gt; `
布尔类型分为两类：
  * TRUE：表示事实成立，为真，也可写作true
  * FALSE：表示事实不成立，为假，也可写作false
当其他类型转换为 boolean 时，以下值被认为是 FALSE ：
  * 整型值 0（零）
  * 浮点型值 0.0（零）
  * 空白字符串和字符串 "0"
  * 没有成员变量的数组
  * 没有单元的对象（仅适用于 PHP 4）
  * 特殊类型 NULL（包括尚未设定的变量）
 除上所列意外的所有其它值都被认为是 TRUE（包括任何资源）。
is_null() 函数检测变量是否为 NULL 

**PHP 数组**
数组在一个变量中存储多个值。数值是可变长度的。
在下面的例子中，我们将测试不同的数组。PHP **var_dump() 会返回变量的数据类型和值**：
实例
```

<?php 
$cars=array("Volvo","BMW","SAAB");//第一种形式
var_dump($cars);
print_r($cars); //两种打印数组方式
$cars2=array("name"=>"Volvo","BMW"=>"SAAB");//第二种形式，PHP 关联数组
?>

```
**PHP7中可以这样直接定义数组**
```

$cars=["Volvo","BMW","SAAB"];

```
**PHP 对象**
对象是存储数据和有关如何处理数据的信息的数据类型。
在 PHP 中，必须明确地声明对象。
首先我们必须声明对象的类。对此，我们使用 class 关键词。类是包含属性和方法的结构。
然后我们在对象类中定义数据类型，然后在该类的实例中使用此数据类型：
```

<?php
class Car
{
  var $color;
  function Car($color="green") {
    $this->color = $color;
  }
  function what_color() {
    return $this->color;
  }
}
?>

```
**PHP null 值**
特殊的 null 值表示变量无值。null 是数据类型 null 唯一可能的值。
null 值标示变量是否为空。也用于区分空字符串与空值数据库。
可以通过把值设置为 null，将变量清空：
实例
```

<?php
$x="Hello world!";
$x=null;
var_dump($x);
?>

```
PHP 整数、浮点数
###  查看变量和对象类型 
通过 ` gettype() 函数 `可以方便的查看某个变量的类型：
<fc #ff0000>不要使用 gettype() 来测试某种类型，因为其返回的字符串在未来的版本中可能需要改变。此外，由于包含了字符串的比较，它的运行也是较慢的。
使用 is_* 函数代替。</fc>
```

<?php
$var_bool = TRUE;	    // a boolean
$var_str  = "foo";	    // a string
$var_int  = 12;		    // an integer

echo gettype($var_bool);    // 输出 boolean
echo gettype($var_str);	    // 输出 string
echo gettype($var_int);	    // 输出 integer
?>

```
提示
由于历史原因，**如果是 float 类型数据，gettype() 函数返回的是 double，而不是 float 。**
如果想查看某个表达式的值和类型，请使用用 ` var_dump() 函数、print_r() `。

检测对象类型：
get_class()
get_parent_class() 
is_subclass_of()
get_called_class() - 后期静态绑定（"Late Static Binding"）类的名称

  * ` gettype() 函数 `可以方便的查看某个变量的类型
  * var_dump()用于打印变量类型和值
  * is_array()
  * isset()：检测变量是否设置
  * unset():销毁变量
  * defined()：检测常量是否被定义
  * is_init()
  * is_string()
  * is_bool()
  * is_numeric()
  * is_double()、is_float()
  * is_scalar():是否是标量
  * is_null() 函数检测变量是否为 NULL 
  * is_callable();检查改变量是否是有效的函数名称
  * empty() 用于检测一个变量是否为空,empty() 是一个语言结构而非函数，因此它无法被变量函数调用。

###  empty()与isset()比较 
![](/data/dokuwiki/php/pasted/20160407-160026.png)
###  数据类型转换 
PHP 中的类型强制转换和 C 中的非常像：在要转换的变量之前加上用括号括起来的目标类型：
```

<?php
$foo = 10;		// $foo 为整型
$bar = (boolean) $foo;	// $bar 为布尔型
?>

```
允许的强制转换有：
  * (int)或(integer) - 转换成整型
  * (bool)或(boolean) - 转换成布尔型
  * (float)或(double)或(real) - 转换成浮点型
  * (string) - 转换成字符串
  * (array) - 转换成数组
  * (object) - 转换成对象
另外，将一个变量还原为字符串，还可以将变量放置在双引号中：
```

<?php
$foo = 10;	// $foo 为整型
$str = "$foo";	// $str 为字符串
?>

```
**类型戏法**
PHP 在变量定义中不需要（或不支持）明示的类型定义，变量类型是根据使用该变量的上下文所决定的：
```

<?php
$foo = "0";				// $foo 为字符型
$foo = $foo+2;				// $foo 现在为整型，值为2
$foo = $foo + 1.3;			// $foo 为浮点型：3.3
$foo2 = 5 + "10 Little Piggies";	// $foo2 为整型，参考下面 延伸阅读 部分
?>

```
##  PHP 字符串函数 
字符串是字符序列。
strlen() 函数返回字符串的长度，以字符计。提示：strlen() 常用于循环和其他函数，在确定字符串何时结束很重要时。（例如，在循环中，我们也许需要在字符串的最后一个字符之后停止循环）。
strpos() 函数用于检索字符串内指定的字符或文本。如果找到匹配，则会返回首个匹配的字符位置。如果未找到匹配，则将返回 false。
下例检索字符串 "Hello world!" 中的文本 "world"：echo strpos("Hello world!","world");
##  PHP 常量 
常量是单个值的标识符（名称）。在脚本中无法改变该值。
有效的常量名以字符或下划线开头（常量名称前面没有 $ 符号）。
如需设置常量，请使用 ` define() 函数 ` - 它使用三个参数：
  * 首个参数定义常量的名称
  * 第二个参数定义常量的值
  * 可选的第三个参数规定常量名是否对大小写敏感。boolan值，默认敏感(false)。
下例创建了一个对大小写敏感的常量：
define("GREETING", "Welcome to W3School.com.cn!");
echo GREETING;
下例创建了一个对大小写不敏感的常量:
define("GREETING", "Welcome to W3School.com.cn!", true);
echo greeting;

##  运算符 
**PHP 字符串运算符**
运算符	名称	例子	结果
.	串接	$txt1 = "Hello" $txt2 = $txt1 . " world!"	现在 $txt2 包含 "Hello world!"
.=	串接赋值	$txt1 = "Hello" $txt1 .= " world!"	现在 $txt1 包含 "Hello world!"
```

<?php
$a = "Hello";
$b = $a . " world!";
echo $b; // 输出 Hello world!

$x="Hello";
$x .= " world!";
echo $x; // 输出 Hello world!
?>

```
**PHP 比较运算符**
```

运算符	名称	例子	结果
==	等于	$x == $y	如果 $x 等于 $y，则返回 true。
#### 	全等（完全相同）	$x  $y	如果 $x 等于 $y，且它们类型相同，则返回 true。
!=	不等于	$x != $y	如果 $x 不等于 $y，则返回 true。
<>	不等于	$x <> $y	如果 $x 不等于 $y，则返回 true。
!==	不全等（完全不同）	$x !== $y	如果 $x 不等于 $y，且它们类型不相同，则返回 true。
>	大于	$x > $y	如果 $x 大于 $y，则返回 true。
<	大于	$x < $y	如果 $x 小于 $y，则返回 true。
>=	大于或等于	$x >= $y	如果 $x 大于或者等于 $y，则返回 true.
<=	小于或等于	$x <= $y	如果 $x 小于或者等于 $y，则返回 true。

```
**PHP 逻辑运算符**
```

运算符	名称	例子	结果
and	与	$x and $y	如果 $x 和 $y 都为 true，则返回 true。
or	或	$x or $y	如果 $x 和 $y 至少有一个为 true，则返回 true。
xor	异或	$x xor $y	如果 $x 和 $y 有且仅有一个为 true，则返回 true。
&&	与	$x && $y	如果 $x 和 $y 都为 true，则返回 true。
||	或	$x || $y	如果 $x 和 $y 至少有一个为 true，则返回 true。
!	非	!$x	如果 $x 不为 true，则返回 true。

```
**PHP 数组运算符**
```

运算符	名称	例子	结果
+	联合	$x + $y	$x 和 $y 的联合（但不覆盖重复的键）
==	相等	$x == $y	如果 $x 和 $y 拥有相同的键/值对，则返回 true。
#### 	全等	$x  $y	如果 $x 和 $y 拥有相同的键/值对，且顺序相同类型相同，则返回 true。
!=	不相等	$x != $y	如果 $x 不等于 $y，则返回 true。
<>	不相等	$x <> $y	如果 $x 不等于 $y，则返回 true。
!==	不全等	$x !== $y	如果 $x 与 $y 完全不同，则返回 true。

```
  
**引用操作符&**
引用操作符&

**三元操作符**
expression ? truevalue: false value
**PHP7中改进的三目运算符:**
把这个放在第一个说是因为我觉得它很有用。用法：
$a = $_GET['a'] ?? 1;
它相当于：
$a = isset($_GET['a']) ? $_GET['a'] : 1;
新增的 ?? 运算符可以简化判断。

**错误抑制操作符@**
错误抑制操作符@可以在任何表达式前面使用。
$a=@(57/0)
@ $a= 57/0
如果没有@这行代码将产生除0警告，使用@就会被抑制住。一旦你抑制住一些警告，那么你就需要写错误处理代码。
启用PHP配置文件中的**trace_errors特性**，错误信息会被保存在全局变量` $php_errormsg `

**执行操作符**
执行操作符实际上是**一对反向单引号（` `）**.PHP将试着将反向单引号里面的命令**当作服务器端的命令行来执行**。表达式的值就是命令执行的结果。
例如：linux下：
```

echo `ls -la`;

```
windows下:(windows下命令行输出默认为gbk编码，所以需要通过` iconv()函数进行字符编码转换 `)
```

echo iconv('GB2312','UTF-8',`dir c:`);

```

**类型操作符:instanceof**


##  语句控制 
在 PHP 中，我们可以使用以下**条件语句**：
  * if 语句 - 如果指定条件为真，则执行代码
  * if...else 语句 - 如果条件为 true，则执行代码；如果条件为 false，则执行另一端代码
  * if...elseif....else 语句 - 选择若干段代码块之一来执行
  * switch 语句 - 语句多个代码块之一来执行
```

switch (expression)
{
case label1:
  code to be executed if expression = label1;
  break;  
case label2:
  code to be executed if expression = label2;
  break;
default:
  code to be executed
  if expression is different 
  from both label1 and label2;
}

```
在 PHP 中，我们有以下**循环语句**：
  * while - 只要指定条件为真，则循环代码块
  * do...while - 先执行一次代码块，然后只要指定条件为真则重复循环
  * for - 循环代码块指定次数
  * foreach - **遍历数组**中的每个元素并循环代码块
```

<?php 
for ($x=0; $x<=10; $x++) {
  echo "数字是：$x <br>";
} 

//foreach数组遍历
foreach ($array as $value) {
  code to be executed;
}
foreach ($array as $key=>$value) {
  code to be executed;
}
?>

```
##  使用declare 
http://php.net/manual/en/control-structures.declare.php
declare用于设置代码的运行规则。
ticks 指令在 PHP 5.3.0 中是过时指令，将会从 PHP 6.0.0 移除。
encoding 是 PHP 5.3.0 新增指令。
```

<?php
declare(encoding='ISO-8859-1');
// code here
?>

```
##  函数 
```

function functionName() {
  被执行的代码;
}
function setHeight($minheight=50) {
  echo "The height is : $minheight <br>";
}

```
注释：函数名能够以字母或下划线开头（而非数字）。
注释：**函数名对大小写不敏感。**，这点与变量是有区别的
参数传递都是按值传递，会拷贝一个副本。如需按引用传递，加个` & `，如:&$var1

` function_exists() 函数 `用于检测函数是否被定义。bool function_exists( string function_name )
```

<?php
function testfunc(){
    echo '我是自定义函数';
}
if(!function_exists('testfunc')){
    function testfunc(){
        echo '我是自定义函数';
    }
}
testfunc();
?>

```

###  函数限定参数类型 
http://stackoverflow.com/questions/4103480/really-php-argument-1-passed-to-my-function-must-be-an-instance-of-string-s
http://blog.jobbole.com/91735/
Type Hints can only be of the object and array (since PHP 5.1) type. Traditional type hinting with int and string isn't supported.
**PHP7之前不提供对int和string限定类型参数的支持。仅支持对象和数组类型.。但是在PHP7之后开始支持原生类型限定(4种新的标量类型声明：int，float，string和bool)了。PHP7更进一步，支持返回值类型限定**
如下:**这样写是错误的**
```

//PHP7之前错误写法
function endsWith(string $src, string $needle){
	if($src=="") return false;
	$pos=strlen($src)-strlen($needle);
	if($pos<0) return false;
	if(strrpos($src,$needle)===false || $pos!=strrpos($src,$needle)) return false;
	echo "wwww";
	return true;
}

//PHP7正确写法
function endsWith(string $src, string $needle): bool{
	...
}

```

改为如下形式:通过is_string进行判断
```

function endsWith($src,$needle){
    if (!is_string($src)) {
        trigger_error('No, you fool!');
        return;
    }
    ...
}

```
PHP7写法说明：
RFC更推荐给每一个PHP文件，添加一句新的可选指令（declare(strict_type=1);），让同一个PHP文件内的全部函数调用和语句返回，都有一个“严格约束”的标量类型声明检查。此外，在开启严格类型约束后，调用拓展或者PHP内置函数在参数解析失败，将产生一个E_RECOVERABLE_ERROR级错误。
如果配合强制类型开关指令，则可以变为这样：**declare(strict_types=1)必须是文件的第一个语句。**strict_types指令**只影响指定使用的文件，不会影响被它包含**（通过include等方式）进来的其他文件。
```

<?php
declare(strict_types=1);
 
function add(float $a, float $b): float {
    return $a + $b;
}
 
add(1, 2); 
// float(3)

```
如果不开启strict_type，PHP将会尝试帮你转换成要求的类型，而开启之后，会改变PHP就不再做类型转换，类型不匹配就会抛出错误。对于喜欢“强类型”语言的同学来说，这是一大福音。

##  PHP - 数组的排序函数 
在本节中，我们将学习如下 PHP 数组排序函数：
bool sort( array &array [, int sort_flags] )
![](/data/dokuwiki/php/pasted/20160407-151656.png)
  * sort() - 以升序对数组排序 
  * rsort() - 以降序对数组排序
  * asort() - 根据值，以升序对关联数组进行排序
  * ksort() - 根据键，以升序对关联数组进行排序
  * arsort() - 根据值，以降序对关联数组进行排序
  * krsort() - 根据键，以降序对关联数组进行排序
  * shuffle() 函数用于将数组打乱。
参考阅读
natsort()：对数组使用 自然算法 进行排序。
array_multisort()：对多个数组或多维数组进行排序。
参考：http://www.5idev.com/p-php_sort_asort_ksort.shtml
##  通用函数 
function_exists() 函数用于检测函数是否被定义。
method_exists()：检查类的方法是否存在。
is_callable()：检测参数是否为合法的可调用结构。
class_exists()：检查类是否已定义。
isset()：检测变量是否设置。
defined()：检测常量是否被定义。
###  count() 函数 
` count() 函数 `用于**计算数组中的单元数目或对象中的属性个数**，返回数组的单元个数或对象中的属性个数。
语法：int count( mixed var [, int mode] )
如果 var 是非数组的普通变量，则返回 1 ，对于不存在、未初始化或空数组返回 0 。
可选参数 mode 设为 COUNT_RECURSIVE（或 1），count() 将递归地对数组计数，这对计算多维数组的所有单元尤其有用，但 count() 识别不了无限递归。mode 的默认值是 0 。
sizeof() 是本函数的别名。

###  数组操作函数 
` count() 函数 `用于**计算数组中的单元数目或对象中的属性个数**
` in_array() 函数 `用于检查数组中是否存在某个值。bool in_array( mixed needle, array array [, bool strict] )
  * needle	需要在数组中搜索的值，如果是字符串，则区分大小写
  * array	需要检索的数组
  * strict	可选，如果设置为 TRUE ，则还会对 needle 与 array 中的值类型进行检查
` array_count_values() 函数 `用于统计数组中所有的值出现的次数。返回一个数组
```

<?php
$arr_a = array("a", "b", "a", 1);
print_r(array_count_values($arr_a));
?>
例子输出结果如下：
Array
(
    [a] => 2
    [b] => 1
    [1] => 1
)

```
array_sum() 函数用于计算数组中所有值的和。
array_product() 函数用于计算数组中所有值的乘积。
` array_key_exists() 函数 `用于检查给定的键名或索引是否存在于数组中。bool array_key_exists( mixed key, array search )
` array_search() 函数 `用于在数组中搜索给定的值。mixed array_search( mixed needle, array array [, bool strict] )如果成功则**返回相应的键名**，否则返回 FALSE 。
####  list() 语言结构 
` list() 语言结构 `用于**把数组中的值赋给一些变量**。同 array() 一样，list() 不是真正的函数，而是语言结构。语法：void list( mixed var, mixed ... )
` 注意: list() 仅能用于数字索引的数组并假定数字索引从 0 开始。 `
例子1：
```

<?php
$arr_age = array(18, 20, 25);
list($wang, $li, $zhang) = $arr_age;
echo $wang;        //输出：18
echo $zhang;        //输出：25
?>

```
例子2，数据表查询：
```

$result = mysql_query("SELECT id, username, email FROM user",$conn);
while(list($id, $username, $email) = mysql_fetch_row($result)) {
    echo "用户名：$username<br />";
    echo "电子邮箱：$email";
}

```
##  PHP 超全局变量 
PHP 中的许多预定义变量都是“超全局的”，这意味着它们在一个脚本的全部作用域中都可用。在函数或方法中无需执行 global $variable; 就可以访问它们。
这些超全局变量是：
  * $GLOBALS
  * $_SERVER
  * $_REQUEST
  * $_POST
  * $_GET
  * $_FILES
  * $_ENV
  * $_COOKIE
  * $_SESSION
**$GLOBALS** — 引用全局作用域中可用的全部变量
$GLOBALS 这种全局变量用于在 PHP 脚本中的任意位置访问全局变量（从函数或方法中均可）。
PHP 在名为 $GLOBALS[index] 的数组中存储了所有全局变量。变量的名字就是数组的键。
```

$x = 75; 
$y = 25;
 
function addition() { 
  $GLOBALS['z'] = $GLOBALS['x'] + $GLOBALS['y']; 
}

```
**PHP $_SERVER**
$_SERVER 这种超全局变量保存**关于HTTP报头、路径和脚本位置的信息**。
下面的例子展示了如何使用 $_SERVER 中的某些元素：
```

<?php 
echo $_SERVER['PHP_SELF'];
echo "<br>";
echo $_SERVER['SERVER_NAME'];
echo "<br>";
echo $_SERVER['HTTP_HOST'];
echo "<br>";
echo $_SERVER['HTTP_REFERER'];
echo "<br>";
echo $_SERVER['HTTP_USER_AGENT'];
echo "<br>";
echo $_SERVER['SCRIPT_NAME'];
?>

```
下表列出了您能够在 $_SERVER 中访问的最重要的元素：
![](/data/dokuwiki/php/pasted/20160406-102758.png)
![](/data/dokuwiki/php/pasted/20160406-102813.png)
**PHP $_REQUEST**
PHP $_REQUEST 用于**收集 HTML 表单提交的数据。**
下面的例子展示了一个包含输入字段及提交按钮的表单。当用户通过点击提交按钮来提交表单数据时, 表单数据将发送到 <form> 标签的 action 属性中指定的脚本文件。在这个例子中，我们指定文件本身来处理表单数据。如果您需要使用其他的 PHP 文件来处理表单数据，请修改为您选择的文件名即可。然后，我们可以使用超级全局变量 $_REQUEST 来收集 input 字段的值：
```

<form method="post" action="<?php echo $_SERVER['PHP_SELF'];?>">
Name: <input type="text" name="fname">
<input type="submit">
</form>

<?php 
$name = $_REQUEST['fname']; 
echo $name; 
?>

```
PHP $_POST 广泛用于收集提交 method="post" 的 HTML 表单后的表单数据。$_POST 也常用于传递变量。
PHP $_GET 也可用于收集提交 HTML 表单 (method="get") 之后的表单数据。$_GET 也可以收集 URL 中的发送的数据。

##  php内嵌HTML 
```

<?php
while($con) {
?>
<p>laruence</P>
<?php
 }
?>

```
就结果来说, 上面的代码, 其实和下面的结果一样:
```

<?php
 while($con) {
   echo "<p>laruence</P>";
 }
?>

```

##  终止执行die()和exit 
exit;语言结构，终止一段脚本的执行，且不返回任何值。使用
exit;
exit('errormsg');
die()函数，终止脚本执行.使用:
mysqli_query($query) or die('error msg');
