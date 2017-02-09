title: autocodegen项目开发数据库兼容记录 

#  AutoCodeGen项目开发数据库兼容记录 
##  sql server 
SQLServer查询某数据库所有表的表名：
select name from sysobjects where xtype='U';

sqlserver查询某表的表结构：
SELECT syscolumns.name,systypes.name,syscolumns.isnullable, syscolumns.length  
FROM syscolumns, systypes  
WHERE syscolumns.xusertype = systypes.xusertype  AND syscolumns.id = object_id('表名')

查询主键：
SELECT 
```

tableName=case when a.colorder=1 then d.name else '' end,
feildIndex=a.colorder,
feildName=a.name
FROM syscolumns a
inner join sysobjects d on a.id=d.id  and d.xtype='U' and  d.name<>'dtproperties'
where exists(SELECT 1 FROM sysobjects where xtype='PK' and name in (
SELECT name FROM sysindexes WHERE indid in(
SELECT indid FROM sysindexkeys WHERE id = a.id AND colid=a.colid
)))
order by a.id,a.colorder

```

sqlserver oracle JDBC metadata相关方法返回空值：
ResultSetMetaData   metaData = resultSet.getMetaData();
String tableName = metaData.getTableName(1);
输出显示tableName等于空，为了解决此问题，我摆渡到一些相关资料
请看以下内容，我的问题是：怎样设置SQL Server 驱动程序的ResultSetMetaDataOptions 属性？
以下内容摘自http://edocs.weblogicfans.net/wls/docs92/jdbc_drivers/mssqlserver.html#wp1079514

性能注意事项 
设置 SQL Server 驱动程序的下列连接属性（如下列列表所述）可以提高应用程序的性能： 
ResultSetMetaDataOptions 
默认情况下，在调用 ResultSetMetaData.getTableName() 方法时，SQL Server 驱动程序跳过为结果集中的每列返回正确的表名所需的附加处理。正因为这一点，对于结果集中的每列，getTableName() 方法可能会返回空字符串。如果知道应用程序不需要表名信息，则此设置提供最佳性能。 
ResultSet 元数据支持 
如果应用程序需要表名信息，则 SQL Server 驱动程序可以在 Select 语句的 ResultSet 元数据中返回表名信息。通过将 ResultSetMetaDataOptions 属性设置为 1，SQL Server 驱动程序执行其他处理，以确定调用 ResultSetMetaData.getTableName() 方法时结果集中每列的正确表名。否则，getTableName() 方法可能对结果集中的每一列都返回空字符串。 
在 ResultSetMetaDataOptions 属性设置为 1 且调用了 ResultSetMetaData.getTableName() 方法时，SQL Server 驱动程序返回的表名信息取决于结果集中的列是否映射到数据库中表的列。对于结果集中映射到数据库表的列的每列，SQL Server 驱动程序返回与该列关联的表名。对于结果集中未映射到表列的列（例如，聚合和字面值），SQL Server 驱动程序返回空字符串。 

##  达梦 
查询所有表名
String dmStr="SELECT TABLE_NAME FROM ALL_TABLES where TABLESPACE_NAME='MAIN'"; / /达梦数据库
查询主键：
String sql="select u.COLUMN_NAME from user_cons_columns  u,ALL_CONSTRAINTS a where u.table_name='org_role' and u.CONSTRAINT_NAME=a.CONSTRAINT_NAME";

查询表结构：
String sql="select  NULLABLE,DATA_LENGTH,DATA_TYPE,DATA_LENGTH,COLUMN_NAME,' ' as COLUMN_COMMENT from user_tab_columns where Table_Name='org_role'";


