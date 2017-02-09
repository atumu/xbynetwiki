title: sql 

#  Chapter3 SQLite中的SQL 
Created Thursday 05 March 2015

主要介绍SQL中的Select查询语句

#  示例： 
```

create table foods(
  id integer primary key,
  type_id integer,
  name text );

select *
from foods
where name='JujyFruit'
and type_id=9;

select f.name name, types.name type
from foods f
inner join (
  select *
  from food_types
  where id=6) types
on f.type_id=types.id;

select burger
from kitchen
where patties=2
and toppings='jalopenos'
and condiment != 'mayo'
limit 1;

select id, name from foods;
insert into foods values (null, 'Whataburger');
delete from foods where id=413;

```
#  基础语法介绍 
```

SQL是一种声明式语言，由命令组成，每条命令以分好（；）结束。
常量：
字符串常量用单引号（‘asb’）
包含单引号的字符串需要连续两个单引号（'keey''s chicken'）
数字常量：-1 3.142 6.023e15
二进制常量：x'0fff'

关键字和标识符
关键字：如create insert update drop select begin等，关键字不区分大小写
字符串常量是大小写敏感的。

注释：(--单行注释) (/**/多行注释)
--This is comment
/* this is
comment */

```
#  创建数据库 
表是关系数据库中信息的标准单元，所有操作都是以表为中心。
DDL数据库定义语言
DML数据库操作语言

##  创建表 
create [temp] table table_name(column_definition [,constraints]);

###  SQLite中的5种本地类型：int real text blob null 

###  字段约束与表约束： 
```

create table contacts ( id integer primary key,
						name text not null collate nocase,
						phone text not null default 'UNKNOWN',
						email text not null default '' collate nocase,
						unique (name,phone) );


```
##  修改表 
```

alter table contacts
		add column email text not null default '' collate nocase;

.schema contacts

```
表还可以由select语句的结果创建。即由select语句的结果输出作为另外一个语句的输入。

#  数据库查询 
DML
select命令是DML核心。也是最复杂的指令

###  关系操作： 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054222.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054229.png)

` SQLite SQL不支持右外连接和全外连接 `

**注意：任何Select语句的输出都可以是另一个语句的输入，有如shell中的管道**

##  select命令与操作管道 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054254.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054259.png)
每一个关键字如：from、where、having都是一个单独的子句。
在SQLite中理解select命令的最好办法是将其当作处理关系的管道。
**where子句用于过滤行，select子句用于过滤字段列**

##  过滤 
如果select是SQL中最复杂的命令，那么where就是select中最复杂的子句。
SQLite对from子句产生的每一行应用where子句进行逻辑预测并过滤行。

###  值 

###  操作符 
算术操作符
逻辑表达式中0为false,1或其他非0为true
逻辑操作符：AND , OR, NOT, IN

###  LIKE与GLOB操作符 
匹配符：LIKE匹配符 % 或 _ , GLOB匹配符 *
```

select id, name from foods where name like 'J%';
select id, name from foods where name like '%ac%P%';
select id, name from foods
		where name like '%ac%P%' and name not like '%Sch%'
select id, name from foods where name glob 'xby*'

```
##  限定和排序 
可以用limit和offset限定结果集的大小和范围。limit指定返回记录的最大数量，offset指定**偏移**的记录数。一般与order by一起使用。
select * from food_types order by id limit 1 offset 1; **偏移1，从第2个记录开始返回一行。**
**注意：你无法保证select中返回的记录是按照某种顺序的，所以如果你需要顺序返回，请使用order by子句。而且可以配合asc默认升序，desc降序。**
```

select * from foods where name like 'B%'
		order by type_id **desc,** name limit 10;
 </code>
**注意：limit offset并不是标准ANSI SQL，但大部分数据库都支持同等功能。offset必须与limit一起使用，不能单独使用。**

##  函数（Function）和聚合(Aggregate) 
SQLite提供多种函数和聚合，**可以用在不同的子句中**。函数的种类包括数学函数如abs()、字符串函数如upper(),lower()
<code sql>
select id, upper(name), length(name) from foods
		where type_id=1 limit 10;
select id, upper(name), length(name) from foods
		where length(name) < 5 limit 5;
聚合是一类特殊的函数。它从一组记录中计算聚合值。如sum().avg(),count(),min(),max()
select count(*) from foods where type_id=1;
select avg(length(name)) from foods;

```
##  分组（Grouping） 
聚合的主要部分就是分组。聚合可以对多个分组分别计算聚合值。配合group by一起使用。
select type_id from foods group by type_id;
group by接收where的输出并将其分割成共享某个字段（或多个字段）上同等值的小组。这些组再传递给select子句。
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054356.png)


**使用group by时，select子句对每组单独应用聚合，而不是对整个结果应用聚合。因此聚合对每组生成一个值，并将这些组的行作为单行。**
select type_id, count(*) from foods group by type_id;
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054407.png)

group by 使用类似的值创建分组，但没有在select子句处理前过滤这些组。having具备这一功能。having可以从group by中过滤分组。形成对group by的约束。
having是一个可以应用到group by的断言，这些断言是针对聚合值的。
select type_id, count(*) from foods
		group by type_id having count(*) < 20;
goup by接收where子句的约束，将结果行分为共享某值的组，having对每组应用过滤，通过过滤的组传递给select子句来做聚合和映射。

##  去掉重复值 
distinct处理select结果并过滤重复的行。
select distinct type_id from foods;

##  多表连接 
连接（join）是关系数据工作的关键。连接操作的结果作为select子句的输入，供其余部分（过滤）处理。
外键与外键关系。
```

select foods.name, food_types.name
		from foods, food_types
		where foods.type_id=food_types.id limit 10;
 </code>
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054434.png)
通过连接表改变了输入。

###  内连接 
内连接就是通过表中的连个字段进行连接。这是最常见的连接类型。内连接使用关系代数中的交叉集合操作。
内连接只返回满足给定字段关系的行，称为**连接条件。内连接需要连接条件。**
Select *
From foods **inner join** food_types on foods.id = food_types.id;
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054443.png)

###  交叉连接 
交叉连接没有连接条件。行之间只是简单的组合在一起。（笛卡尔积）。它是强制的几乎无意义的连接。
select * from foods, food_types;

###  外连接 
内连接根据给定关系选择表中的行，外连接选择内连接的所有行外加一些关系之外的行。三种外连接分为左外，右外，全外连接
select *
from foods left outer join foods_episodes on foods.id=foods_episodes.food_id;
左外连接Left outer join左边的表foods会被全部包含，右边的表foods_episodes只会包含匹配的行，没有相应行时以null填充字段。
**注意：SQLite不支持右外连接和全外连接**

##  名称和别名 
在from子句中对表重命名。然后在其他子句中直接引用。
<code sql>
select foods.name, food_types.name
from foods, food_types
where foods.type_id = food_types.id
limit 10;

select f.name, t.name
from foods f, food_types t
where f.type_id = t.id
limit 10;

elect f.name as food, e1.name, e1.season, e2.name, e2.season
from episodes e1, foods_episodes fe1, foods f,
	 episodes e2, foods_episodes fe2
where
  -- Get foods in season 4
  (e1.id = fe1.episode_id and e1.season = 4) and fe1.food_id = f.id
  -- Link foods with all other epsisodes
  and (fe1.food_id = fe2.food_id)
  -- Link with their respective episodes and filter out e1's season
  and (fe2.episode_id = e2.id AND e2.season != e1.season)
order by f.name;   

```
##  子查询 
子查询最常用的地方是where子句，特别是在in操作符中。
```

select count(*)
from foods
where type_id **in**
 (select id
  from food_types
  where name='Bakery' or name='Cereal');
select子句中的子查询可以用来从其他表向结果集中添加额外数据。
select name,(select count(id) from foods_episodes where food_id=f.id) count from foods f order by count desc limit 10;

from子句中的子查询作为一个关系源常常被称为内联视图或派生表。
select f.name, types.name from foods f
inner join (select * from food_types where id=6) types
on f.type_id=types.id;

```
##  复合查询 
使用三个特殊的关系操作符（联合，交叉，差集）处理多个查询的结果。union, intersect, except。
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054528.png)
```

select f.*, top_foods.count from foods f
inner join
  (select food_id, count(food_id) as count from foods_episodes
	 group by food_id
	 order by count(food_id) desc limit 1) top_foods
  on f.id=top_foods.food_id
union
select f.*, bottom_foods.count from foods f
inner join
  (select food_id, count(food_id) as count from foods_episodes
	 group by food_id
	 order by count(food_id) limit 1) bottom_foods
  on f.id=bottom_foods.food_id
order by top_foods.count desc;

select f.* from foods f
inner join
  (select food_id, count(food_id) as count
	 from foods_episodes
	 group by food_id
	 order by count(food_id) desc limit 10) top_foods
  on f.id=top_foods.food_id
intersect
select f.* from foods f
  inner join foods_episodes fe on f.id = fe.food_id
  inner join episodes e on fe.episode_id = e.id
  where e.season between 3 and 5
order by f.name;

select f.* from foods f
inner join
  (select food_id, count(food_id) as count from foods_episodes
	 group by food_id
	 order by count(food_id) desc limit 10) top_foods
  on f.id=top_foods.food_id
except
select f.* from foods f
  inner join foods_episodes fe on f.id = fe.food_id
  inner join episodes e on fe.episode_id = e.id
  where e.season between 3 and 5
order by f.name;

```
##  条件结果 
case表达式允许在select语句中处理各种情况，它有两种形式。第一种，接收静态值并列出各种情况下的case值。
```

case value
	when x then value_1
	when y then value_2
end
简单示例：
select name **|| case type_id**
**                 when 7  then ' is a drink'**
**                 when 8  then ' is a fruit'**
**                 when 9  then ' is junkfood'**
**                 when 13 then ' is seafood'**
**                 else null**
**               end** description
from foods
where **description is not null**
order by name
limit 10;
注意："||"为SQL字符串连接符。

第二种形式的case允许when中有表达式：此处的case包含在子查询中
select name, **(select**
			**case**
	**                 when count(*) > 4 then 'Very High'**
	**                 when count(*) = 4 then 'High'**
	**                 when count(*) in (2,3) then 'Moderate'**
	**                 else 'Low'**
**                 end**
**             from foods_episodes**
**             where food_id=f.id)** frequency
from foods f
where frequency like '%High';

```
##  处理SQLite中的null 
null不等于任何值包括null本身。（这点与java是不同的）
要判断null需要通过is null 或者is not null来判断。
注意引用可能存在null值字段的查询。
