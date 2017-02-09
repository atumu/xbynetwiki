title: dao操作相关工具类 

#  DAO操作相关工具类 
MapUtil:
```

public class MapUtil {
	public static final <V> Map<String,Object> voToMap(V v){
		Method[] methods= v.getClass().getDeclaredMethods();
		Map<String,Object> map=new HashMap<>();
		for(Method m:methods){
			String name=m.getName();
			if(name.startsWith("get")){
				String nameKey=m.getName().substring(3);
                          	nameKey=nameKey.subString(0,1).toLowerCase()+nameKey.subString(1);
				try {
					map.put(nameKey, m.invoke(v));
				} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
		return map;
	}
	public static final Map<String,Object> formatKey(Map<String,Object> fromMap,String[] oldKeys,String[] formatKeys){
		if(fromMap!=null){
			for(int i=0;i<oldKeys.length;i++){
				if(fromMap.containsKey(oldKeys[i])){
					Object value=fromMap.remove(oldKeys[i]);
					fromMap.put(formatKeys[i], value);
				}
				
			}
		}
		return fromMap;
	}
}


```