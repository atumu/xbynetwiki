title: phplearn021 

#  PHP学习之面向对象学习2魔术方法和getter、setter等 
http://php.net/manual/zh/language.oop5.overloading.php
http://stackoverflow.com/questions/4478661/getter-and-setter
**PHP所提供的"重载"（overloading）是指动态地"创建"类属性和方法。我们是通过魔术方法（magic methods）来实现的。**
**当调用当前环境下未定义或不可见的类属性或方法时，重载方法会被调用**。本节后面将使用"不可访问属性（inaccessible properties）"和"不可访问方法（inaccessible methods）"来称呼这些未定义或不可见的类属性或方法。
Note:
  * 所有的重载方法都必须被声明为 public。
  * 这些魔术方法的参数都不能通过引用传递。
  * ` PHP中的"重载"与其它绝大多数面向对象语言不同。传统的"重载"是用于提供多个同名的类方法，但各方法的参数类型和个数不同。 `

###  属性重载 

```

public void __set ( string $name , mixed $value )
public mixed __get ( string $name )
public bool __isset ( string $name )
public void __unset ( string $name )
参数 $name 是指要操作的变量名称。__set() 方法的 $value 参数指定了 $name 变量的值。
在给不可访问属性赋值时，__set() 会被调用。
读取不可访问属性的值时，__get() 会被调用。
当对不可访问属性调用 isset() 或 empty() 时，__isset() 会被调用。
当对不可访问属性调用 unset() 时，__unset() 会被调用。

```
属性重载只能在对象中进行。在静态方法中，这些魔术方法将不会被调用。**所以这些方法都不能被 声明为 static**。从 PHP 5.3.0 起, 将这些魔术方法定义为 static 会产生一个警告。
###  方法重载 
```

public mixed __call ( string $name , array $arguments )
public static mixed __callStatic ( string $name , array $arguments )
在对象中调用一个不可访问方法时，__call() 会被调用。
用静态方式中调用一个不可访问方法时，__callStatic() 会被调用。(5.3新增)

```
$name 参数是要调用的方法名称。$arguments 参数是一个枚举数组，包含着要传递给方法 $name 的参数。

##  示例说明 
```

<?php 
class MyClass{
	private $f1=1;
	private $f2=[];

	public function __get($property){
          if(property_exists($this,$name)){
		return $this->$property;
          }
	}
	public function __set($name,$value){
		if(property_exists($this,$name)){
			$this->$name=$value;
		}
	}
}
$clz = new MyClass();
$clz1=new MyClass();
echo $clz->f1,"<br/>";
$clz->f1+=10;
echo $clz->f1,"<br/>";
$arr=["a","b"];
$clz->f2=$arr;
echo implode(',',$clz->f2),"<br/>";
$arr[]="c";
echo implode(',',$clz->f2),"<br/>";


$clz->f2=$clz1;
print_r($clz->f2);
echo "<br/>";
$clz1->f1=2;
print_r($clz->f2);
?>

```
输出:
1
11
a,b
a,b
MyClass Object ( [f1:MyClass:private] => 1 [f2:MyClass:private] => Array ( ) ) 
MyClass Object ( [f1:MyClass:private] => 2 [f2:MyClass:private] => Array ( ) )