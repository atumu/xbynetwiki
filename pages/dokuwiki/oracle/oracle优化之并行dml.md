title: oracle优化之并行dml 

#  oracle优化之并行DML 
如果需要查询语句及INSERT 语句都使用并行，那么必须运行以下命令，否则只有查询语句使用到
并行， INSERT 语句使用不到。 
alter session enable parallel dml;
实际上只有对于insert into … select … 这样的SQL语句启用并行才有意义。 对于insert into .. values… 并行没有意义，因为这条语句本身就是一个单条记录的操作。
Insert 并行常用的语法是：Insert /*+parallel(t 2) */ into t select /*+parallel(t1 2) */ * from t1;
这条SQL 语句中，可以让两个操作insert 和select 分别使用并行，这两个并行是相互独立，互补干涉的，也可以单独使用其中的一个并行。
最后注意要把session的并行度关掉：alter session disable parallel dml;

参考:http://www.cnblogs.com/xd502djj/archive/2012/04/26/2471526.html
http://blog.csdn.net/luckyman100/article/details/9103853