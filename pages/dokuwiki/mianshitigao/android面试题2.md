title: android面试题2 

#  android面试题2 

本文由安卓航班网收集www.apkway.com
1. 请描述下Activity的生命周期。
2. 如果后台的Activity由于某原因被系统回收了，如何在被系统回收之前保存当前状态？
3. 如何将一个Activity设置成窗口的样式。(Edited by Sodino)
4. 如何退出Activity？如何安全退出已调用多个Activity的Application？
5. 请介绍下Android中常用的五种布局。
6. 请介绍下Android的数据存储方式。(Edited by Sodino)
7. 请介绍下ContentProvider是如何实现数据共享的。(Edited by Sodino)
8. 如何启用Service，如何停用Service。(Edited by Sodino)
9. 注册广播有几种方式，这些方式有何优缺点？请谈谈Android引入广播机制的用意。
10. 请解释下在单线程模型中Message、Handler、Message Queue、Looper之间的关系。
11. AIDL的全称是什么？如何工作？能处理哪些类型的数据？
12. 请解释下Android程序运行时权限与文件系统权限的区别。(Edited by Sodino)
13. 系统上安装了多种浏览器，能否指定某浏览器访问指定页面？请说明原由。
14. 有一个一维整型数组int[]data保存的是一张宽为width，高为height的图片像素值信息。请写一个算法，将该图片所有的白色不透明(0xffffffff)像素点的透明度调整为50%。
15. 你如何评价Android系统？优缺点。
1. activity的生命周期。 activity主要生命周期的方法说明： onCreate(Bundle savedInstanceState)：创建activity时调用。设置在该方法中，还以Bundle的形式提供对以前储存的任何状态的访问！ onStart()：activity变为在屏幕上对用户可见时调用。 onResume()：activity开始与用户交互时调用（无论是启动还是重新启动一个活动，该方法总是被调用的）。 onPause()：activity被暂停或收回cpu和其他资源时调用，该方法用于保存活动状态的，也是保护现场，压栈吧！ onStop()：activity被停止并转为不可见阶段及后续的生命周期事件时调用。 onRestart()：重新启动activity时调用。该活动仍在栈中，而不是启动新的活动。 onDestroy()：activity被完全从系统内存中移除时调用，该方法被
2.横竖屏切换时候activity的生命周期 3.android中的动画有哪几类，它们的特点和区别是什么 4.handler机制的原理 5.说说activity，intent，service是什么关系 6.android中线程与线程，进程与进程之间如何通信 7.widget相对位置的完成在antivity的哪个生命周期阶段实现 8.说说mvc模式的原理，它在android中的运用 9.说说在android中有哪几种数据存储方式 10.android中有哪几种解析xml的类，官方推荐哪种？以及它们的原理和区别 一，listview你是怎么优化的。 二，view的刷新，之前说过 三，IPC及原理 四，Android多线程 五，Android为什么要设计4大组件，他们之间的联系，不设计行不行（主要是为了实现MVC模式，然而java中最难的模式也是这个，很少有产品能将这个模式做得很好【Technicolor的面试官问的这个】 六，service的周期，activity的周期，谈下你对Android内部应用的了解，比如他做电话，以及联系人等等应用。框架层有很多东西还是多看看，熟悉Android怎么做的，不管你做应用程开发还是应用框架层开发很有好处的。 在就是你项目经验，突出你遇到什么难点，然后是怎么解决的！尽量将每个技术点凸显出来，当然面试官有时候会为了体现你是否真正做过，他会问你，你在这个应用中做那个模块，用了多少个类之类的问题。 偶尔有的面试官会问你，你用过Android自带的单元测试了没，怎么用的？当然我面试过很多家单位，有的是做平板，手机，数字电视，有的是做出个erp之类的客户端等等，出于前面的三个，基本上都是将Android的全部改掉，如果真正要做Android的话，大家要学的还很多。 总之，一句话，什么样的面试官都有，去面试的时候要做好一切心理准备，不管是技术还是基础都得扎实。一个人的交谈能力也很重要，总之不是非常标准的普通话，最起码你说的得让别人听得懂，而且得把面试官讲得非常彻底，这样你获得offer的机会更大，谈工资也有优势~~当然曾经一家公司的面试官跟我说过，技术是不惜钱的，只要你有能力，多少钱他都请。_ 确实，来北京求职期间，牛人真的很多，而且有的面试官也非常好，给了很多忠肯的意见。并不是每个面试官都特想为难你的~最主要的还是想知道你的技术，因为他们也是吃公司饭，得为这个负责。 Basic: 1. 基本的UI控件和布局文件 2. UI配套的Adapter的使用 3. Activity, Intent,Service,broadCast Receiver他们的生命周期管理熟悉一下 4. 操作手机上的数据库SQLite应用
Advanced_1: 1. 为什么看好 Android 2. 现在在公司做哪些工作（关于 Android) 3. Android 的框架以及一些基础知识 4. Android 一些方面的领悟（如Android框架的 IoC特性，View System 的状态机机制等) Advanced_2: 1.对多线程的运用和理解，及多线程之间handle的传值。 2.对android 虚拟机的理解，包括内存管理机制垃圾回收机制。 3.framework工作方式及原理，Activity是如何生成一个view的，机制是什么。 4. android本身的一些限制，不如apk包大小限制，读取大文件 时的时间限制。 5. Linux中跨进程通信的集中方式 Android_4: 1. dvm的进程和Linux的进程, 应用程序的进程是否为同一个概念 2. sim卡的EF 文件有何作用 3. AT命令的User case的概念 4.嵌入式操作系统内存管理有哪几种， 各有何特性 5. 什么是嵌入式实时操作系统, Android 操作系统属于实时操作系统吗? 6. 一条最长的短信息约占多少byte?
2. 1. Android dvm的进程和Linux的进程, 应用程序的进程是否为同一个概念
3. DVM 执行时，在linux看来就是一应用程序进程，所以说是同一概念
4. 2. sim卡的EF 文件有何作用
5. sim卡的文件系统有自己规范，主要是为了和手机通讯，sim本 身可以有自己的操作系统，EF就是作存储并和手机通讯用的
6. 4.嵌入式操作系统内存管理有哪几种， 各有何特性
7. 页式，段式，段页，用到了MMU,虚拟空间等技术
8. 5. 什么是嵌入式实时操作系统, Android 操作系统属于实时操作系统吗?
9. 分 硬实时和软实时，android属于linux内核，linux在用户空间可抢占，内核空间在2.4以后可局部抢占，严格来讲 Android属于软实时系统
10. 6. 一条最长的短信息约占多少byte? 一条短信可以输入
11. 中文70(包括标点) 英文160 160个字节
12. Android 面试题积累 收藏 1、什么是ANR 如何避免它？
13. http://blog.csdn.net/Zengyangtech/archive/2010/11/21/6025671.aspx
14. 2、什么情况会导致Force Close ？如何避免？能否捕获导致其的异常？
15. 3、Android本身的api并未声明会抛出异常，则其在运行时有无可能抛出runtime异常，你遇到过吗？诺有的话会导致什么问题？如何解决？
16. 4、简要解释一下activity、 intent 、intent filter、service、Broadcast、BroadcaseReceiver
17. http://blog.csdn.net/Zengyangtech/archive/2010/11/21/6025676.aspx
18. 5、IntentService有何优点?
19. IntentService is a base class for Services that handle asynchronous requests (expressed as Intents) on demand. Clients send requests through startService(Intent) calls; the service is started as needed, handles each Intent in turn using a worker thread, and stops itself when it runs out of work.
20. This „work queue processor‟ pattern is commonly used to offload tasks from an application‟s main thread. The IntentService class exists to simplify this pattern and take care of the mechanics. To use it, extend IntentService and implement onHandleIntent(Intent). IntentService will receive the Intents, launch a worker thread, and stop the service as appropriate.
21. All requests are handled on a single worker thread — they may take as long as necessary (and will not block the application‟s main loop), but only one request will be processed at a time.”
22. IntentService 的好处
23. Acitivity的进程，当处理Intent的时候，会产生一个对应的Service
24. Android的进程处理器现在会尽可能的不kill掉你
25. 非常容易使用
26. 日历中IntentService的应用
27. public class DismissAllAlarmsService extends IntentService {
28. @Override public void onHandleIntent(Intent unusedIntent) {
29. ContentResolver resolver = getContentResolver();
30. ...
31. resolver.update(uri, values, selection, null);
32. }
33. }
34. in AlertReceiver extends BroadcastReceiver, onReceive()： (main thread)
35. Intent intent = new Intent(context, DismissAllAlarmsService.class);
36. context.startService(intent);
37. 6.根据自己的理解描述下Android数字签名
38. Android 数字签名 在Android系统中，所有安装到系统的应用程序都必有一个数字证书，此数字证书用于标识应用程序的作者和在应用程序之间建立信任关系,如果一个permission的protectionLevel为signature，那么就只有那些跟该permission所在的程序拥有同一个数字证书的应用程序才能取得该权限。Android使用Java的数字证书相关的机制来给apk加盖数字证书，要理解android的数字证书，需要先了解以下数字证书的概念和java的数字证书机制。Android系统要求每一个安装进系统的应用程序都是经过数字证书签名的，数字证书的私钥则保存在程序开发者的手中。Android将数字证书用来标识应用程序的作者和在应用程序之间建立信任关系，不是用来决定最终用户可以安装哪些应用程序。这个数字证书并不需要权威的数字证书签名机构认证，它只是用来让应用程序包自我认证的。 同一个开发者的多个程序尽可能使用同一个数字证书，这可以带来以下好处。 (1)有利于程序升级，当新版程序和旧版程序的数字证书相同时，Android系统才会认为这两个程序是同一个程序的不同版本。如果新版程序和旧版程序的数字证书不相同，则Android系统认为他们是不同的程序，并产生冲突，会要求新程序更改包名。 (2)有利于程序的模块化设计和开发。Android系统允许拥有同一个数字签名的程序运行在一个进程中，Android程序会将他们视为同一个程序。所以开发者可以将自己的程序分模块开发，而用户只需要在需要的时候下载适当的模块。 (3)可以通过权限(permission)的方式在多个程序间共享数据和代码。Android提供了基于数字证书的权限赋予机制，应用程序可以和其他的程序共享概功能或者数据给那那些与自己拥有相同数字证书的程序。如果某个权限(permission)的protectionLevel是signature，则这个权限就只能授予那些跟该权限所在的包拥有同一个数字证书的程序。 在签名时，需要考虑数字证书的有效期： (1)数字证书的有效期要包含程序的预计生命周期，一旦数字证书失效，持有改数字证书的程序将不能正常升级。 (2)如果多个程序使用同一个数字证书，则该数字证书的有效期要包含所有程序的预计生命周期。 (3)Android Market强制要求所有应用程序数字证书的有效期要持续到2033年10月22日以后。 Android数字证书包含以下几个要点： (1)所有的应用程序都必须有数字证书，Android系统不会安装一个没有数字证书的应用程序 (2)Android程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证 (3)如果要正式发布一个Android ，必须使用一个合适的私钥生成的数字证书来给程序签名，而不能使用adt插件或者ant工具生成的调试证书来发布。 (4)数字证书都是有有效期的，Android只是在应用程序安装的时候才会检
查证书的有效期。如果程序已经安装在系统中，即使证书过期也不会影响程序的正常功能。
说一下你眼中的Android的优点和不足之处(面试华为的人有被问过)
随着Android的越来越红火，不少应聘Android开发的人，难免会被问到这样的问题，就是这个平台的优点，当然有优点也会有缺点的，/ G* L: ]1 X; E. ?( z' V0 N
下面是我从网上总结出来的，希望对大家应聘Android开发有所帮助:1 j( Q" A5 n% Z
Android平台手机 5大优势：
一、开放性 ; C7 h6 e4 e0 U1 p, f: |
在优势方面，Android平台首先就是其开发性，开发的平台允许任何移动终端厂商加入到Android联盟中来。显著的开放性可以使其拥有更多的开发者，随着用户和应用的日益丰富，一个崭新的平台也将很快走向成熟
' T6 F5 h' }* C* d7 J% F
开发性对于Android的发展而言，有利于积累人气，这里的人气包括消费者和厂商，而对于消费者来讲，随大的受益正是丰富的软件资源。开放的平台也会带来更大竞争，如此一来，消费者将可以用更低的价位购得心仪的手机。; Q# ~' ?" \( b6 }8 Y
二、挣脱运营商的束缚 % o! h# S1 s: q6 s" R i+ f
% A. L) C- E! M9 z6 u5 C' q" V
在过去很长的一段时间，特别是在欧美地区，手机应用往往受到运营商制约，使用什么功能接入什么网络，几乎都受到运营商的控制。从去年iPhone 上市 ，用户可以更加方便地连接网络，运营商的制约减少。随着EDGE、HSDPA这些2G至3G移动网络的逐步过渡和提升，手机随意接入网络已不是运营商口中的笑谈，
当你可以通过手机IM软件方便地进行即时聊天时，再回想不久前天价的彩信和图铃下载业务，是不是像噩梦一样？
互联网巨头Google推动的Android终端天生就有网络特色，将让用户离互联网更近。
三、丰富的硬件选择
这一点还是与Android平台的开放性相关，由于Android的开放性，众多的厂商会推出千奇百怪，功能特色各具的多种产品。功能上的差异和特色，却不会影响到数据同步、甚至软件的兼容，好比你从诺基亚 Symbian风格手机 一下改用苹果 iPhone ，同时还可将Symbian中优秀的软件带到iPhone上使用、联系人等资料更是可以方便地转移，是不是非常方便呢？
7 k- K* A' x( @3 i1 ^
四、不受任何限制的开发商
Android平台提供给第三方开发商一个十分宽泛、自由的环境，不会受到各种条条框框的阻扰，可想而知，会有多少新颖别致的软件会诞生。但也有其两面性，血腥、暴力、情色方面的程序和游戏如可控制正是留给Android难题之一。) b9 C/ z1 H1 O3 i
8 m9 M/ j' t2 W8 T2 j. u! \
五、无缝结合的Google应用 * e! l! C3 q! B
如今叱诧互联网的Google已经走过10年度历史，从搜索巨人到全面的互联网渗透，Google服务如地图、邮件、搜索等已经成为连接用户和互联网的重要纽带，而Android平台手机将无缝结合这些优秀的Google服务。4 [ \0 E; v+ G/ C
" l2 a+ q }$ J n2 b
再说Android的5大不足：
一、安全和隐私
: D- l# {# x) s! N
由于手机 与互联网的紧密联系，个人隐私很难得到保守。除了上网过程中经意或不经意留下的个人足迹，Google这个巨人也时时站在你的身后，洞穿一切，因此，互联网的深入将会带来新一轮的隐私危机。1 S' Y6 @$ ]5 h1 J4 `; z" h
二、首先开卖Android手机的不是最大运营商 * m6 W7 X6 y( X/ N+ o" E1 N9 I) x& R' H
众所周知，T-Mobile在23日，于美国纽约发布 了Android首款手机G1。但是在北美市场，最大的两家运营商乃AT&T和Verizon，而目前所知取得Android手机销售权的仅有T-Mobile和Sprint，其中T-Mobile的3G网络相对于其他三家也要逊色不少，因此，用户可以买账购买G1，能否体验到最佳的3G网络服务则要另当别论了！
% @" z2 }" x3 P" \1 F
三、运营商仍然能够影响到Android手机
在国内市场，不少用户对购得移动定制机不满，感觉所购的手机被人涂画了广告一般。这样的情况在国外市场同样出现。Android手机的另一发售运营商Sprint就将在其机型中内置其手机商店程序。
) A7 a& y( v' }5 R
四、同类机型用户减少 ) p; h/ Z8 b% e
在不少手机论坛 都会有针对某一型号的子论坛，对一款手机的使用心得交流，并分享软件资源。而对于Android平台手机，由于厂商丰富，产品类型多样，这样使用同一款机型的用户越来越少，缺少统一机型的程序强化。举个稍显不当的例子，现在山寨机泛滥，品种各异，就很少有专门针对某个型号山寨机的讨论和群组，除了哪些功能异常抢眼、颇受追捧的机型以外。
五、过分依赖开发商缺少标准配置 : |/ F# Q+ I6 I* U+ u, O. |
/ a8 O4 z8 f3 B9 Q8 Z0 m. m: [
在使用PC端的Windows Xp系统的时候，都会内置微软Windows Media Player这样一个浏览器程序，用户可以选择更多样的播放器，如Realplay或暴风影音等。但入手开始使用默认的程序同样可以应付多样的需要。在 Android平台中，由于其开放性，软件更多依赖第三方厂商，比如Android系统的SDK中就没有内置音乐 播放器，全部依赖第三方开发，缺少了产品的统一性。
什么是ANR 如何避免它？
ANR：Application Not Responding，五秒
在Android中，活动管理器和窗口管理器这两个系统服务负责监视应用程序的响应。当出现下列情况时，Android就会显示ANR对话框了：
对输入事件（如按键、触摸屏事件）的响应超过5秒
意向接受器（intentReceiver）超过10秒钟仍未执行完毕
Android应用程序完全运行在一个独立的线程中（例如main）。这就意味着，任何在主线程中运行的，需要消耗大量时间的操作都会引发ANR。因为此时，你的应用程序已经没有机会去响应输入事件和意向广播（Intent broadcast）。
因此，任何运行在主线程中的方法，都要尽可能的只做少量的工作。特别是活动生命周期中的重要方法如onCreate()和 onResume()等更应如此。潜在的比较耗时的操作，如访问网络和数据库；或者是开销很大的计算，比如改变位图的大小，需要在一个单独的子线程中完成（或者是使用异步请求，如数据库操作）。但这并不意味着你的主线程需要进入阻塞状态已等待子线程结束 -- 也不需要调用Therad.wait()或者Thread.sleep()方法。取而代之的是，主线程为子线程提供一个句柄（Handler），让子线程在即将结束的时候调用它（xing:可以参看Snake的例子，这种方法与以前我们所接触的有所不同）。使用这种方法涉及你的应用程序，能够保证你的程序对输入保持良好的响应，从而避免因为输入事件超过5秒钟不被处理而产生的ANR。这种实践需要应用到所有显示用户界面的线程，因为他们都面临着同样的超时问题。
什么情况会导致Force Close ？如何避免？能否捕获导致其的异常？
一般像空指针啊，可以看起logcat，然后对应到程序中 来解决错误
简要解释一下activity、 intent 、intent filter、service、Broadcase、BroadcaseReceiver
一个activity呈现了一个用户可以操作的可视化用户界面
一个service不包含可见的用户界面，而是在后台无限地运行
可以连接到一个正在运行的服务中，连接后，可以通过服务中暴露出来的借口与其进行通信
一个broadcast receiver是一个接收广播消息并作出回应的component，broadcast receiver没有界面
intent:content provider在接收到ContentResolver的请求时被激活。
activity, service和broadcast receiver是被称为intents的异步消息激活的。
一个intent是一个Intent对象，它保存了消息的内容。对于activity和service来说，它指定了请求的操作名称和待操作数据的URI
Intent对象可以显式的指定一个目标component。如果这样的话，android会找到这个component（基于manifest文件中的声明）并激活它。但如果一个目标不是显式指定的，android必须找到响应intent的最佳component。
它是通过将Intent对象和目标的intent filter相比较来完成这一工作的。一个component的intent filter告诉android该component能处理的intent。intent filter也是在manifest文件中声明的。
IntentService有何优点?
其实它也是避免ANR的方法：
IntentService 的好处
* Acitivity的进程，当处理Intent的时候，会产生一个对应的Service
* Android的进程处理器现在会尽可能的不kill掉你
* 非常容易使用
Android dvm的进程和Linux的进程, 应用程序的进程是否为同一个概念
DVM 执行时，在linux看来就是一应用程序进程，所以说是同一概念
sim卡的EF 文件有何作用
sim卡的文件系统有自己规范，主要是为了和手机通讯，sim本 身可以有自己的操作系统，EF就是作存储并和手机通讯用的
嵌入式操作系统内存管理有哪几种， 各有何特性
页式，段式，段页，用到了MMU,虚拟空间等技术
什么是嵌入式实时操作系统, Android 操作系统属于实时操作系统吗?
分 硬实时和软实时，android属于linux内核，linux在用户空间可抢占，内核空间在2.4以后可局部抢占，严格来讲 Android属于软实时系统
一条最长的短信息约占多少byte?
一条短信可以输入
中文70(包括标点)
英文160
160个字节
View如何刷新？
View 可以调用invalidate()和postInvalidate()这两个方法刷新
DDMS与TraceView的区别？
DDMS是一个程序执行查看器，在里面你可以看见线程和堆栈等信息，TraceView是程序性能分析器
activity被回收了怎么办？
activity回收了，那就只有另起了
在Java中如何引入C语言？
java调用C语言程序，可以用JNI接口来实现
Android的基本组件都有什么？
Activity ，Service， Intent ，content Provider ，Brodecast Receiver
IPC通信机制
IPC机制
有了Intent这种基于消息的进程内或进程间通信模型，我们就可以通过Intent去开启一个Service，可以通过Intent跳转到另一个Activity，不论上面的Service或Activity是在当前进程还是其它进程内即不论是当前应用还是其它应用的Service或Activity，通过消息机制都可以进行通信！
但是通过消息机制实现的进程间通信，有一个弊端就是，如果我们的Activity与Service之间的交往不是简单的Activity开启Service操作，而是要随时发一些控制请求，那么必须就要保证Activity在Service的运行过程中随时可以连接到Service。
eg：音乐播放程序
后台的播放服务往往独立运行，以方便在使用其他程序界面时也能听到音乐。同时这个后台播放服务也会定义一个控制接口，比如播放，暂停，快进等方法，任何时候播放程序的界面都可以连接到播放服务，然后通过这组控制接口方法对其控制。
如上的需求仅仅通过Intent去开启Service就无法满足了！从而Android的显得稍微笨重的IPC机制就出现了，然而它的出现只适用于Activity与Service之间的通信，类似于远程方法调用，就像是C/S模式的访问，通过定义AIDL接口文件来定义一个IPC接口，Server端实现IPC接口，Client端调用IPC接口的本地代理。
由于IPC调用是同步的，如果一个IPC服务需要超过几毫秒的时间才能完成的话，你应该避免在Activity的主线程中调用，否则IPC调用会挂起应用程序导致界面失去响应。在这种情况下，应该考虑单起一个线程来处理IPC访问。
两个进程间IPC看起来就象是一个进程进入另一个进程执行代码然后带着执行的结果返回。
IPC机制鼓励我们“尽量利用已有功能，利用IPC和包含已有功能的程序协作完成一个完整的项目
谈谈你对Android NDK的理解
1、前言
6月 26 日， Google Android 发布了 NDK ，引起了很多发人员的兴趣。 NDK 全称： Native Development Kit 。下载地址为： http://developer.android.com/sdk/ndk/1.5_r1/index.html 。
2、误解
新出生的事物，除了惊喜外，也会给我们带来一定的迷惑、误解。
2.1、误解一： NDK 发布之前， Android 不支持进行 C 开发
在Google 中搜索 “NDK” ，很多 “Android 终于可以使用 C++ 开发 ” 之类的标题，这是一种对 Android 平台编程方式的误解。其实， Android 平台从诞生起，就已经支持 C 、 C++ 开发。众所周知， Android 的 SDK 基于 Java 实现，这意味着基于 Android SDK 进行开发的第三方应用都必须使用 Java 语言。但这并不等同于 “ 第三方应用只能使用 Java” 。在 Android SDK 首次发布时， Google 就宣称其虚拟机 Dalvik 支持 JNI 编程方式，也就是第三方应用完全可以通过 JNI 调用自己的 C 动态库，即在 Android 平台上， “Java+C” 的编程方式是一直都可以实现的。
当然这种误解的产生是有根源的：在Android SDK 文档里，找不到任何 JNI 方面的帮助。即使第三方应用开发者使用 JNI 完成了自己的 C 动态链接库（ so ）开发，但是 so 如何和应用程序一起打包成 apk 并发布？这里面也存在技术障碍。我曾经
花了不少时间，安装交叉编译器创建 so ，并通过 asset （资源）方式，实现捆绑 so 发布。但这种方式只能属于取巧的方式，并非官方支持。所以，在 NDK 出来之前，我们将 “Java+C” 的开发模式称之为灰色模式，即官方既不声明 “ 支持这种方式 ” ，也不声明 “ 不支持这种方式 ” 。
2.2、误解二：有了 NDK ，我们可以使用纯 C 开发 Android 应用
Android SDK采用 Java 语言发布，把众多的 C 开发人员排除在第三方应用开发外（ 注意：我们所有讨论都是基于“ 第三方应用开发 ” ， Android 系统基于 Linux ，系统级别的开发肯定是支持 C 语言的。 ）。NDK 的发布，许多人会误以为，类似于 Symbian 、 WM ，在 Android 平台上终于可以使用纯 C 、 C++ 开发第三方应用了！其实不然， NDK 文档明确说明： it is not a good way 。因为 NDK 并没有提供各种系统事件处理支持，也没有提供应用程序生命周期维护。此外，在本次发布的 NDK 中，应用程序 UI 方面的 API 也没有提供。至少目前来说，使用纯 C 、 C++ 开发一个完整应用的条件还不完备。
3、NDK 是什么
对NDK 进行了粗略的研究后，我对 “NDK 是什么 ” 的理解如下：
1、NDK 是一系列工具的集合。
NDK提供了一系列的工具，帮助开发者快速开发 C （或 C++ ）的动态库，并能自动将 so 和 java 应用一起打包成 apk 。这些工具对开发者的帮助是巨大的。
NDK集成了交叉编译器，并提供了相应的 mk 文件隔离 CPU 、平台、 ABI 等差异，开发人员只需要简单修改 mk 文件（指出 “ 哪些文件需要编译 ” 、 “ 编译特性要求 ” 等），就可以创建出 so 。
NDK可以自动地将 so 和 Java 应用一起打包，极大地减轻了开发人员的打包工作。
2、NDK 提供了一份稳定、功能有限的 API 头文件声明。
Google明确声明该 API 是稳定的，在后续所有版本中都稳定支持当前发布的 API 。从该版本的 NDK 中看出，这些 API 支持的功能非常有限，包含有：C 标准库（ libc ）、标准数学库（ libm ）、压缩库（ libz ）、 Log 库（ liblog ）。
4、NDK 带来什么
1、NDK 的发布，使 “Java+C” 的开发方式终于转正，成为官方支持的开发方式。
使用NDK ，我们可以将要求高性能的应用逻辑使用 C 开发，从而提高应用程序的执行效率。
使用NDK ，我们可以将需要保密的应用逻辑使用 C 开发。毕竟， Java 包都是可以反编译的。
NDK促使专业 so 组件商的出现。（乐观猜想，要视乎 Android 用户的数量）
2、NDK 将是 Android 平台支持 C 开发的开端。
NDK提供了的开发工具集合，使开发人员可以便捷地开发、发布 C 组件。同时， Google承诺在 NDK 后续版本中提高 “ 可调式 ” 能力，即提供远程的 gdb 工具，使我们可以便捷地调试 C 源码。在支持 Android 平台 C 开发，我们能感觉到 Google 花费了很大精力，我们有理由憧憬 “C 组件支持 ” 只是 Google Android 平台上C 开
发的开端。毕竟， C 程序员仍然是码农阵营中的绝对主力，将这部分人排除在 Android 应用开发之外，显然是不利于 Android 平台繁荣昌盛的。
如何优化LISTVIEW
1、什么是ANR 如何避免它？
http://blog.csdn.net/Zengyangtech/archive/2010/11/21/6025671.aspx
2、什么情况会导致Force Close ？如何避免？能否捕获导致其的异常？
3、Android本身的api并未声明会抛出异常，则其在运行时有无可能抛出runtime异常，你遇到过吗？诺有的话会导致什么问题？如何解决？
4、简要解释一下activity、 intent 、intent filter、service、Broadcast、BroadcaseReceiver
http://blog.csdn.net/Zengyangtech/archive/2010/11/21/6025676.aspx
5、IntentService有何优点?
IntentService is a base class for Services that handle asynchronous requests (expressed as Intents) on demand. Clients send requests through startService(Intent) calls; the service is started as needed, handles each Intent in turn using a worker thread, and stops itself when it runs out of work.
This „work queue processor‟ pattern is commonly used to offload tasks from an application‟s main thread. The IntentService class exists to simplify this pattern and take care of the mechanics. To use it, extend IntentService and implement onHandleIntent(Intent). IntentService will receive the Intents, launch a worker thread, and stop the service as appropriate.
All requests are handled on a single worker thread — they may take as long as necessary (and will not block the application‟s main loop), but only one request will be processed at a time.”
IntentService 的好处
Acitivity的进程，当处理Intent的时候，会产生一个对应的Service
Android的进程处理器现在会尽可能的不kill掉你
非常容易使用
日历中IntentService的应用
public class DismissAllAlarmsService extends IntentService {
@Override public void onHandleIntent(Intent unusedIntent) {
ContentResolver resolver = getContentResolver();
...
resolver.update(uri, values, selection, null);
}
}
in AlertReceiver extends BroadcastReceiver, onReceive()： (main thread)
Intent intent = new Intent(context, DismissAllAlarmsService.class);
context.startService(intent);
6.根据自己的理解描述下Android数字签名
Android 数字签名
在Android系统中，所有安装到系统的应用程序都必有一个数字证书，此数字证书用于标识应用程序的作者和在应用程序之间建立信任关系,如果一个permission的protectionLevel为signature，那么就只有那些跟该permission所在的程序拥有同一个数字证书的应用程序才能取得该权限。Android使用Java的数字证书相关的机制来给apk加盖数字证书，要理解android的数字证书，需要先了解以下数字证书的概念和java的数字证书机制。Android系统要求每一个安装进系统的应用程序都是经过数字证书签名的，数字证书的私钥则保存在程序开发者的手中。Android将数字证书用来标识应用程序的作者和
在应用程序之间建立信任关系，不是用来决定最终用户可以安装哪些应用程序。这个数字证书并不需要权威的数字证书签名机构认证，它只是用来让应用程序包自我认证的。
同一个开发者的多个程序尽可能使用同一个数字证书，这可以带来以下好处。
(1)有利于程序升级，当新版程序和旧版程序的数字证书相同时，Android系统才会认为这两个程序是同一个程序的不同版本。如果新版程序和旧版程序的数字证书不相同，则Android系统认为他们是不同的程序，并产生冲突，会要求新程序更改包名。
(2)有利于程序的模块化设计和开发。Android系统允许拥有同一个数字签名的程序运行在一个进程中，Android程序会将他们视为同一个程序。所以开发者可以将自己的程序分模块开发，而用户只需要在需要的时候下载适当的模块。
(3)可以通过权限(permission)的方式在多个程序间共享数据和代码。Android提供了基于数字证书的权限赋予机制，应用程序可以和其他的程序共享概功能或者数据给那那些与自己拥有相同数字证书的程序。如果某个权限(permission)的protectionLevel是signature，则这个权限就只能授予那些跟该权限所在的包拥有同一个数字证书的程序。
在签名时，需要考虑数字证书的有效期：
(1)数字证书的有效期要包含程序的预计生命周期，一旦数字证书失效，持有改数字证书的程序将不能正常升级。
(2)如果多个程序使用同一个数字证书，则该数字证书的有效期要包含所有程序的预计生命周期。
(3)Android Market强制要求所有应用程序数字证书的有效期要持续到2033年10月22日以后。
Android数字证书包含以下几个要点：
(1)所有的应用程序都必须有数字证书，Android系统不会安装一个没有数字证书的应用程序
(2)Android程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证
(3)如果要正式发布一个Android ，必须使用一个合适的私钥生成的数字证书来给程序签名，而不能使用adt插件或者ant工具生成的调试证书来发布。
(4)数字证书都是有有效期的，Android只是在应用程序安装的时候才会检查证书的有效期。如果程序已经安装在系统中，即使证书过期也不会影响程序的正常功能。
Android面试题
1. 请描述下Activity的生命周期。
2. 如果后台的Activity由于某原因被系统回收了，如何在被系统回收之前保存当前状态？
3. 如何将一个Activity设置成窗口的样式。(Edited by Sodino)
4. 如何退出Activity？如何安全退出已调用多个Activity的Application？
5. 请介绍下Android中常用的五种布局。
6. 请介绍下Android的数据存储方式。(Edited by Sodino)
7. 请介绍下ContentProvider是如何实现数据共享的。(Edited by Sodino)
8. 如何启用Service，如何停用Service。(Edited by Sodino)
9. 注册广播有几种方式，这些方式有何优缺点？请谈谈Android引入广播机制的用意。
10. 请解释下在单线程模型中Message、Handler、Message Queue、Looper之间的关系。
11. AIDL的全称是什么？如何工作？能处理哪些类型的数据？
12. 请解释下Android程序运行时权限与文件系统权限的区别。(Edited by Sodino)
13. 系统上安装了多种浏览器，能否指定某浏览器访问指定页面？请说明原由。
14. 有一个一维整型数组int[]data保存的是一张宽为width，高为height的图片像素值信息。请写一个算法，将该图片所有的白色不透明(0xffffffff)像素点的透明度调整为50%。
15. 你如何评价Android系统？优缺点。
1.activity的生命周期。
2.横竖屏切换时候activity的生命周期
总结：
1、不设置Activity的android:configChanges时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次
2、设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次
3、设置Activity的android:configChanges="orientation|keyboardHidden"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法
3.android中的动画有哪几类，它们的特点和区别是什么
4.handler机制的原理
5.说说activity，intent，service是什么关系
6.android中线程与线程，进程与进程之间如何通信
7.widget相对位置的完成在antivity的哪个生命周期阶段实现
8.说说mvc模式的原理，它在android中的运用
9.说说在android中有哪几种数据存储方式
10.android中有哪几种解析xml的类，官方推荐哪种？以及它们的原理和区别