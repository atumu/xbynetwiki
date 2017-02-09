title: android_保持设备唤醒 

#  Android 保持设备唤醒 
##  保持屏幕亮着 
确定你的应用需要保持屏幕变亮，比如游戏与电影的应用。最好的方式是使用FLAG_KEEP_SCREEN_ON在你的Activity（仅在Activity不在Service或其他组件里） 例如：
```

public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
  }

```
另外一种实现在你的应用布局xml文件里，通过使用android:keepScreenOn属性:
```

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:keepScreenOn="true">
    ...
</RelativeLayout>

```
但是如果想你通过显式清除flag从而使得屏幕能够关闭，可以使用-clearFlags()); getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON).
##  保持CPU运行 
你可以使用` PowerManager `系统服务特性回调唤醒锁。唤醒锁允许你应用控制本地设备电源状态。
为了使用**唤醒锁**，首先你增加WAKE_LOCK权限在应用主要清单文件：
` <uses-permission andriod:name="andriod.permission.WAKE_LOCK"/> `
这里告诉你之间设置唤醒锁：
```

PowerManager powerManager = (PowerManager) getSystemService(POWER_SERVICE);
Wakelock wakelock = powerManager.newWakeLock(PowerManager.PARTIAL_WAKELOCK),
    "MyWakelockTag");
wakelock.acquire();

```
为了释放唤醒锁，使用` wakelock.release() `。当你的应用使用完毕时释放CPU，这对避免消耗电量是重要的。
