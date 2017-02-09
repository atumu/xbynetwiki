title: 高级sql 

#   SQLite中的高级SQL 
Created Tuesday 10 March 2015

主要介绍：insert update delete语句，保护数据的约束，以及更高级的主题。

#  修改数据 
insert update delete

##  插入记录： 
insert into foods (name, type_id) values ('Cinnamon Bobka', 1);
select max(id) from foods;
select last_insert_rowid(); 返回最后一行的id即最大值

###  插入一行 
insert into foods values(NULL, 1, 'Blueberry Bobka');
select * from foods where name like '%Bobka';
.schema foods

###  使用子查询插入一组行 
```

insert into foods values(null,
			(select id from food_types where name='bakery'),
			'Blackberry');
			
insert into foods select last_insert_rowid()+1 ,type_id,name from foods where name='Chocolate Bobka';

```
###  插入多行 
使用select形式的insert可以一次插入多行。只要字段匹配。insert可以插入结果集中的所有行。
```

create table foods2 (id int, type_id int, name text);
insert into foods2 select * from foods;
select count(*) from foods2;
或者更直接的create table foods2 **as** select * from foods;
这种方式用于创建临时表特别有用。
create temp table list as select f.name food,t.name name, (select count(episode_id) from foods_episodees where food_id=f.id) episodes 
from foods f , food_types t where f.type_id=t.id;
select * from list;

```
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054721.png)

##  更新记录 
update命令用于更新表中的记录
update foods set name='CHOCOLATE　BOBKA'　where name='CHOCOLATE';

##  删除记录 
delete from table where name='colcolate';

#  数据完整性 
数据完整性是通过约束实现的。
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054751.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054757.png)
```

create table contacts(
	id integer primary key,
	name text not null collate nocase,
	phone text not null default 'UNKNOWN',
	unique(name,phone)
);

```
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054806.png)

##  实体完整性 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054812.png)

###  唯一性约束 
unique

###  主键约束 
主键内部别名rowid , oid等
select rowid, oid,_rowid_,id, name, phone from contacts;
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054825.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054829.png)

create table maxed_out(id integer primary key autoincrement, x text);

####  复合主键约束 
create table pkey(x text, y text, primary key(x,y));

##  域完整性 
最简单的域完整性定义就是字段的值遵从赋予的规定。域处理两件事情：类型和范围。
域完整性有两个组成部分：类型检查和范围检查。
default; not null ; check; collate nocase排序规则

###  默认值default： 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054838.png)
```

create table times ( id int,
  date not null default current_date,
  time not null default current_time,
  timestamp not null default current_timestamp );

```
###  NOT NULL约束 

###  check约束 
check约束允许定义表达式来测试要插入或者更新的字段值。
```

create table contacts
( id integer primary key,
name text not null collate nocase,
phone text not null default 'UNKNOWN',
unique (name,phone),
check (length(phone)>=7) );

create table foo
( x integer,
y integer check (y>x),
z integer check (z>abs(y)) );

```
###  外键约束 
关系完整性也叫外键。它确保了一个表中的关键值必须从另一个表中引用。
```

create table foods(
  id integer primary key,
  type_id integer references food_types(id)
  on delete restrict
  deferrable initially deferred,
  name text );
  </code>
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054920.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054925.png)

###  排序规则 
collate
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054930.png)

#  存储类 
SQLite有5种原始的数据类型称为存储类型。存储类就是指值在磁盘上存储的格式。
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-054937.png)
SQL函数typeof()函数可以返回存储类型。
<code sql>
select typeof(3.14), typeof('3.14'),
	   typeof(314), typeof(x'3142'), typeof(NULL);
 </code>

#  视图 
视图即虚拟表或者拍深表。因为他们的内容来源于其他表的查询结果。与基本表的区别，是否为持久性存储。
<code sql>
create view details as
select f.name as fd, ft.name as tp, e.name as ep, e.season as ssn
from foods f
inner jonin foods_tyoes on f.type_id-ft.id
inner join foods_episodes fe on f.id=fe.food_id
inner join episodes e on fe.episode_id=e.id;

```
注意：SQLite不支持可更新的视图
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055010.png)

#  索引 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055017.png)
create index foods_name_idx on foods(name collate nocase);
.indices foods查看索引信息。
.schema foods 查看schema信息

#  触发器 
当具体的表发生特定的数据库事件时，触发器执行对应的SQL命令。创建触发器的命令：
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055026.png)

##  更新触发器 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055033.png)
```

create temp trigger foods_update_log after update of name on foods
begin
  insert into log values('updated foods: new name=' || new.name);
end;

begin;
update foods set name='JUJYFRUIT' where name='JujyFruit';
rollback;

```
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055049.png)

##  使用触发器构建可更新的视图 
```

create view foods_view as
  select f.id fid, f.name fname, t.id tid, t.name tname
  from foods f, food_types t;

create trigger on_update_foods_view
instead of update on foods_view
for each row
begin
   update foods set name=new.fname where id=new.fid;
   update food_types set name=new.tname where id=new.tid;
end;

```
#  事务 
事务的原子性原则

##  事务的范围 
事务由三个命令控制：begin,commit,rollback
默认每条语句都是自动提交模式的事务。
SQLite也支持savepoint与release命令。回滚可以返回到某个savepoint而不用返回整个事务。
savepoint point1;
rollback to point1;

##  冲突解决 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055110.png)
记得加上前缀or
</code sql>
create unique index test_idx on test(id);
alter table test add column modified text not null default 'no';
select count(*) from test where modified='no';

update or fail test set id=800-id, modified='yes';

select count(*) from test where modified='yes';

drop table test;
在表内定义时，可以为单个字段指定冲突解决办法
create temp table cast(name text unique on conflict rollback);
insert into cast values ('Jerry');
insert into cast values ('Elaine');
insert into cast values ('Kramer');

begin;
insert into cast values('Jerry');
commit;
</code>
##  数据库锁 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055134.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055152.png)

##  事务类型 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055209.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055219.png)

#  数据库管理 
attach附加,
pragma编译指示

###  附加数据库 
attach database '/tmp/db' as db2;
select * from db2;
detach database db2;

###  数据库清理 
reindex; vacuum； auto_vacuum
vacuum重构数据库文件来清理那些未使用的空间。

###  数据库配置 
SQLite没有配置文件，通过pragma编译指示可以进行配置。
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055324.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055433.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055358.png)

##  系统目录 
sqlite_master表
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055532.png)

#  查看查询计划 
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055549.png)
![](/data/dokuwiki/booknote/sqlit_tutorial_2th/pasted/20150521-055601.png)
