title: python学习笔记219 

#  Python学习笔记之对日期格式化 
http://www.cnblogs.com/65702708/archive/2011/04/17/2018936.html
Python格式化日期时间的函数为**datetime.datetime.strftime()**；由字符串转为日期型的函数为：**datetime.datetime.strptime()**，两个函数都涉及日期时间的格式化字符串，列举如下：
%a Abbreviated(缩写的) weekday name 
%A Full weekday name 
%b Abbreviated month name 
%B Full month name 
%c Date and time representation appropriate for locale 
**%d** Day of month as decimal number (01 - 31) 
**%H** Hour in 24-hour format (00 - 23) 
%I Hour in 12-hour format (01 - 12) 
%j Day of year as decimal number (001 - 366) 
**%m** Month as decimal number (01 - 12) 
**%M** Minute as decimal number (00 - 59) 
%p Current locale's A.M./P.M. indicator for 12-hour clock 
**%S** Second as decimal number (00 - 59) 
%U Week of year as decimal number, with Sunday as first day of week (00 - 51) 
**%w** Weekday as decimal number (0 - 6; Sunday is 0) 
%W Week of year as decimal number, with Monday as first day of week (00 - 51) 
%x Date representation for current locale 
%X Time representation for current locale 
%y Year without century, as decimal number (00 - 99) 
**%Y** Year with century, as decimal number 
%z, %Z Time-zone name or abbreviation; no characters if time zone is unknown 
% % Percent sign

举一个例子：
ebay中时间格式为‘Sep-21-09 16:34’
则通过下面代码将这个字符串转换成datetime
```

>>> c = datetime.datetime.strptime('Sep-21-09 16:34','%b-%d-%y %H:%M');
>>> c
datetime.datetime(2009, 9, 21, 16, 34)

```
又如：datetime转换成字符串
```

>>> datetime.datetime.now().strftime('%b-%d-%y %H:%M:%S');
'Sep-22-09 16:48:08'

```

对于我们常用的则应该是这样 **%Y-%m-%d %H-%M-%S**

##  timedelta时间间隔 
datetime.timedelta对象代表两个时间之间的的时间差，两个date或datetime对象相减时可以返回一个timedelta对象。
构造函数：
class datetime.timedelta([days[, seconds[, microseconds[, milliseconds[, minutes[, hours[, weeks]]]]]]])
所有参数可选，且默认都是0，参数的值可以是整数，浮点数，正数或负数。
内部只存储**days，seconds，microseconds**，其他参数的值会自动按如下规则抓转换：
  * 1 millisecond（毫秒） 转换成 1000 microseconds（微秒）
  * 1 minute 转换成 60 seconds
  * 1 hour 转换成 3600 seconds
  * 1 week转换成 7 days
支持的操作有：+，-，*,取反-
此外，timedelta和可以和date，datetime对象进行加减操作，如：
```

>>> datetime.datetime.now()  
datetime.datetime(2013, 5, 23, 10, 49, 27, 182057)  
>>> datetime.datetime.now()+datetime.timedelta(2)  
datetime.datetime(2013, 5, 25, 10, 49, 29, 385559)

```
100天前是几号?
```

import datetime
(datetime.datetime.now() - datetime.timedelta(days = 100)).strftime("%Y-%m-%d")

```
计算昨天时间：
```

from datetime import datetime
from datetime import timedelta
now = datetime.now()
aDay = timedelta(days=-1)
now = now + aDay
print now.strftime('%Y-%m-%d')

```
timedelta.total_seconds()
