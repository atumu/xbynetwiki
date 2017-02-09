title: phplearn26 

#  PHP学习之JSON处理 
http://cn2.php.net/manual/zh/book.json.php
  * json_decode — 对 JSON 格式的字符串进行编码
  * json_encode — 对变量进行 JSON 编码
  * json_last_error_msg — Returns the error string of the last json_encode() or json_decode() call
  * json_last_error — 返回最后发生的错误
**mixed json_decode ( string $json [, bool $assoc = false [, int $depth = 512 [, int $options = 0 ]]] )**
接受一个 JSON 格式的字符串并且把它转换为 PHP 变量
参数
json待解码的 json string 格式的字符串(UTF-8)。
assoc当该参数为 TRUE 时，将返回 array 而非 object 。
depth:User specified recursion depth.
options:Bitmask of JSON decode options. Currently only JSON_BIGINT_AS_STRING is supported (default is to cast large integers as floats)
```

<?php

$json = '{"foo-bar": 12345}';

$obj = json_decode($json);
print $obj->{'foo-bar'}; // 12345

?>

```
**string json_encode ( mixed $value [, int $options = 0 [, int $depth = 512 ]] )**
返回 value 值的 JSON 形式
value待编码的 value ，除了resource 类型之外，可以为任何数据类型,该函数只能接受 UTF-8 编码的数据
options由以下常量组成的二进制掩码： JSON_HEX_QUOT, JSON_HEX_TAG, JSON_HEX_AMP, JSON_HEX_APOS, JSON_NUMERIC_CHECK, JSON_PRETTY_PRINT, JSON_UNESCAPED_SLASHES, JSON_FORCE_OBJECT, JSON_PRESERVE_ZERO_FRACTION, JSON_UNESCAPED_UNICODE, JSON_PARTIAL_OUTPUT_ON_ERROR。 关于 JSON 常量详情参考JSON 常量页面。
depth设置最大深度。 必须大于0。
```

<?php
$arr = array ('a'=>1,'b'=>2,'c'=>3,'d'=>4,'e'=>5);

echo json_encode($arr);
?>
以上例程会输出：

{"a":1,"b":2,"c":3,"d":4,"e":5}

```
` json_encode中文支持 `
我们知道, 用PHP的json_encode来处理中文的时候, 中文都会被编码, 变成不可读的, 类似”\u* *”的格式, 还会在一定程度上增加传输的数据量.
```

<?php
echo json_encode("中文");
//"\u4e2d\u6587"

```
这就让我们这些在天朝做开发的同学, 很是头疼, 有的时候还不得不自己写json_encode.
**而在PHP5.4, 这个问题终于得以解决, Json新增了一个选项: JSON_UNESCAPED_UNICODE, 故名思议, 就是说, Json不要编码Unicode.**
看下面的例子:
```

<?php
echo json_encode("中文", JSON_UNESCAPED_UNICODE);
//"中文"

```
##  JsonSerializable抽象类 
http://php.net/manual/zh/jsonserializable.jsonserialize.php
http://www.laruence.com/2011/10/10/2204.html
Json是Ajax应用中最为通用的数据传输格式(协议), 主流的编程语言都带有对Json的支持, 在PHP中, 有json_encode/json_decode, 可以很方便的构造Json数据格式.
```

<?php
echo json_encode(array(1,2,3,4));
?>
//[1,2,3,4]

```
也可以Json化一个对象:
```

<?php
$o = new stdclass();
$o->a = 42;
echo json_encode($o);
?>
//{"a":42}

```
但这样就有个问题, **现实生活中的对象是很复杂的, Json的这种默认只对属性做操作的做法有的时候是不能解决问题的**, 比如我们希望通过私有成员来做一些计算得到最后的Json化数据, 又或者我们希望用一个字符串来代替一个object.
在以前, 那你只能自己拼凑Json串了. **在PHP5.4中, Json新增了一个JsonSerializable接口, 任何实现了这个接口的类, 需要定义一个jsonSerialize()方法, 这个方法会在对这个类的对象做Json化的时候被调用, 这个时候你就可以在这个方法内 , 随意调整最终的Json化的结果:**

JsonSerializable::jsonSerialize — 指定需要被序列化成 JSON 的数据
说明 ¶
abstract public mixed JsonSerializable::jsonSerialize ( void )
序列化物体（Object）成能被 json_encode() 原生地序列化的值。
**返回能被 json_encode() 序列化的数据**， 这个值可以是除了 resource 外的任意类型。
```

<?php
class ArrayValue implements JsonSerializable {
    public function __construct(array $array) {
        $this->array = $array;
    }

    public function jsonSerialize() {
        return $this->array;
    }
}
$array = ['foo' => 'bar', 'quux' => 'baz'];
echo json_encode(new ArrayValue($array), JSON_PRETTY_PRINT);
?>
以上例程会输出：

{
    "foo": "bar",
    "quux": "baz"
}

```