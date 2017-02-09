title: oracle入门之概念与基本操作 

#  oracle入门之概念与基本操作 
A. 小型数据库：access、foxbase。负载量小，100人内，成本千元内，安全性要求不高。例如留言板等。
B. 中型数据库：Mysql、SQL Server、Informix。日访问量5000-15000，成本万元内。例如电子商务网站等。
C. 大型数据库：Sybase<Oracle<DB2。海量负载。

Oracle与其他数据库不同的是，每次登陆，登陆的是Oracle数据库的**实例**。每个用户，例如sys、system、scott，每个用户名登陆的数据库看到的**数据对象（表、存储过程等**）不同。
Oracle默认生成三个用户：
1. sys用户：超级管理员，权限最高，角色dba
2. system用户：系统管理员，角色dbaoper
3. scott用户：普通用户，密码tiger
sys有create database的权限，而system没有。

**Oracle管理工具：**
1. SQLPLUS
2. Oracle Enterprise Manager Console
3. pl/sql developer

**SQL * PLUS重要命令**
sqlplus可以参考：http://blog.csdn.net/v123411739/article/details/29282719
1. 连接命令conn[etc]:conn system/conanswp;
2. 断开连接：disc[onnect]
3. 修改密码：passw[ord];需要使用sys、system用户登录。
4. 显示当前用户：show user;
5. 退出：exit

**Oracle用户管理：**
1. 创建用户：create user；eg: create username identified by password；
2. 修改密码：password user或者alter user identified password；
3. 删除用户：drop user [cascade];如果待删除的用户已经创建了表，就需要带上cascade参数。
4. 指定用户的权限：grant；eg：grant connect to user；connect是权限。Oracle大约有140多种系统权限，25种对象权限。
5. 回收权限使用命令：revoke；revoke select on emp from user;
6. 一个用户查询另一个用户的表（使用对象权限）：grant select on emp to user;允许user查询emp表（sys、system和scott）。user查询如下：select * from scott.emp;
7. 一个用户允许查询另一个用户的表并希望此用户也可以继续让第三个用户查看表：
-如果是对象权限，使用with grant option。Eg：grant select on emp to user1 with grant option;
-如果是系统权限，使用with admin option。
A->B->C（A让B访问A中表，B让C访问A中表），如果A回收B，现在C也不能访问A中的表了。

**权限：**
-系统权限：用户对数据库的相关权限。
-对象权限：用户对其他用户的数据对象操作的权限。对象权限分为：select、insert、update、delete、all等。

**数据对象**：用户创建的表、存储过程、视图等。

**角色**：权限批量授权给角色。方便权限的授予。角色分为：自定义角色和预定义角色。**重要的角色：DBA(系统管理)，resource（允许在表空间建表），connect**。grant resource to user；允许user创建表。


**profile**是口令限制，资源限制的命令集合。
1. 账户锁定：create profile lock_account limit failed_login_attempts 3 password_lock_time 2;alter user xiaoming profile lock_account每个用户只能尝试登陆3此，锁定时间为2天。创建的profile名字为lock_account。
2. 给账户解锁：alter user xiaoming account unlock;
3. 终止口令，为了让用户定期修改密码。Create profile myprofile limit password_life_time 10 password_grace_time 2;alter user xiaoming profile myprofile;每隔10天修改自家的登陆密码，宽限期为2天。
4. 禁止新旧密码相同：create profile password_history limit password_life_time 10 password_grace_time 2 password_reuse_time 10; alter user xiaoming profile password_history;
5. 删除profile：drop profile password_history

**Oracle字段的数据类型。**
1. 表名、列名的命名规则：字母开头，不超过30字符，不能使用oracle保留字。
2. 支持的**数据类型：**
a) 字符型：
  * i. char（定长）：char(10) ‘Conan’。5个存放字符，5个空着，但是查询效率高。最大2000字符。
  * ii. varchar2（变长）：自动增加、减少，节省空间。最大4000字符。
  * iii. clob（character large object）：字符型大对象，最大4G。
  * b) 数字型：number(4):4位。number(5,2)：共5位，2位小数。
c) 日期类型：
  * i. Date：包括年月日时分秒
  * ii. Timestamp：时间戳
d) 二进制数据：blob。可存放图片、声音


**表与字段管理：**
修改表：
a).添加字段
alter table student add (classid number(2));
b).修改字段
alter table student modify (classid number(2));
c).删除字段
alter table student drop column sal;
d).修改表名
rename tablename to stu；
e).删除表名
drop table tablename；
5. 查看表：desc tablename；
6、添加数据：Oracle默认日期格式’DD-MON-YY’
01-12月-13中“月”必须写上！
insert into student values(‘a001’,’conan’,’male’,’01-12月-13’);
插入部分字段
insert into student (xh) values(‘a001’);
添加空值
Insert into student(xh,xm,sex,birthday) values (‘a004’,’MARTIN’,’男’,null);

7、修改字段数据：
修改一个字段
update student set sex=’female’ where xh=”a004”;
修改多个字段
update student set sex=’female’，birthday=’xx’  where xh=”a004”;

8、删除数据
删除所有记录，表结构还在，因为写入日志，可以恢复数据
delete from student;
删除表的结构和数据
drop table student;
删除一条记录
delete from student where xh=’a001’;
删除表中所有记录，表结构还在，不写日志，无法找回
truncate table student;

9. 设置保存点：savepoint aa;
10. 恢复到保存点：rollback to aa;

5.用查询结果创建新表：
Create table mytable(id,name,sal,job,deptno) as select empno,ename,sal,job,deptno from emp;
6.合并查询。使用集合操作符union，union，all，intersect，minus。
A.union:取集合的并集
Select ename,sal,job from emp where sal>2500 union select ename,sal,job from emp where job=’manager’;
B.union all:不取消重复行，不排序
C.intersect:取两个集合的交集。
D.minus:取两个集合的差集。显示前面的不现实后面的。


**分页查询**。例如，按照雇员的id号升序取出。
Oracle分页查询一共有三种：
A.**rownum**分页：select a1.*, rownum from (select * from emp) a1;
select a1.*, rownum from (select * from emp) a1 where rownum <=10;不能再rownum>=6，**oracle规定rn只能用一次。**
Select * from (select a1.*, rownum from (select * from emp) a1 where rownum <=10) where rownum>=6;修改查询只需要在里层改动。
B**.ROWID**:效率很高
C.按照分析函数来分。效率太差。

Oracle中操作数据
1.使用**to_date**函数：to_date(‘1988-04-16’,’yyyy-mm-dd’);
2.**使用子查询插入数据**：导入emp表中10号部门到新表kkk中。
Insert into kkk (MyID,MyName) select empo,ename,detpno from emp where deptno=10;
3.**使用子查询更新数据**：更新员工scott的岗位、工资和smith员工一样
Update emp set (job,sal)=(select job,sal from emp where ename=’SMITH’) where ename=’SCOTT’;

##  Java操作oracle 
```

//使用jdbc_odbc桥连接oracle（使用数据源），不能远程连接。
Public class TestOra
{
  Public static void main()
{
  //加载驱动
  Class.froName(“sun.jdbc.odbc.JdbcOdbcDriver”);
  //得到连接，需要配置数据源。假设数据源名为：test
  Connection ct = DriverManager.getConnection(
“jdbc:odbc:test”,”scott”,”tiger”);
    //
    Statement sm=ct.createStatement();
    ResultSet rs = sm.executeQuery(“select * from emp”);
    While(rs.next())
    {
        System.out.println(rs.getString(2));//默认从1开始编号
}
Sm.close();
Ct.close();
}
}
//使用jdbc
Public class TestOra
{
  Public static void main()
  {
    //加载驱动
Class.forName(“oracle.jdbc.driver.OracleDriver”);
Connection ct = DriverManager.getConnection(
“jdbc:oracle:thin:@localhost:1521:MyDB”,”scott”,”tiger”);
  Statement sm=ct.createStatement();
    ResultSet rs = sm.executeQuery(“select * from emp”);
    While(rs.next())
    {
        System.out.println(rs.getString(2));//默认从1开始编号
}
Sm.close();
Ct.close();
}
}
//分页查询：
String s_pageNow = (String)request.getParameter(“pageNow”);
If(s_pageNow!=null)
{
 pageNow=Integer.parseInt(s_pageNow);
}
Int pageCount=0;    //计算值
Int rowCount=0;    //共有几条记录
Int pageSize=0;    //每页显示几条记录
 
Result rs = sm.executeQuery(“select count(*) from emp”);
If(rs.next())
{
    rowCount=rs.getInt(1);
}
If(rowCount%pageSize==0)
{
    pageCount = rowCount/pageSize;
}
Else
{
  pageCount = rowCount/pageSize+1;
}
Rs=sm.executeQuery(“elect * from (select a1.*, rownum rn from (select * from emp) a1 where rn <=”+pageNow*pageSize+”) where rn>=”+((pageNow-1)*pageSize+1));
While(rs.next())
{
Out.println(“<tr>”);
Out.println(“<td>”+rs.getString(2)+”</td>”);
Out.println(“<td>”+rs.getString(2)+”</td>”);
Out.println(“</tr>”);
}
//打印总页数
For(int i=1;i<=pageCount;i++)
{
  Out.pirntln(“<a href=MyTest.jsp?pageNow=> ”+i+” </a>”);
}

```

**Oracle中的事务：**
事务用于保证数据的一致性，它由一组相关的dml（数据操纵语言）语句组成，该组的dml语句要么全部成功，要么全部失败。Dml语言是说操纵语言，包括增删改操作。当执行事务操作的时候，oracle会在被作用的表进行加锁，防止其他用户对该表进行操作。
使用commit提交事务。当执行commit后，会确认事务的变化，结束事务、删除保存点、释放锁，当使用commit语句结束事务后，其他绘画可以查看事务变化后的新数据。
保存点（savepoint）：是事务中的一点，用于取消部分事务，当结束事务时，会自动的删除该事务所定义的所有保存点。当执行回退事务rallback是，通过指定保存点可以回退到指定的点。这里需要注意的是这些保存点要想能够回退，是以没有commit为前提的。使用命令commit提交，使用exit退出的时候也会自动commit。
事务的几个重要操作：
1. 设置保存点：savepoint a;
2. 取消部分事务：rallback to a;
3. 取消全部事务：rollback;
只读事务：只允许执行查询的操作，不允许执行其他dml操作的事务。只读事务可以确保用户只能够取得某时间点的数据。 
设置只读事务：set transaction read only


##  SQL函数的使用 
1.字符函数：
lower(char):将字符串转化为小写
upper(char):将字符串转换为大写
length(char):返回字符串长度
substr(char,m,n):取字符串的字串
replace(char1,serarch_string,replace_string):替换函数
instr(char1,char2,[,n[,m[]):取字串位置函数
eg：将所有员工的名字按照小写显示：select lower(name) from emp;
2.数学函数：
cos,cosh,exp,ln,log,sin,sinh,sqrt,tan…
round(n,[m]):执行四舍五入，保留m位小数
trunk(n,[m]);截取数字，截取n小数点后的m位
mod(m,n):去模
floor(n):返回小于或者等于n的最大整数
ceil(n):返回大于或是等于n的最小整数
abs(n):绝对值
3.日期函数：默认格式为dd-mon-yy，例如12-7月-15,显示为2015-7-12。
Sysdate：当前系统时间。Select sysdate from emp;
Add_months(d,n):加一个月份。Eg：查找入职8个月入职的员工。
Select from emp where sysdate>add_months(hiredate,8);
Last_day(d):返回指定日期的最后一天
4.转换函数：用于将数据类型的转换，oracle提供自动转换。
to_char():转换字符函数
eg1：日期显示时分秒：select ename ,to_char(hiredate,’yyyy-mm-dd hh24:mi:ss’);
eg:显示日民币：select ename ,to_char(hiredate,’yyyy-mm-dd hh24:mi:ss’) to_char(sal,’L9999.99’);L表示当地语言，9999.99代表显示数字格式。
5.系统函数
Sys_context：查询系统参数

##  数据库管理： 
Sys：董事长
System：总经理
主要区别：
1. 存储的数据的重要性不同
Sys：所有oracle的数据字典的基表和视图都是放在sys用户中的，sys拥有**dba，sysdba，sysoperar**的角色，是用户权限最高的用户。
System：存放系统次一级的内部数据，拥有dba和sysdbajuese或者系统权限。

2. 权限的不同
` Sys必须以as sysdba或者as stsoper形式登录，不能以normal形式登录 `
Systm如果正常登录，其实就是一个普通的dba用户，如果以as sysdba登录，结果实际上就是作为sys用户登录。
Sysoper较之sysdba不不能改变字符集、创建、删除数据库

sysdba>sysoper>dba（大致如此）。其中dba不能启动关闭数据库。


##  imp/exp导入导出 
导出：导出表，导出方案，导出数据库，使用exp命令完成，常用的选项有：
Userid：用于指定齿形导出操作的命令名，口令
Tables：用于指定导出操作的表
Owner：用于指定执行导出操作的方案
Full=y：用于指定执行导出操作的数据库
Rows：用于指定导出操作的增量类型
File：用于指定导出文件名
Inctype：用于指定导出操作的增量类型
A导出表：
Exp userid=scott/tiger@MyDB tables=(emp) file=d:\e1.dmp;导出自己的表。
Exp userid=system/manager@myoral tables=(scott.emp) file=d:\e2.emp;导出其他方法的表，需要dba权限
B.导出表结构：
Exp userid=scott/tiger@MyDB tables=(emp) file=d:\e3.dmp rows=n
C.直接导出方式:速度快
Exp userid=scott/tiger@MyDB tables=(emp) file=d:\e3.dmp direct=y;需要数据库的字符集合客户端的字符集完全一致。
D.导出方案：
Exp scott/tiger@myDB owner=scott file=d:\1.dmp导出自己方案
Exp system/manager@mydb owner=(system,scott) file=d:\2.dmp;导出其他方案（使用system导出scott的方案）
E.导出数据库
Exp userid=system/manager@mydb full=y inctype=complete file=x.dmp;inctype表示增量备份。
F.导入表：
Imp userid=scott/tiger@mydb tables=(emp) file=d:\xx.dmp;导入自己的表
Imp userid=system/manager@mydb tables=(emp) file=d:\d.dmp;导入其他表
Imp userid=scott/tiger@mydb tables=(emp) file=d:\1.dmp row=n;导入数据表结构
Imp userid=scott/tiger@mydb tables=(emp) file=d:\1.dmp ignore=y;如果数据表已经存在，只导入数据。
G.导入方：
Imp userid=scott/tiger file=1.dmp;导入自己方案
Imp userid=system/manager file=d:\xxx.dmp fromuser=system touser=scott;
H.导入数据库：
Imp userid=system/manager full=y file=1.dmp;


##  数据字典和动态性能视图（V$) 
数据字典提供数据库的系统信息；数据字典属于sys用户。
动态性能视图记载例程启动后的相关信息。
数据字典分为：字典基表和动态视图构成。基表存储数据库基本信息，普通用户不能直接访问数据字典的基表。
数据字典视图是基于数据字典基表简历的视图，普通用户可以同过查询字典视图获取系统信息。**数据字典视图主要包括：user_xxx，all_xxx，dba_xxx三种类型。**
User_tables:显示当前用户所拥有的表。Select table_name from user_tables;
All_tables:显示当前用户可访问的所有表。
Dba_tables：显示所有方案拥有的数据表。

###  常用动态视图和字典表 
http://blog.sina.com.cn/s/blog_6d6e54f70100nlsi.html
一、DBA最常用的数据字典
**dba_data_files:通常用来查询关于数据库文件的信息    
dba_db_links:包括数据库中的所有数据库链路，也就是databaselinks。**
dba_extents:数据库中所有分区的信息                      
dba_free_space:所有表空间中的自由分区
**dba_indexs:关于数据库中所有索引的描述**                 
dba_ind_columns:在所有表及聚集上压缩索引的列
` dba_objects:数据库中所有的对象 `                             
dba_rollback_segs:回滚段的描述
dba_segments:所有数据库段分段的存储空间             
dba_synonyms:关于同义词的信息查询
**dba_tables:数据库中所有数据表的描述 **                     
**dba_tabespaces:关于表空间的信息**
**dba_tab_columns:所有表描述、视图以及聚集的列 **     
**dba_tab_grants/privs:对象所授予的权限**
dba_ts_quotas:所有用户表空间限额                         
**dba_users:关于数据的所有用户的信息**
**dba_views:数据库中所有视图的文本**
二、DBA最常用的动态性能视图
**v$database:数据库的信息，如数据库名，创建时间等。**
**v$instance 实例信息，如实例名，启动时间。**
**v$datafile：数据库使用的数据文件位置信息**                    
v$librarycache：共享池中SQL语句的管理信息
v$lock：通过访问数据库会话，设置对象锁的所有信息  
v$log：从控制文件中提取有关重做日志组的信息
v$logfile有关实例重置日志组文件名及其位置的信息     
v$parameter：初始化参数文件中所有项的值
**v$nls_parameters：语言与字符编码相关信息**
v$process：当前进程的信息                                  
v$rollname：回滚段信息 
v$rollstat：联机回滚段统计信息                              
v$rowcache：内存中数据字典活动/性能信息
v$session:有关会话的信息                                     
v$sesstat：在v$session中报告当前会话的统计信息 
v$sqlarea：共享池中使用当前光标的统计信息，光标是一块内存区域，有Oracle处理SQL语句时打开。
v$statname：在v$sesstat中报告各个统计的含义    
v$sysstat：基于当前操作会话进行的系统统计


###  设置客户端编码 
select * from v$nls_parameters;查询相关信息，然后设置
NLS_LANG=AMERICAN_AMERICA.ZHS16GBK,三部分，AMERICAN语言，提示信息，AMERICA国家影响日期格式，ZHS16GBK编码。

##   管理表空间和数据文件 
Orracle的逻辑结构包括：表空间、段、区和块。数据库由表空间构成，表空间由段构成，段由区构成，区由块构成。
表空间：表空间是数据库的逻辑组成部分。表空间由一个或者多个文件组成。表空间控制数据库占用的磁盘空间，dba可以将不同数据类型不熟到不同的位置，这样有利于提高i/o性能。
1. 建立表空间：create tablespace
2. 建立数据表空间：create tablespace data01 datafile ‘c:\data01.dbf’ size 20m uniform size 128k;区大小128K
3. 使用表空间：create table mypart(deptno number(4),dname vchar2(14)) tablespace sp001;
```

---创建表空间
CREATE TABLESPACE "TS_SJCQ_DATA" DATAFILE
     'F:\ORA11GR2\ORADATA\ORCL\DB_SJCQ_DATA01.DBF' SIZE 400m REUSE
     LOGGING ONLINE PERMANENT BLOCKSIZE 8192
     EXTENT MANAGEMENT LOCAL UNIFORM SIZE 4m SEGMENT SPACE MANAGEMENT AUTO
     ;
---删除表空间，同时删除数据文件
drop tablespace TS_SJCQ_DATA including contents and datafiles;

```
##  约束 
数据的完整性用于确保数据库的数据遵从一定的商业和逻辑规则。在
中，数据的完整性可以使用约束、触发器，应用程序（过程、函数）三种方式完成。
约束用于确保数据库数据满足特定的商业规则**。Oracle中约束包括：not null、unique，primary key，foreign key和check五中。**
Not null：非空，插入数据时必须提供数据
Unique：该值不能重复，但是可以为null
Primary key：唯一的标示表行的数据，该列不能重复且不能为null，一张表只能有一个主键，但可以有多个unique。
Foreign key：用于定义主表和从表之间的关系。外键约束要定义在从表上，主表则必须具有主键约束或者unique约束当定义外键约束后，要求外键列数据必须在主表的主键列或者是null。
Check：强制行数据满足条件。
Create table goods(goodsID char(8) primary key,--主键
goodsName varchar2(30),
nitprice number(10,2) check (unitprice>0),
category varchar2(8),
provider varchar2(30));

Create table customer(customerID char(8) privary key,
name vchar2(30) not null,
address varchar2(50),
email varchar2(50) unique,
sex char(2) default ‘男’ check(sex in ‘男’,’女’)
cardId char(18));

Create table purchase (customerId char(8) references customer(customerId),
goodsId char(8) references goods(goodId),
nums number(5) check (nums between 1 and 30));
**删除约束**：alter table 表名 drop constraint 约束名称。在删除主键时，有必要加入cascade关键字，以防主键有关联关系。
##  索引：用于加速数据存取的数据对象。 
单列索引：基于单个列锁创建的索引：create index 索引名 on 表名（列名）
复合索引：基于两列或者是多列的索引。

权限：执行特定类型sql命令或是访问其他方案对象的权利，**包括系统权限和对象权限两种。**
角色：相关权限的集合，用于简化权限的管理。

##  PL/SQL、存储过程、函数、包简介 
PL/SQL（procedural language/sql）是oracle标准的sql语言的扩展。它不仅允许嵌入sql语言，还可以定义变量和常量，并允许使用条件和循环语言，以及使用例外处理各种错误等。
/ /创建
Create procedure mypc is
Begin
Insert into mytabel values(‘a’,’sf’);
End;
/用于通知执行该存储过程
/ /调用
**Exec 过程名或则call过程名**

块构成：
Declare：定义部分
Begin：执行部分
Exception：里外部分
End；

##  在java中使用存储过程 
```

CallableStatement cs = conn.prepareCall(“{call mypc(?,?)}”);
Cs.setString(1,”Smith”);
Cs.setInt(2,20);
Cs.execute();
Cs.close();
Conn.close();

```
**函数**：用于返回特定的数据，使用create function创建函数
Create function myfun(name varchar2) return Number is mysal number(7,2);
Begin
Select sal*12+nvl(comm.,0)*12 into mysal from emp where ename=name;
Return mysal;
End;
调用：
Sqlplus中：
Var income number;
Call myfun(‘SCOTT’) into:income;
Print income;
Java中：
Select myfun(‘SCOTT’) from emp;
然后通过rs返回结果集合。

**包**：用于在逻辑上组合过程和函数，使用create package创建。
Create package mypackage is
Procedure update_sal(name carchar);
Function annual_income(name varchar2) return number;
End;
**包由包规范（上面）和包体构成**，包体使用create package body is命令。在包体中分别实现包规范中对过程和函数的声明。调用包：call mypack.update_sal(‘Scott’);带上包名即可。

触发器：隐含执行的存储过程。使用create trigger来建立触发器，需要制定触发的时间和触发的操作。

变量类型：
1. 标量类型：v_ename varchar2(10);v_sal number(6,2):5,4;定义并初始化
2. 复合类型:用于存放多个值的变量。
3. 参照类型
4. lob（large object）
##  示例问题 
1. 查询SMITH的薪水、工作，所在部门：(oracle大小写不区分，但是单引号中的区分大小写)
Select deptno,job,sal from emp where ename=’SMITH’;
2. 使用算数表达式显示每个员工的年工资
Select ename, sal*12+nvl(comm.,0)*12 from emp；nvl表示如果comm是null，则替代为0；不为空，就用comm本身。否则，oracle规定，同含有null的进行运算，结果整体为null。
Select ename, sal*12 “年工资” from emp;(年工资是sal*12的别名)
3. 如何显示工资高于3000的员工
Select ename,sal from emp where sal>3000 and sal<10000;
4. like操作符。%表示0到多个字符；_:表示任意单个字符。显示首字母为S的员工姓名和工资。
Select ename,sal from emp where ename like ‘S%’;
5. where中使用in。显示empno为123,345,800的雇员情况
select * form emp where ename in (123,345,566);
6. is null.显示没有上级的雇员
select * from emp where MGR is null.
7. 逻辑运算符。and or
Select * from emp where (sal > 50 or job =’MANAGER’)注意括号
8. order by.按照工资从低到高的顺序显示雇员信息。
Select * from emp order by sal [asc];//从低到高
Select * from emp order by sal desc;//从高到底
按照部门号升序而雇员工资降序排列：
Select * from emp order by deptno, sal desc;
对年薪排序：
Select ename,sal*12 “年薪” from emp order by “年薪”;
9. 分页查询：详见子查询。
17. Oracle复杂查询
1. 数据分组：max,min,avg,sum,cout;
显示所有员工中最高工资和最低工资：
select max(sal),min(sal) from emp;
select ename,sal from emp where sal=(select max(sal) from emp);
2. group by（查询结果分组统计）和having子句（限制分组显示结果）
显示每个部门的平均工资和最高工资：
Select avg(sal),max(sal),deptno from emp group by deptno;
显示每个部门的每种岗位的平均工资和最低工资
Select avg(sal),min(sal),deptno,job from emp group by deptno,job;
显示平均工资低于2000的部门号和他的平均工资：
Select avg(sal),max(sal),deptno from emp group by deptno having avg(sal)>2000;
**总结：
1. 分组函数只能出现在选择列表、having、order by子句中；
2. 如果在select语句中同时包含group by，having，order by，那么他的书序是group by，having，order by。
3. 在选择列中如果有列、表达式和分组函数，那么这些列和表达式必须有一个出现子group by 子句中，否则就会出错。**


18. 多表查询：
1.显示雇员名，雇员工资及所在部门的名字：
Select a1.ename,a1.sal,a2.dname from emp a1,dept a2 where a1.deptno=a2.deptno;不加where条件的话，是用第一个表中的每一条跟第二个表的全部构成，一共为第一表长度乘以第二个表长度。多表查询的条件至少是（表数-1）！
2.自连接：在同一张表的连接查询。显示某个员工的上级领导的姓名。
select worker.ename,boss.ename from emp worker,emp boss where boss.empno=worker.mgr;
3.子查询：嵌入在其他sql语句中的select查询。
单行子查询：返回一行数据的子查询语句。显示与SMITH同一部门的所有员工。
Select * form emp where deptno=(select deptno from emp where ename=’SMITH’);从右往左执行。
多行子查询：返回多行数据的子查询。查询和部门10的工作相同的雇员的名字、岗位、工资和部门号。
Select * from emp where job in (select distinct job from emp where deptno=10);
多行查询中使用all操作符。显示工资比部门30的所有员工的工资高的员工的姓名、工资和部门号。
Select ename,sal,dept from emp where sal>all(select sal from emp where deptno=30);
在多行查询中使用any操作符。显示工资比部门30的任意一个员工的工资高的员工的姓名、工资和部门号。
select ename,sal,dept from emp where sal>any(select sal from emp where deptno=30);
多列子查询。查询与smith的部门和岗位完全相同的所有雇员。
Select deptno,job from emp where ename=’smith’;
Select * from emp where (deptno,job)=(select deptno,job from emp where ename =’smith’);
显示高于自己部门平均工资的员工的信息：
(1.查询各部门的平均工资)Select deptno,avg(sal) mysal from emp group by deptno;
(2.将上面的查询看做是一张子表)select a2.ename,a2.sal,a2.deptno from emp a2, (Select deptno,avg(sal) mysal from emp group by deptno) a1 where a2.deptno=a1.deptno and a2.sal>a1.mysal;
在from中使用子查询的时候，该子查询会被当做成一个视图对待，成为内前视图。在from中使用子查询时必须制定别名。


参考：http://blog.csdn.net/conanswp/article/details/37834061

