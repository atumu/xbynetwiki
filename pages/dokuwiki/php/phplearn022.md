title: phplearn022 

#  PHP学习之面向对象学习3常用内置函数 
##  获取所有已定义的类、接口、扩展、函数、常量 
  * phpinfo() 输出关于 PHP 配置的信息
  * get_declared_classes() - 返回由已定义类的名字所组成的数组
  * get_declared_interfaces() - 返回一个数组包含所有已声明的接口
  * get_defined_functions() - 返回一个数组包含所有已定义的函数。取值分为$arr["internal"]、$arr["user"]。请看下面示例
  * get_extension_funcs — 返回模块函数名称的数组
  * get_loaded_extensions — 返回所有编译并加载模块名的 array
  * get_defined_constants() - 返回所有常量的关联数组，键是常量名，值是常量值。取值方式分为$arr["Core"][常量名]、$arr["pcre"][常量名]、$arr["user"][常量名]等。请看下面示例
  * get_defined_vars() - 返回由所有已定义变量所组成的数组。（多维数组，这些变量包括环境变量、服务器变量和用户定义的变量）
接口举例说明:
array get_declared_interfaces ( void )
array get_loaded_extensions ([ bool $zend_extensions = false ] )
array get_extension_funcs ( string $module_name )

get_defined_functions()说明：
```

<?php
function myrow($id, $data)
{
    return "<tr><th>$id</th><td>$data</td></tr>\n";
}

$arr = get_defined_functions();

print_r($arr);
?>
以上例程的输出类似于：
Array
(
    [internal] => Array
        (
            [0] => zend_version
            [1] => func_num_args
            ...
            [750] => bcscale
            [751] => bccomp
        )

    [user] => Array
        (
            [0] => myrow
        )
)

```

get_defined_constants()说明:
```

<?php
define("MY_CONSTANT", 1);
print_r(get_defined_constants(true));
?>
以上例程的输出类似于：
Array
(
    [Core] => Array
        (
            [E_ERROR] => 1
            [E_WARNING] => 2
            [E_PARSE] => 4
	    .....
        )
   
    [pcre] => Array
        (
            [PREG_PATTERN_ORDER] => 1
	    ....
            [PREG_SPLIT_DELIM_CAPTURE] => 2
        )
    .....
    [user] => Array
        (
            [MY_CONSTANT] => 1
        )

)

```

##  判断属性、方法是否存在 
property_exists — 检查对象或类是否具有该属性 .与isset()类似，但是property_exists可以用于判断对象的私有属性。
method_exists() - 检查类的方法是否存在
bool property_exists ( mixed $class , string $property )
bool method_exists ( mixed $object , string $method_name )
```

<?php

class myClass {
    public $mine;
    private $xpto;
    static protected $test;

    static function test() {
        var_dump(property_exists('myClass', 'xpto')); //true
    }
}
var_dump(property_exists('myClass', 'mine'));   //true
var_dump(property_exists(new myClass, 'mine')); //true
var_dump(property_exists('myClass', 'xpto'));   //true, as of PHP 5.3.0
var_dump(property_exists('myClass', 'bar'));    //false
var_dump(property_exists('myClass', 'test'));   //true, as of PHP 5.3.0
myClass::test();

?>

```
```

<?php 
class Test { 
  public function explicit(  ) { 
      // ... 
  } 
   
  public function __call( $meth, $args ) { 
      // ... 
  } 
} 

$Tester = new Test(); 

var_export(method_exists($Tester, 'anything')); // false 
var_export(is_callable(array($Tester, 'anything'))); // true 
?>

```
##  检查类、接口是否已定义 
class_exists() - 检查类是否已定义
interface_exists() - 检查接口是否已被定义
bool class_exists ( string $class_name [, bool $autoload = true ] )
bool interface_exists ( string $interface_name [, bool $autoload = true ] )
参数autoload：是否默认调用 _ _autoload方法(如果定义了的话)。
```

<?php
function __autoload($class)
{
    include($class . '.php');

    // Check to see whether the include declared the class
    if (!class_exists($class, false)) {
        trigger_error("Unable to load class: $class", E_USER_WARNING);
    }
}

if (class_exists('MyClass')) {
    $myclass = new MyClass();
}

?>

```
```

<?php
// 在尝试使用前先检查接口是否存在
if (interface_exists('MyInterface')) {
    class MyClass implements MyInterface
    {
        // Methods
    }
}

?>

```
##  判断扩展是否被加载 
extension_loaded — 检查一个扩展是否已经加载
bool extension_loaded ( string $name )
dl() - 运行时载入一个 PHP 扩展
bool dl ( string $library )
你也可以用php -m或者phpinfo()函数来查看

```

<?php
if (!extension_loaded('gd')) {
    if (!dl('gd.so')) {
        exit;
    }
}
?>

```
##  返回当前已实现的接口、继承链上的所有父类 
class_implements — 返回指定的类实现的所有接口。
class_parents() - 返回指定类的父类。
array class_implements ( mixed $class [, bool $autoload=true ] )
array class_parents ( mixed $class [, bool $autoload ] )

##  获取某个类中的方法、属性等 
get_class() - 返回对象的类名
get_class_vars() - 返回由类的默认属性组成的关联数组
get_object_vars() - 返回由对象属性组成的关联数组
get_class_methods — 返回由类的方法名组成的数组

