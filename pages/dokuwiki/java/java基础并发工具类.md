title: java基础并发工具类 

#  java基础并发工具类 
Java平台类库提供了很多并发工具类供我们使用。
##  同步容器类 
同步容器类包括古老的Vector和Hashtable，还有通过Collections.synchronizedXxx方法创建的。这些类实现线程安全的方式是：将它们的状态封装起来，并对每个公有方法进行同步，使得每次都只有一个线程能访问容器的状态。（提示：这在高并发的情况下是不允许的。这种方式虽然能够进行同步，也是线程安全的，但是效率非常低。性能很差）
同步容器类还存在一些问题：例如客户端先检查后执行会出现问题。如下：
```

public static Object getLast(Vector list){
	int lastIndex=list.size()-1;   这里暗含了一个先检查后执行的条件。
	return list.get(lastIndex)  如果在执行get方法时，list.size()已经改变了。那么它将抛出ArrayIndexOutOfBoundsException
}
public static void deleteLast(Vector list){
	int lastIndex=list.size()-1;   这里暗含了一个先检查后执行的条件。
	list.remove(lastIndex);
}

```
![](/data/dokuwiki/java/pasted/20150814-082308.png)
由于同步容器类遵守同步策略，即支持客户端加锁。因此只要我们知道应该使用哪一个锁。
同步容器类通过自身的锁来保护它的每一个方法，那么我们就可以通过获取容器类的锁来进行客户端加锁。
```

public static Object getLast(Vector list){
  synchonize(list){
	int lastIndex=list.size()-1; 
	return list.get(lastIndex);
  }
}
public static void deleteLast(Vector list){
  synchonize(list){
	int lastIndex=list.size()-1;   
	list.remove(lastIndex);
  }
}

```
###  迭代器不可靠迭代与ConcurrentModificationException异常 

**还有一个比较严重的问题就是迭代操作中也会存在问题，发生不可靠迭代。**而且同步容器返回的迭代器是”即时失败的“，它会通过抛出ConcurrentModificationException。
**要解决这个问题就必须要在迭代器迭代期间对整个容器进行加锁，而这通常是无法接受的。**
那么我们只能寻求java5给出的并发容器类。
##  并发容器 
Java 5提供了多种并发容器类来改进同步容器的性能。
新增了ConcurrentHashMap,CopyOnWriteArrayList,CopyOnWriteArraySet,ConcurrentLinkedQueue和BlockingQueue等并发容器。
Java6又引入了ConcurrentSkipListMap和ConcurrentSkipListSet,用于替代SortedMap和SortedSet
###  ConcurrentHashMap 
ConcurrentHashMap使用一种称为分段锁（Lock Striping）的粒度更细的加锁机制来实现更大程度的并发性能。在这种机制中，任意数量的读取线程可以并发地访问Map，执行读取操作的线程和执行写入操作的线程可以并发地访问Map，并且一定数量的写入线程可以并发地修改Map。以实现更高的吞吐量。而在单线程环境中也只损失非常小的性能。
**ConcurrentHashMap与其他的并发容器类所提供的迭代器不会平抛出ConcurrentModificationException。因此无需对迭代过程进行加锁。返回的迭代器具有弱一致性（Weakly Consistent）.而非”及时失败“。弱一致性的迭代器允许并发修改。**
不足：一些需要在整个Map上进行的计算方法如**size,isEmpty等被减弱了。一般只能返回近似值而非准确值**。不过这也是有道理的，因为一般并发环境下修改是很频繁地。
###  CopyOnWriteArrayList 
它能提供很好的并发性能，并且**在迭代期间不需要对容器进行加锁或复制。**
它的原理是在每次修改时都会创建并重新发布一个新的容器副本，从而实现可变性。
不足：当数据量大的时候，会存在一定的性能开销和内存开销。只适用于迭代操作远多于修改操作时才应该使用它。
使用场景举例：事件通知系统中，注册监听器的操作远少于接收事件通知的操作。

###  BlockingQueue和生产者-消费者模式 
BlockingQueue提供了可阻塞的put和get方法。以及支持定时的offer和poll方法。如果队列已经满了，那么put方法将阻塞直到有空间可用，如果队列为空，那么take方法将阻塞直到有元素可用。
**阻塞队列支持生产者-消费者**这种设计模式。生产者-消费者模式能简化开发过程，因为它消除了生产者类和消费者类之间的代码依赖性。
一种常见的生产者-消费者模式就是线程池与工作队列组合，在Executor任务执行框架中就体现了这种模式。
如果默认的阻塞队列不符合你的要求，你可以使用信号量(Semaphore)来创建其他的阻塞数据结构。
**在类库中，有多种BlockingQueue的实现：如：**
  * FIFO队列的：ArrayBlockingQueue,LinkedBlockingQueue
  * 优先级队列:PriorityBlockingQueue
  * 线程队列： SynchronousQueue
```

 class Producer implements Runnable {
   private final BlockingQueue queue;
   Producer(BlockingQueue q) { queue = q; }
   public void run() {
     try {
       while (true) { queue.put(produce()); }
     } catch (InterruptedException ex) { ... handle ...}
   }
   Object produce() { ... }
 }

 class Consumer implements Runnable {
   private final BlockingQueue queue;
   Consumer(BlockingQueue q) { queue = q; }
   public void run() {
     try {
       while (true) { consume(queue.take()); }
     } catch (InterruptedException ex) { ... handle ...}
   }
   void consume(Object x) { ... }
 }

 class Setup {
   void main() {
     BlockingQueue q = new SomeQueueImplementation();
     Producer p = new Producer(q);
     Consumer c1 = new Consumer(q);
     Consumer c2 = new Consumer(q);
     new Thread(p).start();
     new Thread(c1).start();
     new Thread(c2).start();
   }
 }

```
###  串行线程封闭 
**阻塞队列与生产者-消费者模式一起，促进了串行线程封闭。**
对象池就利用了**串行线程封闭**，将对象”借给“一个请求线程。只要对象池内包含足够的内部同步机制来安全地发布池中的对象。并且只要客户代码不会发布池中的对象，或者在将对象返回给对象池之后就不再使用它，那么就可以安全地在线程之间传递所有权。

###  双端队列Deque与工作密取 
略。。。
##  同步工具类 
同步工具类可以是任何一个对象，只要它根据其自身的状态来协调线程的控制流。阻塞队列可以作为同步工具类。其他类型的同步工具类还包括信号量Semaphore、闭锁Latch、栅栏Barrier等
###  闭锁 
闭锁是一种同步工具类，可以延迟线程的进度直到其到达终止状态。闭锁的作用相当于一扇门：在闭锁达到结束状态之前，这扇门一致是关闭的，并且没有任何线程能过通过，当达到结束状态时，这扇门会打开并允许所有线程通过。当闭锁到达结束状态后，将不会再改变状态，因此这扇门将永远保持打开状态。
**闭锁可以用来确保某些活动直到其他活动都完成后才继续执行。**
例如：lol中等到所有玩家都达到100%状态之后游戏才能正式开始。
` CountDownLatch `是一种灵活的闭锁实现。它可以使一个或多个线程等待一组事件发生，闭锁状态包括一个**计数器**，该计数器被初始化为一个正数表示需要等待的事件数量。
` countDown() `方法则递减计数器，表示有一个事件已经发生了，
而` await `方法等待计数器一直到达到0，这表示所有需要等待的事件都已经发生。
以下示例：
这里设计了两个闭锁，分别表示启动门和结束门。每个工作线程首先要做的是在启动门上等待，从而确保所有线程都处于就绪状态才开始执行，而每个线程最后咬嘴的一件事就是调用结束门的countDown方法减一，这能使主线程在所有工作线程都完成后统计耗时。
```

public class TestHarness {
    public long timeTasks(int nThreads, final Runnable task)
            throws InterruptedException {
        final CountDownLatch startGate = new CountDownLatch(1);
        final CountDownLatch endGate = new CountDownLatch(nThreads);

        for (int i = 0; i < nThreads; i++) {
            Thread t = new Thread() {
                public void run() {
                    try {
                        startGate.await();
                        try {
                            task.run();
                        } finally {
                            endGate.countDown();
                        }
                    } catch (InterruptedException ignored) {
                    }
                }
            };
            t.start();
        }

        long start = System.nanoTime();
        startGate.countDown();
        endGate.await();
        long end = System.nanoTime();
        return end - start;
    }
}

```

###  信号量Semaphore 
计数信号量用来控制同时访问某个特定资源或执行某个特定操作的数量。计数**信号量**还可以用来实现某种**资源池**，或者对容器施加边界。
` Semaphore `中管理着一组虚拟的许可(permit),并在使用以后释放许可。如果没有许可，那么` acquire `将阻塞直到有许可（或者因为超时或被中断）。` release `方法将释放一个许可。
若给Semaphore初始值设为1，则可以充当互斥锁。
示例：使用Semaphore为容器设置边界
```

public class BoundedHashSet <T> {
    private final Set<T> set;
    private final Semaphore sem;

    public BoundedHashSet(int bound) {
        this.set = Collections.synchronizedSet(new HashSet<T>());
        sem = new Semaphore(bound);
    }

    public boolean add(T o) throws InterruptedException {
        sem.acquire();
        boolean wasAdded = false;
        try {
            wasAdded = set.add(o);
            return wasAdded;
        } finally {
            if (!wasAdded)
                sem.release();
        }
    }

    public boolean remove(Object o) {
        boolean wasRemoved = set.remove(o);
        if (wasRemoved)
            sem.release();
        return wasRemoved;
    }
}

```
###  栅栏 
暂略。。。。
###  FuntureTask 
略。。。。
##  构建高效且可伸缩的结果缓存 

略。。。