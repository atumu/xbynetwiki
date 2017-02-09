title: pl_sql学习一 

#  PL/SQL学习一 
PL/SQL 是Procedure Language & Structured Query Language 的缩写。ORACLE的SQL 是支持ANSI(American national Standards Institute)和ISO92 (International
Standards Organization)标准的产品。PL/SQL 是对SQL 语言存储过程语言的扩展。
##  PL/SQL 的优点或特征 
1、过程化
PL/SQL 是Oracle 在标准SQL 上的过程性扩展，不仅允许在PL/SQL 程序内嵌入SQL 语句，而且允许使用各种类型的条件分支语句和循环语句。
2、模块化
PL/SQL 程序结构是一种描述性很强、界限分明的块结构、嵌套块结构，**被分成单独的过程、函数、触发器，且可以把它们组合为程序包**，提高程序的模块化能力。
3、运行错误的可处理性
使用PL/SQL 提供的异常处理（EXCEPTION），开发人员可集中处理各种ORACLE 错误和PL/SQL 错误，或处理系统错误与自定义错误，以增强应用程序的健壮性。
4、提供大量内置程序包
ORACLE 提供了大量的内置程序包。通过这些程序包能够实现DBS 的一些低层操作、高级功能，不论对DBA 还是应用开发人员都具有重要作用。

PL/SQL 可用的SQL 语句
在PL/SQL 中可以使用的SQL 语句有：
INSERT，UPDATE，DELETE，SELECT INTO，COMMIT，ROLLBACK，SAVEPOINT。
` 提示：在PL/SQL 中只能用SQL 语句中的DML 部分，不能用DDL 部分，如果要在PL/SQL 中使用DDL(如CREATE table 等)的话，只能以动态的方式来使用。 `

##  PL/SQL 块 
PL/SQL 程序由三个块组成，即声明部分、执行部分、异常处理部分。
PL/SQL块的结构如下：
```

DECLARE
--声明部分: 在此声明PL/SQL 用到的变量,类型及游标，以及局部的存储过程和函数
BEGIN
-- 执行部分: 过程及SQL 语句, 即程序的主要部分
EXCEPTION
-- 执行异常部分: 错误处理
END;

```
标识符：**提示: 一般不要把变量名声明与表中字段名完全一样,如果这样可能得到不正确的结果.**
例如：下面的例子将会删除所有的纪录，而不是’EricHu’的记录；
```

DECLARE
ename varchar2(20) :='EricHu';
BEGIN
DELETE FROM scott.emp WHERE ename=ename;
END;

```
PL/SQL 变量类型
char, varchar2,BINARY_INTEGER,NUMBER(p,s),LONG,Date,BOOLEAN,ROWID,UROWID

例1. 插入一条记录并显示；
```

DECLARE
Row_id ROWID;
info VARCHAR2(40);
BEGIN
INSERT INTO scott.dept VALUES (90, '财务室', '海口')
RETURNING rowid, dname||':'||to_char(deptno)||':'||loc
INTO row_id, info;
DBMS_OUTPUT.PUT_LINE('ROWID:'||row_id);
DBMS_OUTPUT.PUT_LINE(info);
END;

```
RETURNING 子句用于检索INSERT 语句中所影响的数据行数，
当INSERT 语句使用VALUES 子句插入数据时，RETURNING 字句还可将列表达式、ROWID 和REF 值返回到输出变量中。
在使用RETURNING 子句是应注意以下几点限制：
1．不能与DML 语句和远程对象一起使用；
2．不能检索LONG 类型信息；
3．当通过视图向基表中插入数据时，只能与单基表视图一起使用。

##  复合类型 
ORACLE 在PL/SQL 中除了提供象前面介绍的各种类型外,还提供一种**称为复合类型的类型---记录和表.**
###  记录类型 
记录类型类似于C 语言中的结构数据类型，它把逻辑相关的、分离的、基本数据类型的变量组成一个整体存储起来，它必须包括至少一个标量型或RECORD 数据类型的成员，称作**PL/SQL RECORD** 的域(FIELD)
```

TYPE record_name IS RECORD(
v1 data_type1 [NOT NULL] [:= default_value ],
v2 data_type2 [NOT NULL] [:= default_value ],
......
vn data_typen [NOT NULL] [:= default_value ] );

```
```

DECLARE
TYPE test_rec IS RECORD(
Name VARCHAR2(30) NOT NULL := '胡勇',
Info VARCHAR2(100));
rec_book test_rec;
BEGIN
rec_book.Name :='胡勇';
rec_book.Info :='谈PL/SQL 编程;';
DBMS_OUTPUT.PUT_LINE(rec_book.Name||' ' ||rec_book.Info);
END;

```
**可以用SELECT 语句对记录变量进行赋值**,只要保证记录字段与查询结果列表中的字段相配即可。
```

DECLARE
--定义与hr.employees 表中的这几个列相同的记录数据类型
TYPE RECORD_TYPE_EMPLOYEES IS RECORD(
f_name hr.employees.first_name%TYPE,
h_date hr.employees.hire_date%TYPE,
j_id hr.employees.job_id%TYPE);
--声明一个该记录数据类型的记录变量
v_emp_record RECORD_TYPE_EMPLOYEES;
BEGIN
SELECT first_name, hire_date, job_id INTO v_emp_record
FROM employees
WHERE employee_id = &emp_id;
DBMS_OUTPUT.PUT_LINE('雇员名称：'||v_emp_record.f_name
||' 雇佣日期：'||v_emp_record.h_date
||' 岗位：'||v_emp_record.j_id);
END;

```
` 一个记录类型的变量只能保存从数据库中查询出的一行记录，若查询出了多行记录，就会出现错误。 `
###  数组类型 
数据是具有相同数据类型的一组成员的集合。每个成员都有一个唯一的下标，它取决于成员在数组中的位置。在PL/SQL 中，数组数据类型是**VARRAY**。
` 注意：PL/SQL中数组下标从1开始 `
定义VARRY 数据类型语法如下：
```

TYPE varray_name IS VARRAY(size) OF element_type [NOT NULL];

```
```

DECLARE
--定义一个最多保存5 个VARCHAR(25)数据类型成员的VARRAY 数据类型
TYPE reg_varray_type IS VARRAY(5) OF VARCHAR(25);
--声明一个该VARRAY 数据类型的变量
v_reg_varray REG_VARRAY_TYPE;
BEGIN
--用构造函数语法赋予初值
v_reg_varray := reg_varray_type
('中国', '美国', '英国', '日本', '法国');
DBMS_OUTPUT.PUT_LINE('地区名称：'||v_reg_varray(1)||'、'
||v_reg_varray(2)||'、'
||v_reg_varray(3)||'、'
||v_reg_varray(4));
DBMS_OUTPUT.PUT_LINE(' 赋予初值NULL 的第5 个成员的值：
'||v_reg_varray(5));
--用构造函数语法赋予初值后就可以这样对成员赋值
v_reg_varray(5) := '法国';
DBMS_OUTPUT.PUT_LINE('第5 个成员的值：'||v_reg_varray(5));
END;

```
###  使用%TYPE和%ROWTYPE 
定义一个变量，其数据类型与已经定义的某个数据变量(尤其是表的某一列)的数据类型相一致，这时可以**使用%TYPE**。
```

DECLARE
-- 用%TYPE 类型定义与表相配的字段
TYPE T_Record IS RECORD(
T_no emp.empno%TYPE,
T_name emp.ename%TYPE,
T_sal emp.sal%TYPE );
-- 声明接收数据的变量
v_emp T_Record;
BEGIN
SELECT empno, ename, sal INTO v_emp FROM emp WHERE empno=7788;
DBMS_OUTPUT.PUT_LINE
(TO_CHAR(v_emp.t_no)||' '||v_emp.t_name||' ' || TO_CHAR(v_em
p.t_sal));
END;

```

**使用%ROWTYPE**
PL/SQL 提供%ROWTYPE 操作符, **返回一个记录类型**, 其数据类型和数据库表的数据结构相一致。
```

DECLARE
v_empno emp.empno%TYPE :=&no;
rec emp%ROWTYPE;
BEGIN
SELECT * INTO rec FROM emp WHERE empno=v_empno;
DBMS_OUTPUT.PUT_LINE('姓名:'||rec.ename||'工资:'||rec.sal||'工
作时间:'||rec.hiredate);
END;

```

###  LOB 类型 
ORACLE 提供了LOB (Large OBject)类型，用于存储大的数据对象的类型。ORACLE目前主要支持**BFILE（Movie）, BLOB(PHOTO), CLOB(BOOK) 及NCLOB** 类型。
###  BIND 变量 
**绑定变量是在主机环境中定义的变量。**在PL/SQL 程序中可以使用绑定变量作为他们将要使用的其它变量。为了在PL/SQL 环境中声明绑定变量，使用命令**VARIABLE**。
```

VARIABLE result NUMBER;
BEGIN
SELECT (sal*10)+nvl(comm, 0) INTO :result FROM emp
WHERE empno=7844;
END;
--然后再执行
PRINT result

```
###  PL/SQL 表(TABLE)类型 
定义记录表（或索引表）数据类型。它与记录类型相似，但它是对记录类型的扩展。**它可以处理多行记录，类似于高级中的二维数组，使得可以在PL/SQL 中模仿数据库中的表。**
```

TYPE table_name IS TABLE OF element_type [NOT NULL]
INDEX BY [BINARY_INTEGER | PLS_INTEGER | VARRAY2];

```
关键字INDEX BY 表示创建一个主键索引，以便引用记录表变量中的特定行。
方法描述
  * EXISTS(n) 如果集合的第n 个成员存在，则返回true
  * COUNT 返回已经分配了存储空间即赋值了的成员数量
  * FIRST：返回成员的最低下标值
  * LAST： 返回成员的最高下标值
  * PRIOR(n) 返回下标为n 的成员的前一个成员的下标。如果没有则返回NULL
  * NEXT(N) 返回下标为n 的成员的后一个成员的下标。如果没有则返回NULL
  * TRIM TRIM：删除末尾一个成员
  * TRIM(n) ：删除末尾n 个成员
  * DELETE DELETE：删除所有成员
  * DELETE(n) ：删除第n 个成员
  * DELETE(m, n) ：删除从n 到m 的成员
  * EXTEND EXTEND：添加一个null 成员
  * EXTEND(n)：添加n 个null 成员
  * EXTEND(n,i)：添加n 个成员，其值与第i 个成员相同
  * LIMIT 返回在varray 类型变量中出现的最高下标值
```

DECLARE
TYPE dept_table_type IS TABLE OF
dept%ROWTYPE INDEX BY BINARY_INTEGER;
my_dname_table dept_table_type;
v_count number(2) :=4;
BEGIN
FOR int IN 1 .. v_count LOOP
SELECT * INTO my_dname_table(int) FROM dept WHERE deptno=int*
10; END LOOP;
FOR int IN my_dname_table.FIRST .. my_dname_table.LAST LOOP
DBMS_OUTPUT.PUT_LINE('Department number: '||my_dname_table(int)
.deptno);
DBMS_OUTPUT.PUT_LINE('Department name: '|| my_dname_table(int).
dname);
END LOOP;
END;

```

##  变量操作 

###  运算符 
```

= 等于
<> , != , ~= , ^= 不等于
:= 赋值号
=> 关系号
.. 范围运算符
|| 字符连接符

IS NULL 是空值
BETWEEN AND 介于两者之间
IN 在一列值中间
AND 逻辑与
OR 逻辑或
NOT 取返,如IS NOT NULL,


```
  * 空值加数字仍是空值：NULL + < 数字> = NULL
  * 空值加（连接）字符，结果为字符：NULL || <字符串> = < 字符串>
  * 布尔值只有TRUE, FALSE 及NULL 三个值。
###  变量赋值 
**数据库赋值**是通过**SELECT 语句**来完成的，每次执行SELECT 语句就赋值一次，一般要求被赋值的变量与SELECT 中的列名要一一对应。如：
```

DECLARE
emp_id emp.empno%TYPE :=7788;
emp_name emp.ename%TYPE;
wages emp.sal%TYPE;
BEGIN
SELECT ename, NVL(sal,0) + NVL(comm,0) INTO emp_name, wages
FROM emp WHERE empno = emp_id;
DBMS_OUTPUT.PUT_LINE(emp_name||'----'||to_char(wages));
END;

```
` 提示：不能将SELECT 语句中的列赋值给布尔变量。 `

**可转换的类型赋值**
 CHAR 转换为NUMBER：
v_total := TO_NUMBER('100.0') + sal;
NUMBER 转换为CHAR：
v_comm := TO_CHAR('123.45') || '元' ;
字符转换为日期：
v_date := TO_DATE('2001.07.03','yyyy.mm.dd');
日期转换为字符
v_to_day := TO_CHAR(SYSDATE, 'yyyy.mm.dd hh24:mi:ss') ;

###  变量作用范围及可见性 
PL/SQL 的变量作用范围特点是：
 变量的作用范围是在你所引用的程序单元（块、子程序、包）内。即从
声明变量开始到该块的结束。
 一个变量（标识）只能在你所引用的块内是可见的。
 当一个变量超出了作用范围，PL/SQL 引擎就释放用来存放该变量的空
间（因为它可能不用了）。
 在子块中重新定义该变量后，它的作用仅在该块内。
###  注释 
在PL/SQL 里，可以使用两种符号来写注释，即：
 使用**双‘-‘ ( 减号)** 加注释 --
 使用**/* */** 来加一行或多行注释，


##  语句结构 
###  条件语句 
```

IF <布尔表达式> THEN
PL/SQL 和SQL 语句
END IF;
-----------------------
IF <布尔表达式> THEN
PL/SQL 和SQL 语句
ELSE
其它语句
END IF;
-----------------------
IF <布尔表达式> THEN
PL/SQL 和SQL 语句
ELSIF < 其它布尔表达式> THEN
其它语句
ELSIF < 其它布尔表达式> THEN
其它语句
ELSE
其它语句
END IF;

```
**提示: ELSIF 不能写成ELSEIF**

```

DECLARE
v_empno employees.employee_id%TYPE :=&empno;
V_salary employees.salary%TYPE;
V_comment VARCHAR2(35);
BEGIN
SELECT salary INTO v_salary FROM employees
WHERE employee_id = v_empno;
IF v_salary < 1500 THEN
V_comment:= '太少了,加点吧~!';
ELSIF v_salary <3000 THEN
V_comment:= '多了点,少点吧~!';
ELSE
V_comment:= '没有薪水~!';
END IF;
DBMS_OUTPUT.PUT_LINE(V_comment);
exception
when no_data_found then
DBMS_OUTPUT.PUT_LINE('没有数据~!');
when others then
DBMS_OUTPUT.PUT_LINE(sqlcode || '---' || sqlerrm);
END;

```

###  CASE 表达式 
```

CASE 条件表达式
WHEN 条件表达式结果1 THEN
语句段1
WHEN 条件表达式结果2 THEN
语句段2
......
WHEN 条件表达式结果n THEN
语句段n
[ELSE 条件表达式结果]
END;


DECLARE
V_grade char(1) := UPPER('&p_grade');
V_appraisal VARCHAR2(20);
BEGIN
V_appraisal :=
CASE v_grade
WHEN 'A' THEN 'Excellent'
WHEN 'B' THEN 'Very Good'
WHEN 'C' THEN 'Good'
ELSE 'No such grade'
END;
DBMS_OUTPUT.PUT_LINE('Grade:'||v_grade||' Appraisal: '|| v_appraisal);
END;

```
```

CASE
WHEN 条件表达式1 THEN
语句段1
WHEN 条件表达式2 THEN
语句段2
......
WHEN 条件表达式n THEN
语句段n
[ELSE 语句段]
END;

DECLARE
v_first_name employees.first_name%TYPE;
v_job_id employees.job_id%TYPE;
v_salary employees.salary%TYPE;
v_sal_raise NUMBER(3,2);
BEGIN
SELECT first_name, job_id, salary INTO
v_first_name, v_job_id, v_salary
FROM employees WHERE employee_id = &emp_id;
CASE
WHEN v_job_id = 'PU_CLERK' THEN
IF v_salary < 3000 THEN v_sal_raise := .08;
ELSE v_sal_raise := .07;
END IF;
WHEN v_job_id = 'SH_CLERK' THEN
IF v_salary < 4000 THEN v_sal_raise := .06;
ELSE v_sal_raise := .05;
END IF;
WHEN v_job_id = 'ST_CLERK' THEN
IF v_salary < 3500 THEN v_sal_raise := .04;
ELSE v_sal_raise := .03;
END IF;
ELSE
DBMS_OUTPUT.PUT_LINE('该岗位不涨工资: '||v_job_id);
END CASE;
DBMS_OUTPUT.PUT_LINE(v_first_name||'的岗位是'||v_job_id
||'、的工资是'||v_salary
||'、工资涨幅是'||v_sal_raise);
END;

```


###  循环 
**简单loop循环**
```

LOOP
要执行的语句;
EXIT WHEN <条件语句> --条件满足，退出循环语句
END LOOP;

```
**WHILE 循环**
```

WHILE <布尔表达式> LOOP
要执行的语句;
END LOOP;

```
**数字式循环**
```

[<<循环标签>>]
FOR 循环计数器IN [ REVERSE ] 下限.. 上限LOOP
要执行的语句;
END LOOP [循环标签];

```
每循环一次，循环变量自动加1；使用关键字REVERSE，循环变量自动减1。跟在IN REVERSE 后面的数字必须是从小到大的顺序，而且必须是整数，不能是变量或表达式。可以使用**EXIT** 退出循环。
```

BEGIN
FOR int in 1..10 LOOP
DBMS_OUTPUT.PUT_LINE('int 的当前值为: '||int);
END LOOP;
END;

```
```

BEGIN
  FOR v_counter IN REVERSE 20 .. 25 LOOP
      DBMS_OUTPUT.put_line(v_counter);
  END LOOP;
END;
输出 
25
24
23
22
21
20

```
```

DECLARE
TYPE jobids_varray IS VARRAY(12) OF VARCHAR2(10); --定义一个VARRAY 数据类型
v_jobids JOBIDS_VARRAY; --声明一个具有JOBIDS_VARRAY 数据类型的变量
v_howmany NUMBER; --声明一个变量来保存雇员的数量
BEGIN
--用某些job_id 值初始化数组
v_jobids := jobids_varray('FI_ACCOUNT', 'FI_MGR', 'ST_CLERK', 'ST_MAN');
--用FOR...LOOP...END LOOP 循环使用每个数组成员的值
FOR i IN v_jobids.FIRST..v_jobids.LAST LOOP
--针对数组中的每个岗位，决定该岗位的雇员的数量
SELECT count(*) INTO v_howmany FROM employees WHERE job_id = v_jobids(i);
DBMS_OUTPUT.PUT_LINE ( '岗位'||v_jobids(i)||
'总共有'|| TO_CHAR(v_howmany) || '个雇员');
END LOOP;
END;

```
###  标号和GOTO 
PL/SQL 中GOTO 语句是无条件跳转到指定的标号去的意思。语法如下：
GOTO label;
......
<<label>> /*标号是用<< >>括起来的标识符*/
###  NULL 语句 
**在PL/SQL 程序中，NULL 语句是一个可执行语句**，可以用null 语句来说明“不用做任何事情”的意思，相当于一个占位符或不执行任何操作的空语句，可以使某些语句变得有意义，提高程序的可读性，保证其他语句结构的完整性和正确性。
```

DECLARE
...
BEGIN
...
IF v_num IS NULL THEN
GOTO labelPrint;
END IF;
…
<<labelPrint>>
NULL; --不需要处理任何数据。
END;

```
