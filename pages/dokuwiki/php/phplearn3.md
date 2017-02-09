title: phplearn3 

#  PHP学习之日期和时间 
##  time() 
PHP提供了内置函数 time() 来取得服务器当前时间的时间戳。
```

<?php
echo time();
?>

```
上面的例子运行后得到的是一串类似这样的数字：1279115455
我们可以通过 date() 等函数将它格式化为我们需要的时间日期格式。

还有一个微秒函数microtime()
###  计算页面执行时间 
```

<?php
//do something
sleep(3);
//do something
$running_time = time() - $_SERVER['REQUEST_TIME'];
echo '页面运行时间：',$running_time,' 秒';
?>

```
##  date() 
http://php.net/manual/zh/function.date.php
**PHP Date() 函数**把时间戳格式化为更易读的日期和时间。
语法
date(format,timestamp)
参数	描述
  * format	必需。规定时间戳的格式。
  * timestamp	可选。规定时间戳。默认是当前时间和日期。
注释：时间戳是一种字符序列，它表示具体事件发生的日期和事件。
![](/data/dokuwiki/php/pasted/20160407-144754.png)
获得简单的日期
date() 函数的格式参数是必需的，它们规定如何格式化日期或时间。
下面列出了一些**常用于日期的字符**：
d - 表示月里的某天（01-31）
m - 表示月（01-12）
Y - 表示年（四位数）
l - 表示周里的某天
下面是**常用于时间的字符**：
h - 带有首位零的 12 小时小时格式
i - 带有首位零的分钟
s - 带有首位零的秒（00 -59）
a - 小写的午前和午后（am 或 pm）
其他字符，比如 "/", "." 或 "-" 也可被插入字符中，以增加其他格式。
实例
```

date("Y-m-d",time());		//显示格式如 2008-12-01
date("Y.m.d",time());		//显示格式如 2008.12.01
date("M d Y",time());		//显示格式如 Dec 01 2008
date("Y-m-d H:i",time());	//显示格式如 2008-12-01 12:01

```
PHP 提示 - **自动版权年份**
使用 date() 函数在您的网站上自动更新版本年份：
```

© 2010-<?php echo date("Y")?>

```
如果您输出的时间和实际时间差8个小时（假设您采用的北京时区）的话，请检查**php.ini**文件，做如下设置：
**date.timezone = Asia/Shanghai**
如需做其他时区的设置请参考：http://www.php.net/manual/en/timezones.php

##  getdate() 
array getdate ([ int $timestamp = time() ] )
返回一个根据 timestamp 得出的包含有日期信息的**关联数组** array。如果没有给出时间戳则认为是当前本地时间。
```

<?php
$today = getdate();
print_r($today);
?>
以上例程的输出类似于：
Array
(
    [seconds] => 40
    [minutes] => 58
    [hours]   => 21
    [mday]    => 17
    [wday]    => 2
    [mon]     => 6
    [year]    => 2003
    [yday]    => 167
    [weekday] => Tuesday
    [month]   => June
    [0]       => 1055901520
)

```
###  获得时区 
setlocale — 设置地区信息
string setlocale ( int $category , string $locale [, string $... ] )
string setlocale ( int $category , array $locale )

如果从代码返回的不是正确的时间，有可能是因为您的服务器位于其他国家或者被设置为不同时区。
因此，如果您需要基于具体位置的准确时间，您可以设置要用的时区。
下面的例子把时区设置为 "Asia/Shanghai"，然后以指定格式输出当前时间：
```

<?php
date_default_timezone_set("Asia/Shanghai");
echo "当前时间是 " . date("h:i:sa");
?>

```
###  通过 PHP mktime() 创建日期 
date() 函数中可选的时间戳参数规定时间戳。如果您未规定时间戳，将使用当前日期和时间（正如上例中那样）。
mktime() 函数返回日期的 Unix 时间戳。Unix 时间戳包含 Unix 纪元（1970 年 1 月 1 日 00:00:00 GMT）与指定时间之间的秒数。
**mktime(hour,minute,second,month,day,year)** ,int mktime(时, 分, 秒, 月, 日, 年)
下面的例子使用 mktime() 函数中的一系列参数来创建日期和时间：
```

<?php
$d=mktime(9, 12, 31, 6, 10, 2015);
echo "创建日期是 " . date("Y-m-d h:i:sa", $d);
?>

```
任何给定月份的最后一天都可以被表示为下个月的第 "0" 天
```

<?php
$lastday = mktime(0, 0, 0, 3, 0, 2008);
echo strftime("2008年2月最后一天是：%d", $lastday);
?>

```
浏览器输出：
2008年2月最后一天是：29
##  strftime — 根据区域设置格式化本地时间／日期 
http://php.net/manual/zh/function.strftime.php
string strftime ( string $format [, int $timestamp = time() ] )

###  通过 PHP strtotime() 用字符串来创建日期 
PHP strtotime() 函数用于把人类可读的字符串转换为 Unix 时间。
**strtotime(time,now)**
下面的例子通过 strtotime() 函数创建日期和时间：
```

<?php
$d=strtotime("10:38pm April 15 2015");
echo "创建日期是 " . date("Y-m-d h:i:sa", $d);
echo strtotime("2009-10-21 16:00:10");	//输出 1256112010
echo strtotime("10 September 2008");	//输出 1220976000
echo strtotime("+1 day"), "<br />";	//输出明天此时的时间戳
?>

<?php
//PHP 在将字符串转换为日期这方面非常聪明，所以您能够使用各种值：
$d=strtotime("tomorrow");
echo date("Y-m-d h:i:sa", $d) . "<br>";

$d=strtotime("next Saturday");
echo date("Y-m-d h:i:sa", $d) . "<br>";

$d=strtotime("+3 Months");
echo date("Y-m-d h:i:sa", $d) . "<br>";
?>

```
