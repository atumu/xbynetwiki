title: pl_sql学习之游标_异常 

#  PL/SQL学习之游标、异常 
##  游标概念 
在PL/SQL 块中执行SELECT、INSERT、DELETE 和UPDATE 语句时，ORACLE 会在内存中为其分配上下文区（Context Area），即缓冲区。**游标是指向该区的一个指针**，或
是命名一个工作区（Work Area），或是一种结构化数据类型。它为应用等量齐观提供了一种对具有多行数据查询结果集中的每一行数据分别进行单独处理的方法，是设计嵌入式SQL 语句的应用程序的常用编程方式。**在每个用户会话中，可以同时打开多个游标，其数量由数据库初始化参数文件中的OPEN_CURSORS 参数定义**
对于不同的SQL 语句，游标的使用情况不同：
SQL 				语句游标
非查询语句 			    隐式的
结果是单行的查询语句		隐式的或显示的
结果是多行的查询语句		显示的

###  显式游标处理 
显式游标处理需四个PL/SQL 步骤:
**定义/声明游标**：就是定义一个游标名，以及与其相对应的SELECT 语句。
格式：
```

CURSOR cursor_name[(parameter[, parameter]…)]
[RETURN datatype]
IS
select_statement;

```
**游标参数只能为输入参数**，其格式为：
```

parameter_name [IN] datatype [{:= | DEFAULT} expression]

```
**在指定数据类型时，不能使用长度约束。如NUMBER(4),CHAR(10) 等都是错误的。**
[RETURN datatype]是可选的，表示**游标返回数据的类型**。如果选择，则应该严格与select_statement 中的选择列表在次序和数据类型上匹配。一般是记录数据类型或带“%ROWTYPE”的数据。

**打开游标**：就是执行游标所对应的SELECT 语句，将其查询结果放入工作区，并且指针指向工作区的首部，标识游标结果集合。如果游标查询语句中带有**FOR UPDATE** 选
项，**OPEN 语句**还将锁定数据库表中游标结果集合对应的数据行。
格式：
```

OPEN cursor_name[([parameter =>] value[, [parameter =>] value]…)];

```
在向游标传递参数时，可以使用与函数参数相同的**传值方法，即位置表示法和名称表示**
法。` PL/SQL 程序不能用OPEN 语句重复打开一个游标。 `

**提取游标数据**：就是检索结果集合中的数据行，放入指定的输出变量中。
格式：
```

FETCH cursor_name INTO {variable_list | record_variable };

```
**执行FETCH 语句时，每次返回一个数据行，然后自动将游标移动指向下一个数据行**。当
检索到最后一行数据时，如果再次执行FETCH 语句，将操作失败，**并将游标属性%NOTFOUND 置为TRUE**。所以每次执行完FETCH 语句后，检查游标属性%NOTFOUND
就可以判断FETCH 语句是否执行成功并返回一个数据行，以便确定是否给对应的变量赋了值。
对该记录进行处理；
继续处理，直到活动集合中没有记录；

**关闭游标**：当提取和处理完游标结果集合数据后，应及时关闭游标，以释放该游标所占用的系统资源，并使该游标的工作区变成无效，不能再使用FETCH 语句取其中数据。
**关闭后的游标可以使用OPEN 语句重新打开**。
格式：
```

CLOSE cursor_name;

```
` 注：定义的游标不能有INTO 子句。 `

```

DECLARE
CURSOR c_cursor
IS SELECT first_name || last_name, Salary
FROM EMPLOYEES
WHERE rownum<11;
v_ename EMPLOYEES.first_name%TYPE;
v_sal EMPLOYEES.Salary%TYPE;
BEGIN
OPEN c_cursor;
FETCH c_cursor INTO v_ename, v_sal;
WHILE c_cursor%FOUND LOOP
DBMS_OUTPUT.PUT_LINE(v_ename||'---'||to_char(v_sal) );
FETCH c_cursor INTO v_ename, v_sal;
END LOOP;
CLOSE c_cursor;
END;

```
例2. 游标参数的传递方法。
```

DECLARE
DeptRec DEPARTMENTS%ROWTYPE;
Dept_name DEPARTMENTS.DEPARTMENT_NAME%TYPE;
Dept_loc DEPARTMENTS.LOCATION_ID%TYPE;
CURSOR c1 IS
SELECT DEPARTMENT_NAME, LOCATION_ID FROM DEPARTMENTS
WHERE DEPARTMENT_ID <= 30;
CURSOR c2(dept_no NUMBER DEFAULT 10) IS
SELECT DEPARTMENT_NAME, LOCATION_ID FROM DEPARTMENTS
WHERE DEPARTMENT_ID <= dept_no;
CURSOR c3(dept_no NUMBER DEFAULT 10) IS
SELECT * FROM DEPARTMENTS
WHERE DEPARTMENTS.DEPARTMENT_ID <=dept_no;
BEGIN
OPEN c1;
LOOP
FETCH c1 INTO dept_name, dept_loc;
EXIT WHEN c1%NOTFOUND;
DBMS_OUTPUT.PUT_LINE(dept_name||'---'||dept_loc);
END LOOP;
CLOSE c1;
OPEN c2;
LOOP
FETCH c2 INTO dept_name, dept_loc;
EXIT WHEN c2%NOTFOUND;
DBMS_OUTPUT.PUT_LINE(dept_name||'---'||dept_loc);
END LOOP;
CLOSE c2;
OPEN c3(dept_no =>20);
LOOP
FETCH c3 INTO deptrec;
EXIT WHEN c3%NOTFOUND;
DBMS_OUTPUT.PUT_LINE(deptrec.DEPARTMENT_ID||'---'||deptrec.DEPARTMENT_NAME||'
---'||deptrec.LOCATION_ID);
END LOOP;
CLOSE c3;
END;

```

游标属性属性作用
Cursor_name%FOUND 布尔型属性，当最近一次提取游标操作FETCH成功则为TRUE,否则为FALSE；
Cursor_name%NOTFOUND 布尔型属性，与%FOUND 相反；
Cursor_name%ISOPEN 布尔型属性，当游标已打开时返回TRUE；
Cursor_name%ROWCOUNT 数字型属性，返回已从游标中读取的记录数。

```

例3：给工资低于1200 的员工增加工资50。
DECLARE
v_empno EMPLOYEES.EMPLOYEE_ID%TYPE;
v_sal EMPLOYEES.Salary%TYPE;
CURSOR c_cursor IS SELECT EMPLOYEE_ID, Salary FROM EMPLOYEES;
BEGIN
OPEN c_cursor;
LOOP
FETCH c_cursor INTO v_empno, v_sal;
EXIT WHEN c_cursor%NOTFOUND;
IF v_sal<=1200 THEN
UPDATE EMPLOYEES SET Salary=Salary+50 WHERE EMPLOYEE_ID=v_empno;
DBMS_OUTPUT.PUT_LINE('编码为'||v_empno||'工资已更新!');
END IF;
DBMS_OUTPUT.PUT_LINE('记录数:'|| c_cursor %ROWCOUNT);
END LOOP;
CLOSE c_cursor;
END;

```
例6：有参数且有返回值的游标。

```

DECLARE
TYPE emp_record_type IS RECORD(
f_name employees.first_name%TYPE,
h_date employees.hire_date%TYPE);
v_emp_record EMP_RECORD_TYPE;
CURSOR c3(dept_id NUMBER, j_id VARCHAR2) --声明游标,有参数有返回值
RETURN EMP_RECORD_TYPE
IS
SELECT first_name, hire_date FROM employees
WHERE department_id = dept_id AND job_id = j_id;
BEGIN
OPEN c3(j_id => 'AD_VP', dept_id => 90); --打开游标,传递参数值
LOOP
FETCH c3 INTO v_emp_record; --提取游标

```
例7：基于游标定义记录变量。
```

DECLARE
CURSOR c4(dept_id NUMBER, j_id VARCHAR2) --声明游标,有参数没有返回值
IS
SELECT first_name f_name, hire_date FROM employees
WHERE department_id = dept_id AND job_id = j_id;
--基于游标定义记录变量，比声明记录类型变量要方便，不容易出错
v_emp_record c4%ROWTYPE;
BEGIN
OPEN c4(90, 'AD_VP'); --打开游标,传递参数值
LOOP
FETCH c4 INTO v_emp_record; --提取游标
IF c4%FOUND THEN
DBMS_OUTPUT.PUT_LINE(v_emp_record.f_name||'的雇佣日期是'
||v_emp_record.hire_date);
ELSE
DBMS_OUTPUT.PUT_LINE('已经处理完结果集了');
EXIT;
END IF;
END LOOP;
CLOSE c4; --关闭游标
END;

```

###  游标的FOR 循环 
**PL/SQL 语言提供了游标FOR 循环语句，自动执行游标的OPEN、FETCH、CLOSE语句和循环语句的功能；**当进入循环时，游标FOR 循环语句自动打开游标，并提取第一行游标数据，当程序处理完当前所提取的数据而进入下一次循环时，游标FOR 循环语句自动提取下一行数据供程序处理，当提取完结果集合中的所有数据行后结束循环，并自动关闭游
标。
格式：
```

FOR index_variable IN cursor_name[(value[, value]…)] LOOP
-- 游标数据处理代码
END LOOP;

```
其中：
index_variable 为游标FOR 循环语句隐含声明的索引变量，**该变量为记录变量**，其结构与游标查询语句返回的结构集合的结构相同。在程序中可以通过引用该索引记录变量元素来读
取所提取的游标数据，index_variable 中各元素的名称与游标查询语句选择列表中所制定的列名相同

```

DECLARE
CURSOR c_sal IS SELECT employee_id, first_name || last_name ename, salary
FROM employees ;
BEGIN
--隐含打开游标
FOR v_sal IN c_sal LOOP
--隐含执行一个FETCH 语句
DBMS_OUTPUT.PUT_LINE(to_char(v_sal.employee_id)||'---'|| v_sal.ename||'---'||to_char(v
_sal.salary)) ;
--隐含监测c_sal%NOTFOUND
END LOOP;
--隐含关闭游标
END;

```

例9：当所声明的游标带有参数时，通过游标FOR 循环语句为游标传递参数。
```

FOR c1_rec IN c_cursor(30) LOOP DBMS_OUTPUT.PUT_LINE(c1_rec.department_name||'--
-'||c1_rec.location_id);
END LOOP;

```
例10：PL/SQL 还**允许在游标FOR 循环语句中使用子查询来实现游标的功能**。
```

BEGIN
FOR c1_rec IN(SELECT department_name, location_id FROM departments) LOOP DBMS_O
UTPUT.PUT_LINE(c1_rec.department_name||'---'||c1_rec.location_id);
END LOOP;
END;

```
##  处理隐式游标 
**显式游标主要是用于对查询语句的处理，尤其是在查询结果为多条记录的情况下；**
**而对于非查询语句，如修改、删除操作，则由ORACLE 系统自动地为这些操作设置游标并创建其工作区，这些由系统隐含创建的游标称为隐式游标，隐式游标的名字为SQL**，这是由ORACLE 系统定义的。
对于隐式游标的操作，如定义、打开、取值及关闭操作，都由ORACLE系统自动地完成，无需用户进行处理。**用户只能通过隐式游标的相关属性**，来完成相应的操作。在隐式游标的工作区中，所存放的数据是与用户自定义的显示游标无关的、最新处理的一条SQL 语句所包含的数据。
**格式调用为： SQL%**
注：INSERT, UPDATE, DELETE, SELECT 语句中不必明确定义游标。
隐式游标属性
属性值		SELECT 	 INSERT  UPDATE  DELETE
SQL%ISOPEN   	  FALSE   FALSE   FALSE   FALSE
SQL%FOUND 	   TRUE  有结果    成功   成功
SQL%FOUND 	   FALSE  没结果    失败   失败
SQL%NOTFUOND 	    TRUE 没结果     失败   失败
SQL%NOTFOUND 		FALSE 有结果   成功   失败
SQL%ROWCOUNT      返回行数，只为1 插入的行数 修改的行数 删除的行数

删除EMPLOYEES 表中某部门的所有员工，如果该部门中已没有员工，则在DEPARTMENT
表中删除该部门。
```

DECLARE
V_deptno department_id%TYPE :=&p_deptno;
BEGIN
DELETE FROM employees WHERE department_id=v_deptno;
IF SQL%NOTFOUND THEN
DELETE FROM departments WHERE department_id=v_deptno;
END IF;
END;

```

例12: 通过隐式游标SQL 的%ROWCOUNT 属性来了解修改了多少行。
```

DECLARE
v_rows NUMBER;
BEGIN
--更新数据
UPDATE employees SET salary = 30000
WHERE department_id = 90 AND job_id = 'AD_VP';
--获取默认游标的属性值
v_rows := SQL%ROWCOUNT;
DBMS_OUTPUT.PUT_LINE('更新了'||v_rows||'个雇员的工资');
--回退更新，以便使数据库的数据保持原样
ROLLBACK;
END;

```
##  使用游标更新和删除数据 
游标修改和删除操作是指在游标定位下，修改或删除表中指定的数据行。这时，要求游标查询语句中**必须使用FOR UPDATE 选项**，以便在打开游标时锁定游标结果集合在表中对应数据行的所有列和部分列。
为了对正在处理(查询)的行不被另外的用户改动，ORACLE 提供一个FOR UPDATE子句来对所选择的行进行锁住。该需求迫使ORACLE 锁定游标结果集合的行，可以防止其他事务处理更新或删除相同的行，直到您的事务处理提交或回退为止。
语法：
```

SELECT column_list FROM table_list FOR UPDATE [OF column[, column]…] [NOWAIT]

```
如果另一个会话已对活动集中的行加了锁，那么SELECT FOR UPDATE 操作**一直等待**到其它的会话释放这些锁后才继续自己的操作，对于这种情况，当加上**NOWAIT** 子句时，如果这些行真的被另一个会话锁定，则OPEN 立即返回并给出：ORA-0054 ：resource busy and acquire with nowait specified
**如果使用FOR UPDATE 声明游标，则可在DELETE 和UPDATE 语句中使用WHERE CURRENT OF cursor_name 子句，**修改或删除游标结果集合当前行对应的数据库表中的数据行。
13：从EMPLOYEES 表中查询某部门的员工情况，将其工资最低定为1500；
```

DECLARE
V_deptno employees.department_id%TYPE :=&p_deptno;
CURSOR emp_cursor
IS
SELECT employees.employee_id, employees.salary
FROM employees WHERE employees.department_id=v_deptno
FOR UPDATE NOWAIT;
BEGIN
FOR emp_record IN emp_cursor LOOP
IF emp_record.salary < 1500 THEN
UPDATE employees SET salary=1500 WHERE CURRENT OF emp_cursor;
END IF;
END LOOP;
-- COMMIT;
END;

```
例14：将EMPLOYEES 表中部门编码为90、岗位为AD_VP 的雇员的工资都更新为2000 元；
```

DECLARE
v_emp_record employees%ROWTYPE;
CURSOR c1
IS
SELECT * FROM employees FOR UPDATE;
BEGIN
OPEN c1;
LOOP
FETCH c1 INTO v_emp_record;
EXIT WHEN c1%NOTFOUND;
IF v_emp_record.department_id = 90 AND
v_emp_record.job_id = 'AD_VP'
THEN
UPDATE employees SET salary = 20000
WHERE CURRENT OF c1; --更新当前游标行对应的数据行
END IF;
END LOOP;
COMMIT; --提交已经修改的数据
CLOSE c1;
END;

```

##  游标变量 
与游标一样，游标变量也是一个指向多行查询结果集合中当前数据行的指针。但与游标不同的是，游标变量是动态的，而游标是静态的。游标只能与指定的查询相连， 即固定指向
一个查询的内存处理区域，而游标变量则可与不同的查询语句相连，它可以指向不同查询语句的内存处理区域（但不能同时指向多个内存处理区域，在某一时刻只能与一个查询语句相连），只要这些查询语句的返回类型兼容即可。
TYPE ref_type_name IS REF CURSOR
[ RETURN return_type];
**游标变量操作**
与游标一样，游标变量操作也包括打开、提取和关闭三个步骤。
1． 打开游标变量
打开游标变量时使用的是**OPEN…FOR** 语句。格式为：
OPEN {cursor_variable_name | :host_cursor_variable_name}
FOR select_statement;
提取游标变量数据
使用FETCH 语句提取游标变量结果集合中的数据。格式为：
FETCH {cursor_variable_name | :host_cursor_variable_name}
INTO {variable [, variable]…| record_variable};
关闭游标变量
CLOSE 语句关闭游标变量，格式为：
CLOSE {cursor_variable_name | :host_cursor_variable_name}

```

DECLARE
TYPE emp_job_rec IS RECORD(
Employee_id employees.employee_id%TYPE,
Employee_name employees.first_name%TYPE,
Job_title employees.job_id%TYPE
);
TYPE emp_job_refcur_type IS REF CURSOR RETURN emp_job_rec;
Emp_refcur emp_job_refcur_type ;
Emp_job emp_job_rec;
BEGIN
OPEN emp_refcur FOR
SELECT employees.employee_id, employees.first_name||employees.last_name, employees.jo
b_id
FROM employees
ORDER BY employees.department_id;
FETCH emp_refcur INTO emp_job;
WHILE emp_refcur%FOUND LOOP
DBMS_OUTPUT.PUT_LINE(emp_job.employee_id||': '||emp_job.employee_name||' is a '||e
mp_job.job_title);
FETCH emp_refcur INTO emp_job;
END LOOP;
END;

```


##  异常 
有三种类型的异常错误：
1． 预定义( Predefined )错误
ORACLE 预定义的异常情况大约有24 个。对这种异常情况的处理，无需在程序中定义，由ORACLE 自动将其引发。
2． 非预定义( Predefined )错误
即其他标准的ORACLE 错误。对这种异常情况的处理，需要用户在程序中定义，然后由ORACLE 自动将其引发。
3． 用户定义(User_define) 错误
程序执行过程中，出现编程人员认为的非正常情况。对这种异常情况的处理，需要用户在程序中定义，然后显式地在程序中将其引发。
异常处理部分一般放在PL/SQL 程序体的后半部,结构为:
```

EXCEPTION
WHEN first_exception THEN <code to handle first exception >
WHEN second_exception THEN <code to handle second exception >
WHEN OTHERS THEN <code to handle others exception >
END;

```
异常处理可以按任意次序排列,**但OTHERS 必须放在最后.**

更新指定员工工资，如工资小于1500，则加100；
```


DECLARE
v_empno employees.employee_id%TYPE := &empno;
v_sal employees.salary%TYPE;
BEGIN
SELECT salary INTO v_sal FROM employees WHERE employee_id = v_empno;
IF v_sal<=1500 THEN
UPDATE employees SET salary = salary + 100 WHERE employee_id=v_empno;
DBMS_OUTPUT.PUT_LINE('编码为'||v_empno||'员工工资已更新!');
ELSE
DBMS_OUTPUT.PUT_LINE('编码为'||v_empno||'员工工资已经超过规定值!');
END IF;
EXCEPTION
WHEN NO_DATA_FOUND THEN
DBMS_OUTPUT.PUT_LINE('数据库中没有编码为'||v_empno||'的员工');
WHEN TOO_MANY_ROWS THEN
DBMS_OUTPUT.PUT_LINE('程序运行错误!请使用游标');
WHEN OTHERS THEN
DBMS_OUTPUT.PUT_LINE(SQLCODE||'---'||SQLERRM);
END;

```

##  非预定义的异常处理 
对于这类异常情况的处理，首先必须对非定义的ORACLE 错误进行定义。步骤如下：
1. 在PL/SQL 块的定义部分定义异常情况：
<异常情况> EXCEPTION;
2. 将其定义好的异常情况，与标准的ORACLE 错误联系起来，使用EXCEPTION_INIT 语
句：
PRAGMA EXCEPTION_INIT(<异常情况>, <错误代码>);
3. 在PL/SQL 块的异常情况处理部分对异常情况做出相应的处理。
```

INSERT INTO departments VALUES(50, 'FINANCE', 'CHICAGO');
DECLARE
v_deptno departments.department_id%TYPE := &deptno;
deptno_remaining EXCEPTION;
PRAGMA EXCEPTION_INIT(deptno_remaining, -2292);
/* -2292 是违反一致性约束的错误代码*/
BEGIN
DELETE FROM departments WHERE department_id = v_deptno;
EXCEPTION
WHEN deptno_remaining THEN
DBMS_OUTPUT.PUT_LINE('违反数据完整性约束!');
WHEN OTHERS THEN
DBMS_OUTPUT.PUT_LINE(SQLCODE||'---'||SQLERRM);
END;

```

##  用户自定义的异常处理 
当与一个异常错误相关的错误出现时，就会隐含触发该异常错误。用户定义的异常错误是通过显式使用**RAISE 语句**来触发。当引发一个异常错误时，控制就转向到EXCEPTION
块异常错误部分，执行错误处理代码。
对于这类异常情况的处理，步骤如下：
1． 在PL/SQL 块的定义部分定义异常情况：
<异常情况> EXCEPTION;
2． RAISE <异常情况>；
3． 在PL/SQL 块的异常情况处理部分对异常情况做出相应的处理。

```

更新指定员工工资，增加100；
DECLARE
v_empno employees.employee_id%TYPE :=&empno;
no_result EXCEPTION;
BEGIN
UPDATE employees SET salary = salary+100 WHERE employee_id = v_empno;
IF SQL%NOTFOUND THEN
RAISE no_result;
END IF;
EXCEPTION
WHEN no_result THEN
DBMS_OUTPUT.PUT_LINE('你的数据更新语句失败了!');
WHEN OTHERS THEN
DBMS_OUTPUT.PUT_LINE(SQLCODE||'---'||SQLERRM);
END;

```
##  在PL/SQL 中使用SQLCODE, SQLERRM 异常处理函数 
由于ORACLE 的错信息最大长度是512 字节，为了得到完整的错误提示信息，我们可
用SQLERRM 和SUBSTR 函数一起得到错误提示信息，方便进行错误，特别是如果WHEN
OTHERS 异常处理器时更为方便。
SQLCODE 返回遇到的Oracle 错误号,
SQLERRM 返回遇到的Oracle 错误信息.
```

EXCEPTION
WHEN OTHERS THEN
DBMS_OUTPUT.PUT_LINE(SQLCODE||'---'||SQLERRM);
END;

```