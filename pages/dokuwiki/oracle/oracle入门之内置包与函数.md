title: oracle入门之内置包与函数 

#  oracle入门之内置包与函数 
Oracle 10g提供的系统包多达几百个，此处只介绍一些常用的系统包。
Oracle 一些内置的程序包
  * STANDARD和DBMS_STANDARD 定义和扩展PL/SQL语言环境
  * dbms_metadata:获取数据库的一些DDL、元数据等信息
  * DBMS_LOG 提供对 Oracle LOB数据类型进行操作的功能
  * DBMS_LOCK 用户定义的锁
  * DBMS_OUTPUT 处理PL/SQL 块和子程序**输出调试信息**
  * DBMS_SESSION 提供ALTER SESSION 命令的PL/SQL等效功能
  * DBMS_ROWID 获得**ROWID** 的详细信息
  * DBMS_RANDOM 提供**随机生成器**
  * DBMS_SQL 允许用户**使用动态SQL**,构造和执行任意DML和DDL语句
  * DBMS_SCHEDULER/JOB 提交和管理在数据库中执行的**定时任务**
  * DBMS_XMLDOM 用DOM模型读写XML类型的数据
  * DBMS_XMLPARSER XML解析,处理XML文档内容和结构
  * DBMS_XMLGEN 将SQL 查询结果转换为规范的**XML**格式
  * DBMS_XMLQUERY 提供将数据转换为XML类型的功能 DBMS_XSLPROCESSOR 提供XSLT功能. 转换XML文档
  * UTL_FILE 用PL/SQL程序来**读写操作系统文本文件**
  * DBMS_PIPE:用于在同一例程(实例)的不同会话之间进行通信。比如连接到同一个数据库的两个独立会话之间可以通过管道进行通信，另外也可以在存储过程和Pro*C之间进行通信，这样就大大地增强了PL/SQL的处理能力。

例如：
##  一、dbms_metadata.get_ddl:生成数据库对象的ddl信息: 

![](/data/dokuwiki/oracle/pasted/20160920-105023.png)
##  一、DBMS_OUTPUT 

作用：用于输入和输出信息,使用过程PUT和PUT_LINES可以将信息发送到缓冲区,使用过程GET_LINE和GET_LINES可以显示缓冲区信息。
该包用来输出plsql变量的值，属于系统用户sys。下面讲述包的组成：
ENABLE
说明：该过程用于激活本包，如果没有被激活，将无法调用本包的其它其余过程和函数。当调用该过程，缓冲区最大尺寸为1000000字节，最小为2000字节，默认为20000字节。
注意：如果在SQL*PLUS中使用SERVEROUTPUT选项，则没有必要使用该过程。
语法：DBMS_OUTPUT.ENABLE（buffer_size in integer default 20000）;
DISABLE
说明：该过程用于禁止本包，并清除缓冲区的内容。当本包被禁止，将无法调用本包的其它其余过程和函数。
注意：如果在SQL*PLUS中使用SERVEROUTPUT选项，则没有必要使用该过程。
语法：DBMS_OUTPUT.DISABLE;
put和put_line
说明：过程put_line用于将一个完整行的信息写入到缓冲区中，会自动在行的尾部追加行结束符；
过程put则用地分块建立行信息，需要换行需要使用过程new_line追加行结束符。
语法：dbms_output.put(item in number\varchar2\date);dbms_output.put_line(item in number\varchar2\date);

当在sql*plus中使用包过程put、put_line时，需要设置serveroutput选项。
例子：
```

set serveroutput on
begin
dbms_output.put_line('伟大的中华民族');
dbms_output.put('中国');
dbms_output.put(',伟大的祖国');
dbms_output.new_line;
end;

```

##  二、定时任务DBMS_SCHEDULER 
http://blog.csdn.net/fw0124/article/details/6753715
http://www.cnblogs.com/lanzi/archive/2012/11/23/2784815.html
Oracle 10g之前，可以使用dbms_job来管理定时任务。
10g之后，Oracle引入dbms_scheduler来替代先前的dbms_job,
**使用dbms_scheduler创建一个定时任务有两种形式**
1）创建1个SCHEDULER来定义计划，1个PROGRAM来定义任务内容，
再创建1个JOB，为这个JOB指定上面的SCHEDULER和PROGRAM。
2）直接创建JOB，在参数里面直接指定计划和任务内容。

###  1. 创建job 
job是什么呢? 简单的说就是计划(schedule)加上任务说明. 另外还有一些必须的参数.
这里提到的**"任务"可以是数据库内部的存储过程,匿名的PL/SQL块,也可以是操作系统级别的脚本.**
可以有两种方式来定义"计划":
1) 使用DBMS_SCHDULER.CREATE_SCHEDULE 定义一个计划;
2) 调用DBMS_SCHDULER.CREATE_JOBE过程直接指定 (下面会详细说明)
在创建一个计划时，你至少需要指定下面的属性，它们是job运行所必须的:
  * 开始时间 (start_time);
  * 重复频率 (repeat_interval);
  * 结束时间 (end_time)
**Q1:怎么从数据库中查询job的属性 ?**
A1: 有两种方法:
1) 查询(DBA|ALL|USER)_SCHEDULER_JOBS 视图
(提示: 根据用户权限的不同，选择性的查询 DBA|ALL|USER视图)
2) 调用DBMS_SCHEDULER包中的GET_ATTRIBUTE 过程
**Q2: 怎么设置这些属性呢?**
A2: 也是有两种方法
1) 在创建job时直接指定
2) 调用DBMS_SCHEDULER包中的SET_ATTRIBUTE 过程
**Q3: "我需要什么权限才能创建job" ?**
A3: 你至少需要create_job这个系统权限。如果用户拥有create any job这个权限，
它可以创建属主为任何用户(SYS用户除外)的job.
缺省情况下,job会被创建在当前的schema下，并且是没有激活的; 如果要使job一创建
就自动激活，需要显式的设置enabled 属性为true, 来看一个例子:
```

begin
dbms_scheduler.create_job
(
job_name => 'ARC_MOVE',
schedule_name => 'EVERY_60_MINS',
job_type => 'EXECUTABLE',
job_action => '/home/dbtools/move_arcs.sh',
enabled => true,
comments => 'Move Archived Logs to a Different Directory'
);
end;

```
**Q4: 能不能详细地讲述一下上面这个过程用到的各个参数?**
A4:
job_name: 顾名思义,每个job都必须有一个的名称
schedule_name: 如果定义了计划，在这里指定计划的名称
job_type: 目前支持三种类型:
  * PL/SQL块: PLSQL_BLOCK,
  * 存储过程: STORED_PROCEDURE
  * 外部程序: EXECUTABLE (外部程序可以是一个shell脚本,也可以是操作系统级别的指令).
job_action: 根据job_type的不同，job_action有不同的含义.
  * 如果job_type指定的是存储过程，就需要指定存储过程的名字;
  * 如果job_type指定的是PL/SQL块，就需要输入完整的PL/SQL代码;
  * 如果job_type指定的外部程序，就需要输入script的名称或者操作系统的指令名

enabled: 上面已经说过了，指定job创建完毕是否自动激活
comments: 对于job的简单说明

###  2. 指定job的执行频率 
如果我们创建了一个job,并且希望它按照我们指定的日期和时间来运行，就需要定义
job的重复频度了. 例如每天运行，每周日的22:00运行, 每周一,三,五运行，每年的
最后一个星期天运行等等.
10G 支持两种模式的repeat_interval,一种是PL/SQL表达式，这也是dbms_job包
中所使用的,例如SYSDATE+1, SYSDATE + 30/24*60; 另一种就是日历表达式。
例如MON表示星期一,SUN表示星期天,DAY表示每天,WEEK表示每周等等. 下面来
看几个使用日历表达式的例子：
repeat_interval => 'FREQ=HOURLY; INTERVAL=2'
每隔2小时运行一次job
repeat_interval => 'FREQ=DAILY'
每天运行一次job
repeat_interval => 'FREQ=WEEKLY; BYDAY=MON,WED,FRI"
每周的1,3,5运行job
repeat_interval => 'FREQ=YEARLY; BYMONTH=MAR,JUN,SEP,DEC; BYMONTHDAY=30'
每年的3,6,9,12月的30号运行job

日历表达式基本分为三部分: 第一部分是频率，也就是"FREQ"这个关键字，
它是必须指定的; 第二部分是时间间隔，也就是"INTERVAL"这个关键字，
取值范围是1-999. 它是可选的参数; 最后一部分是附加的参数,可用于
精确地指定日期和时间,它也是可选的参数,例如下面这些值都是合法的:
BYMONTH,BYWEEKNO,BYYEARDAY,BYMONTHDAY,BYDAY
BYHOUR,BYMINUTE,BYSECOND

###  3. 创建程序 (program) 
什么是程序? 我的理解就是准备计划需要的元数据(metadata),它
包括以下部分:
程序名;
程序中用到的参数: 例如程序的类型，以及具体操作的描述
来看一个例子
begin
dbms_scheduler.create_program(
program_name=> 'DAILY_BACKUP_SH',
program_type=> 'EXECUTABLE',
program_action=> '/home/oracle/script/daily_backup.sh');
end;
Q1: 程序和作业相比，有什么区别呢?
A1: 程序其实是可以与作业分离的，因此不同的用户可以在不同的时间段去重用它.而一个作业是属于特定的用户的;
Q2: 能否解释一下 create_program与create_job的关系?
A2: 首先，你应该知道创建程序并不是一个计划的必须组成部分，一个计划可以没有程序,但是必须有一个已经定义好的作业;
另外,program_action这个参数也是可选的，假如程序的类型是pl/sql 块,你完全可以在创建作业时来指定它.

###  5. 创建计划(Schedule) 
其实，如果你已经了解了怎样创建作业和程序，就等于已经掌握怎样创建计划了。你需要做的附加工作只是指定计划的开始时间，结束时间，重复频率等等.
来看一个例子
begin
dbms_scheduler.create_job(
job_name=> 'leo.UPDATE_STATS_JOB',
program_name=> 'leo.UPDATE_STATS_2',
start_date=>'2005-06-20 11:00.00.000000 PM +8:00',
repeat_interval=>'FREQ=MONTHLY;INTERVAL=1',
end_date=>'2006-06-20 11:00.00.000000 PM +8:00',
comments=>'Monthly statistics collection job');
end;

start_date和end_date这两个参数需要说明一下: 它们的数据类型
是timestamp，因此需要精确的指定时间和时区. 时间格式继承的是NLS_DATE_FORMAT这个初始化参数的值.
参考：http://blog.csdn.net/bbliutao/article/details/10462919
http://blog.itpub.net/26770925/viewspace-1243303/
http://blog.itpub.net/22521389/viewspace-763746