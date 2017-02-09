title: phplearn001 

#  PHP学习之函数进阶 
##  判断函数是否存在、可调用 
function_exists — 如果给定的函数已经被定义就返回 TRUE
bool function_exists ( string $function_name )
**在已经定义的函数列表（包括系统自带的函数和用户自定义的函数）中查找 function_name。**

is_callable — 检测参数是否为合法的可调用结构
bool is_callable ( callable $name [, bool $syntax_only = false [, string &$callable_name ]] )
```

function someFunction() 
{
}
$functionVariable = 'someFunction';
var_dump(is_callable($functionVariable, false, $callable_name));  // bool(true)
echo $callable_name, "\n";  // someFunction

```
```

//  Array containing a method
class someClass {
  function someMethod() 
  {
  }

}
$anObject = new someClass();
$methodVariable = array($anObject, 'someMethod');
var_dump(is_callable($methodVariable, true, $callable_name));  //  bool(true)
echo $callable_name, "\n";  //  someClass::someMethod

```


##  函数的可变数量参数 
在函数内部使用func_num_args()、func()_get_arg()、func_get_args()来获取参数信息。
mixed func_get_arg ( int $arg_num )
```

<?php
function foo()
{
    $numargs = func_num_args();
    echo "Number of arguments: $numargs<br />\n";
    if ($numargs >= 2) {
        echo "Second argument is: " . func_get_arg(1) . "<br />\n";
    }
    $arg_list = func_get_args();
    for ($i = 0; $i < $numargs; $i++) {
        echo "Argument $i is: " . $arg_list[$i] . "<br />\n";
    }
}

foo(1, 2, 3);
?>

```
##  将函数作为callback参数传递的几种形式 
在PHP5.4引入匿名函数与lambda之前:
方式一、使用某个对象的某个函数作为回调函数。
```

array_map(array($obj, 'MyMethon'), $array)

```
方式二、使用某个函数作为回调函数.
```

array_map('MyFunction', $array)

```

在引入lambda之后，函数可以作为参数和返回值传递了。所以可以这样
```

  array_map (function ($value) {
    return new MyFormElement ($value);
  }, $_POST);

```
###  PHP5.3前后匿名函数创建的区别 
然后单独函数的创建和调用方式也有区分：
在PHP5.3之前：使用create_function创建一个匿名函数
string create_function ( string $args , string $code )
```

$newfunc = create_function('$a,$b', 'return "ln($a) + ln($b) = " . log($a * $b);');
echo "New anonymous function: $newfunc\n";
echo $newfunc(2, M_E) . "\n";

```
在PHP5.3之后直接创建匿名函数
```

<?php
$greet = function($name)
{
    printf("Hello %s\r\n", $name);
};

$greet('World');
$greet('PHP');
?>

```
##  调用函数的几种方式
1、直接通过函数名调用。略。
2、通过函数变量调用。
3、使用call_user_func
mixed call_user_func ( callable $callback [, mixed $parameter [, mixed $... ]] )
```

function increment(&$var)
{
    $var++;
}

$a = 0;
call_user_func('increment', $a);
echo $a."\n";

```
```

<?php
class myclass {
    static function say_hello()
    {
        echo "Hello!\n";
    }
}
$classname = "myclass";
call_user_func(array($classname, 'say_hello'));
call_user_func($classname .'::say_hello'); // As of 5.2.3

$myobject = new myclass();
call_user_func(array($myobject, 'say_hello'));

```
4、使用反射API。例如ReflectionFunction::invoke() - Invokes function
ReflectionMethod::invoke() - Invoke

##  生成器Generators 
http://php.net/manual/zh/language.generators.overview.php
PHP5.5引入Generators
所谓Generators, 我们以下称为”生成器”, 是一种可以返回迭代器的生成器. 呵呵, 这话有点绕, 让我们看看一个代码, 在没有迭代器之前, 如果我们遍历一个动态生成的数组:
```

<?php
   function return_array() {
       $array = dummy(); //计算全部数组内容
       return $array;
   }
  foreach (return_array() as $v) {
  }

```
这里就有一个问题, 我们需要一次性生成全部数组内容, 并且返回, 想象一下如果数据来源非常大, 我们无法一次性读入内存.
当然, 我们可以采用一个类, 封装一个支持迭代的实现:
```

<?php
  class dummy implements Iterator {
     public function rewind() {
       //实现代码
     }
    public function valid() {
       //实现代码
    }
    public function current() {
       //实现代码
    }
    public function key() {
       //实现代码
    }
    public function next() {
       //实现代码
    }
  }
 
  foreach (new Dummy() as $v) {
  }

```
相比这种实现, 生成器提供了一种更加简便的选择, 比如实现如上同样的功能:
```

<?php
function genrators() {
   while ($i = dummy_line()) //生成数组的一个元素
   {
          yield $i;
   }
}
 
foreach (generators() as $v) {
}

```
也就是说,** 每当产生一个数组元素, 就通过yield关键字返回成一个, 并且函数执行暂停, 当返回的迭代器的next方法被调用的时候, 会恢复刚才函数的执行, 从上一次被yield暂停的位置开始继续执行, 到下一次遇到yield的时候, 再次返回.**

###  将 range() 实现为生成器 
```

<?php
function xrange($start, $limit, $step = 1) {
    if ($start < $limit) {
        if ($step <= 0) {
            throw new LogicException('Step must be +ve');
        }

        for ($i = $start; $i <= $limit; $i += $step) {
            yield $i;
        }
    } else {
        if ($step >= 0) {
            throw new LogicException('Step must be -ve');
        }

        for ($i = $start; $i >= $limit; $i += $step) {
            yield $i;
        }
    }
}

/* 
 * 注意下面range()和xrange()输出的结果是一样的。
 */

echo 'Single digit odd numbers from range():  ';
foreach (range(1, 9, 2) as $number) {
    echo "$number ";
}
echo "\n";

echo 'Single digit odd numbers from xrange(): ';
foreach (xrange(1, 9, 2) as $number) {
    echo "$number ";
}
?>
以上例程会输出：

Single digit odd numbers from range():  1 3 5 7 9 
Single digit odd numbers from xrange(): 1 3 5 7 9 

```
**指定键名来生成值**
PHP的数组支持关联键值对数组，生成器也一样支持。所以除了生成简单的值，你也可以在生成值的时候指定键名。
如下所示，生成一个键值对与定义一个关联数组十分相似。

###  PHP7中的新特性yield from 
可以通过 yield from 的新语法进入一个另外一个生成器中（生成器委托）。请参考https://wiki.php.net/rfc/generator-delegation
```

<?php
 
function g1() {
  yield 2;
  yield 3;
  yield 4;
}
 
function g2() {
  yield 1;
  yield from g1();
  yield 5;
}
 
$g = g2();
foreach ($g as $yielded) {
    var_dump($yielded);
}
 
/*
int(1)
int(2)
int(3)
int(4)
int(5)
*/

```
Delegating to an array
```

<?php
 
function g() {
  yield 1;
  yield from [2, 3, 4];
  yield 5;
}
 
$g = g();
foreach ($g as $yielded) {
    var_dump($yielded);
}
 
/*
int(1)
int(2)
int(3)
int(4)
int(5)
*/

```
```

function subgenerator() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    return 42;
}
function delegator(Generator $shared) {
    return yield from $shared;
}
 
$shared = subgenerator();
$shared->next();
$delegator = delegator($shared);
var_dump($delegator->current());
$shared->next();
while($delegator->valid()) {
    var_dump($delegator->current());
    $delegator->next();
}
var_dump($delegator->getReturn());
 
/*
int(2);
int(3);
int(4);
int(42)
*/

```
###  生成一个键值对 
```

<?php
/* 
 * 下面每一行是用分号分割的字段组合，第一个字段将被用作键名。
 */

$input = <<<'EOF'
1;PHP;Likes dollar signs
2;Python;Likes whitespace
3;Ruby;Likes blocks
EOF;

function input_parser($input) {
    foreach (explode("\n", $input) as $line) {
        $fields = explode(';', $line);
        $id = array_shift($fields);

        yield $id => $fields;
    }
}

foreach (input_parser($input) as $id => $fields) {
    echo "$id:\n";
    echo "    $fields[0]\n";
    echo "    $fields[1]\n";
}
?>

```
##  Traits language construct 
Traits不是什么新概念了, c++, java都有类似的东西, 只不过这次**PHP5.4**也引入了进来.
**Traits是一种轻量级的方法复用(相对继承来说).**为什么这么说呢? 这就好比, 如果你提供一个基类供用户去继承使用, 那么不可避免的你就需要考虑如何能让用户继承后可用, 如何避免用户改写了某些关键属性, 覆盖了某些关键方法而造成出错. 这个时候你就要考虑改用public还是private, 或者protected, 你还要考虑, 那个方法应该申明为FINAL..
而如果使用Traits, 那么你就不需要担心这些问题,** 它是一种组合方式,** 你提供的素材, 无论在任何地方都是自我一个整体.
 ```

<?php
 trait SayWorld {
   public function sayHello() {
     echo 'Hello World!';
   }
 }
 
 class MyHelloWorld extends Base {
   use SayWorld;
 }
 
 $o = new MyHelloWorld();
 $o->sayHello(); // Hello World

```
**Trait和继承以及当前类的同名函数之间的冲突, 有一套固定的解决方案, 也就是当前类的方法覆盖Trait的同名方法, 而Trait中的方法, 覆盖基类的同名方法.** 比如:
```

 <?php
 class Base {
   public function sayHello() {
     echo 'Hello ';
   }
 }
 
 trait SayWorld {
   public function sayHello() {
     parent::sayHello();
     echo 'World!';
   }
 }
 
 class MyHelloWorld extends Base {
   use SayWorld;
 }
 
 $o = new MyHelloWorld();
 $o->sayHello(); // echos Hello World

```
##  闭包与匿名函数 
http://www.php.net/manual/zh/functions.anonymous.php
php闭包是PHP5.3版本之后才有的
在编程领域我们可以通俗的说：**子函数可以使用父函数中的局部变量，这种行为就叫做闭包**。或者这么说：**闭包是指在创建时封装周围状态的函数，即便闭包所在的环境不存在了，闭包中封装的状态依然存在。**
**匿名函数没有名称，可以赋值给变量，还能被当作参数传递。匿名函数特别适合用作回调函数**。理论上闭包和匿名函数是有区别的，但是在PHP中不加以区分。
```

Example #1 匿名函数示例
<?php
echo preg_replace_callback('~-([a-z])~', function ($match) {
    return strtoupper($match[1]);
}, 'hello-world');
// 输出 helloWorld
?>

```
闭包的语法相当简单：
```

$a = function() use($b) {
 //TO-DO
};//带结束符号";"

```
<wrap em>闭包函数也可以作为变量的值来使用。PHP 会自动把此种表达式转换成内置类 Closure 的对象实例。
把一个 closure 对象赋值给一个变量的方式与普通变量赋值的语法是一样的，
最后也要加上分号：
闭包可以从父作用域中封装变量(附加状态)。 任何此类变量都应该用 use 语言结构传递进去。在PHP中必须手动调用闭包对象的bindTo()方法或者使用use关键字把状态附加到PHP闭包中(而这一切在JS中是自动的)。 
匿名函数目前是通过 Closure 类来实现的。
</wrap>
这么看来，匿名函数和闭包其实是伪装成函数的对象。
<note>我们之所以能够调用$closure变量，是因为这个变量的值是一个闭包Closure对象，而且闭包对象实现了_ _invoke()魔术方法，只要变量名后面有()，PHP就会查找并调用_ _invoke()方法。</note>

```

Example #2 匿名函数变量赋值示例
<?php
$greet = function($name)
{
    printf("Hello %s\r\n", $name);
};

$greet('World');
$greet('PHP');
?>

```
Example #3 从父作用域继承变量
```

<?php
$message = 'hello';

// 没有使用"use"
$example = function () {
    var_dump($message);
};
echo $example(); //输出 NULL

// 封装$message
$example = function () use ($message) {
    var_dump($message);
};
echo $example(); //输出string(5) "hello"

// 继承变量的值是发生在函数定义的时候而不是函数调用的时候，所以这个时候仍然输出string(5) "hello"
$message = 'world';
echo $example();//string(5) "hello"

// The changed value in the parent scope
// is reflected inside the function call
$message = 'world';
echo $example();

// Closures can also accept regular arguments
$example = function ($arg) use ($message) {
    var_dump($arg . ' ' . $message);
};
$example("hello"); //输出string(11) "hello world"
?>

```
这些变量都必须在函数或类的头部声明。 从父作用域中继承变量与使用全局变量是不同的。<wrap em>全局变量存在于一个全局的范围，
无论当前在执行的是哪个函数。而 闭包的父作用域是定义该闭包的函数（不一定是调用它的函数）。</wrap>示例如下：

闭包虽然语法和实现非常简单，但是用好却不易。
用好闭包，可以帮我们
1 减少foreach的循环的代码
2 减少函数的参数
3 解除递归函数 
大家可以去深入学习，推荐几个相关博文和资料：
http://www.cnblogs.com/yjf512/archive/2012/10/29/2744702.html 
http://blog.zol.com.cn/1722/article_1721359.html 

**闭包的bindTo方法(重要)：**
bindTo()方法为闭包增加了一些有趣的潜力，我们可以使用这个方法把Closure对象的内部状态绑定到其他对象上。bindTo()方法的第二个参数很重要，其作用是指定绑定闭包的那个对象所属的PHP类。因此，闭包可以访问绑定闭包的对象中受保护和私有的成员变量。PHP框架经常使用bindTo()方法把路由URL映射到匿名回调函数上。框架会把匿名函数绑定到应用对象上。这么做可以在这个匿名函数中使用$this关键字引用应用对象。如下所示:
```

<?php
class App
{
    protected $routes = [];
    protected $responseStatus = '200 OK';
    protected $responseContentType = 'text/html';
    protected $responseBody = 'Hello world';

    public function addRoute($routePath, $routeCallback)
    {
        $this->routes[$routePath] = $routeCallback->bindTo($this, __CLASS__);
    }

    public function dispatch($currentPath)
    {
        foreach ($this->routes as $routePath => $callback) {
            if ($routePath === $currentPath) {
                $callback();
            }
        }

        header('HTTP/1.1 ' . $this->responseStatus);
        header('Content-type: ' . $this->responseContentType);
        header('Content-length: ' . mb_strlen($this->responseBody));
        echo $this->responseBody;
    }
}

$app = new App();
$app->addRoute('/users/josh', function () {
    $this->responseContentType = 'application/json;charset=utf8';
    $this->responseBody = '{"name": "Josh"}';
});
$app->dispatch('/users/josh');


```
##  自定义endsWith函数 
http://stackoverflow.com/questions/834303/startswith-and-endswith-functions-in-php
```

function endsWith( $src,  $needle){
	if($src=="") return false;
	$pos=strlen($src)-strlen($needle);
	if($pos<0) return false;
	if(strrpos($src,$needle)===false || $pos!=strrpos($src,$needle)) return false;
	return true;
}

```
或者更简单粗暴的方式:
```

function startsWith($haystack, $needle)
{
     $length = strlen($needle);
     return (substr($haystack, 0, $length) === $needle);
}

function endsWith($haystack, $needle)
{
    $length = strlen($needle);
    if ($length == 0) {
        return true;
    }

    return (substr($haystack, -$length) === $needle);
}

```