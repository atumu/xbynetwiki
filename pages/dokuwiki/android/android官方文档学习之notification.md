title: android官方文档学习之notification 

#  android官方文档学习之Notification 
示例：![](/data/dokuwiki/android/notifyuser.zip|)
官方文档：http://developer.android.com/training/notify-user/
参考中文汉化文档：http://hukai.me/android-training-course-in-chinese/ux/notify-user/build-notification.html

##  建立Notification 
基于` NotificationCompat.Builder `类的，NotificationCompat.Builder在**Support Library**中。为了给许多各种不同的平台提供最好的notification支持，你应该使用NotificationCompat以及它的子类，特别是NotificationCompat.Builder。
###  创建Notification Buider 
创建Notification时，可以用` NotificationCompat.Builder `对象指定Notification的UI内容与行为。一个Builder至少包含以下内容：
  * 一个小的icon，用` setSmallIcon() `)方法设置
  * 一个标题，用` setContentTitle() `)方法设置。
  * 详细的文本，用` setContentText() `)方法设置

```

NotificationCompat.Builder mBuilder =
    new NotificationCompat.Builder(this)
    .setSmallIcon(R.drawable.notification_icon)
    .setContentTitle("My notification")
    .setContentText("Hello World!");

```
##  Notification priority 
` NotificationCompat.Builder.setPriority() ` ：PRIORITY_MIN (-2) to PRIORITY_MAX (2); PRIORITY_DEFAULT (0).
` setPriority(Notification.PRIORITY_HIGH) `
##  Notification actions 
见后文
##  示例 
```

NotificationCompat.Builder mBuilder =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!");
// Creates an explicit intent for an Activity in your app
Intent resultIntent = new Intent(this, ResultActivity.class);

// The stack builder object will contain an artificial back stack for the
// started Activity.
// This ensures that navigating backward from the Activity leads out of
// your application to the Home screen.
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
// Adds the back stack for the Intent (but not the Intent itself)
stackBuilder.addParentStack(ResultActivity.class);
// Adds the Intent that starts the Activity to the top of the stack
stackBuilder.addNextIntent(resultIntent);
PendingIntent resultPendingIntent =
        stackBuilder.getPendingIntent(
            0,
            PendingIntent.FLAG_UPDATE_CURRENT
        );
mBuilder.setContentIntent(resultPendingIntent);
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// mId allows you to update the notification later on.
mNotificationManager.notify(mId, mBuilder.build());

```
##  Notification特性 
<note>注意Notification.defaults与flags的区别</note>
Notification对象定义了**通知的细节信息和其他提醒的设置，例如声音和闪灯**。
一个状态通知需要以下：
  * 状态栏的图标
  * 标题和信息，除非你定义一个custom notification layout
  * PendingIntent,当通知被点击选择时将会被触发。
可选的设定包括：
  * 标题栏的滚动文本
  * 提醒的声音
  * 振动的设定
  * 闪屏的设定
###  添加声音 
你可以使用默认的声音文件或者自定义的声音文件来提醒用户。
为了使用默认声音，要给defaults属性赋值"DEFAULT_SOUND"：
` notification.defaults |= Notification.DEFAULT_SOUND; `
为了给你的通知使用不同的声音，需要把声音文件的Uri传递给sound属性。如下：
```

notification.sound = Uri.parse("file:///sdcard/notification/ringer.mp3");

```
在下面的例子里，我们从内部的MediaStore's ContentProvider:
```

notification.sound = Uri.withAppendedPath(Audio.Media.INTERNAL_CONTENT_URI, "6");

```
在上面的例子中，数字6是媒体文件的ID，而且被加在了Uri的后面。如果你不知道确切的ID，你必须使用ContentResolver在MediaStore要查询一下。你可以查看Content Providers文档，来了解如何使用ContentResolver。
如果你希望提示音可以反复的播放，直到用户对通知做出了反应或者通知被取消了，可以把` FLAG_INSISTENT `赋值给flags属性。
注意：如果defaults属性的值是DEFAULT_SOUND,那么无论设置什么声音都不会有效果的，仍只会播放默认的声音。
###  添加振动 
你可以用默认的振动方式或自定义的振动方式来提示用户。
默认的方式，要用到DEFAULT_VIBRATE
` notification.defaults |= Notification.DEFAULT_VIBRATE; `
自定义的方式，要是定义一个long型数组，赋值给vibrate属性：
''long[] vibrate = {0,100,200,300};
notification.vibrate = vibrate;''
long型的数组定义了交替振动的方式和振动的时间（毫秒）。第一个值是指振动前的准备（间歇）时间，第二个值是第一次振动的时间，第三个值又是间歇的时间，以此类推。振动的方式任你设定。但是不能够反复不停。
###  添加闪灯 
默认的方式，DEFAULT_LIGHTS
` notification.defaults |= Notification.DEFAULT_LIGHTS; `
可以自定义灯光的颜色和闪动的方式。ledARGB属性是定义颜色的，ledOffMS属性是定义灯光关闭的时间（毫秒），ledOnMs是灯光打开的时间（毫秒），还要给flags属性赋值为FLAG_SHOW_LIGHTS
''notification.ledARGB = 0xff00ff00;
notification.ledOnMS = 300;
notification.ledOffMS = 1000;
notification.flags |= Notification.FLAG_SHOW_LIGHTS;''
上面的例子中，**绿色的灯亮了300毫秒，暗了1秒，，，，如此循环。**

**更多特性**
FLAG_AUTO_CANCEL标志
使用这个标志可以让在通知被选择之后，通知提示会自动的消失。
FLAG_INSISTENT标志
使用这个标志，可以让提示音循环播放，知道用户响应。
FLAG_ONGOING_EVENT标志
使用这个标记，可以让该通知成为正在运行的应用的通知。 这说明应用还在运行-它的进程还跑在后台，即使是当应用在前台不可见（就像音乐播放和电话通话）。
FLAG_NO_CLEAR标志
使用这个标志，说明通知必须被清除，通过"Clear notifications"按钮。如果你的应用还在运行，那么这个就非常有用了。
number属性
这个属性的值，指出了当前通知的数量。这个数字是显示在状态通知的图标上的。如果你想使用这个属性，那么当第一个通知被创建的时候，它的值要从1开始。而不是零。
iconLevel
这个属性的值，指出了LevelListDrawable的当前水平。你可以通过改变它的值与LevelListDrawable定义的drawable相关联，从而实现通知图标在状态栏上的动画。查看LevelListDrawable可以获得更多信息。
查看Notification可以了解更多。

###  定义Notification的Action（行为） 
尽管在Notification中Actions是可选的，但是你应该至少添加一种Action。一种Action可以让用户从Notification直接进入你应用内的Activity，在这个activity中他们可以查看引起Notification的事件或者做下一步的处理。在Notification中，action本身是由` PendingIntent `定义的，**PendingIntent包含了一个启动你应用内Activity的Intent。**
```

Intent resultIntent = new Intent(this, ResultActivity.class);
...
// Because clicking the notification opens a new ("special") activity, there's
// no need to create an artificial back stack.
PendingIntent resultPendingIntent =
    PendingIntent.getActivity(
    this,
    0,
    resultIntent,
    PendingIntent.FLAG_UPDATE_CURRENT
);

```

```

PendingIntent resultPendingIntent;
...
mBuilder.setContentIntent(resultPendingIntent);

```
###  发布Notification 
```

NotificationCompat.Builder mBuilder;
...
// Sets an ID for the notification
int mNotificationId = 001;
// Gets an instance of the NotificationManager service
NotificationManager mNotifyMgr =
        (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
// Builds the notification and issues it.
mNotifyMgr.notify(mNotificationId, mBuilder.build());

```
##  Notification启动Activity时保留导航或回退栈到Main Activity 
**常规方式**
1 在manifest中定义你application的Activity层次，最终的manifest文件应该像这个：
```

<activity
    android:name=".MainActivity"
    android:label="@string/app_name" >
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity
    android:name=".ResultActivity"
    android:parentActivityName=".MainActivity">
    <meta-data
        android:name="android.support.PARENT_ACTIVITY"
        android:value=".MainActivity"/>
</activity>

```
2 在基于启动Activity的Intent中**创建一个返回栈(` TaskStackBuilder `)**，比如：
```

int id = 1;
...
Intent resultIntent = new Intent(this, ResultActivity.class);
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
// Adds the back stack
stackBuilder.addParentStack(ResultActivity.class);
// Adds the Intent to the top of the stack
stackBuilder.addNextIntent(resultIntent);
// Gets a PendingIntent containing the entire back stack
PendingIntent resultPendingIntent =
        stackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);
...
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
builder.setContentIntent(resultPendingIntent);
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mNotificationManager.notify(id, builder.build());

```

**特定方式**
一个特定的Activity不需要一个返回栈，所以你不需要在manifest中定义Activity的层次，以及你不需要调用 addParentStack())方法去构建一个返回栈。作为代替，你需要用manifest设置Activity任务选项，以及调用 getActivity())创建PendingIntent
1、manifest中，在Activity的 标签中增加下列属性： android:name="activityclass" activity的完整的类名。 android:taskAffinity="" 结合你在代码里设置的FLAG_ACTIVITY_NEW_TASK标识， 确保这个Activity不会进入application的默认任务。任何与 application的默认任务有密切关系的任务都不会受到影响。 android:excludeFromRecents="true" 将新任务从最近列表中排除，目的是为了防止用户不小心返回到它。

2、建立以及发布notification： a.创建一个启动Activity的Intent. b.通过调用setFlags())方法并设置标识FLAG_ACTIVITY_NEW_TASK 与 FLAG_ACTIVITY_CLEAR_TASK，来设置Activity在一个新的，空的任务中启动。 c.在Intent中设置其他你需要的选项。 d.通过调用 getActivity()方法从Intent中创建一个 PendingIntent，你可以把这个PendingIntent 当做 setContentIntent()的参数来使用。 下面的代码片段演示了这个过程：
```

// Instantiate a Builder object.
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
// Creates an Intent for the Activity
Intent notifyIntent =
        new Intent(new ComponentName(this, ResultActivity.class));
// Sets the Activity to start in a new, empty task
notifyIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK |
        Intent.FLAG_ACTIVITY_CLEAR_TASK);
// Creates the PendingIntent
PendingIntent notifyIntent =
        PendingIntent.getActivity(
        this,
        0,
        notifyIntent,
        PendingIntent.FLAG_UPDATE_CURRENT
);

// Puts the PendingIntent into the notification builder
builder.setContentIntent(notifyIntent);
// Notifications are issued by sending them to the
// NotificationManager system service.
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// Builds an anonymous Notification object from the builder, and
// passes it to the NotificationManager
mNotificationManager.notify(id, builder.build());

```
##  更新Notification 
当你需要对同一事件发布多次Notification时，你应该避免每次都生成一个全新的Notification。相反，你应该考虑去更新先前的Notification，或者改变它的值，或者增加一些值，或者两者同时进行。
` 一般保留Builder的实例引用和ID，在需要更新Notification的时候重新Builder.build()然后notify()即可。 `
###  改变一个Notification 
想要设置一个可以被更新的Notification，需要在发布它的时候调用NotificationManager.notify(ID, notification))方法为它指定一` 个notification ID `。更新一个已经发布的Notification，需要更新或者创建一个NotificationCompat.Builder对象，并从这个对象创建一个Notification对象，然后用与先前一样的ID去发布这个Notification。

下面的代码片段演示了更新一个notification来反映事件发生的次数，它把notification堆积起来，显示一个总数。
```

mNotificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// Sets an ID for the notification, so it can be updated
int notifyID = 1;
mNotifyBuilder = new NotificationCompat.Builder(this)
    .setContentTitle("New Message")
    .setContentText("You've received new messages.")
    .setSmallIcon(R.drawable.ic_notify_status)
numMessages = 0;
// Start of a loop that processes data and then notifies the user
...
    mNotifyBuilder.setContentText(currentText)
        .setNumber(++numMessages);
    // Because the ID remains unchanged, the existing notification is
    // updated.
    mNotificationManager.notify(
            notifyID,
            mNotifyBuilder.build());

```
###  移除Notification 
* 用户清除Notification单独地或者使用“清除所有”（如果Notification能被清除）。
* 你在创建notification时调用了 ` setAutoCancel(boolean) `方法，以及用户点击了这个notification，
* 你为一个指定的 notification ID调用了` cancel(int) `方法。这个方法也会删除正在进行的notifications。
* 你调用了` cancelAll() `方法，它将会移除你先前发布的所有Notification。
##  BigView与BigText显示多行文本style 
关于Notification显示多行文本更多详情请参考：[ Notification显示多行文本](/pages/dokuwiki/android/Notification显示多行文本)
Notification抽屉中的Notification主要有两种视觉展示形式，**normal view**（平常的视图，下同） 与 **big view**（大视图，下同）。**Notification的 big view样式只有当Notification被扩展时才能出现。当Notification在Notification抽屉的最上方或者用户点击Notification时才会展现大视图。**（注意360与其他的状态栏显示会影响显示，所以最好还是在模拟器上做实验）
**Big views在Android4.1被引进的**，它不支持老版本设备。这节课叫你如何让把big view notifications合并进你的APP，同时提供normal view的全部功能。
![](/data/dokuwiki/android/pasted/20150608-133206.png)![](/data/dokuwiki/android/pasted/20150608-133211.png)
###  构造big view 
这个代码片段展示了如何在big view中设置buttons
```

// Sets up the Snooze and Dismiss action buttons that will appear in the
// big view of the notification.
Intent dismissIntent = new Intent(this, PingService.class);
dismissIntent.setAction(CommonConstants.ACTION_DISMISS);
PendingIntent piDismiss = PendingIntent.getService(this, 0, dismissIntent, 0);

Intent snoozeIntent = new Intent(this, PingService.class);
snoozeIntent.setAction(CommonConstants.ACTION_SNOOZE);
PendingIntent piSnooze = PendingIntent.getService(this, 0, snoozeIntent, 0);

```
###  显示多行文本与设置Action 

这个代码片段展示了如何构造一个Builder对象，它设置了big view 的样式为"big text",同时设置了它的内容为提醒文字。它使用addAction())方法来添加将要在big view中出现的Snooze与Dismiss按钮（以及它们相关联的pending intents).
```

// Constructs the Builder object.
NotificationCompat.Builder builder =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.ic_stat_notification)
        .setContentTitle(getString(R.string.notification))
        .setContentText(getString(R.string.ping))
        .setDefaults(Notification.DEFAULT_ALL) // requires VIBRATE permission
        /*
         * Sets the big view "big text" style and supplies the
         * text (the user's reminder message) that will be displayed
         * in the detail area of the expanded notification.
         * These calls are ignored by the support library for
         * pre-4.1 devices.
         */
        .setStyle(new NotificationCompat.BigTextStyle()
                .bigText(msg))
        .addAction (R.drawable.ic_stat_dismiss,
                getString(R.string.dismiss), piDismiss)
        .addAction (R.drawable.ic_stat_snooze,
                getString(R.string.snooze), piSnooze);

```
##  显示Notification进度 
Notifications可以包含一个展示用户正在进行的操作状态的动画进度指示器。“determinate（确定的,indeterminate（不确定.进度指示器用ProgressBar平台实现类来显示。
使用进度指示器，可以调用 ` setProgress() `方法。
###  展示determinate的进度指示器 
为了展示一个确定长度的进度条，调用`  setProgress(max, progress, false)) `方法将进度条添加进notification，然后发布这个notification，第三个参数是个boolean类型，决定进度条是 indeterminate (true) 还是 determinate (false)。**在你操作进行时，增加progress，更新notification。**在操作结束时，progress应该等于max。一个常用的调用 setProgress())的方法是设置max为100，然后增加progress就像操作的“完成百分比”。
当操作完成的时候，你可以选择或者让进度条继续展示，或者移除它。**无论哪种情况下，记得更新notification的文字来显示操作完成。移除进度条，调用` setProgress(0, 0, false)) `方法.**比如：
```

int id = 1;
...
mNotifyManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mBuilder = new NotificationCompat.Builder(this);
mBuilder.setContentTitle("Picture Download")
    .setContentText("Download in progress")
    .setSmallIcon(R.drawable.ic_notification);
// Start a lengthy operation in a background thread
new Thread(
    new Runnable() {
        @Override
        public void run() {
            int incr;
            // Do the "lengthy" operation 20 times
            for (incr = 0; incr <= 100; incr+=5) {
                    // Sets the progress indicator to a max value, the
                    // current completion percentage, and "determinate"
                    // state
                    mBuilder.setProgress(100, incr, false);
                    // Displays the progress bar for the first time.
                    mNotifyManager.notify(id, mBuilder.build());
                        // Sleeps the thread, simulating an operation
                        // that takes time
                        try {
                            // Sleep for 5 seconds
                            Thread.sleep(5*1000);
                        } catch (InterruptedException e) {
                            Log.d(TAG, "sleep failure");
                        }
            }
            // When the loop is finished, updates the notification
            mBuilder.setContentText("Download complete")
            // Removes the progress bar
                    .setProgress(0,0,false);
            mNotifyManager.notify(id, mBuilder.build());
        }
    }
// Starts the thread by calling the run() method in its Runnable
).start();

```
![](/data/dokuwiki/android/pasted/20150608-134116.png)![](/data/dokuwiki/android/pasted/20150608-134121.png)
###  展示indeterminate的指示器 
为了展示一个持续的(indeterminate)活动的指示器,用` setProgress(0, 0, true)) `方法把指示器添加进notification，然后发布这个notification 。前两个参数忽略，第三个参数决定indicator 还是 indeterminate。结果是指示器与进度条有同样的样式，除了它的动画正在进行。
在操作开始的时候发布notification，动画将会一直进行直到你更新notification。当操作完成时，调用`  setProgress(0, 0, false)) ` 方法，然后更新notification来移除这个动画指示器。
![](/data/dokuwiki/android/pasted/20150608-134315.png)
##  使用InBox布局样式to a notification 
```

NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(this)
    .setSmallIcon(R.drawable.notification_icon)
    .setContentTitle("Event tracker")
    .setContentText("Events received")
NotificationCompat.InboxStyle inboxStyle =
        new NotificationCompat.InboxStyle();
String[] events = new String[6];
// Sets a title for the Inbox in expanded layout
inboxStyle.setBigContentTitle("Event tracker details:");
...
// Moves events into the expanded layout
for (int i=0; i < events.length; i++) {

    inboxStyle.addLine(events[i]);
}
// Moves the expanded layout object into the notification object.
mBuilder.setStyle(inBoxStyle);
...
// Issue the notification here.

```
##  自定义通知的布局 
然而，你也可以使用RemoteViews为通知界面定义一个布局。它看起来与默认的布局很像，但实际上是由XML创建的。
为了自定义通知的布局，首先实例化` RemoteViews `创建一个布局文件，然后把RemoteViews传递给` Notification的contentView `属性。
通过下面的例子可以更好的理解：
1. 创建一个XML布局文件 custom_notification.xml:
```

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:padding="10dp" >
    <ImageView android:id="@+id/image"
        android:layout_width="wrap_content"
        android:layout_height="fill_parent"
        android:layout_alignParentLeft="true"
        android:layout_marginRight="10dp" />
    <TextView android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/image"
        style="@style/NotificationTitle" />
    <TextView android:id="@+id/text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/image"
        android:layout_below="@id/title"
        style="@style/NotificationText" />
</RelativeLayout>

```
注意那两个TextView的style属性。在定制的通知界面中，**为文本使用style文件进行定义是很重要的**，因为通知界面的背景色会因为不同的硬件，不同的os版本而改变。从android2.3(API 9)开始，系统为默认的通知界面定义了文本的style属性。因此，你应该使用style属性，以便于在android2.3或更高的版本上可以清晰地显示你的文本，而不被背景色干扰。
```

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="NotificationText" parent="android:TextAppearance.StatusBar.EventContent" />
    <style name="NotificationTitle" parent="android:TextAppearance.StatusBar.EventContent.Title" />
</resources>

```
2. 现在，在应用的代码中，使用RemoveViews方法定义了图片和文本。然后把RemoveViews对象传给contentView属性。例子如下：
```

RemoteViews contentView = new RemoteViews(getPackageName(), R.layout.custom_notification_layout);
contentView.setImageViewResource(R.id.image, R.drawable.notification_image);
contentView.setTextViewText(R.id.title, "Custom notification");
contentView.setTextViewText(R.id.text, "This is a custom layout");
notification.contentView = contentView;

```
RemoteViews类还包括一些方法可以让你轻松地在通知界面的布局里添加Chronometer和ProgressBar。如果要为你的通知界面做更多的定制，请参考RemoteViews。
