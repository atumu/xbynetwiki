title: phplearn24 

#  PHP学习之命名空间 
PHP的命名空间（namespace）是php5.3之后才有的。
不过网上都建议：还是不要用命名空间了，如果每次使用类都写上完整的命名空间，对于智能提示一向不强的所有PHP IDE，都会是很大的输入负担。如果使用命名空间导入功能，在php的多级include后，经常产生莫名其妙的错误，合理地规划类名，不会用到命名空间的。
所以本文只是进行了解。
##  namespace的定义和使用 
```

file1:
<?php namespace foo;
  class Cat { 
    static function says() {echo 'meoow';}  } ?>

file2:
<?php namespace bar;
  class Dog {
    static function says() {echo 'ruff';}  } ?>

file3:
<?php namespace animate;
  class Animal {
    static function breathes() {echo 'air';}  } ?>

file4:
<?php namespace fub;
  include 'file1.php';
  include 'file2.php';
  include 'file3.php';
  use foo as feline; //别名
  use bar as canine;
  use animate;
  echo \feline\Cat::says(), "<br />\n";
  echo \canine\Dog::says(), "<br />\n";
  echo \animate\Animal::breathes(), "<br />\n";  ?>

```
  * namespace操作符和_ _NAMESPACE_ _常量
  * 命名空间必须是程序脚本的第一条语句
  * 命名空间是运行时解析的。use就相当于一种声明，并不解析和加载。
##  在同一个文件中定义多个命名空间 
在同一个文件中定义多个命名空间有两种语法形式：
Example #1 定义多个命名空间，简单组合语法
```

<?php
namespace MyProject;

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }

namespace AnotherProject;

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
?>

```
不建议使用这种语法在单个文件中定义多个命名空间。建议使用下面的大括号形式的语法。
Example #2 定义多个命名空间，大括号语法
```

<?php
declare(encoding='UTF-8');
namespace MyProject {

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
}

namespace AnotherProject {

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
}

namespace { // global code
session_start();
$a = MyProject\connect();
echo MyProject\Connection::start();
}
?>

```
在实际的编程实践中，非常不提倡在同一个文件中定义多个命名空间。这种方式的主要用于将多个 PHP 脚本合并在同一个文件中。
全局代码必须用一个不带名称的 namespace 语句加上大括号括起来
除了开始的declare语句外，命名空间的括号外不得有任何PHP代码。
注意访问任意全局类、函数或常量，都可以使用完全限定名称
##  全局类和命名空间类 
如果要new一个全局类使用 new \A()
如果要new一个命名空间类，使用new my\namespace\A() 省略最开始的\
##  命名空间的顺序与解析 
###  命名空间名称定义 
  * 非限定名称Unqualified name：名称中不包含命名空间分隔符的标识符，例如 Foo
  * 
  * 限定名称Qualified name：名称中含有命名空间分隔符的标识符，例如 Foo\Bar
  * 
  * 完全限定名称Fully qualified name：名称中包含命名空间分隔符，并以命名空间分隔符开始的标识符，例如 \Foo\Bar。 namespace\Foo 也是一个完全限定名称。

###  命名空间名称解析规则： 
  * **对完全限定名称的函数，类和常量的调用**在编译时解析。例如 new \A\B 解析为类 A\B。
  * **所有的非限定名称和限定名称（非完全限定名称）根据当前的导入规则在编译时进行转换。**例如，如果命名空间 A\B\C 被导入为 C，那么对 C\D\e() 的调用就会被转换为 A\B\C\D\e()。
  * 在命名空间内部，**所有的没有根据导入规则转换的限定名称均会在其前面加上当前的命名空间名称。**例如，在命名空间 A\B 内部调用 C\D\e()，则 C\D\e() 会被转换为 A\B\C\D\e() 。
  * 非限定类名根据当前的导入规则在编译时转换（用全名代替短的导入名称）。例如，如果命名空间 A\B\C 导入为C，则 new C() 被转换为 new A\B\C() 。
  * 在命名空间内部（例如A\B），**对非限定名称的` 函数 `调用是在运行时解析的。**例如对函数 foo() 的调用是这样解析的：
1、在当前命名空间中查找名为 A\B\foo() 的函数
2、` 尝试查找并调用 全局(global) 空间中的函数 ` foo()。
  * 在命名空间（例如A\B）内部**对非限定名称或限定名称` 类 `（非完全限定名称）的调用是在运行时解析的。**下面是调用 new C() 及 new D\E() 的解析过程： 
new C()的解析:
1、在当前命名空间中查找A\B\C类并尝试自动装载类A\B\C。` 与函数不同，对非限定名称或限定名称类不会自动在全局空间中装载 `
new D\E()的解析:
在类名称前面加上当前命名空间名称变成：A\B\D\E，然后查找该类。
尝试自动装载类 A\B\D\E。
**为了引用全局命名空间中的全局类，必须使用完全限定名称** new \C()。

###  类和函数、常量解析优先级次序与区别 
在一个命名空间中，当 PHP 遇到一个非限定的类、函数或常量名称时，它使用不同的优先策略来解析该名称。
**类名称总是解析到当前命名空间中的名称。因此在访问系统内部或不包含在命名空间中的` 类名称时，必须使用完全限定名称(这点与函数和常量不同 `)**，例如：
Example #1 在命名空间中访问全局类
```

<?php
namespace A\B\C;
class Exception extends \Exception {}

$a = new Exception('hi'); // $a 是类 A\B\C\Exception 的一个对象
$b = new \Exception('hi'); // $b 是类 Exception 的一个对象

$c = new ArrayObject; // 致命错误, 找不到 A\B\C\ArrayObject 类
?>

```
` **对于函数和常量来说，如果当前命名空间中不存在该函数或常量，PHP 会退而使用全局空间中的函数或常量。（这点和类不同）** ` 
Example #2 命名空间中后备的全局函数/常量
```

<?php
namespace A\B\C;

const E_ERROR = 45;
function strlen($str)
{
    return \strlen($str) - 1;
}

echo E_ERROR, "\n"; // 输出 "45"
echo INI_ALL, "\n"; // 输出 "7" - 使用全局常量 INI_ALL

echo strlen('hi'), "\n"; // 输出 "1"
if (is_array('hi')) { // 输出 "is not array"
    echo "is array\n";
} else {
    echo "is not array\n";
}
?>

```

```

file1.php

<?php
namespace Foo\Bar\subnamespace;

const FOO = 1;
function foo() {}
class foo
{
    static function staticmethod() {}
}
?>
file2.php

<?php
namespace Foo\Bar;
include 'file1.php';

const FOO = 2;
function foo() {}
class foo
{
    static function staticmethod() {}
}

/* 非限定名称 */
foo(); // 解析为 Foo\Bar\foo resolves to function Foo\Bar\foo
foo::staticmethod(); // 解析为类 Foo\Bar\foo的静态方法staticmethod。resolves to class Foo\Bar\foo, method staticmethod
echo FOO; // resolves to constant Foo\Bar\FOO

/* 限定名称 */
subnamespace\foo(); // 解析为函数 Foo\Bar\subnamespace\foo
subnamespace\foo::staticmethod(); // 解析为类 Foo\Bar\subnamespace\foo,
                                  // 以及类的方法 staticmethod
echo subnamespace\FOO; // 解析为常量 Foo\Bar\subnamespace\FOO
                                  
/* 完全限定名称 */
\Foo\Bar\foo(); // 解析为函数 Foo\Bar\foo
\Foo\Bar\foo::staticmethod(); // 解析为类 Foo\Bar\foo, 以及类的方法 staticmethod
echo \Foo\Bar\FOO; // 解析为常量 Foo\Bar\FOO
?>

```

##  命名空间导入和别名 
**从PHP5.3之后可以使用命令空间导入类、接口和其他命名空间，并为其创建别名。**
命令空间导入后就不能再输出全限定名称了。
1、使用命令空间，没创建别名:
```

<?php
$response=new \Symfony\Component\HttpFoundation\Response('Oops',400);
$response->send();

```
2、使用命名空间和默认的别名:
```

<?php
use Symfony\Component\HttpFoundation\Response;
$response=new Response('Oops',400);
$response->send();

```
3、使用命名空间，并自定义别名:
```

<?php
use Symfony\Component\HttpFoundation\Response as Res;
$r=new Res('Oops',400);
$r->send();

```
**从PHP5.6开始之后还可以导入函数和常量:**
导入函数,要把use改成use func:
```

<?php
use func Namespace\funcName;
funcName();

```
导入常量,要把use改成use constant:
```

<?php
use constant Namespace\CONST_NAME;
echo CONST_NAME;

```


##  自动加载类 
http://cn2.php.net/manual/zh/function.spl-autoload-register.php
在 PHP5 中可以定义一个 _ _autoload() 函数，当调用一个未定义的类的时候就会启动此函数，从而在抛出错误之前做最后的补救，不过这个函数的本意已经被完全曲解使用了，现在都用来做自动加载。
```

function __autoload($classname){
    $dir = './libralies';
    set_include_path(get_include_path(). PATH_SEPARATOR. $dir);
    $class = str_replace('\\', '/', $classname) . '.php'; 
    require_once($class);
}

```
**注意，这个_ _autoload() 函数实际上已经不被推荐使用了，相反，现在应当使用 ` spl_autoload_register() ` 来注册类的自动加载函数。**
bool spl_autoload_register ([ callable $autoload_function [, bool $throw = true [, bool $prepend = false ]]] )
参数说明:
  * 第一个参数: 需要注册的自动装载函数，如果此项为空，则会注册 spl_autoload 函数，
  * throw 此参数设置了 autoload_function 无法成功注册时， spl_autoload_register() 是否抛出异常。
  * prepend 如果是 true， spl_autoload_register() 会添加函数到队列之首，而不是队列尾部。
上面提到了 **` spl_autoload ` 函数，实际上注册函数的规范就应当遵循此函数**，函数声明如下：
void spl_autoload ( string $class_name [, string $file_extensions ] )
例如:
```

<?php
    // Your custom class dir
    define('CLASS_DIR', 'class/')
    // Add your class dir to include path
    set_include_path(get_include_path().PATH_SEPARATOR.CLASS_DIR);
    // You can use this trick to make autoloader look for commonly used "My.class.php" type filenames
    spl_autoload_extensions('.class.php');// comma-separated list
    // Use default autoload implementation
    spl_autoload_register();
?>

```
你也可以自己实现autoload函数：
```

spl_autoload_register(function ($class) {  
    if ($class) {  
        $file = str_replace('\\', '/', $class);  
        $file .= '.class.php';  
        if (file_exists($file)) {  
            var_dump($file);  
            include $file;  
        }  
    }  
});  

```
不过还是建议单独使用一个加载类进行加载:
```

class ClassAutoloader {
        public function __construct() {
            spl_autoload_register(array($this, 'loader'));
        }
        private function loader($className) {
            echo 'Trying to load ', $className, ' via ', __METHOD__, "()\n";
            include $className . '.php';
        }
    }
$autoloader = new ClassAutoloader();

```
**需要注意的问题：**
0、命令空间与自动加载没有直接的联系。我们只是简单的组合使用它们。
1、get_include_path()是相对于当前脚本的路径去搜寻 ./;如需改变这种情况，可以使用:
```

    // Your custom class dir
    define('CLASS_DIR', 'class/')
    // Add your class dir to include path
    set_include_path(get_include_path().PATH_SEPARATOR.CLASS_DIR);

```
2、 use aaa\bbb ,不用在最前面加\
3、使用类外的常量或方法之前必须先实例化类，不然会报错。因为只有在new bbb()实例化类的时候spl_autoload才会去加载类文件。而use aaa\bbb只是一个声明。

###  自动加载示例 
```

spl_autoload_register(function($class){
	$class=str_replace("\\","/",$class);
	echo $class.".php";
	require $class.".php"; //这种写法加载的路径是相对于get_include_path()中的值，不过它总是包含当前路径。即'.',你也可以自定义路径形式。只要能找到对应的文件即可。
});
use common\lib\Comment;//只是声明，不会触发自动加载。不需要加前缀\。类似于java的import，导入后可以直接使用Commet来代表类了，而不用使用完全限定类名。
$comment=new Comment();//只有在类实例化的时候才会去加载类文件common\lib\Comment.php,自动加载只会include这个文件，而文件内是否定义命名空间，自动加载不会关心。只是在我们调用的时候才会有影响。也就是说命名空间与自动加载时没有联系的。自动加载只是按规定去加载文件。
\common\lib\outter();//调用类之外的方法，必须在类实例化之后使用，否则报undefined错误,必须加前缀\
$ca=\common\lib\AAA;//调用类之外的常量,必须在类实例化之后使用，否则报undefined错误

```
common\lib\Comment.php文件:
```

<?php
namespace common\lib;

const AAA="aaa";
class Comment{
	function __construct(){
		echo "comment";
	}
}
function outter(){
	echo "outter";
}
?>

```

###  自动加载器实例二PSR-4风格自动加载器实现 
由于目前基本上都采用Composer来管理依赖，而Composer有提供了PSR-4自动加载器的标准实现。故项目中一般不推荐自定义自动加载器。
在此仅仅只是出于演示的目的。
```

<?php
/**
 * An example of a project-specific implementation.
 *
 * After registering this autoload function with SPL, the following line
 * would cause the function to attempt to load the \Foo\Bar\Baz\Qux class
 * from /path/to/project/src/Bar/Baz/Qux.php:
 *
 *      new \Foo\Bar\Baz\Qux;
 *
 * @param string $class The fully-qualified class name.
 * @return void
 */
spl_autoload_register(function ($class) {

    // project-specific namespace prefix
    $prefix = 'Foo\\';

    // base directory for the namespace prefix
    $base_dir = __DIR__ . '/src/';

    // does the class use the namespace prefix?
    $len = strlen($prefix);
    if (strncmp($prefix, $class, $len) !== 0) {
        // no, move to the next registered autoloader
        return;
    }

    // get the relative class name
    $relative_class = substr($class, $len);

    // replace the namespace prefix with the base directory, replace namespace
    // separators with directory separators in the relative class name, append
    // with .php
    $file = $base_dir . str_replace('\\', '/', $relative_class) . '.php';

    // if the file exists, require it
    if (file_exists($file)) {
        require $file;
    }
});

$bar = new \Foo\Bar();
$bar->sayHello();


```
##  自动加载与PSR-4 
PSR-4是PHP Framework Interop Group (PHP-FIG)制定的自动加载器标准。命令空间为PSR-4提供了基础。
而依赖管理器Composer实现的PSR-4标准的自动加载器是大多数现代PHP组件的自动加载器。

##  PHP7中的变化 
use 批量声明
PHP 7 中 use 可以在一句话中声明多个类或函数或 const 了：
```

<php 
use some/namespace/{ClassA, ClassB, ClassC as C}; 
use function some/namespace/{fn_a, fn_b, fn_c}; 
use const some/namespace/{ConstA, ConstB, ConstC};

``` 
但还是要写出每个类或函数或 const 的名称（并没有像 python 一样的 from some import * 的方法）。

需要留意的问题是：如果你使用的是基于 composer 和 PSR-4 的框架，这种写法是否能成功的加载类文件？其实是可以的，composer 注册的自动加载方法是在类被调用的时候根据类的命名空间去查找位置，这种写法对其没有影响。

参考:http://www.php.net/manual/zh/language.namespaces.php
http://www.tuicool.com/articles/JVJjqyy
http://developer.51cto.com/art/201510/494674.htm
http://www.cnblogs.com/yuxing/archive/2010/06/19/1760742.html