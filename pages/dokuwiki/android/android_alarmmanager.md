title: android_alarmmanager 

#  Android 定时器/闹钟AlarmManager 
AlarmManager这个类提供对系统闹钟服务的访问接口。AlarmManager的使用机制有的称呼为全局定时器，有的称呼为闹钟。
你可以为你的应用设定一个在未来某个时间唤醒的功能。
当闹钟响起，实际上是系统发出了为这个闹钟注册的广播，会自动开启目标应用。
注册的闹钟在设备睡眠的时候仍然会保留，可以选择性地设置是否唤醒设备，但是当设备关机和重启后，闹钟将会被清除。
注意：Alarm Manager主要是用来在特定时刻运行你的代码，即便是你的应用在那个特定时刻没有跑的情况。
对于常规的计时操作(ticks, timeouts, etc)，使用Handler处理更加方便和有效率。

` 当你的应用不在运行，而此时你仍然需要你的应用去执行一些操作（比如，短信拦截），只有这种时候才使用AlarmManager, 其他正常情况下的，推荐使用Handler。 `
` AlarmManager ` 包含的主要方法：
```

// 取消已经注册的与参数匹配的定时器     
void   cancel(PendingIntent operation)    
//注册一个新的延迟定时器  
void   set(int type, long triggerAtTime, PendingIntent operation)    
//注册一个重复类型的定时器  
void   setRepeating(int type, long triggerAtTime, long interval, PendingIntent operation)    
//注册一个非精密的重复类型定时器  
void setInexactRepeating (int type, long triggerAtTime, long interval, PendingIntent operation)  
//设置时区    
void   setTimeZone(String timeZone)  

```
  
**定时器主要类型：**
```

public   static   final   int  ELAPSED_REALTIME    
// 当系统进入睡眠状态时，这种类型的闹铃不会唤醒系统。直到系统下次被唤醒才传递它，该闹铃所用的时间是相对时间，是从系统启动后开始计时的,包括睡眠时 间，可以通过调用SystemClock.elapsedRealtime()获得。系统值是3    (0x00000003)。     
    
public   static   final   int  ELAPSED_REALTIME_WAKEUP    
//能唤醒系统，用法同ELAPSED_REALTIME，系统值是2 (0x00000002) 。     
    
public   static   final   int  RTC    
//当系统进入睡眠状态时，这种类型的闹铃不会唤醒系统。直到系统下次被唤醒才传递它，该闹铃所用的时间是绝对时间，所用时间是UTC时间，可以通过调用 System.currentTimeMillis()获得。系统值是1 (0x00000001) 。     
    
public   static   final   int  RTC_WAKEUP    
//能唤醒系统，用法同RTC类型，系统值为 0 (0x00000000) 。     
    
Public static   final   int  POWER_OFF_WAKEUP    
//能唤醒系统，它是一种关机闹铃，就是说设备在关机状态下也可以唤醒系统，所以我们把它称之为关机闹铃。使用方法同RTC类型，系统值为4(0x00000004)。     

```
**AlarmManager 生命周期：**
repeating AlarmManager一旦启动就会一直在后台运行(除非执行cancel方法)，可以在“应用管理”中看到这个应用状态是正在运行。 “强行停止”可以让Alarmmanager停掉。
尝试了几种任务管理器， 都只能重置计数器（确实释放内存了），但都无法关闭定时器，只有系统自带的“强行停止”奏效。
**如何使用AlarmManager？**
使用AlarmManager共有三种方式， 都是通过` PendingIntent。 `
这边就举一个使用BroadCast的例子。
首先是创建一个BroadCast类，需要继承BroadCastReceiver， 如下：
```

public class ActionBroadCast extends BroadcastReceiver {  
      
    private static int num = 0;  
    /* (non-Javadoc) 
     * @see android.content.BroadcastReceiver#onReceive(android.content.Context, android.content.Intent) 
     */  
    @Override  
    public void onReceive(Context context, Intent intent) {  
        // TODO Auto-generated method stub  
        Log.e("ActionBroadCast", "New Message !" + num++);  
    }  
  
}  

```
下面就让我们启动AlarmManager, 这边就直接在Activity中启动了， 如下：
```

public class AlarmTestActivity extends Activity {  
    /** Called when the activity is first created. */  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
          
        AlarmManager am = (AlarmManager)getSystemService(ALARM_SERVICE);  
          
        PendingIntent pi = PendingIntent.getBroadcast(this, 0, new Intent(this, ActionBroadCast.class), Intent.FLAG_ACTIVITY_NEW_TASK);  
        long now = System.currentTimeMillis();  
        am.setInexactRepeating(AlarmManager.RTC_WAKEUP, now, 3000, pi);  
    }  
}  

```
这边用Repeating的方式。 每隔3秒发一条广播消息过去。RTC_WAKEUP的方式，保证即使手机休眠了，也依然会发广播消息。
最后看一下AndroidManifest文件，主要是注册一下Activity和BroadCast。  （实际使用中最好再加个filter，自己定义一个Action比较好）
##  AlarmManager的取消 
**（其中需要注意的是取消的Intent必须与启动Intent保持绝对一致才能支持取消AlarmManager）**
```


  Intent intent =new Intent(Main.this, alarmreceiver.class);
  intent.setAction("repeating");
  PendingIntent sender=PendingIntent
         .getBroadcast(Main.this, 0, intent, 0);
  AlarmManager alarm=(AlarmManager)getSystemService(ALARM_SERVICE);
  alarm.cancel(sender);

```
参考：
http://blog.csdn.net/feng88724/article/details/6989227
http://www.cnblogs.com/jico/archive/2010/11/03/1868361.html