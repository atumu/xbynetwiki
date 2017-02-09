title: 理解java异常转译的强大 

#  理解Java异常转译的强大 
知识储备：
java中` Exception `是异常的顶级父类，不管是checked exception还是RuntimeException**都继承自它。**
那么在何种情况下是checkedException，何种情况下是RuntimeException呢？
**一般地，继承自Exception的类，或者继承非RuntimeException的Exception子类的类（如继承自IOException）是checkedException.**
**继承自RuntimeExcpetion的类是非受检的运行时异常。**

受检异常必须被捕获，因为编译器会进行检查。而非受检的运行时异常则无需进行捕获，且建议不要捕获它。

但是今天举得例子是我们要**将RuntimeException转译为Checked Exception进行捕获**。（注意：建议不要直接捕获RuntimeException。）
```

	public void deleteObject(String bucketName, String objectKey) throws BOSObjectNotFoundException {
		try {
			/**会发生RuntimeException*/
			client.deleteObject(bucketName, objectKey); 
		} catch (Exception e) { //RuntimeException也是Exception的子类
			e.printStackTrace();
			log.warn("bos对象没有找到{}", e);
			throw new BOSObjectNotFoundException(e); //进行异常转译，抛出checked exception
		}
	}

```
```

/**这是一个受检异常*/
public class BOSObjectNotFoundException extends Exception{
	public BOSObjectNotFoundException(){
		super();
	}
	public BOSObjectNotFoundException(String message, Throwable cause) {
		super(message, cause);
		// TODO Auto-generated constructor stub
	}
	public BOSObjectNotFoundException(String message) {
		super(message);
		// TODO Auto-generated constructor stub
	}
	public BOSObjectNotFoundException(Throwable cause) {
		super(cause);
		// TODO Auto-generated constructor stub
	}
}

```
```
	
		/**捕获转译而来受检异常*/
		try {
			bUtil.deleteObject(preInfo.getBucketName(), preInfo.getKey());
			log.info("删除旧备份成功");
		} catch (BOSObjectNotFoundException e1) {
			// TODO Auto-generated catch block
			e1.printStackTrace();
			log.info("bos对象没有找到，删除失败");
		}

```
**那我为什么要这么做？**
因为如果我不处理这个运行时异常，将导致我的任务调度程序终止。这是无法接受的。
那我为什么不对deleteObject这个方法进行预先检查对象是否存在呢？
第一、用的第三方接口没有提供该功能。
第二、虽然可以通过先调用listObject,然后通过for循环逐一对比来看是否存在该对象。
但是考虑到我还有一个deleteAllObjects.如果每次都在deleteObject方法中调用listObject循环判断是否存在。
那么当在deleteAllObjects循环调用deleteObject的时候，listObject将会调用很多次。这对性能是很有影响的。
**所以我们可以考虑将运行时异常进行转译，然后捕获转以后的受检异常，一旦发生异常捕获，然后我们就可以知道该对象不存在。**

下面继续贴代码：
```

public void deleteCategory(String bucketName, String prefix) {
		List<ObjectInfo> infoList = listObjects(bucketName, prefix);
		try {
			for (ObjectInfo info : infoList) {
				log.debug("删除：" + info.getKey());

				deleteObject(bucketName, info.getKey());

			}
		} catch (BOSObjectNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		log.debug("删除目录" + prefix + "成功");
	}

```
我相信看完这段代码，应该能理解我为什么没有在deleteObject中调用listObjects进行循环判断对象是否存在了。
