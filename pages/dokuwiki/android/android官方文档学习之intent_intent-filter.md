title: android官方文档学习之intent_intent-filter 

#  android官方文档学习之Intent、intent-filter 
地址:http://developer.android.com/guide/components/intents-filters.html
常用Intent列表：http://developer.android.com/guide/components/intents-common.html
##  intent 
下面是**激活各种类型组件**的几个方法:
  * 你可以通过传一个(或者一些要做新的事情)Intent参数给` startActivity()或startActivityForResult() `(当你想要activity返回一个参数)函数()，来启动一个activity.
  * 你可以传一个Intent给` startService() `方法,（或给一个新的指令给正在运行的服启）,或者你可以传一个Intent给` bindService() `方法来邦定到服务.
  * 你可以通过使用` sendBroadcast(), sendOrderedBroadcast(), 或者 sendStickyBroadcast() `三种方法来广播一个intent。
  * 你可以对` ContentResolver `调用` query() `方法，对内容提供者进行查询
###  Intent的Component属性 

Intent对象的` setComponent `(ComponentName comp)方法用于设置Intent的Component属性.`  ComponentName `包含如下几个构造器:
ComponentName(String pkg, String cls)
ComponentName(Context pkg, String cls)
ComponentName(Context pkg, Class<?> cls)
由以上的构造器可知, 创建一个ComponentName对象需要指定包名和类名--这就可以唯一确定一个组件类, 这样应用程序即可根据给定的组件类去启动特定的组件. 例如:
ComponentName comp = new ComponentName(FirstActivity.this, SecondActivity.class);
Intent intent = new Intent();
intent.setComponent(comp);
以上三句代码创建了一个intent对象, 并为其指定了Component属性, 完全等价于下面的代码:
Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
除了使用` setComponent() ` 之外, 还可以使用` setClass(), setClassName() `来显式指定目标组件, 还可以调用` getComponent()方 `法获得Intent中封装的ComponentName对象.
当程序采用这种形式启动组件时, 在Intent中明确的指定了待启动的组件类, 此时的Intent属于显式intent, 显式Intent应用场合比较狭窄, 多用于启动本应用中的component, 因为这种方式需要提前获知目标组件类的全限定名. 而隐式Intent则通过Intent中的action, category, data属性指定目标组件需要满足的若干条件, 系统筛选出满足所有条件的component, 从中选择最合适的component或者由用户选择一个component作为目标组件启动.
如果Intent中指定了ComponentName属性, 则Intent的其他属性将被忽略.
###  Intent的Action属性 

` action属性是一个字符串, 代表某一种特定的动作 `. **Intent类预定义了一些action常量**, 开发者也可以自定义action. 一般来说, 自定义的action应该以application的包名作为前缀, 然后附加特定的大写字符串, 例如"cn.xing.upload.action.UPLOAD_COMPLETE"就是一个命名良好的action.
Intent类的` setAction() `方法用于设定action,`  getAction() `方法可以获取Intent中封装的action.
以下是Intent类中预定义的部分action:
ACTION_VIEW ACTION_SEND ACTION_DIAL
![](/data/dokuwiki/android/pasted/20150607-075939.png)
###  Intent的Category属性 

` category属性也是一个字符串, 用于指定一些目标组件需要满足的额外条件. ` Intent对象中**可以包含任意多个category属性**. Intent类也预定义了一些category常量, 开发者也可以自定义category属性.
Intent类的` addCategory() `方法为Intent添加Category属性,`  getCategories() `方法用于获取Intent中封装的所有category.
以下是Intent类中预定义的部分category:
CATEGORY_HOME--表示目标activity必须是一个显示home screen的activity;
` CATEGORY_LAUNCHER `--表示目标activity可以作为task栈中的初始activity, 并可出现在系统程序启动列表中。 ` 常与ACTION_MAIN配合使用 `;
CATEGORY_GADGET--表示目标activity可以被作为另一个activity的一部分嵌入.
CATEGORY_BROWSABLE:表明target activity允许自己被浏览器调用以用来显示链接的数据。比如：一个图像链接或email链接。
![](/data/dokuwiki/android/pasted/20150607-080132.png)
###  Intent的Data属性 

` data属性指定所操作数据的URI `.** data经常与action配合使用,** 如果action为ACTION_EDIT, data的值应该指明被编辑文档的URI; 如果
action为ACTION_CALL, data的值应该是一个以"tel:"开头并在其后附加号码的URI; **如果action为ACTION_VIEW, data的值应该是一个以"http: "开头并在其后附加网址的URI..**.
Intent类的` setData() `方法用于设置data属性, ` setType() `方法用于设置data的**MIME类型**, ` setDataAndType() `方法可以同时设定两者. 可以通过` getData() `方法获取data属性的值, 通过` getType() `方法获取data的MIME类型.
###  Intent的Extra属性 

` extra是用于传递的键值对数据载体 `。通过Intent启动一个component时, 经常需要携带一些额外的数据过去. 携带数据需要调用Intent的` putExtra() `方法, 该方法存在多个重载方法, 可用于携带基本数据类型及其数组, String类型及其数组, Serializable类型及其数组, Parcelable类型及其数组, Bundle类型等. ` Serializable和Parcelable `类型代表一个可序列化的对象, Bundle与Map类似,可用于存储键值对.**Intent定义了一些以EXTRA_开头的常量。**如一些特殊的extra：ACTION_SEND的EXTRA_EMAIL、EXTRA_SUBJECT


###  Intent的Flag属性 

flag属性是一个int值, 用于通知android系统如何启动目标activity, 或者启动目标activity之后应该采取怎样的后续操作.` setFlags() `  所有的flag都在Intent类中定义, 部分常用flag如下:
  * FLAG_ACTIVITY_NEW_TASK--通知系统将目标activity作为一个新task的初始activity;
  * FLAG_ACTIVITY_NO_HISTORY--通知系统不要将目标activity放入历史栈中;
  * FLAG_FROM_BACKGROUND--通知系统这个Intent来源于后台操作, 而非用户的直接选择...

完整列表见Intent的setFlags方法中的说明。
##  Intent Example 
```

// Create the text message with a string
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType(HTTP.PLAIN_TEXT_TYPE); // "text/plain" MIME type
// This says something like "Share this photo with"
String title = getResources().getString(R.string.chooser_title);
// Create intent to show the chooser dialog
Intent chooser = Intent.createChooser(sendIntent, title);
// Verify that the intent will resolve to an activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}

```
##  示例：返回系统桌面 
```

/** 
* 返回到桌面 
*  
* @param context 
* @return 
*/  
public static void returnDesktop(Context context) { 
Intent intent = new Intent(Intent.ACTION_MAIN);  //
intent.addCategory(Intent.CATEGORY_HOME);  //
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);  //optional
context.startActivity(intent);  
}  

```
##  intent-filter 

如果一个 Intent 请求在一片数据上执行一个动作， Android 如何知道哪个应用程序（和组件）能用来响应这个请求呢？ 
Intent Filter就是 用来注册 Activity 、 Service 和 Broadcast Receiver 具有能在某种数据上执行一个动作的能力。
**使用 Intent Filter ，应用程序组件告诉 Android ，它们能为其它程序的组件的动作请求提供服务**，包括同一个程序的组件、本地的或第三方的应用程序。
为了**注册一个应用程序组件为 Intent 处理者**，在组件的 manifest 节点添加一个 ` intent-filter 标签 `。

在 Intent Filter 节点里使用下面的标签（关联属性），你能指定组件支持的动作、种类和数据：

` <action> `：一条<intent-filter>元素至少应该包含一个<action>，否则任何Intent请求都不能和该<intent-filter>匹配。**如果Intent请求的Action和<intent-filter>中个某一条<action>匹配**，那么该Intent就通过了这条<intent-filter>的动作测试。使用 android:name 特性来指定对响应的动作名，一个好的习惯是使用基于 Java 包的命名方式的命名系统。
```

 < intent-filter >
 < action android:name="com.example.project.SHOW_CURRENT" />
 < action android:name="com.example.project.SHOW_RECENT" />
 < action android:name="com.example.project.SHOW_PENDING" />
 </ intent-filter >

```
   
` <category> `：**只有当Intent请求中所有的Category与组件中某一个IntentFilter的<category>完全匹配时**，才会让该 Intent请求通过测试，IntentFilter中多余的<category>声明并不会导致匹配失败。一个没有指定任何类别测试的 IntentFilter仅仅只会匹配没有设置类别的Intent请求。使用 android:category 属性用来指定在什么样的环境下动作才被响应。每个 Intent Filter 标签可以包含多个 category 标签。你可以指定自定义的种类或使用 Android 提供的标准值，如下所示：
  * ` CATEGORY_DEFAULT：对于接收隐士Intent的intent-filter这是必须的。 ` startActivity() and startActivityForResult() treat all intents as if they declared the CATEGORY_DEFAULT category. ` 如果你没有指定这个category，那么任何的隐式Intent都不会解析到这里。 `
  * ALTERNATIVE:你将在这章的后面所看到的，一个 Intent Filter 的用途是使用动作来帮忙填入上下文菜单。 ALTERNATIVE 种类指定，在某种数据类型的项目上可以替代默认执行的动作。例如，一个联系人的默认动作时浏览它，替代的可能是去编辑或删除它。
  * SELECTED_ALTERNATIVE:与 ALTERNATIVE 类似，但 ALTERNATIVE 总是使用下面所述的 Intent 解析来指向单一的动作。SELECTED_ALTERNATIVE在需要一个可能性列表时使用。
  * BROWSABLE:指定在浏览器中的动作。当 Intent 在浏览器中被引发，都会被指定成 BROWSABLE 种类。
  * GADGET:通过设置 GADGET 种类，你可以指定这个 Activity 可以嵌入到其他的 Activity 来允许。
  *`  HOME `:HOME Activity 是设备启动（登陆屏幕）时显示的第一个 Activity 。**通过指定 Intent Filter 为 HOME 种类而不指定动作的话，你正在将其设为本地 home 画面的替代(即桌面程序)。**
  *`  LAUNCHER `:使用这个种类来让一个 Activity 作为应用程序的启动项。（与ACTION_MAIN配置指定应用入口）

 < intent-filter . . . >
 < category android:name="android.Intent.Category.DEFAULT" />
 < category android:name="android.Intent.Category.BROWSABLE" />
 </ intent-filter >

` <data> `:<data>元素指定了希望接受的Intent请求的` 数据URI和数据类型 `，URI被分成三部分来进行匹配：` scheme、 authority和path `.其中，用setData()设定的Inteat请求的URI数据类型和scheme必须与IntentFilter中所指定的一致。若IntentFilter中还指定了authority或path，它们也需要相匹配才会通过测试。data 标签允许你指定组件能作用的数据的匹配；如果你的组件能处理多个的话，你可以包含多个条件。你可以使用下面属性的任意组合来指定组件支持的数据：```

典型的URI为:scheme://host:port/path
scheme, host, post和path都是可选的. 比较2个data时, 只比较filter中包含的部分. 比如filter的一个data只是指定了scheme部分, 则测试时只是比较data的scheme部分, 只要两者的scheme部分相同, 就视为"相同的data".

```
  * android:host:指定一个有效的主机名（例如， com.google ）。
  * android:mimetype:允许你设定组件能处理的数据类型。例如，<type android:value=”vnd.android.cursor.dir/*”/>能匹配任何 Android 游标。
  * android:path:有效地 URI 路径值（例如， /transport/boats/ ）。
  * android:port:特定主机上的有效端口。
  * android:scheme:需要一个特殊的图示（例如， content 或 http ）。
 < intent-filter . . . >
 < data android:type="video/mpeg" android:scheme="http" . . . />
 < data android:type="audio/mpeg" android:scheme="http" . . . />
 </ intent-filter >
示例1
```

<activity android:name="ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
 <activity android:name="MainActivity">
    <!-- This activity is the main entry, should appear in app launcher -->
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>

<activity android:name="ShareActivity">
    <!-- This activity handles "SEND" actions with text data -->
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
    <!-- This activity also handles "SEND" and "SEND_MULTIPLE" with media data -->
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <action android:name="android.intent.action.SEND_MULTIPLE"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="application/vnd.google.panorama360+jpg"/>
        <data android:mimeType="image/*"/>
        <data android:mimeType="video/*"/>
    </intent-filter>
</activity>

```
` 注意上面都有: <category android:name="android.intent.category.DEFAULT"/> `
接下来的代码片段显示了如何配置 Activity 的 Intent Filter ，使其以在特定数据下的默认的或可替代的动作的身份来执行 SHOW_DAMAGE动作.
```

 <activity android:name=".EarthquakeDamageViewer" 
   android:label="View Damage">
   <intent-filter> 
   <action   
   android:name="com.paad.earthquake.intent.action.SHOW_DAMAGE">    
  </action>    
  <category android:name="android.intent.category.DEFAULT"/>    
   <category 
   android:name="android.intent.category.ALTERNATIVE_SELECTED"  /> 
  <data android:mimeType="vnd.earthquake.cursor.item/*"/> 
  </intent-filter> 
  
  </activity> 
<activity android:name=".EarthquakeDamageViewer"
android:label="View Damage">
<intent-filter>
<action
android:name="com.paad.earthquake.intent.action.SHOW_DAMAGE">
</action>
<category android:name="android.intent.category.DEFAULT"/>
<category

```
##  PendingIntent 
PendingIntent即是一个Intent的描述，我们可以把这个描述交给别的程序，别的程序根据这个描述在后面的别的时间做你安排做的事情。换种说法Intent字面意思是意图，我们想要做的事情，在Activity中我们可以立即执行它，**PendingIntent相当于对Intent执行了包装**，**我们不一定要马上执行它**，我们将其包装后传递给其他Activity或Application。这时获取到PendingIntent的Application能够根据里面的Intent来得知发出者的意图选择执行。
**PendingIntent是Android系统管理并持有**的用于描述和获取原始数据的对象的标志(引用)。这就意味着，**即便创建该PendingIntent对象的进程被杀死了，这个PendingItent对象自己在其他进程中还是可用的。**
**PendingIntent有以下flag：**
  * FLAG_CANCEL_CURRENT:如果当前系统中已经存在一个相同的PendingIntent对象，那么就将先将已有的PendingIntent取消，然后重新生成一个PendingIntent对象。
  * FLAG_NO_CREATE:如果当前系统中不存在相同的PendingIntent对象，系统将不会创建该PendingIntent对象而是直接返回null。
  * FLAG_ONE_SHOT:该PendingIntent只作用一次。
  * FLAG_UPDATE_CURRENT:如果系统中已存在该PendingIntent对象，那么系统将保留该PendingIntent对象，但是会使用新的Intent来更新之前PendingIntent中的Intent对象数据，例如更新Intent中的Extras。

**创建方式:**
 getActivity(Context, int requestCode, Intent, int flag), getActivities(Context, int, Intent[], int), getBroadcast(Context, int, Intent, int), and getService(Context, int, Intent, int)
**主要用途：**
Declare an intent to be executed when the user performs an action with your ` Notification ` (the Android system's ` NotificationManager ` executes the Intent).
Declare an intent to be executed when the user performs an action with your ` App Widget ` (the Home screen app executes the Intent).
Declare an intent to be executed at a specified time in the future (the Android system's`  AlarmManager ` executes the Intent).
###  PendingIntent示例 
```

  NotificationManager mNotificationManager = (NotificationManager)this.getSystemService(NOTIFICATION_SERVICE);  
              Notification mNotification = new Notification(R.drawable.icon,"This is a notification.",System.currentTimeMillis());  
              
              //将使用默认的声音来提醒用户  
////                      mNotification.defaults = Notification.DEFAULT_SOUND;  
                
              Intent mIntent = new Intent(mContext,TestNotificationActivity.class);  
               //这里需要设置Intent.FLAG_ACTIVITY_NEW_TASK属性  
              mIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);    
              PendingIntent mContentIntent =PendingIntent.getActivity(mContext,0, mIntent, 0);  
//            //这里必需要用setLatestEventInfo(上下文,标题,内容,PendingIntent)不然会报错.  
              mNotification.setLatestEventInfo(mContext, "10086", "您的当前话费不足,请充值.哈哈~", mContentIntent);  
//            //这里发送通知(消息ID,通知对象)  
                  mNotificationManager.notify(NOTIFICATION_ID, mNotification); 


```
###   自定义PendingIntent布局 
```


NotificationManager mNotificationManager = (NotificationManager)this.getSystemService(NOTIFICATION_SERVICE);  
Notification mNotification = new Notification(R.drawable.icon,"This is a notification.",System.currentTimeMillis());  
  
//将使用默认的声音和振动来提醒用户  
mNotification.defaults = Notification.DEFAULT_SOUND|Notification.DEFAULT_VIBRATE;  
  
//自定义界面     
RemoteViews rv = new RemoteViews(getPackageName(), R.layout.notification);    
rv.setTextViewText(R.id.notification_tv_one, "one");  
rv.setTextViewText(R.id.notification_tv_two, "two");  
rv.setTextViewText(R.id.notification_tv_three, "three" + System.currentTimeMillis());  
rv.setImageViewResource(R.id.notification_image_one,R.id.notification_image_one);   
   
mNotification.contentView = rv;  
//点击notification之后，该notification自动消失  
mNotification.flags = Notification.FLAG_AUTO_CANCEL;  
//点击notification之后，像Android QQ一样能出现在 “正在运行的”栏目  
//            mNotification.flags = Notification.FLAG_ONGOING_EVENT;  
   
//该意图用来打开NotificationList这个新的Activity  
Intent mIntent = new Intent(mContext,TestNotificationActivity.class);  
  
//这里需要设置Intent.FLAG_ACTIVITY_NEW_TASK属性  
mIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);      
  
//包装该Intent，只有包装后的Intent才能被Notification所用，这是因为Notification需要指定一些额外的参数  
PendingIntent mContentIntent =PendingIntent.getActivity(mContext,0, mIntent, 0);  
  
//这里必需要用setLatestEventInfo(上下文,标题,内容,PendingIntent)不然会报错.  
mNotification.contentIntent = mContentIntent;  
  
//这里发送通知(消息ID,通知对象)  
//NOTIFICATION_ID由自己指定，为每一个Notification对应的唯一标志  
mNotificationManager.notify(NOTIFICATION_ID, mNotification);   

```
` 注意，如果使用了contentView，那么便不要使用Notification.setLatestEventInfo `。如果setLatestEventInfo在赋给 Notification.contentView 的代码之后，那么contentView的效果将被覆盖，显示的便是 setLatestEventInfo 的效果；如果 setLatestEventInfo 在 Notification.contentView 的代码之前，那么显示的便是 Notification.contentView 的效果，也就是说不管你想要setLatestEventInfo 或 contentView 的自定义效果，请保证始终只有一句设置代码，因为在最后一句绑定的时候，之前的设置contentView或setLatestEventInfo的代码都是完全没有必要的
##  Android系统Intent调用： 
```

//显示网页:
Uri uri = Uri.parse("http://www.google.com");
Intent it = new Intent(Intent.ACTION_VIEW,uri);
startActivity(it);
 
//显示地图:
Uri uri = Uri.parse("geo:38.899533,-77.036476");
Intent it = new Intent(Intent.Action_VIEW,uri);
startActivity(it);

//路径规划:
Uri.parse("http://maps.google.com/maps?f=d&saddr=startLat%20startLng&daddr=endLat%20endLng&hl=en");
Intent it = new Intent(Intent.ACTION_VIEW,URI);
startActivity(it);
 
//拨打电话:
Uri uri = Uri.parse("tel:xxxxxx");
Intent it = new Intent(Intent.ACTION_DIAL, uri);
startActivity(it);
//要使用这个必须在配置文件中加入<uses-permission id="android.permission.CALL_PHONE" />
 
//发送 SMS/MMS
Intent it = new Intent(Intent.ACTION_VIEW);
it.putExtra("sms_body", "The SMS text");
it.setType("vnd.android-dir/mms-sms");
startActivity(it);

//发送短信
Uri uri = Uri.parse("smsto:0800000123");
Intent it = new Intent(Intent.ACTION_SENDTO, uri);
it.putExtra("sms_body", "The SMS text");
startActivity(it);
 
//发送彩信
 
Uri uri = Uri.parse("content://media/external/images/media/23");
Intent it = new Intent(Intent.ACTION_SEND);
it.putExtra("sms_body", "some text");
it.putExtra(Intent.EXTRA_STREAM, uri);
it.setType("image/png");
startActivity(it);
 
//发送 Email
Uri uri = Uri.parse("mailto:xxx@abc.com");
Intent it = new Intent(Intent.ACTION_SENDTO, uri);
startActivity(it);
 
Intent it = new Intent(Intent.ACTION_SEND);
it.putExtra(Intent.EXTRA_EMAIL, "me@abc.com");
it.putExtra(Intent.EXTRA_TEXT, "The email body text");
it.setType("text/plain");
startActivity(Intent.createChooser(it, "Choose Email Client"));
 
Intent it=new Intent(Intent.ACTION_SEND);
String[] tos={"me@abc.com"};
String[] ccs={"you@abc.com"};
it.putExtra(Intent.EXTRA_EMAIL, tos);
it.putExtra(Intent.EXTRA_CC, ccs);
it.putExtra(Intent.EXTRA_TEXT, "The email body text");
it.putExtra(Intent.EXTRA_SUBJECT, "The email subject text");
it.setType("message/rfc822");
startActivity(Intent.createChooser(it, "Choose Email Client"));
 
//添加附件
Intent it = new Intent(Intent.ACTION_SEND);
it.putExtra(Intent.EXTRA_SUBJECT, "The email subject text");
it.putExtra(Intent.EXTRA_STREAM, "file:///sdcard/mysong.mp3");
sendIntent.setType("audio/mp3");
startActivity(Intent.createChooser(it, "Choose Email Client"));
 
//播放多媒体
Intent it = new Intent(Intent.ACTION_VIEW);
Uri uri = Uri.parse("file:///sdcard/song.mp3");
it.setDataAndType(uri, "audio/mp3");
startActivity(it);
 
Uri  uri  =  Uri.withAppendedPath(MediaStore.Audio.Media.INTERNAL_CONTENT_URI,"1");
Intent it = new Intent(Intent.ACTION_VIEW, uri);
startActivity(it);
 
//Uninstall  程序
Uri uri = Uri.fromParts("package", strPackageName, null);
Intent it = new Intent(Intent.ACTION_DELETE, uri);
startActivity(it);

//安装APK
Uri installUri = Uri.fromParts("package", "xxx", null);
returnIt = new Intent(Intent.ACTION_PACKAGE_ADDED, installUri);

//调用搜索
Intent intent = new Intent();
intent.setAction(Intent.ACTION_WEB_SEARCH);
intent.putExtra(SearchManager.QUERY,"android123")
startActivity(intent);

//打开照相机 
<1>Intent i = new Intent(Intent.ACTION_CAMERA_BUTTON, null); 
this.sendBroadcast(i); 

<2>long dateTaken = System.currentTimeMillis(); 
String name = createName(dateTaken) + ".jpg"; 
fileName = folder + name; 
ContentValues values = new ContentValues(); 
values.put(Images.Media.TITLE, fileName); 
values.put("_data", fileName); 
values.put(Images.Media.PICASA_ID, fileName); 
values.put(Images.Media.DISPLAY_NAME, fileName); 
values.put(Images.Media.DESCRIPTION, fileName); 
values.put(Images.ImageColumns.BUCKET_DISPLAY_NAME, fileName); 
Uri photoUri = getContentResolver().insert( 
MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values); 
             
Intent inttPhoto = new Intent(MediaStore.ACTION_IMAGE_CAPTURE); 
inttPhoto.putExtra(MediaStore.EXTRA_OUTPUT, photoUri); 
startActivityForResult(inttPhoto, 10); 

//从gallery选取图片 
Intent i = new Intent(); 
i.setType("image/*"); 
i.setAction(Intent.ACTION_GET_CONTENT); 
startActivityForResult(i, 11); 

//打开录音机 
Intent mi = new Intent(Media.RECORD_SOUND_ACTION); 
startActivity(mi); 

//显示应用详细列表       
Uri uri = Uri.parse("market://details?id=app_id");         
Intent it = new Intent(Intent.ACTION_VIEW, uri);         
startActivity(it);         
      
//刚才找app id未果，结果发现用package name也可以 
Uri uri = Uri.parse("market://details?id=<packagename>"); 


//寻找应用       
Uri uri = Uri.parse("market://search?q=pname:pkg_name");         
Intent it = new Intent(Intent.ACTION_VIEW, uri);         
startActivity(it); 
     
//打开联系人列表 
<1>Intent i = new Intent(); 
i.setAction(Intent.ACTION_GET_CONTENT); 
i.setType("vnd.android.cursor.item/phone"); 
startActivityForResult(i, REQUEST_TEXT); 

<2> Uri uri = Uri.parse("content://contacts/people"); 
Intent it = new Intent(Intent.ACTION_PICK, uri); 
startActivityForResult(it, REQUEST_TEXT); 

//打开另一程序 
Intent i = new Intent(); 
ComponentName cn = new ComponentName("com.yellowbook.android2","com.yellowbook.android2.AndroidSearch"); 
i.setComponent(cn); 
i.setAction("android.intent.action.MAIN"); 
startActivityForResult(i, RESULT_OK); 

//调用系统编辑添加联系人（高版本SDK有效）：
Intent it = newIntent(Intent.ACTION_INSERT_OR_EDIT);
it.setType("vnd.android.cursor.item/contact");
//it.setType(Contacts.CONTENT_ITEM_TYPE);
it.putExtra("name","myName");
it.putExtra(android.provider.Contacts.Intents.Insert.COMPANY,  "organization");
it.putExtra(android.provider.Contacts.Intents.Insert.EMAIL,"email");
it.putExtra(android.provider.Contacts.Intents.Insert.PHONE,"homePhone");
it.putExtra(android.provider.Contacts.Intents.Insert.SECONDARY_PHONE,"mobilePhone");
it.putExtra(android.provider.Contacts.Intents.Insert.TERTIARY_PHONE,"workPhone");
it.putExtra(android.provider.Contacts.Intents.Insert.JOB_TITLE,"title");
startActivity(it);
 
//调用系统编辑添加联系人（全有效）：
Intent intent = newIntent(Intent.ACTION_INSERT_OR_EDIT);
intent.setType(People.CONTENT_ITEM_TYPE);
intent.putExtra(Contacts.Intents.Insert.NAME, "My Name");
intent.putExtra(Contacts.Intents.Insert.PHONE, "+1234567890");
intent.putExtra(Contacts.Intents.Insert.PHONE_TYPE,Contacts.PhonesColumns.TYPE_MOBILE);
intent.putExtra(Contacts.Intents.Insert.EMAIL, "com@com.com");
intent.putExtra(Contacts.Intents.Insert.EMAIL_TYPE,Contacts.ContactMethodsColumns.TYPE_WORK);
startActivity(intent);

```
##  Android系统Action大全 
###  标准的Activity Actions 
```
 
ACTION_MAIN   //作为一个主要的进入口，而并不期望去接受数据
ACTION_VIEW   //向用户去显示数据
ACTION_ATTACH_DATA //别用于指定一些数据应该附属于一些其他的地方，例如，图片数据应该附属于联系人
ACTION_EDIT   //访问已给的数据，提供明确的可编辑
ACTION_PICK   //从数据中选择一个子项目，并返回你所选中的项目
ACTION_CHOOSER  //显示一个activity选择器，允许用户在进程之前选择他们想要的
ACTION_GET_CONTENT //允许用户选择特殊种类的数据，并返回（特殊种类的数据：照一张相片或录一段音）
ACTION_DIAL   //拨打一个指定的号码，显示一个带有号码的用户界面，允许用户去启动呼叫
ACTION_CALL   //根据指定的数据执行一次呼叫（ACTION_CALL在应用中启动一次呼叫有缺陷，多数应用ACTION_DIAL，ACTION_CALL不能用在紧急呼叫上，紧急呼叫可以用ACTION_DIAL来实现）
ACTION_SEND   //传递数据，被传送的数据没有指定，接收的action请求用户发数据
ACTION_SENDTO  //发送一跳信息到指定的某人
ACTION_ANSWER  //处理一个打进电话呼叫
ACTION_INSERT  //插入一条空项目到已给的容器
ACTION_DELETE  //从容器中删除已给的数据
ACTION_RUN   //运行数据，无论怎么
ACTION_SYNC   //同步执行一个数据
ACTION_PICK_ACTIVITY//为已知的Intent选择一个Activity，返回别选中的类
ACTION_SEARCH  //执行一次搜索
ACTION_WEB_SEARCH //执行一次web搜索
ACTION_FACTORY_TEST //工场测试的主要进入点，

```

###  标准的广播Actions 
```
 
ACTION_TIME_TICK   //当前时间改变，每分钟都发送，不能通过组件声明来接收，只有通过Context.registerReceiver()方法来注册
ACTION_TIME_CHANGED   //时间被设置
ACTION_TIMEZONE_CHANGED  //时间区改变
ACTION_BOOT_COMPLETED  //系统完成启动后，一次广播
ACTION_PACKAGE_ADDED  //一个新应用包已经安装在设备上，数据包括包名（最新安装的包程序不能接收到这个广播）
ACTION_PACKAGE_CHANGED  //一个已存在的应用程序包已经改变，包括包名
ACTION_PACKAGE_REMOVED  //一个已存在的应用程序包已经从设备上移除，包括包名（正在被安装的包程序不能接收到这个广播）
ACTION_PACKAGE_RESTARTED  //用户重新开始一个包，包的所有进程将被杀死，所有与其联系的运行时间状态应该被移除，包括包名（重新开始包程序不能接收到这个广播）
ACTION_PACKAGE_DATA_CLEARED //用户已经清楚一个包的数据，包括包名（清除包程序不能接收到这个广播）
ACTION_BATTERY_CHANGED   //电池的充电状态、电荷级别改变，不能通过组建声明接收这个广播，只有通过Context.registerReceiver()注册
ACTION_UID_REMOVED    //一个用户ID已经从系统中移除
ACTION_BATTERY_CHANGED //电量变化
ACTION_POWER_CONNECTED //电源连接
ACTION_POWER_DISCONNECTED //电源断开
ACTION_SHUTDOWN  //关机

```
###  非标准Actions 

```

ADD_SHORTCUT_ACTION         "android.intent.action.ADD_SHORTCUT"   //动作：在系统中添加一个快捷方式。
ALL_APPS_ACTION              "android.intent.action.ALL_APPS"    //动作：列举所有可用的应用。输入：无。
ALTERNATIVE_CATEGORY           "android.intent.category.ALTERNATIVE"  //类别：说明 activity 是用户正在浏览的数据的一个可选操作。 
ANSWER_ACTION            "android.intent.action.ANSWER"    //动作：处理拨入的电话。
BATTERY_CHANGED_ACTION         "android.intent.action.BATTERY_CHANGED"  //广播：充电状态，或者电池的电量发生变化。
BOOT_COMPLETED_ACTION          "android.intent.action.BOOT_COMPLETED"  //广播：在系统启动后，这个动作被广播一次（只有一次）。
BROWSABLE_CATEGORY          "android.intent.category.BROWSABLE"   //类别：能够被浏览器安全使用的 activities 必须支持这个类别。
BUG_REPORT_ACTION          "android.intent.action.BUG_REPORT"    //动作：显示 activity 报告错误。
CALL_ACTION           "android.intent.action.CALL"    //动作：拨打电话，被呼叫的联系人在数据中指定。
CALL_FORWARDING_STATE_CHANGED_ACTION    "android.intent.action.CFF"     //广播：语音电话的呼叫转移状态已经改变。
CLEAR_CREDENTIALS_ACTION       "android.intent.action.CLEAR_CREDENTIALS" //动作：清除登陆凭证 (credential)。
CONFIGURATION_CHANGED_ACTION      "android.intent.action.CONFIGURATION_CHANGED"//广播：设备的配置信息已经改变.
DATA_ACTIVITY_STATE_CHANGED_ACTION     "android.intent.action.DATA_ACTIVITY"  //广播：电话的数据活动(data activity)状态（即收发数据的状态）已经改变。
DATA_CONNECTION_STATE_CHANGED_ACTION    "android.intent.action.DATA_STATE"   //广播：电话的数据连接状态已经改变。
DATE_CHANGED_ACTION        "android.intent.action.DATE_CHANGED"  //广播：日期被改变。
DEFAULT_ACTION          "android.intent.action.VIEW"    //动作：和 VIEW_ACTION 相同，是在数据上执行的标准动作。
DEFAULT_CATEGORY         "android.intent.category.DEFAULT"   //类别：如果 activity 是对数据执行确省动作（点击, center press）的一个选项，需要设置这个类别。
DELETE_ACTION          "android.intent.action.DELETE"    //动作：从容器中删除给定的数据。
DEVELOPMENT_PREFERENCE_CATEGORY     "android.intent.category.DEVELOPMENT_PREFERENCE"//类别：说明 activity 是一个设置面板 (development preference panel).
DIAL_ACTION           "android.intent.action.DIAL"    //动作：拨打数据中指定的电话号码。
EDIT_ACTION           "android.intent.action.EDIT"    //动作：为制定的数据显示可编辑界面。
EMBED_CATEGORY          "android.intent.category.EMBED"    //类别：能够在上级（父）activity 中运行。
EMERGENCY_DIAL_ACTION       "android.intent.action.EMERGENCY_DIAL"  //动作：拨打紧急电话号码。
FOTA_CANCEL_ACTION         "android.server.checkin.FOTA_CANCEL"  //广播：取消所有被挂起的 (pending) 更新下载。
FOTA_INSTALL_ACTION         "android.server.checkin.FOTA_INSTALL"  //广播：更新已经被确认，马上就要开始安装。
FOTA_READY_ACTION         "android.server.checkin.FOTA_READY"   //广播：更新已经被下载，可以开始安装。
FOTA_RESTART_ACTION         "android.server.checkin.FOTA_RESTART"  //广播：恢复已经停止的更新下载。
FOTA_UPDATE_ACTION        "android.server.checkin.FOTA_UPDATE"   //广播：通过 OTA 下载并安装操作系统更新。
FRAMEWORK_INSTRUMENTATION_TEST_CATEGORY "android.intent.category.FRAMEWORK_INSTRUMENTATION_TEST" //类别：To be used as code under test for framework instrumentation tests.
GADGET_CATEGORY          "android.intent.category.GADGET"   //类别：这个 activity 可以被嵌入宿主 activity (activity that is hosting gadgets)。
GET_CONTENT_ACTION         "android.intent.action.GET_CONTENT"   //动作：让用户选择数据并返回。
HOME_CATEGORY          "android.intent.category.HOME"    //类别：主屏幕 (activity)，设备启动后显示的第一个 activity。
INSERT_ACTION         "android.intent.action.INSERT"     //动作：在容器中插入一个空项 (item)。
INTENT_EXTRA          "android.intent.extra.INTENT"    //附加数据：和 PICK_ACTIVITY_ACTION 一起使用时，说明用户选择的用来显示的 activity；和 ADD_SHORTCUT_ACTION一起使用的时候，描述要添加的快捷方式。
LABEL_EXTRA           "android.intent.extra.LABEL"     //附加数据：大写字母开头的字符标签，和 ADD_SHORTCUT_ACTION 一起使用。
LAUNCHER_CATEGORY         "android.intent.category.LAUNCHER"    //类别：Activity 应该被显示在顶级的 launcher 中。
LOGIN_ACTION          "android.intent.action.LOGIN"    //动作：获取登录凭证。
MAIN_ACTION          "android.intent.action.MAIN"    //动作：作为主入口点启动，不需要数据。 
MEDIABUTTON_ACTION         "android.intent.action.MEDIABUTTON"   //广播：用户按下了“Media Button”。
MEDIA_BAD_REMOVAL_ACTION       "android.intent.action.MEDIA_BAD_REMOVAL" //广播：扩展介质（扩展卡）已经从 SD 卡插槽拔出，但是挂载点 (mount point) 还没解除 (unmount)。
MEDIA_EJECT_ACTION         "android.intent.action.MEDIA_EJECT"   //广播：用户想要移除扩展介质（拔掉扩展卡）。
MEDIA_MOUNTED_ACTION        "android.intent.action.MEDIA_MOUNTED"   //广播：扩展介质被插入，而且已经被挂载。
MEDIA_REMOVED_ACTION        "android.intent.action.MEDIA_REMOVED"   //广播：扩展介质被移除。
MEDIA_SCANNER_FINISHED_ACTION     "android.intent.action.MEDIA_SCANNER_FINISHED"//广播：已经扫描完介质的一个目录。
MEDIA_SCANNER_STARTED_ACTION      "android.intent.action.MEDIA_SCANNER_STARTED"//广播：开始扫描介质的一个目录。
MEDIA_SHARED_ACTION        "android.intent.action.MEDIA_SHARED"   //广播：扩展介质的挂载被解除 (unmount)，因为它已经作为 USB 大容量存储被共享。
MEDIA_UNMOUNTED_ACTION        "android.intent.action.MEDIA_UNMOUNTED"  //广播：扩展介质存在，但是还没有被挂载 (mount)。
MESSAGE_WAITING_STATE_CHANGED_ACTION    "android.intent.action.MWI"     //广播：电话的消息等待（语音邮件）状态已经改变。
NETWORK_TICKLE_RECEIVED_ACTION      "android.intent.action.NETWORK_TICKLE_RECEIVED"//广播：设备收到了新的网络 "tickle" 通知。
PACKAGE_ADDED_ACTION        "android.intent.action.PACKAGE_ADDED"  //广播：设备上新安装了一个应用程序包。
PACKAGE_REMOVED_ACTION        "android.intent.action.PACKAGE_REMOVED"  //广播：设备上删除了一个应用程序包。
PHONE_STATE_CHANGED_ACTION       "android.intent.action.PHONE_STATE"   //广播：电话状态已经改变。
PICK_ACTION          "android.intent.action.PICK"    //动作：从数据中选择一个项目 (item)，将被选中的项目返回。
PICK_ACTIVITY_ACTION        "android.intent.action.PICK_ACTIVITY"  //动作：选择一个 activity，返回被选择的 activity 的类（名）。
PREFERENCE_CATEGORY         "android.intent.category.PREFERENCE"  //类别：activity是一个设置面板 (preference panel)。
PROVIDER_CHANGED_ACTION        "android.intent.action.PROVIDER_CHANGED" //广播：更新将要（真正）被安装。
PROVISIONING_CHECK_ACTION      "android.intent.action.PROVISIONING_CHECK" //广播：要求 polling of provisioning service 下载最新的设置。
RUN_ACTION           "android.intent.action.RUN"     //动作：运行数据（指定的应用），无论它（应用）是什么。
SAMPLE_CODE_CATEGORY        "android.intent.category.SAMPLE_CODE"  //类别：To be used as an sample code example (not part of the normal user experience).
SCREEN_OFF_ACTION         "android.intent.action.SCREEN_OFF"   //广播：屏幕被关闭。
SCREEN_ON_ACTION         "android.intent.action.SCREEN_ON"    //广播：屏幕已经被打开。
SELECTED_ALTERNATIVE_CATEGORY      "android.intent.category.SELECTED_ALTERNATIVE"//类别：对于被用户选中的数据，activity 是它的一个可选操作。
SENDTO_ACTION          "android.intent.action.SENDTO"    //动作：向 data 指定的接收者发送一个消息。
SERVICE_STATE_CHANGED_ACTION      "android.intent.action.SERVICE_STATE"  //广播：电话服务的状态已经改变。
SETTINGS_ACTION         "android.intent.action.SETTINGS"    //动作：显示系统设置。输入：无。
SIGNAL_STRENGTH_CHANGED_ACTION     "android.intent.action.SIG_STR"    //广播：电话的信号强度已经改变。
STATISTICS_REPORT_ACTION       "android.intent.action.STATISTICS_REPORT" //广播：要求 receivers 报告自己的统计信息。
STATISTICS_STATE_CHANGED_ACTION    "android.intent.action.STATISTICS_STATE_CHANGED"//广播：统计信息服务的状态已经改变。 
SYNC_ACTION           "android.intent.action.SYNC"    //动作：执行数据同步。
TAB_CATEGORY          "android.intent.category.TAB"    //类别：这个 activity 应该在 TabActivity 中作为一个 tab 使用。
TEMPLATE_EXTRA         "android.intent.extra.TEMPLATE"    //附加数据：新记录的初始化模板。
TEST_CATEGORY          "android.intent.category.TEST"    //类别：作为测试目的使用，不是正常的用户体验的一部分。
TIMEZONE_CHANGED_ACTION      "android.intent.action.TIMEZONE_CHANGED"  //广播：时区已经改变。
TIME_CHANGED_ACTION         "android.intent.action.TIME_SET"   //广播：时间已经改变（重新设置）。
TIME_TICK_ACTION         "android.intent.action.TIME_TICK"    //广播：当前时间已经变化（正常的时间流逝）。
UMS_CONNECTED_ACTION        "android.intent.action.UMS_CONNECTED"   //广播：设备进入 USB 大容量存储模式。

``` 

##  Standard Categories 
CATEGORY_ALTERNATIVE  设置这个activity是否可以被认为是用户正在浏览的数据的一个可选择的action  
CATEGORY_APP_BROWSER  和ACTION_MAIN一起使用，用来启动浏览器应用程序     
CATEGORY_APP_CALCULATOR  和ACTION_MAIN一起使用，用来启动计算器应用程序     
CATEGORY_APP_CALENDAR  和ACTION_MAIN一起使用，用来启动日历应用程序     
CATEGORY_APP_CONTACTS  和ACTION_MAIN一起使用，用来启动联系人应用程序     
CATEGORY_APP_EMAIL  和ACTION_MAIN一起使用，用来启动邮件应用程序     
CATEGORY_APP_GALLERY  和ACTION_MAIN一起使用，用来启动图库应用程序  
CATEGORY_APP_MAPS  和ACTION_MAIN一起使用，用来启动地图应用程序  
CATEGORY_APP_MARKET  这个activity允许用户浏览和下载新的应用程序  
CATEGORY_APP_MESSAGING  和ACTION_MAIN一起使用，用来启动短信应用程序  
CATEGORY_APP_MUSIC  和ACTION_MAIN一起使用，用来启动音乐应用程序  
CATEGORY_BROWSABLE   能够被浏览器安全调用的activity必须支持这个category  
CATEGORY_DEFAULT   设置这个activity对于默认的action是否是一个可选的  
CATEGORY_EMBED   可以运行在父activity容器内  
CATEGORY_HOME   主activity，当应用程序启动时，它是第一个显示的activity     
CATEGORY_LAUNCHER  应该在上层的启动列表里显示    
CATEGORY_MONKEY  这个activity可能被monkey或者其他的自动测试工具执行     
CATEGORY_OPENABLE   用来指示一个GET_CONTENT意图只希望ContentResolver.openInputStream能够打开URI     
CATEGORY_PREFERENCE   这个activity是一个选项卡     
CATEGORY_SAMPLE_CODE   作为一个简单的代码示例使用（一般情况下不使用）     
CATEGORY_SELECTED_ALTERNATIVE  设置这个activity是否可以被认为是用户当前选择的数据的一个可选择的action     
CATEGORY_TAB   想要在已有的TabActivity内部作为一个Tab使用     
CATEGORY_TEST  供测试使用（一般情况不使用）     
CATEGORY_UNIT_TEST  联合测试使用  
##  Standard Extra Data 
EXTRA_ALARM_COUNT
EXTRA_BCC
EXTRA_CC
EXTRA_CHANGED_COMPONENT_NAME
EXTRA_DATA_REMOVED
EXTRA_DOCK_STATE
EXTRA_DOCK_STATE_HE_DESK
EXTRA_DOCK_STATE_LE_DESK
EXTRA_DOCK_STATE_CAR
EXTRA_DOCK_STATE_DESK
EXTRA_DOCK_STATE_UNDOCKED
EXTRA_DONT_KILL_APP
EXTRA_EMAIL
EXTRA_INITIAL_INTENTS
EXTRA_INTENT
EXTRA_KEY_EVENT
EXTRA_ORIGINATING_URI
EXTRA_PHONE_NUMBER
EXTRA_REFERRER
EXTRA_REMOTE_INTENT_TOKEN
EXTRA_REPLACING
EXTRA_SHORTCUT_ICON
EXTRA_SHORTCUT_ICON_RESOURCE
EXTRA_SHORTCUT_INTENT
EXTRA_STREAM
EXTRA_SHORTCUT_NAME
EXTRA_SUBJECT
EXTRA_TEMPLATE
EXTRA_TEXT
EXTRA_TITLE
EXTRA_UID
##  Standard Flags 
FLAG_GRANT_READ_URI_PERMISSION
FLAG_GRANT_WRITE_URI_PERMISSION
FLAG_GRANT_PERSISTABLE_URI_PERMISSION
FLAG_GRANT_PREFIX_URI_PERMISSION
FLAG_DEBUG_LOG_RESOLUTION
FLAG_FROM_BACKGROUND
FLAG_ACTIVITY_BROUGHT_TO_FRONT
FLAG_ACTIVITY_CLEAR_TASK
FLAG_ACTIVITY_CLEAR_TOP
FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET
FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
FLAG_ACTIVITY_FORWARD_RESULT
FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY
FLAG_ACTIVITY_MULTIPLE_TASK
FLAG_ACTIVITY_NEW_DOCUMENT
FLAG_ACTIVITY_NEW_TASK
FLAG_ACTIVITY_NO_ANIMATION
FLAG_ACTIVITY_NO_HISTORY
FLAG_ACTIVITY_NO_USER_ACTION
FLAG_ACTIVITY_PREVIOUS_IS_TOP
FLAG_ACTIVITY_RESET_TASK_IF_NEEDED
FLAG_ACTIVITY_REORDER_TO_FRONT
FLAG_ACTIVITY_SINGLE_TOP
FLAG_ACTIVITY_TASK_ON_HOME
FLAG_RECEIVER_REGISTERED_ONLY
参考
http://www.oschina.net/question/157182_45601
http://blog.csdn.net/wuwenxiang91322/article/details/7671593
http://www.cnblogs.com/liushengjie/archive/2012/08/30/2663066.html
http://blog.csdn.net/yangwen123/article/details/8019739
http://blog.csdn.net/yangwen123/article/details/8019514
http://blog.csdn.net/yangwen123/article/details/8019727
http://blog.csdn.net/ygc87/article/details/7480695