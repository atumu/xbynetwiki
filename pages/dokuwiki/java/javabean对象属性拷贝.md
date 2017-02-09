title: javabean对象属性拷贝 

#  JavaBean对象属性拷贝 
对象拷贝的应用现状简介：
业务系统中经常需要两个对象进行属性的拷贝，不能否认逐个的对象拷贝是最快速最安全的做法，但是当数据对象的属性字段数量超过程序员的容忍的程度，代码因此变得臃肿不堪，
 在做业务的时候，我们有时为了隔离变化，会将DAO查询出来的Entity，和对外提供的DTO隔离开来。大概90%的时候，它们的结构都是类似的，但是我们很不喜欢写很多冗长的b.setF1(a.getF1())这样的代码，
使用一些方便的对象拷贝工具类将是很好的选择。
目前流行的较为公用认可的工具类：
Apache的两个版本：（反射机制）
org.apache.commons.beanutils.**PropertyUtils**.copyProperties(Object dest, Object orig)
org.apache.commons.beanutils.**BeanUtils**.copyProperties(Object dest, Object orig)
Spring版本：（反射机制）
org.springframework.beans.**BeanUtils**.copyProperties(Object source, Object target, Class editable, String[] ignoreProperties)
` cglib版本：（使用动态代理，效率高） `
net.sf.cglib.beans.` BeanCopier `.copy(Object paramObject1, Object paramObject2, Converter paramConverter)

从网上查的资料显示：
PropertyUtils的性能是BeanUtils(apache-common)的1.71倍
BeanCopier的性能是PropertyUtils (apache-common)的504倍。
可见，对于对象的拷贝，应尽量使用cglib的BeanCopier. 

##  原理简介 
**反射类型：（apache）**
都使用静态类调用，最终转化虚拟机中两个单例的工具对象。
```

public BeanUtilsBean()
{
  this(new ConvertUtilsBean(), new PropertyUtilsBean());
}

```
ConvertUtilsBean可以通过ConvertUtils全局自定义注册。
ConvertUtils.register(new DateConvert(), java.util.Date.class);
PropertyUtilsBean的copyProperties方法实现了拷贝的算法。
1、动态bean：orig instanceof DynaBean：Object value = ( ( DynaBean)orig).get(name);然后把value复制到动态bean类
2、Map类型：orig instanceof Map：key值逐个拷贝
3、其他普通类：：从beanInfo【每一个对象都有一个缓存的bean信息，包含属性字段等】取出name，然后把sourceClass和targetClass逐个拷贝

**Cglib类型：BeanCopier**
```

copier = BeanCopier.create(source.getClass(), target.getClass(), false);
copier.copy(source, target, null);

```
Create对象过程：产生sourceClass-> TargetClass的**拷贝代理类**，放入jvm中，**所以创建的代理类的时候比较耗时。最好保证这个对象的单例模式**，可以参照最后一部分的优化方案。
创建过程：源代码见jdk：net.sf.cglib.beans.BeanCopier.Generator.generateClass(ClassVisitor)
1、  获取sourceClass的所有public get 方法-》PropertyDescriptor[] getters
2、  获取TargetClass 的所有 public set 方法-》PropertyDescriptor[] setters
3、  遍历setters的每一个属性，执行4和5
4、  按setters的name生成sourceClass的所有setter方法-》PropertyDescriptor getter【不符合javabean规范的类将会可能出现空指针异常】
5、  PropertyDescriptor[] setters-》PropertyDescriptor setter
6、  将setter和getter名字和类型 配对，生成代理类的拷贝方法。
` Copy属性过程：调用生成的代理类，代理类的代码和手工操作的代码很类似，效率非常高。 `

##  陷阱预防 
![](/data/dokuwiki/java/pasted/20151212-235317.png)

##  性能优化建议 
将beancopier做成静态类，方便拷贝
```

public class BeanCopierUtils {
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
参考：http://blog.csdn.net/jianhua0902/article/details/8155368
http://my.oschina.net/flashsword/blog/404288