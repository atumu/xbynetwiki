title: mysql高级编程 

#  mysql高级编程 
##  LOAD DATA INFILE语句表数据导入 
http://blog.sina.com.cn/s/blog_605f5b4f0100zoqb.html
使用该语句可以从一个额文件载入表数据。
 LOAD DATA INFILE 'data.txt' INTO TABLE my_table;
注意：文件路径指定Windows 路径名时，使用的是斜线而不是反斜线。如果要用反斜线，必须双写。当对服务器上的文件执行LOAD DATA INFILE 时，用户必须获得FILE 权限。

例如：
Load Data InFile 'C:/Data.txt' Into Table `TableTest` Lines Terminated By '\r\n';
这个语句，**字段默认用制表符隔开，每条记录用换行符隔开**，在Windows下换行符为“\r\n”
C:/Data.txt 文件内容如下面两行：
1 A
2 B
“1”和“A”之间有一个制表符
这样就导进两条记录了。

其余选项说明:
  * Fields Terminated By ',' Enclosed By '"' Escaped By '"'表示每个字段用逗号分开，内容包含在双引号内
  * Lines Terminated By '\r\n';表示每条数据用换行符分开
例如:
Load Data InFile 'C:/Data.txt' Into Table `TableTest` Fields Terminated By ',' Enclosed By '"' Escaped By '"' Lines Terminated By '\r\n';
"老三","24"
"老四","34"

**和 Load Data InFile 相反的是**
` Select * From `TableTest` Into OutFile 'C:/Data_OutFile.txt';把表的数据导出 `

##  存储引擎 
http://www.cnblogs.com/gbyukg/archive/2011/11/09/2242271.html
http://www.jb51.net/article/55849.htm
**MySQL5.5以后默认使用InnoDB存储引擎，其中InnoDB和BDB提供` 事务安全 `表，其它存储引擎都是非事务安全表。**
**若要修改默认引擎，可以修改配置文件中的default-storage-engine。**可以通过：**show variables like 'default_storage_engine';查看当前数据库到默认引擎。**命令：show engines和show variables like 'have%'可以列出当前数据库所支持到引擎。其中Value显示为disabled的记录表示数据库支持此引擎，而在数据库启动时被禁用。
在MySQL5.1以后，INFORMATION_SCHEMA数据库中存在一个ENGINES的表，它提供的信息与show engines;语句完全一样，可以使用下面语句来查询哪些存储引擎支持事物处理：select engine from information_chema.engines where transactions = 'yes';
**可以通过engine关键字在创建或修改数据库时指定所使用到引擎。**

**主要存储引擎：MyISAM、InnoDB、MEMORY和MERGE介绍：**
**在创建表到时候通过engine=...或type=...来指定所要使用到引擎。show table status from DBname来查看指定表到引擎。**
(一)MyISAM
　　它不支持事务，也不支持外键，尤其是访问速度快，对事务完整性没有要求或者以SELECT、INSERT为主的应用基本都可以使用这个引擎来创建表。

(二)InnoDB
　　**InnoDB存储引擎提供了具有提交、回滚和崩溃恢复能力的事务安全。**但是对比MyISAM的存储引擎，InnoDB写的处理效率差一些并且会占用更多的磁盘空间以保留数据和索引。
**1)自动增长列：**
　　InnoDB表的自动增长列可以手工插入，但是插入的如果是空或0，则实际插入到则是自动增长后到值。可以通过"ALTER TABLE...AUTO_INCREMENT=n;"语句强制设置自动增长值的起始值，默认为1，但是该强制到默认值是保存在内存中，数据库重启后该值将会丢失。可以使用LAST_INSERT_ID()查询当前线程最后插入记录使用的值。如果一次插入多条记录，那么返回的是第一条记录使用的自动增长值。
对于InnoDB表，自动增长列必须是索引。如果是组合索引，也必须是组合索引的第一列，但是对于MyISAM表，自动增长列可以是组合索引的其他列，这样插入记录后，自动增长列是按照组合索引到前面几列排序后递增的。
**2)外键约束：**
**MySQL支持外键的存储引擎只有InnoDB**，在创建外键的时候，父表必须有对应的索引，子表在创建外键的时候也会自动创建对应的索引。
当某个表被其它表创建了外键参照，那么该表对应的索引或主键被禁止删除。
可以使用set foreign_key_checks=0;临时关闭外键约束，set foreign_key_checks=1;打开约束。

(三)MEMORY
memory使用存在内存中的内容来创建表。每个MEMORY表实际对应一个磁盘文件，格式是.frm。MEMORY类型的表访问非常快，因为它到数据是放在内存中的，并且默认使用HASH索引，但是一旦服务器关闭，表中的数据就会丢失，但表还会继续存在。

(四)MERGE
　　merge存储引擎是一组MyISAM表的组合，这些MyISAM表结构必须完全相同，MERGE表中并没有数据，对MERGE类型的表可以进行查询、更新、删除的操作，这些操作实际上是对内部的MyISAM表进行操作。

修改存储引擎：
alter table orders type=innodb;

create table my{

} type=innodb;

##  事务 
http://www.cnblogs.com/ymy124/p/3718439.html
只有InnoDB支持事务
**事务的ACID原则** 
Atomicity（原子性）、Consistency（稳定性）、Isolation（隔离性）、Durability（可靠性）
1、事务的原子性
一组事务，要么成功；要么撤回。
2、稳定性
有非法数据（外键约束之类），事务撤回。
3、隔离性
事务独立运行。一个事务处理后的结果，影响了其他事务，那么其他事务会撤回。事务的100%隔离，需要牺牲速度。
4、可靠性
软、硬件崩溃后，InnoDB数据表驱动会利用日志文件重构修改。可靠性和高速度不可兼得， innodb_flush_log_at_trx_commit选项 决定什么时候吧事务保存到日志里。

默认为“自动提交”模式(SET AUTOCOMMIT = 0)
开启事务
START TRANSACTION 或 BEGIN
提交事务（关闭事务）
COMMIT
放弃事务（关闭事务）
ROLLBACK
折返点
SAVEPOINT adqoo_1
ROLLBACK TO SAVEPOINT adqoo_1
发生在折返点 adqoo_1 之前的事务被提交，之后的被忽略

##  外键 
http://www.cnblogs.com/xiangxiaodong/archive/2013/05/05/3061049.html
为已经添加好的数据表添加外键:
语法:alter table 表名 add constraint 外键的名称 foreign key(你的外键字段名) REFERENCES 外表表名(对应的表的主键字段名);
例: alter table tb_active add constraint FK_ID foreign key(user_id) REFERENCES tb_user(id)
```

CREATE TABLE `tb_active` (
 `id` int(11) NOT NULL AUTO_INCREMENT,
 `title` varchar(100) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
 `content` text CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
 `user_id` int(11) NOT NULL,
 PRIMARY KEY (`id`),
 KEY `user_id` (`user_id`),
 KEY `user_id_2` (`user_id`),
 CONSTRAINT `FK_ID` FOREIGN KEY (`user_id`) REFERENCES `tb_user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf-8


``` 
**删除外键**
语法： ALTER TABLE table-name DROP FOREIGN KEY key-id;
例：   ALTER TABLE `tb_active` DROP FOREIGN KEY `FK_ID`

自动键更新和删除：
外键可以保证新插入的记录的完整性，但是，如果在REFERENCES从句中已命名的表删除记录会怎么样？在使用同样的值作为外键的辅助表中会发生什么?
 很明显，那些记录也应该被删除，否则在数据库中就会有很多无意义的孤立记录，MYSQL可以通过向FOREIGN KEY...REFERENCES修饰符添加一个ON DELETE 或ON UPDATE子句简化任务，它告诉了数据库在这种情况如何处理孤立任务
 关键字     含义
  *  CASCADE    删除包含与已删除键值有参照关系的所有记录
  *  SET NULL   修改包含与已删除键值有参照关系的所有记录，使用NULL值替换（只能用于已标记为NOT NULL的字段）
  *  RESTRICT   拒绝删除要求，直到使用删除键值的辅助表被手工删除，并且没有参照时(这是默认设置，也是最安全的设置)
  *  NO ACTION  啥也不做
 请注意，通过ON UPDATE 和 ON DELETE规则，设置MYSQL能够实现自动操作时，如果键的关系没有设置好，可能会导致严重的数据破坏，
 例如：如果一系列的表通过外键关系和ON DELETE CASCADE 规则连接时，任意一个主表的变化都会导致甚至只和原始删除有一些将要联系的记录在没有警告的情况被删除，所以，我们在操作之前还要检查这些规则的，操作之后还要再次检查.
查看表有哪些外键
show create table locstock
##  存储过程与存储函数 
http://www.cnblogs.com/lyhabc/p/3793524.html
MYSQL中创建存储过程和函数分别使用` CREATE PROCEDURE和CREATE FUNCTION `
使用` CALL语句 `来调用存储过程，存储过程也可以调用其他存储过程
函数可以从语句外调用，能返回标量值
**执行存储过程和存储函数需要拥有EXECUTE权限**，EXECUTE权限的信息存储在information_schema数据库下面的USER_PRIVILEGES表中
###  创建存储过程 
语法
CREATE PROCEDURE sp_name ([ proc_parameter ])
proc_parameter指定存储过程的参数列表，列表形式如下：
[IN|OUT|INOUT] param_name type
其中**in表示输入参数，out表示输出参数，inout表示既可以输入也可以输出**；param_name表示参数名称；type表示参数的类型
该类型可以是MYSQL数据库中的任意类型

COMMENT 'string' ：注释信息，可以用来描述存储过程或函数
内容，可以用BEGIN...END来表示SQL代码的开始和结束。

```

DROP PROCEDURE IF EXISTS Proc; 

DELIMITER //
CREATE PROCEDURE Proc() 
BEGIN
  SELECT * FROM t3;
END
//
DELIMITER ;

CALL Proc(); 

```
这里的逻辑是
1、先判断是否有Proc() 这个存储过程，有就drop掉
2、创建Proc() 存储过程
3、执行Proc() 存储过程
<note>注意：“DELIMITER / /”语句的作用是将MYSQL的结束符设置为/ /，因为MYSQL默认的语句结束符为分号;，为了避免与存储过程
中SQL语句结束符相冲突，需要使用DELIMITER 改变存储过程的结束符，并以“END/ /”结束存储过程。
存储过程定义完毕之后再使用DELIMITER ;恢复默认结束符。DELIMITER 也可以指定其他符号为结束符！！！</note>

```

DELIMITER //
CREATE PROCEDURE CountProc(OUT param1 INT)
BEGIN
	SELECT COUNT(*) INTO  param1 FROM t3;
END
//
DELIMITER ;

```
上面代码的作用是创建一个获取t3表记录数的存储过程，名称是CountProc，
COUNT(*)计算后把结果放入out参数param1中。

###  存储函数 
创建存储函数，需要使用CREATE FUNCTION语句，基本语法如下：
CREATE FUNCTION func_name([func_parameter]) RETURNS TYPE
CREATE FUNCTION为用来创建存储函数的关键字；func_name表示存储函数的名称
func_parameter为存储函数的参数列表，参数列表如下
[IN|OUT|INOUT]PARAM_NAMETYPE
其中，IN表示输入参数，OUT表示输出参数，INOUT表示既可以输入也可以输出；
param_name表示参数名称；type表示参数类型，该类型可以是MYSQL数据库中的任意类型
` RETURNS TYPE语句表示函数返回数据的类型 `；
创建存储函数，名称为NameByT，该函数返回SELECT语句的查询结果，数值类型为字符串型
```

DELIMITER //

CREATE FUNCTION NameByT() RETURNS CHAR(50)
BEGIN
	RETURN (SELECT NAME FROM t3 WHERE id=2);
END
//
DELIMITER ;

```
**注意：RETURNS CHAR(50)数据类型的时候，RETURNS 是有S的，而RETURN (SELECT NAME FROM t3 WHERE id=2)的时候RETURN是没有S的**

调用函数
SELECT nameByT()

  * 指定参数为IN、OUT、INOUT只对PROCEDURE是合法的。
  * （FUNCTION中总是默认是IN参数）RETURNS子句对FUNCTION做指定，对函数而言这是强制的。
  * 他用来指定函数的返回类型，而且函数体必须包含一个RETURN value语句

###  操作存储过程与函数 
调用存储过程 call myproc();
调用函数 select myfunc();
查看定义过程和函数的代码：
show create procedure myproc;
show create function myfunc;
删除存储过程和函数:
drop procedure myproc;
drop function myfun;

###  存储过程与函数语法(局部变量、控制结构、游标、continue和handler等) 
1、变量
变量可以在子程序中声明并使用，这些变量的作用范围是在BEGIN...END程序中
**定义变量**
DECLARE var_name[,varname]...date_type[DEFAULT VALUE];
var_name为局部变量的名称。DEFAULT VALUE子句给变量提供一个默认值。值除了可以被声明为一个常数外，还可以被指定为一个表达式。如果没有DEFAULT子句，初始值为NULL
` DECLARE myvar INT DEFAULT 100; `
**为变量赋值**
MYSQL中使用SET语句为变量赋值
SET var_name=expr[,var_name=expr]...
set myvar=10
声明3个变量，分别为var1，var2和var3
```

DECLARE var1,var2,var3 INT;
SET var1=10,var2=20;
SET var3=var1+var2;

```
MYSQL中还可以**通过SELECT...INTO为一个或多个变量赋值**
```

DECLARE NAME CHAR(50);
DECLARE id DECIMAL(8,2);
SELECT id,NAME INTO id ,NAME FROM t3 WHERE id=2;

```

定义条件和处理程序
特定条件需要特定处理。
**这些条件可以联系到错误，以及子程序中的一般流程控制。定义条件是事先定义程序执行过程中遇到的问题，**
**处理程序定义了在遇到这些问题时候应当采取的处理方式，并且保证存储过程或函数在遇到警告或错误时能继续执行。**
这样可以增强存储程序处理问题的能力，避免程序异常停止运行

2、定义条件
DECLARE condition_name CONDITION FOR[condition_type]
[condition_type]:
SQLSTATE[VALUE] sqlstate_value |mysql_error_code
condition_name：表示条件名称
condition_type：表示条件的类型
sqlstate_value和mysql_error_code都可以表示mysql错误
sqlstate_value为长度5的字符串错误代码
mysql_error_code为数值类型错误代码，例如：ERROR1142(42000)中，sqlstate_value的值是42000，
mysql_error_code的值是1142
这个语句指定需要特殊处理条件。他将一个名字和指定的错误条件关联起来。
这个名字随后被用在定义处理程序的DECLARE HANDLER语句中
定义ERROR1148(42000)错误，名称为command_not_allowed。
可以用两种方法定义
```

//方法一：使用sqlstate_value
DECLARE command_not_allowed CONDITION FOR SQLSTATE '42000'
//方法二：使用mysql_error_code
DECLARE command_not_allowed CONDITION FOR SQLSTATE 1148

```
关于sqlstate异常编码查询请看[[mysql:mysql高级编程2]]

3．定义处理程序
MySQL中可以使用DECLARE关键字来定义处理程序。其基本语法如下：
DECLARE handler_type HANDLER FOR 
condition_value[,...] sp_statement 
handler_type: 
CONTINUE | EXIT | UNDO 
condition_value: 
SQLSTATE [VALUE] sqlstate_value |
condition_name | SQLWARNING |NOT FOUND |SQLEXCEPTION |mysql_error_code 
其中，handler_type参数指明错误的处理方式，该参数有3个取值。**这3个取值分别是CONTINUE、EXIT和UNDO。**
  * CONTINUE表示遇到错误不进行处理，继续向下执行；
  * EXIT表示遇到错误后马上退出；
  * UNDO表示遇到错误后撤回之前的操作，MySQL中暂时还不支持这种处理方式。
注意：通常情况下，执行过程中遇到错误应该立刻停止执行下面的语句，并且撤回前面的操作。
但是，MySQL中现在还不能支持UNDO操作。
因此，遇到错误时最好执行EXIT操作。如果事先能够预测错误类型，并且进行相应的处理，那么可以执行CONTINUE操作。
condition_value参数指明错误类型，该参数有6个取值。
sqlstate_value和mysql_error_code与条件定义中的是同一个意思。
condition_name是DECLARE定义的条件名称。
SQLWARNING表示所有以01开头的sqlstate_value值。
NOT FOUND表示所有以02开头的sqlstate_value值。
SQLEXCEPTION表示所有没有被SQLWARNING或NOT FOUND捕获的sqlstate_value值。
sp_statement表示一些存储过程或函数的执行语句。
下面是定义处理程序的几种方式。代码如下：
```

//方法一：捕获sqlstate_value 
DECLARE CONTINUE HANDLER FOR SQLSTATE '42000' set myvar=1
SET @info='CAN NOT FIND'; 
//方法二：捕获mysql_error_code 
DECLARE CONTINUE HANDLER FOR 1148 SET @info='CAN NOT FIND'; 
//方法三：先定义条件，然后调用 
DECLARE can_not_find CONDITION FOR 1146 ; 
DECLARE CONTINUE HANDLER FOR can_not_find SET @info='CAN NOT FIND'; 
//方法四：使用SQLWARNING 
DECLARE EXIT HANDLER FOR SQLWARNING SET @info='ERROR'; 
//方法五：使用NOT FOUND 
DECLARE EXIT HANDLER FOR NOT FOUND SET @info='CAN NOT FIND'; 
//方法六：使用SQLEXCEPTION 
DECLARE EXIT HANDLER FOR SQLEXCEPTION SET @info='ERROR';

```
上述代码是6种定义处理程序的方法。
第一种方法是捕获sqlstate_value值。如果遇到sqlstate_value值为42000，执行CONTINUE操作，并且输出"CAN NOT FIND"信息。
第二种方法是捕获mysql_error_code值。如果遇到mysql_error_code值为1148，执行CONTINUE操作，并且输出"CAN NOT FIND"信息。
第三种方法是先定义条件，然后再调用条件。这里先定义can_not_find条件，遇到1148错误就执行CONTINUE操作。
第四种方法是使用SQLWARNING。SQLWARNING捕获所有以01开头的sqlstate_value值，然后执行EXIT操作，并且输出"ERROR"信息。
第五种方法是使用NOT FOUND。NOT FOUND捕获所有以02开头的sqlstate_value值，然后执行EXIT操作，并且输出"CAN NOT FIND"信息。
第六种方法是使用SQLEXCEPTION。SQLEXCEPTION捕获所有没有被SQLWARNING或NOT FOUND捕获的sqlstate_value值，然后执行EXIT操作，并且输出"ERROR"信息


4、定义条件和处理程序
```

# Procedure to find the orderid with the largest amount
# could be done with max, but just to illustrate stored procedure principles

delimiter //

create procedure largest_order(out largest_id int)
begin
  declare this_id int;
  declare this_amount float;
  declare l_amount float default 0.0;
  declare l_id int;

  declare done int default 0; #声明一个循环控制变量
  /**声明条件和处理程序，条件为sqlstate '02000'，处理程序为continue handler,条件发生之后执行命令为 set done = 1;
  02000这个sqlstate异常编码代表(ER_SP_FETCH_NO_DATA) 消息：FETCH无数据。我们使用它来间接设置循环标志，使得在游标到达最后时设置循环标志来结束循环。
  */
  declare continue handler for sqlstate '02000' set done = 1;
  /**
  声明一个游标curosr，名字为cl，游标数据来源select orderid, amount from orders;但是此时并不会执行查询，只有调用open cl的时候才会执行查询。
  */
  declare c1 cursor for select orderid, amount from orders;

  open c1; #执行查询
  repeat #采用repeat循环，还有while-do循环，loop循环，但是没有for循环
    fetch c1 into this_id, this_amount; #要获取每一个数据行，必须运行一个fetch语句。并把一个行的值赋值给多个变量
    if not done then #if-then条件判断，还有if-then-elseif-then-else和case-when-then-when-then-else
      if this_amount > l_amount then
        set l_amount=this_amount;
        set l_id=this_id;
      end if;
    end if;
   until done end repeat; #循环结束条件,while循环，loop循环不能设置循环结束条件，只能通过leave语句退出循环
  
  close c1; #记得关闭游标
  set largest_id=l_id; #最后将变量值赋值给out变量largest_id,需要注意，这个out参数不能作为中间临时变量使用，只能用来保存最终值。

end
//

delimiter ;#恢复分号


```


###  流程控制的使用 
存储过程和函数中可以使用流程控制来控制语句的执行。
MySQL中可以使用IF语句、CASE语句、LOOP语句、LEAVE语句、ITERATE语句、REPEAT语句和WHILE语句来进行流程控制。
每个流程中可能包含一个单独语句，或者是使用BEGIN...END构造的复合语句，构造可以被嵌套
**1．IF语句**
IF语句用来进行条件判断。根据是否满足条件，将执行不同的语句。其语法的基本形式如下：
IF search_condition THEN statement_list 
[ELSEIF search_condition THEN statement_list] ... 
[ELSE statement_list] 
END IF 
注意：MYSQL还有一个IF()函数，他不同于这里描述的IF语句
下面是一个IF语句的示例。代码如下：
```

IF age>20 THEN SET @count1=@count1+1;  
ELSEIF age=20 THEN SET @count2=@count2+1;  
ELSE SET @count3=@count3+1;  
END IF;

``` 
**2．CASE语句**
CASE语句也用来进行条件判断，其可以实现比IF语句更复杂的条件判断。CASE语句的基本形式如下：
CASE case_value 
WHEN when_value THEN statement_list 
[WHEN when_value THEN statement_list] ... 
[ELSE statement_list] 
END CASE 
其中，case_value参数表示条件判断的变量；
when_value参数表示变量的取值；
statement_list参数表示不同when_value值的执行语句。
CASE语句还有另一种形式。该形式的语法如下：
CASE 
WHEN search_condition THEN statement_list 
[WHEN search_condition THEN statement_list] ... 
[ELSE statement_list] 
END CASE 
其中，search_condition参数表示条件判断语句；
statement_list参数表示不同条件的执行语句。
下面是一个CASE语句的示例。代码如下：
```

CASE age 
WHEN 20 THEN SET @count1=@count1+1; 
ELSE SET @count2=@count2+1; 
END CASE ; 
代码也可以是下面的形式：
CASE 
WHEN age=20 THEN SET @count1=@count1+1; 
ELSE SET @count2=@count2+1; 
END CASE ;

``` 
本示例中，如果age值为20，count1的值加1；否则count2的值加1。CASE语句都要使用END CASE结束。
注意：这里的CASE语句和“控制流程函数”里描述的SQL CASE表达式的CASE语句有轻微不同。这里的CASE语句不能有ELSE NULL子句

**3．LOOP语句**
LOOP语句可以使某些特定的语句重复执行，实现一个简单的循环。
但是LOOP语句本身没有停止循环的语句，必须是遇到LEAVE语句等才能停止循环。
LOOP语句的语法的基本形式如下：
[begin_label:] LOOP 
statement_list 
END LOOP [end_label] 
其中，begin_label参数和end_label参数分别表示循环开始和结束的标志，这两个标志必须相同，而且都可以省略；
statement_list参数表示需要循环执行的语句。
下面是一个LOOP语句的示例。代码如下：
add_num: LOOP  
SET @count=@count+1;  
END LOOP add_num ; 
该示例循环执行count加1的操作。因为没有跳出循环的语句，这个循环成了一个死循环。
LOOP循环都以END LOOP结束。

**4．LEAVE语句**
LEAVE语句主要用于**跳出循环控制**。其语法形式如下：
LEAVE label 
其中，**label参数表示循环的标志。**
下面是一个LEAVE语句的示例。代码如下：
```

add_num: LOOP 
SET @count=@count+1; 
IF @count=100 THEN 
	LEAVE add_num ; 
END LOOP add_num ;

``` 
该示例循环执行count加1的操作。当count的值等于100时，则LEAVE语句跳出循环。

**5．ITERATE语句**
ITERATE语句也是用来跳出循环的语句。但是，**ITERATE语句是跳出本次循环，然后直接进入下一次循环。**
ITERATE语句只可以出现在LOOP、REPEAT、WHILE语句内。
ITERATE语句的基本语法形式如下：
ITERATE label 
其中，label参数表示循环的标志。
下面是一个ITERATE语句的示例。代码如下：
```

add_num: LOOP 
SET @count=@count+1; 
IF @count=100 THEN 
	LEAVE add_num ; 
ELSE IF MOD(@count,3)=0 THEN 
	ITERATE add_num; 
SELECT * FROM employee ; 
END LOOP add_num ;

``` 
该示例循环执行count加1的操作，count值为100时结束循环。如果count的值能够整除3，则跳出本次循环，不再执行下面的SELECT语句。
说明：LEAVE语句和ITERATE语句都用来跳出循环语句，但两者的功能是不一样的。
LEAVE语句是跳出整个循环，然后执行循环后面的程序。而ITERATE语句是跳出本次循环，然后进入下一次循环。
使用这两个语句时一定要区分清楚。

**6．REPEAT语句**
REPEAT语句是有条件控制的循环语句。**当满足特定条件时，就会跳出循环语句**。REPEAT语句的基本语法形式如下：
[begin_label:] REPEAT 
statement_list 
UNTIL search_condition 
END REPEAT [end_label] 
其中，statement_list参数表示循环的执行语句；search_condition参数表示结束循环的条件，满足该条件时循环结束。
下面是一个ITERATE语句的示例。代码如下：
```

REPEAT 
SET @count=@count+1; 
UNTIL @count=100 END REPEAT ;

``` 
该示例循环执行count加1的操作，count值为100时结束循环。
REPEAT循环都用END REPEAT结束。

**7．WHILE语句**
WHILE语句也是有条件控制的循环语句。但WHILE语句和REPEAT语句是不一样的。
**WHILE语句是当满足条件时，执行循环内的语句。**
WHILE语句的基本语法形式如下：
[begin_label:] WHILE search_condition DO 
statement_list 
END WHILE [end_label] 
其中，search_condition参数表示循环执行的条件，满足该条件时循环执行；
statement_list参数表示循环的执行语句。
下面是一个ITERATE语句的示例。代码如下：
```

WHILE @count<100 DO 
SET @count=@count+1; 
END WHILE ;

``` 
该示例循环执行count加1的操作，count值小于100时执行循环。
如果count值等于100了，则跳出循环。WHILE循环需要使用END WHILE来结束。

