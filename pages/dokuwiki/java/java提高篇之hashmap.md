title: java提高篇之hashmap 

#  java提高篇之HashMap 
HashMap也是我们使用非常多的Collection，它是**基于哈希表**的 Map 接口的实现，以key-value的形式存在。在HashMap中，**key-value总是会当做一个整体来处理**，系统会根据hash算法来来计算key-value的存储位置，我们总是可以通过key快速地存、取value。下面就来分析HashMap的存取。
在官方文档中是这样描述HashMap的：
Hash table based implementation of the Map interface**. This implementation provides all of the optional map operations, and permits null values and the null key**. (The HashMap class is roughly equivalent to Hashtable, except that it is unsynchronized and permits nulls.) T**his class makes no guarantees as to the order of the map; in particular, it does not guarantee that the order will remain constant over time.**
几个关键的信息：` 基于Map接口实现、允许null键/值、非同步、不保证有序(比如插入的顺序)、也不保证序不随时间变化。 `
**两个重要的参数**
在HashMap中有两个很重要的参数，容量(Capacity)和负载因子(Load factor)
Capacity就是bucket的大小，Load factor就是bucket填满程度的最大比例。如果对迭代性能要求很高的话不要把capacity设置过大，也不要把load factor设置过小。
**当bucket中的entries的数目大于capacity*load factor时就需要调整bucket的大小为当前的2倍**。


一、定义
HashMap实现了Map接口，继承AbstractMap。其中Map接口定义了键映射到值的规则，而AbstractMap类提供 Map 接口的骨干实现，以最大限度地减少实现此接口所需的工作，其实AbstractMap类已经实现了Map，这里标注Map LZ觉得应该是更加清晰吧！
```

public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable

```
**二、构造函数**
HashMap提供了三个构造函数：
  * HashMap()：构造一个具有默认初始容量 (16) 和默认加载因子 (0.75) 的空 HashMap。
  * HashMap(int initialCapacity)：构造一个带指定初始容量和默认加载因子 (0.75) 的空 HashMap。
  * HashMap(int initialCapacity, float loadFactor)：构造一个带指定初始容量和加载因子的空 HashMap。

在这里提到了**两个参数：初始容量，加载因子**。这两个参数是影响HashMap**性能**的重要参数，其中**容量表示哈希表中` 桶 `的数量**，初始容量是创建哈希表时的容量，**加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度，它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小**。对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果**负载因子**越大，对**空间的利用**更充分，然而后果是**查找效率**的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。系统默认负载因子为0.75，一般情况下我们是无需修改的。

HashMap是一种**支持快速存取**的数据结构，要了解它的性能必须要了解它的数据结构。

##  三、数据结构 
我们知道在Java中最常用的两种结构是数组和模拟指针(引用)，几乎所有的数据结构都可以利用这两种来组合实现，HashMap也是如此。实际上HashMap是一个“链表散列”，如下是它数据结构：
![](/data/dokuwiki/java/pasted/20160531-145102.png)
从上图我们可以看出**HashMap底层实现还是数组，只是数组的每一项都是一条链**。其中参数initialCapacity就代表了该数组的长度。下面为HashMap构造函数的源码：
```

public HashMap(int initialCapacity, float loadFactor) {
        //初始容量不能<0
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: "
                    + initialCapacity);
        //初始容量不能 > 最大容量值，HashMap的最大容量值为2^30
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //负载因子不能 < 0
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: "
                    + loadFactor);

        // 计算出大于 initialCapacity 的最小的 2 的 n 次方值。
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;
        
        this.loadFactor = loadFactor;
        //设置HashMap的容量极限，当HashMap的容量达到该极限时就会进行扩容操作
        threshold = (int) (capacity * loadFactor);
        //初始化table数组
        table = new Entry[capacity];
        init();
    }

```
 从源码中可以看出，每次新建一个HashMap时，都会初始化一个table数组。table数组的元素为**Entry**节点。
```

static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        final int hash;

        /**
         * Creates new entry.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
        .......
    }

```
其中**Entry为HashMap的内部类，它包含了键key、值value、下一个节点next，以及hash值**，这是非常重要的，**正是由于Entry才构成了table数组的项为链表。**
 上面简单分析了HashMap的数据结构，下面将探讨HashMap是如何实现快速存取的。

##  四、存储实现：put(key,vlaue) 
![](/data/dokuwiki/java/pasted/20160531-150415.png)

put函数大致的思路为：
  * 对key的hashCode()做hash，然后再计算index;
  * 如果没碰撞直接放到bucket里；
  * 如果碰撞了，以链表的形式存在buckets后；
  * 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；
  * 如果节点已经存在就替换old value(保证key的唯一性)
  * 如果bucket满了(超过load factor*current capacity)，就要resize。

首先我们先看源码
```

public V put(K key, V value) {
        //当key为null，调用putForNullKey方法，保存null与table第一个位置中，这是HashMap允许为null的原因
        if (key == null)
            return putForNullKey(value);
        //计算key的hash值
        int hash = hash(key.hashCode());                  ------(1)
        //计算key hash 值在 table 数组中的位置
        int i = indexFor(hash, table.length);             ------(2)
        //从i出开始迭代 e,找到 key 保存的位置
        for (Entry<K, V> e = table[i]; e != null; e = e.next) {
            Object k;
            //判断该条链上是否有hash值相同的(key相同)
            //若存在相同，则直接覆盖value，返回旧value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;    //旧值 = 新值
                e.value = value;
                e.recordAccess(this);
                return oldValue;     //返回旧值
            }
        }
        //修改次数增加1
        modCount++;
        //将key、value添加至i位置处
        addEntry(hash, key, value, i);
        return null;
    }

```
通过源码我们可以清晰看到**HashMap保存数据的过程为：
首先判断key是否为null，若为null，则直接调用putForNullKey方法。若不为空则先计算key的hash值，然后根据hash值搜索在table数组中的索引位置，如果table数组在该位置处有元素，则通过比较是否存在相同的key，若存在则覆盖原来key的value，否则将该元素保存在链头（最先保存的元素放在链尾）。若table在该处没有元素，则直接保存。**
这个过程看似比较简单，其实深有内幕。有如下几点：
1、 先看迭代处。此处迭代原因就是为了防止存在相同的key值，若发现两个hash值（key）相同时，HashMap的处理方式是用新value替换旧value，这里并没有处理key，这就解释了HashMap中没有两个相同的key。
2、 在看（1）、（2）处。这里是HashMap的精华所在。首先是hash方法，该方法为一个纯粹的数学计算，就是计算h的hash值。
```

static int hash(int h) {
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

```
我们知道**对于HashMap的table而言，数据分布需要` 均匀 `（最好每项都只有一个元素，这样就可以直接找到），不能太紧也不能太松，太紧会导致查询速度慢，太松则浪费空间**。计算hash值后，怎么才能保证table元素分布均与呢？我们会想到取模，但是由于取模的消耗较大，HashMap是这样处理的：调用indexFor方法。
```

static int indexFor(int h, int length) {
        return h & (length-1);
    }

```
**HashMap的底层数组长度总是2的n次方**，在构造函数中存在：capacity <<= 1;这样做总是能够保证HashMap的底层数组长度为2的n次方。**当length为2的n次方时，h&(length – 1)就相当于对length取模，而且速度比直接取模快得多**，这是HashMap在速度上的一个优化。至于为什么是2的n次方下面解释。
我们回到indexFor方法，该方法仅有一条语句：h&(length – 1)，这句话除了上面的取模运算外还有一个非常重要的责任：**均匀分布table数据和充分利用空间。**
这里我们假设length为16(2^n)和15，h为5、6、7。
![](/data/dokuwiki/java/pasted/20160531-145612.png)
**当n=15时，6和7的结果一样，这样表示他们在table存储的位置是相同的，也就是产生了碰撞，6、7就会在一个位置形成链表，这样就会导致查询速度降低。**诚然这里只分析三个数字不是很多，那么我们就看0-15。
![](/data/dokuwiki/java/pasted/20160531-145637.png)
从上面的图表中我们看到**总共发生了8此碰撞，同时发现浪费的空间非常大**，有1、3、5、7、9、11、13、15处没有记录，也就是没有存放数据。这是因为他们在与14进行&运算时，得到的结果最后一位永远都是0，即0001、0011、0101、0111、1001、1011、1101、1111位置处是不可能存储数据的，**空间减少，进一步增加碰撞几率，这样就会导致查询速度慢**。而当length = 16时，length – 1 = 15 即1111，那么进行低位&运算时，值总是与原来hash值相同，而进行高位运算时，其值等于其低位值。所以说当length = 2^n时，不同的hash值发生碰撞的概率比较小，这样就会使得数据在table数组中分布较均匀，查询速度也较快。
在get和put的过程中，计算下标时，先对hashCode进行hash操作，然后再通过hash值进一步计算下标，如下图所示：
![](/data/dokuwiki/java/pasted/20160531-151603.png)
这里我们再来复习**put的流程**：
  * 当我们想一个HashMap中添加一对key-value时，系统**首先会计算key的hash值**，然后根据hash值确认在table中存储的位置。
  * 若该位置没有元素，则直接插入。否则迭代该处元素链表并依此比较其key的hash值。
  * 如果两个hash值相等且key值相等(e.hash == hash && ( ( k = e.key) == key || key.equals(k))),则用新的Entry的value覆盖原来节点的value。
  * 如果两个hash值相等但key值不等 ，则将该节点插入该链表的链头。

具体的实现过程见addEntry方法，如下：
```

void addEntry(int hash, K key, V value, int bucketIndex) {
        //获取bucketIndex处的Entry
        Entry<K, V> e = table[bucketIndex];
        //将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry 
        table[bucketIndex] = new Entry<K, V>(hash, key, value, e);
        //若HashMap中元素的个数超过极限了，则容量扩大两倍
        if (size++ >= threshold)
            resize(2 * table.length);
    }

```
这个方法中有两点需要注意：
**一是链的产生。**
这是一个非常优雅的设计。系统总是将新的Entry对象添加到bucketIndex处。如果bucketIndex处已经有了对象，那么新添加的Entry对象将指向原有的Entry对象，**形成一条Entry链**，但是若bucketIndex处没有Entry对象，也就是e==null,那么新添加的Entry对象指向null，也就不会产生Entry链了。
** 二、扩容问题。**
随着HashMap中元素的数量越来越多，发生碰撞的概率就越来越大，所产生的链表长度就会越来越长，这样势必会影响HashMap的速度，**为了保证HashMap的效率，系统必须要在某个临界点进行扩容处理。该临界点在当HashMap中元素的数量等于table数组长度*加载因子。**但是扩容是一个非常耗时的过程，因为它需要重新计算这些数据在新table数组中的位置并进行复制处理。所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。


##  五、读取实现：get(key) 
相对于HashMap的存而言，取就显得比较简单了。通过key的hash值找到在table数组中的索引处的Entry，然后返回该key对应的value即可。

```

public V get(Object key) {
        // 若为null，调用getForNullKey方法返回相对应的value
        if (key == null)
            return getForNullKey();
        // 根据该 key 的 hashCode 值计算它的 hash 码  
        int hash = hash(key.hashCode());
        // 取出 table 数组中指定索引处的值
        for (Entry<K, V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
            Object k;
            //若搜索的key与查找的key相同，则返回相对应的value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                return e.value;
        }
        return null;
    }

```
在这里能够根据key快速的取到value除了和HashMap的数据结构密不可分外，还和Entry有莫大的关系，在前面就提到过，**HashMap在存储过程中并没有将key，value分开来存储，而是当做一个整体key-value来处理的，这个整体就是Entry对象。**同时value也只相当于key的附属而已。
**在存储的过程中，系统根据key的hashcode来决定Entry在table数组中的存储位置，在取的过程中同样根据key的hashcode取出相对应的Entry对象。**


##  resize的实现 
当put时，如果发现目前的bucket占用程度已经超过了Load Factor所希望的比例，那么就会发生resize。` 在resize的过程，简单的说就是把bucket扩充为2倍，之后重新计算index，把节点再放到新的bucket中 `。resize的注释是这样描述的：
Initializes or doubles table size. If null, allocates in accord with initial capacity target held in field threshold. Otherwise, because we are using power-of-two expansion, the elements from each bin must either stay at same index, or move with a power of two offset in the new table.
大致意思就是说，当超过限制的时候会resize，然而又因为我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。
怎么理解呢？例如我们从16扩展为32时，具体的变化如下所示：
![](/data/dokuwiki/java/pasted/20160531-152254.png)
因此元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：
![](/data/dokuwiki/java/pasted/20160531-152545.png)
因此，我们在扩充HashMap的时候，不需要重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”。可以看看下图为16扩充为32的resize示意图：
![](/data/dokuwiki/java/pasted/20160531-152612.png)
这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。
```

final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
 
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}

```

##  HashMap常见问题总结 

**你知道HashMap的工作原理吗？**
通过hash的方法，通过put和get存储和获取对象。存储对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过Load Facotr则resize为原来的2倍)。获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。
你知道get和put的原理吗？**equals()和hashCode()的都有什么作用？**
答：通过对key的hashCode()进行hashing，并计算下标( n-1 & hash)，从而获得buckets的位置。如果产生碰撞，则利用key.equals()方法去链表或树中去查找对应的节点。
**为什么String, Interger这样的wrapper类适合作为键？**
String, Interger这样的wrapper类作为HashMap的键是再适合不过了，而且String最为常用。**因为String是不可变的，也是final的，而且已经重写了equals()和hashCode()方法了。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。**不可变性还有其他的优点如**线程安全**。如果你可以仅仅通过将某个field声明成final就能保证hashCode是不变的，那么请这么做吧。**因为获取对象的时候要用到` equals()和hashCode() `方法**，那么**键对象正确的重写这两个方法是非常重要的**。如果两个不相等的对象返回不同的hashcode的话，那么**碰撞的几率**就会小些，**这样就能提高HashMap的性能。**
**5. 如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？**
如果超过了负载因子(默认0.75)，则会重新resize一个原来长度两倍的HashMap，并且重新调用hash方法。

参考：
http://www.cnblogs.com/chenssy/p/3521565.html
http://www.importnew.com/18604.html
http://www.importnew.com/18633.html