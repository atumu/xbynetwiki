title: javabean属性拷贝对比 

#  JavaBean对象属性拷贝之反射与Cglib性能对比 
本文同步在OSChina上:http://www.oschina.net/code/snippet_2438265_52823
业务分层的时候通过除了Entity类之外还会有一个Dto或者Vo类，在相互传值的时候我们就需要进行一系列繁琐的setter/getter调用。而且当实体特别大的时候会导致代码繁琐，特别难读。不好维护。
所以我们需要自动化JavaBean对象属性拷贝这一过程。现提供三种解决方案及其性能对比：
1、原生Java反射机制，原汁原味。
2、Cglib的BeanMap
3、Cglib的BeanCopier
当然还有一些，比如commos BeanUtils ,PropertyUtils.或者Spring的BeanUtils.
代码如下。
循环100*100*1000次的情况下:
方案1：输出
6135
id:111
方案2输出：
12328
id:111
方案三输出:
5444
id:111

从这里可以看出，其实反射性能也不是太差。性能排序为如下
BeanMap<反射<BeanCopier
反射比BeanMap快2倍，
而BeanCopier比反射也没快太多。

不能这个测试也是有缺陷的：
1、没有对复杂数据类型进行测试。感兴趣的朋友可以进行测试。
2、没有缓存BeanCopier对象。这样的话估计方案三会再快点。不过最后代码提供了改良方案

```

public class Dto2Entity {
	private static final Logger log = LoggerFactory.getLogger(Dto2Entity.class);

	/**
	 * DTO对象转换为实体对象。如命名不规范或其他原因导致失败返回null
	 * 
	 * @param t
	 *            进行转换的对象
	 * @param e
	 *            进行转换的对象
	 * @return 实体对象
	 */
	public static <T, E> void transalte(T t, E e) {
		Method[] tms = t.getClass().getDeclaredMethods();
		Method[] tes = e.getClass().getDeclaredMethods();

		for (Method m1 : tms) {
			if (m1.getName().startsWith("get")||m1.getName().startsWith("is")) {
				String mNameSubfix="";
				if(m1.getName().startsWith("get")){
					mNameSubfix = m1.getName().substring(3);
				}else{
					mNameSubfix = m1.getName().substring(2);
				}
				String forName = "set" + mNameSubfix;
				for (Method m2 : tes) {
					if (m2.getName().equals(forName)) {
						// 如果参数类型一致，或者m1的返回类型是m2的参数类型的父类或接口
						boolean canContinue = m1.getReturnType().isAssignableFrom(m2.getParameterTypes()[0]);
						if (canContinue) {
							try {
								m2.invoke(e, m1.invoke(t));
								break;
							} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e1) {
								// TODO Auto-generated catch block
								log.debug("DTO 2 Entity转换失败");
								e1.printStackTrace();
							}
						}
					}
				}
			}

		}
//		log.debug("转换完成");

	}
//	/**
//	 *为了性能考虑，采用cglib来创建动态代理对象
//	 * @param t
//	 * @return
//	 */
//	@SuppressWarnings("unchecked")
//	public static <T> T generateProxy(T t){
//		Enhancer enhancer=new Enhancer();
//		enhancer.setSuperclass(t.getClass());
//		//直接调用方式的回调
//		enhancer.setCallback(NoOp.INSTANCE);
//		//设置类装载器
//		enhancer.setClassLoader(t.getClass().getClassLoader());
//		return (T)enhancer.create();
//	}
	public static void main(String[] args) {
		GlobalConfig a=new GlobalConfig();
		GlobalConfig b=new GlobalConfig();
		a.setId("111");
		a.setIsFirst(0);
		long start=System.currentTimeMillis();
		for(int i=0;i<1000*100*100;i++){
			//测试1
			//transalte(a, b); 
			//测试2
			/*BeanMap map1=BeanMap.create(a);
			BeanMap map2=BeanMap.create(b);
			for(Object key:map1.keySet()){
				if(map2.containsKey(key)){
					map2.put(key, map1.get(key));
				}
			}*/
			
			//测试3
			//BeanCopier bc=BeanCopier.create(GlobalConfig.class, GlobalConfig.class, false);
			//bc.copy(a, b, null);

		}
		
		long end=System.currentTimeMillis();
		System.out.println(end-start);
		System.out.println("id:"+b.getId());
	}
}


```

方案3性能优化:
```

public class MyBeanUtil {
	// 使用多线程安全的Map来缓存BeanCopier，由于读操作远大于写，所以性能影响可以忽略
	private static ConcurrentHashMap<String, BeanCopier> beanCopierMap = new ConcurrentHashMap<String, BeanCopier>();
        private static final Object lock=new Object() ;
	public static void copyProperties(Object source, Object target) {
		String beanKey = generateKey(source.getClass(), target.getClass());
		BeanCopier copier = beanCopierMap.get(beanKey);
		 if(copier==null){ 
                   synchronized(lock){
                      copier = beanCopierMap.get(beanKey);
                      if(copier==null){
                        copier = BeanCopier.create(source.getClass(), target.getClass(), false); 
                         beanCopierMap.putIfAbsent(beanKey, copier);// putIfAbsent已经实现原子操作了。
                      }
                   }
                   
                }
		copier.copy(source, target, null);
	}

	private static String generateKey(Class<?> class1, Class<?> class2) {
		return class1.toString() + class2.toString();
	}
}

```