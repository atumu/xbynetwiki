title: oracle使用merge_into同时更新_插入数据 

#  Oracle使用MERGE INTO同时更新、插入数据 
MERGE语句是Oracle9i新增的语法，用来合并UPDATE和INSERT语句。
通过MERGE语句，根据一张表或子查询的连接条件对另外一张表进行查询，
**连接条件匹配上的进行UPDATE，无法匹配的执行INSERT**。
**这个语法仅需要一次全表扫描就完成了全部工作，执行效率要高于INSERT＋UPDATE。** 
語法：
MERGE [INTO [schema .] table [t_alias] 
USING [schema .] { table | view | subquery } [t_alias] 
ON ( condition ) 
WHEN MATCHED THEN merge_update_clause 
WHEN NOT MATCHED THEN merge_insert_clause;

```

--全部男生记录
create table fzq1 as select * from fzq where sex=1;
--全部女生记录
create table fzq2 as select * from fzq where sex=0;
/*涉及到两个表关联的例子*/
--更新表fzq1使得id相同的记录中chengji字段＋1，并且更新name字段。
--如果id不相同，则插入到表fzq1中.
--将fzq1表中男生记录的成绩＋1，女生插入到表fzq1中
merge into fzq1  aa     --fzq1表是需要更新的表
using fzq bb            -- 关联表
on (aa.id=bb.id)        --关联条件
when matched then       --匹配关联条件，作更新处理
update set
aa.chengji=bb.chengji+1,
aa.name=bb.name         --此处只是说明可以同时更新多个字段。
when not matched then    --不匹配关联条件，作插入处理。如果只是作更新，下面的语句可以省略。
insert values( bb.id, bb.name, bb.sex,bb.kecheng,bb.chengji);
--可以自行查询fzq1表。
/*涉及到多个表关联的例子，我们以三个表为例，只是作更新处理，不做插入处理。当然也可以只做插入处理*/
--将fzq1表中女生记录的成绩＋1，没有直接去sex字段。而是fzq和fzq2关联。
merge into fzq1  aa     --fzq1表是需要更新的表
using (select fzq.id,fzq.chengji 
       from fzq join fzq2
       on fzq.id=fzq2.id) bb  -- 数据集
on (aa.id=bb.id)        --关联条件
when matched then       --匹配关联条件，作更新处理
update set
aa.chengji=bb.chengji+1
--可以自行查询fzq1表。

```
```

merge into t_dd_etl t
    using (select gv_dd_etl.task_uuid as task_uuid from dual) t1
    on ( t.task_uuid = t1.task_uuid)
    when matched then
      update set t.task_name = gv_dd_etl.task_name,
                 t.dblink = gv_dd_etl.dblink,
                 t.source_owner = gv_dd_etl.source_owner,
                 t.source_table = gv_dd_etl.source_table,
                 t.targer_owner = gv_dd_etl.targer_owner,
                 t.targer_table = gv_dd_etl.targer_table,
                 t.tablespaces = gv_dd_etl.tablespaces,
                 t.col_time = gv_dd_etl.col_time,
                 t.where_sql = gv_dd_etl.where_sql,
                 t.bz = gv_dd_etl.bz
    when not matched then
      insert (TASK_UUID				,
              SCHEDULE_UUID   ,
              TASK_NAME       ,
              SOURCE_OWNER    ,
              SOURCE_TABLE    ,
              DBLINK          ,
              TARGER_OWNER    ,
              TARGER_TABLE    ,
              ETL_TYPE        ,
              ETL_TIME        ,
              WHERE_SQL       ,
              COL_TIME        ,
              YXBZ            ,
              BZ              ,
              LOG_UUID        ,
              LRR_DM          ,
              LRSJ            ,
              TASK_DM         ,
              LIST_UUID       ,
              TABLESPACES    ,
              FUN_NAME)
              values (
              gv_dd_etl.TASK_UUID				,
              gv_dd_etl.SCHEDULE_UUID   ,
              gv_dd_etl.TASK_NAME       ,
              gv_dd_etl.SOURCE_OWNER    ,
              gv_dd_etl.SOURCE_TABLE    ,
              gv_dd_etl.DBLINK          ,
              gv_dd_etl.TARGER_OWNER    ,
              gv_dd_etl.TARGER_TABLE    ,
              gv_dd_etl.ETL_TYPE        ,
              gv_dd_etl.ETL_TIME        ,
              gv_dd_etl.WHERE_SQL       ,
              gv_dd_etl.COL_TIME        ,
              gv_dd_etl.YXBZ            ,
              gv_dd_etl.BZ              ,
              gv_dd_etl.LOG_UUID        ,
              gv_dd_etl.LRR_DM          ,
              gv_dd_etl.LRSJ            ,
              gv_dd_etl.TASK_DM         ,
              gv_dd_etl.LIST_UUID       ,
              gv_dd_etl.TABLESPACES     ,
              gv_dd_etl.fun_name
              );
   commit;

```