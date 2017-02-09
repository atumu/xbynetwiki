title: hibernateutil 

#  HibernateUtil工具类 
非常使用的采用Session ThreadLocal模式的确保Session线程安全的HibernateUtil工具类
```

public class HibernateUtil {  
  /**SessionFactory作为重量级选手，应该对于整个应用来说保存为全局变量。*/
public static final SessionFactory sessionFactory;  
 /**采用ThreadLocal模式的hibernate Session*/
public static final ThreadLocal session = new ThreadLocal();  
  /**静态初始化SessionFactory*/
static{  
	try{  
		Configuration configuration=new Configuration().configure();   
		sessionFactory = configuration.buildSessionFactory();  
	}catch (Throwable ex){  
		System.err.println("Initial SessionFactory creation failed." + ex);  
		throw new ExceptionInInitializerError(ex);  
	}  
}  
 /**返回当前线程上下文相关的hibernate Session*/
public static Session currentSession() throws HibernateException{  
	Session s = (Session) session.get();  
	if (s == null)  
	{  
		s = sessionFactory.openSession();  
		session.set(s);  
	}  
	return s;  
}  
 /**关闭hibernate session*/
public static void closeSession() throws HibernateException {  
	Session s = (Session) session.get();  
	if (s != null)  
		s.close();  
		session.set(null);  
	}
	/**返回SessionFactory单例*/
public static SessionFactory getSessionFactory() {
      // Alternatively, we could look up in JNDI here
      return sessionFactory;
  }
  /**关闭SessionFactory*/
public static void shutdown() {
      // Close caches and connection pools
      getSessionFactory().close();
  }	
} 

```