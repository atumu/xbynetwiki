title: phplearn02 

#  PHP学习之数组进阶 
http://php.net/manual/zh/book.array.php
数组运算符：http://php.net/manual/zh/language.operators.array.php
![](/data/dokuwiki/php/pasted/20160410-161749.png)

` 注意：PHP数组内部会维护一个位置指针。如果出现问题，请考虑这个 `
##  数组内部指针操作 
current() - 返回数组中的当前单元.别名为pos()
each() - 返回数组中当前的键／值对并将数组指针向前移动一步
end() - 将数组的内部指针指向最后一个单元
next() - 将数组中的内部指针向前移动一位
prev() - 将数组的内部指针倒回一位
reset() — 将数组的内部指针指向第一个单元

```

<?php
$fruit = array('a' => 'apple', 'b' => 'banana', 'c' => 'cranberry');

reset($fruit);
while (list($key, $val) = each($fruit)) {
    echo "$key => $val\n";
}
?>

```
##  常用数组函数 
**01、range()函数**:建立一个包含指定范围单元的数组
array range ( mixed $start , mixed $limit [, number $step = 1 ] )
```

// The step parameter was introduced in 5.0.0
// array(0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100)
foreach (range(0, 100, 10) as $number) {
    echo $number;
}

```

02、**count()和sizeof()**:计算数组中的单元数目或对象中的属性个数
int count ( mixed $var [, int $mode = COUNT_NORMAL ] )
mode
如果可选的 mode 参数设为** COUNT_RECURSIVE**（或 1），count() 将**递归地对数组计数**。对计算多维数组的所有单元尤其有用。mode 的默认值是 0。count() 识别不了无限递归。
```

<?php
$food = array('fruits' => array('orange', 'banana', 'apple'),
              'veggie' => array('carrot', 'collard', 'pea'));
// recursive count
echo count($food, COUNT_RECURSIVE); // output 8

// normal count
echo count($food); // output 2

?>

```

03、**is_array()** - 检测变量是否是数组
04、array_rand — 从数组中随机取出一个或多个单元
mixed array_rand ( array $input [, int $num_req = 1 ] )
```

<?php
$input = array("Neo", "Morpheus", "Trinity", "Cypher", "Tank");
$rand_keys = array_rand($input, 2);
echo $input[$rand_keys[0]] . "\n";
echo $input[$rand_keys[1]] . "\n";
?>

```
05、shuffle — 将数组打乱
bool shuffle ( array &$array )
###  list()操作 
04、**list()** — 把数组中的值赋给一些变量.` list() 仅能用于数字索引的数组并假定数字索引从 0 开始。 `
array list ( mixed $varname [, mixed $... ] )
```

<?php

$info = array('coffee', 'brown', 'caffeine');

// 列出所有变量
list($drink, $color, $power) = $info;
echo "$drink is $color and $power makes it special.\n";

// 列出他们的其中一个
list($drink, , $power) = $info;
echo "$drink has $power.\n";

// 或者让我们跳到仅第三个
list( , , $power) = $info;
echo "I need $power!\n";

// list() 不能对字符串起作用
list($bar) = "abcde";
var_dump($bar); // NULL
?>

<?php

$result = mysql_query("SELECT id, name, salary FROM employees", $conn);
while (list($id, $name, $salary) = mysql_fetch_row($result)) {
    echo " <tr>\n" .
          "  <td><a href=\"info.php?id=$id\">$name</a></td>\n" .
          "  <td>$salary</td>\n" .
          " </tr>\n";
}

?>

使用嵌套的 list()
<?php
list($a, list($b, $c)) = array(1, array(2, 3));
var_dump($a, $b, $c);
?>

在 list() 中使用数组索引
<?php
$info = array('coffee', 'brown', 'caffeine');
list($a[0], $a[1], $a[2]) = $info;
var_dump($a);
?>

```
###  键值、关联、集合操作 
1、key —  返回数组中**当前单元的键名**。current() - 返回数组中的当前单元。next() - 将数组中的内部指针向前移动一位
mixed key ( array &$array )
2、array_key_exists — 检查给定的键名或索引是否存在于数组中。有个别名key_exists()
bool array_key_exists ( mixed $key , array $search )
```

<?php
$search_array = array('first' => 1, 'second' => 4);
if (array_key_exists('first', $search_array)) {
    echo "The 'first' element is in the array";
}
?>

```
3、in_array — 检查数组中是否存在某个值。
bool in_array ( mixed $needle , array $haystack [, bool $strict = FALSE ] )在 haystack 中搜索 needle，如果没有设置 strict 则使用宽松的比较。
  * needle待搜索的值。
  * haystack这个数组。
  * strict如果第三个参数 strict 的值为 TRUE 则 in_array() 函数还会检查 needle 的类型是否和 haystack 中的相同。

4、array_keys — 返回数组中部分的或所有的键名
array array_keys ( array $array [, mixed $search_value [, bool $strict = false ]] )
参数search_value如果指定了这个参数，只有包含这些值的键才会返回。

5、array_values() - 返回数组中所有的值
array array_values ( array $input )

6、array_search — 在数组中搜索给定的值，如果成功则返回相应的键名
mixed array_search ( mixed $needle , array $haystack [, bool $strict = false ] )

7、array_combine-创建一个数组，用一个数组的值作为其键名，另一个数组的值作为其值
array array_combine ( array $keys , array $values )
```

$a = array('green', 'red', 'yellow');
$b = array('avocado', 'apple', 'banana');
$c = array_combine($a, $b);

print_r($c);

```

7、array_unique() - 移除数组中重复的值
array array_unique ( array $array [, int $sort_flags = SORT_STRING ] )
Sorting type flags:
  * SORT_REGULAR - compare items normally (don't change types)
  * SORT_NUMERIC - compare items numerically
  * SORT_STRING - compare items as strings
  * SORT_LOCALE_STRING - compare items as strings, based on the current locale.

8、compact — 建立一个数组，包括变量名和它们的值.compact() 接受可变的参数数目。每个参数可以是一个包括变量名的字符串或者是一个包含变量名的数组，该数组中还可以包含其它单元内容为变量名的数组， compact() 可以递归处理。
返回输出的数组，包含了添加的所有变量。
So **compact('var1', 'var2')** is the same as saying **array('var1' => $var1, 'var2' => $var2)** as long as $var1 and $var2 are set.
###  数组分片操作 
1、array_slice — 从数组中取出一段
array array_slice ( array $array , int $offset [, int $length = NULL [, bool $preserve_keys = false ]] )
  * offset如果 offset 非负，则序列将从 array 中的此偏移量开始。如果 offset 为负，则序列将从 array 中距离末端这么远的地方开始。
  * length如果给出了 length 并且为正，则序列中将具有这么多的单元。**如果给出了 length 并且为负，则序列将终止在距离数组末端这么远的地方。**如果省略，则序列将从 offset 开始一直到 array 的末端。
  * preserve_keys注意 array_slice() **默认会重新排序并重置数组的数字索引**。你可以通过将 preserve_keys 设为 TRUE 来改变此行为。

```

<?php
$input = array("a", "b", "c", "d", "e");

$output = array_slice($input, 2);      // returns "c", "d", and "e"
$output = array_slice($input, -2, 1);  // returns "d"
$output = array_slice($input, 0, 3);   // returns "a", "b", and "c"

// note the differences in the array keys
print_r(array_slice($input, 2, -1));//return 'c','d'
print_r(array_slice($input, 2, -1, true));//
?>

```
以上例程会输出：
```

Array
(
    [0] => c
    [1] => d
)
Array
(
    [2] => c
    [3] => d
)

```

  
2、array_chunk — 将一个数组分割成多个
array array_chunk ( array $input , int $size [, bool $preserve_keys = false ] )
将一个数组分割成多个数组，其中每个数组的单元数目由 size 决定。最后一个数组的单元数目可能会少于 size 个。
```

<?php
$input_array = array('a', 'b', 'c', 'd', 'e');
print_r(array_chunk($input_array, 2));
print_r(array_chunk($input_array, 2, true));
?>

```

3、array_merge — 合并一个或多个数组。还有一个array_merge_recursive() - 递归地合并一个或多个数组
array array_merge ( array $array1 [, array $... ] )
array_merge() 将一个或多个数组的单元合并起来，一个数组中的值附加在前一个数组的后面。返回作为结果的数组。
**如果输入的数组中有相同的字符串键名，则该键名后面的值将覆盖前一个值。然而，如果数组包含数字键名，后面的值将不会覆盖原来的值，而是附加到后面。如果只给了一个数组并且该数组是数字索引的，则键名会以连续方式重新索引。**

###  栈、队列操作 
2、**array_pop** — 将数组最后一个单元弹出（出栈）。Note: 使用此函数后会重置（reset()）array 指针。
**array_shift()** - 将数组开头的单元移出数组。与array_pop相反。Note: 使用此函数后会重置（reset()）array 指针。
mixed array_pop ( array &$array )
mixed array_shift ( array &$array )

3、**array_push** — 将一个或多个单元压入数组的末尾（入栈）
**array_unshift()** - 在数组开头插入一个或多个单元。与array_push相反
int array_push ( array &$array , mixed $var [, mixed $... ] )
int array_unshift ( array &$array , mixed $var [, mixed $... ] )
array_push() 将 array 当成一个栈，并将传入的变量压入 array 的末尾**。array 的长度将根据入栈变量的数目增加。和如下效果相同**
```

<?php
$array[] = $var;
?>

```

###  排序操作 
http://php.net/manual/zh/array.sorting.php
4、sort — 对数组排序。当本函数结束时数组单元将被从最低到最高重新安排。
bool sort ( array &$array [, int $sort_flags = SORT_REGULAR ] )
可选的第二个参数 sort_flags 可以用以下值改变排序的行为：
  * SORT_REGULAR - 正常比较单元（不改变类型）
  * SORT_NUMERIC - 单元被作为数字来比较
  * SORT_STRING - 单元被作为字符串来比较
  * SORT_LOCALE_STRING - 根据当前的区域（locale）设置来把单元当作字符串比较，可以用 setlocale() 来改变。
  * SORT_NATURAL - 和 natsort() 类似对每个单元以“自然的顺序”对字符串进行排序。 PHP 5.4.0 中新增的。
  * SORT_FLAG_CASE - 能够与 SORT_STRING 或 SORT_NATURAL 合并（OR 位运算），不区分大小写排序字符串。
**PHP 有一些用来排序数组的函数， 这个文档会把它们列出来。**
主要区别有：
  * 有些函数基于 array 的键来排序， 而其他的基于值来排序的：$array['key'] = 'value';。
  * 排序之后键和值之间的关联关系是否能够保持， 是指排序之后数组的键可能 会被重置为数字型的（0,1,2 ...）。
  * 排序的顺序有：字母表顺序， 由低到高（升序）， 由高到低（降序），数字排序，自然排序，随机顺序或者用户自定义排序。
  * 注意：下列的所有排序函数都是直接作用于数组本身， 而不是返回一个新的有序的数组。
  * 以下函数对于数组中相等的元素，它们在排序后的顺序是未定义的。 （也即相等元素之间的顺序是不稳定的）。
![](/data/dokuwiki/php/pasted/20160410-155110.png)

###  数组函数式编程操作 
1、array_walk — 使用用户自定义函数对数组中的每个元素做回调处理.还有一个array_walk_recursive() - 对数组中的每个成员递归地应用用户函数
bool array_walk ( array &$array , callable $funcname [, mixed $userdata = NULL ] )
将用户自定义函数 funcname 应用到 array 数组中的每个单元。
  * 参数funcname典型情况下 funcname 接受两个参数。array 参数的值作为第一个，键名作为第二个。即funcname($value,$key)
  * Note:如果 funcname 需要直接作用于数组中的值，则给 funcname 的第一个参数指定为引用。这样任何对这些单元的改变也将会改变原始数组本身。
  * userdata**如果提供了可选参数 userdata，将被作为第三个参数传递给 callback funcname。**
```

<?php
$fruits = array("d" => "lemon", "a" => "orange", "b" => "banana", "c" => "apple");

function test_alter(&$item1, $key, $prefix)
{
    $item1 = "$prefix: $item1";
}
function test_print($item2, $key)
{
    echo "$key. $item2<br />\n";
}
echo "Before ...:\n";
array_walk($fruits, 'test_print');

array_walk($fruits, 'test_alter', 'fruit');

```

2、array_map — 将回调函数作用到给定数组的单元上。注意与array_walk()的区别。前者用于改变数组值，后者用于遍历数组元素。
array array_map ( callable $callback , array $arr1 [, array $... ] )
返回一个数组，该数组的每个元素都数组（arr1）里面的每个元素经过回调函数（callback）处理了的。
```

<?php
function cube($n)
{
    return($n * $n * $n);
}

$a = array(1, 2, 3, 4, 5);
$b = array_map("cube", $a);
print_r($b);
?>

```
array_map() using a lambda function (as of PHP 5.3.0)
```

<?php
$func = function($value) {
    return $value * 2;
};

print_r(array_map($func, range(1, 5)));
?>

```

3、array_reduce — 用回调函数迭代地将数组简化为单一的值
mixed array_reduce ( array $input , callable $function [, mixed $initial = NULL ] )
array_reduce() 将回调函数 function 迭代地作用到 input 数组中的每一个单元中，从而将数组简化为单一的值。
参数The callback function.应该形如mixed callback ( mixed &$result , mixed $item )
initial如果指定了可选参数 initial，该参数将被当成是数组中的第一个值来处理，或者如果数组为空的话就作为最终返回值。
```

<?php
function rsum($v, $w)
{
    $v += $w;
    return $v;
}
$a = array(1, 2, 3, 4, 5);
$b = array_reduce($a, "rsum");//返回15

```

4、array_filter — 用回调函数过滤数组中的单元
array array_filter ( array $array [, callable $callback [, int $flag = 0 ]] )
依次将 array 数组中的每个值传递到 callback 函数。如果 callback 函数返回 TRUE，则 input 数组的当前值会被包含在返回的结果数组中。数组的键名保留不变。
决定callback接收的参数形式:
  * ARRAY_FILTER_USE_KEY - callback接受键名作为的唯一参数
  * ARRAY_FILTER_USE_BOTH - callback同时接受键名和键值
```

<?php
function odd($value)
{
    // returns whether the input integer is odd
    return($value & 1);
}
$array1 = array("a"=>1, "b"=>2, "c"=>3, "d"=>4, "e"=>5);
echo "Odd :\n";
print_r(array_filter($array1, "odd"));
?>

```
##  附录：数组函数列表 
数组 函数
array_change_key_case — 返回字符串键名全为小写或大写的数组
array_chunk — 将一个数组分割成多个
array_column — 返回数组中指定的一列
array_combine — 创建一个数组，用一个数组的值作为其键名，另一个数组的值作为其值
array_count_values — 统计数组中所有的值出现的次数
array_diff_assoc — 带索引检查计算数组的差集
array_diff_key — 使用键名比较计算数组的差集
array_diff_uassoc — 用用户提供的回调函数做索引检查来计算数组的差集
array_diff_ukey — 用回调函数对键名比较计算数组的差集
array_diff — 计算数组的差集
array_fill_keys — 使用指定的键和值填充数组
array_fill — 用给定的值填充数组
array_filter — 用回调函数过滤数组中的单元
array_flip — 交换数组中的键和值
array_intersect_assoc — 带索引检查计算数组的交集
array_intersect_key — 使用键名比较计算数组的交集
array_intersect_uassoc — 带索引检查计算数组的交集，用回调函数比较索引
array_intersect_ukey — 用回调函数比较键名来计算数组的交集
array_intersect — 计算数组的交集
array_key_exists — 检查给定的键名或索引是否存在于数组中
array_keys — 返回数组中部分的或所有的键名
array_map — 将回调函数作用到给定数组的单元上
array_merge_recursive — 递归地合并一个或多个数组
array_merge — 合并一个或多个数组
array_multisort — 对多个数组或多维数组进行排序
array_pad — 用值将数组填补到指定长度
array_pop — 将数组最后一个单元弹出（出栈）
array_product — 计算数组中所有值的乘积
array_push — 将一个或多个单元压入数组的末尾（入栈）
array_rand — 从数组中随机取出一个或多个单元
array_reduce — 用回调函数迭代地将数组简化为单一的值
array_replace_recursive — 使用传递的数组递归替换第一个数组的元素
array_replace — 使用传递的数组替换第一个数组的元素
array_reverse — 返回一个单元顺序相反的数组
array_search — 在数组中搜索给定的值，如果成功则返回相应的键名
array_shift — 将数组开头的单元移出数组
array_slice — 从数组中取出一段
array_splice — 把数组中的一部分去掉并用其它值取代
array_sum — 计算数组中所有值的和
array_udiff_assoc — 带索引检查计算数组的差集，用回调函数比较数据
array_udiff_uassoc — 带索引检查计算数组的差集，用回调函数比较数据和索引
array_udiff — 用回调函数比较数据来计算数组的差集
array_uintersect_assoc — 带索引检查计算数组的交集，用回调函数比较数据
array_uintersect_uassoc — 带索引检查计算数组的交集，用回调函数比较数据和索引
array_uintersect — 计算数组的交集，用回调函数比较数据
array_unique — 移除数组中重复的值
array_unshift — 在数组开头插入一个或多个单元
array_values — 返回数组中所有的值
array_walk_recursive — 对数组中的每个成员递归地应用用户函数
array_walk — 使用用户自定义函数对数组中的每个元素做回调处理
array — 新建一个数组
arsort — 对数组进行逆向排序并保持索引关系
asort — 对数组进行排序并保持索引关系
compact — 建立一个数组，包括变量名和它们的值
count — 计算数组中的单元数目或对象中的属性个数
current — 返回数组中的当前单元
each — 返回数组中当前的键／值对并将数组指针向前移动一步
end — 将数组的内部指针指向最后一个单元
extract — 从数组中将变量导入到当前的符号表
in_array — 检查数组中是否存在某个值
key_exists — 别名 array_key_exists
key — 从关联数组中取得键名
krsort — 对数组按照键名逆向排序
ksort — 对数组按照键名排序
list — 把数组中的值赋给一些变量
natcasesort — 用“自然排序”算法对数组进行不区分大小写字母的排序
natsort — 用“自然排序”算法对数组排序
next — 将数组中的内部指针向前移动一位
pos — current 的别名
prev — 将数组的内部指针倒回一位
range — 建立一个包含指定范围单元的数组
reset — 将数组的内部指针指向第一个单元
rsort — 对数组逆向排序
shuffle — 将数组打乱
sizeof — count 的别名
sort — 对数组排序
uasort — 使用用户自定义的比较函数对数组中的值进行排序并保持索引关联
uksort — 使用用户自定义的比较函数对数组中的键名进行排序
usort — 使用用户自定义的比较函数对数组中的值进行排序