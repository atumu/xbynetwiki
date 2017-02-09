title: phplearn020 

#  PHP学习之面向对象学习2反射 
http://php.net/manual/zh/book.reflection.php
http://blog.csdn.net/hguisu/article/details/7357421
PHP 5 具有完整的反射 API，添加了对类、接口、函数、方法和扩展进行反向工程的能力。 此外，反射 API 提供了方法来取出函数、类和方法中的文档注释。
主要类:
Reflector 接口
Reflection
ReflectionClass
ReflectionExtension 获取扩展
ReflectionZendExtension
ReflectionFunction 
ReflectionFunctionAbstract 
ReflectionMethod 
ReflectionObject 
ReflectionParameter 
ReflectionProperty 
ReflectionException异常

辅助方法:
get_class — 返回对象的类名.string get_class ([ object $obj ] ).类似于标量类型的gettype()函数。
get_declared_classes()-返回由当前脚本中已定义类的名字组成的数组。包括SPL库定义的类。
class_exists() - 检查类是否已定义
get_declared_interfaces() - 返回一个数组包含所有已声明的接口。包括SPL库定义的接口
get_defined_functions() - Returns an array of all defined functions。包括SPL库定义的函数
get_loaded_extensions()
get_extension_funcs()

```

<?php

class A
{
    public    $name = 'Vasiliy';
    public    $text = 'Once upon a time';
}

class B
{
    public    $name = 'Had bought a cat';
    public    $text = 'named Valentine';
}

class grid
{
    public function build(  $dataSource, $headers, $fields  )
    {
        $result = '<table border="1">';
        $result .= $this -> make_head( $headers );
        
        foreach ($dataSource as $source):
            $class_name = get_class($source);
            if ( $class_name != FALSE ):
                $reflector = new ReflectionClass($source);
                echo 'Class "'. $class_name .'" found.<br />';
                $result .= $this -> make_row( $reflector, $fields, $source );
            endif;
        endforeach;
        
        $result .= '</table>';
        return $result;
    }
    
    private function make_head( $headers )
    {
        $result = '<tr>';
        foreach ( $headers as $header):
            $result .= "<th>$header</th>";
        endforeach;
        $result .= '</tr>';
        return $result;
    }
    
    private function make_row( $reflector, $fields, $source )
    {
        $result = '<tr>';
        foreach ( $fields as $field ):
            if ( $reflector -> hasProperty($field) ):
                $property = $reflector -> getProperty($field);
                $result .= '<td>'. $property -> getValue($source) .'</td>';
            endif;
        endforeach;
        $result .= '</tr>';
        return $result;
    }
}

$A = new A;
$B = new B;
$C = 'Test';

$dataSource = array( $A, $B, $C );
$headers = array( 'H1', 'H2' );
$fields = array( 'name', 'text' );

$grid = new grid;
$table = $grid -> build( $dataSource, $headers, $fields );
echo $table;

?>

```
##  命令行反射 
Example #1 Shell 里的一个反射例子（一个终端）
```

$ php --rf strlen
$ php --rc finfo
$ php --re json
$ php --ri dom

```

##  示例 
```

class Person {    
    /**  
     * For the sake of demonstration, we"re setting this private 
     */   
    private $_allowDynamicAttributes = false;  
   
    /** type=primary_autoincrement */  
    protected $id = 0;  
   
    /** type=varchar length=255 null */  
    protected $name;  
   
    /** type=text null */  
    protected $biography;  
   
        public function getId()  
        {  
            return $this->id;  
        }  
        public function setId($v)  
        {  
            $this->id = $v;  
        }  
        public function getName()  
        {  
            return $this->name;  
        }  
        public function setName($v)  
        {  
            $this->name = $v;  
        }  
        public function getBiography()  
        {  
            return $this->biography;  
        }  
        public function setBiography($v)  
        {  
            $this->biography = $v;  
        }  
}  

```
接下来反射它，只要把类名"Person"传递给ReflectionClass就可以了：
```

$class = new ReflectionClass('Person');//建立 Person这个类的反射类  
$instance  = $class->newInstanceArgs($args);//相当于实例化Person 类  

```
1、获取属性(Properties)：
```

$properties = $class->getProperties();  
foreach($properties as $property) {  
    echo $property->getName()."\n";  
}  
// 输出:  
// _allowDynamicAttributes  
// id  
// name  
// biography  

```
默认情况下，ReflectionClass会获取到所有的属性，private 和 protected的也可以。如果只想获取到private属性，就要额外传个参数：
$private_properties = $class->getProperties(ReflectionProperty::IS_PRIVATE);
可用参数列表：
  * ReflectionProperty::IS_STATIC
  * ReflectionProperty::IS_PUBLIC
  * ReflectionProperty::IS_PROTECTED
  * ReflectionProperty::IS_PRIVATE
如果要同时获取public 和private 属性，就这样写：ReflectionProperty::IS_PUBLIC | ReflectionProperty::IS_PROTECTED。
通过$property->getName()可以得到属性名。

2、获取注释：
通过getDocComment可以得到写给property的注释。
```

foreach($properties as $property) {  
    if($property->isProtected()) {  
        $docblock = $property->getDocComment();  
        preg_match('/ type\=([a-z_]*) /', $property->getDocComment(), $matches);  
        echo $matches[1]."\n";  
    }  
}  
// Output:  
// primary_autoincrement  
// varchar  
// text  

```
3、获取类的方法
获取方法(methods)：通过getMethods() 来获取到类的所有methods。

4）执行类的方法：
```

$instance->getBiography(); //执行Person 里的方法getBiography  
//或者：  
$ec=$class->getmethod('getName');  //获取Person 类中的getName方法  
$ec->invoke($instance);       //执行getName 方法 

```

##  附录：API详细说明 
详细说明：（例子详见php手册）
```

①Reflection类
<?php
class Reflection
{
    public static mixed export(Reflector r [,bool return])
    //导出一个类或方法的详细信息
    public static array getModifierNames(int modifiers)
    //取得修饰符的名字
}
?>

②ReflectionException类

该类继承标准类，没特殊方法和属性。

③ReflectionFunction类
<?php
class ReflectionFunction implements Reflector
{
    final private __clone()
    public object __construct(string name)
    public string __toString()
    public static string export()
    //导出该函数的详细信息
    public string getName()
    //取得函数名
    public bool isInternal()
    //测试是否为系统内部函数
    public bool isUserDefined()
    //测试是否为用户自定义函数
    public string getFileName()
    //取得文件名，包括路径名
    public int getStartLine()
    //取得定义函数的起始行
    public int getEndLine()
    //取得定义函数的结束行
    public string getDocComment()
    //取得函数的注释
    public array getStaticVariables()
    //取得静态变量
    public mixed invoke(mixed* args)
    //调用该函数，通过参数列表传参数
    public mixed invokeArgs(array args)
    //调用该函数，通过数组传参数
    public bool returnsReference()
    //测试该函数是否返回引用
    public ReflectionParameter[] getParameters()
    //取得该方法所需的参数，返回值为对象数组
    public int getNumberOfParameters()
    //取得该方法所需的参数个数
    public int getNumberOfRequiredParameters()
    //取得该方法所需的参数个数
}
?>

④ReflectionParameter类：
<?php
class ReflectionParameter implements Reflector
{
    final private __clone()
    public object __construct(string name)
    public string __toString()
    public static string export()
    //导出该参数的详细信息
    public string getName()
    //取得参数名
    public bool isPassedByReference()
    //测试该参数是否通过引用传递参数
    public ReflectionClass getClass()
    //若该参数为对象，返回该对象的类名
    public bool isArray()
    //测试该参数是否为数组类型
    public bool allowsNull()
    //测试该参数是否允许为空
    public bool isOptional()
    //测试该参数是否为可选的，当有默认参数时可选
    public bool isDefaultValueAvailable()
    //测试该参数是否为默认参数
    public mixed getDefaultValue()
    //取得该参数的默认值
}
?>

⑤ReflectionClass类：
<?php
class ReflectionClass implements Reflector
{
    final private __clone()
    public object __construct(string name)
    public string __toString()
    public static string export()
    //导出该类的详细信息
    public string getName()
    //取得类名或接口名
    public bool isInternal()
    //测试该类是否为系统内部类
    public bool isUserDefined()
    //测试该类是否为用户自定义类
    public bool isInstantiable()
    //测试该类是否被实例化过
    public bool hasConstant(string name)
    //测试该类是否有特定的常量
    public bool hasMethod(string name)
    //测试该类是否有特定的方法
    public bool hasProperty(string name)
    //测试该类是否有特定的属性
    public string getFileName()
    //取得定义该类的文件名，包括路径名
    public int getStartLine()
    //取得定义该类的开始行
    public int getEndLine()
    //取得定义该类的结束行
    public string getDocComment()
    //取得该类的注释
    public ReflectionMethod getConstructor()
    //取得该类的构造函数信息
    public ReflectionMethod getMethod(string name)
    //取得该类的某个特定的方法信息
    public ReflectionMethod[] getMethods()
    //取得该类的所有的方法信息
    public ReflectionProperty getProperty(string name)
    //取得某个特定的属性信息
    public ReflectionProperty[] getProperties()
    //取得该类的所有属性信息
    public array getConstants()
    //取得该类所有常量信息
    public mixed getConstant(string name)
    //取得该类特定常量信息
    public ReflectionClass[] getInterfaces()
    //取得接口类信息
    public bool isInterface()
    //测试该类是否为接口
    public bool isAbstract()
    //测试该类是否为抽象类
    public bool isFinal()
    //测试该类是否声明为final
    public int getModifiers()
    //取得该类的修饰符，返回值类型可能是个资源类型
    //通过Reflection::getModifierNames($class->getModifiers())进一步读取
    public bool isInstance(stdclass object)
    //测试传入的对象是否为该类的一个实例
    public stdclass newInstance(mixed* args)
    //创建该类实例
    public ReflectionClass getParentClass()
    //取得父类
    public bool isSubclassOf(ReflectionClass class)
    //测试传入的类是否为该类的父类
    public array getStaticProperties()
    //取得该类的所有静态属性
    public mixed getStaticPropertyValue(string name [, mixed default])
    //取得该类的静态属性值，若private，则不可访问
    public void setStaticPropertyValue(string name, mixed value)
    //设置该类的静态属性值，若private，则不可访问，有悖封装原则
    public array getDefaultProperties()
    //取得该类的属性信息，不含静态属性
    public bool isIterateable()
    public bool implementsInterface(string name)
    //测试是否实现了某个特定接口
    public ReflectionExtension getExtension()
    public string getExtensionName()
}
?>

⑥ReflectionMethod类：
<?php
class ReflectionMethod extends ReflectionFunction
{
    public __construct(mixed class, string name)
    public string __toString()
    public static string export()
    //导出该方法的信息
    public mixed invoke(stdclass object, mixed* args)
    //调用该方法
    public mixed invokeArgs(stdclass object, array args)
    //调用该方法，传多参数
    public bool isFinal()
    //测试该方法是否为final
    public bool isAbstract()
    //测试该方法是否为abstract
    public bool isPublic()
    //测试该方法是否为public
    public bool isPrivate()
    //测试该方法是否为private
    public bool isProtected()
    //测试该方法是否为protected
    public bool isStatic()
    //测试该方法是否为static
    public bool isConstructor()
    //测试该方法是否为构造函数
    public bool isDestructor()
    //测试该方法是否为析构函数
    public int getModifiers()
    //取得该方法的修饰符
    public ReflectionClass getDeclaringClass()
    //取得该方法所属的类
    // Inherited from ReflectionFunction
    final private __clone()
    public string getName()
    public bool isInternal()
    public bool isUserDefined()
    public string getFileName()
    public int getStartLine()
    public int getEndLine()
    public string getDocComment()
    public array getStaticVariables()
    public bool returnsReference()
    public ReflectionParameter[] getParameters()
    public int getNumberOfParameters()
    public int getNumberOfRequiredParameters()
}
?>

⑦ReflectionProperty类：
<?php
class ReflectionProperty implements Reflector
{
    final private __clone()
    public __construct(mixed class, string name)
    public string __toString()
    public static string export()
    //导出该属性的详细信息
    public string getName()
    //取得该属性名
    public bool isPublic()
    //测试该属性名是否为public
    public bool isPrivate()
    //测试该属性名是否为private
    public bool isProtected()
    //测试该属性名是否为protected
    public bool isStatic()
    //测试该属性名是否为static
    public bool isDefault()
    public int getModifiers()
    //取得修饰符
    public mixed getValue(stdclass object)
    //取得该属性值
    public void setValue(stdclass object, mixed value)
    //设置该属性值
    public ReflectionClass getDeclaringClass()
    //取得定义该属性的类
    public string getDocComment()
    //取得该属性的注释
}
?>

⑧ReflectionExtension类
<?php
class ReflectionExtension implements Reflector {
    final private __clone()
    public __construct(string name)
    public string __toString()

    public static string export()
    //导出该扩展的所有信息
    public string getName()
    //取得该扩展的名字
    public string getVersion()
    //取得该扩展的版本
    public ReflectionFunction[] getFunctions()
    //取得该扩展的所有函数
    public array getConstants()
    //取得该扩展的所有常量
    public array getINIEntries()
    //取得与该扩展相关的，在php.ini中的指令信息
    public ReflectionClass[] getClasses()
    public array getClassNames()
}
?>

```

##  PHP反射API--利用反射技术实现的插件系统架构Demo 
```

     1. /** 
     2. * @name    PHP反射API--利用反射技术实现的插件系统架构 
     3. * @author :PHPCQ.COM 
     4. */    
     5. interface Iplugin{    
     6.                 public static function getName();    
     7. }    
     8. function findPlugins(){    
     9.                 $plugins = array();    
    10.                 foreach (get_declared_classes() as $class){    
    11.                                 $reflectionClass = new ReflectionClass($class);    
    12.                                 if ($reflectionClass->implementsInterface('Iplugin')) {    
    13.                                                 $plugins[] = $reflectionClass;    
    14.                                 }    
    15.                 }    
    16.                 return $plugins;    
    17. }    
    18. function computeMenu(){    
    19.                 $menu = array();    
    20.                 foreach (findPlugins() as $plugin){    
    21.                                 if ($plugin->hasMethod('getMenuItems')) {    
    22.                                                 $reflectionMethod = $plugin->getMethod('getMenuItems');    
    23.                                                 if ($reflectionMethod->isStatic()) {    
    24.                                                                 $items = $reflectionMethod->invoke(null);    
    25.                                                 } else {    
    26.                                                                 $pluginInstance = $plugin->newInstance();    
    27.                                                                 $items = $reflectionMethod->invoke($pluginInstance);    
    28.                                                 }    
    29.                                                 $menu = array_merge($menu,$items);    
    30.                                 }    
    31.                 }    
    32.                 return $menu;    
    33. }    
    34. function computeArticles(){    
    35.                 $articles = array();    
    36.                 foreach (findPlugins() as $plugin){    
    37.                                 if ($plugin->hasMethod('getArticles')) {    
    38.                                                 $reflectionMethod = $plugin->getMethod('getArticles');    
    39.                                                 if ($reflectionMethod->isStatic()) {    
    40.                                                                 $items = $reflectionMethod->invoke(null);    
    41.                                                 } else {    
    42.                                                                 $pluginInstance = $plugin->newInstance();    
    43.                                                                 $items = $reflectionMethod->invoke($pluginInstance);    
    44.                                                 }    
    45.                                                 $articles = array_merge($articles,$items);    
    46.                                 }    
    47.                 }    
    48.                 return $articles;    
    49. }    
    50. require_once('plugin.php');    
    51. $menu = computeMenu();    
    52. $articles    = computeArticles();    
    53. print_r($menu);    
    54. print_r($articles);    
    55.     
    56.     
    57. //plugin.php 代码如下    
    58. <?php    
    59. class MycoolPugin implements Iplugin {    
    60.                 public static function getName(){    
    61.                                 return 'MycoolPlugin';    
    62.                 }    
    63.                 public static function getMenuItems(){    
    64.                                 return array(array('description'=>'MycoolPlugin','link'=>'/MyCoolPlugin'));    
    65.                 }    
    66.                 public static function getArticles(){    
    67.                                 return array(array('path'=>'/MycoolPlugin','title'=>'This is a really cool article','text'=>xxxxxxxxx));    
    68.                 }    
    69. }

``` 