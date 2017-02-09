title: android官方文档学习之应用程序组成部分 

#  android官方文档学习之应用程序组成部分 
管理Activity的生命周期
Building a Dynamic UI with Fragments
Content Provider
Using DialogFragments-使用DialogFragments
Fragments For All
Multithreading for Performance-多线程展示
这个文档的其他部分将向你介绍如下 内容：
  * 定义在你的应用中核心框架组件（四大组件）
  * 在manifest中,给你的应用,声明组件及设备特点请求
  * 独立于应用代码的资源，可以让你的应用极大的优化它在各种配置设备的表现
##  应用组件 

应用组件是构建Android应用程序的关键和基石。 **每个组件是一个不同的入点**，系统可以从这些点进入到你的应用。
有四个不同类型的应用组件，**每个类型服务于一个不同的目的，并有不同的生命周期**，生命周期定义了如何创建和销毁它.
下面是四种应用组件:
**Activities活动**
一个activity在一个屏幕,显示一个用户接口.
**Services服务**
一个service是长期运行在后台，执行操作的组件，甚至可以为远程进程工作.一个服务不提供用户界面.比如，当用户在其他应用中时，一个服务可能在后台播放音乐,或者在后台获取数据，这并不影响用户跟其他的活动进行交互操作.其他的组件，比如一个activity，可以启动一个服务，并可以让它运行或者邦定到这个activity，以便与其进行交互操作.
**Content providers 内容提供者**
一个content provider管理共享的应用数据集.你可以把数据存在文件系统中，一个SQLite数据库中，网上，或你应用可以访问的永久存储器中.通过内容提供者，其他的应用可以查询甚至修改数据(如果内容提供者允许的话). 比如，Android系统提供一个内容提供者管理用户通信录信息.因此，任何拥用适当权限的应用，可以查询内容提供者的部分来(比如ContactsContract.Data)读取和写入关于某个人的信息.内容提供者对于读取和写入属于你的应用的私有的非共享数据也是非常有用的，比如Note Pad样例应用程序，就使用内容提供者来保存笔记的.

**任何一个应用能启动另一个其他应用的组件,是Android系统设计独一无二的方面(aspect)**.比如，你想要用设备的照相机拍一张图片.其他的应用已经有了这个功能，并且你的应用可以使用它，而不需要你自己去开发一个拍照相的activity.你并不需要合并（包含）或者甚至是链接camera应中的代码; 而只是，简单的启动camera应用中的活动，来拍照就可以了.当拍照完成，甚至把照片返回给你的应用，所以你能使用它。对于用户来讲，camera像是你应用中一部分.当系统开启一个组件时，它会启动那个应用的进程（如果该应用没有运行），并实例化该组件所需要的类.举例，如果你的应用开启一个camera应用的activity,来拍照，**这个activity将运行在属于camera应用的进程中，而不是在你的应用的进程中**.因此，不像大多数其他的系统的应用**,Android应用，没有单个的入点(比如没有main()函数).**
因为系统运行的每个应用,在一**个带有文件权限的，独立的进程**中,这样限制了对其他应用的访问,你的应用不能直接访问其他应用中的组件.但时，Android系统也能激活其他应用的组件.你**必须传一个消息给系统，指定你想要启动的组件，然后系统为你激活这个组件**.
##  激活组件 

4个组件中的其中三个组件---activities,serivces,和broadcast receivers----是被叫做intent的异步消息激活的.在运行时,Intents把某个的组件与其他的组件互相邦定，而不管这个组件是否属于你的应用还是其他的应用(你可以把它们想像成一个消息，用于请求一个其他组件的动作).

**对于activities和services，一个intent(意图)定义了一个要执行的动作(比如：to”view”或"send" 些什么),并指定了要采用的URI格式的数据**(其中一些，是其他组件启动所需要知道的).比如，一个intent可能传送一个请求给一个activity，要显示一张图片或打开一个网页.在有些情况,你启动一个activity接收一个结果,这种情况下，activity将在Intent中返回一个结果.(比如，你可以指示一个intent，让用户取一个人的联系方式，并返回给你，返回的intent中会包含一个指向选定联系方式的URI.)
**对于广播接收者，intent只是定义了一个做为广播的公告.(比如，一个广播指出，设备电池低，它只是包含了一个动作字串，表示”电池低”).**
其他组件,**内容提供者,不会被intents所激活.进一步讲，它是内容解释者(ContentResolver)所请求的目标所激活的.**内容解释者，处理所有与内容提供者的直接交换.所以组件不需要执行与提供者交换,而是调用ContentResolver对象方法.(这一句不好理解。)为了安全起见，组件请求信息与内容提供者之间有一个抽象层.

下面是**激活各种类型组件**的几个方法:
  * 你可以通过传一个(或者一些要做新的事情)Intent参数给` startActivity()或startActivityForResult() `(当你想要activity返回一个参数)函数()，来启动一个activity.
  * 你可以传一个Intent给` startService() `方法,（或给一个新的指令给正在运行的服启）,或者你可以传一个Intent给` bindService() `方法来邦定到服务.
  * 你可以通过使用` sendBroadcast(), sendOrderedBroadcast(), 或者 sendStickyBroadcast() `三种方法来广播一个intent。
  * 你可以对` ContentResolver `调用` query() `方法，对内容提供者进行查询

##  清单文件 
` 在Android系统开启一个应用组件之前,系统必须通过读取AndroidManifest.xml文件来知道组件的存在.你的应用必须把它所有的组件声明在这个文件中 `，并且必须在应用工程的根目录下.
这个manifest文件除了` 声明组件 `外，还处理了许多其他的事情，比如:
  * 指定应用请求的其他` 权限 `，访问网络或访问用户的通信录
  * 声明应用要求的最小API Level,应用使用的是那个API
  * 声明应用请求和使用的` 软硬件特征 `，比如照相机，蓝牙服务，或多点触模屏
  * 应用` 需要链接的API库 `,比如Google Maps library等等

<activity> 声明活动的元素
<service> 声明服务的元素
<receiver> 声明广播接收者元素
<provider> 声明内容提供者元素
在你代码中包含的,**Activites,services和内容提供者，若没有在manifest中声明，对系统来说是不可见的，即将永远不会运行**。但是，**广播接收者即可以在manifest中声明，也可以在代码中动态创建(做为BroadcastReceiver对象)并且通过registerReceiver()方法向系统注册。**

###  声明组件功能（通过intent-filter） 
用一个Activating Components启动activities,services和broadcast 接收者.你也可以在intent中显式的指定目标组件（使用组件类名）。然而，**intent真正强大的是它的intent action.(动作)**。通过使用intent动作，你只须简单的描述**你要执行的action类型,(并且，可选的与执行动作有关的数据)**,并且允许系统在设备上找到一个组件，这样就可以执行那个动作并启动它。如果有多个组件可以执行，intent指定的action，那么用户选择执行那一个.
通过比较设备上的其他应用的manifest文件上的` intent filters `与接收到的intent.**系统确定那个组件可以响应一个intent.** 当你在你的应用的manifest中声明一个组件时，你可以可选择包括` intent filters(意图过滤器),来指定组件的功能 `，以让其能响应其他应用的intents.你可以**加一个组件声明的元素的子元素<intent-filter>，为你组件声明一个意图过滤器。**
比如，一个email应用中，新建email的一个activity可能在它的manifest 中声明了一个意图过滤器intent-filter，以便能响应”send”意图(为了发送邮件)。
然后，在你的应用中的一个activity，创建了一个带有”send” ACTION_SEND的意图。
###  声明应用需求 
通过在你的manifest文件中声明软件硬件要求，明了的指出你的应用支持的硬件类型.比如，如果你的应用需要有照相机，并且使用的API是2.1(API Level 7),你应在你的manifest文件中声明这些要求.这样，那些没有照相机并且Android版本低于2.1的设备，就不能从Android市场上安装你的应用.
下面是一些重要的设备特性，你在设计和开发应用时必须要考虑的..
screen size and density 屏幕尺寸与解释度:<supports-screens> 
Input configurations 输入配置:<uses-configuration>
Device features 设备特性:<uses-feature>
Platform Version 平台版本:<uses-sdk>
##  应用资源 
一个Android应用的组成不仅只是代码----它还有与代码独立的资源，比如图像，音频文件，及与应用可显图像任何其他相关的.比如，你应该定义动画，菜单，风格，颜色，和用XML文件定义活动的布局.
##  自定义权限 
完整权限在` android.Manifest.permission `类中
```

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.me.app.myapp" >
    <permission android:name="com.me.app.myapp.permission.DEADLY_ACTIVITY"
        android:label="@string/permlab_deadlyActivity"
        android:description="@string/permdesc_deadlyActivity"
        android:permissionGroup="android.permission-group.COST_MONEY"
        android:protectionLevel="dangerous" />
    ...
</manifest>

```
Android权限被分为四个等级：（需要注意的是，在声明权限时需要一个` android:protectionLevel `的属性，它代表“风险级别”。必须是以下值之一：
` normal、dangerous、signature、signatureOrSystem `）
  * 普通级：这些权限并不能真正伤害到用户（比如更换壁纸），当程序需要这些权限是，开发者不需要指定程序会自动赋予这些权限。
  * 危险级：这些权限可能会带来真的伤害（比如打电话，打开网络链接等），如果要使用它们需要开发者在AndroidManifest.xml中声明对应的权限。
  * 签名级：如果应用使用的是相同的签名证书时，这些权限会自动授予给声明或者创建这些权限的程序。设计这一层级权限的目的是方便组件间数据共享。
  * 签名/系统级：和签名级一样，例外的是系统镜像是自动获取这些权限的，这一层级是专为设备制造商设计的。

在开发Android应用程序的过程中，如果我们要使用系统的某些服务（比如网络、待机、读写文件权限等）都需要首先像下面这样在AndroidManifest.xml中声明对应的权限，然后才可以在代码中访问这些服务：
```

<!-- 文件读写权限 -->  
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />  
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />  
    <uses-permission android:name="android.permission.WRITE_SETTINGS" />  
  
    <!-- 访问网络的权限 -->  
    <uses-permission android:name="android.permission.INTERNET" />  
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />  
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />  
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />  
      
    <!-- 屏幕唤醒权限 -->  
    <uses-permission android:name="android.permission.WAKE_LOCK" />  
    <uses-permission android:name="android.permission.DEVICE_POWER" />  

```
同样，除了能够使用Android系统提供的各种权限外，我们还可以自定义权限来限制其它程序访问应用的各种服务或者组件。任何程序想要和此程序的组件交互时，都需要声明相应的权限时才能成功地访问。
**自定义权限步骤如下**：
以为一个服务CalledService定义访问权限为例，具体步骤如下：
1、在被调用程序Called的AndroidManifest.xml文件中作如下定义：
```

<!-- Service Permission -->  
    <permission  
        android:name="com.uperone.permission.SERVICE"  
        android:label="@string/app_name"  
        android:permissionGroup="@string/app_name"  
        android:protectionLevel="normal" >  
    </permission> 
          
   <service  
            android:name="com.uperone.called.service.CalledService"  
            android:permission="com.uperone.permission.SERVICE">  
            <intent-filter>  
                <action android:name="com.uperone.action.SERVICE" />  
                <category android:name="android.intent.category.DEFAULT"/>  
            </intent-filter>  
        </service>  

```
2、在需要调用该组件的应用程序Call工程的AndroidManifest.mxl文件中声明对应的权限：
` <uses-permission android:name="com.uperone.permission.SERVICE" />  `
3、在需要调用该组件的应用程序Call工程中启动、停止改服务：
 Intent intent = new Intent( "com.uperone.action.SERVICE" );  
startService(intent);  
特别注意：如果在调用需要权限的组件时没有在Manifest.xml中声明权限，则会在运行对应代码段时报异常！！！！
参考：http://blog.csdn.net/ekeuy/article/details/39697683