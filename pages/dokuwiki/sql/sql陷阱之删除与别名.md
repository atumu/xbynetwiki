title: sql陷阱之删除与别名 

#  sql陷阱之删除与别名 
删除记录时怎样给表取别名?取别名会报错。如:delete from User u ；就会报错
解决：
```

delete a
from A a
where not exists (
select 1
from B b
where A.id = b.aid 
)

```