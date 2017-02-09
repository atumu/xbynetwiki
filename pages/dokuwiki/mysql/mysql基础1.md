title: mysql基础1 

#  mysql基础语法 
##  运算符 
http://www.cnblogs.com/nerxious/archive/2012/12/30/2839707.html
![](/data/dokuwiki/mysql/pasted/20160409-121956.png)
![](/data/dokuwiki/mysql/pasted/20160409-122312.png)
![](/data/dokuwiki/mysql/pasted/20160409-122332.png)

##  表达式 
2. 表达式
根据连接表达式的运算符进行分类，可以将表达式分为算术表达式、比较表达式、逻辑表达式、按位表达式和混合表达式等；根据表达式的作用进行分类，可以将表达式分为字段名表达式、目标表达式和条件表达式。
2.1> 字段名表达式
字段名表达式可以是单一的字段名或几个字段的组合，还可以是由字段、作用于字段的集合函数和常量的任意算术组成的运算表达式。主要包括数值表达式、字符表达式、逻辑表达式和日期表达式。

2.2> 目标表达式
　　目标表达式有4中构成方式：
　　（1）*：表示选择相应基表和视图的所有字段。
　　（2）<表名>.：表示选择指定的基表和视图的所有字段。
　　（3）集函数()：表示在相应的表中按集函数操作和运算。
　　（4）[<表名>.]字段名表达式[,[<表名>.]<字段名表达式>]...：表示按字段名表达式在多个指定的表中选择。

**2.3> 条件表达式**
　　常用的条件表达式有以下6种：
　　（1）比较大小——应用比较运算符构成表达式。
　　（2）指定范围——（NOT）BETWEEN...AND...运算符查找字段值在或者不在指定范围内的记录。BETWEEN后面指定范围的最小值，AND指定范围的最大值。
　　（3）集合（NOT）IN——查询字段值属于或不属于指定集合内的记录。
　　（4）字符匹配——（NOT）LIKE查找字段值满足匹配字符串中指定的匹配条件的记录。匹配字符串可以是一个完整的字符串，也可以包含通配符“_”和“%”，“_”表示任意单个字符，"%"表示任意长度的字符串。
　　（5）空值IS（NOT） NULL——查找字段值（不）为空的记录。
　　（6）多重条件AND和OR。AND表达式用来查找字段值同时满足AND相连接的查询条件的记录。OR表达式用来查询字段值满足OR连接的查询条件中的任意一个的记录。AND运算符的优先级高于OR运算符。

##  SQL 函数 
http://www.yiibai.com/sql/sql-useful-functions.html
http://www.cnblogs.com/moss_tan_jun/archive/2010/08/23/1806861.html
SQL COUNT()函数 - SQL COUNT聚合函数是用来统计在数据库表中的行数。
SQL MAX()函数 - SQL MAX聚合函数允许我们选择某个/些列最高(最大)值。
SQL MIN()函数 - SQL MIN聚合函数允许我们选择某些列最低(最小)值。
SQL AVG()函数 - SQL AVG聚合函数选择某些表列的平均值。
SQL SUM()函数 - SQL SUM聚合函数允许选择列的总和。
SQL SQRT()函数 - 这是用来生成给定数的平方根。
SQL RAND()函数 - 这用于生成使用SQL命令的随机数。
SQL CONCAT()函数 - 这是用来连接内部SQL命令的任何字符串。
SQL 数值函数(有很多) - SQL操作数字需要SQL函数的完整列表。http://www.yiibai.com/sql/sql-numeric-functions.html
SQL 字符串函数(有很多) - SQL操作字符串需要SQL函数的完整列表。http://www.yiibai.com/sql/sql-string-functions.html
###  字符串函数 
http://www.yiibai.com/sql/sql-string-functions.html

##  sql group by 与 having的用法 
http://www.cnblogs.com/wang-123/archive/2012/01/05/2312676.html
**1. GROUP BY 是分组查询,** 一般 GROUP BY 是和聚合函数配合使用
` group by 有一个原则,就是 select 后面的所有列中,没有使用聚合函数的列,必须出现在 group by 后面（重要） `
例如,有如下数据库表：
A    B 
1    abc 
1    bcd 
1    asdfg
如果有如下查询语句（**该语句是错误的**，原因见前面的原则）
select A,B from table group by A  
右边3条如何变成一条,所以需要用到聚合函数，如下(下面是正确的写法):
select A,count(B) as 数量 from table group by A 
这样的结果就是 
A    数量 
1    3  

**2. Having**
  * **where 子句的作用是**在对查询结果进行分组前，将不符合where条件的行去掉，**即在分组之前过滤数据**，**条件中不能包含聚组函数**，使用where条件显示特定的行。
  * **having 子句的作用是筛选满足条件的组，即在分组之后过滤数据，条件中经常包含聚组函数**，使用having 条件显示特定的组，也可以使用多个分组标准进行分组。
注意:group by 是先排序后分组；
having 子句被限制子已经在SELECT语句中定义的列和聚合表达式上。通常，你需要通过在HAVING子句中重复聚合函数表达式来引用聚合值，就如你在SELECT语句中做的那样。例如：
SELECT A ,COUNT(B) FROM TABLE GROUP BY A HAVING COUNT(B)>2

问题一：sql查询分数都大于60的名字:
有这样的表，查询所有有分数都大于60分的人名
姓名 科目 成绩
张 语文 69
张 数学 60
张 英语 100
王 语文 40
王 数学 60
返回名字就可以了
select name from student GROUP BY name having min(score)>60

##  sql join用法 
http://coolshell.cn/articles/3463.html
假设我们有两张表。
Table A 是左边的表。Table B 是右边的表。
其各有四条记录，其中有两条记录是相同的，如下所示：
![](/data/dokuwiki/mysql/pasted/20160409-141507.png)
下面让我们来**看看不同的Join会产生什么样的结果**。

###  1、INNER JOIN 
```

SELECT * FROM TableA INNER JOIN TableB ON TableA.name = TableB.name
id  name       id   name
--  ----       --   ----
1   Pirate     2    Pirate
3   Ninja      4    Ninja

```
![](/data/dokuwiki/mysql/pasted/20160409-141158.png)
Inner join 产生的结果集中，是A和B的**交集**。

###  2、Full outer join 
```

SELECT * FROM TableA FULL OUTER JOIN TableB ON TableA.name = TableB.name

id    name       id    name
--    ----       --    ----
1     Pirate     2     Pirate
2     Monkey     null  null
3     Ninja      4     Ninja
4     Spaghetti  null  null
null  null       1     Rutabaga
null  null       3     Darth Vader

```
![](/data/dokuwiki/mysql/pasted/20160409-141744.png)
Full outer join **产生A和B的并集。但是需要注意的是，对于没有匹配的记录，则会以null做为值。**

####  2、1 Full outer join变通之产生A表和B表都没有出现的数据集 
```

SELECT * FROM TableA FULL OUTER JOIN TableB ON TableA.name = TableB.name WHERE TableA.id IS null OR TableB.id IS null
id    name       id    name
--    ----       --    ----
2     Monkey     null  null
4     Spaghetti  null  null
null  null       1     Rutabaga
null  null       3     Darth Vader

```
![](/data/dokuwiki/mysql/pasted/20160409-142257.png)
###  3、Left outer join 
```

SELECT * FROM TableA LEFT OUTER JOIN TableB ON TableA.name = TableB.name

id  name       id    name
--  ----       --    ----
1   Pirate     2     Pirate
2   Monkey     null  null
3   Ninja      4     Ninja
4   Spaghetti  null  null

```
Left outer join **产生表A的完全集**，而B表中匹配的则有值，没有匹配的则以null值取代。
![](/data/dokuwiki/mysql/pasted/20160409-141928.png)

####  3.1、Left outer join变通之产生在A表中有而在B表中没有的集合。 
```

SELECT * FROM TableA LEFT OUTER JOIN TableB ON TableA.name = TableB.name WHERE TableB.id IS null 
id  name       id     name
--  ----       --     ----
2   Monkey     null   null
4   Spaghetti  null   null

```
![](/data/dokuwiki/mysql/pasted/20160409-142107.png)

###  4、Right outer join 
```

SELECT * FROM TableA RIGHT OUTER JOIN TableB ON TableA.name = TableB.name
id    name       id    name
--    ----       --    ----
null  null       1     Rutabaga
1     Pirate     2     Pirate
null  null       3     Darth Vader
3     Ninja      4     Ninja

```
其实也可以用Left outer join来实现。交换一下表的顺序即可。如SELECT * FROM TableB LEFT OUTER JOIN TableA ON TableA.name = TableB.name

###  5、总结一下 
![](/data/dokuwiki/mysql/pasted/20160409-142832.png)