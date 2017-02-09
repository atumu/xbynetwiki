title: memcached缓存 

#  memcached缓存 
Memcached是一个免费开源、高性能、分布式的**内存对象缓存系统**。Memcached是在**内存**中，为特定数据（字符串或对象）构建**key-value**的**小块数据存储**。
Memcached是由Danga Interactive开发的，高性能的，分布式的内存对象缓存系统，**通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。**
Memcached能缓存什么？ 通过在内存里维护一个统一的巨大的hash表，Memcached能够用来存储各种格式的数据，包括图像、视频、文件以及数据库检索的结果等。 
![](/data/dokuwiki/opensourcelearn/pasted/20150730-040019.png)
##  在服务器上部署Memcached Server 
以下以Windows平台为例：
参考：http://www.codeforest.net/how-to-install-memcached-on-windows-machine
下载下来的Windows版本解压到C:/memcached/
在控制台输入命令安装：
 c:/memcached/memcached.exe  -d install   
启动： c:/memcached/memcached.exe -d  start   或：net start "memcached Server"  
默认的缓存大小为64M，如果不够用，请打开注册表，找到：
HKEY_LOCAL_MACHINE/SYSTEM/CurrentControlSet/Services/memcached  Server .  
将其内容修改为：
“C:/memcached/memcached.exe” -d runservice -m 512  
##  下载Memcached的客户端API包 
###  spymemcached 

https://github.com/dustin/java-memcached-client
采用spymemcached：http://code.google.com/p/spymemcached
Maven依赖：
```

<dependency>
    <groupId>net.spy</groupId>
    <artifactId>spymemcached</artifactId>
    <version>2.12.0</version>
</dependency>

```
```

package com.sinosuperman.memcached;  
  
import java.io.IOException;  
import java.net.InetSocketAddress;  
  
import net.spy.memcached.MemcachedClient;  
  
public class TestMemcached {  
    public static void main(String[] args) throws IOException {  
        MemcachedClient cache = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));  
        for (int i = 1; i < 10; i++) {  
            cache.set("T0001" + i, 3600, new User(i + ""));   
        }  
        User myObject = (User) cache.get("T00011");  
        System.out.println("Get object from mem :" + myObject);   
    }   
}  

```
###  async get与超时 

```

// Get a memcached client connected to several servers
MemcachedClient c=new MemcachedClient(
        AddrUtil.getAddresses("server1:11211 server2:11211"));

// Try to get a value, for up to 5 seconds, and cancel if it doesn't return
Object myObj=null;
Future<Object> f=c.asyncGet("someKey");
try {
    myObj=f.get(5, TimeUnit.SECONDS);
} catch(TimeoutException e) {
    // Since we don't need this, go ahead and cancel the operation.  This
    // is not strictly necessary, but it'll save some work on the server.
    f.cancel(false);
    // Do other timeout related stuff
}

```
###  Establishing Connections 

**Establishing a Binary Protocol Connection**
```

// Get a memcached client connected to several servers with the binary protocol
MemcachedClient c=new MemcachedClient(
        new BinaryConnectionFactory(),
        AddrUtil.getAddresses("server1:11212 server2:11212"));

// Proceed to do cool stuff with memcached using the binary protocol.

```
**Establishing a Binary Protocol SASL Connection**
```

// Create an AuthDescriptor, this one is for plain SASL, so the 
//   username and passwords are just Strings
AuthDescriptor ad = new AuthDescriptor(new String[]{"PLAIN"},
                                new PlainCallbackHandler(username, password));

// Then connect using the ConnectionFactoryBuilder.  Binary is required.
        try {
            if (mc == null) {
                mc = new MemcachedClient(
                        new ConnectionFactoryBuilder().setProtocol(Protocol.BINARY)
                        .setAuthDescriptor(ad)
                        .build(),
                        AddrUtil.getAddresses(host));
            }
        } catch (IOException ex) {
            System.err.println("Couldn't create a connection, bailing out: \nIOException " + ex.getMessage());
        }

```
###  Using CAS 
```

public List<Item> addAnItem(final Item newItem) throws Exception {

    // This is how we modify a list when we find one in the cache.
    CASMutation<List<Item>> mutation = new CASMutation<List<Item>>() {

        // This is only invoked when a value actually exists.
        public List<Item> getNewValue(List<Item> current) {
            // Not strictly necessary if you specify the storage as
            // LinkedList (our initial value isn't), but I like to keep
            // things functional anyway, so I'm going to copy this list
            // first.
            LinkedList<Item> ll = new LinkedList<Item>(current);

            // If the list is already "full", pop one off the end.
            if(ll.size() > 10) {
                ll.removeLast();
            }
            // Add mine first.
            ll.addFirst(newItem);

            return ll;
        }

    };

    // The initial value -- only used when there's no list stored under
    // the key.
    List<Item> initialValue=Collections.singletonList(newItem);

    // The mutator who'll do all the low-level stuff.
    CASMutator<List<Item>> mutator = new CASMutator<List<Item>>(client, transcoder);

    // This returns whatever value was successfully stored within the
    // cache -- either the initial list as above, or a mutated existing
    // one
    return mutator.cas("myKey", initialValue, 0, mutation);
}

```
###  spymemcached与Spring集成 
http://code.google.com/p/spymemcached/wiki/SpringIntegration
```

  <bean id="memcachedClient" class="net.spy.memcached.spring.MemcachedClientFactoryBean">
    <property name="servers" value="host1:11211,host2:11211,host3:11211"/>
    <property name="protocol" value="BINARY"/>
    <property name="transcoder">
      <bean class="net.spy.memcached.transcoders.SerializingTranscoder">
        <property name="compressionThreshold" value="1024"/>
      </bean>
    </property>
    <property name="opTimeout" value="1000"/>
    <property name="timeoutExceptionThreshold" value="1998"/>
    <property name="hashAlg" value="KETAMA_HASH"/>
    <property name="locatorType" value="CONSISTENT"/> 
    <property name="failureMode" value="Redistribute"/>
    <property name="useNagleAlgorithm" value="false"/>
  </bean>

```
###  demo示例 
可以参考：http://blog.csdn.net/gtuu0123/article/details/4849905
```

import java.io.IOException;
import java.io.OutputStream;
import java.net.SocketAddress;
import java.util.Collection;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

import net.spy.memcached.AddrUtil;
import net.spy.memcached.BinaryConnectionFactory;
import net.spy.memcached.CASMutation;
import net.spy.memcached.CASMutator;
import net.spy.memcached.CASValue;
import net.spy.memcached.ConnectionObserver;
import net.spy.memcached.MemcachedClient;
import net.spy.memcached.transcoders.LongTranscoder;
import net.spy.memcached.transcoders.SerializingTranscoder;
import net.spy.memcached.transcoders.Transcoder;

public class SpyMemcachedManager {

	private List<SpyMemcachedServer> servers;

	private MemcachedClient memClient;

	public SpyMemcachedManager(List<SpyMemcachedServer> servers) {
		this.servers = servers;
	}

	public void connect() throws IOException {
		if (memClient != null) {
			return;
		}
		StringBuffer buf = new StringBuffer();
		for (int i = 0; i < servers.size(); i++) {
			SpyMemcachedServer server = servers.get(i);
			buf.append(server.toString()).append(" ");
		}
		memClient = new MemcachedClient(
				AddrUtil.getAddresses(buf.toString()));
	}

	public void disConnect() {
		if (memClient == null) {
			return;
		}
		memClient.shutdown();
	}
	
	public void addObserver(ConnectionObserver obs) {
		memClient.addObserver(obs);
	}
	
	public void removeObserver(ConnectionObserver obs) {
		memClient.removeObserver(obs);
	}
	
	//---- Basic Operation Start ----//
	public boolean set(String key, Object value, int expire) {
		Future<Boolean> f = memClient.set(key, expire, value);
		return getBooleanValue(f);
	}

	public Object get(String key) {
		return memClient.get(key);
	}

	public Object asyncGet(String key) {
		Object obj = null;
		Future<Object> f = memClient.asyncGet(key);
		try {
			obj = f.get(SpyMemcachedConstants.DEFAULT_TIMEOUT,
					SpyMemcachedConstants.DEFAULT_TIMEUNIT);
		} catch (Exception e) {
			f.cancel(false);
		}
		return obj;
	}

	public boolean add(String key, Object value, int expire) {
		Future<Boolean> f = memClient.add(key, expire, value);
		return getBooleanValue(f);
	}

	public boolean replace(String key, Object value, int expire) {
		Future<Boolean> f = memClient.replace(key, expire, value);
		return getBooleanValue(f);
	}

	public boolean delete(String key) {
		Future<Boolean> f = memClient.delete(key);
		return getBooleanValue(f);
	}

	public boolean flush() {
		Future<Boolean> f = memClient.flush();
		return getBooleanValue(f);
	}

	public Map<String, Object> getMulti(Collection<String> keys) {
		return memClient.getBulk(keys);
	}

	public Map<String, Object> getMulti(String[] keys) {
		return memClient.getBulk(keys);
	}

	public Map<String, Object> asyncGetMulti(Collection<String> keys) {
		Map map = null;
		Future<Map<String, Object>> f = memClient.asyncGetBulk(keys);
		try {
			map = f.get(SpyMemcachedConstants.DEFAULT_TIMEOUT,
					SpyMemcachedConstants.DEFAULT_TIMEUNIT);
		} catch (Exception e) {
			f.cancel(false);
		}
		return map;
	}

	public Map<String, Object> asyncGetMulti(String keys[]) {
		Map map = null;
		Future<Map<String, Object>> f = memClient.asyncGetBulk(keys);
		try {
			map = f.get(SpyMemcachedConstants.DEFAULT_TIMEOUT,
					SpyMemcachedConstants.DEFAULT_TIMEUNIT);
		} catch (Exception e) {
			f.cancel(false);
		}
		return map;
	}
	//---- Basic Operation End ----//

		
	//---- increment & decrement Start ----//
	public long increment(String key, int by, long defaultValue, int expire) {
		return memClient.incr(key, by, defaultValue, expire);
	}
	
	public long increment(String key, int by) {
		return memClient.incr(key, by);
	}
	
	public long decrement(String key, int by, long defaultValue, int expire) {
		return memClient.decr(key, by, defaultValue, expire);
	}
	
	public long decrement(String key, int by) {
		return memClient.decr(key, by);
	}
	
	public long asyncIncrement(String key, int by) {
		Future<Long> f = memClient.asyncIncr(key, by);
		return getLongValue(f);
	}
	
	public long asyncDecrement(String key, int by) {
		Future<Long> f = memClient.asyncDecr(key, by);
		return getLongValue(f);
	}
    //	---- increment & decrement End ----//
	
	public void printStats() throws IOException {
		printStats(null);
	}
	
	public void printStats(OutputStream stream) throws IOException {
		Map<SocketAddress, Map<String, String>> statMap = 
			memClient.getStats();
		if (stream == null) {
			stream = System.out;
		}
		StringBuffer buf = new StringBuffer();
		Set<SocketAddress> addrSet = statMap.keySet();
		Iterator<SocketAddress> iter = addrSet.iterator();
		while (iter.hasNext()) {
			SocketAddress addr = iter.next();
			buf.append(addr.toString() + "/n");
			Map<String, String> stat = statMap.get(addr);
			Set<String> keys = stat.keySet();
			Iterator<String> keyIter = keys.iterator();
			while (keyIter.hasNext()) {
				String key = keyIter.next();
				String value = stat.get(key);
				buf.append("  key=" + key + ";value=" + value + "/n");
			}
			buf.append("/n");
		}
		stream.write(buf.toString().getBytes());
		stream.flush();
	}
	
	public Transcoder getTranscoder() {
		return memClient.getTranscoder();
	}
	
	private long getLongValue(Future<Long> f) {
		try {
			Long l = f.get(SpyMemcachedConstants.DEFAULT_TIMEOUT,
					SpyMemcachedConstants.DEFAULT_TIMEUNIT);
			return l.longValue();
		} catch (Exception e) {
			f.cancel(false);
		}
		return -1;
	}

	private boolean getBooleanValue(Future<Boolean> f) {
		try {
			Boolean bool = f.get(SpyMemcachedConstants.DEFAULT_TIMEOUT,
					SpyMemcachedConstants.DEFAULT_TIMEUNIT);
			return bool.booleanValue();
		} catch (Exception e) {
			f.cancel(false);
			return false;
		}
	}

}

```
##  Memcached 客户端程序三种API： 

memcached client for java  ：较早推出的memcached JAVA客户端API，应用广泛，运行比较稳定。  
spymemcached  ： 支持异步，单线程的memcached客户端，用到了java1.5版本的concurrent和nio，存取速度会高于前者
xmemcached  ：XMemcached同样是基于java nio的客户端，java nio相比于传统阻塞io模型来说，有效率高（特别在高并发下）和资源耗费相对较少的优点。传统阻塞IO为了提高效率，需要创建一定数量的连接形成连接 池，而nio仅需要一个连接即可（当然,nio也是可以做池化处理），相对来说减少了线程创建和切换的开销，这一点在高并发下特别明显。因此 XMemcached与Spymemcached在性能都非常优

参考：
http://www.linuxidc.com/Linux/2011-12/49516.htm