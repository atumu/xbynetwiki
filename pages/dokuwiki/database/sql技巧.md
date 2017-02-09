title: sql技巧 

#  sql技巧 
问题一：sql查询分数都大于60的名字:
有这样的表，查询已有分数都大于60分的人名
姓名 科目 成绩
张 语文 69
张 数学 60
张 英语 100
王 语文 40
王 数学 60
返回名字就可以了
` select name from student  GROUP BY name having min(fenshu)>60 `

##  SQL中使用ESCAPE定义转义符 
在使用LIKE关键字进行模糊查询时，“%”、“_”和“[]”单独出现时，会被认为是通配符。为了在字符数据类型的列中查询是否存在百分号（%）、下划线（_）或者方括号（[]）字符，就需要有一种方法告诉DBMS，将LIKE判式中的这些字符看作是实际值，而不是通配符。关键字ESCAPE允许确定一个转义字符，告诉DBMS紧跟在转义字符之后的字符看作是实际值。如下面的表达式：
```

LIKE '%M%' ESCAPE ‘M’

``` 
使用` ESCAPE关键字 `定义了转义字符“M”，告诉DBMS将搜索字符串“%M%”中的第二个百分符（%）作为实际值，而不是通配符。当然，第一个百分符（%）仍然被看作是通配符，因此满足该查询条件的字符串为所有以％结尾的字符串。
类似地，下面的表达式：
LIKE  'AB&_%'   ESCAPE  ‘&’
此时，定义了转义字符“&”，搜索字符串中紧跟“&”之后的字符，即“_”看作是实际字符值，而不是通配符。而表达式中的“%”，仍然作为通配符进行处理。该表达式的查询条件为以“AB_”开始的所有字符串。

MySQL中还可使用` \\ `进行转义
```

SELECT   des,COUNT(id) FROM pmp_demand_info WHERE des LIKE'%\\%%'
str=str.replaceAll("%", "\\\\\\\\%");str=haha%输出str=haha\\%

```
参考：http://www.2cto.com/database/201212/172678.html
http://blog.csdn.net/linan0930/article/details/36188679