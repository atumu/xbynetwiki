title: mysql学习一 

#  Mysql学习一 
标准的SQL语句通常划分为以下类型：

查询语句：主要由于select关键字完成，查询语句是SQL语句中最复杂，功能最丰富的语句。

DML（Data Munipulation Language，数据操作语言）语句，这组DML语句修改后数据将保持较好的一致性；操作表的语句，如插入、修改、删除等；

DDL（Data Definition Language，数据定义语言）语句，操作数据对象的语言，有create、alter、drop。

DCL（Data Control Language，数据控制语言）语句，主要有grant、revoke语句。

事务控制语句：主要有commit、rollback和savepoint三个关键字完成

执行顺序如下： 
FROM
WHERE
GROUP BY
HAVING
SELECT
DISTINCT
UNION
ORDER BY

常见表对象
![](/data/dokuwiki/database/pasted/20150724-053414.png)

常见数据类型：
![](/data/dokuwiki/database/pasted/20150724-053507.png)

##  笔记部分一 
```

Ø 常用查询

MySQL结束符是“；”结束。
 
1、    显示所有数据库
show databases;
 
2、    删除数据库
drop database dbName;
 
3、    创建数据库
create database [if not exists] dbName;
中括号部分可选的，判断该数据不存在就创建
 
4、    切换、使用指定数据库
use dbName;
 
5、    显示当前使用数据库所有的表对象
show tables；
 
6、    显示表结构describe（desc）
desc tableName;
 
7、    创建一张表
create table user (
        --int 整型
        uId int,
        --小数
        uPrice decimal,
        --普通长度文本，default设置默认值
        uName varchar(255) default ‘zhangsan’,
        --超长文本
        uRemark text,
        --图片
        uPhoto blob,
        --日期
        uBirthday datetime
);
 
8、    子查询建表方法
部分列名匹配模式：
create table userInfo (
name varchar(20),
sex char
) 
as 
select name, sex from user;
上面的列名和子查询的列名以及类型要对应
 
全部列名模式：
create table userInfo
as
select * from user;
直接将整个表的类型和数据备份到新表userInfo中
 
9、    添加表字段
添加单列
alter table user add tel varchar(11) default ‘02012345678’;
 
添加多列
alter table user 
add ( 
photo blob,
birthday date
);
上面就同时增加了多列字段
 
10、    修改表字段
修改tel列
alter table user modify tel varchar(15) default ‘02087654321’;
修改tel列的位置，在第一列显示
alter table user modify tel varchar(15) default '02087654321' first;
修改tel列的位置，在指定列之后显示
alter table user modify tel varchar(15) default '02087654321' after age;
注意：alter modify不支持一次修改多个列，但是Oracle支持多列修改
但是MySQL可以通过多个modify的方式完成：
alter table user 
modify tel varchar(15) default '02087654321' first, 
modify name varchar(20) after tel;
 
11、    删除指定字段
alter table user drop photo;
 
12、    重命名表数据
表重命名
alter table user rename to users;
 
字段重命名
alter table users change name u_name varchar(10);
alter table users change sex u_sex varchar(10) after u_name;
如果需要改变列名建议使用change，如果需要改变数据类型和显示位置可以使用modify
13、 删除表

drop table users;
drop删除表会删除表结构，表对象将不存在数据中；数据也不会存在；表内的对象也不存在，如：索引、视图、约束；
 
truncate删除表
truncate都被当成DDL出来，truncate的作用就是删除该表里的全部数据，保留表结构。相当于DDL中的delete语句，
但是truncate比delete语句的速度要快得多。但是truncate不能带条件删除指定数据，只会删除所有的数据。如果删除的表有外键，
删除的速度类似于delete。但新版本的MySQL中truncate的速度比delete速度快。
Ø 约束

MySQL中约束保存在information_schema数据库的table_constraints中，可以通过该表查询约束信息；

约束主要完成对数据的检验，保证数据库数据的完整性；如果有相互依赖数据，保证该数据不被删除。
 
常用五类约束：
not null：非空约束，指定某列不为空
unique： 唯一约束，指定某列和几列组合的数据不能重复
primary key：主键约束，指定某列的数据不能重复、唯一
foreign key：外键，指定该列记录属于主表中的一条记录，参照另一条数据
check：检查，指定一个表达式，用于检验指定数据
MySQL不支持check约束，但可以使用check约束，而没有任何效果；
 
根据约束数据列限制，约束可分为：
单列约束：每个约束只约束一列
多列约束：每个约束约束多列数据
 
MySQL中约束保存在information_schema数据库的table_constraints中，可以通过该表查询约束信息；
1、    not null约束
非空约束用于确保当前列的值不为空值，非空约束只能出现在表对象的列上。
Null类型特征：
所有的类型的值都可以是null，包括int、float等数据类型
空字符串“”是不等于null，0也不等于null
create table temp(
        id int not null,
        name varchar(255) not null default ‘abc’,
        sex char null
)
上面的table加上了非空约束，也可以用alter来修改或增加非空约束
增加非空约束
alter table temp
modify sex varchar(2) not null;
 
取消非空约束
alter table temp modify sex varchar(2) null;
 
取消非空约束，增加默认值
alter table temp modify sex varchar(2) default ‘abc’ null;
 
2、    unique
唯一约束是指定table的列或列组合不能重复，保证数据的唯一性。虽然唯一约束不允许出现重复的值，但是可以为多个null
同一个表可以有多个唯一约束，多个列组合的约束。在创建唯一约束的时候，如果不给唯一约束名称，就默认和列名相同。
唯一约束不仅可以在一个表内创建，而且可以同时多表创建组合唯一约束。
MySQL会给唯一约束的列上默认创建一个唯一索引；
create table temp (
        id int not null,
        name varchar(25),
        password varchar(16),
        --使用表级约束语法，
        constraint uk_name_pwd unique(name, password)
);
表示用户名和密码组合不能重复
添加唯一约束
alter table temp add unique(name, password);
alter table temp modify name varchar(25) unique;
删除约束
alter table temp drop index name;
 
3、    primary key
主键约束相当于唯一约束+非空约束的组合，主键约束列不允许重复，也不允许出现空值；如果的多列组合的主键约束，
那么这些列都不允许为空值，并且组合的值不允许重复。
每个表最多只允许一个主键，建立主键约束可以在列级别创建，也可以在表级别上创建。MySQL的主键名总是PRIMARY，
当创建主键约束时，系统默认会在所在的列和列组合上建立对应的唯一索引。
列模式：
create table temp(
    /*主键约束*/
    id int primary key,
    name varchar(25)
);
 
create table temp2(
    id int not null,
    name varchar(25),
    pwd varchar(15),
    constraint pk_temp_id primary key(id)
);
 
组合模式：
create table temp2(
    id int not null,
    name varchar(25),
    pwd varchar(15),
    constraint pk_temp_id primary key(name, pwd)
);
 
alter删除主键约束
alter table temp drop primary key;
 
alter添加主键
alter table temp add primary key(name, pwd);
 
alter修改列为主键
alter table temp modify id int primary key;
 
设置主键自增
create table temp(
        id int auto_increment primary key,
        name varchar(20),
        pwd varchar(16)
);
auto_increment自增模式，设置自增后在插入数据的时候就不需要给该列插入值了。
 
4、    foreign key 约束
外键约束是保证一个或两个表之间的参照完整性，外键是构建于一个表的两个字段或是两个表的两个字段之间的参照关系。
也就是说从表的外键值必须在主表中能找到或者为空。
当主表的记录被从表参照时，主表的记录将不允许删除，如果要删除数据，需要先删除从表中依赖该记录的数据，
然后才可以删除主表的数据。还有一种就是级联删除子表数据。
注意：外键约束的参照列，在主表中引用的只能是主键或唯一键约束的列，假定引用的主表列不是唯一的记录，
那么从表引用的数据就不确定记录的位置。同一个表可以有多个外键约束。
创建外键约束：
主表
create table classes(
        id int auto_increment primary key,
        name varchar(20)
);
从表
create table student(
        id int auto_increment,
        name varchar(22),
        constraint pk_id primary key(id),
        classes_id int references classes(id)
);
 
通常先建主表，然后再建从表，这样从表的参照引用的表才存在。
表级别创建外键约束：
create table student(
        id int auto_increment primary key,
        name varchar(25),
        classes_id int,
        foreign key(classes_id) references classes(id)
);
上面的创建外键的方法没有指定约束名称，系统会默认给外键约束分配外键约束名称，命名为student_ibfk_n，
其中student是表名，n是当前约束从1开始的整数。
 
指定约束名称：
create table student(
        id int auto_increment primary key,
        name varchar(25),
        classes_id int,
        /*指定约束名称*/
        constraint fk_classes_id foreign key(classes_id) references classes(id)
);
 
多列外键组合，必须用表级别约束语法：
create table classes(
        id int,
        name varchar(20),
        number int,
        primary key(name, number)
);
create table student(
        id int auto_increment primary key,
        name varchar(20),
        classes_name varchar(20),
        classes_number int,
        /*表级别联合外键*/
        foreign key(classes_name, classes_number) references classes(name, number)
);
 
删除外键约束：
alter table student drop foreign key student_ibfk_1;
alter table student drop foreign key fk_student_id;
 
增加外键约束
alter table student add foreign key(classes_name, classes_number) references classes(name, number);
 
自引用、自关联（递归表、树状表）
create table tree(
        id int auto_increment primary key,
        name varchar(50),
        parent_id int,
        foreign key(parent_id) references tree(id)
);
 
级联删除：删除主表的数据时，关联的从表数据也删除，则需要在建立外键约束的后面增加on delete cascade
或on delete set null，前者是级联删除，后者是将从表的关联列的值设置为null。
create table student(
        id int auto_increment primary key,
        name varchar(20),
        classes_name varchar(20),
        classes_number int,
        /*表级别联合外键*/
        foreign key(classes_name, classes_number) references classes(name, number) on delete cascade
);
 
5、    check约束
MySQL可以使用check约束，但check约束对数据验证没有任何作用。
create table temp(
        id int auto_increment,
        name varchar(20),
        age int,
        primary key(id),
/*check约束*/
check(age > 20)
);
上面check约束要求age必须大于0，但没有任何作用。但是创建table的时候没有任何错误或警告。
 
 
Ø 索引

索引是存放在模式（schema）中的一个数据库对象，索引的作用就是提高对表的检索查询速度，
索引是通过快速访问的方法来进行快速定位数据，从而减少了对磁盘的读写操作。
索引是数据库的一个对象，它不能独立存在，必须对某个表对象进行依赖。
提示：索引保存在information_schema数据库里的STATISTICS表中。
 
创建索引方式：
自动：当表上定义主键约束、唯一、外键约束时，该表会被系统自动添加上索引。
手动：手动在相关表或列上增加索引，提高查询速度。
 
删除索引方式：
自动：当表对象被删除时，该表上的索引自动被删除
手动：手动删除指定表对象的相关列上的索引
索引类似于书籍的目录，可以快速定位到相关的数据，一个表可以有多个索引。
 
创建索引：
create index idx_temp_name on temp(name);
 
组合索引：
create index idx_temp_name$pwd on temp(name, pwd);
 
删除索引：
drop index idx_temp_name on temp;
 

Ø 视图

视图就是一个表或多个表的查询结果，它是一张虚拟的表，因为它并不能存储数据。
视图的作用、优点：
限制对数据的访问
让复杂查询变得简单
提供数据的独立性
可以完成对相同数据的不同显示
    
创建、修改视图
create or replace view view_temp
as
    select name, age from temp;
通常我们并不对视图的数据做修改操作，因为视图是一张虚拟的表，它并不存储实际数据。如果想让视图不被修改，可以用with check option来完成限制。
create or replace view view_temp
as
    select * from temp
with check option;
 
修改视图：
alter view view_temp
as
    select id, name from temp;
 
删除视图：
drop view view_temp;
 
显示创建语法：
show create view v_temp;
 

Ø DML语句

DML主要针对数据库表对象的数据而言的，一般DML完成：
插入新数据
修改已添加的数据
删除不需要的数据
1、    insert into 插入语句
insert into temp values(null, ‘jack’, 25);
主键自增可以不插入，所以用null代替
 
指定列
insert into temp(name, age) values(‘jack’, 22);
在表面后面带括号，括号中写列名，values中写指定列名的值即可。当省略列名就表示插入全部数据，
注意插入值的顺序和列的顺序需要保持一致。
Set方式插入，也可以指定列
insert into temp set id = 7, name = 'jason';
 
MySQL中外键的table的外键引用列可以插入数据可以为null，不参照主表的数据。
 
使用子查询插入数据
insert into temp(name) select name from classes;
 
多行插入
insert into temp values(null, ‘jack’, 22), (null, ‘jackson’ 23);
 
2、    update 修改语句
update主要完成对数据的修改操作，可以修改一条或多条数据。修改多条或指定条件的数据，需要用where条件来完成。
修改所有数据
update temp set name = ‘jack2’;
所有的数据的name会被修改，如果修改多列用“,”分开
update temp set name = ‘jack’, age = 22;
修改指定条件的记录需要用where
update temp set name = ‘jack’ where age > 22;
 
3、    delete 删除语句
删除table中的数据，可以删除所有，带条件可以删除指定的记录。
删除所有数据
delete from temp;
删除指定条件数据
delete from temp where age > 20;
 

Ø select 查询、function 函数

select查询语句用得最广泛、功能也最丰富。可以完成单条记录、多条记录、单表、多表、子查询等。
1、    查询某张表所有数据
select * from temp;
*代表所有列，temp代表表名，不带条件就查询所有数据
 
2、    查询指定列和条件的数据
select name, age from temp where age = 22;
查询name和age这两列，age 等于22的数据。
 
3、    对查询的数据进行运算操作
select age + 2, age / 2, age – 2, age * 2 from temp where age – 2 > 22;
 
4、    concat函数，字符串连接
select concat(name, ‘-eco’) from temp;
concat和null进行连接，会导致连接后的数据成为null
 
5、    as 对列重命名
select name as ‘名称’ from temp;
as也可以省略不写，效果一样
如果重命名的列名出现特殊字符，如“‘”单引号，那就需要用双引号引在外面
select name as “名’称” from temp;
 
6、    也可以给table去别名
select t.name Name from temp as t;
 
7、    查询常量
类似于SQL Server
select 5 + 2;
select concat('a', 'bbb');
 
8、    distinct 去掉重复数据
select distinct id from temp;
多列将是组合的重复数据
select distinct id, age from temp;
 
9、    where 条件查询
大于>、大于等于>=、小于<、小于等于<=、等于=、不等于<>
都可以出现在where语句中
select * from t where a > 2 or a >= 3 or a < 5 or a <= 6 or a = 7 or a <> 0;
 
10、    and 并且
select * from temp where age > 20 and name = ‘jack’;
查询名称等于jack并且年龄大于20的
 
11、    or 或者
满足一个即可
select * from tmep where name = ‘jack’ or name = ‘jackson’;
 
12、    between v and v2
大于等于v且小于等于v2
select * form temp where age between 20 and 25; 
 
13、    in 查询
可以多个条件 类似于or
select * from temp where id in (1, 2, 3);
查询id在括号中出现的数据
 
14、    like 模糊查询
查询name以j开头的
select * from temp where name like ‘j%’;
 
查询name包含k的
select * from temp where name like ‘%k%’;
 
escape转义
select * from temp where name like ‘\_%’ escape ‘\’;
指定\为转义字符，上面的就可以查询name中包含“_”的数据
 
15、    is null、is not null
查询为null的数据
select * from temp where name is null;
查询不为null的数据
select * from temp where name is not null;
 
16、    not
select * from temp where not (age > 20);
取小于等于20的数据
select * from temp where id not in(1, 2);
 
17、    order by
排序，有desc、asc升序、降序
select * from temp order by id;
默认desc排序
select * from temp order by id asc;
多列组合
select * from temp order by id, age;
Ø function 函数

函数的作用比较大，一般多用在select查询语句和where条件语句之后。按照函数返回的结果，
可以分为：多行函数和单行函数；所谓的单行函数就是将每条数据进行独立的计算，然后每条数据得到一条结果。
如：字符串函数；而多行函数，就是多条记录同时计算，得到最终只有一条结果记录。如：sum、avg等
多行函数也称为聚集函数、分组函数，主要用于完成一些统计功能。MySQL的单行函数有如下特征：
    单行函数的参数可以是变量、常量或数据列。单行函数可以接受多个参数，但返回一个值。
    单行函数就是它会对每一行单独起作用，每一行（可能包含多个参数）返回一个结果。
    单行函数可以改变参数的数据类型。单行函数支持嵌套使用：内层函数的返回值是外层函数的参数。
 
单行函数可以分为：
    类型转换函数；
    位函数；
    流程控制语句；
    加密解密函数；
    信息函数
单行函数

 
1、    char_length字符长度
select char_length(tel) from user;
 
2、    sin函数
select sin(age) from user;
select sin(1.57);
 
3、    添加日期函数
select date_add('2010-06-21', interval 2 month);
interval是一个关键字，2 month是2个月的意思，2是数值，month是单位
select addDate('2011-05-28', 2);
在前面的日期上加上后面的天数
 
4、    获取当前系统时间、日期
select curdate();
select curtime();
 
5、    加密函数
select md5('zhangsan');
 
6、    Null 处理函数
select ifnull(birthday, 'is null birthday') from user;
如果birthday为null，就返回后面的字符串
 
select nullif(age, 245) from user;
如果age等于245就返回null，不等就返回age
 
select isnull(birthday) from user;
判断birthday是否为null
 
select if(isnull(birthday), 'birthday is null', 'birthday not is null') from user;
如果birthday为null或是0就返回birthday is null，否则就返回birthday not is null；类似于三目运算符
 
7、    case 流程函数
case函数是一个流程控制函数，可以接受多个参数，但最终只会返回一个结果。
select name, 
age, 
(case sex
    when 1 then '男'
    when 0 then '女'
    else '火星人'
    end
) sex
from user;
 

组函数

组函数就是多行函数，组函数是完成一行或多行结果集的运算，最后返回一个结果，而不是每条记录返回一个结果。

1、    avg平均值运算
select avg(age) from user;
select avg(distinct age) from user;
 
2、    count 记录条数统计
select count(*), count(age), count(distinct age) from user;
 
3、    max 最大值
select max(age), max(distinct age) from user;
 
4、    min 最小值
select min(age), min(distinct age) from user;
 
5、    sum 求和、聚和
select sum(age), sum(distinct age) from user;
select sum(ifnull(age, 0)) from user;
 
6、    group by 分组
select count(*), sex from user group by sex;
select count(*) from user group by age;
select * from user group by sex, age;
 
7、    having进行条件过滤
不能在where子句中过滤组，where子句仅用于过滤行。过滤group by需要having
不能在where子句中用组函数，having中才能用组函数
select count(*) from user group by sex having sex <> 2;
 

Ø 多表查询和子查询

数据库的查询功能最为丰富，很多时候需要用到查询完成一些事物，而且不是单纯的对一个表进行操作。而是对多个表进行联合查询，
MySQL中多表连接查询有两种规范，较早的SQL92规范支持，如下几种表连接查询：
    等值连接
    非等值连接
    外连接
    广义笛卡尔积
SQL99规则提供了可读性更好的多表连接语法，并提供了更多类型的连接查询，SQL99支持如下几种多表连接查询：
    交叉连接
    自然连接
    使用using子句的连接
    使用on子句连接
    全部连接或者左右外连接
 
SQL92的连接查询
SQL92的连接查询语法比较简单，多将多个table放置在from关键字之后，多个table用“，”隔开；
连接的条件放在where条件之后，与查询条件直接用and逻辑运算符进行连接。如果条件中使用的是相等，
则称为等值连接，相反则称为非等值，如果没有任何条件则称为广义笛卡尔积。
广义笛卡尔积：select s.*, c.* from student s, classes c;
等值：select s.*, c.* from student s, classes c where s.cid = c.id;
非等值：select s.*, c.* from student s, classes c where s.cid <> c.id;
select s.*, c.name classes from classes c, student s where c.id = s.classes_id and s.name is not null;
 
SQL99连接查询
1、交叉连接cross join，类似于SQL92的笛卡尔积查询，无需条件。如：
select s.*, c.name from student s cross join classes c;
 
2、自然连接 natural join查询，无需条件，默认条件是将2个table中的相同字段作为连接条件，如果没有相同字段，查询的结果就是空。
select s.*, c.name from student s natural join classes c;
 
3、using子句连接查询：using的子句可以是一列或多列，显示的指定两个表中同名列作为连接条件。
如果用natural join的连接查询，会把所有的相同字段作为连接查询。而using可以指定相同列及个数。
select s.*, c.name from student s join classes c using(id);
 
4、    join … on连接查询，查询条件在on中完成，每个on语句只能指定一个条件。
select s.*, c.name from student s join classes c on s.classes_id = c.id;
 
5、    左右外连接：3种外连接，left [outer] join、right [outer] join，连接条件都是通过用on子句来指定，条件可以等值、非等值。
select s.*, c.name from student s left join classes c on s.classes_id = c.id;
select s.*, c.name from student s right join classes c on s.classes_id = c.id;
 
    子查询
    子查询就是指在查询语句中嵌套另一个查询，子查询可以支持多层嵌套。子查询可以出现在2个位置：
    from关键字之后，被当做一个表来进行查询，这种用法被称为行内视图，因为该子查询的实质就是一个临时视图
    出现在where条件之后作为过滤条件的值
 
子查询注意点：
    子查询用括号括起来，特别情况下需要起一个临时名称
    子查询当做临时表时（在from之后的子查询），可以为该子查询起别名，尤其是要作为前缀来限定数据列名时
    子查询用作过滤条件时，将子查询放在比较运算符的右边，提供可读性
    子查询作为过滤条件时，单行子查询使用单行运算符，多行子查询用多行运算符
 
将from后面的子查询当做一个table来用：
select * from (select id, name from classes) s where s.id in (1, 2);
当做条件来用：
select * from student s where s.classes_id in (select id from classes);
select * from student s where s.classes_id = any (select id from classes);
select * from student s where s.classes_id > any (select id from classes);
Ø 操作符和函数

1、    boolean只判断
select 1 is true, 0 is false, null is unknown;
select 1 is not unknown, 0 is not unknown, null is not unknown;
 
2、    coalesce函数，返回第一个非null的值
select coalesce(null, 1);
select coalesce(1, 1);
select coalesce(null, 1);
select coalesce(null, null);
 
3、    当有2个或多个参数时，返回最大的那个参数值
select greatest(2, 3);
select greatest(2, 3, 1, 9, 55, 23);
select greatest('D', 'A', 'B');
 
4、    Least函数，返回最小值，如果有null就返回null值
select least(2, 0);
select least(2, 0, null);
select least(2, 10, 22.2, 35.1, 1.1);
 
5、    控制流函数
select case 1 when 1 then 'is 1' when 2 then 'is 2' else 'none' end;
select case when 1 > 2 then 'yes' else 'no' end;
 
6、    ascii字符串函数
select ascii('A');
select ascii('1');
 
7、    二进制函数
select bin(22);
 
8、    返回二进制字符串长度
select bit_length(11);
 
9、    char将值转换成字符，小数取整四舍五入
select char(65);
select char(65.4);
select char(65.5);
select char(65.6);
select char(65, 66, 67.4, 68.5, 69.6, '55.5', '97.3');
 
10、    using改变字符集
select charset(char(0*65)), charset(char(0*65 using utf8));
 
11、    得到字符长度char_length,character_length
select char_length('abc');
select character_length('eft');
 
12、    compress压缩字符串、uncompress解压缩
select compress('abcedf');
select uncompress(compress('abcedf'));
 
13、    concat_ws分隔字符串
select concat_ws('#', 'first', 'second', 'last');
select concat_ws('#', 'first', 'second', null, 'last');
Ø 事务处理

动作
    开始事务：start transaction
    提交事务：commit
    回滚事务：rollback
    设置自动提交：set autocommit 1 | 0 
    atuoCommit系统默认是1立即提交模式；如果要手动控制事务，需要设置set autoCommit 0;
    这样我们就可以用commit、rollback来控制事务了。
 
在一段语句块中禁用autocommit 而不是set autocommit
start transaction;
select @result := avg(age) from temp;
update temp set age = @result where id = 2;
select * from temp where id = 2;//值被改变
rollback;//回滚
select * from temp where id = 2;//变回来了
在此期间只有遇到commit、rollback，start Transaction的禁用autocommit才会结束。然后就恢复到原来的autocommit模式；
 
不能回滚的语句
    有些语句不能被回滚。通常，这些语句包括数据定义语言（DDL）语句，比如创建或取消数据库的语句，
    和创建、取消或更改表或存储的子程序的语句。
    您在设计事务时，不应包含这类语句。如果您在事务的前部中发布了一个不能被回滚的语句，
    则后部的其它语句会发生错误，在这些情况下，通过发布ROLLBACK语句不能 回滚事务的全部效果。
 
一些操作也会隐式的提交事务
如alter、create、drop、rename table、lock table、set autocommit、start transaction、truncate table 等等,
在事务中出现这些语句也会提交事务的
    事务不能嵌套事务
    事务的保存点
Savepoint pointName/Rollback to savepoint pointName
一个事务可以设置多个保存点，rollback可以回滚到指定的保存点，恢复保存点后面的操作。
如果有后面的保存点和前面的同名，则删除前面的保存点。
Release savepoint会删除一个保存点，如果在一段事务中执行commit或rollback，则事务结束，所以保存点删除。
 
 Set Transaction设计数据库隔离级别
SET [GLOBAL | SESSION] TRANSACTION ISOLATION LEVEL
{ READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE }
本语句用于设置事务隔离等级，用于下一个事务，或者用于当前会话。
在默认情况下，SET TRANSACTION会为下一个事务（还未开始）设置隔离等级。
如果您使用GLOBAL关键词，则语句会设置全局性的默认事务等级，
用于从该点以后创建的所有新连接。原有的连接不受影响。使用SESSION关键测可以设置默认事务等级，
用于对当前连接执行的所有将来事务。
默认的等级是REPEATABLE READ全局隔离等级。
 

Ø 注释

select 1+1;     # 单行注释
select 1+1;     -- 单行注释
select 1 /* 多行注释 */ + 1;
Ø 基本数据类型操作

字符串
    select 'hello', '"hello"', '""hello""', 'hel''lo', '\'hello';
    select "hello", "'hello'", "` hello `", "hel""lo", "\"hello";
 
\n换行
    select 'This\nIs\nFour\nLines';
 
\转义
    select 'hello \ world!';
    select 'hello \world!';
    select 'hello \\ world!';
    select 'hello \' world!';
Ø 设置数据库mode模式

SET sql_mode='ANSI_QUOTES';
create table t(a int);
create table "tt"(a int);
create table "t""t"(a int);
craate talbe tab("a""b" int);
Ø 用户变量

set @num1 = 0, @num2 = 2, @result = 0; 
select @result := (@num1 := 5) + @num2 := 3, @num1, @num2, @result; 
Ø 存储过程

创建存储过程：
delimiter //
create procedure get(out result int)
begin
 select max(age) into result from temp;
end//
调用存储过程：
call get(@temp);
查询结果：
select @temp;
 
删除存储过程：
drop procedure get;
 
查看存储过程创建语句：
show create procedure get;
 
select…into 可以完成单行记录的赋值：
create procedure getRecord(sid int)
begin
    declare v_name varchar(20) default 'jason';
    declare v_age int;
    declare v_sex bit;
    select name, age, sex into v_name, v_age, v_sex from temp where id = sid;
    select v_name, v_age, v_sex;
end;
call getRecord(1);
Ø 函数

函数类似于存储过程，只是调用方式不同
例如：select max(age) from temp;
 
创建函数：
create function addAge(age int) returns int
     return age + 5;
 
使用函数：
select addAge(age) from temp;
 
删除函数：
drop function if exists addAge;
drop function addAge;
 
显示创建语法：
show create function addAge; 
Ø 游标

声明游标：declare cur_Name cursor for select name from temp;
打开游标：open cur_Name;
Fetch游标：fetch cur_Name into @temp;
关闭游标：close cur_Name;
 
示例：
CREATE PROCEDURE cur_show()
BEGIN
  DECLARE done INT DEFAULT 0;
  DECLARE v_id, v_age INT;
  DECLARE v_name varchar(20);
  DECLARE cur_temp CURSOR FOR SELECT id, name, age FROM temp;
  DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
 
  OPEN cur_temp;
 
  REPEAT
    FETCH cur_temp INTO v_id, v_name, v_age;
    IF NOT done THEN
       IF isnull(v_name) THEN
          update temp set name = concat('test-json', v_id) where id = v_id;
       ELSEIF isnull(v_age) THEN
          update temp set age = 22 where id = v_id;
       END IF;
    END IF;
  UNTIL done END REPEAT;
 
  CLOSE cur_temp;
END
Ø 触发器

触发器分为insert、update、delete三种触发器事件类型
还有after、before触发时间
创建触发器：
create trigger trg_temp_ins
before insert
on temp for each row
begin
insert into temp_log values(NEW.id, NEW.name);
end//
 
删除触发器：
drop trigger trg_temp_ins

```
##  笔记部分 
/* 建表规范 */ ------------------
    -- Normal Format, NF
        - 每个表保存一个实体信息
        - 每个具有一个ID字段作为主键
        - ID主键 + 原子表
    -- 1NF, 第一范式
        字段不能再分，就满足第一范式。
    -- 2NF, 第二范式
        满足第一范式的前提下，不能出现部分依赖。
        消除符合主键就可以避免部分依赖。增加单列关键字。
    -- 3NF, 第三范式
        满足第二范式的前提下，不能出现传递依赖。
        某个字段依赖于主键，而有其他字段依赖于该字段。这就是传递依赖。
        将一个实体信息的数据放在一个表内实现。

###  /* 启动MySQL与修改密码 */ 

```


net start mysql

/* 连接与断开服务器 */
mysql -h 地址 -P 端口 -u 用户名 -p 密码

/* 跳过权限验证登录MySQL */
mysqld --skip-grant-tables
-- 修改root密码
密码加密函数password()
update mysql.user set password=password('root');

SHOW PROCESSLIST -- 显示哪些线程正在运行
SHOW VARIABLES -- 

```
###  /* 数据库操作 */DDL 

```

/* 数据库操作 */ ------------------
-- 查看当前数据库
    select database();
-- 显示当前时间、用户名、数据库版本
    select now(), user(), version();
-- 创建库
    create database[ if not exists] 数据库名 数据库选项
    数据库选项：
        CHARACTER SET charset_name
        COLLATE collation_name
-- 查看已有库
    show databases[ like 'pattern']
-- 查看当前库信息
    show create database 数据库名
-- 修改库的选项信息
    alter database 库名 选项信息
-- 删除库
    drop database[ if exists] 数据库名
        同时删除该数据库相关的目录及其目录内容

/* 表的操作 */ ------------------
-- 创建表
    create [temporary] table[ if not exists] [库名.]表名 ( 表的结构定义 )[ 表选项]
        每个字段必须有数据类型
        最后一个字段后不能有逗号
        temporary 临时表，会话结束时表自动消失
        对于字段的定义：
            字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']
-- 表选项
    -- 字符集
        CHARSET = charset_name
        如果表没有设定，则使用数据库字符集
    -- 存储引擎
        ENGINE = engine_name    
        表在管理数据时采用的不同的数据结构，结构不同会导致处理方式、提供的特性操作等不同
        常见的引擎：InnoDB MyISAM Memory/Heap BDB Merge Example CSV MaxDB Archive
        不同的引擎在保存表的结构和数据时采用不同的方式
        MyISAM表文件含义：.frm表定义，.MYD表数据，.MYI表索引
        InnoDB表文件含义：.frm表定义，表空间数据和日志文件
        SHOW ENGINES -- 显示存储引擎的状态信息
        SHOW ENGINE 引擎名 {LOGS|STATUS} -- 显示存储引擎的日志或状态信息
    -- 数据文件目录
        DATA DIRECTORY = '目录'
    -- 索引文件目录
        INDEX DIRECTORY = '目录'
    -- 表注释
        COMMENT = 'string'
    -- 分区选项
        PARTITION BY ... (详细见手册)
-- 查看所有表
    SHOW TABLES[ LIKE 'pattern']
    SHOW TABLES FROM 表名
-- 查看表机构
    SHOW CREATE TABLE 表名    （信息更详细）
    DESC 表名 / DESCRIBE 表名 / EXPLAIN 表名 / SHOW COLUMNS FROM 表名 [LIKE 'PATTERN']
    SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']
-- 修改表
    -- 修改表本身的选项
        ALTER TABLE 表名 表的选项
        EG:    ALTER TABLE 表名 ENGINE=MYISAM;
    -- 对表进行重命名
        RENAME TABLE 原表名 TO 新表名
        RENAME TABLE 原表名 TO 库名.表名    （可将表移动到另一个数据库）
        -- RENAME可以交换两个表名
    -- 修改表的字段机构
        ALTER TABLE 表名 操作名
        -- 操作名
            ADD[ COLUMN] 字段名        -- 增加字段
                AFTER 字段名            -- 表示增加在该字段名后面
                FIRST                -- 表示增加在第一个
            ADD PRIMARY KEY(字段名)    -- 创建主键
            ADD UNIQUE [索引名] (字段名)-- 创建唯一索引
            ADD INDEX [索引名] (字段名)    -- 创建普通索引
            ADD 
            DROP[ COLUMN] 字段名        -- 删除字段
            MODIFY[ COLUMN] 字段名 字段属性        -- 支持对字段属性进行修改，不能修改字段名(所有原有属性也需写上)
            CHANGE[ COLUMN] 原字段名 新字段名 字段属性        -- 支持对字段名修改
            DROP PRIMARY KEY    -- 删除主键(删除主键前需删除其AUTO_INCREMENT属性)
            DROP INDEX 索引名    -- 删除索引
            DROP FOREIGN KEY 外键    -- 删除外键

-- 删除表
    DROP TABLE[ IF EXISTS] 表名 ...
-- 清空表数据
    TRUNCATE [TABLE] 表名
-- 复制表结构
    CREATE TABLE 表名 LIKE 要复制的表名
-- 复制表结构和数据
    CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名
-- 检查表是否有错误
    CHECK TABLE tbl_name [, tbl_name] ... [option] ...
-- 优化表
    OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
-- 修复表
    REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]
-- 分析表
    ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...

```


#### = /* 数据操作 */ DML

```

-- 增
    INSERT [INTO] 表名 [(字段列表)] VALUES (值列表)[, (值列表), ...]
        -- 如果要插入的值列表包含所有字段并且顺序一致，则可以省略字段列表。
        -- 可同时插入多条数据记录！
        REPLACE 与 INSERT 完全一样，可互换。
    INSERT [INTO] 表名 SET 字段名=值[, 字段名=值, ...]
-- 查
    SELECT 字段列表 FROM 表名[ 其他子句]
        -- 可来自多个表的多个字段
        -- 其他子句可以不使用
        -- 字段列表可以用*代替，表示所有字段
-- 删
    DELETE FROM 表名[ 删除条件子句]
        没有条件子句，则会删除全部
-- 改
    UPDATE 表名 SET 字段名=新值[, 字段名=新值] [更新条件]

/* 字符集编码 */ ------------------
-- MySQL、数据库、表、字段均可设置编码
-- 数据编码与客户端编码不需一致
SHOW VARIABLES LIKE 'character_set_%'    -- 查看所有字符集编码项
    character_set_client        客户端向服务器发送数据时使用的编码
    character_set_results        服务器端将结果返回给客户端所使用的编码
    character_set_connection    连接层编码
SET 变量名 = 变量值
    set character_set_client = gbk;
    set character_set_results = gbk;
    set character_set_connection = gbk;
SET NAMES GBK;    -- 相当于完成以上三个设置
-- 校对集
    校对集用以排序
    SHOW CHARACTER SET [LIKE 'pattern']/SHOW CHARSET [LIKE 'pattern']    查看所有字符集
    SHOW COLLATION [LIKE 'pattern']        查看所有校对集
    charset 字符集编码        设置字符集编码
    collate 校对集编码        设置校对集编码

```
###  /* 数据类型（列类型） */ 

```

1. 数值类型
-- a. 整型 ----------
    类型            字节        范围（有符号位）
    tinyint        1字节    -128 ~ 127        无符号位：0 ~ 255
    smallint    2字节    -32768 ~ 32767
    mediumint    3字节    -8388608 ~ 8388607
    int            4字节
    bigint        8字节

    int(M)    M表示总位数
    - 默认存在符号位，unsigned 属性修改
    - 显示宽度，如果某个数不够定义字段时设置的位数，则前面以0补填，zerofill 属性修改
        例：int(5)    插入一个数'123'，补填后为'00123'
    - 在满足要求的情况下，越小越好。
    - 1表示bool值真，0表示bool值假。MySQL没有布尔类型，通过整型0和1表示。常用tinyint(1)表示布尔型。

-- b. 浮点型 ----------
    类型                字节        范围
    float(单精度)        4字节
    double(双精度)    8字节
    浮点型既支持符号位 unsigned 属性，也支持显示宽度 zerofill 属性。
        不同于整型，前后均会补填0.
    定义浮点型时，需指定总位数和小数位数。
        float(M, D)        double(M, D)
        M表示总位数，D表示小数位数。
        M和D的大小会决定浮点数的范围。不同于整型的固定范围。
        M既表示总位数（不包括小数点和正负号），也表示显示宽度（所有显示符号均包括）。
        支持科学计数法表示。
        浮点数表示近似值。

-- c. 定点数 ----------
    decimal    -- 可变长度
    decimal(M, D)    M也表示总位数，D表示小数位数。
    保存一个精确的数值，不会发生数据的改变，不同于浮点数的四舍五入。
    将浮点数转换为字符串来保存，每9位数字保存为4个字节。

2. 字符串类型
-- a. char, varchar ----------
    char    定长字符串，速度快，但浪费空间
    varchar    变长字符串，速度慢，但节省空间
    M表示能存储的最大长度，此长度是字符数，非字节数。
    不同的编码，所占用的空间不同。
    char,最多255个字符，与编码无关。
    varchar,最多65535字符，与编码有关。
    一条有效记录最大不能超过65535个字节。
        utf8 最大为21844个字符，gbk 最大为32766个字符，latin1 最大为65532个字符
    varchar 是变长的，需要利用存储空间保存 varchar 的长度，如果数据小于255个字节，则采用一个字节来保存长度，反之需要两个字节来保存。
    varchar 的最大有效长度由最大行大小和使用的字符集确定。
    最大有效长度是65532字节，因为在varchar存字符串时，第一个字节是空的，不存在任何数据，然后还需两个字节来存放字符串的长度，所以有效长度是64432-1-2=65532字节。
    例：若一个表定义为 CREATE TABLE tb(c1 int, c2 char(30), c3 varchar(N)) charset=utf8; 问N的最大值是多少？ 答：(65535-1-2-4-30*3)/3

-- b. blob, text ----------
    blob 二进制字符串（字节字符串）
        tinyblob, blob, mediumblob, longblob
    text 非二进制字符串（字符字符串）
        tinytext, text, mediumtext, longtext
    text 在定义时，不需要定义长度，也不会计算总长度。
    text 类型在定义时，不可给default值

-- c. binary, varbinary ----------
    类似于char和varchar，用于保存二进制字符串，也就是保存字节字符串而非字符字符串。
    char, varchar, text 对应 binary, varbinary, blob.

3. 日期时间类型
    一般用整型保存时间戳，因为PHP可以很方便的将时间戳进行格式化。
    datetime    8字节    日期及时间        1000-01-01 00:00:00 到 9999-12-31 23:59:59
    date        3字节    日期            1000-01-01 到 9999-12-31
    timestamp    4字节    时间戳        19700101000000 到 2038-01-19 03:14:07
    time        3字节    时间            -838:59:59 到 838:59:59
    year        1字节    年份            1901 - 2155
    
datetime    “YYYY-MM-DD hh:mm:ss”
timestamp    “YY-MM-DD hh:mm:ss”
            “YYYYMMDDhhmmss”
            “YYMMDDhhmmss”
            YYYYMMDDhhmmss
            YYMMDDhhmmss
date        “YYYY-MM-DD”
            “YY-MM-DD”
            “YYYYMMDD”
            “YYMMDD”
            YYYYMMDD
            YYMMDD
time        “hh:mm:ss”
            “hhmmss”
            hhmmss
year        “YYYY”
            “YY”
            YYYY
            YY

4. 枚举和集合
-- 枚举(enum) ----------
enum(val1, val2, val3...)
    在已知的值中进行单选。最大数量为65535.
    枚举值在保存时，以2个字节的整型(smallint)保存。每个枚举值，按保存的位置顺序，从1开始逐一递增。
    表现为字符串类型，存储却是整型。
    NULL值的索引是NULL。
    空字符串错误值的索引值是0。

-- 集合（set） ----------
set(val1, val2, val3...)
    create table tab ( gender set('男', '女', '无') );
    insert into tab values ('男, 女');
    最多可以有64个不同的成员。以bigint存储，共8个字节。采取位运算的形式。
    当创建表时，SET成员值的尾部空格将自动被删除。

/* 选择类型 */
-- PHP角度
1. 功能满足
2. 存储空间尽量小，处理效率更高
3. 考虑兼容问题

-- IP存储 ----------
1. 只需存储，可用字符串
2. 如果需计算，查找等，可存储为4个字节的无符号int，即unsigned
    1) PHP函数转换
        ip2long可转换为整型，但会出现携带符号问题。需格式化为无符号的整型。
        利用sprintf函数格式化字符串
        sprintf("%u", ip2long('192.168.3.134'));
        然后用long2ip将整型转回IP字符串
    2) MySQL函数转换(无符号整型，UNSIGNED)
        INET_ATON('127.0.0.1') 将IP转为整型
        INET_NTOA(2130706433) 将整型转为IP
        

```


###  /* 列属性（列约束） */ 
```

1. 主键
    - 能唯一标识记录的字段，可以作为主键。
    - 一个表只能有一个主键。
    - 主键具有唯一性。
    - 声明字段时，用 primary key 标识。
        也可以在字段列表之后声明
            例：create table tab ( id int, stu varchar(10), primary key (id));
    - 主键字段的值不能为null。
    - 主键可以由多个字段共同组成。此时需要在字段列表后声明的方法。
        例：create table tab ( id int, stu varchar(10), age int, primary key (stu, age));

2. unique 唯一索引（唯一约束）
    使得某字段的值也不能重复。
    
3. null 约束
    null不是数据类型，是列的一个属性。
    表示当前列是否可以为null，表示什么都没有。
    null, 允许为空。默认。
    not null, 不允许为空。
    insert into tab values (null, 'val');
        -- 此时表示将第一个字段的值设为null, 取决于该字段是否允许为null
    
4. default 默认值属性
    当前字段的默认值。
    insert into tab values (default, 'val');    -- 此时表示强制使用默认值。
    create table tab ( add_time timestamp default current_timestamp );
        -- 表示将当前时间的时间戳设为默认值。
        current_date, current_time

5. auto_increment 自动增长约束
    自动增长必须为索引（主键或unique）
    只能存在一个字段为自动增长。
    默认为1开始自动增长。可以通过表属性 auto_increment = x进行设置，或 alter table tbl auto_increment = x;

6. comment 注释
    例：create table tab ( id int ) comment '注释内容';

7. foreign key 外键约束
    用于限制主表与从表数据完整性。
    alter table t1 add constraint `t1_t2_fk` foreign key (t1_id) references t2(id);
        -- 将表t1的t1_id外键关联到表t2的id字段。
        -- 每个外键都有一个名字，可以通过 constraint 指定

    存在外键的表，称之为从表（子表），外键指向的表，称之为主表（父表）。

    作用：保持数据一致性，完整性，主要目的是控制存储在外键表（从表）中的数据。

    MySQL中，可以对InnoDB引擎使用外键约束：
    语法：
    foreign key (外键字段） references 主表名 (关联字段) [主表记录删除时的动作] [主表记录更新时的动作]
    此时需要检测一个从表的外键需要约束为主表的已存在的值。外键在没有关联的情况下，可以设置为null.前提是该外键列，没有not null。

    可以不指定主表记录更改或更新时的动作，那么此时主表的操作被拒绝。
    如果指定了 on update 或 on delete：在删除或更新时，有如下几个操作可以选择：
    1. cascade，级联操作。主表数据被更新（主键值更新），从表也被更新（外键值更新）。主表记录被删除，从表相关记录也被删除。
    2. set null，设置为null。主表数据被更新（主键值更新），从表的外键被设置为null。主表记录被删除，从表相关记录外键被设置成null。但注意，要求该外键列，没有not null属性约束。
    3. restrict，拒绝父表删除和更新。

    注意，外键只被InnoDB存储引擎所支持。其他引擎是不支持的。

```



###  /* select */DQL

```

select [all|distinct] select_expr from -> where -> group by [合计函数] -> having -> order by -> limit

a. select_expr
    -- 可以用 * 表示所有字段。
        select * from tb;
    -- 可以使用表达式（计算公式、函数调用、字段也是个表达式）
        select stu, 29+25, now() from tb;
    -- 可以为每个列使用别名。适用于简化列标识，避免多个列标识符重复。
        - 使用 as 关键字，也可省略 as.
        select stu+10 as add10 from tb;

b. from 子句
    用于标识查询来源。
    -- 可以为表起别名。使用as关键字。
        select * from tb1 as tt, tb2 as bb;
    -- from子句后，可以同时出现多个表。
        -- 多个表会横向叠加到一起，而数据会形成一个笛卡尔积。
        select * from tb1, tb2;

c. where 子句
    -- 从from获得的数据源中进行筛选。
    -- 整型1表示真，0表示假。
    -- 表达式由运算符和运算数组成。
        -- 运算数：变量（字段）、值、函数返回值
        -- 运算符：
            =, <=>, <>, !=, <=, <, >=, >, !, &&, ||, 
            in (not) null, (not) like, (not) in, (not) between and, is (not), and, or, not, xor
            is/is not 加上ture/false/unknown，检验某个值的真假
            <=>与<>功能相同，<=>可用于null比较

d. group by 子句, 分组子句
    group by 字段/别名 [排序方式]
    分组后会进行排序。升序：ASC，降序：DESC
    
    以下[合计函数]需配合 group by 使用：
    count 返回不同的非NULL值数目    count(*)、count(字段)
    sum 求和
    max 求最大值
    min 求最小值
    avg 求平均值
    group_concat 返回带有来自一个组的连接的非NULL值的字符串结果。组内字符串连接。

e. having 子句，条件子句
    与 where 功能、用法相同，执行时机不同。
    where 在开始时执行检测数据，对原数据进行过滤。
    having 对筛选出的结果再次进行过滤。
    having 字段必须是查询出来的，where 字段必须是数据表存在的。
    where 不可以使用字段的别名，having 可以。因为执行WHERE代码时，可能尚未确定列值。
    where 不可以使用合计函数。一般需用合计函数才会用 having
    SQL标准要求HAVING必须引用GROUP BY子句中的列或用于合计函数中的列。

f. order by 子句，排序子句
    order by 排序字段/别名 排序方式 [,排序字段/别名 排序方式]...
    升序：ASC，降序：DESC
    支持多个字段的排序。

g. limit 子句，限制结果数量子句
    仅对处理好的结果进行数量限制。将处理好的结果的看作是一个集合，按照记录出现的顺序，索引从0开始。
    limit 起始位置, 获取条数
    省略第一个参数，表示从索引0开始。limit 获取条数

h. distinct, all 选项
    distinct 去除重复记录
    默认为 all, 全部记录


/* UNION */ ------------------
    将多个select查询的结果组合成一个结果集合。
    SELECT ... UNION [ALL|DISTINCT] SELECT ...
    默认 DISTINCT 方式，即所有返回的行都是唯一的
    建议，对每个SELECT查询加上小括号包裹。
    ORDER BY 排序时，需加上 LIMIT 进行结合。
    需要各select查询的字段数量一样。
    每个select查询的字段列表(数量、类型)应一致，因为结果中的字段名以第一条select语句为准。


/* 子查询 */ ------------------
    - 子查询需用括号包裹。
-- from型
    from后要求是一个表，必须给子查询结果取个别名。
    - 简化每个查询内的条件。
    - from型需将结果生成一个临时表格，可用以原表的锁定的释放。
    - 子查询返回一个表，表型子查询。
    select * from (select * from tb where id>0) as subfrom where id>1;
-- where型
    - 子查询返回一个值，标量子查询。
    - 不需要给子查询取别名。
    - where子查询内的表，不能直接用以更新。
    select * from tb where money = (select max(money) from tb);
    -- 列子查询
        如果子查询结果返回的是一列。
        使用 in 或 not in 完成查询
        exists 和 not exists 条件
            如果子查询返回数据，则返回1或0。常用于判断条件。
            select column1 from t1 where exists (select * from t2);
    -- 行子查询
        查询条件是一个行。
        select * from t1 where (id, gender) in (select id, gender from t2);
        行构造符：(col1, col2, ...) 或 ROW(col1, col2, ...)
        行构造符通常用于与对能返回两个或两个以上列的子查询进行比较。

    -- 特殊运算符
    != all()    相当于 not in
    = some()    相当于 in。any 是 some 的别名
    != some()    不等同于 not in，不等于其中某一个。
    all, some 可以配合其他运算符一起使用。


/* 连接查询(join) */ ------------------
    将多个表的字段进行连接，可以指定连接条件。
-- 内连接(inner join)
    - 默认就是内连接，可省略inner。
    - 只有数据存在时才能发送连接。即连接结果不能出现空行。
    on 表示连接条件。其条件表达式与where类似。也可以省略条件（表示条件永远为真）
    也可用where表示连接条件。
    还有 using, 但需字段名相同。 using(字段名)

    -- 交叉连接 cross join
        即，没有条件的内连接。
        select * from tb1 cross join tb2;
-- 外连接(outer join)
    - 如果数据不存在，也会出现在连接结果中。
    -- 左外连接 left join
        如果数据不存在，左表记录会出现，而右表为null填充
    -- 右外连接 right join
        如果数据不存在，右表记录会出现，而左表为null填充
-- 自然连接(natural join)
    自动判断连接条件完成连接。
    相当于省略了using，会自动查找相同字段名。
    natural join
    natural left join
    natural right join

select info.id, info.name, info.stu_num, extra_info.hobby, extra_info.sex from info, extra_info where info.stu_num = extra_info.stu_id;

/* 导入导出 */ ------------------
select * into outfile 文件地址 [控制格式] from 表名;    -- 导出表数据
load data [local] infile 文件地址 [replace|ignore] into table 表名 [控制格式];    -- 导入数据
    生成的数据默认的分隔符是制表符
    local未指定，则数据文件必须在服务器上
    replace 和 ignore 关键词控制对现有的唯一键记录的重复的处理
-- 控制格式
fields    控制字段格式
默认：fields terminated by '\t' enclosed by '' escaped by '\\'
    terminated by 'string'    -- 终止
    enclosed by 'char'        -- 包裹
    escaped by 'char'        -- 转义
    -- 示例：
        SELECT a,b,a+b INTO OUTFILE '/tmp/result.text'
        FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
        LINES TERMINATED BY '\n'
        FROM test_table;
lines    控制行格式
默认：lines terminated by '\n'
    terminated by 'string'    -- 终止

```

###  /* insert */ 
```

select语句获得的数据可以用insert插入。

可以省略对列的指定，要求 values () 括号内，提供给了按照列顺序出现的所有字段的值。
    或者使用set语法。
    insert into tbl_name set field=value,...；

可以一次性使用多个值，采用(), (), ();的形式。
    insert into tbl_name values (), (), ();

可以在列值指定时，使用表达式。
    insert into tbl_name values (field_value, 10+10, now());
可以使用一个特殊值 default，表示该列使用默认值。
    insert into tbl_name values (field_value, default);

可以通过一个查询的结果，作为需要插入的值。
    insert into tbl_name select ...;

可以指定在插入的值出现主键（或唯一索引）冲突时，更新其他非主键列的信息。
    insert into tbl_name values/set/select on duplicate key update 字段=值, …;

```
###  /* delete */ 
```

DELETE FROM tbl_name [WHERE where_definition] [ORDER BY ...] [LIMIT row_count]

按照条件删除

指定删除的最多记录数。Limit

可以通过排序条件删除。order by + limit

支持多表删除，使用类似连接语法。
delete from 需要删除数据多表1，表2 using 表连接操作 条件。

```

###  /* truncate 备份与还原*/ 
```

TRUNCATE [TABLE] tbl_name
清空数据
删除重建表

区别：
1，truncate 是删除表再创建，delete 是逐条删除
2，truncate 重置auto_increment的值。而delete不会
3，truncate 不知道删除了几条，而delete知道。
4，当被用于带分区的表时，truncate 会保留分区


/* 备份与还原 */ ------------------
备份，将数据的结构与表内数据保存起来。
利用 mysqldump 指令完成。

-- 导出
1. 导出一张表
　　mysqldump -u用户名 -p密码 库名 表名 > 文件名(D:/a.sql)
2. 导出多张表
　　mysqldump -u用户名 -p密码 库名 表1 表2 表3 > 文件名(D:/a.sql)
3. 导出所有表
　　mysqldump -u用户名 -p密码 库名 > 文件名(D:/a.sql)
4. 导出一个库 
　　mysqldump -u用户名 -p密码 -B 库名 > 文件名(D:/a.sql)

可以-w携带备份条件

-- 导入
1. 在登录mysql的情况下：
　　source  备份文件
2. 在不登录的情况下
　　mysql -u用户名 -p密码 库名 < 备份文件

```

###  /* 视图 */ 
```

什么是视图：
    视图是一个虚拟表，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。
    视图具有表结构文件，但不存在数据文件。
    对其中所引用的基础表来说，视图的作用类似于筛选。定义视图的筛选可以来自当前或其它数据库的一个或多个表，或者其它视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。
    视图是存储在数据库中的查询的sql语句，它主要出于两种原因：安全原因，视图可以隐藏一些数据，如：社会保险基金表，可以用视图只显示姓名，地址，而不显示社会保险号和工资数等，另一原因是可使复杂的查询易于理解和使用。

-- 创建视图
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement
    - 视图名必须唯一，同时不能与表重名。
    - 视图可以使用select语句查询到的列名，也可以自己指定相应的列名。
    - 可以指定视图执行的算法，通过ALGORITHM指定。
    - column_list如果存在，则数目必须等于SELECT语句检索的列数

-- 查看结构
    SHOW CREATE VIEW view_name 

-- 删除视图
    - 删除视图后，数据依然存在。
    - 可同时删除多个视图。
    DROP VIEW [IF EXISTS] view_name ...

-- 修改视图结构
    - 一般不修改视图，因为不是所有的更新视图都会映射到表上。
    ALTER VIEW view_name [(column_list)] AS select_statement

-- 视图作用
    1. 简化业务逻辑
    2. 对客户端隐藏真实的表结构

-- 视图算法(ALGORITHM)
    MERGE        合并
        将视图的查询语句，与外部查询需要先合并再执行！
    TEMPTABLE    临时表
        将视图执行完毕后，形成临时表，再做外层查询！
    UNDEFINED    未定义(默认)，指的是MySQL自主去选择相应的算法。

```


###  /* 事务(transaction) */ 
```

事务是指逻辑上的一组操作，组成这组操作的各个单元，要不全成功要不全失败。 
    - 支持连续SQL的集体成功或集体撤销。
    - 事务是数据库在数据晚自习方面的一个功能。
    - 需要利用 InnoDB 或 BDB 存储引擎，对自动提交的特性支持完成。
    - InnoDB被称为事务安全型引擎。

-- 事务开启
    START TRANSACTION; 或者 BEGIN;
    开启事务后，所有被执行的SQL语句均被认作当前事务内的SQL语句。
-- 事务提交
    COMMIT;
-- 事务回滚
    ROLLBACK;
    如果部分操作发生问题，映射到事务开启前。

-- 事务的特性
    1. 原子性（Atomicity）
        事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
    2. 一致性（Consistency）
        事务前后数据的完整性必须保持一致。
        - 事务开始和结束时，外部数据一致
        - 在整个事务过程中，操作是连续的
    3. 隔离性（Isolation）
        多个用户并发访问数据库时，一个用户的事务不能被其它用户的事物所干扰，多个并发事务之间的数据要相互隔离。
    4. 持久性（Durability）
        一个事务一旦被提交，它对数据库中的数据改变就是永久性的。

-- 事务的实现
    1. 要求是事务支持的表类型
    2. 执行一组相关的操作前开启事务
    3. 整组操作完成后，都成功，则提交；如果存在失败，选择回滚，则会回到事务开始的备份点。

-- 事务的原理
    利用InnoDB的自动提交(autocommit)特性完成。
    普通的MySQL执行语句后，当前的数据提交操作均可被其他客户端可见。
    而事务是暂时关闭“自动提交”机制，需要commit提交持久化数据操作。

-- 注意
    1. 数据定义语言（DDL）语句不能被回滚，比如创建或取消数据库的语句，和创建、取消或更改表或存储的子程序的语句。
    2. 事务不能被嵌套

-- 保存点
    SAVEPOINT 保存点名称 -- 设置一个事务保存点
    ROLLBACK TO SAVEPOINT 保存点名称 -- 回滚到保存点
    RELEASE SAVEPOINT 保存点名称 -- 删除保存点

-- InnoDB自动提交特性设置
    SET autocommit = 0|1;    0表示关闭自动提交，1表示开启自动提交。
    - 如果关闭了，那普通操作的结果对其他客户端也不可见，需要commit提交后才能持久化数据操作。
    - 也可以关闭自动提交来开启事务。但与START TRANSACTION不同的是，
        SET autocommit是永久改变服务器的设置，直到下次再次修改该设置。(针对当前连接)
        而START TRANSACTION记录开启前的状态，而一旦事务提交或回滚后就需要再次开启事务。(针对当前事务)


/* 锁表 */
表锁定只用于防止其它客户端进行不正当地读取和写入
MyISAM 支持表锁，InnoDB 支持行锁
-- 锁定
    LOCK TABLES tbl_name [AS alias]
-- 解锁
    UNLOCK TABLES

```

###  /* 触发器 */ 
```

    触发程序是与表有关的命名数据库对象，当该表出现特定事件时，将激活该对象
    监听：记录的增加、修改、删除。

-- 创建触发器
CREATE TRIGGER trigger_name trigger_time trigger_event ON tbl_name FOR EACH ROW trigger_stmt
    参数：
    trigger_time是触发程序的动作时间。它可以是 before 或 after，以指明触发程序是在激活它的语句之前或之后触发。
    trigger_event指明了激活触发程序的语句的类型
        INSERT：将新行插入表时激活触发程序
        UPDATE：更改某一行时激活触发程序
        DELETE：从表中删除某一行时激活触发程序
    tbl_name：监听的表，必须是永久性的表，不能将触发程序与TEMPORARY表或视图关联起来。
    trigger_stmt：当触发程序激活时执行的语句。执行多个语句，可使用BEGIN...END复合语句结构

-- 删除
DROP TRIGGER [schema_name.]trigger_name

可以使用old和new代替旧的和新的数据
    更新操作，更新前是old，更新后是new.
    删除操作，只有old.
    增加操作，只有new.

-- 注意
    1. 对于具有相同触发程序动作时间和事件的给定表，不能有两个触发程序。


-- 字符连接函数
concat(str1[, str2,...])

-- 分支语句
if 条件 then
    执行语句
elseif 条件 then
    执行语句
else
    执行语句
end if;

-- 修改最外层语句结束符
delimiter 自定义结束符号
    SQL语句
自定义结束符号

delimiter ;        -- 修改回原来的分号

-- 语句块包裹
begin
    语句块
end

-- 特殊的执行
1. 只要添加记录，就会触发程序。
2. Insert into on duplicate key update 语法会触发：
    如果没有重复记录，会触发 before insert, after insert;
    如果有重复记录并更新，会触发 before insert, before update, after update;
    如果有重复记录但是没有发生更新，则触发 before insert, before update
3. Replace 语法 如果有记录，则执行 before insert, before delete, after delete, after insert


```
###  /* SQL编程 */ 
```


--// 局部变量 ----------
-- 变量声明
    declare var_name[,...] type [default value] 
    这个语句被用来声明局部变量。要给变量提供一个默认值，请包含一个default子句。值可以被指定为一个表达式，不需要为一个常数。如果没有default子句，初始值为null。 

-- 赋值
    使用 set 和 select into 语句为变量赋值。

    - 注意：在函数内是可以使用全局变量（用户自定义的变量）


--// 全局变量 ----------
-- 定义、赋值
set 语句可以定义并为变量赋值。
set @var = value;
也可以使用select into语句为变量初始化并赋值。这样要求select语句只能返回一行，但是可以是多个字段，就意味着同时为多个变量进行赋值，变量的数量需要与查询的列数一致。
还可以把赋值语句看作一个表达式，通过select执行完成。此时为了避免=被当作关系运算符看待，使用:=代替。（set语句可以使用= 和 :=）。
select @var:=20;
select @v1:=id, @v2=name from t1 limit 1;
select * from tbl_name where @var:=30;

select into 可以将表中查询获得的数据赋给变量。
    -| select max(height) into @max_height from tb;

-- 自定义变量名
为了避免select语句中，用户自定义的变量与系统标识符（通常是字段名）冲突，用户自定义变量在变量名前使用@作为开始符号。
@var=10;

    - 变量被定义后，在整个会话周期都有效（登录到退出）


--// 控制结构 ----------
-- if语句
if search_condition then 
    statement_list    
[elseif search_condition then
    statement_list]
...
[else
    statement_list]
end if;

-- case语句
CASE value WHEN [compare-value] THEN result
[WHEN [compare-value] THEN result ...]
[ELSE result]
END


-- while循环
[begin_label:] while search_condition do
    statement_list
end while [end_label];

- 如果需要在循环内提前终止 while循环，则需要使用标签；标签需要成对出现。

    -- 退出循环
        退出整个循环 leave
        退出当前循环 iterate
        通过退出的标签决定退出哪个循环


```
###  --// 内置函数 
```

-- 数值函数
abs(x)            -- 绝对值 abs(-10.9) = 10
format(x, d)    -- 格式化千分位数值 format(1234567.456, 2) = 1,234,567.46
ceil(x)            -- 向上取整 ceil(10.1) = 11
floor(x)        -- 向下取整 floor (10.1) = 10
round(x)        -- 四舍五入去整
mod(m, n)        -- m%n m mod n 求余 10%3=1
pi()            -- 获得圆周率
pow(m, n)        -- m^n
sqrt(x)            -- 算术平方根
rand()            -- 随机数
truncate(x, d)    -- 截取d位小数

-- 时间日期函数
now(), current_timestamp();     -- 当前日期时间
current_date();                    -- 当前日期
current_time();                    -- 当前时间
date('yyyy-mm-dd hh:ii:ss');    -- 获取日期部分
time('yyyy-mm-dd hh:ii:ss');    -- 获取时间部分
date_format('yyyy-mm-dd hh:ii:ss', '%d %y %a %d %m %b %j');    -- 格式化时间
unix_timestamp();                -- 获得unix时间戳
from_unixtime();                -- 从时间戳获得时间

-- 字符串函数
length(string)            -- string长度，字节
char_length(string)        -- string的字符个数
substring(str, position [,length])        -- 从str的position开始,取length个字符
replace(str ,search_str ,replace_str)    -- 在str中用replace_str替换search_str
instr(string ,substring)    -- 返回substring首次在string中出现的位置
concat(string [,...])    -- 连接字串
charset(str)            -- 返回字串字符集
lcase(string)            -- 转换成小写
left(string, length)    -- 从string2中的左边起取length个字符
load_file(file_name)    -- 从文件读取内容
locate(substring, string [,start_position])    -- 同instr,但可指定开始位置
lpad(string, length, pad)    -- 重复用pad加在string开头,直到字串长度为length
ltrim(string)            -- 去除前端空格
repeat(string, count)    -- 重复count次
rpad(string, length, pad)    --在str后用pad补充,直到长度为length
rtrim(string)            -- 去除后端空格
strcmp(string1 ,string2)    -- 逐字符比较两字串大小

-- 流程函数
case when [condition] then result [when [condition] then result ...] [else result] end   多分支
if(expr1,expr2,expr3)  双分支。

-- 聚合函数
count()
sum();
max();
min();
avg();
group_concat()

-- 其他常用函数
md5();
default();


--// 存储函数，自定义函数 ----------
-- 新建
    CREATE FUNCTION function_name (参数列表) RETURNS 返回值类型
        函数体

    - 函数名，应该合法的标识符，并且不应该与已有的关键字冲突。
    - 一个函数应该属于某个数据库，可以使用db_name.funciton_name的形式执行当前函数所属数据库，否则为当前数据库。
    - 参数部分，由"参数名"和"参数类型"组成。多个参数用逗号隔开。
    - 函数体由多条可用的mysql语句，流程控制，变量声明等语句构成。
    - 多条语句应该使用 begin...end 语句块包含。
    - 一定要有 return 返回值语句。

-- 删除
    DROP FUNCTION [IF EXISTS] function_name;

-- 查看
    SHOW FUNCTION STATUS LIKE 'partten'
    SHOW CREATE FUNCTION function_name;

-- 修改
    ALTER FUNCTION function_name 函数选项

```

###  --// 存储过程，自定义功能 
```

-- 定义
存储存储过程 是一段代码（过程），存储在数据库中的sql组成。
一个存储过程通常用于完成一段业务逻辑，例如报名，交班费，订单入库等。
而一个函数通常专注与某个功能，视为其他程序服务的，需要在其他语句中调用函数才可以，而存储过程不能被其他调用，是自己执行 通过call执行。

-- 创建
CREATE PROCEDURE sp_name (参数列表)
    过程体

参数列表：不同于函数的参数列表，需要指明参数类型
IN，表示输入型
OUT，表示输出型
INOUT，表示混合型

注意，没有返回值。


/* 存储过程 */ ------------------
存储过程是一段可执行性代码的集合。相比函数，更偏向于业务逻辑。
调用：CALL 过程名
-- 注意
- 没有返回值。
- 只能单独调用，不可夹杂在其他语句中

-- 参数
IN|OUT|INOUT 参数名 数据类型
IN        输入：在调用过程中，将数据输入到过程体内部的参数
OUT        输出：在调用过程中，将过程体处理完的结果返回到客户端
INOUT    输入输出：既可输入，也可输出

-- 语法
CREATE PROCEDURE 过程名 (参数列表)
BEGIN
    过程体
END

```

###  /* 用户和权限管理 */ 
```

用户信息表：mysql.user
-- 刷新权限
FLUSH PRIVILEGES
-- 增加用户
CREATE USER 用户名 IDENTIFIED BY [PASSWORD] 密码(字符串)
    - 必须拥有mysql数据库的全局CREATE USER权限，或拥有INSERT权限。
    - 只能创建用户，不能赋予权限。
    - 用户名，注意引号：如 'user_name'@'192.168.1.1'
    - 密码也需引号，纯数字密码也要加引号
    - 要在纯文本中指定密码，需忽略PASSWORD关键词。要把密码指定为由PASSWORD()函数返回的混编值，需包含关键字PASSWORD
-- 重命名用户
RENAME USER old_user TO new_user
-- 设置密码
SET PASSWORD = PASSWORD('密码')    -- 为当前用户设置密码
SET PASSWORD FOR 用户名 = PASSWORD('密码')    -- 为指定用户设置密码
-- 删除用户
DROP USER 用户名
-- 分配权限/添加用户
GRANT 权限列表 ON 表名 TO 用户名 [IDENTIFIED BY [PASSWORD] 'password']
    - all privileges 表示所有权限
    - *.* 表示所有库的所有表
    - 库名.表名 表示某库下面的某表
-- 查看权限
SHOW GRANTS FOR 用户名
    -- 查看当前用户权限
    SHOW GRANTS; 或 SHOW GRANTS FOR CURRENT_USER; 或 SHOW GRANTS FOR CURRENT_USER();
-- 撤消权限
REVOKE 权限列表 ON 表名 FROM 用户名
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 用户名    -- 撤销所有权限
-- 权限层级
-- 要使用GRANT或REVOKE，您必须拥有GRANT OPTION权限，并且您必须用于您正在授予或撤销的权限。
全局层级：全局权限适用于一个给定服务器中的所有数据库，mysql.user
    GRANT ALL ON *.*和 REVOKE ALL ON *.*只授予和撤销全局权限。
数据库层级：数据库权限适用于一个给定数据库中的所有目标，mysql.db, mysql.host
    GRANT ALL ON db_name.*和REVOKE ALL ON db_name.*只授予和撤销数据库权限。
表层级：表权限适用于一个给定表中的所有列，mysql.talbes_priv
    GRANT ALL ON db_name.tbl_name和REVOKE ALL ON db_name.tbl_name只授予和撤销表权限。
列层级：列权限适用于一个给定表中的单一列，mysql.columns_priv
    当使用REVOKE时，您必须指定与被授权列相同的列。
-- 权限列表
ALL [PRIVILEGES]    -- 设置除GRANT OPTION之外的所有简单权限
ALTER    -- 允许使用ALTER TABLE
ALTER ROUTINE    -- 更改或取消已存储的子程序
CREATE    -- 允许使用CREATE TABLE
CREATE ROUTINE    -- 创建已存储的子程序
CREATE TEMPORARY TABLES        -- 允许使用CREATE TEMPORARY TABLE
CREATE USER        -- 允许使用CREATE USER, DROP USER, RENAME USER和REVOKE ALL PRIVILEGES。
CREATE VIEW        -- 允许使用CREATE VIEW
DELETE    -- 允许使用DELETE
DROP    -- 允许使用DROP TABLE
EXECUTE        -- 允许用户运行已存储的子程序
FILE    -- 允许使用SELECT...INTO OUTFILE和LOAD DATA INFILE
INDEX     -- 允许使用CREATE INDEX和DROP INDEX
INSERT    -- 允许使用INSERT
LOCK TABLES        -- 允许对您拥有SELECT权限的表使用LOCK TABLES
PROCESS     -- 允许使用SHOW FULL PROCESSLIST
REFERENCES    -- 未被实施
RELOAD    -- 允许使用FLUSH
REPLICATION CLIENT    -- 允许用户询问从属服务器或主服务器的地址
REPLICATION SLAVE    -- 用于复制型从属服务器（从主服务器中读取二进制日志事件）
SELECT    -- 允许使用SELECT
SHOW DATABASES    -- 显示所有数据库
SHOW VIEW    -- 允许使用SHOW CREATE VIEW
SHUTDOWN    -- 允许使用mysqladmin shutdown
SUPER    -- 允许使用CHANGE MASTER, KILL, PURGE MASTER LOGS和SET GLOBAL语句，mysqladmin debug命令；允许您连接（一次），即使已达到max_connections。
UPDATE    -- 允许使用UPDATE
USAGE    -- “无权限”的同义词
GRANT OPTION    -- 允许授予权限

```

###  /* 表维护 */ 
```

-- 分析和存储表的关键字分布
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE 表名 ...
-- 检查一个或多个表是否有错误
CHECK TABLE tbl_name [, tbl_name] ... [option] ...
option = {QUICK | FAST | MEDIUM | EXTENDED | CHANGED}
-- 整理数据文件的碎片
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...

```

###  /* 杂项注释等 
```

1. 可用反引号（`）为标识符（库名、表名、字段名、索引、别名）包裹，以避免与关键字重名！中文也可以作为标识符！
2. 每个库目录存在一个保存当前数据库的选项文件db.opt。
3. 注释：
    单行注释 # 注释内容
    多行注释 /* 注释内容 */
    单行注释 -- 注释内容        (标准SQL注释风格，要求双破折号后加一空格符（空格、TAB、换行等）)
4. 模式通配符：
    _    任意单个字符
    %    任意多个字符，甚至包括零字符
    单引号需要进行转义 \'
5. CMD命令行内的语句结束符可以为 ";", "\G", "\g"，仅影响显示结果。其他地方还是用分号结束。delimiter 可修改当前对话的语句结束符。
6. SQL对大小写不敏感
7. 清除已有语句：\c

```
##  了解SQL的十步 
很多程序员认为SQL是一头难以驯服的野兽。它是为数不多的声明性语言之一，也因为这样，其展示了完全不同于其他的表现形式、命令式语言、 面向对象语言甚至函数式编程语言（虽然有些人觉得SQL 还是有些类似功能）。

　　我每天都写SQL，我的开源软件JOOQ中也包含SQL。因此我觉得有必要为还在为此苦苦挣扎的你呈现SQL的优美！下面的教程面向于：

已经使用过但没有完全理解SQL的读者

已经差不多了解SQL但从未真正考虑过它的语法的读者

想要指导他人学习SQL的读者

　　本教程将重点介绍SELECT 语句。其他 DML 语句将在另一个教程中在做介绍。接下来就是…

　　1、SQL是声明性语言

　　首先你需要思考的是，声明性。你唯一需要做的只是声明你想获得结果的性质，而不需要考虑你的计算机怎么算出这些结果的。

SELECT first_name, last_name FROM employees WHERE salary > 100000
　　这很容易理解，你无须关心员工的身份记录从哪来，你只需要知道谁有着不错的薪水。

　　从中我们学到了什么呢？

　　那么如果它是这样的简单，会出现什么问题吗？问题就是我们大多数人会直观地认为这是命令式编程。如：“机器，做这，再做那，但在这之前，如果这和那都发生错误，那么会运行一个检测”。这包括在变量中存储临时的编写循环、迭代、调用函数，等等结果。

　　把那些都忘了吧，想想怎么去声明，而不是怎么告诉机器去计算。

　　2、SQL语法不是“有序的”

　　常见的混乱源于一个简单的事实，SQL语法元素并不会按照它们的执行方式排序。语法顺序如下：

SELECT [DISTINCT]

FROM

WHERE

GROUP BY

HAVING

UNION

ORDER BY

　　为简单起见，并没有列出所有SQL语句。这个语法顺序与逻辑顺序基本上不同，执行顺序如下： 

FROM

WHERE

GROUP BY

HAVING

SELECT

DISTINCT

UNION

ORDER BY

　　这有三点需要注意：

　　1、第一句是FROM，而不是SELECT。首先是将数据从磁盘加载到内存中，以便对这些数据进行操作。

　　2、SELECT是在其他大多数语句后执行，最重要的是，在FROM和GROUP BY之后。重要的是要理解当你觉得你可以从WHERE语句中引用你定义在SELECT语句当中的时候，。以下是不可行的：

SELECT A.x + A.y AS z

FROM A

WHERE z = 10 -- z is not available here!
　　如果你想重用z,您有两种选择。要么重复表达式: 

SELECT A.x + A.y AS z

FROM A

WHERE (A.x + A.y) = 10
　　或者你使用派生表、公用表表达式或视图来避免代码重复。请参阅示例进一步的分析：

　　3、在语法和逻辑顺序里，UNION都是放在ORDER BY之前，很多人认为每个UNION子查询都可以进行排序，但根据SQL标准和大多数的SQL方言，并不是真的可行。虽然一些方言允许子查询或派生表排序，但却不能保证这种顺序能在UNION操作后保留。

　　需要注意的是，并不是所有的数据库都以相同的形式实现，例如规则2并不完全适用于MySQL,PostgreSQL,和SQLite上

　　从中我们学到了什么呢？

　　要时刻记住SQL语句的语法顺序和逻辑顺序来避免常见的错误。如果你能明白这其中的区别，就能明确知道为什么有些可以执行有些则不能。

　　如果能设计一种在语法顺序上实际又体现了逻辑顺序的语言就更好了，因为它是在微软的LINQ上实现的。

　　3、SQL是关于数据表引用的 

　　因为语法顺序和逻辑顺序的差异，大多数初学者可能会误认为SQL中列的值是第一重要的。其实并非如此，最重要的是数据表引用。

　　该SQL标准定义了FROM语句，如下：

<from clause> ::= FROM &lt;table reference&gt; [ { &lt;comma&gt; &lt;table reference&gt; }... ]
　　ROM语句的"output"是所有表引用的结合程度组合表引用。让我们慢慢地消化这些。 

FROM a, b
　　上述产生一个a+b度的组合表引用，如果a有3列和b有5列，那么"输出表"将有8（3+5）列。

　　包含在这个组合表引用的记录是交叉乘积/笛卡儿积的axb。换句话说，每一条a记录都会与每一条b记录相对应。如果a有3个记录和b有5条记录,然后上面的组合表引用将产生15条记录(3×5)。

　　在WHERE语句筛选后，GROUP BY语句中"output"是"fed"/"piped"，它已转成新的"output"，我们会稍后再去处理。

　　如果我们从关系代数/集合论的角度来看待这些东西，一个SQL表是一个关系或一组元素组合。每个SQL语句将改变一个或几个关系，来产生新的关系。

　　从中我们学到了什么呢？

　　一直从数据表引用角度去思考，有助于理解数据怎样通过你的sql语句流水作业的

　　4、SQL数据表引用可以相当强大

　　表引用是相当强大的东西。举个简单的例子，JOIN关键字其实不是SELECT语句的一部分，但却是"special"表引用的一部分。连接表，在SQL标准中有定义（简化的）：

复制代码
<table reference> ::=

<table name>

| <derived table>

| <joined table>
复制代码
　　如果我们又拿之前的例子来分析： 

FROM a, b
　　a可以作为一个连接表，如：

a1 JOIN a2 ON a1.id = a2.id
　　这扩展到前一个表达式，我们会得到：

FROM a1 JOIN a2 ON a1.id = a2.id, b
　　虽然结合了数据表引用语法与连接表语法的逗号分隔表让人很无语，但你肯定还会这样做的。结果，结合数据表引用将有a1+a2+b度。

　　派生表甚至比连接表更强大，我们接下来将会说到。

　　从中我们学到了什么呢？

　　要时时刻刻考虑表引用，重要的是这不仅让你理解数据怎样通过你的sql语句流水作业的，它还将帮助你了解复杂表引用是如何构造的。

　　而且，重要的是，了解JOIN是构造连接表的关键字。不是的SELECT语句的一部分。某些数据库允许JOIN在插入、更新、删除中使用。

　　5、应使用SQL JOIN的表，而不是以逗号分隔表 

　　前面，我们已经看到这语句： 

FROM a, b
　　高级SQL开发人员可能会告诉你，最好不要使用逗号分隔的列表，并且一直完整的表达你的JOINs。这将有助于改进你的SQL语句的可读性从而防止错误出现。

　　一个非常常见的错误是忘记某处连接谓词。思考以下内容：

复制代码
FROM a, b, c, d, e, f, g, h

WHERE a.a1 = b.bx

AND a.a2 = c.c1

AND d.d1 = b.bc

-- etc...
复制代码
　　使用join来查询表的语法

更安全，你可以把连接谓词与连接表放一起，从而防止错误。

更富于表现力，你可以区分外部连接，内部连接，等等。​​

　　从中我们学到了什么呢？

　　使用JOIN，并且永远不在FROM语句中使用逗号分隔表引用。 

　　6、SQL的不同类型的连接操作

　　连接操作基本上有五种

EQUI JOIN

SEMI JOIN

ANTI JOIN

CROSS JOIN

DIVISION

　　这些术语通常用于关系代数。对上述概念，如果他们存在，SQL会使用不同的术语。让我们仔细看看:

　　EQUI JOIN（同等连接）

　　这是最常见的JOIN操作。它有两个子操作:

INNER JOIN(或者只是JOIN)

OUTER JOIN(可以再次拆分为LEFT, RIGHT,FULL OUTER JOIN)

　　例子是其中的区别最好的解释:

复制代码
-- This table reference contains authors and their books.

-- There is one record for each book and its author.

-- authors without books are NOT included

author JOIN book ON author.id = book.author_id



-- This table reference contains authors and their books

-- There is one record for each book and its author.

-- ... OR there is an "empty" record for authors without books

-- ("empty" meaning that all book columns are NULL)

author LEFT OUTER JOIN book ON author.id = book.author_id
复制代码
　　SEMI JOIN（半连接）

　　这种关系的概念在SQL中用两种方式表达：使用IN谓词或使用EXISTS谓语。"Semi"是指在拉丁语中的"half"。这种类型的连接用于连接只有"half"的表引用。再次考虑上述加入的作者和书。让我们想象，我们想要作者/书的组合,但只是那些作者实际上也有书。然后我们可以这样写:

复制代码
-- Using IN

FROM author

WHERE author.id IN (SELECT book.author_id FROM book)

-- Using EXISTS

FROM author

WHERE EXISTS (SELECT 1 FROM book WHERE book.author_id = author.id)
复制代码
　　虽然不能肯定你到底是更加喜欢IN还是EXISTS，而且也没有规则说明，但可以这样说：

IN往往比EXISTS更具可读性

EXISTS往往比IN更富表现力（如它更容易表达复杂的半连接）

一般情况下性能上没有太大的差异，但，在某些数据库可能会有巨大的性能差异。

　　因为INNER JOIN有可能只产生有书的作者，因为很多初学者可能认为他们可以使用DISTINCT删除重复项。他们认为他们可以表达一个像这样的半联接：

-- Find only those authors who also have books

SELECT DISTINCT first_name, last_name

FROM author
　　这是非常不好的做法，原因有二：

它非常慢，因为该数据库有很多数据加载到内存中，只是要再删除重复项。

它不完全正确，即使在这个简单的示例中它产生了正确的结果。但是，一旦你JOIN更多的表引用，，你将很难从你的结果中正确删除重复项。

　　更多的关于DISTINCT滥用的问题，可以访问这里的博客。

　　ANTI JOIN（反连接）

　　这个关系的概念跟半连接刚好相反。您可以简单地通过将 NOT 关键字添加到IN 或 EXISTS中生成它。在下例中，我们选择那些没有任何书籍的作者：

复制代码
-- Using IN

FROM author

WHERE author.id NOT IN (SELECT book.author_id FROM book)


-- Using EXISTS

FROM author

WHERE NOT EXISTS (SELECT 1 FROM book WHERE book.author_id = author.id)
复制代码
　　同样的规则对性能、可读性、表现力都适用。然而，当使用NOT IN时对NULLs会有一个小警告，这个问题有点超出本教程范围。

　　CROSS JOIN（交叉连接）

　　结合第一个表中的内容和第二个表中的内容，引用两个join表交叉生成一个新的东西。我们之前已经看到，这可以在FROM语句中通过逗号分隔表引用来实现。在你确实需要的情况下，可以在SQL语句中明确地写一个CROSS JOIN。

-- Combine every author with every book

author CROSS JOIN book
　　DIVISION（除法）

　　关系分割就是一只真正的由自己亲自喂养的野兽。简而言之，如果JOIN是乘法，那么除法就是JOIN的反义词。在SQL中，除法关系难以表达清楚。由于这是一个初学者的教程，解释这个问题超出了我们的教程范围。当然如果你求知欲爆棚，那么就看这里，这里还有这里。

　　从中我们学到了什么呢？

　　让我们把前面讲到的内容再次牢记于心。SQL是表引用。连接表是相当复杂的表引用。但关系表述和SQL表述还是有点区别的，并不是所有的关系连接操作都是正规的SQL连接操作。对关系理论有一点实践与认识，你就可以选择JOIN正确的关系类型并能将其转化为正确的SQL。

　　7、SQL的派生表就像表的变量

　　前文，我们已经了解到SQL是一种声明性语言，因此不会有变量。（虽然在一些SQL语句中可能会存在）但你可以写这样的变量。那些野兽一般的表被称为派生表。

　　派生表只不过是包含在括号里的子查询。

-- A derived table

FROM (SELECT * FROM author)
　　需要注意的是，一些SQL方言要求派生表有一个关联的名字（也被称为别名）。

-- A derived table with an alias

FROM (SELECT * FROM author) a
　　当你想规避由SQL子句逻辑排序造成的问题时，你会发现派生表可以用帅呆了来形容。例如，如果你想在SELECT和WHERE子句中重用一个列表达式，只写（Oracle方言）：

复制代码
-- Get authors' first and last names, and their age in days

SELECT first_name, last_name, age

FROM (

SELECT first_name, last_name, current_date - date_of_birth age

FROM author

)

-- If the age is greater than 10000 days

WHERE age > 10000
复制代码
　　注意，一些数据库和SQL:1999标准里已将派生表带到下一级别，,引入公共表表达式。这将允许你在单一的SQL SELECT中重复使用相同的派生表。上面的查询将转化为类似这样的：

复制代码
WITH a AS (

SELECT first_name, last_name, current_date - date_of_birth age

FROM author

)

SELECT *

FROM a

WHERE age > 10000
复制代码
　　很明显，对广泛重用的常见SQL子查询，你也可以灌输具体"a"到一个独立视图中。想要了解更多就看这里。

　　从中我们学到了什么呢？

　　再温习一遍，SQL主要是关于表引用，而不是列。好好利用这些表引用。不要害怕写派生表或其他复杂的表引用。

　　8、SQL GROUP BY转换之前的表引用

　　让我们重新考虑我们之前的FROM语句：

FROM a, b
　　现在,让我们来应用一个GROUP BY语句到上述组合表引用

GROUP BY A.x, A.y, B.z
　　这会产生一个只有其余三个列(!)的新的表引用。让我们再消化一遍。如果你用了GROUP BY，那么你在所有后续逻辑条款-包括选择中减少可用列的数量。这就是为什么你只可以从SELECT语句中的GROUP BY语句引用列语法的原因。

请注意，其他列仍然可能可作为聚合函数的参数：
SELECT A.x, A.y, SUM(A.z) 
FROM A 
GROUP BY A.x, A.y
 

值得注意并很不幸的是，MySQL不遵守这一标准，只会造成混乱。不要陷入MySQL的把戏。GROUP BY转换表引用，因此你可以只引用列也引用GROUPBY语句。
从中我们学到了什么呢？

GROUP BY，在表引用上操作，将它们转换成一个新表。

　　9、SQL SELECT在关系代数中被称为投影

　　当它在关系代数中使用时，我个人比较喜欢用"投影"一词中。一旦你生成你的表引用，过滤，转换它，你可以一步将它投影到另一个表中。SELECT语句就像一个投影机。表函数利用行值表达式将之前构造的表引用的每个记录转化为最终结果。

　　在SELECT语句中，你终于可以在列上操作，创建复杂的列表达式作为记录/行的一部分。

　　有很多关于可用的表达式，函数性质等的特殊规则。最重要的是，你应该记住这些：

　　1、你只能使用从“output”表引用产生的列引用

　　2、如果你有GROUP BY语句，你只可能从该语句或聚合函数引用列

　　3、当你没有GROUP BY语句时，你可以用窗口函数替代聚合函数

　　4、如果你没有GROUP BY语句，你就不能将聚合函数与非聚合函数结合起来

　　5、这有一些关于在聚合函数包装常规函数的规则，反之亦然

　　6、还有…

　　嗯，这有很多复杂的规则。他们可以填补另一个教程。例如，之所以你不能将聚合函数与非聚合函数结合起来投影到没有GROUP BY的SELECT语句中是因为：

　　1、凭直觉，没有任何意义。

　　2、对一个SQL初学者来说，直觉还是毫无帮助的，语法规则则可以。SQL:1999年引入了分组集，SQL：2003引入了空分组集GROUP BY()。每当存在没有显式GROUP BY语句的聚合函数时就会应用隐式的空分组集（规则2）。因此，最初关于逻辑顺序的那个规则就不完全正确了，SELECT的投影也会影响前面的逻辑结果，但语法语句GROUP BY却不受影响。

是不是有点迷糊？其实我也是，让我们看一些简单的吧。

　　从中我们学到了什么呢？

　　在SQL语句中，SELECT语句可能是最复杂的语句之一，即使它看起来是那么的简单。所有其他语句只不过是从这个到另一个表引用的的输送管道。通过将它们完全转化，后期地对它们应用一些规则，SELECT语句完完全全地搅乱了这些表引用的美。

　　为了了解SQL，重要的是要理解其他的一切，都要尝试在SELECT之前解决。即便SELECT在语法顺序中排第一的语句，也应该将它放到最后。

　　10.相对简单一点的SQL DISTINCT,UNION,ORDER BY,和OFFSET

　　看完复杂的SELECT之后，我们看回一些简单的东西。

集合运算（DISTINCT和UNION）

排序操作（ORDER BY,OFFSET..FETCH）

　　集合运算

　　集合运算在除了表其实没有其他东西的“集”上操作。嗯，差不多是这样，从概念上讲，它们还是很容易理解的

DISTINCT投影后删除重复项。

UNION求并集，删除重复项。

UNION ALL求并集，保留重复项。

EXCEPT求差集（在第一个子查询结果中删除第二个子查询中也含有的记录删除），删除重复项。

INTERSECT求交集（保留所有子查询都含有的记录），删除重复项。

　　所有这些删除重复项通常是没有意义的，很多时候，当你想要连接子查询时，你应该使用UNION ALL。

　　排序操作

　　排序不是一个关系特征，它是SQL仅有的特征。在你的SQL语句中，它被应用在语法排序和逻辑排序之后。保证可以通过索引访问记录的唯一可靠方法是使用ORDER BY a和OFFSET..FETCH。所有其他的排序总是任意的或随机的，即使它看起来像是可再现的。

　　OFFSET..FETCH是唯一的语法变体。其他变体包括MySQL'和PostgreSQL的LIMIT..OFFSET，或者SQL Server和Sybase的TOP..START AT（这里）。

　　让我们开始应用吧

　　跟其他每个语言一样，要掌握SQL语言需要大量的实践。上述10个简单的步骤将让你每天编写SQL时更有意义。另一方面，你也可以从常见的错误中学习到更多。下面的两篇文章列出许多Java（和其他）开发者写SQL时常见的错误：
参考：
http://www.cnblogs.com/hoojo/archive/2011/06/20/2085390.html
http://www.cnblogs.com/hoojo/archive/2011/06/20/2085416.html
http://www.cnblogs.com/shockerli/p/1000-plus-line-mysql-notes.html
http://www.cnblogs.com/shockerli/p/10-easy-steps-to-a-complete-understanding-of-sql.html
还可参考:http://www.cnblogs.com/nerxious/archive/2013/01/04/2844354.html mysql学习系列笔记
http://www.cnblogs.com/lyhabc/p/3691555.html mysql学习心得系列
