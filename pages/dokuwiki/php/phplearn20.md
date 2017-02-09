title: phplearn20 

#  PHP学习之面向对象学习1类 
**一，如何创建一个类**
class 类名 {
成员属性
成员方法
}
**属性**
` 通过在类定义中使用关键字public / protected /private 来声明变量 `，即创建了类的属性，也叫类的成员属性。
（注意，也可通过var来声明，但是var就是public的别名，是用在类中定义公有属性的，**只不过历史问题，现在不用var了。**）
语法：
```

class class_name{
    public $var_name;
    public $var2;
}

```
```

class Myclass { 
public $name; 
public $type; 
function Myfun(){ 
return "这是我的方法"; 
} 
}

class Nclass{  
public $name;  
public $type;  
function __construct($name){ //构造函数
    $this->name=$name; 
} 
function myfun(){  
return $this->name."演示如何使用this关键字 ";  
} 
function __destruct(){ //析构函数
    echo $this->name."已经被释放了<br>";} 
} 

function __get($name){ //通用getter
  if($this->name=$name){
	return $this->name;  
  }
}  
function __set($n,$v){  //通用setter
$this->$n=$v;  
} 
function __tostring(){  //对象打印方法
  return "这是一个测试的类";
  //还可以通过var_export()函数打印泪中的所有属性值。如
  //var_export($this,TRUE);
} 
} 

``` 
**二，实例化一个对象**
```

$myclass = new Myclass();
//只有实例化一个对象后，才可使用 
$myclass->name="phpmylove"; 
echo $myclass->name.$myclass->Myfun(); 

``` 
输出结果 phpmylove这是我的方法。


**封装关键字**
  * public 全局属性，任何地方都可以访问
  * protected 表示受保护的，只有本类的子类或父类可以访问
  * private表示私有的，只有在本类中可以访问

##  类的继承(单继承) 
```

<?   
 class Root {     
 public $name="Root";      
 function myfun()  
 {     
  return $this->name.'是一个基类';    
 }     
 }  
class Myclass extends Root {     
  public $name2;   
  function __construct($name2){  
      $this->name2 = $name2;  
      }     
     
  function myfun2(){   
  	return $this->myfun().$this->name2."这是一个派生类";       
  } 
  function myfun(){    
	return Root::myfun().$this->name2."这是一个派生类":    
 }   
}   
$myclass = new Myclass("Myclass");     
echo $myclass->myfun2();   
?> 

```
在派生类中使用父类的成员，需要用** 类名::成员属性名（或成员方法名）** 

##  抽象关键字abstract 
```

<?  
abstract class Cxclass{  
	abstract function Name();  
	abstract function Stroe();  
	abstract function Sex();  
}  

```
##  ::class的含义 
自 PHP 5.5 起，关键词 class 也可用于类名的解析。使用 ` ClassName::class ` 你可以**获取一个字符串，包含了类 ClassName 的完全限定名称。这对使用了 命名空间 的类尤其有用。**
##  范围解析操作符（::）与静态成员、self::,parent:: 
http://www.nowamagic.net/php/php_UsageOfDoubleColon.php
http://php.net/manual/zh/language.oop5.paamayim-nekudotayim.php
**范围解析操作符（::）是一对冒号，可以用于访问静态成员、方法和常量，以及被覆盖类中的成员和方法。**
主要用途如下：
  * YouClassname::在类的外部使用使用类的名字+ :: 符号访问这些静态成员、方法和常量时
  * self::在类的内部访问本类的静态成员，
  * parent::在类的内部访问父类的方法,非静态也可以。
  * self::在父类的内部访问子类的方法，非静态也可以。
  * static::访问本类静态成员
```

<?php
Class Person{
    // 定义静态成员属性
    public static $country = "中国";
    // 定义静态成员方法
    public static function myCountry() {
        //内部访问静态成员属性
        echo "我是".self::$country."人<br />";
    }
}
// 外部访问静态成员，输出静态成员属性值
echo Person::$country."<br />";
// 访问静态方法
Person::myCountry();
?>

```
**:: 访问父类覆盖的成员和方法的例子**
```

class Person {
    var $name;
    var $sex;
    var $age;

    function say() {
        echo "我的名字叫：".$this->name."<br />";
	echo "性别：".$this->sex."<br />";
	echo "我的年龄是：".$this->age;
    }
}
class Student extends Person {
    var $school;
	
    function say() {
      //访问父类方法，非静态也可以
        parent::say();
        echo "我在".$this->school."上学";
    }
}

```
总结：
**在类定义外使用的话，使用类名调用。在PHP 5.3.0，可以使用变量代替类名。**
1、Program List：用变量在类定义外部访问
```

<?php
class Fruit {
    const CONST_VALUE = 'Fruit Color';
}

$classname = 'Fruit';
echo $classname::CONST_VALUE; // As of PHP 5.3.0

echo Fruit::CONST_VALUE;
?>


```
2、Program List：在类定义外部使用::
```

  
<?php
class Fruit {
    const CONST_VALUE = 'Fruit Color';
}

class Apple extends Fruit
{
    public static $color = 'Red';

    public static function doubleColon() {
        echo parent::CONST_VALUE . "\n";
        echo self::$color . "\n";
    }
}

Apple::doubleColon();
?>


```

3、Program List：调用parent方法
```

  
<?php
class Fruit {
    const CONST_VALUE = 'Fruit Color';
}

class Apple extends Fruit
{
    public static $color = 'Red';

    public static function doubleColon() {
        echo parent::CONST_VALUE . "\n";
        echo self::$color . "\n";
    }
}

Apple::doubleColon();
?>


```

4、Program List：使用作用域限定符(注意$this)
```

<?php
    class Apple
    {
        public function showColor()
        {
            return $this->color;
        }
    }

    class Banana
    {
        public $color;

        public function __construct()
        {
            $this->color = "Banana is yellow";
        }

        public function GetColor()
        {
            return Apple::showColor();
        }
    }

    $banana = new Banana;
    echo $banana->GetColor();
?>


```
5、Program List：调用基类的方法(注意)
```

<?php
class Fruit
{
    static function color()
    {
        return "color";
    }

    static function showColor()
    {
        echo "show " . self::color();
    }
}

class Apple extends Fruit
{
    static function color()
    {
        return "red";
    }
}

Apple::showColor();
// output is "show color"!

?>
输出:show color

```


##  static 关键字 
PHP 类中定义静态的成员属性和方法使用 static 关键字。
声明类成员或方法为 static ，就可以不实例化类而直接访问，不能通过一个对象来访问其中的静态成员（静态方法除外）。静态成员属于类，不属于任何对象实例，但类的对象实例都能共享。
```

<?php
Class Person{
    // 定义静态成员属性
    public static $country = "中国";
    // 定义静态成员方法
    public static function myCountry() {
        // 内部访问静态成员属性
        echo "我是".self::$country."人<br />";
    }
}
class Student extends Person {
    function study() {
        echo "我是". parent::$country."人<br />";
    }
}
// 输出成员属性值
echo Person::$country."<br />";		// 输出：中国
$p1 = new Person();
//echo $p1->country;			// 错误写法
// 访问静态成员方法
Person::myCountry();			// 输出：我是中国人
// 静态方法也可通过对象访问：
$p1->myCountry();

// 子类中输出成员属性值
echo Student::$country."<br />";	// 输出：中国
$t1 = new Student();
$t1->study();				// 输出：我是中国人
?>

```
运行该例子，输出：
中国
我是中国人
我是中国人
中国
我是中国人
**小结**
在类内部访问静态成员属性或者方法，使用 ` self:: `（注意不是 $slef），如：
  * slef:: $country
  * slef:: myCountry()
在子类访问父类静态成员属性或方法，使用 ` parent:: `（注意不是 $parent），如：
  * parent:: $country
  * parent:: myCountry()
外部访问静态成员属性和方法为 类名/子类名:: ，如：
  * Person::$country
  * Person::myCountry()
  * Student::$country
但静态方法也可以通过普通对象的方式访问。
##  关键字 final self static const 
final 关键字 用fianl定义的类将不能被继承，定义的方法不能被重写
self 关键字 不需要实例化，直接可以访问当前类中的静态内部成员
static关键字 定义类成员的静态属性或方法，可以在不实例化的情况使用，单独占用内存，不会因为创建了多个实例化对象而重复占用同样的方法或属性  
const关键字 定义类中的常量 等于PHP中的 difine 只能在类中使用

用static定义的成员属性，在使用self::属性名访问时，不会因为实例化中重新定义而改变
```

<?  
class Myclass {  
    static $name="MT";
   // 定义常量
    const country = "中国";
    final function Myfun(){ //这里的方法用final定义  
        return "我是".self::$name."第四季第10集";  
    }  
}  
$Myclass = new Myclass();  
$Myclass->name="小德"; //这里的赋值时不起任何作用的  
echo $Myclass->Myfun();  
?>

``` 
还是会输出 我是MT第四季第10集
但是如果用$this->成员属性名，就会改变
```

<?  
class Myclass {  
    static $name="MT";  
    final function Myfun(){ //这里的方法用final定义  
        return "我是".$this->name."第四季第10集";  
    }  
}  
/*class My extends Myclass{  
    function Myfun2(){  
        return self::Myfun()."第四季第10集";  
    }  
}*/ 
$Myclass = new Myclass();  
$Myclass->name="小德";  //这里就会起作用
echo $Myclass->Myfun(); 
?> 

```

##  接口 
interface 和 implements
使用interface代替class建立接口，接口中成员必须是常量，方法是抽象方法，并且不必加abstract
```

interface demo {  
    const NAME="我叫MT";  
    function ji();  
    function ji2();  
      
    }

```
**使用implements 引用接口，单继承，多接口，先继承后接口**
##  多态性 instanceof 
instanceof  类中的运算符，功能是，测定一个给定的对象是否来自指定的对象类
```

<?    
class A {}  
class B {}  
  $a= new B();   
 if ($a instanceof A)   
    {   
     echo "A";  
    }   
 if ($a instanceof B)  
    {    
     echo "B";   
    }   
?>

```
##  _call() 方法用于监视错误的方法调用 
&lt;nowiki&gt;__call()（Method overloading）
为了避免当调用的方法不存在时产生错误，可以使用 __call() 方法来避免。该方法在调用的方法不存在时会自动调用，程序仍会继续执行下去。&lt;/nowiki&gt;
语法：
```

function __call(string $function_name, array $arguments)
{
    ......
}

```
该方法有两个参数，第一个参数 $function_name 会自动接收不存在的方法名，第二个 $args 则以数组的方式接收不存在方法的多个参数。
在类里面加入：
```

function __call($function_name, $args)
{
    echo "你所调用的函数：$function_name(参数：<br />";
    var_dump($args);
    echo ")不存在！";
}

```
当调用一个不存在的方法时（如 test() 方法）：
$p1=new Person();
$p1->test(2,"test");
输出的结果如下：
你所调用的函数：test(参数：
array(2) {
[0]=>int(2)
[1]=>string(4) "test"
}
)不存在！
##  clone 关键字与 __clone() 方法 
clone 关键字用于克隆一个完全一样的对象，_ _clone() 方法来重写原本的属性和方法。
有的时候我们需要在一个项目里面使用两个或多个一样的对象，如果使用 new 关键字重新创建对象，再赋值上相同的属性，这样做比较烦琐而且也容易出错。PHP 提供了对象克隆功能，可以根据一个对象完全克隆出一个一模一样的对象，而且克隆以后，两个对象互不干扰。
使用关键字 clone 来克隆对象。语法：
**$object2 = clone $object;**
例子：
```

<?php
class Person {
    private $name;
    private $age;

    function __construct($name, $age) {
        $this->name=$name;
        $this->age=$age;
    }

    function say() {
        echo "我的名字叫：".$this->name."<br />";
	echo "我的年龄是：".$this->age;
    }
}

$p1 = new Person("张三", 20);
$p2 = clone $p1;
$p2->say();
?>

```
运行例子，输出：
我的名字叫：张三
我的年龄是：20
**_ _clone()**
如果想在克隆后改变原对象的内容，需要在类中添加一个特殊的 _ _clone() 方法来重写原本的属性和方法。_ _clone() 方法只会在对象被克隆的时候自动调用。
```

<?php
class Person {
    private $name;
    private $age;

    function __construct($name, $age) {
        $this->name = $name;
        $this->age = $age;
    }

    function say() {
        echo "我的名字叫：".$this->name;
	echo " 我的年龄是：".$this->age."<br />";
    }
    function __clone() {
        $this->name = "我是假的".$this->name;
        $this->age = 30;
    }
}

$p1 = new Person("张三", 20);
$p1->say();
$p2 = clone $p1;
$p2->say();
?>

```
运行例子，输出：
我的名字叫：张三 我的年龄是：20
我的名字叫：我是假的张三 我的年龄是：30

##  对象的存储与传输（序列化 serialize 对象） 
**序列化对象**
对象序列化，就是将对象转换成可以存储的字节流。当我们需要把一个对象在网络中传输时或者要把对象写入文件或是数据库时，就需要将对象进行序列化。
序列化完整过程包括两个步骤：一个是序列化，就是把对象转化为二进制的字符串，` serialize() 函数 `用于序列化一个对象；另一个是反序列化，就是把对象被序列转化的二进制字符串再转化为对象，` unserialize() 函数 `来反序列化一个被序列化的对象。
语法：
string serialize( mixed value )
mixed unserialize( string str [, string callback] )
```

<?php
class Person {
    private $name;
    private $age;

    function __construct($name, $age) {
        $this->name = $name;
        $this->age = $age;
    }

    function say() {
	echo "我的名字叫：".$this->name."<br />";
	echo " 我的年龄是：".$this->age;
    }
}

$p1 = new Person("张三", 20);
$p1_string = serialize($p1);
//将对象序列化后写入文件
$fh = fopen("p1.text", "w");
fwrite($fh, $p1_string);
fclose($fh);
?>

```
打开 p1.text 文件，里面写入的内容如下：
O:6:"Person":2:{s:12:" Person name";s:4:"张三";s:11:" Person age";i:20;}
但通常不去直接解析上述序列化生成的字符。
**反序列化：**
```

<?php
class Person {
    private $name;
    private $age;

    function __construct($name, $age) {
        $this->name = $name;
        $this->age = $age;
    }

    function say() {
	echo "我的名字叫：".$this->name."<br />";
	echo " 我的年龄是：".$this->age;
    }
}

$p2 = unserialize(file_get_contents("p1.text"));
$p2 -> say();
?>

```
运行该例子，输出：
我的名字叫：张三
我的年龄是：20
**由于序列化对象不能序列化其方法，所以在 unserialize 的时候，当前文件必须包含对应的类或者 require 对应的类文件。**
##  自动加载类 __autoload() 方法 
_ _autoload() 方法用于自动加载类。
在实际项目中，不可能把所有的类都写在一个 PHP 文件中，当在一个 PHP 文件中需要调用另一个文件中声明的类时，就需要通过 include 把这个文件引入。不过有的时候，在文件众多的项目中，要一一将所需类的文件都 include 进来，一个很大的烦恼是不得不在每个类文件开头写一个长长的包含文件的列表。我们能不能在用到什么类的时候，再把这个类所在的 php 文件导入呢？
为此，PHP 提供了 _ _autoload() 方法，它会在试图使用尚未被定义的类时自动调用。通过调用此函数，脚本引擎在 PHP 出错失败前有了最后一个机会加载所需的类。
**_ _autoload() 方法接收的一个参数，就是欲加载的类的类名，所以这时候需要类名与文件名对应，如 Person.php ，对应的类名就是 Pserson 。**
例子：Pserson.php
```

<?php
class Person {
    private $name;
    private $age;

    function __construct($name, $age) {
        $this->name = $name;
        $this->age = $age;
    }

    function say() {
	echo "我的名字叫：".$this->name."<br />";
	echo " 我的年龄是：".$this->age;
    }
}
?>

```
test.php
```

<?php
function __autoload($class_name) 
{
    require_once $class_name.'.php';
}
//当前页面 Pserson 类不存在则自动调用 __autoload() 方法，传入参数 Person
$p1 = new Person("张三","20");
$p1 -> say();
?>

```
运行 test.php ，输出：
我的名字叫：张三
我的年龄是：20

##  自定义迭代器 
` 注意：现在可以通过yield关键字来实现迭代器。更简单 `
```

<?php
class ObjectIterator implements Iterator {

   private $obj;
   private $count;
   private $currentIndex;

   function __construct($obj)
   {
     $this->obj = $obj;
     $this->count = count($this->obj->data);
   }
   function rewind()
   {
     $this->currentIndex = 0;
   }
   function valid()
   {
     return $this->currentIndex < $this->count;
   }
   function key()
   {
     return $this->currentIndex;
   }
   function current()
   {
     return $this->obj->data[$this->currentIndex];
   }
   function next()
   {
     $this->currentIndex++;
   }
}

class Object implements IteratorAggregate
{
  public $data = array();

  function __construct($in)
  {
    $this->data = $in;
  }

  function getIterator()
  {
    return new ObjectIterator($this);
  }
}

$myObject = new Object(array(2, 4, 6, 8, 10));

$myIterator = $myObject->getIterator();
for($myIterator->rewind(); $myIterator->valid(); $myIterator->next())
{
  $key = $myIterator->key();
  $value = $myIterator->current();
  echo $key." => ".$value."<br />";
}
?>

```

参考：http://phpmylove.blog.51cto.com/3389265/677184
http://www.5idev.com/p-php_method_overloading.shtml