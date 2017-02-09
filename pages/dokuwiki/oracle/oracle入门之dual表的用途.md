title: oracle入门之dual表的用途 

#  oracle入门之dual表的用途 
dual是一个虚拟表，用来构成select的语法规则，oracle保证dual里面永远只有一条记录。我们可以用它来做很多事情。
如下：
　　1、查看当前用户，可以在 SQL Plus中执行下面语句 select user from dual;
　　2、用来调用系统函数
　　select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') from dual;--获得当前系统时间
　　select SYS_CONTEXT('USERENV','TERMINAL') from dual;--获得主机名
　　select SYS_CONTEXT('USERENV','language') from dual;--获得当前 locale
　　select dbms_random.random from dual;--获得一个随机数
　　4、可以用做计算器 select 7*9 from dual;
dual 确实是一张表.是一张只有一个字段,一行记录的表. 
如果我们不需要从具体的表来取得表中数据,而是单纯地为了得到一些我们想得到的信息,并要通过select 完成时,就要借助一个对象,这个对象,就是dual;
当然,我们不一定要dual ,也可以这样做.例如:
create table mydual( dummy varchar2(1));
也可以实现和dual 同样的效果:
select 999*999 from mydual;
不过,dual 我们都用习惯了,就无谓自己再搞一套了.
