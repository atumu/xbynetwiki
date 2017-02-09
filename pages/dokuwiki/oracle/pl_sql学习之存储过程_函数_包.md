title: pl_sql学习之存储过程_函数_包 

#  PL/SQL学习之存储过程、函数、包 
过程与函数（另外还有包与触发器）是命名的PL/SQL 块（也是用户的方案对象），被编译后存储在数据库中，以备执行。因此，其它PL/SQL 块可以按名称来使用他们。所以，可以将商业逻辑、企业规则写成函数或过程保存到数据库中，以便共享。
**过程和函数统称为PL/SQL 子程序**，他们是被命名的PL/SQL 块，均存储在数据库中，并通过输入、输出参数或输入/输出参数与其调用者交换信息。
**过程和函数的唯一区别是函数总向调用者返回数据，而过程则不返回数据。**
##  函数 
语法如下：
```

CREATE [OR REPLACE] FUNCTION function_name
(arg1 [ { IN | OUT | IN OUT }] type1 [DEFAULT value1],
[arg2 [ { IN | OUT | IN OUT }] type2 [DEFAULT value1]],
......
[argn [ { IN | OUT | IN OUT }] typen [DEFAULT valuen]])
[ AUTHID DEFINER | CURRENT_USER ]
RETURN return_type
IS | AS
<类型.变量的声明部分>
BEGIN
执行部分
RETURN expression
EXCEPTION
异常处理部分
END function_name;

```
 **IN,OUT,IN OUT 是形参的模式**。**若省略，则为IN 模式**。
  * **IN 模式**的形参只能将实参传递给形参，进入函数内部，但**只能读不能写**，函数返回时实参的值不变。
  * **OUT 模式**的形参会忽略调用时的实参值（或说该形参的初始值总是NULL），但在函数内部可以被读或写，函数返回时形参的值会赋予给实参。
  * **IN OUT** 具有前两种模式的特性，即调用时，实参的值总是传递给形参，结束时，形参的值传递给实参。
调用时，对于IN模式的实参可以是常量或变量，但对于OUT 和IN OUT 模式的实参必须是变量。
 一般，只有在确认function_name 函数是新函数或是要更新的函数时，才使用OR REPALCE 关键字，否则容易删除有用的函数。
例1. 获取某部门的工资总和：
```

  --获取某部门的工资总和
  CREATE OR REPLACE FUNCTION GET_SALARY(DEPT_NO   NUMBER,
                                        EMP_COUNT OUT NUMBER) RETURN NUMBER IS
  V_SUM NUMBER;
BEGIN
  SELECT SUM(SALARY), COUNT(*)
    INTO V_SUM, EMP_COUNT
    FROM EMPLOYEES
   WHERE DEPARTMENT_ID = DEPT_NO;
  RETURN V_SUM;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('你需要的数据不存在!');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE(SQLCODE || '---' || SQLERRM);
END GET_SALARY;


```
2. 函数的调用
函数声明时所定义的参数称为形式参数，应用程序调用时为函数传递的参数称为实际参数。
应用程序在调用函数时，可以使用以下三种方法向函数传递参数：
第一种参数传递格式：位置表示法。
V_sum :=get_salary(10, v_num);
第二种参数传递格式：名称表示法。
格式为：argument => parameter [,…]
其中：argument 为形式参数，它必须与函数定义时所声明的形式参数名称相同，parameter 为实际参数。
第三种参数传递格式：组合传递。
Var := demo_fun('user1', 30, sex => '男');
无论采用哪一种参数传递方法，**实际参数和形式参数之间的数据传递只有两种方法：传址法和传值法。**所谓传址法是指在调用函数时，将实际参数的地址指针传递给形式参
数，使形式参数和实际参数指向内存中的同一区域，从而实现参数数据的传递。**这种方法又称作参照法**，即形式参数参照实际参数数据。
传值法是指将实际参数的数据拷贝到形式参数，而不是传递实际参数的地址。默认时，输出参数和输入/输出参数均采用传值法。在函数调用时，ORACLE 将实际参数数据拷贝到输入/输出参数，而当函数正常运行退出时，又将输出形式参数和输入/输出形式参数数据拷贝到实际参数变量中。

3. 参数默认值
在CREATE OR REPLACE FUNCTION 语句中声明函数参数时可以使用**DEFAULT**
关键字为输入参数指定默认值。


##  存储过程 
在ORACLE SERVER 上建立存储过程,可以被多个应用程序调用,可以向存储过程传递参数,也可以向存储过程传回参数.
创建过程语法:
```

CREATE [OR REPLACE] PROCEDURE procedure_name
([arg1 [ IN | OUT | IN OUT ]] type1 [DEFAULT value1],
[arg2 [ IN | OUT | IN OUT ]] type2 [DEFAULT value1]],
......
[argn [ IN | OUT | IN OUT ]] typen [DEFAULT valuen])
[ AUTHID DEFINER | CURRENT_USER ]
{ IS | AS }
<声明部分>
BEGIN
<执行部分>
EXCEPTION
<可选的异常错误处理程序>
END procedure_name;

```
删除指定员工记录；
```

CREATE OR REPLACE PROCEDURE DELEMP(V_EMPNO IN EMPLOYEES.EMPLOYEE_ID%TYPE) AS
  NO_RESULT EXCEPTION;
BEGIN
  DELETE FROM EMPLOYEES WHERE EMPLOYEE_ID = V_EMPNO;
  IF SQL%NOTFOUND THEN
    RAISE NO_RESULT;
  END IF;
  DBMS_OUTPUT.PUT_LINE('编码为' || V_EMPNO || '的员工已被删除!');
EXCEPTION
  WHEN NO_RESULT THEN
    DBMS_OUTPUT.PUT_LINE('温馨提示:你需要的数据不存在!');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE(SQLCODE || '---' || SQLERRM);
END DELEMP;


```

###  调用存储过程 
存储过程建立完成后，只要**通过授权**，用户就可以在SQLPLUS 、ORACLE 开发工具或**第三方开发工具**中来调用运行。对于参数的传递也有三种：按位置传递、按名称传递和组合传递，传递方法与函数的一样。ORACLE 使用**EXECUTE 语句**来实现对存储过程的调用：
EXEC[UTE] procedure_name( parameter1, parameter2…);
##  本地函数和过程 
在PL/SQL 程序中还可以在块内建立本地函数和过程，这些函数和过程不存储在数据库中，但可以在创建它们的PL/SQL 程序中被重复调用。本地函数和过程在PL/SQL块的声明部分定义，它们的语法格式与存储函数和过程相同，**但不能使用CREATE OR REPLACE 关键字。**
例13：建立本地过程，用于计算指定部门的工资总和，并统计其中的职工数量；
```

DECLARE
  V_NUM NUMBER;
  V_SUM NUMBER(8, 2);
  PROCEDURE PROC_DEMO(DEPT_NO   NUMBER DEFAULT 10,
                      SAL_SUM   OUT NUMBER,
                      EMP_COUNT OUT NUMBER) IS
  BEGIN
    SELECT SUM(SALARY), COUNT(*)
      INTO SAL_SUM, EMP_COUNT
      FROM EMPLOYEES
     WHERE DEPARTMENT_ID = DEPT_NO;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      DBMS_OUTPUT.PUT_LINE('你需要的数据不存在!');
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE(SQLCODE || '---' || SQLERRM);
  END PROC_DEMO;
  --调用方法：
BEGIN
  PROC_DEMO(30, V_SUM, V_NUM);
  DBMS_OUTPUT.PUT_LINE('30 号部门工资总和：' || V_SUM || '，人数：' || V_NUM);
  PROC_DEMO(SAL_SUM => V_SUM, EMP_COUNT => V_NUM);
  DBMS_OUTPUT.PUT_LINE('10 号部门工资总和：' || V_SUM || '，人数：' || V_NUM);
END;


```

###  存储过程权限赋 
GRANT EXECUTE ON logexecution TO PUBLIC WITH GRANT OPTION;

**PRAGMA AUTONOMOUS_TRANSACTION**
ORACLE8i 可以支持事务处理中的事务处理的概念．这种子事务处理可以完成它
自己的工作，独立于父事务处理进行提交或者回滚．通过使用这种方法，开发者就能够
这样的过程，无论父事务处理是提交还是回滚，它都可以成功执行．

###  与过程相关数据字典 
USER_SOURCE, ALL_SOURCE, DBA_SOURCE, 
USER_ERRORS,
ALL_PROCEDURES,
USER_OBJECTS,ALL_OBJECTS,DBA_OBJECTS
相关的权限:
CREATE ANY PROCEDURE
DROP ANY PROCEDURE
EXECUTE

在SQL*PLUS 中，可以用DESCRIBE 命令查看过程的名字及其参数表。
DESC[RIBE] Procedure_name;

##  删除过程和函数 
1．删除过程
可以使用DROP PROCEDURE 命令对不需要的过程进行删除，语法如下：
DROP PROCEDURE [user.]Procudure_name;
2．删除函数
可以使用DROP FUNCTION 命令对不需要的函数进行删除，语法如下：
DROP FUNCTION [user.]Function_name;
--删除上面实例创建的存储过程与函数
DROP PROCEDURE logexecution;
DROP PROCEDURE delemp;
DROP FUNCTION demo_fun;
DROP FUNCTION get_salary;

###  使用过程与函数的原则 
1、如果需要返回多个值和不返回值，就使用过程；如果只需要返回一个值，就使用函数。
2、过程一般用于执行一个指定的动作，函数一般用于计算和返回一个值。
3、可以SQL 语句内部（如表达式）调用函数来完成复杂的计算问题，但不能调用过程。所以这是函数的特色。

##  程序包(PACKAGE) 
程序包(PACKAGE，简称包)是一组相关过程、函数、变量、常量和游标等PL/SQL 程序设计元素的组合，作为一个完整的单元存储在数据库中，用名称来标识包。它具有面向对象程序设计语言的特点，是对这些PL/SQL 程序设计元素的封装。**包类似于c#和JAVA 语言中的类，其中变量相当于类中的成员变量，过程和函数相当于类方法。**
一个包由两个分开的部分组成：
**包声明（PACKAGE）**：包说明部分声明包内数据类型、变量、常量、游标、子程序和异常错误处理等元素，这些元素为**包的公有元素**。
**包主体（PACKAGE BODY**）：包主体则是包定义部分的具体实现，它定义了包定义部分所声明的游标和子程序，在包主体中还可以声明包的私有元素。
包说明和包主体分开编译，并作为两部分分开的对象存放在数据库字典中，可查看数据字典**user_source, all_source, dba_source**，分别了解包说明与包主体的详细信息。

创建程序**包说明**语法格式:
```

CREATE [OR REPLACE] PACKAGE package_name
[AUTHID {CURRENT_USER | DEFINER}]
{IS | AS}
[公有数据类型定义[公有数据类型定义]…]
[公有游标声明[公有游标声明]…]
[公有变量、常量声明[公有变量、常量声明]…]
[公有函数声明[公有函数声明]…]
[公有过程声明[公有过程声明]…]
END [package_name];

```
其中：AUTHID CURRENT_USER 和AUTHID DEFINER 选项说明应用程序在调用函数时所使用的权限模式，它们与CREATE FUNCTION 语句中invoker_right_clause 子句的作用相同。

创建程序**包主体**语法格式:
```

CREATE [OR REPLACE] PACKAGE BODY package_name
{IS | AS}
[私有数据类型定义[私有数据类型定义]…]
[私有变量、常量声明[私有变量、常量声明]…]
[私有异常错误声明[私有异常错误声明]…]
[私有函数声明和定义[私有函数声明和定义]…]
[私有函过程声明和定义[私有函过程声明和定义]…]
[公有游标定义[公有游标定义]…]
[公有函数定义[公有函数定义]…]
[公有过程定义[公有过程定义]…]
BEGIN
执行部分(初始化部分)
END package_name;

```
其中：在包主体定义公有程序时，它们必须与包定义中所声明子程序的格式完全一致。

**7.3 包的开发步骤**
与开发存储过程类似，包的开发需要几个步骤：
1． 将每个存储过程调式正确；
2． 用文本编辑软件将各个存储过程和函数集成在一起；
3． 按照包的定义要求将集成的文本的前面加上包定义；
4． 按照包的定义要求将集成的文本的前面加上包主体；
5． 使用SQLPLUS 或开发工具进行调式。

例1:创建的包为DEMO_PKG, 该包中包含一个记录变量DEPTREC、两个函数和一个过程。
实现对dept 表的增加、删除与查询。
```

CREATE OR REPLACE PACKAGE DEMO_PKG
IS
DEPTREC DEPT%ROWTYPE;
--Add dept...
FUNCTION add_dept(
dept_no NUMBER,
dept_name VARCHAR2,
location VARCHAR2)
RETURN NUMBER;
--delete dept...
FUNCTION delete_dept(dept_no NUMBER)
RETURN NUMBER;
--query dept...
PROCEDURE query_dept(dept_no IN NUMBER);
END DEMO_PKG;

```
包主体的创建方法，它实现上面所声明的包定义，并在包主体中声明一个私有变量flag和一个私有函数check_dept ， 由于在add_dept 和remove_dept 等函数中需要调用check_dpet 函数，所以，在定义check_dept 函数之前首先对该函数进行声明，这种声明方法称作**前向声明**。
```

CREATE OR REPLACE PACKAGE BODY DEMO_PKG IS
  FUNCTION ADD_DEPT(DEPT_NO NUMBER, DEPT_NAME VARCHAR2, LOCATION VARCHAR2)
    RETURN NUMBER IS
    EMPNO_REMAINING EXCEPTION; --自定义异常
    PRAGMA EXCEPTION_INIT(EMPNO_REMAINING, -1);
    /* -1 是违反唯一约束条件的错误代码*/
  BEGIN
    INSERT INTO DEPT VALUES (DEPT_NO, DEPT_NAME, LOCATION);
    IF SQL%FOUND THEN
      RETURN 1;
    END IF;
  EXCEPTION
    WHEN EMPNO_REMAINING THEN
      RETURN 0;
    WHEN OTHERS THEN
      RETURN - 1;
  END ADD_DEPT;
  FUNCTION DELETE_DEPT(DEPT_NO NUMBER) RETURN NUMBER IS
  BEGIN
    DELETE FROM DEPT WHERE DEPTNO = DEPT_NO;
    IF SQL%FOUND THEN
      RETURN 1;
    ELSE
      RETURN 0;
    END IF;
  EXCEPTION
    WHEN OTHERS THEN
      RETURN - 1;
  END DELETE_DEPT;
  PROCEDURE QUERY_DEPT(DEPT_NO IN NUMBER) IS
  BEGIN
    SELECT * INTO DEPTREC FROM DEPT WHERE DEPTNO = DEPT_NO;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:数据库中没有编码为' || DEPT_NO || '的部门');
    WHEN TOO_MANY_ROWS THEN
      DBMS_OUTPUT.PUT_LINE('程序运行错误,请使用游标进行操作!');
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE(SQLCODE || '----' || SQLERRM);
  END QUERY_DEPT;
BEGIN
  NULL;
END DEMO_PKG;


```
对包内共有元素的调用格式为：包名.元素名称调用DEMO_PKG 包内函数对dept 表进行插入、查询和删除操作，并通过DEMO_PKG包中的记录变量DEPTREC 显示所查询到的数据库信息：
```

DECLARE
  VAR NUMBER;
BEGIN
  VAR := DEMO_PKG.ADD_DEPT(90, 'HKLORB', 'HAIKOU');
  IF VAR = -1 THEN
    DBMS_OUTPUT.PUT_LINE(SQLCODE || '----' || SQLERRM);
  ELSIF VAR = 0 THEN
    DBMS_OUTPUT.PUT_LINE('温馨提示:该部门记录已经存在！');
  ELSE
    DBMS_OUTPUT.PUT_LINE('温馨提示:添加记录成功！');
    DEMO_PKG.QUERY_DEPT(90);
    DBMS_OUTPUT.PUT_LINE(DEMO_PKG.DEPTREC.DEPTNO || '---' ||
                         DEMO_PKG.DEPTREC.DNAME || '---' ||
                         DEMO_PKG.DEPTREC.LOC);
    VAR := DEMO_PKG.DELETE_DEPT(90);
    IF VAR = -1 THEN
      DBMS_OUTPUT.PUT_LINE(SQLCODE || '----' || SQLERRM);
    ELSIF VAR = 0 THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:该部门记录不存在！');
    ELSE
      DBMS_OUTPUT.PUT_LINE('温馨提示:删除记录成功！');
    END IF;
  END IF;
END;


```
```

--创建序列从100 开始,依次增加1
CREATE SEQUENCE empseq
START WITH 100
INCREMENT BY 1
ORDER NOCYCLE;
--创建序列从100 开始,依次增加10
CREATE SEQUENCE deptseq
START WITH 100
INCREMENT BY 10
ORDER NOCYCLE;
-- *******************************************
-- 创建包说明
-- 包名：MANAGE_EMP_PKG
-- 功能描述：对员工进行管理(新增员工,新增部门
-- ,删除员工,删除部门,增加工资与奖金等)
-- 创建人员：kenny
-- 创建日期：2010-05-19
-- ******************************************
CREATE OR REPLACE PACKAGE MANAGE_EMP_PKG AS
  --增加一名员工
  FUNCTION HIRE_EMP(ENAME  VARCHAR2,
                    JOB    VARCHAR2,
                    MGR    NUMBER,
                    SAL    NUMBER,
                    COMM   NUMBER,
                    DEPTNO NUMBER) RETURN NUMBER;
  --新增一个部门
  FUNCTION ADD_DEPT(DNAME VARCHAR2, LOC VARCHAR2) RETURN NUMBER;
  --删除指定员工
  PROCEDURE REMOVE_EMP(EMPNO NUMBER);
  --删除指定部门
  PROCEDURE REMOVE_DEPT(DEPTNO NUMBER);
  --增加指定员工的工资
  PROCEDURE INCREASE_SAL(EMPNO NUMBER, SAL_INCR NUMBER);
  --增加指定员工的奖金
  PROCEDURE INCREASE_COMM(EMPNO NUMBER, COMM_INCR NUMBER);
END MANAGE_EMP_PKG; --创建包说明结束
-- *******************************************
-- 创建包体
-- 包名：MANAGE_EMP_PKG
-- 功能描述：对员工进行管理(新增员工,新增部门
-- ,删除员工,删除部门,增加工资与奖金等)
-- 创建人员：kenny
-- ******************************************
CREATE OR REPLACE PACKAGE BODY MANAGE_EMP_PKG AS
  TOTAL_EMPS  NUMBER; --员工数
  TOTAL_DEPTS NUMBER; --部门数
  NO_SAL  EXCEPTION;
  NO_COMM EXCEPTION;
  --增加一名员工
  FUNCTION HIRE_EMP(ENAME  VARCHAR2,
                    JOB    VARCHAR2,
                    MGR    NUMBER,
                    SAL    NUMBER,
                    COMM   NUMBER,
                    DEPTNO NUMBER) RETURN NUMBER --返回新增加的员工编号
   IS
    NEW_EMPNO NUMBER(4);
  BEGIN
    SELECT EMPSEQ.NEXTVAL INTO NEW_EMPNO FROM DUAL;
    SELECT COUNT(*) INTO TOTAL_EMPS FROM EMP; --当前记录总数
    INSERT INTO EMP
    VALUES
      (NEW_EMPNO, ENAME, JOB, MGR, SYSDATE, SAL, COMM, DEPTNO);
    TOTAL_EMPS := TOTAL_EMPS + 1;
    RETURN(NEW_EMPNO);
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:发生系统错误!');
  END HIRE_EMP;
  --新增一个部门
  FUNCTION ADD_DEPT(DNAME VARCHAR2, LOC VARCHAR2) RETURN NUMBER IS
    NEW_DEPTNO NUMBER(4); --部门编号
  BEGIN
    --得到一个新的自增的员工编号
    SELECT DEPTSEQ.NEXTVAL INTO NEW_DEPTNO FROM DUAL;
    SELECT COUNT(*) INTO TOTAL_DEPTS FROM DEPT; --当前部门总数
    INSERT INTO DEPT VALUES (NEW_DEPTNO, DNAME, LOC);
    TOTAL_DEPTS := TOTAL_DEPTS;
    RETURN(NEW_DEPTNO);
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:发生系统错误!');
  END ADD_DEPT;
  --删除指定员工
  PROCEDURE REMOVE_EMP(EMPNO NUMBER) IS
    NO_RESULT EXCEPTION; --自定义异常
  BEGIN
    DELETE FROM EMP WHERE EMP.EMPNO = REMOVE_EMP.EMPNO;
    IF SQL%NOTFOUND THEN
      RAISE NO_RESULT;
    END IF;
    TOTAL_EMPS := TOTAL_EMPS - 1; --总的员工数减1
  EXCEPTION
    WHEN NO_RESULT THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:你需要的数据不存在!');
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:发生系统错误!');
  END REMOVE_EMP;
  --删除指定部门
  PROCEDURE REMOVE_DEPT(DEPTNO NUMBER) IS
    NO_RESULT                  EXCEPTION; --自定义异常
    EXCEPTION_DEPTNO_REMAINING EXCEPTION; --自定义异常
    /*-2292 是违反一致性约束的错误代码*/
    PRAGMA EXCEPTION_INIT(EXCEPTION_DEPTNO_REMAINING, -2292);
  BEGIN
    DELETE FROM DEPT WHERE DEPT.DEPTNO = REMOVE_DEPT.DEPTNO;
    IF SQL%NOTFOUND THEN
      RAISE NO_RESULT;
    END IF;
    TOTAL_DEPTS := TOTAL_DEPTS - 1; --总的部门数减1
  EXCEPTION
    WHEN NO_RESULT THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:你需要的数据不存在!');
    WHEN EXCEPTION_DEPTNO_REMAINING THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:违反数据完整性约束!');
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:发生系统错误!');
  END REMOVE_DEPT;
  --给指定员工增加指定数量的工资
  PROCEDURE INCREASE_SAL(EMPNO NUMBER, SAL_INCR NUMBER) IS
    CURR_SAL NUMBER(7, 2); --当前工资
  BEGIN
    --得到当前工资
    SELECT SAL INTO CURR_SAL FROM EMP WHERE EMP.EMPNO = INCREASE_SAL.EMPNO;
    IF CURR_SAL IS NULL THEN
      RAISE NO_SAL;
    ELSE
      UPDATE EMP
         SET SAL = SAL + INCREASE_SAL.SAL_INCR --当前工资加新增的工资
       WHERE EMP.EMPNO = INCREASE_SAL.EMPNO;
    END IF;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:你需要的数据不存在!');
    WHEN NO_SAL THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:此员工的工资不存在!');
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:发生系统错误!');
  END INCREASE_SAL;
  --给指定员工增加指定数量的奖金
  PROCEDURE INCREASE_COMM(EMPNO NUMBER, COMM_INCR NUMBER) IS
    CURR_COMM NUMBER(7, 2);
  BEGIN
    --得到指定员工的当前资金
    SELECT COMM
      INTO CURR_COMM
      FROM EMP
     WHERE EMP.EMPNO = INCREASE_COMM.EMPNO;
    IF CURR_COMM IS NULL THEN
      RAISE NO_COMM;
    ELSE
      UPDATE EMP
         SET COMM = COMM + INCREASE_COMM.COMM_INCR
       WHERE EMP.EMPNO = INCREASE_COMM.EMPNO;
    END IF;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:你需要的数据不存在!');
    WHEN NO_COMM THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:此员工的奖金不存在!');
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('温馨提示:发生系统错误!');
  END INCREASE_COMM;
END MANAGE_EMP_PKG; --创建包体结束
--调用
SQL> variable empno number
SQL>execute :empno:= manage_emp_pkg.hire_emp('HUYONG',PM,1455,5500,14,10)
PL/SQL procedure successfully completed
empno
---------

```

###  子程序重载 
PL/SQL 允许对包内子程序和本地子程序进行重载。所谓重载时指两个或多个子程序有相同的名称，但拥有不同的参数变量、参数顺序或参数数据类型。

###  加密实用程序 
ORACLE 提供了一个实用工具来加密或者包装用户的PL/SQL，它会将用户的PL/SQL
改变为只有ORACLE 能够解释的代码版本．
WRAP 实用工具位于$ORACLE_HOME/BIN.
格式为：
WRAP INAME=<input_file_name> [ONAME=<output_file_name>]
wrap iname=e:\sample.txt
注意：在加密前，请将PL/SQL 程序先保存一份，以备后用。

###  删除包 

可以使用DROP PACKAGE 命令对不需要的包进行删除，语法如下：
DROP PACKAGE [BODY] [user.]package_name;
DROP PROCEDURE OpenCurType; --删除存储过程
--删除我们实例中创建的各个包
DROP PACKAGE demo_pack;

##  包的管理与源码 
包与过程、函数一样，也是存储在数据库中的，可以随时查看其源码。若有需要，在创建包时可以随时查看更详细的编译错误。不需要的包也可以删除。
同样，为了避免调用的失败，在更新表的结构后，一定要记得重新编译依赖于它的程序包。在更新了包说明或包体后，也应该重新编译包说明与包体。语法如下：
ALTER PACKAGE package_name COMPILE [PACKAGE|BODY|SPECIFICATION];
也可以通过以下数据字典视图查看包的相关。
**DBA_SOURCE, USER_SOURCE, USER_ERRORS, DBA-OBJECTS**
如，我们可以用：**select text from user_source where name = 'DEMO_PKG1';**来查看我们创建的包的源码。
