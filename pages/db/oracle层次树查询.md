modifyAt:2016-12-21 17:08:04
title:Oracle层次树查询
location:db/oracle层次树查询
author:xbynet
createAt:2016-12-21 17:05:20

参考：http://blog.csdn.net/dba_waterbin/article/details/7994812
 层次查询使用树的遍历，走遍含树形结构的数据集合，来获取树的层次关系报表的方法
树形结构的父子关系，你可以控制：
① 遍历树的方向，是自上而下，还是自下而上
②  确定层次的开始点(root)的位置
 层次查询语句正是从这两个方面来确定的,start with确定开始点,connect by确定遍历的方向
 ![475e81d2-c75b-11e6-ab9f-00163e2eed34.png](/data/upload/475e81d2-c75b-11e6-ab9f-00163e2eed34.png)
  注释：
          ① level是伪列，表示等级
          ② from后面只能是一个表或视图，对于from是视图的，那么这个view不能包含join
          ③ Where条件限制了查询返回的行，但是不影响层次关系，属于将节点截断，但是这个被截断的节点的下层child不受影响
          ④ prior是个形容词，可放在任何地方
          ⑤ 彻底剪枝条件应放在connect by；单点剪掉条件应放在where子句。但是，connect by的优先级要高于where，也就是sql引擎先执行connect by
          ⑥ 在start with中表达式可以有子查询，但是connect by中不能有子查询
   
# 遍历树：
 ㈠ Start with子句
start with确定将哪行作为root，如果没有start with,则每行都当作root，然后查找其后代，这不是一个真实的查询。Start with后面可以使用子查询或者任何合法的条件表达式
 例子：
```
select level,id,manager_id,last_name,title from s_emp 
      start with title=(select title from s_emp where manager_id is null) 
      connect by prior id=manager_id; 
```

㈡ Connect by子句
Connect by与prior确定一个层次查询的条件和遍历的方向(prior确定)
 Connect by prior column_1=column_2;
其中prior表示前一个节点的意思，可以在connect by等号的前后，列之前，也可以放到select中的列之前
 Connect by也可以带多个条件，比如 connect by prior id=manager_id and id>10
 
**1. )自顶向下遍历：**
先由根节点，然后遍历子节点。column_1表示父key,column_2表示子key。即这种情况下：connect by prior 父key=子key表示自顶向下，等同于connect by 子key=prior 父key.
例子：
```
select level,employee_id,manager_id,last_name,job_id from s_emp 
      start with manager_id=100 
      connect by  employee_id=prior manager_id; 
```

**2. )自底向上遍历：**
先由最底层的子节点，遍历一直找到根节点。与上面的相反。Connect by之后不能有子查询，但是可以加其他条件,比如加上and id !=2等。这句话则会截断树枝,如果id=2的这个节点下面有很多子孙后代，则全部截断不显示。
例子：

```
select level,employee_id,manager_id,last_name,job_id from s_emp 
      start with manager_id=100 
      connect by prior employee_id=manager_id and employee_id<>120; 
```

# 我的应用：
查询某个机关下的所有子机关(包括自己)
```
select  JGMC,JG_DM  from dm_gy_jg  start with JG_DM='24404000000'  connect BY PRIOR  JG_DM=  SJJG_DM ;
```
# 使用level和lpad格式化报表：
Level是层次查询的一个伪列，如果有level，必须有connect by,start with可以没有
Lpad是在一个string的左边添加一定长度的字符，并且满足中间的参数长度要求，不满足自动添加
例子：

```
select level,employee_id,manager_id,lpad(last_name,length(last_name)+(level*4)-4,'_'),job_id from s_emp 
      start with manager_id=100 
      connect by prior employee_id=manager_id and employee_id<>120 

```

# 修剪branches：
 where子句会将节点删除，但是其后代不会受到影响，connect by 中加上条件会将满足条件的整个树枝包括后代都删除。要注意，如果是connect by之后加条件正好条件选到根，那么结果和没有加一样

# 实际应用
1）查询每个等级上节点的数目

```
  先查看总共有几个等级： 
select count(distinct level) 
from s_emp 
start with manager_id is null 
connect by prior employee_id=manager_id 
  要查看每个等级上有多少个节点，只要按等级分组，并统计节点的数目即可，可以这样写： 
select level,count(last_name) 
from s_emp 
start with manager_id is null 
connect by prior employee_id=manager_id 
group by level 
```
 2）查看等级关系
比如给定一个具体的员工看是否对某个员工有管理权

```
select level,a.* from  
s_emp a 
where first_name='Douglas' --被管理的节点 
start with manager_id is null --开始节点，即：根节点 
connect by prior employee_id=manager_id 
```

3）删除子树
比如有这样的需求，现在要裁员，将某个部门的员工包括经理全部裁掉
将id为2的员工管理的所有员工包括自己删除

```
delete from s_emp where employee_id in( 
elect employee_id from  
s_emp a 
start with employee_id=2 --从id=2的员工开始查找其子节点，把整棵树删除 
connect by prior employee_id=manager_id) 
```

4）找出每个部门的经理

```
select level,a.* from  
s_emp a 
start with manager_id is null 
connect by prior employee_id=manager_id and department_id !=prior department_id;--当前行的dept_id不等于前一行的dept_id，即每个子树中选最高等级节点 
```

5）查询一个组织中最高的几个等级

```
select level,a.* from  
s_emp a 
  where level <=2 –查找前两个等级 
start with manager_id is null 
connect by prior employee_id=manager_id and department_id !=prior department_id; 
```

6)合计层次
有两个需求，一是对一个指定的子树subtree做累加计算salary，一是将每行都作为root节点，然后对属于这个节点的所有子节点累加计算salary。

```
第一种很简单，求下sum就可以了，语句： 
select sum(salary) from  
s_emp a 
start with id=2—比如从id=2开始 
connect by prior id=manager_id; 
 第2个需求，需要用到第1个，对每个root节点求这个树的累加值，然后内部层次查询的开始节点从外层查询获得。 
select last_name,salary,( 
select sum(salary) from  
s_emp 
start with id=a.id –让每个节点都成为root 
connect by prior id=manager_id) sumsalary 
from s_emp a; 
```


 7）找出指定层次中的叶子节点
Leaf(叶子)就是没有子孙的孤立节点。Oracle 10g提供了一个简单的connect_by_isleaf=1,0表示非叶子节点

```
select level,id,manager_id,last_name, title from s_emp 
    where connect_by_isleaf=1 –表示查询叶子节点 
      start with  manager_id=2 
      connect by prior id=manager_id; 
```

  7 10g新特性：
① 使用SIBLINGS关键字排序
如果使用order by排序会破坏层次，在oracle10g中，增加了siblings关键字的排序
语法：order  siblings  by < expre>
它会保护层次，并且在每个等级中按expre排序
例子：

```
select level, 
       employee_id,last_name,manager_id 
       from s_emp 
       start with manager_id is null 
       connect by prior employee_id=manager_id 
       order siblings by last_name; 
```

 ② CONNECT_BY_ROOT
Oracle10g新增connect_by_root,用在列名之前表示此行的根节点的相同列名的值

```
select connect_by_root last_name root_last_name, connect_by_root employee_id root_id, 
      employee_id,last_name,manager_id 
      from s_emp 
      start with manager_id is null 
      connect by prior employee_id=manager_id 
```