title: oracle优化之bulk_collect与forall 

#  oracle优化之BULK COLLECT与FORALL 
Oracle8i中首次引入了Bulk Collect特性，该特性可以让我们在PL/SQL中能使用批处理，批查询在某些情况下能显著提高批处理效率。

PL/SQL程序中运行SQL语句是存在开销的，因为SQL语句是要提交给SQL引擎处理。这种在PL/SQL引擎和SQL引擎之间的控制转移叫做上下文切换，每次切换时，都有额外的开销。
![](/data/dokuwiki/oracle/pasted/20160926-140114.png)
但是，FORALL和BULK COLLECT可以让PL/SQL引擎把多个上下文却换压缩成一个，这使得在PL/SQL中的要处理多行记录的SQL语句执行的花费时间骤降
![](/data/dokuwiki/oracle/pasted/20160926-140131.png)

㈠ 通过BULK COLLECT 加速查询
⑴ BULK COLLECT 的用法
**采用BULK COLLECT可以将查询结果一次性地加载到collections中，而不是通过cursor一条一条地处理**。
可以在select into ，fetch into ， returning into语句使用BULK COLLECT。
注意在使用BULK COLLECT时，` 所有的INTO变量都必须是collections `

举几个简单例子：
① 在select into语句中使用bulk collect
DECLARE 
TYPE sallist **IS TABLE OF** employees.salary%TYPE;
sals sallist;
BEGIN
SELECT salary **BULK COLLECT** INTO sals FROM employees where rownum<=50;
--接下来使用集合中的数据
END;

② 在fetch into中使用bulk collect
```

DECLARE
TYPE deptrectab IS TABLE OF departments%ROWTYPE;
dept_recs deptrectab;
CURSOR cur IS SELECT department_id,department_name FROM departments where department_id>10;
BEGIN
OPEN cur;
FETCH cur BULK COLLECT INTO dept_recs;
--接下来使用集合中的数据

```
END;


③ 在returning into中使用bulk collect
```

CREATE TABLE emp AS SELECT * FROM employees;
DECLARE 
TYPE numlist IS TABLE OF employees.employee_id%TYPE;
enums numlist;
TYPE namelist IS TABLE OF employees.last_name%TYPE;
names namelist;
BEGIN
DELETE emp WHERE department_id=30
RETURNING employee_id,last_name BULK COLLECT INTO enums,names;
DBMS_OUTPUT.PUT_LINE('deleted'||SQL%ROWCOUNT||'rows:');
FOR i IN enums.FIRST .. enums.LAST
LOOP
DBMS_OUTPUT.PUT_LINE('employee#'||enums(i)||':'||names(i));
END LOOP;
END;

```


⑵ BULK COLLECT 对大数据DELETE UPDATE的优化
这里举DELETE就可以了，UPDATE同理
举个案例：
需要在一个1亿行的大表中，删除1千万行数据
需求是在对数据库其他应用影响最小的情况下，以最快的速度完成

如果业务无法停止的话，可以参考下列思路：
根据ROWID分片、再利用Rowid排序、批量处理、回表删除，在业务无法停止的时候，选择这种方式，的确是最好的
一般可以控制在每一万行以内提交一次，不会对回滚段造成太大压力，我在做大DML时，通常选择一两千行一提交，选择业务低峰时做，对应用也不至于有太大影响
代码如下：
```

DECLARE
--按rowid排序的cursor
--删除条件是oo=xx，这个需根据实际情况来定
CURSOR mycursor IS SELECT rowid FROM t WHERE OO=XX ORDER BY rowid;
TYPE rowid_table_type IS TABLE OF rowid index by pls_integer;
v_rowid rowid_table_type;
BEGIN
OPEN mycursor;
LOOP
FETCH mycursor BULK COLLECT INTO v_rowid LIMIT 5000;--5000行提交一次
EXIT WHEN v_rowid.count=0;
FORALL i IN v_rowid.FIRST..v_rowid.LAST
DELETE t WHERE rowid=v_rowid(i);
COMMIT;
END LOOP;
CLOSE mycursor;
END;

```

⑶ 限制BULK COLLECT 提取的记录数
语法：
FETCH cursor BULK COLLECT INTO ...[LIMIT rows]；
其中，rows可以是常量，变量或者求值的结果是整数的表达式

假设你需要查询并处理1W行数据，你可以用BULK COLLECT一次取出所有行，然后填充到一个非常大的集合中
可是，这种方法会消耗该会话的大量PGA，APP可能会因为PGA换页而导致性能下降

这时，LIMIT子句就非常有用，它可以帮助我们控制程序用多大内存来处理数据

例子：

```

DECLARE
CURSOR allrows_cur IS SELECT * FROM employees;
TYPE employee_aat IS TABLE OF allrows_cur%ROWTYPE INDEX BY BINARY_INTEGER;
v_emp employee_aat;
BEGIN
OPEN allrows_cur;
LOOP
FETCH allrows_cur BULK FETCH INTO v_emp LIMIT 100;

/*通过扫描集合对数据进行处理*/
FOR i IN 1 .. v_emp.count
LOOP
upgrade_employee_status(v_emp(i).employee_id);
END LOOP;

EXIT WHEN allrows_cur%NOTFOUND;
END LOOP;

CLOSE allrows_cur;
END;


```
⑷ 批量提取多列

需求：
提取transportation表中的油耗小于 20公里/RMB的交通具体的全部信息
代码如下：
复制代码 代码如下:

DECLARE
--声明集合类型
TYPE vehtab IS TABLE OF transportation%ROWTYPE;
--初始化一个这个类型的集合
gas_quzzlers vehtab;
BEGIN
SELECT * BULK COLLECT INTO gas_quzzlers FROM transportation WHERE mileage < 20;
...

⑸ 对批量操作使用RETURNING子句

有了returning子句后，我们可以轻松地确定刚刚完成的DML操作的结果，无须再做额外的查询工作
例子请见BULK COLLECT 的用法的第三小点


㈡ 通过FORALL 加速DML

FORALL告诉PL/SQL引擎要先把一个或多个集合的所有成员都绑定到SQL语句中，然后再把语句发送给SQL引擎


Bulk Collect批查询在某种程度上可以提高查询效率，它首先将所需数据读入内存，然后再统计分析，这样就可以提高查询效率。但是，如果Oracle数据库的内存较小，Shared Pool Size不足以保存Bulk Collect批查询结果，那么该方法需要将Bulk Collect的集合结果保存在磁盘上，在这种情况下，Bulk Collect方法的效率反而不如。
另外，除了Bulk Collect批查询外，我们还可以使用FORALL语句来实现批插入、删除和更新，这在大批量数据操作时可以显著提高执行效率。

http://www.cnblogs.com/rootq/archive/2008/11/17/1335491.html
http://www.jb51.net/article/35424.htm