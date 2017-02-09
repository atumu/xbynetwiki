title: ehcache缓存到磁盘 

#  ehcache缓存到磁盘 
```

 <dependency>
      <groupId>net.sf.ehcache</groupId>
      <artifactId>ehcache</artifactId>
      <version>2.10.1</version>
    </dependency>

```
参考：http://blog.csdn.net/huwenhu2007/article/details/25283205
```

<diskStore path="C:/temp"/>

<cache name="authCache1"
           maxElementsInMemory="36"
           maxElementsOnDisk="10000"
           overflowToDisk="true"

           diskPersistent="true"
           diskSpoolBufferSizeMB="20"
/>

```

**当内存中的数据量超过了设置的最大记录数maxElementsInMemory，则会将多余的记录数存入硬盘，但是内存中已经存在的内容则不会自动存入硬盘，需要手动使用cache.flush()来强制将内存内容放入硬盘中；
一般maxElementsOnDisk的值需要设置的比maxElementsInMemory大，这样效率会高一些；**

需要设置**overflowToDisk为true**,表示使用硬盘缓存策略；

需要设置**diskPersistent为true**，**否则重启服务后缓存文件会被清理掉**，同时该属性会在一定程度上拖慢缓存速度，因为它需要不停地监控设置的硬盘大小是否已满关联属性为**diskSpoolBufferSizeMB**，如果硬盘够大可以将该属性设置足够大；

最后 会生成*.index和*.data2个文件；
重建缓存会自动从硬盘读取数据到内存
**diskStore** ：指定数据存储位置，可指定磁盘中的文件夹位置
**maxElementsInMemory**： 在内存中缓存的element的最大数目。
**maxElementsOnDisk**： 在磁盘上缓存的element的最大数目，**默认值为0，表示不限制**。 
**eternal**： 设定缓存的elements是否**永远不过期**。如果为true，则缓存的数据始终有效，**如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断。**
**overflowToDisk**： 如果内存中数据超过内存限制，是否要缓存到磁盘上。

以下属性是可选的： 
**timeToIdleSeconds**： **对象空闲时间，指对象在多长时间没有被访问就会失效**。**只对eternal为false的有效。默认值0**，表示一直可以访问。
**timeToLiveSeconds**：** 对象存活时间，指对象从创建到失效所需要的时间**。**只对eternal为false的有效。默认值0**，表示一直可以访问。
**diskPersistent**： **是否在磁盘上持久化。指重启jvm后，数据是否有效**。默认为false。 
**diskExpiryThreadIntervalSeconds**： 对象检测线程运行时间间隔。标识对象状态的线程多长时间运行一次。 
**diskSpoolBufferSizeMB**： **DiskStore使用的磁盘大小，默认值30MB**。每个cache使用各自的DiskStore。
**memoryStoreEvictionPolicy**： 如果内存中数据超过内存限制，向磁盘缓存时的策略。**默认值LRU，可选FIFO、LFU**。

```

<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:noNamespaceSchemaLocation="ehcache.xsd">  
  
    <diskStore path="C:/temp"/>  
      
    <defaultCache  
            maxElementsInMemory="10000"  
            eternal="false"  
            timeToIdleSeconds="120"  
            timeToLiveSeconds="120"  
            overflowToDisk="false"  
            diskSpoolBufferSizeMB="30"  
            maxElementsOnDisk="10000000"  
            diskPersistent="false"  
            diskExpiryThreadIntervalSeconds="120"  
            memoryStoreEvictionPolicy="LRU"  
            />  
  
              
    <cache name="authCache1"  
           maxElementsInMemory="36"  
           maxElementsOnDisk="10000"  
           eternal="true"  
           diskPersistent="true"  
           overflowToDisk="true"  
           diskSpoolBufferSizeMB="20"  
           timeToIdleSeconds="300"  
           timeToLiveSeconds="600"  
           memoryStoreEvictionPolicy="LRU"  
            />  
  
</ehcache>  

```
```

public class EhcacheTest {  
      
    public static void main(String[] args) throws Exception{  
        PropertyConfigurator.configure("src/log4j/log4j.properties");  
        /** 
         * 创建缓存管理对象 
         */  
        URL url = EhcacheTest.class.getResource("ehcache.xml");  
        CacheManager cacheManager = CacheManager.create(url);  
        Thread.sleep(10000);  
          
        /** 
         * 获取缓存对象 
         */  
        Cache cache = cacheManager.getCache("authCache1");  
        /** 
         * 存入数据 
         */  
        SqlMapClient sqlMapClient = SqlClientMap.init();  
        List list = sqlMapClient.queryForList("test.selectTest");  
        for(int i = 0;i < list.size();i++){  
            Element element = new Element(i, list.get(i));  
            cache.put(element);  
            System.out.println(list.get(i));  
        }  
          
        /*强行输出内存数据到硬盘*/  
        cache.flush();  
        Thread.sleep(5000);  
          
        /** 
         * 关闭ehcache 
         */  
        Thread.sleep(5000);  
        cacheManager.shutdown();  
          
        /** 
         * 取出数据 
         */  
        cacheManager = CacheManager.create(url);  
        Thread.sleep(10000);  
          
        cache = cacheManager.getCache("authCache1");  
          
        for(int i = 0;i < 36;i++){  
            Element elements = cache.get(i);  
            System.out.print(elements.getKey() + " ----- ");  
            System.out.println(elements.getValue());  
        }  
          
        cacheManager.shutdown();  
    }  
      
}  

```