title: oracle与mysql在一些标准sql语法上的区别 

#  oracle与mysql在一些标准SQL语法上的区别 
1、不能使用limit,用rownum来代替
例如:
mysql代码
```

SELECT * FROM tablename LIMIT 100,15

```
改写成Oracle的语句:
首先，Oracle是不支持limit的,分页是是利用rownum实现的。
描述一下思路，采用mysql limit 20，30 查询第20至30条数据。用Oracle实现:
第一步，根据过滤条件先查询出前30条数据，这里用rownum记录一下顺序，注意起别名，后面用好。
第二步，在刚刚建立的结果集中查询第20条以后的数据，通过咱们起别名的那个字段。
```

 select * from 
     (select A.*,rownum rn from tablename A 
            where rownum > 30)
      where rn < 20

```