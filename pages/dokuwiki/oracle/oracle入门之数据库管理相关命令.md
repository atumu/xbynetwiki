title: oracle入门之数据库管理相关命令 

#  oracle入门之数据库管理相关命令 
sqlplus本地sys用户登录:
用SYS用户登陆必须加上AS SYSDBA
sqlplus sys/rabbit2016 as sysdba

本地导入导出：
导出文件    修改链接地址   将文件DB_FXXT.DMP拷贝到dumpfile目录下
expdp sys/oracleoracle@testl schemas=DB_FXXT dumpfile=DB_FXXT.DMP DIRECTORY=data_pump_dir
导入文件
impdp system/1234@localhost_orcl remap_schema=DB_FXXT:DB_FXXT directory=data_pump_dir dumpfile=DB_FXXT.DMP logfile=db_testl.log

查询相关信息
```

select  * from dba_users x where x.account_status ='OPEN';
SELECT * FROM V$DATAFILE;
SELECT * FROM V$DATABASE ;
SELECT * FROM DBA_DIRECTORIES
select * from dba_objects x where x.OWNER = 'DB_FXXT'

```

用户管理：
删除用户
drop user test cascade;
创建用户
CREATE USER "test" IDENTIFIED BY VALUES '1234' DEFAULT TABLESPACE "TS_DATA" TEMPORARY TABLESPACE "TEMP";
给对象权限、修改密码 
grant dba,connect,resource to test;
alter user test identified by 4321;

表空间管理:http://blog.csdn.net/starnight_cbj/article/details/6792364
创建表空间
```

CREATE TABLESPACE "TS_SJJG_DATA" DATAFILE
     'D:\ORACLE\ORADATA\ORCL\DB_SJJG01_DATA.DBF' SIZE 400m REUSE
     LOGGING ONLINE PERMANENT BLOCKSIZE 8192
     EXTENT MANAGEMENT LOCAL UNIFORM SIZE 4m SEGMENT SPACE MANAGEMENT AUTO

```
修改已有表空间大小     
alter database datafile 'D:\ORACLE\ORADATA\ORCL\DB_SJJG01_DATA.DBF' resize 4000M;

查看存储过程:
select * from user_procedures 
查看存储过程的定义:
user_source,dba_source,这个表里面有你需要的
select text from user_source where type='PROCEDURE' and name='PROCEDURE_NAME';


