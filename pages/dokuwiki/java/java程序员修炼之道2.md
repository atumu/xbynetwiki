title: java程序员修炼之道2 

#  Java程序员修炼之道之现代并发 
Java线程模型建立在两个基本概念之上:
  * 共享的、默认可见的可变状态
  * 抢占式线程调度
并发设计原则:
  * 安全性
  * 活跃度
  * 性能
  * 重用性
导致系统永久故障的常见原因:
  * 死锁
  * 不可恢复的资源问题(如NFS不可访问)
  * 信号丢失
并发系统开销之源:
  * 锁与检测
  * 线程上下文切换的次数
  * 线程的个数

##  Java内存模型(JMM) 
Java语言规范(JLS)在第17.4节中介绍了JMM
**JMM两个基本概念:**
  * ` 之前发生(Happens-Before)——这种关系表明一段代码块在其他代码块开始之前就已经全部完成了。 `
  * ` 同步约束(Synchronizeds-With)——这意味着动作继续执行之前必须把它的对象视图与主内存进行同步。 `
**JMM主要规则如下:**
  * ` 在synchronized对象上的解锁与后续的加锁操作之间存在同步约束关系。 `
  * ` 对volatile变量的写入与后续对该变量的读取之间存在同步约束关系。 `
  * ` 如果动作A受到动作B的同步约束，则A在B之前发生。即A Happens-Before B `
  * 如果单线程内存在依赖关系的A和B(存在依赖关系就不会被重排序了)，如果A出现在B之前，则A在B之前发生。
  * ` Happens-Before具有传递性。 `
##  块结构并发(synchronized) 
**临界区**:任何一个对象的同步块或方法中，每次只能有一个线程进入，如果其他线程试图进入，JVM会挂起它们，无论其他线程试图进入的是该对象的同一同步块还是不同的同步块。这种结构结构在并发理论中被称为临界区
Java中同步和锁相关的一些基本事实:
  * 只能锁定对象，不能锁定原始类型
  * 同步方法可以视为包含整个方法的synchronize(this){..}代码块
  * 静态同步方法会锁定它的Class对象。
  * 内部类的同步是独立于外部类的。
  * 非同步的方法不查看或关心任何锁的状态，而且在同步方法运行时他们仍能继续运行。
  * Java的线程锁是可重入的。也就是说持有锁的线程在遇到同一个锁的同步点时时可以继续的。
线程的状态模型:
![](/data/dokuwiki/java/pasted/20160326-163437.png)
##  synchronized和volatile、final 
  * synchronized被同步的是什么?被同步的是在不同线程中表示被锁定对象的内存块。任何**对` 被锁定对象 `的修改**在代码块执行完释放同步锁之前被刷新到主存中。**对被锁定对象的任何修改**都是从主内存中读出来的。
  * volatile域遵循如下规则:1、线程所见的值在使用之前总会从主内存中再读出来，2、线程所写的值总会在指令完成之前被刷新到主内存中。
  * volatile变量是真正线程安全的，但是只有写入时不依赖于当前状态(读取的状态)的变量才应该声明为volatile变量。对于要关注当前状态的变量，只能借助线程锁保证其绝对安全性。
  * final不可变性:略。
不可变对象及构造器示例如下:
```

public final class Update implements Comparable<Update> {

  private final Author author;
  private final String updateText;
  private final long createTime;

  public Author getAuthor() {
    return author;
  }

  public String getUpdateText() {
    return updateText;
  }

  private Update(Builder b_) {
    author = b_.author;
    updateText = b_.updateText;
    createTime = b_.createTime;
  }

  public static class Builder implements ObjBuilder<Update> {
    public long createTime;
    private Author author;
    private String updateText;

    public Builder author(Author author_) {
      author = author_;
      return this;
    }

    public Builder updateText(String updateText_) {
      updateText = updateText_;
      return this;
    }

    public Builder createTime(long createTime_) {
      createTime = createTime_;
      return this;
    }

    public Update build() {
      return new Update(this);
    }
  }

  public int compareTo(Update other_) {
    if (null == other_)
      throw new NullPointerException();

    return (int) (other_.createTime - this.createTime);
  }
  。。。

```
##  现在并发应用程序的构件 
介绍一下java.util.concurrent中的主要类及相关包，如atomic、locks
##  原子类： 
java.util.concurrent.atomic 利用硬件和操作系统的支持的CAS无线程锁的原子操作。
##  线程锁: 
java.util.concurrent.locks
Lock接口的两个实现类:
  * ReentrantLock
  * ReentrantReadWriteLock
###  锁存器CountDownLatch 
CountDownLatch是一种简单的同步模式，这种模式允许线程在通过同步屏障之前做些少量准备工作、或全部结束之后继续执行主线程。
主要有两个方法:
  * countDown():对计数器减1
  * await():让调用线程在计数器减到0之前一直等待。
```

public class ConnectionPoolTest {
    static ConnectionPool pool  = new ConnectionPool(10);
    // 保证所有ConnectionRunner能够同时开始
    static CountDownLatch start = new CountDownLatch(1);
    // main线程将会等待所有ConnectionRunner结束后才能继续执行
    static CountDownLatch end;

    public static void main(String[] args) throws Exception {
        // 线程数量，可以线程数量进行观察
        int threadCount = 50;
        end = new CountDownLatch(threadCount);
        int count = 20;
        AtomicInteger got = new AtomicInteger();
        AtomicInteger notGot = new AtomicInteger();
        for (int i = 0; i < threadCount; i++) {
            Thread thread = new Thread(new ConnetionRunner(count, got, notGot), "ConnectionRunnerThread");
            thread.start();
        }
        start.countDown(); //确保所有线程同时开始处理任务
        end.await(); //确保所有线程都结束了才继续
        System.out.println("total invoke: " + (threadCount * count));
        System.out.println("got connection:  " + got);
        System.out.println("not got connection " + notGot);
    }

    static class ConnetionRunner implements Runnable {
        int           count;
        AtomicInteger got;
        AtomicInteger notGot;

        public ConnetionRunner(int count, AtomicInteger got, AtomicInteger notGot) {
            this.count = count;
            this.got = got;
            this.notGot = notGot;
        }

        public void run() {
            try {
                start.await();
            } catch (Exception ex) {

            }
            while (count > 0) {
                try {
                    // 从线程池中获取连接，如果1000ms内无法获取到，将会返回null
                    // 分别统计连接获取的数量got和未获取到的数量notGot
                    Connection connection = pool.fetchConnection(1000);
                    if (connection != null) {
                        try {
                            connection.createStatement();
                            connection.commit();
                        } finally {
                            pool.releaseConnection(connection);
                            got.incrementAndGet();
                        }
                    } else {
                        notGot.incrementAndGet();
                    }
                } catch (Exception ex) {
                } finally {
                    count--;
                }
            }
            end.countDown();
        }
    }
}

```
###  ConcurrentHashMap 
继承自ConcureentMap的主要方法:
  * putIfAbsent()
  * remove()
  * replace()
##  CopyOnWriteArrayList 
适用范围:读远远大于写。
保证迭代时迭代器不用担心会碰到意料之外的修改。
写时复制保证快速、一直的数据快照
```

public class MicroBlogTimeline {
  private final CopyOnWriteArrayList<Update> updates;
  private final Lock lock;
  private final String name;
  private Iterator<Update> it;

  public MicroBlogTimeline(String name_) {
    name = name_;
    updates = new CopyOnWriteArrayList<>();
    lock = new ReentrantLock();
  }

  MicroBlogTimeline(String name_, CopyOnWriteArrayList<Update> l_, Lock lock_) {
    name = name_;
    updates = l_;
    lock = lock_;
  }

  public void addUpdate(Update update_) {
    updates.add(update_); //修改
  }

  public void prep() {
    it = updates.iterator();
  }

  public void printTimeline() {
    lock.lock();  //需要在此锁住
    try {
      if (it != null) {
        System.out.print(name + ": ");
        while (it.hasNext()) { //即使迭代过程中某个线程调用addUpdate发生修改也不用担心。如果不用CopyOnWriteArrayList，那么此时就会抛出ConcurrentModificationException异常
          Update s = it.next();
          System.out.print(s + ", ");
        }
        System.out.println();
      }
    } finally {
      lock.unlock();
    }
  }
}

```
##  Queue队列 
队列经常用来在线程之间传递工作单元。
1、BlockingQueue
两个主要方法:put(),get()及其超时版本
两个实现:ArrayBlockingQueue,LinkedBlockingQueue

2、使用工作单元
不推荐直接使用BlockingQueue<E>形式，而是推荐使用BlockingQueue<W<E> >
其中W代表工作单元接口，E代表具体的工作。这样就会包含额外的元数据了。
```

Appointment相当于工作单元，可以定义额外元数据
public class Appointment<T> {
  private final T toBeSeen;

  public T getPatient() {
    return toBeSeen;
  }

  public Appointment(T incoming) {
    toBeSeen = incoming;
  }
}


```
```

public class Cat extends Pet {
  public Cat(String name) {
    super(name);
  }

  public void examine() {
    System.out.println("Meow!");
  }
}

```
```

public class Veterinarian extends Thread {
  protected final BlockingQueue<Appointment<Pet>> appts;
  protected String text = "";
  protected final int restTime;
  private boolean shutdown = false;

  public Veterinarian(BlockingQueue<Appointment<Pet>> lbq, int pause) {
    appts = lbq;
    restTime = pause;
  }

  public synchronized void shutdown() {
    shutdown = true;
  }

  @Override
  public void run() {
    while (!shutdown) {
      seePatient();
      try {
        Thread.sleep(restTime);
      } catch (InterruptedException e) {
        shutdown = true;
      }
    }
  }

  public void seePatient() {
    try {
      Appointment<Pet> ap = appts.take();
      Pet patient = ap.getPatient();
      patient.examine();
    } catch (InterruptedException e) {
      shutdown = true;
    }
  }
}

```
###  5、TransferQueue 
TransferQueue是Java7中的新贵。本质上是多了一项transfer()操作的BlockingQueue。
如果接受线程处于等待状态，该操作会马上把工作项传给它，否则就会阻塞直到取走工作项的线程开始。这样就可以控制上游生产者线程生产新工作项的速度了。
##  控制执行 
Callable：
Future接口：主要方法get(),cancel(),isDone()
FutureTask类:Future接口的常用实现类，它也实现了Runnable接口。


