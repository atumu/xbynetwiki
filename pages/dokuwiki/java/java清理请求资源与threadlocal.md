title: java清理请求资源与threadlocal 

#  java清理请求资源与ThreadLocal防内存溢出 
思路:通过注册ServletRequestListener,并在其requestDestroyed()方法中调用ThreadLocal的remove()方法来请求当前请求资源.
```

@WebListener
public class RequestListener implements ServletRequestListener {

	@Override
	public void requestDestroyed(ServletRequestEvent arg0) {
		//用于相关请求资源的清理
//		System.out.println("RequestListener线程:"+Thread.currentThread().getName()+":"+Thread.currentThread().getId());
		ServiceUtil.clean();
	}

	@Override
	public void requestInitialized(ServletRequestEvent arg0) {
		// TODO Auto-generated method stub
	}

}


```
```

public class ServiceUtil {
	private static ThreadLocal<Map<String,IPersistenceService>> localDao=new ThreadLocal<Map<String,IPersistenceService>>(){
		@Override
        protected Map<String,IPersistenceService> initialValue() {
           return new HashMap<String,IPersistenceService>();
        }
	};
	public static IPersistenceService genDao(){
//		System.out.println("ServiceUtil线程:"+Thread.currentThread().getName()+":"+Thread.currentThread().getId());
		IPersistenceService dao=SwordPersistenceUtils.getPersistenceService();
		if(dao.equals(localDao.get().get("dao"))){
			dao=localDao.get().get("localDao");
		}else{
			localDao.get().put("dao", dao);
			dao=DaoWrapper.genDao(dao);
			localDao.get().put("localDao",dao);
		}
		return dao;
	}
	/**
	 * 清理资源，防止由于web服务器请求线程池导致dao一直保留问题和可能导致的ThreadLocal内存泄漏问题
	 */
	public static void clean(){
		localDao.remove();
	}
	/**
	 * 构造返回结果。
	 * @return
	 */
	public static ResultBuilder genResultBuilder(){
		return new ResultBuilder();
	}
}


```

参考:
http://stackoverflow.com/questions/3869026/how-to-clean-up-threadlocals
http://blog.csdn.net/vking_wang/article/details/13168743