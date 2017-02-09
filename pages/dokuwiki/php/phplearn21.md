title: phplearn21 

#  PHP学习之正则表达式 
http://php.net/manual/zh/book.pcre.php
分为两类;PCRE与POSIX处理方式。
PCRE 函数(推荐)
  * preg_filter — 执行一个正则表达式搜索和替换
  * preg_grep — 返回匹配模式的数组条目
  * preg_last_error — 返回最后一个PCRE正则执行产生的错误代码
  * preg_match_all — 执行一个全局正则表达式匹配
  * preg_match — 执行一个正则表达式匹配
  * preg_quote — 转义正则表达式字符
  * preg_replace_callback_array — Perform a regular expression search and replace using callbacks
  * preg_replace_callback — 执行一个正则表达式搜索并且使用一个回调进行替换
  * preg_replace — 执行一个正则表达式的搜索和替换
  * preg_split — 通过一个正则表达式分隔字符串
POSIX Regex 函数
  * ereg_replace — 正则表达式替换
  * ereg — 正则表达式匹配
  * eregi_replace — 不区分大小写的正则表达式替换
  * eregi — 不区分大小写的正则表达式匹配
  * split — 用正则表达式将字符串分割到数组中
  * spliti — 用正则表达式不区分大小写将字符串分割到数组中
  * sql_regcase — 产生用于不区分大小的匹配的正则表达式
##  正则表达式基础 
###  元字符 
![](/data/dokuwiki/php/pasted/20160407-124042.png)
提示
  * 当我们要匹配这些元字符的时候，我们需要用到字符转义功能，同样正则表达式里面用 \ 来表示转义，如要匹配 . 符号，则需要用 \. ，否则 . 会被解释成“除换行符外的任意字符”。当然，要匹配 \ ，则需要写成 \\
  * 连续的数字或字母可以用 – 符号连接起来，如 [a-z]匹配所有的小写字母，[1-5] 匹配 1 至 5 这 5 个数字
###  重复 
![](/data/dokuwiki/php/pasted/20160407-124258.png)
###  贪婪与懒惰 
正则表达式默认的情况下，会在满足匹配条件下尽可能的匹配更多内容。
如 a.*b，用他来匹配 aabab ，它会匹配整个 aabab ，而不会只匹配到 aab 为止，这就是贪婪匹配。
与**贪婪匹配**对应的是，在满足匹配条件的情况下尽可能的匹配更少的内容，这就是**懒惰匹配**。
上述例子对应的懒惰匹配规则为：
a.*?b如果用该表达式去匹配 aabab ，那么就会得到 aab 和 ab 这样两个匹配结果。
![](/data/dokuwiki/php/pasted/20160407-124347.png)
###  分枝 
分枝是指制定几个规则，如果满足任意一种规则，则都当作匹配成功。具体来说就是用 | 符号把各种规则分开，且条件从左至右匹配。
由于**分枝规定，只要匹配成功，就不再对后面的条件加以匹配，所以如果你想匹配有包含关系的内容，请注意规则的顺序。**
下面是一个使用分枝的例子。
美国的邮政编码的规则是 5 个数字或者 5 个数字连上 4 个数字，如 12345 或者 54321-1234 ，如果要匹配所有的邮编，则正确的正则表达式为：
```

\d{5}-\d{4}|\d{5}
//错误写法
\d{5}|\d{5}-\d{4}

```
下面的错误写法，只能匹配到 5 位数字及 9 位数字的前 5 位数字的情况，而不能匹配 9 位数字的邮编。

###  分组 
在正则表达式中，可以用小括号将一些规则括起来当作分组，分组可以作为一个元字符来看待。
分组的例子(\d{1,3}\.){3}\d{1,3}
如果要完全匹配正确的 IP 地址，则需要改进如下：
```

((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)

```
规则说明
该规则关键之处在于确定 IP 地址每一段范围为 0-255 ，然后再重复 4 次即可。在：
25[0-5]|2[0-4]\d|[01]?\d\d?中，用分枝首先确定了 250-255 和 200-249 。 [01]?\d\d? 则确定了 0-199 的范围，综合起来就是 0-255 。

###  模式修正符 
模式修正符是标记在整个正则表达式之外的，可以看着是对正则表达式的一些补充说明。
常用的模式修正符如下：
![](/data/dokuwiki/php/pasted/20160407-125235.png)

##  php与正则表达式 
在 PHP 应用中，正则表达式主要用于：
  * 正则匹配：根据正则表达式匹配相应的内容
  * 正则替换：根据正则表达式匹配内容并替换
  * 正则分割：根据正则表达式分割字符串
在 PHP 中有两类正则表达式函数，一类是 Perl 兼容正则表达式函数，一类是 POSIX 扩展正则表达式函数。二者差别不大，而且推荐使用Perl 兼容正则表达式函数，因此下文都是以 Perl 兼容正则表达式函数为例子说明。
**定界符**
Perl 兼容模式的正则表达式函数，其正则表达式需要写在定界符中。任何不是字母、数字或反斜线（）的字符都可以作为定界符，**通常我们使用 / 作为定界符**。具体使用见下面的例子。
尽管正则表达式功能非常强大，但如果用普通字符串处理函数能完成的，就尽量不要用正则表达式函数，因为正则表达式效率会低得多。
###  preg_match() 
` preg_match() ` 函数用于进行正则表达式匹配，成功返回 1 ，否则返回 0 。
int preg_match( string pattern, string subject [, array matches ] )
参数	说明
  * pattern	正则表达式
  * subject	需要匹配检索的对象
  * matches	可选，存储匹配结果的数组， $matches[0] 将包含与整个模式匹配的文本，$matches[1] 将包含与第一个捕获的括号中的子模式所匹配的文本，以此类推
```

<?php
if(preg_match("/php/i", "PHP is the web scripting language of choice.", $matches)){
    print "A match was found:". $matches[0];
} else {
    print "A match was not found.";
}
?>

```
**preg_match() 第一次匹配成功后就会停止匹配，如果要实现全部结果的匹配，即搜索到subject结尾处，则需使用 preg_match_all() 函数。**
例子 2 ，从一个 URL 中取得主机域名 ：
```

<?php
// 从 URL 中取得主机名
preg_match("/^(http:\/\/)?([^\/]+)/i","http://www.5idev.com/index.html", $matches);
$host = $matches[2];
// 从主机名中取得后面两段
preg_match("/[^\.\/]+\.[^\.\/]+$/", $host, $matches);
echo "域名为：{$matches[0]}";
?>

```
浏览器输出：
域名为：5idev.com
###  preg_match_all() 
` preg_match_all() 函数 `用于进行正则表达式**全局匹配**，成功**返回整个模式匹配的次数**（可能为零），如果出错返回 FALSE 。
int preg_match_all( string pattern, string subject, array matches [, int flags ] ) 
![](/data/dokuwiki/php/pasted/20160407-130008.png)
下面的例子演示了将文本中所有 <pre></pre> 标签内的关键字（php）显示为红色。
```

<?php
$str = "<pre>学习php是一件快乐的事。</pre><pre>所有的phper需要共同努力！</pre>";
$kw = "php";
preg_match_all('/<pre>([\s\S]*?)<\/pre>/',$str,$mat);
for($i=0;$i<count($mat[0]);$i++){
    $mat[0][$i] = $mat[1][$i];
    $mat[0][$i] = str_replace($kw, '<span style="color:#ff0000">'.$kw.'</span>', $mat[0][$i]);
    $str = str_replace($mat[1][$i], $mat[0][$i], $str);
}
echo $str;
?>

```
###  正则匹配中文汉字 
正则匹配中文汉字根据页面编码不同而略有区别：
  * GBK/GB2312编码：[x80-xff]+ 或 [xa1-xff]+
  * UTF-8编码：[x{4e00}-x{9fa5}]+/u
例子：
```

<?php
$str = "学习php是一件快乐的事。";
preg_match_all("/[x80-xff]+/", $str, $match);
//UTF-8 使用：
//preg_match_all("/[x{4e00}-x{9fa5}]+/u", $str, $match);
print_r($match);
?>
输出：
Array
(
    [0] => Array
        (
            [0] => 学习
            [1] => 是一件快乐的事。
        )
 
)

```
##  正则替换preg_replace() 
` preg_replace() 函数 `用于正则表达式的搜索和替换。
语法：
mixed preg_replace( mixed pattern, mixed replacement, mixed subject [, int limit ] ) 
参数说明：
  * pattern	正则表达式
  * replacement	替换的内容
  * subject	需要匹配替换的对象
  * limit	可选，指定替换的个数，如果省略 limit 或者其值为 -1，则所有的匹配项都会被替换
上述参数除 limit 外都可以是一个数组。如果 pattern 和 replacement 都是数组，将以其键名在数组中出现的顺序来进行处理，这不一定和索引的数字顺序相同。如果使用索引来标识哪个 pattern 将被哪个 replacement 来替换，应该在调用 preg_replace() 之前用 ksort() 函数对数组进行排序。
例子 1 ：
```

<?php
$str = "The quick brown fox jumped over the lazy dog.";
$str = preg_replace('/\s/','-',$str);
echo $str;
?>

```
输出结果为：
The-quick-brown-fox-jumped-over-the-lazy-dog.
**例子 2 ，使用数组：**
```

<?php
$str = "The quick brown fox jumped over the lazy dog.";

$patterns[0] = "/quick/";
$patterns[1] = "/brown/";
$patterns[2] = "/fox/";

$replacements[2] = "bear";
$replacements[1] = "black";
$replacements[0] = "slow";

print preg_replace($patterns, $replacements, $str);
/*输出：
The bear black slow jumped over the lazy dog.
*/
ksort($replacements);
print preg_replace($patterns, $replacements, $str);
/*输出：
The slow black bear jumped over the lazy dog.
*/
?>

```
##  正则表达式分割 preg_split 与 split 函数 
` preg_ split() 函数 `用于正则表达式分割字符串。(字符串函数对应为explode())
语法：
array preg_split( string pattern, string subject [, int limit [, int flags]] ) 
**返回一个数组**，包含 subject 中沿着与 pattern 匹配的边界所分割的子串。
![](/data/dokuwiki/php/pasted/20160407-132711.png)
例子 1 ：
```

<?php
$str = "php mysql,apache ajax";
$keywords = preg_split("/[\s,]+/", $str);
print_r($keywords);
?>
输出结果为：
Array
(
    [0] => php
    [1] => mysql
    [2] => apache
    [3] => ajax
)

```
split() 函数同 preg_split() 类似，用正则表达式将字符串分割到数组中，返回一个数组，但推荐使用 preg_split() 。
array split( string pattern, string string [, int limit] )

提示
如果不需要正则表达式的功能，可以选择使用更快（也更简单）的替代函数如`  explode() 或 str_split() ` 。
split() 函数对大小写敏感，如果在匹配字母字符时忽略大小写的区别，请使用用法相同的 spliti() 函数


##  常用正则表达式整理 
表单验证匹配
  * 验证账号，字母开头，允许 5-16 字节，允许字母数字下划线：^[a-zA-Z][a-zA-Z0-9_]{4,15}$
  * 验证账号，不能为空，不能有空格，只能是英文字母：^\S+[a-z A-Z]$
  * 验证账号，不能有空格，不能非数字：^\d+$
  * 验证用户密码，以字母开头，长度在 6-18 之间：^[a-zA-Z]\w{5,17}$
  * 验证是否含有 ^%&',;=?$\ 等字符：[^%&',;=?$\x22]+
  * 匹配Email地址：\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*
  * 匹配腾讯QQ号：[1-9][0-9]{4,}
  * 匹配日期，只能是 2004-10-22 格式：^\d{4}\-\d{1,2}-\d{1,2}$
  * 匹配国内电话号码：^\d{3}-\d{8}|\d{4}-\d{7,8}$
  * 评注：匹配形式如 010-12345678 或 0571-12345678 或 0831-1234567
  * 匹配中国邮政编码：^[1-9]\d{5}(?!\d)$
  * 匹配身份证：\d{14}(\d{4}|(\d{3}[xX])|\d{1})
  * 评注：中国的身份证为 15 位或 18 位
  * 不能为空且二十字节以上：^[\s|\S]{20,}$

字符匹配

  * 匹配由 26 个英文字母组成的字符串：^[A-Za-z]+$
  * 匹配由 26 个大写英文字母组成的字符串：^[A-Z]+$
  * 匹配由 26 个小写英文字母组成的字符串：^[a-z]+$
  * 匹配由数字和 26 个英文字母组成的字符串：^[A-Za-z0-9]+$
  * 匹配由数字、26个英文字母或者下划线组成的字符串：^\w+$
  * 匹配空行：\n[\s| ]*\r
  * 匹配任何内容：[\s\S]*
  * 匹配中文字符：[\x80-\xff]+ 或者 [\xa1-\xff]+
  * 只能输入汉字：^[\x80-\xff],{0,}$
  * 匹配双字节字符(包括汉字在内)：[^\x00-\xff]
  * 匹配数字
  * 只能输入数字：^[0-9]*$
  * 只能输入n位的数字：^\d{n}$
  * 只能输入至少n位数字：^\d{n,}$
  * 只能输入m-n位的数字：^\d{m,n}$
  * 匹配正整数：^[1-9]\d*$
  * 匹配负整数：^-[1-9]\d*$
  * 匹配整数：^-?[1-9]\d*$
  * 匹配非负整数（正整数 + 0）：^[1-9]\d*|0$
  * 匹配非正整数（负整数 + 0）：^-[1-9]\d*|0$
  * 匹配正浮点数：^[1-9]\d*\.\d*|0\.\d*[1-9]\d*$
  * 匹配负浮点数：^-([1-9]\d*\.\d*|0\.\d*[1-9]\d*)$
  * 匹配浮点数：^-?([1-9]\d*\.\d*|0\.\d*[1-9]\d*|0?\.0+|0)$
  * 匹配非负浮点数（正浮点数 + 0）：^[1-9]\d*\.\d*|0\.\d*[1-9]\d*|0?\.0+|0$
  * 匹配非正浮点数（负浮点数 + 0）：^(-([1-9]\d*\.\d*|0\.\d*[1-9]\d*))|0?\.0+|0$

其他
```

匹配HTML标记的正则表达式（无法匹配嵌套标签）：<(\S*?)[^>]*>.*?</\1>|<.*? />
匹配网址 URL ：[a-zA-z]+://[^\s]*
匹配 IP 地址：((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)
匹配完整域名：[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+\.?

```
提示
上述正则表达式通常都加了 ^ 与 $ 来限定字符的起始和结束，如果需要匹配的内容包括在字符串当中，可能需要考虑去掉 ^ 和 $ 限定符。
以上正则表达式仅供参考，使用时请检验后再使用
参考http://www.5idev.com/p-php_regular_syntax_1.shtml