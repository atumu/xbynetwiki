title: oracle动态sql学习 

#  Oracle动态SQL学习 
1．静态SQLSQL与动态SQL
Oracle编译PL/SQL程序块分为两个种：其一为前期联编（early binding），即SQL语句在程序编译期间就已经确定，大多数的编译情况属于这种类型；
另外一种是后期联编（late binding），即SQL语句只有在运行阶段才能建立，例如当查询条件为用户输入时，那么Oracle的SQL引擎就无法在编译期对该程序语句进行确定，只能在用户输入一定的查询条件后才能提交给SQL引擎进行处理。通常，静态SQL采用前一种编译方式，而动态SQL采用后一种编译方式。

##  使用Execute immediate语句 

2．动态SQL程序开发
Oracle中提供了Execute immediate语句来执行动态SQL，语法如下：
` Excute immediate 动态SQL语句 using 绑定参数列表 returning into 输出参数列表; `
对这一语句作如下说明：
　　1)动态SQL是指DDL和不确定的DML（即带参数的DML）
　　2)绑定参数列表为输入参数列表，即其类型为in类型，在运行时刻与动态SQL语句中的参数（实际上占位符，可以理解为函数里面的形式参数）进行绑定。
　　3)输出参数列表为动态SQL语句执行后返回的参数列表。
　　4)由于动态SQL是在运行时刻进行确定的，所以相对于静态而言，其更多的会损失一些系统性能来换取其灵活性。

 为了更好的说明其开发的过程，下面列举一个实例：
设数据库的emp表，其数据为如下：
![](/data/dokuwiki/oracle/pasted/20160923-140944.png)
要求：
　　1．创建该表并输入相应的数据。
　　2．根据特定ID可以查询到其姓名和薪水的信息。
　　3．根据大于特定的薪水的查询相应的员工信息。
　　根据前面的要求，可以分别创建三个过程（均使用动态SQL）来实现：
 过程一：
```

CREATE OR REPLACE PROCEDURE CREATE_TABLE AS
BEGIN
  EXECUTE IMMEDIATE '
create table emp(id number,
name varchar2(10),
salary number )'; --动态SQL为DDL语句
  INSERT INTO EMP VALUES (100, 'jacky', 5600);
  INSERT INTO EMP VALUES (101, 'rose', 3000);
  INSERT INTO EMP VALUES (102, 'john', 4500);
END CREATE_TABLE;


```

过程二：
```

create or replace procedure find_info(p_id number) as
v_name varchar2(10);
v_salary number;
begin
execute immediate '
select name,salary from emp
where id=:1'
using p_id
returning into v_name,v_salary; --动态SQL为查询语句
dbms_output.put_line(v_name ||'的收入为：'||to_char(v_salary))；
exception
when others then
dbms_output.put_line('找不到相应数据')；
end find_info;

```
 过程三：
```

create or replace procedure find_emp(p_salary number) as
r_emp emp%rowtype;
type c_type is ref cursor;
c1 c_type;
begin
open c1 for '
select * from emp
where salary >:1'
using p_salary;
loop
fetch c1 into r_emp;
exit when c1%notfound;
dbms_output.put_line('薪水大于‘||to_char(p_salary)||’的员工为：‘);
dbms_output.put_line('ID为'to_char(r_emp)||' 其姓名为：'||r_emp.name);
end loop;
close c1;
end create_table;

```
注意：在过程二中的动态SQL语句使用了占位符“:1“，其实它相当于函数的形式参数，使用”：“作为前缀，然后使用using语句将p_id在运行时刻将:1给替换掉，这里p_id相当于函数里的实参。另外过程三中打开的游标为动态游标，它也属于动态SQL的范畴，其整个编译和开发的过程与execute immediate执行的过程很类似，这里就不在赘述了。

3． 动态SQL语句开发技巧
前面分析到了，动态SQL的执行是以损失系统性能来换取其灵活性的，所以对它进行一定程度的优化也是必要的，笔者根据实际开发经验给出一些开发的技巧，需要指出的是，这里很多经验不仅局限于动态SQL，有些也适用于静态SQL，在描述中会给予标注。

技巧一：尽量使用类似的SQL语句，这样Oracle本身通过SGA中的共享池来直接对该SQL语句进行缓存，那么在下一次执行类似语句时就直接调用缓存中已解析过的语句，以此来提高执行效率。
技巧二：当涉及到集合单元的时候，尽量使用批联编。比如需要对id为100和101的员工的薪水加薪10％，一般情况下应该为如下形式：

```

declare
type num_list is varray(20) of number;
v_id num_list :=num_list(100,101);
begin
...
for i in v_id.first .. v_id.last loop
...
execute immediate 'update emp
set =salary*1.2
where id=:1 '
using v_id(i);
end loop;
end;

```

对于上面的处理，当数据量大的时候就会显得比较慢，那么如果采用批联编的话，则整个集合首先一次性的传入到SQL引擎中进行处理，这样比单独处理效率要高的多，进行批联编处理的代码如下：

```

declare
type num_list is varray(20) of number;
v_id num_list :=num_list(100,101);
begin
...
forall i in v_id.first .. v_id.last loop
...
execute immediate 'update emp
set =salary*1.2
where id=:1 '
using v_id(i);
end loop;
end;

```

这里是使用**forall**来进行**批联编**，这里将批联编处理的情形作一个小结：
　　1) 如果一个循环内执行了insert，delete，update等语句引用了集合元素，那么可以将其移动到一个forall语句中。
　　2) 如果select into，fetch into 或returning into 子句引用了一个集合，应该使用bulk collect 子句进行合并。
　　3) 如有可能，应该使用主机数组来实现在程序和数据库服务器之间传递参数。

　　技巧三：使用NOCOPY编译器来提高PL/SQL性能。缺省情况下，out类型和in out类型的参数是由值传递的方式进行的。但是对于大的对象类型或者集合类型的参数传递而言，其希望损耗将是很大的，为了减少损耗，可以采用引用传递的方式，即在进行参数声明的时候引用NOCOPY关键字来说明即可到达这样的效果。比如创建一个过程：
```

create or replace procedure test(p_object in nocopy square)
...
end;

```
其中square为一个大的对象类型。这样只是传递一个地址，而不是传递整个对象了。显然这样的处理也是提高了效率。

　　4． 小结
　　本文对动态SQL的编译原理、开发过程以及开发技巧的讨论，通过本文的介绍后，相信读者对动态SQL程序开发有了一个总体的认识，为今后深入的工作打下一个良好的基础。



##  二、使用DBMS_SQL包 
使用DBMS_SQL包实现动态SQL的步骤如下：
A、先将要执行的SQL语句或一个语句块放到一个字符串变量中。
B、使用DBMS_SQL包的parse过程来分析该字符串。
C、使用DBMS_SQL包的bind_variable过程来绑定变量。
D、使用DBMS_SQL包的execute函数来执行语句。

**1、使用DBMS_SQL包执行DDL语句**
需求：使用DBMS_SQL包根据用户输入的表名、字段名及字段类型建表。

```

create or replace procedure proc_dbms_sql
(
    table_name in varchar2,       --表名
    field_name1 in varchar2,      --字段名
    datatype1 in varchar2,        --字段类型
    field_name2 in varchar2,      --字段名
    datatype2 in varchar2         --字段类型
)as
    v_cursor number;              --定义光标
    v_string varchar2(200);       --定义字符串变量
    v_row number;                 --行数
begin
    v_cursor:=dbms_sql.open_cursor;      --为处理打开光标
    v_string:='create table'||' '||table_name||'('||field_name1||' '||datatype1||','||field_name2||' '||datatype2||')';
    dbms_sql.parse(v_cursor,v_string,dbms_sql.native);    --分析语句
    v_row:=dbms_sql.execute(v_cursor);   --执行语句,动态SQL执行DDL时可以不写
    dbms_sql.close_cursor(v_cursor);     --关闭光标
    exception when others then
            dbms_sql.close_cursor(v_cursor);  --关闭光标
            raise;
end;

```

** 2、使用DBMS_SQL包执行DML语句**
需求：使用DBMS_SQL包根据用户输入的值更新表中相对应的记录。
建存储过程，并编译通过：
```

create or replace procedure proc_dbms_sql_update
(
    id number,
    name varchar2
)as
    v_cursor number;            --定义光标
    v_string varchar2(200);   --字符串变量
    v_row number;               --行数
begin
    v_cursor:=dbms_sql.open_cursor;    --为处理打开光标
    v_string:='update dinya_test2 a set a.name=:p_name where a.id=:p_id';
    dbms_sql.parse(v_cursor,v_string,dbms_sql.native);     --分析语句
    dbms_sql.bind_variable(v_cursor,':p_name',name);       --绑定变量
    dbms_sql.bind_variable(v_cursor,':p_id',id);                 --绑定变量
    v_row:=dbms_sql.execute(v_cursor);　　　　　       --执行动态SQL
    dbms_sql.close_cursor(v_cursor);                                --关闭光标
    exception when others then
            dbms_sql.close_cursor(v_cursor);                        --关闭光标
            raise;
end;

```
使用DBMS_SQL中，如果要执行的动态语句不是查询语句，使用DBMS_SQL.Execute或DBMS_SQL.Variable_Value来执行，如果要执行动态语句是查询语句，则要使用DBMS_SQL.define_column定义输出变量，然后使用DBMS_SQL.Execute, DBMS_SQL.Fetch_Rows, DBMS_SQL.Column_Value及DBMS_SQL.Variable_Value来执行查询并得到结果。 

http://www.cnblogs.com/gaolonglong/archive/2011/05/31/2064790.html
http://www.blogjava.net/cheneyfree/archive/2007/12/17/168272.html