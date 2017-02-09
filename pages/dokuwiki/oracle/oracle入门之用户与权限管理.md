title: oracle入门之用户与权限管理 

#  oracle入门之用户与权限管理 
权限允许用户访问属于其它用户的对象或执行程序，ORACLE系统提供三种权限：Object 对象级、System 系统级、Role 角色级。
这些权限可以授予给用户、特殊用户public或角色
如果授予一个权限给特殊用户"Public"（用户public是oracle预定义的，每个用户享有这个用户享有的权限）,那么就意味作将该权限授予了该数据库的所有用户。
` 系统权限管理 ` 
系统权限分类：
  * DBA: 拥有全部特权，是系统最高权限，只有DBA才可以创建数据库结构。
  * RESOURCE:拥有Resource权限的用户只可以创建实体，不可以创建数据库结构。
  * CONNECT:拥有Connect权限的用户只可以登录Oracle，不可以创建实体，不可以创建数据库结构。
对于普通用户：授予connect, resource权限。
对于DBA管理用户：授予connect，resource, dba权限。
系统权限授权命令：
**系统权限只能由DBA用户授出**：sys, system(最开始只能是这两个用户)
授权命令：SQL> grant connect, resource, dba to 用户名1 [,用户名2]...;
注:普通用户通过授权可以具有与system相同的用户权限，但永远不能达到与sys用户相同的权限，system用户的权限也可以被回收。 
例： 
SQL> connect system/manager
SQL> Create user user50 identified by user50;
SQL> grant connect, resource to user50;
查询用户拥有哪里权限： 
SQL> select * from dba_role_privs;
SQL> select * from dba_sys_privs;
SQL> select * from role_sys_privs; 
查自己拥有哪些系统权限
SQL> select * from session_privs; 
删除用户
SQL> drop user 用户名 cascade;  / / 加上cascade则将用户连同其创建的东西全部删除
系统权限传递：
增加WITH ADMIN OPTION选项，则得到的权限可以传递。
SQL> grant connect, resorce to user50 with admin option;  / /可以传递所获权限。
系统权限回收：系统权限只能由DBA用户回收
SQL> Revoke connect, resource from user50; 

**实体权限管理** 
实体权限分类
select, update, insert, alter, index, delete, all  //all包括所有权限
execute  //执行存储过程权限
user01:
SQL> grant select, update, insert on product to user02;
SQL> grant all on product to user02;
user02:
SQL> select * from user01.product; 
/ / 此时user02查user_tables，不包括user01.product这个表，但如果查all_tables则可以查到，因为他可以访问。
将表的操作权限授予全体用户：
SQL> grant all on product to public;  / / public表示是所有的用户，这里的all权限不包括drop。
实体权限数据字典
SQL> select owner, table_name from all_tables; / / 用户可以查询的表
SQL> select table_name from user_tables;  / /  用户创建的表
SQL> select grantor, table_schema, table_name, privilege from all_tab_privs; / / 获权可以存取的表（被授权的）
SQL> select grantee, owner, table_name, privilege from user_tab_privs;    / / 授出权限的表(授出的权限)


更多参考：
http://czmmiao.iteye.com/blog/1304934
http://www.ha97.com/4887.html