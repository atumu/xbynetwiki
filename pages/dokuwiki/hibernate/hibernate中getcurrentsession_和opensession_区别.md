title: hibernate中getcurrentsession_和opensession_区别 

#  hibernate中getCurrentSession()和openSession()区别 
参考：http://www.blogjava.net/QJames/archive/2011/04/14/348274.html
http://liusu.iteye.com/blog/380397
Hibernate 3.1之后已经不需要用户自己使用ThreadLocal得方式来管理和持有session，而把这种session管理方式内置了，只要依据依据配置就可以用了 
```

hibernate.current_session_context_class = jta/thread/managed //Use thread

``` 
  * 1 getCurrentSession创建的session**会和绑定到当前线程**,而openSession每次创建新的session。
  * 2 getCurrentSession创建的线程会**在事务回滚或事物提交后` 自动关闭 `**,而openSession必须手动关闭
这里getCurrentSession本地事务(本地事务:jdbc)时 要在配置文件Hibernate.cfg.xml里进行如下设置
```

    * 如果使用的是本地事务（jdbc事务）
 <property name="hibernate.current_session_context_class">thread</property>
 * 如果使用的是全局事务（jta事务）
 <property name="hibernate.current_session_context_class">jta</property> 

```
这个不能忘记，我在测试中如果没有上面在hibernate.cfg.xml中配置，使用getCurrentSession()时，会出错

  * getCurrentSession () 在**事务结束之前**使用当前的session
  * openSession()         每次重新建立一个新的session

在一个应用程序中，如果DAO 层使用Spring 的hibernate 模板，**通过Spring 来控制session 的生命周期，则首选getCurrentSession()。**
在 SessionFactory 启动的时候， Hibernate 会根据配置创建相应的 CurrentSessionContext ，在 getCurrentSession() 被调用的时候，实际被执行的方法是 CurrentSessionContext.currentSession() 。
hibernate.current_session_context_class 配置参数定义了应该采用哪个org.hibernate.context.CurrentSessionContext实现。注意，为了向下兼容，如果未 配置此参数，但是存在org.hibernate.transaction.TransactionManagerLookup的配 置，Hibernate会采用org.hibernate.context.JTASessionContext。一般而言，此参数的值指明了要使用的实 现类的全名，但那两个内置的实现可以使用简写，即"jta"和"thread"。
```

private Long createAndStoreEvent(String title, Date theDate) {  
  
    Session session = HibernateUtil.getSessionFactory().getCurrentSession();  
    session.beginTransaction();  
  
    Event theEvent = new Event();  
    theEvent.setTitle(title);  
    theEvent.setDate(theDate);  
  
    session.save(theEvent);  
  
    session.getTransaction().commit();  //事务提交后会自动关闭current session
  
    return theEvent.getId();  
}  

```
