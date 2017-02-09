title: java自定义lru缓存实现 

#  Java自定义LRU缓存实现 
```

public interface Cache {
	/**
	 * 缓存ID
	 * @return
	 */
	String getId(); 
	Object get(String key);
	void put(String key,Object value);
	void clear();
	void remove(String key);
	int size();
}

```
```

public class LRUCacheImpl implements Cache {
	private long timeout=0; //0代表永不超时
	private Map<Object,CacheEntry> cache=null;
	private String id;
	public LRUCacheImpl(String id,int maxsize,long timeout){
		this.timeout=timeout;
		this.cache=new LRULinkedHashMap(maxsize);
		this.id=id;
	}
	public LRUCacheImpl(int maxsize,long timeout){
		this.timeout=timeout;
		this.cache=new LRULinkedHashMap(maxsize);
		this.id=UUID.randomUUID().toString();
	}

	@Override
	public String getId() {
		// TODO Auto-generated method stub
		return this.id;
	}


	@Override
	public Object get(String key) {
		// TODO Auto-generated method stub
		Object value=null;
		CacheEntry entry;
		synchronized (this) {
			entry=cache.get(key);
		}
		if(entry!=null){
			//如果没有超时，或者超时设置为0
			if((System.currentTimeMillis()-entry.getTimeCached())<timeout||timeout==0){
				value=entry.getValue();
			}else{
				cache.remove(key);
			}
		}
		return value;
	}


	@Override
	public synchronized void put(String key, Object value) {
		// TODO Auto-generated method stub
		cache.put(key, new DefaultCacheEntry(value, System.currentTimeMillis()));
	}

	@Override
	public synchronized void clear() {
		// TODO Auto-generated method stub
		cache.clear();
	}


	@Override
	public synchronized void remove(String key) {
		// TODO Auto-generated method stub
		cache.remove(key);
	}

	@Override
	public int size() {
		// TODO Auto-generated method stub
		return cache.size();
	}
}

```
```

public class LRULinkedHashMap extends LinkedHashMap<Object, CacheEntry>{
	private int maxSize;
	public LRULinkedHashMap(int maxsize){
		super(maxsize/2+1,0.75f,true);
		this.maxSize=maxsize;
	}
	/* 
	 * 当缓存数超过最大容量时，返回true把最后一个移除。 
	 * @see java.util.LinkedHashMap#removeEldestEntry(java.util.Map.Entry)
	 */
	@Override
	 protected boolean removeEldestEntry(Map.Entry eldest) {
        return size()>maxSize;
    }
}

```
```

public interface CacheFactory {
	Cache createCache(String id,int maxsize,long timeout);
}

```
```

public class LRUCacheFactory implements CacheFactory{

	@Override
	public Cache createCache(String id, int maxsize, long timeout) {
		// TODO Auto-generated method stub
		return new LRUCacheImpl(id,maxsize,timeout);
	}

}

```
```

public interface CacheEntry {
	/**
	 * 加入缓存时的时间
	 * @return
	 */
	long getTimeCached();
	Object getValue();
}

```
```

public class DefaultCacheEntry implements CacheEntry{
	private long timecached=-1;
	private Object value;
	public DefaultCacheEntry(Object value,long timecached){
		this.value=value;
		this.timecached=timecached;
	}

	@Override
	public long getTimeCached() {
		// TODO Auto-generated method stub
		return timecached;
	}

	@Override
	public Object getValue() {
		// TODO Auto-generated method stub
		return value;
	}

}

```
```

public class CacheManager{
	private static final ConcurrentHashMap<String, Cache> caches=new ConcurrentHashMap<>();
	/**
	 * 缓存策略枚举
	 */
	public static enum CacheStrategy{
		LRU
	}
	
	/**
	 * 
	 * @param maxsize
	 * @param timeout
	 * @param strategy 可为null。默认LRU
	 * @return
	 */
		public static Cache createCache(int maxsize, long timeout,CacheStrategy strategy) {
		if(strategy==null){
			strategy=CacheStrategy.LRU;
		}
		Cache cache=getCacheFactory(strategy).createCache(UUID.randomUUID().toString(),maxsize, timeout);
		caches.putIfAbsent(cache.getId(), cache);
		return cache;
	}
	private static CacheFactory getCacheFactory(CacheStrategy strategy){
		CacheFactory factory=null;
		if("LRU".equals(strategy.name())){
			factory=new LRUCacheFactory();
		}
		return factory;
	}
	public static Cache getCacheById(String cacheId){
		return caches.get(cacheId);
	}
	public static void clear(){
		caches.clear();
	}
	public static void remove(String cacheId){
		caches.remove(cacheId);
	}
	
}

```