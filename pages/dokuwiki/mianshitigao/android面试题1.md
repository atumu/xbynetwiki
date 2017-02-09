title: android面试题1 

#  android面试题1 

1.android dvm 的进程和Linux的进程，应用程序的进程是否为同一个概
念：
答：dvm是dalivk虚拟机。每一个android应用程序都在自己的进程中运行，都
拥有一个dalivk虚拟机实例。而每一个dvm都是在linux的一个进程。所以说可
以认为是同一个概念。
2.android的动画有哪几种？他们的特点和区别是什么？
答：两种，一种是tween动画，一种是frame动画。tween动画，这种实现方
式可以使视图组件移动，放大或缩小以及产生透明度的变化。frame动画，传
统的动画方法，通过顺序的播放排列好的图片来实现，类似电影。
3.handler进制的原理：
答：android提供了handler和looper来满足线程间的通信。Handler先进先出
原则。looper用来管理特定线程内对象之间的消息交换（message
Exchange）.
1)looper:一个线程可以产生一个looper对象，由它来管理此线程里
的message queue(消息队列)
2)handler:你可以构造一个handler对象来与looper沟通，以便push新消息
ANDROID
ANDROID面试题精选，自己收
藏下
2014年4月17日 XBY1993 发表回复

JAVA_MAN

Android面试题精选，自己收藏下 | JAVA_MAN
http://localhost/wordpress/?p=342[2015/4/8 23:35:26]
到messagequeue里；或者接收looper（从messagequeue里取出）所送来的
消息。
3)messagequeue:用来存放线程放入的消息。
4)线程：UI thread 通常就是main thread,而android启动程序时会为它建立
一个message queue.
4.android view的刷新：
答：Android中对View的更新有很多种方式，使用时要区分不同的应用场合。
我感觉最要紧的是分清：多线程和双缓冲的使用情况。
1).不使用多线程和双缓冲
这种情况最简单了，一般只是希望在View发生改变时对UI进行重绘。你只
需在Activity中显式地调用View对象中的invalidate()方法即可。系统会自动调
用 View的onDraw()方法。
2).使用多线程和不使用双缓冲
这种情况需要开启新的线程，新开的线程就不好访问View对象了。强行访
问的话会
报：android.view.ViewRoot$CalledFromWrongThreadException：Only the
originalthread that created a view hierarchy can touch its views.
这时候你需要创建一个继承了android.os.Handler的子类，并重
写handleMessage(Messagemsg)方法。android.os.Handler是能发送和处理
消息的，你需要在Activity中发出更新UI的消息，然后再你的Handler（可以使
用匿名内部类）中处理消息（因为匿名内部类可以访问父类变量，你可以直
接调用View对象中的invalidate()方法）。也就是说：在新线程创建并发送一
个Message，然后再主线程中捕获、处理该消息。
3).使用多线程和双缓冲
Android中SurfaceView是View的子类，她同时也实现了双缓冲。你可以定
义一个她的子类并实现SurfaceHolder.Callback接口。由于实
现SurfaceHolder.Callback接口，新线程就不需要android.os.Handler帮忙
了。SurfaceHolder中lockCanvas()方法可以锁定画布，绘制玩新的图像后调
用unlockCanvasAndPost(canvas)解锁（显示），还是比较方便得。
Android面试题精选，自己收藏下 | JAVA_MAN
http://localhost/wordpress/?p=342[2015/4/8 23:35:26]
5.说说mvc模式的原理，它在android中的运用:
答：android的官方建议应用程序的开发采用mvc模式。何谓mvc？
mvc是model,view,controller的缩写，mvc包含三个部分：
l模型（model）对象：是应用程序的主体部分，所有的业务逻辑都应该写在
该层。
2视图（view）对象：是应用程序中负责生成用户界面的部分。也是在整
个mvc架构中用户唯一可以看到的一层，接收用户的输入，显示处理结果。
3控制器（control）对象：是根据用户的输入，控制用户界面数据显示及更
新model对象状态的部分，控制器更重要的一种导航功能，想用用户出发的相
关事件，交给m哦得了处理。
android鼓励弱耦合和组件的重用，在android中mvc的具体体现如下：
1)视图层（view）：一般采用xml文件进行界面的描述，使用的时候可以非
常方便的引入，当然，如何你对android了解的比较的多了话，就一定可以想
到在android中也可以使用javascript+html等的方式作为view层，当然这里需
要进行java和javascript之间的通信，幸运的是，android提供了它们之间非常
方便的通信实现。
2)控制层（controller）：android的控制层的重任通常落在了众多的acitvity的
肩上，这句话也就暗含了不要在acitivity中写代码，要通过activity交
割model业务逻辑层处理， 这样做的另外一个原因是android中的acitivity的响
应时间是5s，如果耗时的操作放在这里，程序就很容易被回收掉。
3)模型层（model）：对数据库的操作、对网络等的操作都应该在model里面
处理，当然对业务计算等操作也是必须放在的该层的。
6.Activity的生命周期:
答：onCreate: 在这里创建界面，做一些数据的初始化工作
onStart: 到这一步变成用户可见不可交互的
onResume:变成和用户可交互的，(在activity 栈系统通过栈的方式管理这
些个Activity的最上面，运行完弹出栈，则回到上一个Activity
)
onPause: 到这一步是可见但不可交互的，系统会停止动画等消耗CPU 的事
情从上文的描述已经知道，应该在这里保存你的一些数据,因为这个时候你的
程序的优先级降低，有可能被系统收回。在这里保存的数据，应该在
onstop: 变得不可见，被下一个activity覆盖了
Android面试题精选，自己收藏下 | JAVA_MAN
http://localhost/wordpress/?p=342[2015/4/8 23:35:26]
onDestroy: 这是activity被干掉前最后一个被调用方法了，可能是外面类调
用finish方法或者是系统为了节省空间将它暂时性的干掉
7.让Activity变成一个窗口：
答：Activity属性设定：有时候会做个应用程序是漂浮在手机主界面的。这个
只需要在设置下Activity的主题theme,即在Manifest.xml定义Activity的地方加
一句：
android :theme="@android:style/Theme.Dialog"
如果是作半透明的效果：
android:theme="@android:style/Theme.Translucent"
8.Android中常用的五种布局:
答：LinearLayout线性布局；AbsoluteLayout绝对布局；TableLayout表格布
局；RelativeLayout相对布局；FrameLayout帧布局；
9.Android的五种数据存储方式：
答：sharedPreferences；文件；SQLite；contentProvider；网络
10.请解释下在单线程模型中Message、Handler、Message
Queue、Looper之间的关系：
答：Handler获取当前线程中的looper对象，looper用来从存
有Message的Message Queue里取出message，再由Handler进
行message的分发和处理。
11.AIDL的全称是什么?如何工作?能处理哪些类型的数据?
答：AIDL(AndroidInterface Definition Language)android接口描述语言
12.系统上安装了多种浏览器，能否指定某浏览器访问指定页面？请说明原
由：
答：通过直接发送Uri把参数带过去，或者通过manifest里的intentfilter里
的data属性。代码如
下：
Intent intent = new Intent();
Intent.setAction(“android.intent.action.View”);
Uri uriBrowsers = Uri.parse(“http://www.sina.com.cn”);
Intent.setData(uriBrowsers);
/ /包名、要打开的activity
intent.setClassName(“com.android.browser”,”com.android.browser.Browser
Activity”);
startActivity(intent);
13.什么是ANR,如何避免？
Android面试题精选，自己收藏下 | JAVA_MAN
http://localhost/wordpress/?p=342[2015/4/8 23:35:26]
答：ANR的定义：
在android上，如果你的应用程序有一段时间响应不移灵敏，系统会向用户提
示“应用程序无响应”（ANR：application Not Responding）对话框。因此，
在程序里对响应性能的设计很重要，这样，系统不会显示ANR给用户。
如何避免：
首先来研究下为什么它会在android的应用程序里发生和如何最佳构建应用程
序来避免ANR.
android应用程序通常是运行在一个单独的线程（例如：main）里，这就意
味你的应用程序所做的事情如果在主线程里占用了大长时间的话，就会引
发ANR对话框，因为你的应用程序并没有给自己机会来处理输入事件或
者Intent广播。
因此，运行在主线程里的任何访求都尽可能少做事情。特别是，activity应
该在它的关键生命周期方法（onCreate()和onResume()）里尽可能少的去作
创建操作。潜在的耗时操作，例如网络或数据库操作，或者高耗时的计算如
改变位图尺寸，应该在子线程里（或者以数据库操作为例，通过异步请求的
方式）来完成。然而，不是说你的主线程阻塞在那里等待子线程的完成—也
不是调用Thread.wait()或者Thread.sleep()。替代的方法是：主线程应该为子
线程提供一个Handler,以便完成时能够提交给主线程。以这种方式设计你的应
用程序，将能保证你的主线程保持对输入的响应性并能避免由5秒输入事件的
超时引发的ANR对话框。这种做法应该在其它显示UI的线程里效仿，因为它
们都受相同的超时影响。
IntentReceiver执行时间的特殊限制意味着它应该做：在后台里做小的、琐
碎的工作，如保存设定或注册一个Notification。和在主线程里调用的其它方
法一样，应用程序应该避免在BroadcastReceiver里做耗时的操作或计算，但
也不是在子线程里做这些任务（因为BroadcastReceiver的生命周期短），替
代的是，如果响应Intent广播需要执行一个耗时的动作的话，应用程序应该启
动一个Service。顺便提及一句，你也应该避免在Intent Receiver里启动一
个Activity，因为它会创建一个新的画面，并从当前用户正在运行的程序上抢
夺焦点。如果你的应用程序在响应Intent广播时需要向用户展示什么，你应该
使用Notification Manager来实现。
一般来说，在应用程序里，100到200ms是用户能感知阻滞的时间阈值，
下面总结了一些技巧来避免ANR,并有助于让你的应用程序看起来有响应性。
如果你的应用程序为响应用户输入正在后台工作的话，可以显示工作的进
度（ProgressBar和ProgressDialog对这种情况来说很有用）。特别是游戏，
在子