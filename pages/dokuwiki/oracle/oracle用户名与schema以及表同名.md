title: oracle用户名与schema以及表同名 

#  oracle用户名与Schema以及表同名 
oracle不同表空间中的表名能否重复?
schema为数据库对象的集合，为了区分各个集合，我们需要给这个集合起个名字，这些名字就是我们在企业管理器的方案下看到的许多类似用户名的节点，
这些**类似用户名的节点其实就是一个schema，schema里面包含了各种对象如tables, views, sequences, stored procedures, synonyms, indexes, clusters, and database links。**
**一个用户一般对应一个schema,该用户的schema名等于用户名，并作为该用户缺省schema。**这也就是我们在企业管理器的方案下看到schema名都为数据库用户名的原因。
最简单的理解：以你计算机的用户为例，如果你的计算机有3个用户，那么每个用户登录系统看到的（使用的）功能是可以不相同的！
不同用户的表名是可以重复的。
` 也就是说：同一个schema 下，表的名字不能重复。与表空间无关 `