title: chapter6gui 

#  GUI 
Created Sunday 15 February 2015

参考网站：http://developer.android.com/design/index.html
http://androidpatterns.com
区分术语：窗口小部件（Widget）和应用窗口小部件（appwidget）
主要包android.widget

##  通过解耦视图和模型处理配置更改 
解决方案：解除用户界面和数据模型的耦合，避免Activity的解构影响你的状态数据

##  以5中不同方式连接事件监听器 
成员类，接口类型，匿名内部类，Activity实现，视图布局中设置OnClick事件属性

##  使用图形按钮改进UI设计 
res/drawable下放置一个自定义的xml文件,selector为根元素。
按钮的三种状态：按下，获得焦点，其他状态。

##  通过Spinner提供下拉选择器 
理想情况下选择列表不应该是硬编码的，而应该来源于一个资源文件，以便进行国际化。res/values/创建一个文件 <resource> <string-array>/<string>

##  处理长按/单击事件 
View.setOnLongClickListener()

##  用属性和TextWatcher接口限制EditText值 
TextWatcher的afterTextChanged(Editable s)方法其中Editable就是输入值

##  实现AutoCompleteTextView 
android:completionThreshold

##  用SQLite数据库查询为AutoCompleteTextView提供数据 
解决：使用SimpleCursorAdapter代替ArrayAdapter

##  将编辑字段转换为密码字段 
EditText的password属性

##  将软键盘上的enter键该成Next键 
在相关视图上设置相应的输入法编辑器（IME）属性
android:imeOptions="actionNext"

##  在活动中处理按键事件 
解决方案：覆盖Activity的onKeyDown(int keyCode,KeyEvent e)
keyCode对应于KeyEvent的相关常量

##  星标：使用RatingBar 

##  震动视图 
解决方案：以xml创建一个动画，然后调用View对象的startAnimation()方法，使用AnimationUtils.loadAnimation()方法加载xml
res/anim
translate根元素

##  提供触觉反馈 
震动权限: android.permission.VIBRATE
获取设备震动器：Vibrator vib=(Vibrator)getSystemService(VIBRATOR_SERVICE)

##  在TableView中浏览不同的活动 

##  创建自定义标题栏 
步骤：1.创建一个用于标题栏的xml文件，根布局元素高度40dp,    ui组件高度28dp
requestWindowFeature(Window.FEATURE_CUSTOM_TITLE);
getWindow.setFeatureInt(Window.FEATURE_CUSTOM_TITLE,R.layout.mainTitleBar);

##  格式化数字 
使用String.format()或者NumberFormat的子类

##  创建出现在两个活动之间的“加载中”屏幕 
new Handler.postDelayed(new Runnable(){...},waitTime);

##  使用SlidingDrawer覆盖其他组件 
向上滑动显示
SlidingDrawer应该放置在FrameLayout或RelativeLayout中。
如果要显示从上到下可以使用开源库org.panel包。

##  在Android中检测手势 
使用GestureDetector类检测单击，滚动，挥动，拨动等简单手势

##  使用Fragment构建UI 
ListFragment(ListActivity),DialogFragment(DialogInterface),PreferenceFragment(PreferenceActivity)
使用Photo Gallery

##  使用Gallery 

##  创建简单的应用程序窗口部件 
桌面控件是通过Broadcasr形式来进行控制的，因此每个桌面控件都对应一个BroadcastReceiver.为简化桌面控件的开发，android提供了一个AppWidgetProvider类，它是BroadcastReceiver的子类。
AppWidgetProvider的四个不同生命周期方法：
onUpdate();onDeleted();onEnabled();onDisabled().
开发步骤：1.创建一个RemoteViews对象，可以指定加载的界面布局文件。一般来说该布局文件主要包含ImageView和TextView，RemoteViews提供修改这两种组件的内容的方法。
2.创建一个ComponentName对象
3.调用AppWidgetManager.updateAppWidget()更新桌面控件。
4.在AndroidManifest中配置AppWidgetProvider，它是一个<receiver>同时配置<intent-filter>以及<meta-data>
<receiver android:name="ExampleAppWidgetProvider" >
<intent-filter>
<action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
</intent-filter>
<meta-data android:name="android.appwidget.provider"					 android:resource="@xml/example_appwidget_info" />
</receiver>
		
res/xml/example_appwidget_info.xml文件内容示例：
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
android:minWidth="40dp"
android:minHeight="40dp"
android:updatePeriodMillis="86400000"
android:previewImage="@drawable/preview"
android:initialLayout="@layout/example_appwidget"		android:configure="com.example.android.ExampleAppWidgetConfigure" 
android:resizeMode="horizontal|vertical"
android:widgetCategory="home_screen|keyguard"
android:initialKeyguardLayout="@layout/example_keyguard">
</appwidget-provider>
		
思想：AppWidgetManager通过RemoteViews来以ComponentName形式更新AppWidgetProvider子类对象。
具体有待研读。

Content-Type: text/x-zim-wiki
Wiki-Format: zim 0.4
Creation-Date: 2015-02-15T12:28:18+08:00

#  菜单，对话框，Toast和通知 
Created Sunday 15 February 2015

##  创建和显示菜单： 
在xml中设置菜单，并覆盖onCreateOptionsMenu(), MenuInflater,及覆盖onOptionsItemSelected()
res/menu

##  创建子菜单 
以java代码方式创建。
onCreateOptionsMenu() , addSubMenu()

##  创建弹出/警告对话框 
使用AlertDialog

##  使用Timepicker/Datepicker窗口小部件 

##  创建类似iPhone滚轮选择器 
使用第三方窗口小部件Android-Wheel

##  创建ProgressDialog 

##  创建带有按钮.图像和文本的自定义对话框 

##  创建可重用的“关于”对话框类 
版本名称可以从PackageInfo类获取。（PackageInfo从PackageManager获得，而PackageManager从当前context获得）
```

static String VersionName(Context context){
	try{
		return context.getPackageManager().getPackgeInfo(context.getPackageName(),0).versionName;
	}catch(NameNotFoundException e){
		return "unknow";
	}
}

```

##  自定义Toast显示 

##  在状态栏中创建通知 
解决：创建一个Notification对象，并提供一个PendingIntent,封装用户选中通知时所执行的实际Intent .     NotificationManager用于显示通知
![](/data/dokuwiki/booknote/androidcookbook/pasted/20150521-050427.png)
The callouts in the illustration refer to the following:

1. Content title
2. Large icon
3. Content text
4. Content info
5. Small icon
6. Time that the notification was issued. You can set an explicit value with setWhen(); if you don't it defaults to the time that the system received the notification.
```

NotificationCompat.Builder mBuilder =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!");
// Creates an explicit intent for an Activity in your app
Intent resultIntent = new Intent(this, ResultActivity.class);

// The stack builder object will contain an artificial back stack for the
// started Activity.//
// This ensures that navigating backward from the Activity leads out of//
// your application to the Home screen.//
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);//
// Adds the back stack for the Intent (but not the Intent itself)//
stackBuilder.addParentStack(ResultActivity.class);//
// Adds the Intent that starts the Activity to the top of the stack//
stackBuilder.addNextIntent(resultIntent);//
PendingIntent resultPendingIntent =//
        stackBuilder.getPendingIntent(//
           0,//
            PendingIntent.FLAG_UPDATE_CURRENT//
       );//
mBuilder.setContentIntent(resultPendingIntent);//
NotificationManager mNotificationManager =//
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);//
//// mId allows you to update the notification later on.//
mNotificationManager.notify(mId, mBuilder.build());//

```

#   GUI ListView 
Created Sunday 15 February 2015

##  在ListView中使用段标题 

##  ListView数据改变后需要调用notifyDataSetChanged()方法通知数据改变。 

##  处理方向变化：从ListView数据值到横向图表 
Activity的onConfigurationChanged()处理方向变化
DroidCharts开源图表程序库。
