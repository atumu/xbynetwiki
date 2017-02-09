title: chapter2测试 

#   设计成功的应用程序 
Created Friday 30 January 2015

##  导言：设计指导思想 
* 应用程序应该易于安装，删除和更新。
* 优雅地处理用户的需求
* 具有丰富的特性，并且易于使用。
* 对于通过其他途径（例如网站）访问相同信息的用户来说，应用程序应该很熟悉
* 关键的功能区应该很容易访问到。
* 应该与手机上的其他应用有共同的观感，遵循目标平台的标准和风格指导方针。
* 应用程序应该稳定、可伸缩并且反应灵敏
* 得体地利用平台功能，为用户带来更佳的体验。

##  应该注意的几点 
* 每个视图的局部数据在退出时必要地保存，在视图下次加载的时候，这些数据自动恢复并重新填充到对应的控件。
* 屏幕尺寸和密度以及字体：屏幕密度dp,字体sp
* 设备特性：如wifi，蓝牙，传感器，相机，OpenGL等，**你不能假定某个特性在所有android设备上都可用。**
	Android框架支持的两种菜单：（**只有重要的命令才作为按钮，其余的功能交给菜单完成**）
		选项菜单：一般由用户按下硬件的Menu按钮调用。
		上下文菜单：通常由用户长按调用。
* 数据馈送（Feed）：直接与第三方数据源接口不是一个好主意，我们应该采用一个中间件从多种数据源，以及多种数据格式迁移数据。然后中间件通过一系列REST风格的Web服务API,以JSON数据流的形式将数据传递给应用程序。

##  异常处理 
**Android使用对话框和Toast机制通知用户异常的存在。通常使用Toast报告不重要的信息，而使用对话框现实重要的信息并得到确认。**
关于java的异常机制：
检查型异常和非检查异常，以及Error; 异常转译。

##  作为“单例”访问Android应用程序对象 
问题描述：你需要从Android应用程序中访问“全局”数据
解决方案：**子类化android.app.Application**把它作为一个有静态存取方法的单例处理（每个Android应用在生命周期中都有一个android.app.Application实例。由于没有什么方法可以阻止你创建子类化Application类的其他实例，故而这不是一个真正的单例，但是足够接近了）。使会话处理，web服务网关和Application等对象只需要一个实例就可以全局访问、保证Application实例中添加静态存取方法是有价值的，这样将全局访问的数据合并在一起，保证对一个Context实例的访问。用Application实例来访问所有全局数据。
Application的实现：
public MyAndroidApplication extends Application{
	private Application sInstance;

	pblic static Application getInstance(){
		return sInstance;
	}
	@Override
	public void onCreate(){
		super.onCreate();
		sInstance=this;
		sInstance.initializeInstance();
		
	}
	protected void initializeInstance(){
		//编写初始化代码
	}
}
**最后不要忘了在AndroidManifest.xml中添加应用程序声明**
<application android:icon="@drawable/app_icon"
	android:label="@string/myapp"
	**android:name="com.xby.MyAndroidApplication"**>


##  在用户旋转设备时保存数据 
问题：当用户旋转设备时通常会销毁并重新创建当前Activity,你希望在这一周期中保留某些数据。
解决方案：所需保留的数据以简单数据类型或者Serializable的形式在传入的Bundle的onSaveInstanceState()中保存数据。

##  监控android设备的电量 
解决方案：BroadcastReciever可以用来接收电池状态变化时发送的广播，从而确定电量。
通过Activity的registerReceiver()方法，传入BroadcastReceiver和IntentFilter.具体常量定义在Intent中。

##  在Android中创建闪屏 
问题：在应用程序加载时显示闪屏（splash screen）。闪屏通常用于广告后者品牌宣传
解决方案：采用Activity或者对话框的形式构建闪屏
* 以Activity形式：
	在AndroidManifest.xml中设置启动Activity为该闪屏Activity，同时设置闪屏属性android:nohistory="true"以防止闪屏Activity被加入Activity栈中。
	可以通过在闪屏活动中使用Intent启动Main Activity.
* 以对话框形式：
	略。具体看AndroidCookbook的源代码。


##  设计会议/网络研讨/编程马拉松/机构用的应用程序时 
一般功能需求：
* 地图
* 时间表视图
* 签到或者兴趣小组
* 好友或者餐饮查找
* 分享功能
* 记笔记功能

##  在Android应用中使用Google Analytics 
下载lib库
AndroidManifest.xml中添加授权(一般需要访问网络的程序都应该添加)
**<uses-permission android:name="android.permission.INTERNET"/>**
**<uses-permission android:name="android:permission.ACCESS_NETWORK_STATE"/>**
具体看示例代码把。难得打字

##  简单的手电筒应用程序 
解决方案：相机的LED闪光灯可作文手电筒使用。

####  设计步骤： 
1.检查设备是否支持闪光灯，若支持则访问手机中的Camera对象
2.访问Camera对象的参数
3.获得相机支持的闪光灯模式Camera的getSupportedFlashModes().
4.用两种模式FLASH_MODE_TORCH, FLASH_MODE_OFF分别对应ON或OFF状态。

####  一些思考： 
手电筒应用MainActivity应该在后台运行时不要在onPause或者onStop中释放资源这样用户可以边照明边做其他的事情，而不用对着无聊的手电筒开关，而且也可以防止屏幕旋转错误地关闭手电筒。
我们应该只在onDestroy释放资源，虽然这在设备内存吃紧的情况下不一定会被调用，但是在绝大多数情况下是可以的方案。
我们应该考虑设备屏幕旋转时导致Activtiy重启状态丢失的问题。
注意对相关资源出现空引用问题可能是权限相关问题引起的。

####  注意几点： 
在Activity销毁之前释放资源（Camera）引用,否则会引起FC.
权限与硬件支持相关：
	<uses-feature android:name="android.hardware.Camera"
		android:required="true"></uses-feature>
	<uses-feature android:name="android.hardware.camera.FLASHLIGHT"
		android:required="true"></uses-feature>
	<uses-permission android:name="android.permission.CAMERA"></uses-permission>
	<uses-permission android:name="android.permission.FLASHLIGHT"></uses-permission>


####  发散性思考： 
* Main Activity销毁后，该应用进程并没有结束（可以在缓存的应用程序中看到）,所以我们必须释放必要资源。所以可以得出，即使退出了程序主界面但是应用程序进程仍然缓存在后台运行。
* 手电筒应用中涉及到屏幕旋向问题时。我们可以思考如何禁止应用程序随系统的旋向设置改变。具体看后面发散性思考一节。

源代码下载：http://github.com/SaketSrivastav/SimpleTorchLight
.[/](/pages/dokuwiki/./SimpleTorchLight-master.zip)SimpleTorchLight-master.zip 

##  将Android手机应用改编为平板电脑应用 
主要关注点：屏幕分辨率，方向
指导方针：
1.横向布局
2.按钮位置和大小
3.字体大小

##  设置首次运行的首选项 
问题：匿名收集用户使用情况时需要征求用户同意。
解决方案：将共享的首选项作为持久性存储。保存一个只更新一次的值。每次应用启动时都检查该值，如果该值已经设置，则程序是第一次运行，否则，就是第一次运行。使用Application子类进行操作
讨论：
public class MyApplication extends Application｛
	SharedPreferences mPrefs;
	@Override
	public void onCreate(){
		super.onCreate();
		Context mContext=this.getApplicationContext();
		//0=私有模式。只有这个应用程序能够访问该首选项
		mPrefs=mContext.getSharedPreferences("myPrefs",0);
		//初始化代码
	}
	public boolean getFirstRun(){
		return mPrefs.getBoolean("firstRun",true);
	}
	public void setRunned(){
	SharedPreferences,Editor edit=mPrefs.edit();
	edit.putBoolean("firstRun",false);
	edit.commit();
	}
｝
这个标志将在启动Activity中测试：
if((MyApplication)getApplication()).getFirstRun()){
	(MyApplication)getApplication().setRunned();
	//第一次运行专用代码
}else{

}
注意：这种方式设置了的，即使在应用程序更新之后也不会再运行第一次代码
这种方式也可以作为一种简易共享软件的解决方案。

##  为显示格式化时间和日期 
解决方案：
android.text.format.DateFormat类提供API,这些API使用起来毫不费力。

##  用KeyListener控制输入 
解决方案：KeyListener或者EditText的xml约束
讨论：android.text.method.KeyListener主要实现类DigitsKeyListener,DateKeyListener.使用他们的getInstance()方法即可。

##  备份Android应用程序数据 
解决方案：原生android自带的Backup Manager。在中国就是个废。

##  用提示代替工具提示 
android:hint或者android:textColorHint

##  发散性思考——android 应用才程序中禁止屏幕转向 
* //在Android中要让一个程序的界面始终保持一个方向，不随手机方向转动而变化//的办法： 只要在AndroidManifest.xml里面配置一下就可以了。 
在AndroidManifest.xml的activity(需要禁止转向的activity)//配置中加入 android:screenOrientation=”landscape”属性即可(landscape是横向，portrait是纵向)。// 
**要避免在转屏时重启activity**//，可以通过在androidmanifest.xml文件中重新定义方向(//**给每个**//activity加上 android:configChanges=”keyboardHidden|orientation”属性)，//**并根据Activity的重写 onConfigurationChanged(Configuration newConfig)方法来控制，这样在转屏时就不会重启activity了**，而是会去调用 onConfigurationChanged(Configuration newConfig)这个钩子方法。
例如
@Override public void onConfigurationChanged(Configuration newConfig) { 
	super.onConfigurationChanged(newConfig); 
	if(newConfig.orientation==Configuration.ORIENTATION_LANDSCAPE) { 
		//横向 setContentView(R.layout.file_list_landscape); 
	} else { 
		//竖向 setContentView(R.layout.file_list);
		 }
	 }
在模拟器中可以按 CTL+F11 模拟做屏幕旋转。

* 另外一种方式就是禁用方向传感器：从Android 1.5开始系统可以设置Sensor旋转屏幕，如果你的应用在部分方面没有处理好横屏和竖屏的切换，可能需要强制禁用方向感应器Sensor，相关的方法可以在androidmanifest.xml的相关activity中加入android:screenOrientation="nosensor"属性。
参考：
How to disable Screen Auto-Rotation on Android
http://digitaldumptruck.jotabout.com/?p=897
如何在 Android 程序中禁止屏幕旋转和重启Activity
http://www.androidcn.com/news/20110302/00001299.html


##  任务：手动编程实现手电筒以及各种技巧的使用。 


#  测试 
Created Friday 30 January 2015

”尽早并经常测试“
单元测试独立检查各个组件（不访问网络或者数据库），JUnit和TestNG处于领先地位的框架。在需要与其他组件交互时，单元测试使用模拟对象，有多个好的java模拟框架如android自带的mock。
Android提供多个特殊的测试技术。
相关术语：NPE——Null Pointer Exception; 
		ANR——Application Not Responding
		FC——Force Close


##  在Android中进行TDD? 
模拟支持的缺乏，使得在android中进行TDD开发难以实现。
我们需要进行两种测试：UI测试和非UI测试。UI测试在当前项目里面进行，非UI测试在独立测试项目里，而且仅负责非UI逻辑的单元测试。

##  使用基于云的测试在多种设备上进行测试 
相关网站自查，国内如百度开发者

##  测试项目的创建 
源代码下载：https://github.com/asantalla/Hello-Android-Testing
新建android Test Project
对创建的项目AndroidManifest.xml文件主要有如下变化(斜体表示)
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	//package="com.example.android.notepad.tests"//
	android:versionCode="1"
	android:versionName="1.0">
	 //<application>//
	//        <uses-library android:name="android.test.runner" />//
	//    </application>//
	// <instrumentation android:name="android.test.InstrumentationTestRunner"//
	//                     android:targetPackage="com.example.android.notepad"//
	//                     android:label="Tests for com.example.android.notepad"/>//
</manifest>
基于Activity的UI组件测试用例一般需要继承ActivityInstrumentationTestCase2,这个类能对单个Activity进行功能性测试，测试框架会自动调用InstrumentationTestCase.launchActivity()来初始化测试Activity实例。它还支持以下特性：
		run any test method on the UI thread（通过UiThreadTest注解）
		inject custom Intents into your Activity
ActivityInstrumentationTestCase2 can communicate with the Android system and **send keyboard input and click events to the U**I.
setUp(),tearDown()
示例代码：
public class MyFirstTestActivityTest
		extends ActivityInstrumentationTestCase2<MyFirstTestActivity> {

	private MyFirstTestActivity mFirstTestActivity;
	private TextView mFirstTestText;

	public MyFirstTestActivityTest() {
		super(MyFirstTestActivity.class);
	}
/****使用setUp()初始化Test Fixture有如下几点考虑：**
	* **定义变量用于存储相关状态**
	* **创建和存储Activity的引用**
	* **包含该测试Activity中所有需要测试UI 组件的引用**
*/
	@Override
	protected void setUp() throws Exception {
		super.setUp();
	 setActivityInitialTouchMode(true);//**测试UI组件时需要提供，且必须在getActivity调用之前。**
		mFirstTestActivity = getActivity();
		mFirstTestText =
				(TextView) mFirstTestActivity
				.findViewById(R.id.my_first_test_text_view);
	}
}
//添加 Test Preconditions
	public void testPreconditions() {
		assertNotNull(“mFirstTestActivity is null”, mFirstTestActivity);
		assertNotNull(“mFirstTestText is null”, mFirstTestText);
	}
	//添加具体测试方法
	public void testMyFirstTestTextView_labelText() {
		final String expected =
				mFirstTestActivity.getString(R.string.my_first_test);
		final String actual = mFirstTestText.getText().toString();
		assertEquals(expected, actual);
	}


####  ActivityInstrumentationTestCase2发送按键测试： 
   @MediumTest
	public void testGoingRightFromLeftButtonJumpsOverCenterToRight() {
		sendKeys(KeyEvent.KEYCODE_DPAD_RIGHT);
		assertTrue("right button should be focused", mRightButton.isFocused());
	}


####  ActivityInstrumentationTestCase2发送keyboard input测试 
Generally, to send a string input value to an EditText object in ActivityInstrumentationTestCase2, you should:

Use the runOnMainSync() method to run the requestFocus() call synchronously in a loop. This way, the UI thread is blocked until focus is received.
Call waitForIdleSync() method to wait for the main thread to become idle (that is, have no more events to process).
Send a text string to the EditText by calling sendStringSync() and pass your input string as the parameter.
// Send string input value
getInstrumentation().runOnMainSync(new Runnable() {
	@Override
	public void run() {
		senderMessageEditText.requestFocus();
	}
});
getInstrumentation().waitForIdleSync();
getInstrumentation().sendStringSync("Hello Android!");
getInstrumentation().waitForIdleSync();
具体示例片段：
@MediumTest
	public void testGoingLeftFromRightButtonGoesToCenter()  {
 
		getActivity().runOnUiThread(new Runnable() {
			public void run() {
				mRightButton.requestFocus();
			}
		});
		// wait for the request to go through
		getInstrumentation().waitForIdleSync();

		assertTrue(mRightButton.isFocused());

		sendKeys(KeyEvent.KEYCODE_DPAD_LEFT);
		assertTrue("center button should be focused", mCenterButton.isFocused());
	}


##  应用程序崩溃排错 
常见错误：
	”权限拒绝“，没有在AndroidManifest中声明相关权限uses-permission
	NPE：空指针异常
	XML中UI组件拼写错误，可用android lint工具检查


##  用Log.d和LogCat进行调试 
Log.d在编译时不会被编译进去。

##  用BugSense自动从用户那里得到缺陷报告 
添加库并使用即可在web端自动生成报告。但需要联网。

##  使用本地运行时日志分析现场错误 
编写一个全局的RuntimeLog类进行记录

##  用StrictMode保持应用程序敏捷性 
Android的工具StrictMode能够检查”程序未响应“ANR发生的情况。（例如，它能够检测发生在主线程（GUI线程）的数据库读写并记录到LogCat）
该工具是在Gingerbread版本即API9时引入的。
注意StrictMode只应该在开发模式下使用。
示例：在应用程序中启动StrictMode
import android.os.StrictMode
//....
//Gingerbread版本之后才可用，isDebug自己定义，主要用于限制在debug模式在启动该工具
if(Build.VERSION.SDK_INT>=9&&isDebug()){
	StrictMode.enableDefaults();
}

##  运行Monkey程序 
问题：你希望对应用 程序进行一些随机使用·测试。把这些测试交给一个”猴子“是一个好办法
解决方案：Android Monkey命令行工具测试是个不错的选择
例如
	adb shell monkey -p package.name.here --throttle 100 -s 4564 -v 50000 | tee tmp/monkey.log
	* -p 指代特定包
	* -throttle是事件之间的延时
	* -s种子值
	* -v verbose
	* 50000是Monkey模拟的事件数量。

##  发送文本消息以及AVD之间的通话 
启动两个AVD，使用端口发送消息及拨打电话。

##  任务：练习单元测试 

