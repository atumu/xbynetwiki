title: oracle_sql学习 

#  Oracle SQL学习 
一.简单语句；
set linesize 200;			（设置行的大小）
set pagesize 200; 			（设置页面的大小）
rename stu3 to student;		（重命名表）
rollback					（恢复）
commit						（提交，在这之前没有真正的提交，只是存在缓存中）
DESC stu;/describe stu;		（查看表结构）

（符号标记：[]可选，<>必填，| 任选其一）
（特殊语言：/,r,run表示执行，--表示注释）

二.建表
	
1.借用已经存在的表建表；
SQL> create table stu **as select** * from emp; 
	
2.建表的同时选定要建的哪几项并改名
SQL> create table stu2(员工编号,员工姓名) as select empno,ename from emp;
	
3.只引用表的结构
SQL> create table stu1 as select * from emp where deptno=50;
	
4.自己建表
SQL> create table stu3(学生编号 varchar(10),学生姓名 varchar(10),学生性别 varchar(10));  

三.语言分类
1.DDL数据定义语言 （只真对表结构，是不可恢复的）
2.DML数据操纵语言
3.DQL数据查询语言 (简单查询)
4.DCL数据控制语言

##  四. DDL数据定义语言 

CREATE、ALTER、DROP、TRUNVATE
```

建表------------------------------create table stu3(学生编号 varchar(10),学生姓名 varchar(10));
增加字段--------------------------alter table student add( stuage number(2),stulove varchar2(10));
修改类型--------------------------alter table student modify(stuage number(7)); 
删除一列--------------------------alter table student drop column stuage; 
把一个字段改为可以为空------------alter table EXCHANGE modify BAK_1 null; 
把一个字段改为不可以为空----------alter table items modify id not null;
修改字段默认值--------------------alter table votetable modify (isanyone default 1)；
给表添加释------------------------comment on table votetable is '投票基本配置信息表';
给列添加注释----------------------comment on column votetable.isanyone is '1表示要登入,2表示不需要登入'；
查找主键约束名--------------------select T.constraint_name from user_constraints T where table_name='items' and constraint_type='p' and rownum<2;
添加主键--------------------------alter table items add constraint pk_items primary key(id);
添加外键--------------------------alter table items add constraint fk_items_r_votetable foreign key (voteid) references votetable(voteid);
删除表----------------------------drop table stu;
修改列名--------------------------ALTER TABLE student RENAME COLUMN   sid to sno;
删除表内容------------------------truncate table myemp;
修改表的名称----------------------rename itmes to items;

```

##  五.DML数据操纵语言 
 
INSERT
UPDATE
DELETE
插入:	
insert into student(stuID,stuName,stuSex,stuAge)values('10','tom','男','20');
insert into myemp values(1999,'FHDSGHFD','CERK',1000,sysdate,1300.00,1400.00,20);
insert into myemp values(1999,'FHDSGHFD','CERK',1000,sysdate,1300.00,default,20);/ /default表示默认。没有为null
使用子查询语句插入
insert into stu select * from emp where deptno=10;
     
修改:update student set stuSex='女' where stuID=10;
使用子查询语句修改：
update myemp set(JOB,SAL,COMM)=(select JOB,SAL,COMM from emp where ename='MILLER') where ename='KING';

删除:delete student where stuID=10;


##  六.DQL数据查询语言 
 
SELECT

```

select * from student;
select stuSex from student;
SELECT stuID, stuName, stuAge+10 age FROM student;
SELECT stuID, stuName, stuAge+10 as "You Age" FROM student;
//(给stuAge+10取别名为age，有时别名里面区分大小写或者有空格是用“”)
//算术表达式里包括一个null，则结果也为null。
//取别名as可以用可以不用
SELECT stuID, stuName, stuAge+10 "You Age" FROM student;

//去除重复的
select distinct stuAge from student;

//使用连接操作符||   数据和字符串必须被单引号引起来
select stuID||stuAge as stu from student;
		
//使用计算表dual（虚拟表）
SQL> select sysdate from dual;
SQL> select 1+2 from dual;
		
//用where 限定行
select * from student where stuID=11;
		
//字符串和日期值被单引号所标记，字符的值是大小写敏感的，并且日期值是格式敏感的

		
where 条件：
1.   =，>,<,>=,<=,<>（不等于）
				
2.between .....and.......
select * from student where stuID between 11 and 14;
				
3.使用in条件
select * from student where stuID in(11,13);
				
4.使用like条件，% 表示零个或多个字符. _ 表示一个字符.
当like条件中有%,_的则要转义（escape）;

select * from student where stuName like '%m%';（含有m的）
select * from student where stuName like '%m';  （以m结尾的）
select * from student where stuName like '_j%';	 （第二位是j的）
select * from myemp where job like '%x_%' escape 'x';

5. 使用 is null;
select * from student where stuAge is null;
	相反
select * from student where stuAge is not null;
				
6. 使用逻辑条件and,or not



```
7.优先级规则
1	数学操作符 */+- 
2	连接操作 ||
3	条件比较
4	IS [NOT] NULL, LIKE, [NOT] IN
5	[NOT] BETWEEN
6	NOT 
7	AND 
8	OR 	


排序：
ORDER BY 子句排序行，ASC: 升序排序, 缺省   DESC: 降序排序
有null则null排在最前面
			
1.按某一行降序排列；
select * from student order by stuID desc;
			
2.按列的别名排序
select stuID||stuAge as stu from student order by stu;
			
3.按列的数字排序
select stuID,stuName,stuAge from student order by 3;

4.按多个列排序
select * from student order by stuID,stuAge desc;
（这时则按顺序，先按照stuID升蓄排，相同的再按stuAge降序排列）
##  函数： 
更多：http://www.cnblogs.com/Xonlyone/p/4063082.html
1.nvl(a,b)函数——判断a是不是为null,若为null则取b的值;
 nvl2(a,b,c)函数——判断a是不是为null,若为null则取c的值,否则取b的值
select ename, sal+nvl(comm,0) as 收入 from emp;
select nvl2('c',2,3) from dual;

2.to_char()———
select to_char(sysdate,'DD-MM-YYYY') from dual;
			
3.months_between() ———求两个时间之间有多少月
select months_between(sysdate,hiredate) from emp;
/ /两个日期相减，得到两个日期之间的天数

4.trunc()———不要小数点
round()———四舍五入参数表示从第几位舍
mod()——取余；
 select mod(125,2) from dual;
select trunc(months_between(sysdate,hiredate)) from emp;
select round(1254.2121,-1) from dual;
select trunc(2459.15445,-1) from dual;

5.last_day()——求本月最后一天
select last_day(sysdate) from dual;
select to_char(last_day(sysdate),'DD-MM-YYYY HH24') from dual;
						
6.initcap()——变首字母为大写
update emp set ename=initcap(ename);
			   
7.lower()——变小写
upper()——全部大写	
update emp set job = lower(job);
select upper(job) from myemp;
8.length()——求长度
select length(job) from emp;
			
9.substr()——求子串
instr()——求索引
select substr(ename,1,2) from emp;
select instr('helloworld','d') from dual;
10.replace()——替换；
select replace(ename,'A','B') from emp;
				 
11.add_months()——在原来月上添加
select add_months(hiredate,10) from emp;

12.concat()——与||一样是连接的；
select concat('hello','world') from dual;
select concap(empno,sal) from myemp;

13.lpad/rpad——填充
select lpad(sal,10,'*') from myemp;
select rpad(sal,10,'#') from myemp;
			
14.trim——去除端点字母或空格
select trim(0 from 01202560250) from myemp

15.next_day()下一个星期几是几号
select next_day(sysdate,'星期二') from dual;

16.roung/trunc日期的取舍
select round(sysdate,'yyyy') from dual;
select trunc(sysdate,'mm') from dual;

17.to_date()——转换成日期
select to_date('2008-02-03','yyyy-mm-dd') from dual;
不写模式则使用默认格式
18.nullif(a,b)——若a等于b则返回null;若a不等于b则返回a；
/ /a的值不能为空
select nullif(5,6) from dual;

20.case()when....then...else...end
				
SQL> select deptno,case deptno when 10 then 1.10*sal
2   when 20 then 1.20*sal
3  else sal end from emp;

21.decode(条件，值1，结果1，值2................没有匹配结果)
SQL> select deptno,decode(deptno,10,1.10*sal,20,1.20*sal,sal) from emp;

###  分组函数： 

/在使用分组函数时，除了count(*)以外，其他分组函数都会忽略null行
22.avg()——求平均数；
SQL> select avg(sal) from emp;
select avg(comm) from emp;				
				
23.max()——求最大 ；min()——求最小
select max(sal) from emp;
select min(sal) from emp;
				 
24.sum()——求总和；
select sum(sal) from emp;
				
25.count()——求总记录数
select count(comm) from emp;
select count(distinct deptno) from emp;

###  聚合函数 

聚合函数操作是在行的集合上给每一个组一个结果
使用 GROUP BY 子句汇总数据
使用HAVING子句将已分组的行包含进来或排除在外
			
having是选择组 where是选择行
单列分组和多列分组
			
26.group by;
select deptno,avg(sal) from emp group by deptno;
/ /在select中出现的字段一定要在group by彀中出现
				
27.having;
select avg(sal) from emp group by deptno having avg(sal)>2000;


##  七。连接查询； 

在from后面指定两个或者两个以上的表，有时为了不产生歧义在列名前面加上表名；
	

1.等值连接查询：“=”；
select * from emp,dept where emp.deptno=dept.deptno;			
			
2.and连接：AND
select * from emp e,dept d where e.deptno=d.deptno and e.deptno=10;		
			
3.不等连接，就是除相等连接以外的
select * from emp e,dept d where e.deptno between 10 and 20;
	
4.自连接
select * from emp e,emp y where e.mgr=y.empno and y.ename='KING';
	
5.内连接
//一般连接操作（不指明是外连接）都是内连接；
inner join .....on...
		
select emp.ename,dept.dname from emp,dept where emp.deptno=dept.deptno and emp.deptno=20;
select emp.ename,dept.dname from emp inner join dept on emp.deptno=dept.deptno and emp.deptno=20;
	
6.左外连接
//当左边的表有列右边没有，则左边的表全部显示；
left join .....on..../（+）；
		
select emp.ename,dept.dname from emp left join dept on emp.deptno=dept.deptno and emp.deptno=20;
select emp.ename,dept.dname from emp,dept where emp.deptno=dept.deptno(+) and emp.deptno=20;
7.右外连接
/ /与左外连接相反
right join......on..../（+）；
		
select emp.ename,dept.dname from emp right join dept on emp.deptno=dept.deptno and emp.deptno=20;
select emp.ename,dept.dname from emp,dept where emp.deptno(+)=dept.deptno and emp.deptno(+)=20;

8.全外连接
full join......on.....
		
select emp.ename,dept.dname from emp full join dept on emp.deptno=dept.deptno and emp.deptno=20;

/ /注意：（+）只能表示左右连接，只能使用在where语句中，只适用于列，不适用于表达式，不能与IN/OR连用
当有多个条件时，都要加（+）；

##  八.子查询； 

			
1.单行子查询；
select * from emp where deptno=(select deptno from dept where dname='RESEARCH');
		
2.多行子查询；是指返回多行数据的子查询语句；
in:匹配其中任意一个；
		
all:符合所有条件；
		
any:符合一个；

			
##  九.数据控制语言DCL： 

		
数据控制语言为用户提供权限控制命令，
数据库对象（比如表）的所有者对这些对象拥有独有的控制权限。
所有者可以根据自己的意愿决定其他用户如何访问对象，
授予其他用户权限（INSERT、SELECT、UPDATE……），使他们可以在其权限范围内执行操作
1.创建用户
create user bhw identified by yinhe;
2.给用户授权：
grant connect,create any table,resource,dba to bhw;
3.连接用户；conn bhw/yinhe;
4.显示当前用户：show user;
5.收回授权：revoke connect,create any table,resource,dba from bhw;
6.对用户的表对象授权和权限收回；
		
grant select,update on emp to bhw;
grant all on emp to bhw;

revoke select,update on emp from bhw;
revoke all on emp from bhw;
				
//使用with grant option，被授权的用户bhw拥有把权利授权给别的用户的权利；
grant select,update on emp to bhw with grant option;
grant select on scott.emp to bhw1;
				
//同时当收回给bhw的权利时，bhw授权的权限也将被收回；
revoke select,update on emp from bhw;
		
7.消除用户：
				
drop user bhw1 [casacde];
/ /所有被create创建的

##  8.约束： 

(1) not null;			非空；
(2)unique;				唯一；
(3)primary key;			主键；
(4)foreign key;			外键；
(5)check; 				检查；
		
		
主键；
alter table emp add constraint pk_emp primary key(empno);
增加一列：
alter table owen_action add addtime DATE;
删除一列
ALTER TABLE OWEN_ACTION DROP COLUMN ADDTIME;
外键；
alter table emp add constraint fk_emp foreign key(deptno) references dept(deptno);
检查；
alter table emp add constraint ck_emp check(sal between 700 and 9000);
重命名约束名；
alter table emp rename constraint ck_emp to ck_emp_sal;
禁止约束；
 alter table emp disable constraint ck_emp_sal;
恢复约束；
alter table emp enable constraint ck_emp_sal;
删除约束；
alter table emp drop constraint ck_emp_sal;
查询约束字典；
select * from user_constraints;

##  十.事务控制语言TCL 
 
1.执行命令：commit;
2.回滚命令：rollback;
3.保存点：savepoint;
savepoint a;
rollback to a;


##  十一.数据库对象(视图、序列、同义词、索引） 


###  1.视图 

			
create view emp_v1 as select ename,deptno from emp;
create view emp_v2 as select ename,job,deptno from emp with read only;（创建只读视图）

更新视图必须满足的条件
（1）在视图中使用DML语句只能修改一个底层的基表。
（2）只能修改键值保存表。（如果基表的主键在视图中也为主键，则称这个表为键值保存表。）
（3）如果对记录的修改违反了基表的约束条件，则无法更新视图。
（4）如果创建的视图包含连接运算符、DISTINCT运算符、集合运算符、聚合函数和GROUP BY子句，则将无法更新视图。
（5）如果创建的视图包含伪列或表达式，则将无法更新视图。
（6）不能有WITH  READ ONLY修饰。

drop view emp_v2;(删除视图)
			
注：基表修改，视图也自动修改：视图修改，基表也修改

###  2.序列： 

序列是为生成唯一数字列值创建的数据库对象
			
create sequence qu start with 1 increment by 2 maxvalue 50 cycle;
			
select qu.nextval from dual;
			
select qu.currval from dual;

//更新序列：不能修改名字和start with的值
alter sequence qu  increment by 4 nomaxvalue nocycle;

//删除序列
drop sequence qu;

###  3.同义词： 

同义词是数据库对象的一个别名，这些对象可以是表、视图、序列、过程、函数、程序包，甚至其他同义词。
同义词只是表的一个别名，因此对它的所有操作都会影响到表。

//创建私有同义词；
私有同义词名称不可与当前模式的对象名称相同
				
create synonym emp for scott.emp;
			
//创建公有同义词；
create public synonym emp for scott.emp;
			
注意：公有同义词名称可以与当前模式对象名称相同，但是当公有对象和本地对象具有相同名称时，本地对象优先。


drop synonym emp;
drop public synonym emp;
			
###  4.索引； 

系统会自动为主键加上索引
			
//创建普通索引
create index emp_i1 on emp(deptno);
			
//创建位图索引：如果取值范围很小而且是固定的，可以创建位图索引 
create bitmap index emp_i2 on emp(deptno);

//删除索引
drop index emp_i1;
			
5.数据字典查询	
		
//查询视图信息：
select * from user_views;
			 
//查询序列信息：
select * from user_sequences;
			 
//查询同义词信息：
select * from user_synonyms;
			 
//查询索引信息：
 select * from user_indexes;

查询当前数据名
方法一:select name from v$database;
方法二：show parameter db
方法三：查看参数文件。修改数据库名

http://www.2cto.com/database/201503/381809.html
http://wenku.baidu.com/link?url=Jb5jjdGirFwYvZdzSJxUNuCWMQm8V_7AHweXz_asLuo-F-ZI8eRZrJbciSLdjh-eG1hxtnMJ1B87k4S18bLQrGO_TtEM_2JrRvjCuuRNrG7
http://wenku.baidu.com/link?url=UFueZMU8Wvpp8DPd4aQPLQmEc5HBm6YBtiDaaBfoPQWVVWlm8mRpeWqy0wouC9Nh6vXYgYC6JgfddSuUidALyLYRU0qGn5x9jdabWjTzuLi&from_mod=copy_login###
