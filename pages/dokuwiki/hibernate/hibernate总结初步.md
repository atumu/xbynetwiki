title: hibernate总结初步 

#  Hibernate总结初步 
![](/data/dokuwiki/hibernate/pasted/20150817-035016.png)

##  持久化对象的三种状态 

分别为：瞬时状态（Transient），持久化状态（Persistent），离线状态（Detached）。三种状态下的对象的生命周期如下：
![](/data/dokuwiki/hibernate/pasted/20150817-035103.png)
三种状态的区别是：瞬时状态的对象：没有被session管理，在数据库没有；持久化状态的对象：被session管理，在数据库存在，当属性发生改变，在清理缓存时，会自动和数据库同步；离线状态：没有被session管理，但是在数据库中存在。
##  主键选择策略 
通常分为两种：
1 **自然主键**，也就是带有业务含义的，比如学生的学号，工作的编号，通常包含了年份，部门或者班级，专业等等业务上的意义，**因此需要手动的合成或者拼接指定**。这种情况下就需要使用assinged方式，这种方式如果不指定主键就提交缓存进行更新，会报错！

2 **代理主键**，也就是**没有业务含义的**，**通常是通过编码自动生成**的。
　　**increment**:不依赖于底层数据库，适合单个数据库场合不适合集群，必须为long int short类型。插入式，先选择最大的id值，再加1
　　**identity**:依赖底层数据库系统。Mysql中使用，Oracle不支持。支持自动增长字段： OID 为long,int,short
　　**sequence**:MYSQL不支持序列，Oracle中使用。依赖底层，必须支持序列。Oracle db2 sap db  postgresql
　　**hilo**:计算公式hi*(max_lo+1)+lo 不依赖底层数据库系统，Long,int,short，只能在一个数据库中保持唯一
　　` native `:跨平台，自动选择使用哪个策略。由底层方言产生。
 assigned  不常用，手劢生成id 

　　由于上面的identity,sequence都需要依赖于底层数据库，不同的数据库可能不支持这种方式**。那么一般推荐使用native，自动进行选择。**
##  关于Session缓存——清理缓存 
　　缓存的概念，一般学过基础理论的都应该理解，就是为了缓冲数据，减少与真实数据库的频繁交互。与计算机的缓存类似，经常访问硬盘效率太低，IO太慢，就把内存当做缓存，CPU每次与内存直接交互，内存中找不到的数据再去读硬盘。内存又觉得慢了，就弄个Cahce当做缓存，经常访问的数据再放到这里，更加快了速度。
　　Session缓存也是如此，与Web中的Session也类似。在网页中，也有Session这样一种概念，比如我们登陆淘宝，会记录我们的用户信息，当浏览器关闭或者退出时，Session关闭。这期间就完全通过Session来识别用户的身份，无需每次登陆进行校验。Hibernate中也是如此，我们从SessionFactory中开启这个Session,持久化一个对象，然后提交事务，增删改查，最后关闭Session,就像一个对话一样。
　　那么Session缓存具体有什么作用呢？
　　比如我们通过Session.get(xxx.class,new Long(1));来获取Session中OID为1的对象，它会首先到缓存中查找，如果找到了就直接用。如果找不到就去读取数据库，然后存储到缓存中！第二次，就可以直接从缓存中获取数据了！这样就减少了访问数据库的频率！
另外，我们频繁的修改一个对象，如果这个对象放在缓存中，而且还是用了事务，那么只有事务在commit的时候，才会执行真正的SQL语句！
　　这样就对对象与数据库的表进行了动态的映射！
**Session缓存又是什么时候提交清理的呢？**
　　1 当使用事务时，` transaction.commit() `会触发缓存的清理。
　　2 直接调用` Session.flush() `也会触发缓存的清理。
　　3 如果使用的是` native `，那么在持久化的时候也会清理缓存，也就是` session.save() `时。
　　4 执行查询时。
　　这里就不得不提一下**commit与Session的flush的区别**了：当使用flush时，并没有提交事务，只是清理缓存而已。而commit的时候，是先调用flush再提交事务。
###  Session缓存与状态 
![](/data/dokuwiki/hibernate/pasted/20150817-040007.png)
##  关于Session中的方法使用 
　**　save()**
　　Session调用save时，一般都是创建或者获取到了一个瞬时态的对象，这时对象的OID有可能是空的，session需要指定生成一个OID。再计划生成一条insert语句，**这条语句只是简单的缓存起来，当事务提交时才执行**。而持久化的对象，OID是不能随便更改的，这也是为什么前面的setId推荐设置成private的访问权限。
**　　load()和get()**
　　他们都是加载一个对象，或者从缓存中查找。区别在于，如果**使用load**，如果数据库中不存在该对象对应的数据**，会抛出异常。而get会得到null。**
　**　update()**
　　**这个方法是把一个游离态的对象持久化，比如一个对象如果session清理了，那么session中就找不到这个对象了，但是数据库中仍然存在。我们通过这个对象的引用，可以通过update在Session中创建它的实例。**这样，会生成一条update语句。如果此后在修改无论多少次，都只会生成一条update语句。总结起来就是，**update方法，会生成一条对应的update语句来同步缓存与数据库中的对象**。
　　如果数据库中对应的表设置了触发器，那么就蛋疼了、！因为无论你是否修改了数据，都会生成一条update语句，这样就会导致触发了大量无效的触发器。不要担心，可以通过设置select-before-update属性，一看名字就能猜到，是在update前，进行一次select，如果数据一致，就必须要update了，如果数据不一致，才update。

　　**saveOrUpdate()**
　　这个方法就给力了，它会**自动判断传入的参数**是什么类型的，然后采取什么措施！完全的自动化，最喜欢这样的了！跟native一个套路。
　　**merge()**
　　对象的**复制**，它首先获取到OID，然后去session中查找是否存在这样的对象，如果存在直接修改或者使用；如果不存在，就复制这个对象的属性。
　　**delete()**
　　如果删除的对象时一个游离态的对象，那么需要先进行持久化，在删除。
　　**replicate()**
　　这个方法可以跨Sessionfactory拷贝对象。
![](/data/dokuwiki/hibernate/pasted/20150817-041226.png)
参考资料：
http://blog.csdn.net/yuebinghaoyuan/article/details/7300599 系列
http://www.cnblogs.com/xing901022/p/4151875.html