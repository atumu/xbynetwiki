title: handler_looper_message_queue理解 

#  handler_looper_message_queue理解 

#  概述 

类似于Swing，Android中的UI线程不是线程安全的，采用的是单一线程模型。不过Android提供了一种强大的消息传递机制消息队列（Message Queue）与Handler。使用消息队列的线程叫(Message Loop.)。一个叫做Looper的对象会不断循环检查消息队列上是否有新的消息。消息循环由一个线程和一个Looper组成。Looper对象管理着线程的消息队列。
主线程也是一个消息循环，因此具有一个Looper,` 主线程的所有工作都是由其Looper完成的 `，looper不断从消息队列抓取消息，然后完成消息指定的任务。
<note important>注意：对于主线程的所有工作（包括Activity的onCreate,onDestroy()等）都是由其Looper完成的这一点的理解。稍后分析。</note>

#  Handler、Looper、MessageQueue的工作原理 

Message:消息对象。有三个属性
  * what:int型消息代码；
  * obj:随消息一起发送的对象
  * target:处理消息的Handler
Looper:每个线程只能拥有一个Looper。它的loop方法负责从消息队列MessageQueue中提取消息，并把消息交送给发送该消息的Handler进行处理。主线程的Looper由系统负责初始化。自己启动的线程则需要通过调用Looper.prepare()初始化，并调用Looper.loop()开始消息循环。
MessageQueue：FIFO消息队列.用于管理Message.Looper在初始化就会创建一个与之对应的MessageQueue.
Handler:两个作用：发送消息与接收消息进行处理。**必须注意的是要想正常工作需要确保当前线程当中存在一个工作的Looper对象。否则会报错。默认情况下Handler工作在创建它的线程当中。**

#  在非UI线程使用Handler的几种方式： 

方式一：在UI线程中初始化Handler，通过参数传入其他线程的构造器。这样创建的Handler是工作在主线程当中的Handler.
方式二：在新建线程当中初始化Looper,通过调用其prepare(),loop()方法初始化Looper与消息队列然后初始化使用Handler。
方式三：使用API当中的` HandlerThread `.这是一个包含Looper并自动初始化Looper的Thread。注意覆盖HandlerThread中的onLooperPrepared()方法，在其中初始化Handler是最佳实践。


#  为什么Looper.loop()循环不会阻塞主线程？ 

无论是主线程还是其他线程，要想使用Looper,一般都是采取如下步骤:
  * Looper.prepare();
  * Looper.loop();**这是一个无线循环，我们知道主线程Looper是运行在主线程上的，为什么不会阻塞主线程呢？**
  * Looper.quite();
```

public static void loop() {  
        final Looper me = myLooper();  
        if (me == null) {  
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");  
        }  
        final MessageQueue queue = me.mQueue;  
  
        // Make sure the identity of this thread is that of the local process,  
        // and keep track of what that identity token actually is.  
        Binder.clearCallingIdentity();  
        final long ident = Binder.clearCallingIdentity();  
  
        for (;;) {  
            Message msg = queue.next(); // might block  
            if (msg == null) {  
                // No message indicates that the message queue is quitting.  
                return;  
            }  
  
            // This must be in a local variable, in case a UI event sets the logger  
            Printer logging = me.mLogging;  
            if (logging != null) {  
                logging.println(">>>>> Dispatching to " + msg.target + " " +  
                        msg.callback + ": " + msg.what);  
            }  
  
            msg.target.dispatchMessage(msg);  
  
            if (logging != null) {  
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);  
            }  

```
<WRAP center round help 60%>
很明显for(;;)这是一个无线循环。那为什么运行在主线程中不会阻塞呢？
</WRAP>

那让我们进行下一步，我在Activity的onCreate方法中添加int a=1/0;
```

@Override  
   protected void onCreate(Bundle savedInstanceState) {  
       super.onCreate(savedInstanceState);  
       int a=1/0;  

```
然后运行，我们可以通过**分析exception stack trace**来找到答案，以下为stack trace:
FATAL EXCEPTION: main
java.lang.RuntimeException: Unable to start activity 
java.lang.ArithmeticException: divide by zero
at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2059)
at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2084)
at android.app.ActivityThread.access$600(ActivityThread.java:130)
at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1195)
at android.os.Handler.dispatchMessage(Handler.java:99)
<wrap em>at android.os.Looper.loop(Looper.java:137)
at android.app.ActivityThread.main(ActivityThread.java:4745)</wrap>
看到了吧？原来主线程中执行的过程中，Activity的onCreate方法调用也要通过Looper的loop方法来执行。其他方法的测试类似通过stack trace来找到答案。
<wrap em>所以可以看出，所有发生在UI线程中的事情（如Activity的生命周期等）都需要通过loop循环进行直接或者间接地处理，这样虽然Looper.loop是无限循环，但是却不会阻塞UI线程。

一个小技巧，你可以通过Looper.myLooper==Looper.getMainLooper()来判断是否处于UI线程中</wrap>
