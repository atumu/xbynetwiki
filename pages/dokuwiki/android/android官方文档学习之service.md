title: android官方文档学习之service 

#  android官方文档学习之service 
快速预览
  * 用户即使切换到另外的应用，服务(service)也会在后台运行。
  * 服务允许和多个组件绑定，这样就可以允许多进程和它交互。
  * **默认服务被应用的主线程劫持。**
本质上一个服务有两种类型:
  * 直接启动的服务
应用组件(例如一个Activity)调用` startService() `方法就可以启动一个服务。一旦启动,服务就在后台无限运行，即使启动它的组件被销毁。通常，服务启动一个单操作并且不返回结果给调用者。例如，它会通过网络下载、上传文件，当一个操作结束，该服务应该自动结束。` onStartCommand() `
  * 绑定的服务
应用组件调用` bindService() `绑定服务。绑定的服务提供一个客户端服务器(client)接口允许组件与之交互,发送请求,获得结果，甚至多进程交互执行这些操作。服务和另一个与之绑定的组件运行时间一样长。多个组件只能和一个服务绑定一次，但所有组件取消绑定之后，服务就会销毁。尽管本文档分开讨论两类服务(Started 和Bound)，但服务可以同时用两种方式工作-可以启动之后无限期运行而且允许绑定。这只取决你有木有都实现这两类服务的回调接口:启动服务` onBind() `绑定服务。

` 注意:服务运行于主线程中-即服务不会自己创建另一个线程(除非你指定)。 `这意味着如果服务执行任何cpu耗时操作或异步操作 (像MP3播放或联网)，你应该创建一个新线程执行这类操作，这样，就可以减少死机的风险，主线程就可以专门负责和你的 activity交互。

使用线程还是服务? 即使用户没和应用交互，服务仍然会可以在后台运行。因此，你应该在需要时才使用它。 如果你不想在主线程执行一个任务，而是仅当用户和应用交互时执行该任务，你应该创建新线程而不是服务。
你可以有选择的重载这几个回调函数:
` onCreate() `服务第一次创建，系统都会调用该方法，目的是一次性执行该过程(在调用onStartCommand()或onBind()方法前)。如果服务已
经运行，方法不会被调用。
` onStartCommand() `:当一个组件(例如activity)调用` startService() `方法，系统就会调用该方法(onStartCommand())启动一个服务。一旦此方法被
调用，服务立即启动，在后台一直执行。如果你实现该方法，该服务做的事情结束后，必须调用` stopSelf()或者stopService() ` ，停止该服务。
` onBind() `:当一个组件调用` bindService() `想绑定服务,系统就会调用本方法(onBind())。在你的应用中，你必须为客户端提供和该服务交
互的接口，**该接口返回一个IBinder对象**。该方法必须实现。如果不允许绑定这个服务，可以返回null即可.
onDestroy().
如果一个服务已启动，你 应该为该服务设计下重启时的相关处理。**如果服务被关闭，当有可用资源时才会重启**(尽管这依然依赖onStartComand()函数的 返回值,稍后讨论),关于系统可能销毁服务的更多信息，请参考进程和线程文档资料。
##  在manifest中声明一个服务 
<service android:name=".ExampleService" />
此外,**仅当android:exported="false"时你才能确保服务对应用来说是私有的**.即使你的服务提供intent过滤器，服务仍然私有.
##  创建一个启动的服务 
通常,有两种可用于继承的类来创建service:
  * Service:
这是所有服务类的基类,继承该类,对于在服务中创建新线程很重要.` 因为默认服务使用应用的主线程 `，可能会降低程序的性能.
  * IntentService:
这是一个Service的子类,` 该子类使用线程处理所有启动请求,一次一个 `.这是不使用服务处理多任务请求的最佳选择.你需要做的只是实现
` onHandleIntent() `方法即可.可以为每个启动请求接收到intent,**放到后台工作即可.**
下面几段描述怎么用上面两个类实现服务.
###  继承IntentService类 

**因为大多数服务不必处理同时发生的多个请求.(多线程方案可能会很危险),所以最好用IntentService实现该服务.**
` IntentService `类可以做这些事情:
  * 从应用的主线程当中**创建一个默认的线程执行所有的intents发送给` onStartCommand() `方法,该线程从主线程分离.**
  *** 创建工作队列**,每次传递一个intent 给` onHandleIntent() `方法实现,**所以不必担心多线程**.
  * **所有的请求被处理后服务停止，所以你永远都不必调用stopSelf()函数.**
  * **默认实现onBind()方法返回null.**
  * **默认实现onStartCommand()方法是发送一个intent给工作队列，然后发送给onHandleIntent()方法的实现。**
所有这些都基于` ：实现onHandleIntent()方法 `，来处理客户端的请求的工作。(因此，你还需要为服务提供一个小构造器).
这里给出IntentService类的一个实现：
```

public class HelloIntentService extends IntentService {
 
  /** 
   * A constructor is required, and must call the super IntentService(String)
   * constructor with a name for the worker thread.
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }
 
  /**
   * The IntentService calls this method from the default worker thread with
   * the intent that started the service. When this method returns, IntentService
   * stops the service, as appropriate.
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
      // For our sample, we just sleep for 5 seconds.
      long endTime = System.currentTimeMillis() + 5*1000;
      while (System.currentTimeMillis() < endTime) {
          synchronized (this) {
              try {
                  wait(endTime - System.currentTimeMillis());
              } catch (Exception e) {
              }
          }
      }
  }
}


```
如果你打算也重载其他回调方法，例如onCreate(), onStartCommand(),或者onDestroy(),**确保调用父类对应的方法，****如此，IntentService实例化的对象才能正常工作。**
例如,onStartCommand()必须返回默认实现的方法(意图就是通过它传递给onHandleIntent()的).
```

@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
    return super.onStartCommand(intent,flags,startId);
}

```
除了onHandleIntent(),唯一不需要调用父类对应方法的方法就是onBind()(该方法是设置服务允许绑定时才实现).
###  继承Service类 

正如之前所述,使用IntentService类,可以简化启动服务的代码.**但是，请求启动服务的是多线程同时请求(代替用一个工作队列处理这种情况),那么你就应该继承Service类处理这种情况**.
这样，对于每个启动服务的请求,就开启一个线程处理该请求。
```

 
public class HelloService extends Service {
  private Looper mServiceLooper;
  private ServiceHandler mServiceHandler;
 
  // 处理程序从线程中收到的消息
  private final class ServiceHandler extends Handler {
      public ServiceHandler(Looper looper) {
          super(looper);
      }
      @Override
      public void handleMessage(Message msg) {
	  // 通常我们在这里做一些事情，例如下载文件。
	  // 这里，我们只让程序睡眠5s
          long endTime = System.currentTimeMillis() + 5*1000;
          while (System.currentTimeMillis() < endTime) {
              synchronized (this) {
                  try {
                      wait(endTime - System.currentTimeMillis());
                  } catch (Exception e) {
                  }
              }
          }
	  // 使用startId停止服务,所以我们在处理其他任务的过程中不会停止该服务.
          stopSelf(msg.arg1);
      }
  }
 
  @Override
  public void onCreate() {
    // 启动线程开启服务,注意默认服务通常运行在进程的主线程中，所以我们创建了另一个线程。
    // 而且我们也不想阻塞程序。另外我们把该线程放在后台运行，这样cpu耗时操作就不会破坏应用界面.
    HandlerThread thread = new HandlerThread("ServiceStartArguments",
            Process.THREAD_PRIORITY_BACKGROUND);
    thread.start();
 
    // 获得线程的Looper,用于我们的Handler
    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
  }
 
  @Override
  public int onStartCommand(Intent intent, int flags, int startId) {
      Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
 
      // 对于每个启动请求,发送消息用于启动一个任务,传递start ID,这样当完成此任务时就知道谁发出该请求.
      Message msg = mServiceHandler.obtainMessage();
      msg.arg1 = startId;
      mServiceHandler.sendMessage(msg);
 
      // 如果服务已被杀死,执行return后，线程重启.
      return START_STICKY;
  }
 
  @Override
  public IBinder onBind(Intent intent) {
      // 如果我们不提供服务绑定，返回null
      return null;
  }
 
  @Override
  public void onDestroy() {
    Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show(); 
  }
}
 

```
但是，**在onStartCommand()方法里因为是自己处理每个调用,所以你可以同时处理多个请求**。例子中虽然没演示,但如果需要，你完全可以为每个请求创建一个线程，按正确的方式处理这些请求.(比起上面的IntentService,Service代替了需要等待上次请求处理完毕的方式。)
注意**onStartCommand()方法必须返回一个整型变量.**这个变量描述可能已被系统停止的服务，如果被停止，系统会接着继续服务(正如下面讨论，IntentService的实现中默认自动帮你做了个处理，尽管可以修改它),返回值必须是如下几个常量之一:
` START_NOT_STICKY `
如果系统在onStartCommand()返回后停止服务,**系统不会重新创建服务,除非有等待的意图需要处理**.如果想避免运行不需要的服务，或者让应用可以轻松的启动未完成的任务，这是一个安全的选择。
` START_STICKY `
如果系统在onStartCommand()返回后停止服务,系统重新创建服务并且调用onStartCommand()函数，但是不要传送最后一个意图。相反，**系统用一个空意图调用onStartCommand()**,除非还有想启动服务的意图,这种情况下，这些意图会被传递.这**很适合多媒体播放的情况**(或类似服务)，这种情况下服务不执行命令,但是会无限运行下去，等待处理任务。
` START_REDELIVER_INTENT `
如果系统在onStartCommand()返回后停止服务,**系统使用用最后传递给service的意图,重新创建服务**并且调用onStartCommand()方法， 任何未被处理的意图会接着循环处理。所以用服务**处理像下载之类**的可能需要立即重启的任务，非常合适。
关于这些返回值的更多信息，请参考文档for each constant.

##  启动服务 
```

Intent intent = new Intent(this, HelloService.class);
startService(intent);

```
<note tip>并发请求启动服务导致并发响应调用onStartCommand()函数,但是，唯一停止服务的方法就是调用stopSelf()或者stopService()方法。</note>
<note important>如果系统不提供绑定，通过传递意图给startService()方法是应用组件和服务唯一的沟通方式。但是如果你想服务发送一个返回结果,可以让客户端使用广播(调用getBroadcast()方法获取)创建一个` PendingIntent `启动一个服务，然后把该意图传递给要启动的服务，服务就可以使用此广播传递结果了。</note>
##  创建一个已绑定的服务 
一个已绑定的服务是指应用组件调用` bindService() `绑定该服务，目的是**创建和这个服务的长连接**(通常不允许组件调用 startService()启动该服务).
**当你想和应用组件启动的服务交互,或对其他应用提供部分功能，你应该创建一个绑定服务。** 要创建绑定服务，必须` 实现onBind()回调函数，返回一个IBinder对象 `，定义和该服务交互的接口.其他组件就可以调用包括 bindService()服务接口.该服务就仅对和它绑定的组件起作用，所以当没有组件和服务绑定，系统会自动销毁该服务.
##  发送用户通知 
一旦运行，服务可以用Toast Notifications 或者Status Bar Notifications通知用户一些事情.
##  前台运行一个服务 
前台服务是一种考虑到系统内存不足，但是用户已经意识到而且不想关闭的服务。前台服务必须用状态条做出提示，表示该服 务在持续进行中,意思是这种提示除非服务停止或者从所有的前台服务中移除才能消失.** 例如,用服务操作音乐播放器播放音乐,应该在前台播放，因为用户明确指出了具体操作(就是播放音乐)。**状态条可以包含当前 播放的音乐，并且允许用户启动一个activity和音乐播放器交互.
要请求服务在前台运行，调用` startForeground() `方法.这个方法带两个参数:一个整型变量,表示提示信息的唯一标识符,另一个` Notification `对象,表示状态栏,示例代码:
```

Notification notification = new Notification(R.drawable.icon, getText(R.string.ticker_text),
        System.currentTimeMillis());
Intent notificationIntent = new Intent(this, ExampleActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, 0);
notification.setLatestEventInfo(this, getText(R.string.notification_title),
        getText(R.string.notification_message), pendingIntent);
startForeground(ONGOING_NOTIFICATION, notification);

```
要从前台移除服务，请调用` stopForeground() `.这个方法需要一个布尔参数,表示是否清除状态栏的信息.**调用它，本身不会停止服务.但是,服务仍然在后台运行，**你突然停止服务,状态烂的提示就会移除。
##  管理服务的生命周期 
![](/data/dokuwiki/android/pasted/20150607-072757.png)
服务的生命周期--从他被创建到销毁-会遵循如下2种方式之一:
  * 启动的服务 当另一个组件调用` startService() `，服务就被创建,然后一直运行下去，直到服务自己调用` stopSelf() `方法停止该服务.其他组件也可以通过调用` stopService() `方法停止一个服务.系统会销毁该服务。
  * 绑定的服务 当另一个组件(客户端)调用` bindService() `，服务就被创建,客户端然后通过` IBinder `接口和服务交互.客户端可以调用` unbindService() `关闭和服务的连接.多客户端可以绑定到相同的服务，所有的客户端都解除绑定之后，系统销毁该服务.(服务不必像上面那样自己销毁).

##  绑定服务 
一个绑定的服务允许组件（如：活动）来绑定一个服务，传送请求，接收响应，甚至执行进程间的通信（IPC）。绑定服务通常只生存在其服务于另一个程序组件时，并且不会无限期的在后台运行。
为服务提供绑定，**你必须实现onBind()回调方法**。这个方法**返回一个IBinder对象**，它定义了程序接口，用户可以用其与服务交互。一个客户可以通过调用bindService()来绑定到一个服务。当这么做时，**必须提供一个ServiceConnection的实现，它监视着与服务的连接。**bindService()方法立刻返回了一个空值。但是当Android系统在用户和服务器之间创建一个连接时，**它将会在ServiceConnection上调用onServiceConnected()方法来传递IBinder对象**，使用户可以与服务通信。
多个用户可以同时连接到一个服务。但是，系统只以第一个用户绑定时调用onBind()方法来获得IBinder对象。然后，系统将同一个IBinder对象传递给后续绑定的客户，并不会重新调用onBind()方法。
当最后一个用户从服务上解绑时，系统摧毁这个服务（除非这个服务由startService()启动）。
当你实现绑定服务时，最重要的就是定义onBind()回调方法返回的接口。你可以使用有一些不同的方法来定义服务的IBinder接口，接下来的章节将讨论每一项技术。
当创建一个提供了绑定的服务时，你必须提供一个` IBinder `来提供程序接口，用户可以用这个接口与服务交互。有三种方法可以用来定义这个接口：
  * 扩展Binder类-Extending the Binder class
  * 使用一个消息传递器-Using a Messenger
  * 使用AIDL

在客户端生命周期的创建和摧毁时刻，你应该成对使用绑定和解绑。例如：
  * 当你的活动可见时，如果你需要做的只是与服务交互，你应该在onStart()中绑定，在onStop()中解绑。
  * **如果你希望即使活动在后台停止时也接收响应，那么你可以在onCreate()方法中绑定，并在onDestroy()方法中解绑**。需要注意的是，这个实现使得你的活动需要在所有服务运行的时候占用该服务（即使在后台运行时）。所以，如果服务存在于另一个进程中，那么你增加了进程的比重，因此，系统也将更有可能性将其杀死。

注解：**通常情况下，你不应该在活动的onResume()和onPause()方法中绑定与解绑，**因为这些回调方法发生在生命周期转换的时候，你应该把此期间的发生的进程减小到最少。
<note tip>注解：只有活动（activities），服务（services），和内容提供者（content providers）可以绑定到一个服务——你不能从一个广播接收器（broadcast receiver）绑定到一个服务。</note>
更多的关于怎么样绑定服务的范例代码，请见ApiDemos中的RemoteService.java类。
###  扩展Binder类 

如果你的服务只被本地应用程序所使用，并且不需要在多个进程间工作，那么你可以实现你自己的Binder类，让你的客户可以直接接入方法中的公共方法。
例如，这个方法对于一个音乐应用程序将会非常有用，它需要绑定一个活动到它自己的服务用来以后台播放音乐。
**以下为如何设定这个binder类：**
1.在你的服务中，创建一个Binder类的实例，实现以下功能之一：
  * 包含客户可以调用的公共方法
  * 返回当前Service的实例，其包含了用户可以访问的公共方法
  * 或返回这个服务包含的另一个类，并含有客户可以访问的公共方法

2.从onBind()回调函数返回这个Binder的实例。
3.在客户端，从onServiceConnected()回调方法接收这个Binder,并用提供的方法来调用绑定服务。
<note tip>注解：服务和客户端必须在同一个应用程序中的原因是客户端可以计算返回的对象并恰当的调用其APIs。服务和客户端也必须在同一个线程的原因是这种技术不能执行线程间操作。</note>
```

public class LocalService extends Service {
    // Binder given to clients
    private final IBinder mBinder = new LocalBinder();
    // Random number generator
    private final Random mGenerator = new Random();
 
    /**
     * Class used for the client Binder.  Because we know this service always
     * runs in the same process as its clients, we don't need to deal with IPC.
     */
    public class LocalBinder extends Binder {
        LocalService getService() {
            // Return this instance of LocalService so clients can call public methods
            return LocalService.this;
        }
    }
 
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
 
    /** method for clients */
    public int getRandomNumber() {
      return mGenerator.nextInt(100);
    }
}

```
LocalBinder为客户端提供了getService()方法来取得当前的LocalService实例。这个允许客户端调用服务中的公共方法。例如，客户端可以从服务中调用getRandomNumber()。
```

public class BindingActivity extends Activity {
    LocalService mService;
    boolean mBound = false;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }
 
    @Override
    protected void onStart() {
        super.onStart();
        // Bind to LocalService
        Intent intent = new Intent(this, LocalService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }
 
    @Override
    protected void onStop() {
        super.onStop();
        // Unbind from the service
        if (mBound) {
            unbindService(mConnection);
            mBound = false;
        }
    }
 
    /** Called when a button is clicked (the button in the layout file attaches to
      * this method with the android:onClick attribute) */
    public void onButtonClick(View v) {
        if (mBound) {
            // Call a method from the LocalService.
            // However, if this call were something that might hang, then this request should
            // occur in a separate thread to avoid slowing down the activity performance.
            int num = mService.getRandomNumber();
            Toast.makeText(this, "number: " + num, Toast.LENGTH_SHORT).show();
        }
    }
 
    /** Defines callbacks for service binding, passed to bindService() */
    private ServiceConnection mConnection = new ServiceConnection() {
 
        @Override
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // We've bound to LocalService, cast the IBinder and get LocalService instance
            LocalBinder binder = (LocalBinder) service;
            mService = binder.getService();
            mBound = true;
        }
 
        @Override
        public void onServiceDisconnected(ComponentName arg0) {
            mBound = false;
        }
    };
}

```
bindService()的第一个参数，是一个Intent，其明确的指出了被绑定服务的名字（intent被认为可以暗示性的指出）。
第二个参数是` ServiceConnection `对象。
第三个参数是一个标志位，指示了绑定的选项。通常情况下应该为` BIND_AUTO_CREATE `，为了能够在服务不存在的情况下创建一个服务。其它可选值有` BIND_DEBUG_UNBIND和BIND_NOT_FOREGROUND `，或0表示什么都没有。
以上范例中展示了客户端怎样通过使用一个` ServiceConnection `的实现和` onServiceConnected() `回调方法绑定到一个服务。后续章节中提供了更多的关于绑定到服务流程的信息。
实例代码，请见ApiDemos中LocalService.java和LocalServiceActivities.java类。
###  使用一个消息传递器-Using a Messenger 
如果你需要你的服务能够与远程进程通信，那么你可以使用一个Messenger为你的服务提供接口。**这个方法允许你执行进程间通信（IPC）而不需要使用AIDL。**
以下为怎么样使用Messenger的总结：

  * **服务实现了一个Handler**，用来**接收**每一次调用从客户端返回的回调方法。
  * Handler被用来创建一个` Messenger `对象（其为Handler的一个引用）。
  * **Messenger创建一个IBinder**，服务从onBind()方法将其**返回给客户端**。
  * **客户端使用这个IBinder来实例化这个Messenger**（**其引用到服务的Handler**），**客户端可以用来向服务发送Message对象。**
  * **服务通过它的Handler接收每一个Message**——更确切的说，是在handleMessage()方法中接收。

通过这种方法，在服务端没有客户端能调用的“方法”。而是，客户传递“消息”（Message对象），同时服务在其Handler中接收。

以下为服务使用一个Messenger接口的简单范例：
```

public class MessengerService extends Service {
    /** Command to the service to display a message */
    static final int MSG_SAY_HELLO = 1;
 
    /**
     * Handler of incoming messages from clients.
     */
    class IncomingHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_SAY_HELLO:
                    Toast.makeText(getApplicationContext(), "hello!", Toast.LENGTH_SHORT).show();
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }
 
    /**
     * Target we publish for clients to send messages to IncomingHandler.
     */
    final Messenger mMessenger = new Messenger(new IncomingHandler());
 
    /**
     * When binding to the service, we return an interface to our messenger
     * for sending messages to the service.
     */
    @Override
    public IBinder onBind(Intent intent) {
        Toast.makeText(getApplicationContext(), "binding", Toast.LENGTH_SHORT).show();
        return mMessenger.getBinder();
    }
}

```
客户端所需做的只是基于服务返回的IBinder创建一个Messenger并使用send()方法发送一条消息。例如，以下为一个简单的活动范例，其绑定到了服务，并向服务传递了MSG_SAY_HELLO消息：
```

public class ActivityMessenger extends Activity {
    /** Messenger for communicating with the service. */
    Messenger mService = null;
 
    /** Flag indicating whether we have called bind on the service. */
    boolean mBound;
 
    /**
     * Class for interacting with the main interface of the service.
     */
    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className, IBinder service) {
            // This is called when the connection with the service has been
            // established, giving us the object we can use to
            // interact with the service.  We are communicating with the
            // service using a Messenger, so here we get a client-side
            // representation of that from the raw IBinder object.
            mService = new Messenger(service);
            mBound = true;
        }
 
        public void onServiceDisconnected(ComponentName className) {
            // This is called when the connection with the service has been
            // unexpectedly disconnected -- that is, its process crashed.
            mService = null;
            mBound = false;
        }
    };
 
    public void sayHello(View v) {
        if (!mBound) return;
        // Create and send a message to the service, using a supported 'what' value
        Message msg = Message.obtain(null, MessengerService.MSG_SAY_HELLO, 0, 0);
        try {
            mService.send(msg);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }
 
    @Override
    protected void onStart() {
        super.onStart();
        // Bind to the service
        bindService(new Intent(this, MessengerService.class), mConnection,
            Context.BIND_AUTO_CREATE);
    }
 
    @Override
    protected void onStop() {
        super.onStop();
        // Unbind from the service
        if (mBound) {
            unbindService(mConnection);
            mBound = false;
        }
    }
}

```
###  使用AIDL 
略。。。具体请看官方文档http://developer.android.com/guide/components/aidl.html