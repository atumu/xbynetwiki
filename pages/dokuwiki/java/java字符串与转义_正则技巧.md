title: java字符串与转义_正则技巧 

#  java字符串与转义、正则技巧 
##  替换字符串中包含\ 
```

title=title.replaceAll("%", "\\\\%"); //title=haha%,输出haha\%
title=title.replaceAll("%", "\\\\\\\\%"); //title=haha%,输出haha\\%

```