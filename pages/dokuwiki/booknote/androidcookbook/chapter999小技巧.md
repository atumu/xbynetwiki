title: chapter999小技巧 

#  android权限 
Created Tuesday 17 March 2015
```

读写SD卡权限：
   <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
	<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
联网权限：
	<uses-permission android:name="android.permission.INTERNET"/>

```



# 小技巧 
Created Sunday 01 February 2015

##  1.日志TAG的设置： 
private static final String TAG=CrimeListFragment.class.**getSimpleName();**

##  2.窗口全屏 
```

创建一个新Activity并在onCreate方法中自定义视图为全屏
//setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE)
Activity设置全屏和无标题栏，要用到andorid.view.Window和Android.view.WindowManager。 Window.FEATURE_NO_TITLE表示无标题栏。 
WindowManager.LayoutParams.FLAG_FULLSCREEN表示全屏。 

```
具体用法如下： 
1、设置全屏可以使用如下代码： 
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN); 
2、设置无title bar可以使用如下代码： 
requestWindowFeature(Window.FEATURE_NO_TITLE); 
3、将这些代码段放在你的Activity里的setContentView之前即可。 
注意：此时千万不要继承ActionBarActivity。否则FC

##  Activity加载xml布局文件之后会调用onWindowFocusChanged() 

##  在Android中创建闪屏 
问题：在应用程序加载时显示闪屏（splash screen）。闪屏通常用于广告后者品牌宣传
解决方案：采用Activity或者对话框的形式构建闪屏
```

* 以Activity形式：
	在AndroidManifest.xml中设置启动Activity为该闪屏Activity，同时设置闪屏属性android:nohistory="true"以防止闪屏Activity被加入Activity栈中。
	可以通过在闪屏活动中使用Intent启动Main Activity.
* 以对话框形式：
	略。具体看AndroidCookbook的源代码。

```


##  对于不需要加入活动栈的应该及时调用finish()销毁 

##  getResouces().getDrawable() 

##  获取当前应用的版本号 
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
##  使用SpannableString高亮显示和使用Linkify链接文本 

##  定义版本号和版本名称： 
<manifest android:versionCode="1"  android:versionName="1.0"

##  修改数据之后记得调用相应Adapter的notifyDataSetChanged()方法 

##  android在apk中获取root权限并执行命令 
首先判断是否设备已经root
```

new File("/system/bin/su").exists()
然后可以Process p=Runtime.getRuntime().exec("su");
	dos=new DataOutputStream(p.getOutuputStream());
	dos.writeBytes(cmd+"\n");
	dos.flush();
	dis=new DataInputStream(p.getInputStream());
	p.waitFor();
	或者用下面的方式：
	Runtime.getRuntime().exec(new String[]{"/system/bin/su","-c",cmd});

```	

##  发散性思考——android 应用才程序中禁止屏幕转向 
在Android中要让一个程序的界面始终保持一个方向，不随手机方向转动而变化的办法： 只要在AndroidManifest.xml里面配置一下就可以了。 
在AndroidManifest.xml的activity(需要禁止转向的activity)配置中加入 **android:screenOrientation=”landscape”**属性即可(landscape是横向，portrait是纵向)。
要避免在转屏时重启activity可以通过在androidmanifest.xml文件中重新定义方向(给每个activity加上 **android:configChanges=”keyboardHidden|orientation”**属性)，并根据Activity的重写 onConfigurationChanged(Configuration newConfig)方法来控制，这样在转屏时就不会重启activity了而是会去调用 **onConfigurationChanged(Configuration newConfig)**这个钩子方法。
例如
```

@Override public void onConfigurationChanged(Configuration newConfig) { 
	super.onConfigurationChanged(newConfig); 
	if(newConfig.orientation==Configuration.ORIENTATION_LANDSCAPE) { 
		//横向 setContentView(R.layout.file_list_landscape); 
	} else { 
		//竖向 setContentView(R.layout.file_list);
		 }
	 }

```
在模拟器中可以按 CTL+F11 模拟做屏幕旋转。

* 另外一种方式就是禁用方向传感器：从Android 1.5开始系统可以设置Sensor旋转屏幕，如果你的应用在部分方面没有处理好横屏和竖屏的切换，可能需要强制禁用方向感应器Sensor，相关的方法可以在androidmanifest.xml的相关activity中加入android:screenOrientation="nosensor"属性。
参考：
How to disable Screen Auto-Rotation on Android
http://digitaldumptruck.jotabout.com/?p=897
如何在 Android 程序中禁止屏幕旋转和重启Activity
http://www.androidcn.com/news/20110302/00001299.html

#  android 添加应用程序到文件打开方式 
参考:http://blog.csdn.net/wangchangshuai0010/article/details/7634485
比如通过文档查看器打开一个文本文件时，会弹出一个可用来打开的软件列表；
如何让自己的软件也出现在该列表中呢？ 通过设置AndroidManifest.xml文件即可：
```

<activity android:name=".EasyNote" android:label="@string/app_name" android:launchMode="singleTask" android:screenOrientation="portrait">
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
<intent-filter>
<action android:name="android.intent.action.VIEW"></action>
<category android:name="android.intent.category.DEFAULT"></category>
<data android:mimeType="text/plain"></data>
</intent-filter>
</activity>

```
第一个<intent-filter>标签是每个程序都有的，关键是要添加第二个！这样你的应用程序就会出现在默认打开列表了。。。

注意需要将mimeType修改成你需要的类型，文本文件当然就是：text/plain

还有其它常用的如：
text/plain（纯文本）
text/html（HTML文档）
application/xhtml+xml（XHTML文档）
image/gif（GIF图像）
image/jpeg（JPEG图像）【PHP中为：image/pjpeg】
image/png（PNG图像）【PHP中为：image/x-png】
video/mpeg（MPEG动画）
application/octet-stream（任意的二进制数据）
application/pdf（PDF文档）
application/msword（Microsoft Word文件）
message/rfc822（RFC 822形式）
multipart/alternative（HTML邮件的HTML形式和纯文本形式，相同内容使用不同形式表示）
application/x-www-form-urlencoded（使用HTTP的POST方法提交的表单）
multipart/form-data（同上，但主要用于表单提交时伴随文件上传的场合）

关于mimeType更多信息可以浏览：http://blog.csdn.net/tt5267621/article/details/7173972
将程序设置关联之后，还需要处理参数传递问题！ 需要在onCreate()里面添加如下示例判断代码（未测试）：

''Intent intent = getIntent();String action = intent.getAction();if(intent.ACTION_VIEW.equals(action)){ TextView tv = (TextView)findViewById(R.id.tvText); tv.setText(intent.getDataString());}
''
**"intent.getDataString()"返回的就是所点击的文件路径**。 形式为:file:///storage/sdcard0/learning/api/android/index.html
