title: oracle入门表相关操作 

#  oracle入门表相关操作 
创建表：
```

-- Create table
create table db_fxxt.T_DD_ETL_LOG
(
  log_uuid   VARCHAR2(32) not null,
  task_uuid  VARCHAR2(32) not null,
  begin_time DATE,
  end_time   DATE,
  flag       CHAR(1) not null,
  log        VARCHAR2(1000),
  nums       NUMBER(20) default 0 not null,
  err_nums   NUMBER(20) default 0 not null,
  zxr_dm     VARCHAR2(11),
  etl_time   TIMESTAMP(6)
)
 
   ;
-- Add comments to the table 
comment on table db_fxxt.T_DD_ETL_LOG
  is 'ETL任务日志表';
-- Add comments to the columns 
comment on column db_fxxt.T_DD_ETL_LOG.log_uuid
  is 'UUID';
comment on column db_fxxt.T_DD_ETL_LOG.task_uuid
  is 'T_DD_TASK.TASK_UUID';
comment on column db_fxxt.T_DD_ETL_LOG.begin_time
  is '开始时间';
comment on column db_fxxt.T_DD_ETL_LOG.end_time
  is '终止时间';
comment on column db_fxxt.T_DD_ETL_LOG.flag
  is '状态(t_dm_dd中dm_type为FLAG)';
comment on column db_fxxt.T_DD_ETL_LOG.log
  is '日志';
comment on column db_fxxt.T_DD_ETL_LOG.nums
  is '抽取数据量';
comment on column db_fxxt.T_DD_ETL_LOG.err_nums
  is '错误数据量';
comment on column db_fxxt.T_DD_ETL_LOG.zxr_dm
  is '执行人员代码';
-- Create/Recreate primary, unique and foreign key constraints 
alter table db_fxxt.T_DD_ETL_LOG
  add constraint PK_T_DD_ETL_LOG primary key (LOG_UUID)
  using index 
   ; 


```