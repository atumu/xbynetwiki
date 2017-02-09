title: hibernate陷进之hql_is_not_mapped 

#  Hibernate陷进之hql Xxx is not mapped 
参考：http://www.tuicool.com/articles/NRfaEr
今天项目中使用hql查询时，出现    QingAoCenterInfo is not mapped [from QingAoCenterInfo where...]
显然是Hibernate映射关系出现了问题。
出现这种异常首先要查看查询语句中是否使用了数据库表中的表名，而不是实体类。
查看我的代码：
centerList = manager.find("from QingAoCenterInfo center where center.type = ? and center.centerName = ?", new Object[]{type,centerName});
**发现没有问题啊，百思不得其解，**从昨天下午到今天上午，捣鼓了好久好久。。。。。。。

最后发现了问题所在，**hql查询时使用的from Xxx，Xxx不是实体类的名称,也不是表名，而是 @Entity(name="而是这里指定的name")**
如：
```

@Entity
@Table(name="QING_AO_CENTER_INFO")
public class QingAoCenterInfo {
         ......
}

```
此处，
**@Entity后并没有显示的指明EntityName，因此默认采用实体类的名称。**

但是我的代码如下
```

@Entity(name="QING_AO_CENTER_INFO")
@Table(name="QING_AO_CENTER_INFO")
public class QingAoCenterInfo {
             ......
}

```
可以发现，**显示地指明了 EntityName，因此在使用hql查询的时候，要from   QING_AO_CENTER_INFO，**而不是from  QingAoCenterInfo ；
centerList = manager.find("from QING_AO_CENTER_INFO center where center.type = ? and center.centerName = ?", new Object[]{type,centerName});