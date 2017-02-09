title: phplearn01 

#  PHP学习之字符串处理 
http://php.net/manual/zh/book.strings.php
配置字符编码php.ini:
default_charset="UTF-8"
##  基础 
**一，大括号的使用**
说明，为了区分变量和字符串，
```

<?  
$a="basket";  
$b="i will play {$a}ball in the summertime!!";  
?>

``` 
输出结果 i will play basketball in the summertime!!
为了区分变量$a和ball不是变量$aball

二，字符串的索引 和数组一样，**字符串也可以通过索引输出**
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$a="hello my php!!";  
for ($i=0;$i<15;$i++)  
{  
    echo $a[$i]."<br>";  
}  
?>

```
**三 字符串的连接符，.和.=**
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$a="hello my php!!";  
$b="i will give you a job!!";  
$a.=$b;  
echo $a;  
?>

```
**四，串联字符串**
&lt;nowiki&gt;$变量名=&lt;&lt;&lt;开始表示&lt;/nowiki&gt;
字符串
结束表示;
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$a=<<<sql  
select * from mysql   
where channl_id=0 order  
by channl_id desc  
sql;  
echo $a;  
?>

``` 
输出结果 select * from mysql where channl_id=0 order by channl_id desc

**输出echo print printf print_r**
printf('%.2f',$a);  

**获取字符串的长度 strlen**
语法 strlen($变量名)

**trim ltrim rtrim**
trim($变量名,"被去除的字符串")

**改变字符串大小写**
共四个参数 变为小写strtolower 变为大写strtoupper 首字母大写ucfirst 每个词首字母大写ucwords
语法 strtolower($a) strtoupper($a) ucfirst($a) ucwords($a)

##  常用函数总结 
0、trim() - 去除字符串首尾处的空白字符（或者其他字符）。rtrim(),ltrim()
1、strlen — 获取字符串长度
2、lcfirst — 使一个字符串的第一个字符小写
3、ucfirst() - 将字符串的首字母转换为大写
4、strtolower() - 将字符串转化为小写
5、strtoupper() - 将字符串转化为大写
6、ucwords() - 将字符串中每个单词的首字母转换为大写

###  字符串输出 
1、echo — 输出一个或多个字符串
2、print - 输出字符串
3、printf() - 输出格式化字符串
int printf ( string $format [, mixed $args [, mixed $... ]] )
4、string sprintf ( string $format [, mixed $args [, mixed $... ]] )**sprintf与printf的区别是，sprintf不会打印，它会返回格式化后的字符串。**
参数format:
必需。规定字符串以及如何格式化其中的变量。
可能的格式值：
```

%% - 返回一个百分号 %
%b - 二进制数
%c - ASCII 值对应的字符
%d - 包含正负号的十进制数（负数、0、正数）
%e - 使用小写的科学计数法（例如 1.2e+2）
%E - 使用大写的科学计数法（例如 1.2E+2）
%u - 不包含正负号的十进制数（大于等于 0）
%f - 浮点数（本地设置）
%F - 浮点数（非本地设置）
%g - 较短的 %e 和 %f
%G - 较短的 %E 和 %f
%o - 八进制数
%s - 字符串
%x - 十六进制数（小写字母）
%X - 十六进制数（大写字母）
附加的格式值。必需放置在 % 和字母之间（例如 %.2f）：
+ （在数字前面加上 + 或 - 来定义数字的正负性。默认地，只有负数做标记，正数不做标记）
' （规定使用什么作为填充，默认是空格。它必须与宽度指定器一起使用。）
- （左调整变量值）
[0-9] （规定变量值的最小宽度）
.[0-9] （规定小数位数或最大字符串长度）
注释：如果使用多个上述的格式值，它们必须按照上面的顺序进行使用，不能打乱。

```
4、flush() - 刷新输出缓冲
###  转义相关 
1、**addslashes** — 使用反斜线转义特殊字符串
PHP 5.4 之前 PHP 指令`  magic_quotes_gpc ` 默认是 on， 实际上所有的 GET、POST 和 COOKIE 数据都用被 addslashes() 了。 **不要对已经被 magic_quotes_gpc 转义过的字符串使用 addslashes()，因为这样会导致双层转义。 遇到这种情况时可以使用函数 get_magic_quotes_gpc() 进行检测。**
` 但是在PHP5.4之后已经移除了magic_quotes_gpc配置和get_magic_quotes_gpc()函数。所以在这之后的版本需要显式直接使用addslashes()对表单数据进行转义。 `
addslashes返回字符串，该字符串为了数据库查询语句等的需要在某些字符前加上了反斜线。这些字符是单引号（'）、双引号（"）、反斜线（\）与 NUL（NULL 字符）。
**一个使用 addslashes() 的例子是当你要往数据库中输入数据时。** 例如，将名字 O'reilly 插入到数据库中，这就需要对其进行转义。 **强烈建议使用 DBMS 指定的转义函数 （比如 MySQL 是 mysqli_real_escape_string()，PostgreSQL 是 pg_escape_string()）**，但是如果你使用的 DBMS 没有一个转义函数，并且使用 \ 来转义特殊字符，你可以使用这个函数。
```

<?php
$str = "Is your name O'reilly?";

// 输出： Is your name O\'reilly?
echo addslashes($str);
?>

```
2、**stripslashes()** - 反转义一个使用 addcslashes 转义的字符串
当从数据库取被addcslashes转义的字符串时需要注意使用此函数进行反转义。

3、**htmlspecialchars()** - 转义HTML特殊字符为HTML实体
HTML特殊字符有:
  * '&' (ampersand) becomes '&amp;'
  * '"' (double quote) becomes '&quot;' when ENT_NOQUOTES is not set.
  * "'" (single quote) becomes '&#039;' (or &apos;) only when ENT_QUOTES is set.
  * '<' (less than) becomes '&lt;'
  * '>' (greater than) becomes '&gt;'
4、**htmlspecialchars_decode()** - 将特殊的 HTML 实体转换回普通字符
5、htmlentities-转义所有的HTML标记.更为强大.为防止乱码，使用时应该指定第三个参数，例如：
```

echo htmlentities($str, ENT_QUOTES, "UTF-8");

```
5、**strip_tags()** - 从字符串中去除 HTML 和 PHP 标记
6、**nl2br()** - 在字符串所有新行之前插入 HTML 换行标记
###  字符串分割、合并 
分割
0、substr() - 返回字符串的子串。stristr() - strstr 函数的忽略大小写版本
string substr ( string $string , int $start [, int $length ] )

1、str_split() - 将字符串转换为数组
array str_split ( string $string [, int $split_length = 1 ] )
```

$str = "Hello Friend";
$arr2 = str_split($str, 3);//输出一个数组array('hel','lo ','Fri','end');

```
2、explode() - 使用一个字符串分割另一个字符串为数组、implode则是合并数组到一个字符串。
array explode ( string $delimiter , string $string [, int $limit ] )
```

// 示例 2
$data = "foo:*:1023:1000::/home/foo:/bin/sh";
list($user, $pass, $uid, $gid, $gecos, $home, $shell) = explode(":", $data);
echo $user; // foo
echo $pass; // *

```
3、split() - 用正则表达式将字符串分割到数组中
array split ( string $pattern , string $string [, int $limit ] )
**preg_split() 函数使用了 Perl 兼容正则表达式语法，通常是比 split() 更快的替代方案。如果不需要正则表达式的威力，则使用 explode() 更快**，这样就不会招致正则表达式引擎的浪费。

4、wordwrap() - 打断字符串为指定数量的字串
string wordwrap ( string $str [, int $width = 75 [, string $break = "\n" [, bool $cut = false ]]] )
使用字符串断点将字符串打断为指定数量的字串。
  * str输入字符串。
  * width列宽度。
  * break使用可选的 break 参数打断字符串。
```

<?php
$text = "The quick brown fox jumped over the lazy dog.";
$newtext = wordwrap($text, 20, "<br />\n");

echo $newtext;
?>
以上例程会输出：

The quick brown fox<br />
jumped over the lazy<br />
dog.

```
4、strstr
string strstr ( string $haystack , mixed $needle [, bool $before_needle = false ] )
```

<?php
$email  = 'name@example.com';
$domain = strstr($email, '@');
echo $domain; // 打印 @example.com

$user = strstr($email, '@', true); // 从 PHP 5.3.0 起
echo $user; // 打印 name
?>

```
5、chunk_split — 将字符串分割成小块。不支持中文字符。解决方案见下面。
string chunk_split ( string $body [, int $chunklen = 76 [, string $end = "\r\n" ]] )
  * body要分割的字符。
  * chunklen分割的尺寸。
  * end行尾序列符号。
```

<?php
//对于中文字符
function chunk_split_unicode($str, $l = 76, $e = "\r\n") {
 $tmp = array_chunk(
        preg_split("//u", $str, -1, PREG_SPLIT_NO_EMPTY), $l);
    $str = "";
    foreach ($tmp as $t) {
        $str .= join("", $t) . $e;
    }
    return $str;
};
$str = "我们是中国人，你是谁呢，哈哈哈。";
echo nl2br(chunk_split($str, 4))."<br/>";
//对于中文字符请使用自定义的chunk_split_unicode.
echo nl2br(chunk_split_unicode($str, 4));
?>

```
输出：
我�
����
�中
国�
����
�你
是�
����
�，
哈�
����
�。

我们是中
国人，你
是谁呢，
哈哈哈。


合并：
1、implode — 将一个一维数组的值转化为字符串。还有一个别名join
string implode ( string $glue , array $pieces )
用 glue 将一维数组的值连接为一个字符串。
```

$array = array('lastname', 'email', 'phone');
$comma_separated = implode(",", $array);

echo $comma_separated; // lastname,email,phone

```
###  字符编码转换与中文处理 
http://php.net/manual/zh/refs.international.php
1、ord — 返回字符的 ASCII 码值
Linux下建议安装libiconv库。
2、iconv — 字符串按要求的字符编码来转换
string iconv ( string $in_charset , string $out_charset , string $str )

3、iconv_strlen — 返回字符串的字符数统计。相比strlen()，它识别中文。如strlen("我")返回3，而iconv_strlen("我")返回1
iconv_strpos — Finds position of first occurrence of a needle within a haystack
iconv_strrpos — Finds the last occurrence of a needle within a haystack
iconv_substr — 截取字符串的部分

mbstring 提供了针对多字节字符串的函数，能够帮你处理 PHP 中的多字节编码。 除此以外，mbstring 还能在可能的字符编码之间相互进行编码转换。 为了方便起见，mbstring 设计成了处理基于 Unicode 的编码，类似 UTF-8、UCS-2 及诸多单字节的编码（在以下列出了）。
` 首先需要启用mbstring扩展 `编辑php.ini
extension=php_mbstring.dll
4、mb_convert_encoding — 转换字符的编码
string mb_convert_encoding ( string $str , string $to_encoding [, mixed $from_encoding = mb_internal_encoding() ] )
```

/* 将 GB2312 转换成 UTF-8 */
$str = mb_convert_encoding($str, "UTF-8", "GB2312");

```
mb_send_mail — 发送编码过的邮件
mb_split — 使用正则表达式分割多字节字符串
mb_strcut — 获取字符的一部分
mb_strimwidth — 获取按指定宽度截断的字符串
mb_stripos — 大小写不敏感地查找字符串在另一个字符串中首次出现的位置
mb_stristr — 大小写不敏感地查找字符串在另一个字符串里的首次出现
mb_strlen — 获取字符串的长度
mb_strpos — 查找字符串在另一个字符串中首次出现的位置
mb_strrchr — 查找指定字符在另一个字符串中最后一次的出现
mb_strrichr — 大小写不敏感地查找指定字符在另一个字符串中最后一次的出现
mb_strripos — 大小写不敏感地在字符串中查找一个字符串最后出现的位置
mb_strrpos — 查找字符串在一个字符串中最后出现的位置
mb_strstr — 查找字符串在另一个字符串里的首次出现
mb_strtolower — 使字符串小写
mb_strtoupper — 使字符串大写
mb_strwidth — 返回字符串的宽度
mb_substitute_character — 设置/获取替代字符
mb_substr_count — 统计字符串出现的次数
mb_substr — 获取字符串的部分
###  字符串替换 
1、str_replace — 子字符串替换。str_ireplace() - str_replace 的忽略大小写版本
mixed str_replace ( mixed $search , mixed $replace , mixed $subject [, int &$count ] )
默认全部替换
如果没有一些特殊的替换需求（比如正则表达式），你应该使用该函数替换 ereg_replace() 和 preg_replace()。

2、str_ireplace() - str_replace 的忽略大小写版本
3、substr_replace() - 替换字符串的子串
mixed substr_replace ( mixed $string , mixed $replacement , mixed $start [, mixed $length ] )

4、preg_replace() - 执行一个正则表达式的搜索和替换
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )

5、strtr() - 转换指定字符

###  字符串查找 
1、stripos — 查找字符串首次出现的位置（不区分大小写）。与 strpos() 不同，stripos() 不区分大小写。
int stripos ( string $haystack , string $needle [, int $offset = 0 ] )
2、strpos() - 查找字符串首次出现的位置
3、strrpos() - 计算指定字符串在目标字符串中最后一次出现的位置
4、strripos() - 计算指定字符串在目标字符串中最后一次出现的位置（不区分大小写）


###  加密与散列 
1、crypt — 单向字符串散列.使用hash_equals()函数进行相等验证。
string crypt ( string $str , string $salt  )
password_hash()使用了一个强的哈希算法，来产生足够强的盐值，并且会自动进行合适的轮次。password_hash()是crypt()的一个简单封装，并且完全与现有的密码哈希兼容。推荐使用password_hash()。
```

<?php
$hashed_password = crypt('mypassword'); // 自动生成盐值

/* 你应当使用 crypt() 得到的完整结果作为盐值进行密码校验，以此来避免使用不同散列算法导致的问题。（如上所述，基于标准 DES 算法的密码散列使用 2 字符盐值，但是基于 MD5 算法的散列使用 12 个字符盐值。）*/
if (hash_equals($hashed_password, crypt($user_input, $hashed_password))) {
   echo "Password verified!";
}
?>

```
```

 以不同散列类型使用 crypt()

<?php
if (CRYPT_STD_DES == 1) {
    echo 'Standard DES: ' . crypt('rasmuslerdorf', 'rl') . "\n";
}

if (CRYPT_EXT_DES == 1) {
    echo 'Extended DES: ' . crypt('rasmuslerdorf', '_J9..rasm') . "\n";
}

if (CRYPT_MD5 == 1) {
    echo 'MD5:          ' . crypt('rasmuslerdorf', '$1$rasmusle$') . "\n";
}

if (CRYPT_BLOWFISH == 1) {
    echo 'Blowfish:     ' . crypt('rasmuslerdorf', '$2a$07$usesomesillystringforsalt$') . "\n";
}

if (CRYPT_SHA256 == 1) {
    echo 'SHA-256:      ' . crypt('rasmuslerdorf', '$5$rounds=5000$usesomesillystringforsalt$') . "\n";
}

if (CRYPT_SHA512 == 1) {
    echo 'SHA-512:      ' . crypt('rasmuslerdorf', '$6$rounds=5000$usesomesillystringforsalt$') . "\n";
}
?>

```

2、password_hash() - 创建密码的哈希（hash）。使用password_verify()函数进行相等验证
string password_hash ( string $password , integer $algo [, array $options ] )
当前支持的算法：
PASSWORD_DEFAULT - 使用 bcrypt 算法 (PHP 5.5.0 默认)。 因此，数据库里储存结果的列可超过60个字符（最好是255个字符）。
PASSWORD_BCRYPT - 使用 CRYPT_BLOWFISH 算法创建哈希。 这会产生兼容使用 "$2y$" 的 crypt()。 结果将会是 60 个字符的字符串， 或者在失败时返回 FALSE。
$options支持的选项：
  * salt - 手动提供哈希密码的盐值（salt）。这将避免自动生成盐值（salt）。省略此值后，password_hash() 会为每个密码哈希自动生成随机的盐值。这种操作是有意的模式。
**盐值（salt）选项从 PHP 7.0.0 开始被废弃（deprecated）了**。 现在最好选择简单的使用默认产生的盐值。
  * cost - 代表算法使用的 cost。crypt() 页面上有 cost 值的例子。省略时，默认值是 10。 这个 cost 是个不错的底线，但也许可以根据自己硬件的情况，加大这个值。
使用的算法、cost 和盐值作为哈希的一部分返回。所以验证哈希值的所有信息都已经包含在内。 **这使`  password_verify() ` 函数验证的时候，不需要额外储存盐值或者算法的信息。**
```

<?php
echo password_hash("rasmuslerdorf", PASSWORD_DEFAULT)."\n";
?>
<?php
// 想知道以下字符从哪里来，可参见 password_hash() 的例子
$hash = '$2y$07$BCryptRequires22Chrcte/VlQH0piJtjXl.0t1XkA8pw9dMXTpOq';

if (password_verify('rasmuslerdorf', $hash)) {
    echo 'Password is valid!';
} else {
    echo 'Invalid password.';
}
?>

$options = [
    'cost' => 12,
];
echo password_hash("rasmuslerdorf", PASSWORD_BCRYPT, $options)."\n";

```

3、md5 — 计算字符串的 MD5 散列值
string md5 ( string $str [, bool $raw_output = false ] )
还有以下几个函数
  * md5_file() - 计算指定文件的 MD5 散列值
  * sha1_file() - 计算文件的 sha1 散列值
  * crc32() - 计算一个字符串的 crc32 多项式
  * sha1() - 计算字符串的 sha1 散列值
  * hash() - 生成哈希值 （消息摘要）


##  查找 
**在字符串中查找 substr**
语法 substr(查找的字符串,开始位置,从开始位置起多少字符)
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$st="mysql php .net";  
$str=substr($st,2,5);  
echo $st."<br>";  
    echo $str;  
?>

```

**查找在字符串中出现的第一次的位置，和最后一次出现的位置**
语法 strpos(欲查找的字符串,查找的元素) strrpos（欲查找的字符串,查找的元素）
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$a="welcome to php,php is so easy!!";  
$b=strpos($a,'php');  
$c=strrpos($a,'php');  
echo $b."<br>";  
    echo $c;  
 
?> 

```
##  分解字符串 
**explode 语法，分割字符串为数组**
explode('分隔符',欲分解的字符串)
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$a="asp,php,asp.net";  
$b=explode(',',$a);  
print_r($b);  

?>

``` 
输出结果
Array ( [0] => asp [1] => php [2] => asp.net )  说明:以逗号为分割依据，把字符串分割成数组

**获取字符串分解后最后一个 end(explode())**
end(explode('以什么特征为依据',欲分解的字符串))
多用于截取文件名(也可以basename(路径))
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$filename="/Uplodefile/new/2009/02/18/12606.168.24.jpg";  
$b=end(explode('/',$filename));  
echo $b;  
?> 

``` 
输出结果 12606.168.24.jpg

**按照自定义个数截取字符串 str_split**
语法 str_split(要截取的字符串,截取多少个字符)
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$string="hello world.";  
$b=str_split($string,3);  
foreach ($b as $v)  
{  
    echo $v."<br>";  
}  
?> 

```
输出结果
hel
lo 
wor
ld.
##  将数组转化成字符串 implode 
语法 implode(以什么分割,$欲转换的数组)
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$arr=array('张超','赵永峰','郭磊','王晨');  
$b=implode(',',$arr);  
echo $b.'<br>';  
var_dump($b);  
?> 

``` 
输出结果
张超,赵永峰,郭磊,王晨
string(30) "张超,赵永峰,郭磊,王晨"
##  替换字符串 str_replace 
**语法 str_replace('查找要替换的值','替换后的值',替换的字符串)**
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$content=$_POST['content'];  
$con1=str_replace('张超','方头怪人',$content);  
echo $con1.'<br>';  
?>

```  
##  加密函数 md5 
**语法md5(欲加密的字符串)**
```

<?  
header("Content-Type:text/html; charset=utf-8");  
$content=$_POST['content'];  
$con1=md5($content);  
echo $con1.'<br>';  
?> 

``` 


##  字符串处理函数总结 
字符串输出
  * echo()：输出一个或多个字符串，void echo ( string arg1 [, string ...] )
说明：**双引号内的变量会被解释，而单引号内的变量则原样输出**。字符串计算是从 0 开始计数
  * print()：输出一个字符串。int print( string arg )但只能有一个参数，其用法同 echo ，但不能输出数组和对象。
  * printf()：输出格式化字符串。int printf(string format, arg1, arg2,  ...)
![](/data/dokuwiki/php/pasted/20160407-152431.png)
```

<?php
$str = "This";
$number = 31;
printf("%s month has %u days",$str,$number);    //输出 This month has 31 days
?>

```
字符串去除
  * trim()：去除字符串 首尾 空白等特殊符号或指定字符序列。string trim(string str[, charlist])
  * ltrim()：去除字符串 首 空白等特殊符号或指定字符序列
  * rtrim()：去除字符串 尾 空白等特殊符号或指定字符序列
  * chop()：同 rtrim()
字符串连接
  * implode()：使用字符将数组的内容组合成一个字符串。string implode( string sep, array array )
  * join()：同 implode()
字符串分割
  * explode()：使用一个字符串分割另一个字符串。array explode( string separator, string string [, int limit] )
  * str_split()：将字符串分割到数组中。array str_split( string string [, int length] )
字符串获取
  * substr()：从字符串中获取其中的一部分。string substr ( string string, int start [, int length] )
  * strstr()：查找字符串在另一个字符串中第一次出现的位置，并返回从该位置到字符串结尾的所有字符。string strstr ( string string, string needle )
  * subchr()：同 strstr()
  * strrchr()：查找字符串在另一个字符串中最后一次出现的位置，并返回从该位置到字符串结尾的所有字符
字符串替换
  * substr_replace()：把字符串的一部分替换为另一个字符串。mix substr_replace ( mixed string, string replacement, int start [, int length] )
  * str_replace()：使用一个字符串替换字符串中的另一些字符。mixed str_replace( mixed search, mixed replace, mixed string [, int &count] )。如需进行大小写不敏感的查找替换，请使用 str_ireplace()
字符串计算
  * strlen()：取得字符串的长度
  * strpos()：定位字符串第一次出现的位置。int strpos ( string string, mixed needle [, int start] )
  * strrpos()：定位字符串最后一次出现的位置
字符串 XHTML 格式化显示
  * nl2br()：将换行符 n 转换成 XHTML 换行符 <br />
  * htmlspecialchars()：把一些特殊字符转换为 HTML 实体
转换的特殊字符如下：
& 转换为 &amp;
" 转换为 &quot;
< 转换为 &lt;
&lt;nowiki&gt;&gt; 转换为 &gt;&lt;/nowiki&gt;
  * htmlspecialchars_decode()：把一些 HTML 实体转换为特殊字符，htmlspecialchars() 的反函数
字符串存储（转义）
  * addslashes()：对特殊字符加上转义字符。
  * stripslashes()：addslashes() 的反函数。
提示：如果需要更复杂的字符串处理，可以使用正则表达式，具体参看《PHP 正则表达式》。
###  PHP 字符串存储（转义） addslashes 与 stripslashes 函数 
PHP 字符串存储（转义）
**PHP 的字符串向数据库进行写入时，为避免数据库错误，需要对特殊字符进行转义**（字符前加上 符号）。
这些特殊字符包括：单引号（'）、双引号（"）、反斜线（\）与 NUL（NULL 字符）。
` addslashes() 函数 `用于对特殊字符加上转义字符，返回一个字符串。
语法：string addslashes ( string string )
```

<?php
$str = "Is your name O'reilly?";
echo addslashes($str);		// 输出：Is your name O\'reilly?
?>

```
**默认情况下，PHP 指令 magic_quotes_gpc 为 on，系统会对所有的 GET、POST 和 COOKIE 数据自动运行 addslashes() 。不要对已经被 magic_quotes_gpc 转义过的字符串使用 addslashes() ，因为这样会导致双层转义。**
<wrap em>可以对 get_magic_quotes_gpc() 进行检测以便确定是否需要使用 addslashes() ：
当用户输入提交到数据库时必须进行转义</wrap>
```

<?php
if (!get_magic_quotes_gpc()) {
    $lastname = addslashes($_POST['lastname']);
} else {
    $lastname = $_POST['lastname'];
}
echo $lastname;			//转义后的字符如：O'reilly
?>

```
**stripslashes()**
该函数为 addslashes() 的反函数，返回一个字符串。
语法：string stripslashes ( string string )
例子：
```

<?php
$str = "Is your name O\'reilly?";
echo stripslashes($str);	// 输出：Is your name O'reilly?
?>

```

##  判断字母大小写 
方式一：使用 ord() 判断：
```

<?php
$str = 'A';
function checkCase1($str){
	$str = ord($str);
	if($str>64&&$str<91){
		echo '大写字母';
	}
	else if($str>96&&$str<123){
		echo '小写字母';
	}else
		echo '不是字母';
}

```
方式二：
```

function checkCase2($str){
if(strtoupper($str)===$str){
echo '大写字母';
}else{
echo '小写字母';
}
}
echo checkCase1($str);
echo checkCase2($str);
?>

```
方式三、使用正则表达式判断：(速度最慢)
```

function checkCase($str){
if(preg_match(‘/^[a-z]+$/’, $str)){
echo ‘小写字母’;
}elseif(preg_match(‘/^[A-Z]+$/’, $str)){
echo ‘大写字母’;
}
}

```

##  gettext处理国际化 
http://www.laruence.com/2009/07/19/1003.html


##  字符编码转换 
incov()函数可以进行字符编码转换。
```

iconv('GB2312','UTF-8',`dir c:`);

```
mb_convert_encoding — 转换字符的编码
```

/* 将 GB2312 转换成 UTF-8 */
$str = mb_convert_encoding($str, "UTF-8", "GB2312");

```
##  多字节字符串处理与mb_string 
**mb_convert_encoding** — 转换字符的编码
string mb_convert_encoding ( string $str , string $to_encoding [, mixed $from_encoding = mb_internal_encoding() ] )
**mb_detect_encoding** — 检测字符的编码
mb_send_mail — 发送编码过的邮件
mb_split — 使用正则表达式分割多字节字符串
mb_strcut — 获取字符的一部分
mb_strimwidth — 获取按指定宽度截断的字符串
mb_stripos — 大小写不敏感地查找字符串在另一个字符串中首次出现的位置
mb_stristr — 大小写不敏感地查找字符串在另一个字符串里的首次出现
**mb_strlen** — 获取字符串的长度
**mb_strpos** — 查找字符串在另一个字符串中首次出现的位置
mb_strrchr — 查找指定字符在另一个字符串中最后一次的出现
mb_strrichr — 大小写不敏感地查找指定字符在另一个字符串中最后一次的出现
mb_strripos — 大小写不敏感地在字符串中查找一个字符串最后出现的位置
mb_strrpos — 查找字符串在一个字符串中最后出现的位置
mb_strstr — 查找字符串在另一个字符串里的首次出现
mb_strtolower — 使字符串小写
mb_strtoupper — 使字符串大写
mb_strwidth — 返回字符串的宽度
mb_substitute_character — 设置/获取替代字符
mb_substr_count — 统计字符串出现的次数
mb_substr — 获取字符串的部分
##  附录：字符串函数 
字符串 函数
addcslashes — 以 C 语言风格使用反斜线转义字符串中的字符
addslashes — 使用反斜线引用字符串
bin2hex — 函数把ASCII字符的字符串转换为十六进制值
chop — rtrim 的别名
chr — 返回指定的字符
chunk_split — 将字符串分割成小块
convert_cyr_string — 将字符由一种 Cyrillic 字符转换成另一种
convert_uudecode — 解码一个 uuencode 编码的字符串
convert_uuencode — 使用 uuencode 编码一个字符串
count_chars — 返回字符串所用字符的信息
crc32 — 计算一个字符串的 crc32 多项式
crypt — 单向字符串散列
echo — 输出一个或多个字符串
explode — 使用一个字符串分割另一个字符串
fprintf — 将格式化后的字符串写入到流
get_html_translation_table — 返回使用 htmlspecialchars 和 htmlentities 后的转换表
hebrev — 将逻辑顺序希伯来文（logical-Hebrew）转换为视觉顺序希伯来文（visual-Hebrew）
hebrevc — 将逻辑顺序希伯来文（logical-Hebrew）转换为视觉顺序希伯来文（visual-Hebrew），并且转换换行符
hex2bin — 转换十六进制字符串为二进制字符串
html_entity_decode — Convert all HTML entities to their applicable characters
htmlentities — Convert all applicable characters to HTML entities
htmlspecialchars_decode — 将特殊的 HTML 实体转换回普通字符
htmlspecialchars — Convert special characters to HTML entities
implode — 将一个一维数组的值转化为字符串
join — 别名 implode
lcfirst — 使一个字符串的第一个字符小写
levenshtein — 计算两个字符串之间的编辑距离
localeconv — Get numeric formatting information
ltrim — 删除字符串开头的空白字符（或其他字符）
md5_file — 计算指定文件的 MD5 散列值
md5 — 计算字符串的 MD5 散列值
metaphone — Calculate the metaphone key of a string
money_format — Formats a number as a currency string
nl_langinfo — Query language and locale information
nl2br — 在字符串所有新行之前插入 HTML 换行标记
number_format — 以千位分隔符方式格式化一个数字
ord — 返回字符的 ASCII 码值
parse_str — 将字符串解析成多个变量
print — 输出字符串
printf — 输出格式化字符串
quoted_printable_decode — 将 quoted-printable 字符串转换为 8-bit 字符串
quoted_printable_encode — 将 8-bit 字符串转换成 quoted-printable 字符串
quotemeta — 转义元字符集
rtrim — 删除字符串末端的空白字符（或者其他字符）
setlocale — 设置地区信息
sha1_file — 计算文件的 sha1 散列值
sha1 — 计算字符串的 sha1 散列值
similar_text — 计算两个字符串的相似度
soundex — Calculate the soundex key of a string
sprintf — Return a formatted string
sscanf — 根据指定格式解析输入的字符
str_getcsv — 解析 CSV 字符串为一个数组
str_ireplace — str_replace 的忽略大小写版本
str_pad — 使用另一个字符串填充字符串为指定长度
str_repeat — 重复一个字符串
str_replace — 子字符串替换
str_rot13 — 对字符串执行 ROT13 转换
str_shuffle — 随机打乱一个字符串
str_split — 将字符串转换为数组
str_word_count — 返回字符串中单词的使用情况
strcasecmp — 二进制安全比较字符串（不区分大小写）
strchr — 别名 strstr
strcmp — 二进制安全字符串比较
strcoll — 基于区域设置的字符串比较
strcspn — 获取不匹配遮罩的起始子字符串的长度
strip_tags — 从字符串中去除 HTML 和 PHP 标记
stripcslashes — 反引用一个使用 addcslashes 转义的字符串
stripos — 查找字符串首次出现的位置（不区分大小写）
stripslashes — 反引用一个引用字符串
stristr — strstr 函数的忽略大小写版本
strlen — 获取字符串长度
strnatcasecmp — 使用“自然顺序”算法比较字符串（不区分大小写）
strnatcmp — 使用自然排序算法比较字符串
strncasecmp — 二进制安全比较字符串开头的若干个字符（不区分大小写）
strncmp — 二进制安全比较字符串开头的若干个字符
strpbrk — 在字符串中查找一组字符的任何一个字符
strpos — 查找字符串首次出现的位置
strrchr — 查找指定字符在字符串中的最后一次出现
strrev — 反转字符串
strripos — 计算指定字符串在目标字符串中最后一次出现的位置（不区分大小写）
strrpos — 计算指定字符串在目标字符串中最后一次出现的位置
strspn — 计算字符串中全部字符都存在于指定字符集合中的第一段子串的长度。
strstr — 查找字符串的首次出现
strtok — 标记分割字符串
strtolower — 将字符串转化为小写
strtoupper — 将字符串转化为大写
strtr — 转换指定字符
substr_compare — 二进制安全比较字符串（从偏移位置比较指定长度）
substr_count — 计算字串出现的次数
substr_replace — 替换字符串的子串
substr — 返回字符串的子串
trim — 去除字符串首尾处的空白字符（或者其他字符）
ucfirst — 将字符串的首字母转换为大写
ucwords — 将字符串中每个单词的首字母转换为大写
vfprintf — 将格式化字符串写入流
vprintf — 输出格式化字符串
vsprintf — 返回格式化字符串
wordwrap — 打断字符串为指定数量的字串
参考：http://phpmylove.blog.51cto.com/3389265/632215
http://www.5idev.com/p-php_addslashes_stripslashes.shtml#addslashes