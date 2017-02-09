title: java单例模式 

#   java单例模式 
第一种方式:缺点，无法延迟加载
```

public class Single{
	private static Single instance=new Single();
	private Single(){}
      public static Single getInstance(){ return instance;}
}

```
第二种：缺点：存在线程安全问题。
```

public class Single{
	private static Single instance;
	private Single(){}
      public static Single getInstance(){ 
        if(instance==null{
          instance=new Single();
        }
        return instance;
     }
}

```
第三种：推荐
```

public class Single{
	
	private Single(){}
 	 private static class SingletonHolder{
    		static Single instance=new Single();
 	 }
      public static Single getInstance(){ 
       return SingletonHolder.instance;
     }
}

```