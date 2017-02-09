title: pl_sql学习之触发器 

#  PL/SQL学习之触发器 
触发器类似过程和函数，都有声明，执行和异常处理过程的PL/SQL 块。
触发器在数据库里以独立的对象存储，它与存储过程和函数不同的是，存储过程与函数需要用户显示调用才执行，**而触发器是由一个事件来启动运行**。即触发器是当某个事件发生时自动地隐式运行。
并且，**触发器不能接收参数。所以运行触发器就叫触发或点火（firing）。**ORACLE 事件指的是对数据库的表进行**的INSERT、UPDATE 及DELETE 操作或对视图进行类似的操**作。ORACLE 将触发器的功能扩展到了触发ORACLE**，如数据库的启动与关闭等**。所以触发器常用来完成由数据库的完整性约束难以完成的复杂业务规则的约束，或用来监视对数据库的各种操作，实现审计的功能。
分类：DML 触发器、替代触发器、系统触发器
触发器组成:
 **触发事件**：引起触发器被触发的事件。例如：DML 语句(INSERT, UPDATE,DELETE 语句对表或视图执行数据处理操作)、DDL 语句（如CREATE、ALTER、DROP 语句在数据库中创建、修改、删除模式对象）、数据库系统事件（如系统启动或退出、异常错误）、用户事件（如登录或退出数据库）。
 **触发时间**：即该TRIGGER 是在触发事件发生**之前（BEFORE）还是之后(AFTER)触发**，也就是触发事件和该TRIGGER 的操作顺序。
 **触发操作**：即该TRIGGER 被触发之后的目的和意图，正是触发器本身要做的事情。例如：PL/SQL 块。
 **触发对象**：包括**表、视图、模式、数据库**。只有在这些对象上发生了符合触发条件的触发事件，才会执行触发操作。
 **触发条件**：由WHEN 子句指定一个逻辑表达式。只有当该表达式的值为TRUE 时，遇到触发事件才会自动执行触发器，使其执行触发操作。
 **触发频率**：说明触发器内定义的动作被执行的次数。即语句级(STATEMENT)
触发器和行级(ROW)触发器。
语句级(STATEMENT)触发器：是指当某触发事件发生时，该触发器只执行一次；
行级(ROW)触发器：是指当某触发事件发生时，对受到该操作影响的每一行数据，触发器都单独执行一次。


编写触发器时，需要注意以下几点：
 触发器不接受参数。
 一个表上最多可有12 个触发器，但同一时间、同一事件、同一类型的触发器只能有一个。并各触发器之间不能有矛盾。
 在一个表上的触发器越多，对在该表上的DML 操作的性能影响就越大。
 触发器最大为32KB。若确实需要，可以先建立过程，然后在触发器中用CALL 语句进行调用。
 在触发器的执行部分只能用DML 语句（SELECT、INSERT、UPDATE、DELETE），不能使用DDL 语句（CREATE、ALTER、DROP）。
 触发器中不能包含事务控制语句(COMMIT，ROLLBACK，SAVEPOINT)。因为触发器是触发语句的一部分，触发语句被提交、回退时，触发器也被提交、回退了。
 在触发器主体中调用的任何过程、函数，都不能使用事务控制语句。
 在触发器主体中不能申明任何Long 和blob 变量。新值new 和旧值old也不能向表中的任何long 和blob 列。
 不同类型的触发器(如DML 触发器、INSTEAD OF 触发器、系统触发器)的语法格式和作用有较大区别。

创建触发器的一般语法是:
```

CREATE [OR REPLACE] TRIGGER trigger_name
{BEFORE | AFTER }
{INSERT | DELETE | UPDATE [OF column [, column …]]}
[OR {INSERT | DELETE | UPDATE [OF column [, column …]]}...]
ON [schema.]table_name | [schema.]view_name
[REFERENCING {OLD [AS] old | NEW [AS] new| PARENT as parent}]
[FOR EACH ROW ]
[WHEN condition]
PL/SQL_BLOCK | CALL procedure_name;

```
其中：
FOR EACH ROW 选项说明触发器为行触发器。当省略FOR EACH ROW 选项时，BEFORE 和AFTER 触发器为语句触发器，
而INSTEAD OF 触发器则只能为行触发器。
REFERENCING 子句说明相关名称，在行触发器的PL/SQL 块和WHEN
子句中可以使用相关名称参照当前的新、旧列值，默认的相关名称分别为OLD和NEW。触发器的PL/SQL 块中应用相关名称时，必须在它们之前加冒号(:)，
但在WHEN 子句中则不能加冒号。
**WHEN 子句说明触发约束条件。Condition 为一个逻辑表达时，其中必须包含相关名称，而不能包含查询语句，也不能调用PL/SQL 函数。**
WHEN 子句指定的触发约束条件只能用在BEFORE 和AFTER 行触发器中，不能用在INSTEAD OF 行触发器和其它类型的触发器中。
当一个基表被修改( INSERT, UPDATE, DELETE)时要执行的存储过程，执行时根据其所依附的基表改动而自动触发，因此与应用程序无关，用数据库触发器可以保证数据的一致性和完整性。
每张表最多可建立12 种类型的触发器，它们是:
 BEFORE INSERT
 BEFORE INSERT FOR EACH ROW
 AFTER INSERT
 AFTER INSERT FOR EACH ROW
 BEFORE UPDATE
 BEFORE UPDATE FOR EACH ROW
 AFTER UPDATE
 AFTER UPDATE FOR EACH ROW
 BEFORE DELETE
 BEFORE DELETE FOR EACH ROW
 AFTER DELETE
 AFTER DELETE FOR EACH ROW
8.2.1 触发器触发次序
1. 执行BEFORE 语句级触发器;
2. 对与受语句影响的每一行：
  *  执行BEFORE 行级触发器
  *  执行DML 语句
  *  执行AFTER 行级触发器
3. 执行AFTER 语句级触发器

触发器名与过程名和包的名字不一样，它是单独的名字空间，因而触发器名可以和表或过程有相同的名字，但在一个模式中触发器名不能相同。


**使用触发器谓词**
ORACLE 提供三个参数INSERTING, UPDATING, DELETING 用于判断触发了哪些操作。
谓词行为
  * INSERTING 如果触发语句是INSERT 语句，则为TRUE,否则为FALSE
  * UPDATING 如果触发语句是UPDATE 语句，则为TRUE,否则为FALSE
  * DELETING 如果触发语句是DELETE 语句，则为TRUE,否则为FALSE
##  创建DML触发器 
例1: 建立一个触发器, 当职工表emp 表被删除一条记录时，把被删除记录写
到职工表删除日志表中去。
```

CREATE TABLE emp_his AS SELECT * FROM EMP WHERE 1=2;
CREATE OR REPLACE TRIGGER TR_DEL_EMP
  BEFORE DELETE --指定触发时机为删除操作前触发
ON SCOTT.EMP
  FOR EACH ROW --说明创建的是行级触发器
BEGIN
  --将修改前数据插入到日志记录表del_emp ,以供监督使用。
  INSERT INTO EMP_HIS
    (DEPTNO, EMPNO, ENAME, JOB, MGR, SAL, COMM, HIREDATE)
  VALUES
    (:OLD.DEPTNO,
     :OLD.EMPNO,
     :OLD.ENAME,
     :OLD.JOB,
     :OLD.MGR,
     :OLD.SAL,
     :OLD.COMM,
     :OLD.HIR EDATE);
END;
DELETE emp WHERE empno=7788;
DROP TABLE emp_his;
DROP TRIGGER del_emp;

```
例2：限制对Departments 表修改（包括INSERT,DELETE,UPDATE）的时间
范围，即不允许在非工作时间修改departments 表。
```

CREATE OR REPLACE TRIGGER TR_DEPT_TIME
  BEFORE INSERT OR DELETE OR UPDATE ON DEPARTMENTS
BEGIN
  IF (TO_CHAR(SYSDATE, 'DAY') IN (' 星期六',
                                  ' 星期日
')) OR (TO_CHAR(SYSDATE, 'HH24:MI') NOT BETWEEN '08:30' AND '18:00') THEN
    RAISE_APPLICATION_ERROR(-20001, '不是上班时间，不能修改departments 表');
  END IF;
END;


```
例5：在触发器中调用过程。
```

CREATE OR REPLACE PROCEDURE ADD_JOB_HISTORY(P_EMP_ID        JOB_HISTORY.EMPLOYEE_ID%TYPE,
                                            P_START_DATE    JOB_HISTORY.START_DATE%TYPE,
                                            P_END_DATE      JOB_HISTORY.END_DATE%TYPE,
                                            P_JOB_ID        JOB_HISTORY.JOB_ID%TYPE,
                                            P_DEPARTMENT_ID JOB_HISTORY.DEPARTMENT_ID%TYPE) IS
BEGIN
  INSERT INTO JOB_HISTORY
    (EMPLOYEE_ID, START_DATE, END_DATE, JOB_ID, DEPARTMENT_ID)
  VALUES
    (P_EMP_ID, P_START_DATE, P_END_DATE, P_JOB_ID, P_DEPARTMENT_ID);
END ADD_JOB_HISTORY;
  --创建触发器调用存储过程...
  CREATE OR REPLACE TRIGGER UPDATE_JOB_HISTORY
  AFTER UPDATE OF JOB_ID, DEPARTMENT_ID ON EMPLOYEES
  FOR EACH ROW
BEGIN
  ADD_JOB_HISTORY(:OLD.EMPLOYEE_ID,
                  :OLD.HIRE_DATE,
                  SYSDATE,
                  :OLD.JOB_ID,
                  :OLD.DEPARTMENT_ID);
END;


```
##  其它触发器（略） 
##  数据库触发器的应用实例 
用户可以使用数据库触发器实现各种功能：
###  复杂的审计功能 
例：将EMP 表的变化情况记录到AUDIT_TABLE 和AUDIT_TABLE_VALUES中。
```

CREATE TABLE audit_table(
Audit_id NUMBER,
User_name VARCHAR2(20),
Now_time DATE,
Terminal_name VARCHAR2(10),
Table_name VARCHAR2(10),
Action_name VARCHAR2(10),
Emp_id NUMBER(4));

CREATE TABLE audit_table_val(
Audit_id NUMBER,
Column_name VARCHAR2(10),
Old_val NUMBER(7,2),
New_val NUMBER(7,2));

CREATE SEQUENCE audit_seq
START WITH 1000
INCREMENT BY 1
NOMAXVALUE
NOCYCLE NOCACHE;

CREATE OR REPLACE TRIGGER AUDIT_EMP
  AFTER INSERT OR UPDATE OR DELETE ON EMP
  FOR EACH ROW
DECLARE
  TIME_NOW DATE;
  TERMINAL CHAR(10);
BEGIN
  TIME_NOW := SYSDATE;
  TERMINAL := USERENV('TERMINAL');
  IF INSERTING THEN
    INSERT INTO AUDIT_TABLE
    VALUES
      (AUDIT_SEQ.NEXTVAL,
       USER,
       TIME_NOW,
       TERMINAL,
       'EMP',
       'INSERT',
       :NEW.EMPNO);
  ELSIF DELETING THEN
    INSERT INTO AUDIT_TABLE
    VALUES
      (AUDIT_SEQ.NEXTVAL,
       USER,
       TIME_NOW,
       TERMINAL,
       'EMP',
       'DELETE',
       :OLD.EMPNO);
  ELSE
    INSERT INTO AUDIT_TABLE
    VALUES
      (AUDIT_SEQ.NEXTVAL,
       USER,
       TIME_NOW,
       TERMINAL,
       'EMP',
       'UPDATE',
       :OLD.EMPNO);
    IF UPDATING('SAL') THEN
      INSERT INTO AUDIT_TABLE_VAL
      VALUES
        (AUDIT_SEQ.CURRVAL, 'SAL', :OLD.SAL, :NEW.SAL);
    ELSE
      UPDATING('DEPTNO') INSERT
        INTO AUDIT_TABLE_VAL VALUES(AUDIT_SEQ.CURRVAL,
                                    'DEPTNO',
                                    :OLD.DEPTNO,
                                    :NEW.DEPTNO);
    END IF;
  END IF;
END;


```