title: guava学习之缓存 

#  Guava学习之缓存 
```

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .removalListener(MY_LISTENER)
        .build(
            new CacheLoader<Key, Graph>() {
                public Graph load(Key key) throws AnyException {
                    return createExpensiveGraph(key);
                }
        });

```
##  适用性 

缓存在很多场景下都是相当有用的。例如，计算或检索一个值的代价很高，并且对同样的输入需要不止一次获取值的时候，就应当考虑使用缓存。

Guava Cache与ConcurrentMap很相似，但也不完全一样。最基本的区别是**ConcurrentMap会一直保存所有添加的元素**，直到显式地移除。相对地，**Guava Cache为了限制内存占用，通常都设定为自动回收**元素。在某些场景下，尽管LoadingCache 不回收元素，它也是很有用的，因为它会自动加载缓存。
通常来说，Guava Cache适用于：

  * 你愿意消耗一些内存空间来提升速度。
  * 你预料到某些键会被查询一次以上。
  * 缓存中存放的数据总量不会超出内存容量。（Guava Cache是单个应用运行时的本地缓存。它不把数据存放到文件或外部服务器。如果这不符合你的需求，请尝试Memcached这类工具）
如果你的场景符合上述的每一条，Guava Cache就适合你。
注：如果你不需要Cache中的特性，使用ConcurrentHashMap有更好的内存效率——但Cache的大多数特性都很难基于旧有的ConcurrentMap复制，甚至根本不可能做到。

##  加载 
在使用缓存前，首先问自己一个问题：有没有合理的默认方法来加载或计算与键关联的值？如果有的话，你应当使用` CacheLoader `。如果没有，或者你想要覆盖默认的加载运算，同时保留"获取缓存-如果没有-则计算"[get-if-absent-compute]的原子语义，你应该在调用get时传入一个Callable实例。缓存元素也可以通过` Cache.put方法直接插入 `，但自动加载是首选的，因为它可以更容易地推断所有缓存内容的一致性。
###  CacheLoader 
` LoadingCache `是附带` CacheLoader `构建而成的缓存实现。创建自己的CacheLoader通常只需要简单地实现V load(K key) throws Exception方法。例如，你可以用下面的代码构建LoadingCache：
```

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .build(
            new CacheLoader<Key, Graph>() {
                public Graph load(Key key) throws AnyException {
                    return createExpensiveGraph(key);
                }
            });

...
try {
    return graphs.get(key);
} catch (ExecutionException e) {
    throw new OtherException(e.getCause());
}

```
从LoadingCache查询的正规方式是使用` get(K) `方法。这个方法要么返回已经缓存的值，要么使用CacheLoader向缓存原子地加载新值。
**getAll(Iterable<? extends K>)方法用来执行批量查询。**默认情况下，对每个不在缓存中的键，getAll方法会单独调用CacheLoader.load来加载缓存项。如果批量的加载比多个单独加载更高效，你可以重载` CacheLoader.loadAll `来利用这一点。getAll(Iterable)的性能也会相应提升。

###  Callable 
所有类型的Guava Cache，不管有没有自动加载功能，都支持` get(K, Callable<V>) `方法。这个方法返回缓存中相应的值，或者用给定的Callable运算并把结果加入到缓存中。在整个加载方法完成前，缓存项相关的可观察状态都不会更改。这个方法简便地实现了模式"**如果有缓存则返回；否则运算、缓存、然后返回**"。
```

Cache<Key, Graph> cache = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .build(); // look Ma, no CacheLoader
...
try {
    // If the key wasn't in the "easy to compute" group, we need to
    // do things the hard way.
    cache.get(key, new Callable<Key, Graph>() {
        @Override
        public Value call() throws AnyException {
            return doThingsTheHardWay(key);
        }
    });
} catch (ExecutionException e) {
    throw new OtherException(e.getCause());
}

```
###  显式插入 


使用` cache.put(key, value) `方法可以直接向缓存中插入值，这会直接覆盖掉给定键之前映射的值。使用Cache.asMap()视图提供的任何方法也能修改缓存。但请注意，asMap视图的任何方法都不能保证缓存项被原子地加载到缓存中。进一步说，asMap视图的原子运算在Guava Cache的原子加载范畴之外，所以相比于Cache.asMap().putIfAbsent(K,
V)，` Cache.get(K, Callable<V>) ` 应该总是优先使用。
##  缓存回收 

一个残酷的现实是，我们几乎一定没有足够的内存缓存所有数据。你你必须决定：什么时候某个缓存项就不值得保留了？
Guava Cache提供了三种基本的缓存回收方式：基于容量回收、定时回收和基于引用回收。
**基于容量的回收**（size-based eviction）

如果要规定缓存项的数目不超过固定值，只需使用` CacheBuilder.maximumSize(long) `。缓存将尝试回收最近没有使用或总体上很少使用的缓存项。
**定时回收（**Timed Eviction）

CacheBuilder提供两种定时回收的方法：
` expireAfterAccess `(long, TimeUnit)：缓存项在给定时间内没有被读/写访问，则回收。请注意这种缓存的回收顺序和基于大小回收一样。
` expireAfterWrite `(long, TimeUnit)：缓存项在给定时间内没有被写访问（创建或覆盖），则回收。如果认为缓存数据总是在固定时候后变得陈旧不可用，这种回收方式是可取的。
如下文所讨论，定时回收周期性地在写操作中执行，偶尔在读操作中执行。
**基于引用的回收**（Reference-based Eviction）

通过使用弱引用的键、或弱引用的值、或软引用的值，Guava Cache可以把缓存设置为允许垃圾回收：
` CacheBuilder.weakKeys `()：使用弱引用存储键。当键没有其它（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式（==），使用弱引用键的缓存用==而不是equals比较键。
CacheBuilder.weakValues()：使用弱引用存储值。当值没有其它（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式（==），使用弱引用值的缓存用==而不是equals比较值。
CacheBuilder.softValues()：使用软引用存储值。软引用只有在响应内存需要时，才按照全局最近最少使用的顺序回收。考虑到使用软引用的性能影响，我们通常建议使用更有性能预测性的缓存大小限定（见上文，基于容量回收）。使用软引用值的缓存同样用==而不是equals比较值。
**显式清除**

任何时候，你都可以显式地清除缓存项，而不是等到它被回收：
个别清除：Cache.invalidate(key)
批量清除：Cache.invalidateAll(keys)
清除所有缓存项：Cache.invalidateAll()
##  移除监听器 
通过` CacheBuilder.removalListener(RemovalListener) `，你可以声明一个监听器，以便缓存项被移除时做一些额外操作。缓存项被移除时，` RemovalListener `会获取移除通知[` RemovalNotification `]，其中包含移除原因[` RemovalCause `]、键和值。
```

CacheLoader<Key, DatabaseConnection> loader = new CacheLoader<Key, DatabaseConnection> () {
    public DatabaseConnection load(Key key) throws Exception {
        return openConnection(key);
    }
};

RemovalListener<Key, DatabaseConnection> removalListener = new RemovalListener<Key, DatabaseConnection>() {
    public void onRemoval(RemovalNotification<Key, DatabaseConnection> removal) {
        DatabaseConnection conn = removal.getValue();
        conn.close(); // tear down properly
    }
};

return CacheBuilder.newBuilder()
    .expireAfterWrite(2, TimeUnit.MINUTES)
    .removalListener(removalListener)
    .build(loader);

```
警告：默认情况下，监听器方法是在移除缓存时同步调用的。因为缓存的维护和请求响应通常是同时进行的，**代价高昂的监听器方法在同步模式下会拖慢正常的缓存请求**。在这种情况下，你可以使用` RemovalListeners.asynchronous(RemovalListener, Executor) `把监听器装饰为异步操作。
##  刷新 
刷新和回收不太一样。正如` LoadingCache.refresh(K) `所声明，刷新表示为键加载新值，这个过程可以是**异步**的。在刷新操作进行时，缓存仍然可以向其他线程返回旧值
重载CacheLoader.reload(K, V)可以扩展刷新时的行为，这个方法允许开发者在计算新值时使用旧的值。
CacheBuilder.refreshAfterWrite(long, TimeUnit)可以为缓存增加自动定时刷新功能。和expireAfterWrite相反，refreshAfterWrite通过定时刷新可以让缓存项保持可用，但请注意：缓存项只有在被检索时才会真正刷新（如果CacheLoader.refresh实现为异步，那么检索不会被刷新拖慢）。因此，如果你在缓存上同时声明expireAfterWrite和refreshAfterWrite，缓存并不会因为刷新盲目地定时重置，如果缓存项没有被检索，那刷新就不会真的发生，缓存项在过期时间后也变得可以回收。
##  asMap视图 
asMap视图提供了缓存的ConcurrentMap形式，但asMap视图与缓存的交互需要注意：
cache.asMap()包含当前所有加载到缓存的项。因此相应地，cache.asMap().keySet()包含当前所有已加载键;
asMap().get(key)实质上等同于cache.getIfPresent(key)，而且不会引起缓存项的加载。这和Map的语义约定一致。
所有读写操作都会重置相关缓存项的访问时间，包括Cache.asMap().get(Object)方法和Cache.asMap().put(K, V)方法，但不包括Cache.asMap().containsKey(Object)方法，也不包括在Cache.asMap()的集合视图上的操作。比如，遍历Cache.asMap().entrySet()不会重置缓存项的读取时间。
