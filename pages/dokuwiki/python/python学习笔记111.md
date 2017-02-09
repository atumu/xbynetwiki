title: python学习笔记111 

#  Python学习笔记之PyQt4之多线程2 
Qt中主线程(Main-Thread)称为GUI线程
##  Qt中线程操作可供选择的方案 
` QEventLoop::processEvents() `
Calling QEventLoop::processEvents() repeatedly during a time-consuming calculation prevents GUI blocking. However, this solution doesn't scale well because the call to processEvents() may occur too often, or not often enough, depending on hardware.
` QTimer `	
Background processing can sometimes be done conveniently using a timer to schedule execution of a slot at some point in the future. A timer with an interval of 0 will time out as soon as there are no more events to process.
` QSocketNotifier QNetworkAccessManager QIODevice::readyRead() `	
This is an alternative to having one or multiple threads, each with a blocking read on a slow network connection. As long as the calculation in response to a chunk of network data can be executed quickly, this reactive design is better than synchronous waiting in threads. Reactive design is less error prone and energy efficient than threading. In many cases there are also performance benefits.

##  QObject、QThread and QThreadPool与线程之间的交互 
` Qt的信号与槽机制可以将` 任何继承自QObject类 `的对象捆绑在一起，使不同对象之间能够相互通信。 `
  * 在线程间使用信号槽进行通信时，需要注意必须使用元数据类型
  * Qt内生的元数据类型，如int double QString 等
  * 如果要用自己定义的数据类型，需要在connect前将其注册为元数据类型。
**1.connect函数的第五个参数代表信号与槽的连接模式**，线程间的信号与槽不能使用Qt::DirectConnection直接连接方式，因为它要求在发信号的线程内执行槽函数。而Qt::QueuedConnection队列方式将信号转换成事件发送到槽函数所在线程的消息队列中让槽函数所在线程来处理，可以实现线程安全的线程间的通信。这样的时效性也不差，上面的实现中，会在子线程“run()”函数中的第二个Sleep之前执行主线程的“OnMsgSignal(constQString& tep2)”。
Qt线程间的信号与槽于是在调试的时候子线程的“emit MsgSignal(const QString& tep);”的下一步并不会立即跳转到主线程的“OnMsgSignal(const QString& tep2)”，我开始还以为信号中途丢了没送达呢。。而强制使用Qt::DirectConnection模式却怎么也摆脱不了错误。

2.线程间用“信号与槽”传递引用参数的话，一定要加const，因为const文字常量存在常量区中，生命周期与程序一样的长。这样可以避免slot调用的时候参数的运行期已过而使引用无效。
我开始没注意到需要加const，程序运行时并没有报告错误，但是主线程毫无使用子线程传递来的字符串的迹象，这让我更加错误地以为子线程发送的信号丢了。。


**传递消息的方式有四个取值：**
` Qt::DirectConnection ` When emitted, the signal is immediately delivered to the slot. 假设当前有4个slot连接到QPushButton::clicked(bool)，当按钮被按下时，QT就把这4个slot按连接的时间顺序调用一遍。**显然这种方式不能跨线程（传递消息**）。

` Qt::QueuedConnection ` When emitted, the signal is queued until the event loop is able to deliver it to the slot. 假设当前有4个slot连接到QPushButton::clicked(bool)，当按钮被按下时，**QT就把这个signal包装成一个QEvent，放到消息队列里。QApplication::exec()或者线程的QThread::exec()会从消息队列里取消息，然后调用 signal关联的几个slot。` 这种方式既可以在线程内传递消息，也可以跨线程传递消息 `。**

` Qt::BlockingQueuedConnection ` Same as QueuedConnection, except that the current thread blocks until the slot has been delivered. This connection type should only be used for receivers in a different thread. Note that misuse of this type can lead to dead locks in your application. 与Qt::QueuedConnection类似，但是会阻塞等到关联的slot都被执行。这里出现了阻塞这个词，说明它是专门用来多线程间传递消息的。

` Qt::AutoConnection(默认值) ` If the signal is emitted from the thread in which the receiving object lives, the slot is invoked directly, as with Qt::DirectConnection; otherwise the signal is queued, as with Qt::QueuedConnection. 这种连接类型根据signal和slot是否在同一个线程里自动选择Qt::DirectConnection或Qt::QueuedConnection

PyQt4里面类的位置PyQt4.Qt.Qt.AutoConnection
  * QObject.connect(QObject, SIGNAL(), QObject, SLOT(), Qt.ConnectionType=Qt.AutoConnection) -> bool
  * QObject.connect(QObject, SIGNAL(), callable, Qt.ConnectionType=Qt.AutoConnection) -> bool
  * QObject.connect(QObject, SIGNAL(), SLOT(), Qt.ConnectionType=Qt.AutoConnection) -> bool

QT通过三种形式提供了对线程的支持。它们分别是，
  * 一、平台无关的线程类，
  * 二、线程安全的事件投递，
  * 三、跨线程的信号-槽连接。
从4.0开始，线程总是有效的。
线程类
  * Qt 包含下面一些线程相关的类：
  * QThread 提供了开始一个新线程的方法
  * QThreadStorage 提供逐线程数据存储
  * QMutex 提供相互排斥的锁，或互斥量
  * QMutexLocker 是一个便利类，它可以自动对QMutex加锁与解锁
  * QReadWriterLock 提供了一个可以同时读操作的锁
  * QReadLocker与QWriteLocker 是便利类，它自动对QReadWriteLock加锁与解锁
  * QSemaphore 提供了一个整型信号量，是互斥量的泛化
  * QWaitCondition 提供了一种方法，使得线程可以在被另外线程唤醒之前一直休眠。
###  创建一个线程 
为创建一个线程，子类化QThread(继承自QObject，所以可以利用信号与槽)并且重写它的run()函数，例如：
之后，创建这个线程对象的实例，调用QThread::start()。于是，在run()里出现的代码将会在另外线程中被执行。
注意：QCoreApplication::exec()启动的主事件循环必须总是在主线程(执行main()的那个线程)中被调用，不能从一个QThread中调用。
在GUI程序中，主线程也被称为GUI线程，因为它是唯一一个允许执行GUI相关操作的线程。` 另外，你必须在创建一个QThread之前创建QApplication(orQCoreApplication)对象。 `

###  线程与QObjects 
` QThread 继承自QObject `,它发射信号以指示线程执行开始与结束，而且也提供了许多slots。更有趣的是，**QObjects可以用于多线程，这是因为` 每个线程被允许有它自己的事件循环 `。**
QObject 可重入性
QObject是可重入的。它的大多数非GUI子类，像QTimer,QTcpSocket,QUdpSocket,QHttp,QFtp,QProcess也是可重入的，在多个线程中同时使用这些类是可能的。需要注意的是，这些类被设计成在一个单线程中创建与使用，因此，在一个线程中创建一个对象，而在另外的线程中调用它的函数，这样的行为不能保证工作良好。**有三种约束需要注意：**
1，**QObject的孩子总是应该在它父亲被创建的那个线程中创建**。这意味着，你绝不应该传递QThread对象作为另一个对象的父亲(因为QThread对象本身会在另一个线程中被创建)
2,事件驱动对象仅仅在单线程中使用。明确地说，这个规则适用于"定时器机制“与”网格模块“，举例来讲，你不应该在一个线程中开始一个定时器或是连接一个套接字，当这个线程不是这些对象所在的线程。
3，你必须保证在线程中创建的所有对象在你删除QThread前被删除。这很容易做到:你可以run()函数运行的栈上创建对象。

尽管QObject是可重入的，但GUI类，特别是QWidget与它的所有子类都是不可重入的。它们仅用于主线程。正如前面提到过的，QCoreApplication::exec()也必须从那个线程中被调用。实践上，不会在别的线程中使用GUI类，它们工作在主线程上，把一些耗时的操作放入独立的工作线程中，当工作线程运行完成，把结果在主线程所拥有的屏幕上显示。

###  逐线程事件循环 
` 每个线程可以有它的事件循环，GUI线程开始它的事件循环需使用QCoreApplication::exec(),别的线程开始它的事件循环需要用QThread::exec(). `像QCoreApplication一样，QThreadr提供了exit(int)函数，一个quit() slot。
线程中的事件循环，使得线程可以使用那些需要事件循环的非GUI 类(如，QTimer,QTcpSocket,QProcess)。` 也可以把任何线程的signals连接到特定线程的slots，也就是说信号-槽机制是可以跨线程使用的。 `对于在QApplication之前创建的对象，QObject::thread()返回0,这意味着主线程仅为这些对象处理投递事件，不会为没有所属线程的对象处理另外的事件。可以用QObject::moveToThread()来改变它和它孩子们的线程亲缘关系，假如对象有父亲，它不能移动这种关系。在另一个线程(而不是创建它的那个线程)中deleteQObject对象是不安全的。除非你可以保证在同一时刻对象不在处理事件。可以用QObject::deleteLater(),它会投递一个DeferredDelete事件，这会被对象线程的事件循环最终选取到。
**假如没有事件循环运行，事件不会分发给对象。**举例来说，假如你在一个线程中创建了一个QTimer对象，但从没有调用过exec(),那么QTimer就不会发射它的timeout()信号.对deleteLater()也不会工作。(这同样适用于主线程)。` 你可以手工使用线程安全的函数QCoreApplication::postEvent()，在任何时候，给任何线程中的任何对象投递一个事件，事件会在那个创建了对象的线程中通过事件循环派发。 `事件过滤器在所有线程中也被支持，不过它限定被监视对象与监视对象生存在同一线程中。类似地，QCoreApplication::sendEvent(不是postEvent()),仅用于在调用此函数的线程中向目标对象投递事件。

###  从别的线程中访问QObject子类 
QObject和所有它的子类是非线程安全的。这包括整个的事件投递系统。需要牢记的是，当你正从别的线程中访问对象时，事件循环可以向你的QObject子类投递事件。假如你调用一个不生存在当前线程中的QObject子类的函数时，你必须用mutex来保护QObject子类的内部数据，否则会遭遇灾难或非预期结果。像其它的对象一样，QThread对象生存在创建它的那个线程中- - -不是当QThread::run()被调用时创建的那个线程。
**一般来讲，在你的QThread子类中提供slots是不安全的，除非你用mutex保护了你的成员变量。**
另一方面，**你可以安全的从QThread::run()的实现中发射信号，因为信号发射是线程安全的。**

###  跨线程的信号-槽 
Qt支持三种类型的信号-槽连接：
  * 1，直接连接，当signal发射时，slot立即调用。此slot在发射signal的那个线程中被执行(不一定是接收对象生存的那个线程)
  * 2，队列连接，当控制权回到对象属于的那个线程的事件循环时，slot被调用。此slot在接收对象生存的那个线程中被执行
  * 3，自动连接(缺省)，假如信号发射与接收者在同一个线程中，其行为如直接连接，否则，其行为如队列连接。
连接类型可能通过以向connect()传递参数来指定。注意的是，当发送者与接收者生存在不同的线程中，而事件循环正运行于接收者的线程中，使用直接连接是不安全的。同样的道理，调用生存在不同的线程中的对象的函数也是不是安全的。QObject::connect()本身是线程安全的。

多线程与隐含共享
Qt为它的许多值类型使用了所谓的隐含共享(implicitsharing)来优化性能。原理比较简单，共享类包含一个指向共享数据块的指针，这个数据块中包含了真正原数据与一个引用计数。把深拷贝转化为一个浅拷贝，从而提高了性能。这种机制在幕后发生作用，程序员不需要关心它。如果深入点看，假如对象需要对数据进行修改，而引用计数大于1，那么它应该先detach()。以使得它修改不会对别的共享者产生影响，既然修改后的数据与原来的那份数据不同了，因此不可能再共享了，于是它先执行深拷贝，把数据取回来，再在这份数据上进行修改。
###  QThread::postEvent() 
在 QThread 类中有一种从非主事件线程中将事件提交给一个对象的方法，也就是 QThread::postEvent()方法，该方法提供了Qt 中的一种** Thread-safe 的事件提交过程。**
提交的事件被放进一个队列中，然后 GUI 主事件线程被唤醒并将此事件发给相应的对象，这个过程与一般的窗口系统事件处理过程是一样的。
**当事件处理过程被调用时，是在主事件线程中被调用的，而不是在调用QThread::postEvent 方法的线程中被调用。**
####  1.、系统定义的事件的提交 
在 Qt 系统中，定义了很多种类的事件，如定时器事件、鼠标移动事件、键盘事件、窗口控件事件等。通常，事件都来自底层的窗口系统。
**Qt 的主事件循环函数从系统的事件队列中获取这些事件，并将它们转换为 ` QEvent `，然后传给相应的`  QObjects ` 对象。**
如下所示：
```

QWidget *mywidget;
void MyThread::run()
{
   QThread::postEvent(MyWidget, new QPaintEvent(QRect(0,0,100,100)));
}

```
在MyThread线程中发送重画事件给MyWidget窗体类。
**MyWidget的paintEvent事件响应会被自动调用，用以响应MyThread发送过来的重画事件。**
void MyWidget::paintEvent(QPaintEvent*)
{
}
####  2.、自定义事件的提交 
` Qt 系统还提供了一个 QCustomEvent 类，用于用户自定义事件，这些自定义事件可以利用 QThread::postEvent() 或者QApplication::postEvent() 被发给各种控件或其他 QObject 实例 `，
而 QWidget 类的子类可以通过 QWidget::customEvent() 事件处理函数方便地接收到这些自定义的事件。需要注意的是：QCustomEvent 对象在创建时都带有一个类型标识 id 以定义事件类型，为了避免与 Qt 系统定义的事件类型冲突，该 id 值应该大于枚举类型 QEvent::Type 中给出的 "User" 值。
如下所示：UserEvent类是用户自定义的事件类，其事件标识为346798，显然不会与系统定义的事件类型冲突。
```

　　class UserEvent : public QCustomEvent　 //用户自定义的事件类
　　{
　　public:
　　UserEvent(QString s) : QCustomEvent(346798), sz(s) { ; }
　　QString str() const { return sz; }
　　private:
　　QString sz;　　
　　};

```
UserThread类是由QThread类继承而来的子类，在该类中除了定义有关的变量和线程控制函数外，最主要的是定义线程的启动函数UserThread::run()，在该函数中创建了一个用户自定义事件UserEvent，**并利用QThread类的postEvent函数提交该事件给相应的接收对象。**
```

　　class UserThread : public QThread　　　//用户定义的线程类
　　{
　　public:
　　UserThread(QObject *r, QMutex *m, QWaitCondition *c);
　　QObject *receiver;
　　}
　　void UserThread::run()　　 //线程类启动函数，在该函数中创建了一个用户自定义事件
　　{UserEvent *re = new UserEvent(resultstring);
　　　QThread::postEvent(receiver, re);
　　}

```
UserWidget类是用户定义的用于接收自定义事件的QWidget类的子类，该类利用slotGo()函数创建了一个新的线程recv（UserThread类），当收到相应的自定义事件（即id为346798）时，利用customEvent函数对事件进行处理。
```

　　void UserWidget::slotGo()　　//用户定义控件的成员函数
　　{mutex.lock();　
　　if (! recv)
　　recv = new UserThread(this, &mutex, &condition);
　　recv->start();
　　mutex.unlock();
　　}
　　void UserWidget::customEvent(QCustomEvent *e)　 //用户自定义事件处理函数
　　{if (e->type()==346798) 
　　{
　　UserEvent *re = (UserEvent *) e;
　　　　　　newstring = re->str();
　　　　}
　　}

```
在这个例子中，UserWidget对象中创建了新的线程UserThread，用户可以利用这个线程实现一些周期性的处理（如接收底层发来的消息等），一旦满足特定条件就提交一个用户自定义的事件，当UserWidget对象收到该事件时，可以按需求做出相应的处理，而一般情况下，UserWidget对象可以正常地执行某些例行处理，而完全不受底层消息的影响。

### 手动使用QEventLoop 
Qt 是事件驱动的，所以当你用Qt的时候，几乎时时刻刻和 QEventLoop 打交道、，只是你可能没有意识到：
QCoreApplicaton::exec()
QApplication::exec()
QDialog::exec()
QThread::exec()
QDrag::exec()
QMenu::exec()
...
` 在任何时候，你都可以创建一个QEventLoop的对象，然后调用它的exec() 来开始一个局部的事件循环。 `
开启一个局部的事件循环，让其执行100ms后自己退出，似乎很不错。写来看看：
QEventLoop eventloop;
QTimer::singleShot(100, &eventloop, SLOT(quit()));
eventloop.exec();
创建事件循环
启动定时器，让其100ms后触发事件循环的quit()槽
启动事件循环


##  线程同步 
` QMutex, QReadWriteLock, QSemaphore, QWaitCondition 提供了线程同步的手段。 `使用线程的主要想法是希望它们可以尽可能并发执行，而一些关键点上线程之间需要停止或等待。例如，假如两个线程试图同时访问同一个全局变量，结果可能不如所愿。
**QMutex 提供相互排斥的锁，或互斥量。在一个时刻至多一个线程拥有mutex,**假如一个线程试图访问已经被锁定的mutex,那么它将休眠，直到拥有mutex的线程对此mutex解锁。**Mutexes常用来保护共享数据访问。**
**QReadWriterLock 与QMutex相似，除了它对"read","write"访问进行区别对待。它使得多个读者可以共时访问数据。使用QReadWriteLock而不是QMutex，可以使得多线程程序更具有并发性。**
```

QReadWriteLock lock;
void ReaderThread::run()
{
    // ...
    lock.lockForRead();
     read_file();
     lock.unlock();
     //...
}
void WriterThread::run()
{
   // ...
    lock.lockForWrite();
     write_file();
     lock.unlock();
    // ...
}

```
**QSemaphore 是QMutex的一般化，它可以保护一定数量的相同资源，**与此相对，一个mutex只保护一个资源。下面例子中，使用QSemaphore来控制对环状缓冲的访问，此缓冲区被生产者线程和消费者线程共享。生产者不断向缓冲写入数据直到缓冲末端，再从头开始。消费者从缓冲不断读取数据。信号量比互斥量有更好的并发性，假如我们用互斥量来控制对缓冲的访问，那么生产者，消费者不能同时访问缓冲。然而，我们知道在同一时刻，不同线程访问缓冲的不同部分并没有什么危害。
```

const int DataSize = 100000;
const int BufferSize = 8192;
char buffer[BufferSize];
QSemaphore freeBytes(BufferSize);
QSemaphore usedBytes;
class Producer : public QThread
{
public:
     void run();
};
void Producer::run()
{
     qsrand(QTime(0,0,0).secsTo(QTime::currentTime()));
     for (int i = 0; i < DataSize; ++i) {
         freeBytes.acquire();
         buffer[i % BufferSize] = "ACGT"[(int)qrand() % 4];
         usedBytes.release();
     }
}
class Consumer : public QThread
{
public:
     void run();
};
void Consumer::run()
{
     for (int i = 0; i < DataSize; ++i) {
         usedBytes.acquire();
         fprintf(stderr, "%c", buffer[i %BufferSize]);
         freeBytes.release();
     }
     fprintf(stderr, "\n");
}
int main(int argc, char *argv[])
{
     QCoreApplication app(argc, argv);
     Producer producer;
     Consumer consumer;
     producer.start();
     consumer.start();
     producer.wait();
     consumer.wait();
     return 0;
}

```
**QWaitCondition允许线程在某些情况发生时唤醒另外的线程。一个或多个线程可以阻塞等待一QWaitCondition,用wakeOne()或wakeAll()设置一个条件。wakeOne()随机唤醒一个，wakeAll()唤醒所有。**
下面的例子中，生产者首先必须检查缓冲是否已满(numUsedBytes==BufferSize)，如果是，线程停下来等待bufferNotFull条件。如果不是，在缓冲中生产数据，增加numUsedBytes,激活条件bufferNotEmpty。使用mutex来保护对numUsedBytes的访问。另外，QWaitCondition::wait()接收一个mutex作为参数，这个mutex应该被调用线程初始化为锁定状态。在线程进入休眠状态之前，mutex会被解锁。而当线程被唤醒时，mutex会处于锁定状态,而且，从锁定状态到等待状态的转换是原子操作，这阻止了竞争条件的产生。当程序开始运行时，只有生产者可以工作。消费者被阻塞等待bufferNotEmpty条件，一旦生产者在缓冲中放入一个字节，bufferNotEmpty条件被激发，消费者线程于是被唤醒。
```

const int DataSize = 100000;
const int BufferSize = 8192;
char buffer[BufferSize];
QWaitCondition bufferNotEmpty;
QWaitCondition bufferNotFull;
QMutex mutex;
int numUsedBytes = 0;
class Producer : public QThread
{
public:
     void run();
};
void Producer::run()
{
     qsrand(QTime(0,0,0).secsTo(QTime::currentTime()));
     for (int i = 0; i < DataSize; ++i) {
         mutex.lock();
         if (numUsedBytes == BufferSize)
            bufferNotFull.wait(&mutex);
         mutex.unlock();
         buffer[i % BufferSize] = "ACGT"[(int)qrand() % 4];
         mutex.lock();
         ++numUsedBytes;
         bufferNotEmpty.wakeAll();
         mutex.unlock();
     }
}
class Consumer : public QThread
{
public:
     void run();
};
void Consumer::run()
{
     for (int i = 0; i < DataSize; ++i) {
         mutex.lock();
         if (numUsedBytes == 0)
            bufferNotEmpty.wait(&mutex);
         mutex.unlock();
         fprintf(stderr, "%c", buffer[i %BufferSize]);
         mutex.lock();
         --numUsedBytes;
         bufferNotFull.wakeAll();
         mutex.unlock();
     }
     fprintf(stderr, "\n");
}
int main(int argc, char *argv[])
{
     QCoreApplication app(argc, argv);
     Producer producer;
     Consumer consumer;
     producer.start();
     consumer.start();
     producer.wait();
     consumer.wait();
     return 0;
}

```
###  可重入与线程安全 
在Qt文档中，术语“可重入”与“线程安全”被用来说明一个函数如何用于多线程程序。假如一个类的任何函数在此类的多个不同的实例上，可以被多个线程同时调用，那么这个类被称为是“可重入”的。假如不同的线程作用在同一个实例上仍可以正常工作，那么称之为“线程安全”的。
大多数c++类天生就是可重入的，因为它们典型地仅仅引用成员数据。任何线程可以在类的一个实例上调用这样的成员函数，只要没有别的线程在同一个实例上调用这个成员函数。举例来讲，下面的Counter 类是可重入的：
```

class Counter
{
public:
     Counter() {n=0;}
      void increment() {++n;}
      void decrement() {--n;}
      int value() const {return n;}
private:
      int n;
};

```
这个类不是线程安全的，因为假如多个线程都试图修改数据成员 n,结果未定义。这是因为c++中的++和- -操作符不是原子操作。实际上，它们会被扩展为三个机器指令：
1，把变量值装入寄存器
2，增加或减少寄存器中的值
3，把寄存器中的值写回内存
假如线程A与B同时装载变量的旧值，在寄存器中增值，回写。他们写操作重叠了，导致变量值仅增加了一次。很明显，访问应该串行化：A执行123步骤时不应被打断。使这个类成为线程安全的最简单方法是使用QMutex来保护数据成员：
```

class Counter
{
public:
     Counter() { n = 0; }
     void increment() {QMutexLocker locker(&mutex); ++n; }
     void decrement() { QMutexLockerlocker(&mutex); --n; }
     int value() const { QMutexLockerlocker(&mutex); return n; }
private:
     mutable QMutex mutex;
     int n;
};

```

QMutexLocker类在构造函数中自动对mutex进行加锁，在析构函数中进行解锁。随便一提的是，mutex使用了mutable关键字来修饰，因为我们在value()函数中对mutex进行加锁与解锁操作，而value()是一个const函数。
大多数Qt类是可重入，非线程安全的。有一些类与函数是线程安全的，它们主要是线程相关的类，如QMutex,QCoreApplication::postEvent()。


可参考：http://blog.csdn.net/jmy5945hh/article/details/34796359
http://www.cnblogs.com/dada-911911/p/3711456.html
http://blog.csdn.net/slj_win/article/details/6856397
http://blog.csdn.net/slj_win/article/details/6859620