title: android官方文档学习之broadcastreceiver 

#  android官方文档学习之BroadcastReceiver 
` BroadcastReceiver `也就是“广播接收者”的意思，顾名思义，它就是用来接收来自系统和应用中的广播。
在Android系统中，广播体现在方方面面，例如当开机完成后系统会产生一条广播，接收到这条广播就能实现开机启动服务的功能；当网络状态改变时系统会产生一条广播，接收到这条广播就能及时地做出提示和保存数据等操作；当电池电量改变时，系统会产生一条广播，接收到这条广播就能在电量低时告知用户及时保存进度，等等。
Android中的广播机制设计的非常出色,大大减少了开发的工作量和开发周期。而作为应用开发者，就需要数练掌握Android系统提供的一个开发利器，那就是BroadcastReceiver。
要创建自己的BroadcastReceiver对象，我们需要继承` android.content.BroadcastReceiver `，并实现其` onReceive `方法。下面我们就创建一个名为MyReceiver广播接收者：
```

public class MyReceiver extends BroadcastReceiver {  
    private static final String TAG = "MyReceiver";  
    @Override  
    public void onReceive(Context context, Intent intent) {  
        String msg = intent.getStringExtra("msg");  
        Log.i(TAG, msg);  
    }  
}

```  

##  BroadcastReceiver生命周期 
BroadcastReceiver至在onReceive(Context, Intent)被调用时活动，**一旦return，就失效。**所以必须要注意的是：
不要在onReceiver方法中执行异步操作或者开启一个线程或者显示对话框、bind service等。因为一旦onReceive方法return之后，它就会失效。所以无法处理异步的结果，而且操作系统可能在异步操作没有完成之前将其杀死。**一般可以采用发送Notification的形式**。**通常我们在onReceiver的处理时间不能太长几秒钟。**

##  注册与取消注册广播 

在创建完我们的BroadcastReceiver之后，还不能够使它进入工作状态，**我们需要为它注册一个指定的广播地址**。
###  静态注册 

静态注册是在AndroidManifest.xml文件中配置的，我们就来为MyReceiver注册一个广播地址：
```

<receiver android:name=".MyReceiver">  
 <intent-filter>  
     <action android:name="android.intent.action.MY_BROADCAST"/>  
     <category android:name="android.intent.category.DEFAULT" />  
 </intent-filter>  
 </receiver>  

```
配置了以上信息之后，只要是android.intent.action.MY_BROADCAST这个地址的广播，MyReceiver都能够接收的到。
**注意，这种方式的注册是常驻型的，也就是说当应用关闭后，如果有广播信息传来，MyReceiver也会被系统调用而自动运行。**
###  动态注册 

动态注册需要在代码中动态的指定广播地址并注册，通常我们是在Activity或Service注册一个广播，下面我们就来看一下注册的代码
```

MyReceiver receiver = new MyReceiver();  
          
IntentFilter filter = new IntentFilter();  
filter.addAction("android.intent.action.MY_BROADCAST");  
          
registerReceiver(receiver, filter); 

```
` registerReceiver `是` android.content.ContextWrapper `类中的方法，Activity和Service都继承了ContextWrapper，所以可以直接调用。在实际应用中，我们在Activity或Service中注册了一个BroadcastReceiver，当这个Activity或Service被销毁时如果没有解除注册，系统会报一个异常，提示我们是否忘记解除注册了。所以，**记得在特定的地方执行解除注册操作：**
```

@Override  
protected void onDestroy() {  
    super.onDestroy();  
    unregisterReceiver(receiver);  
}  

```
。**注意，这种注册方式与静态注册相反，不是常驻型的，也就是说广播会跟随程序的生命周期。**
##  发送广播 
```

public void send(View view) {  
    Intent intent = new Intent("android.intent.action.MY_BROADCAST");  
    intent.putExtra("msg", "hello receiver.");  
    sendBroadcast(intent);  
}  

```
sendBroadcast也是android.content.ContextWrapper类中的方法，它可以将一个指定地址和参数信息的Intent对象以广播的形式发送出去。
上面的例子只是一个接收者来接收广播，如果有多个接收者都注册了相同的广播地址，又会是什么情况呢，能同时接收到同一条广播吗，相互之间会不会有干扰呢？这就涉及到**普通广播和有序广播**的概念了。
##  普通广播（Normal Broadcast） 
普通广播对于多个接收者来说是**完全异步**的，通常每个接收者都无需等待即可以接收到广播，接收者相互之间不会有影响。**对于这种广播，接收者无法终止广播，即无法阻止其他接收者的接收动作**。
##  有序广播（Ordered Broadcast） 
有序广播比较特殊，它每次只发送到优先级较高的接收者那里，然后由优先级高的接受者再传播到优先级低的接收者那里，**优先级高的接收者有能力调用` abortBroadcast() `终止这个广播。**
###  发送有序广播 
`  sendOrderedBroadcast(intent, "scott.permission.MY_BROADCAST_PERMISSION");  `
注意，使用sendOrderedBroadcast方法发送有序广播时，**需要一个权限参数，如果为null则表示不要求接收者声明指定的权限，**如果不为null，则表示接收者若要接收此广播**，需声明指定权限**。这样做是**从安全角度考虑的**，例如` 系统的短信就是有序广播的形式 `，一个应用可能是具有拦截垃圾短信的功能，当短信到来时它可以先接受到短信广播，必要时终止广播传递，` 这样的软件就必须声明接收短信的权限。 `
所以我们在AndroidMainfest.xml中**定义一个权限**：
```

<permission android:protectionLevel="normal"  
            android:name="scott.permission.MY_BROADCAST_PERMISSION" /> 

```
然后声明**使用了此权限**：
` <uses-permission android:name="scott.permission.MY_BROADCAST_PERMISSION" />  ` 
##  示例 ：开机启动服务
```

public class BootCompleteReceiver extends BroadcastReceiver {  
      
    private static final String TAG = "BootCompleteReceiver";  
      
    @Override  
    public void onReceive(Context context, Intent intent) {  
        Intent service = new Intent(context, MsgPushService.class);  
        context.startService(service);  
        Log.i(TAG, "Boot Complete. Starting MsgPushService...");  
    }  
  
} 



```
```

public class MsgPushService extends Service {  
  
    private static final String TAG = "MsgPushService";  
      
    @Override  
    public void onCreate() {  
        super.onCreate();  
        Log.i(TAG, "onCreate called.");  
    }  
      
    @Override  
    public int onStartCommand(Intent intent, int flags, int startId) {  
        Log.i(TAG, "onStartCommand called.");  
        return super.onStartCommand(intent, flags, startId);  
    }  
  
    @Override  
    public IBinder onBind(Intent arg0) {  
        return null;  
    }  
}  

```
```

<!--处于安全角度，我们必须声明具有监听开机权限 -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />  
<!-- 开机广播接受者 -->  
<receiver android:name=".BootCompleteReceiver">  
    <intent-filter>  
        <!-- 注册开机广播地址-->  
        <action android:name="android.intent.action.BOOT_COMPLETED"/>  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter>  
</receiver>  
<!-- 消息推送服务 -->  
<service android:name=".MsgPushService"/>  

```
我们看到BootCompleteReceiver注册了“android.intent.action.BOOT_COMPLETED”这个开机广播地址，从安全角度考虑，系统要求必须声明接收开机启动广播的权限
##  示例：网络状态变化 
```

public class NetworkStateReceiver extends BroadcastReceiver {  
      
    private static final String TAG = "NetworkStateReceiver";  
      
    @Override  
    public void onReceive(Context context, Intent intent) {  
        Log.i(TAG, "network state changed.");  
        if (!isNetworkAvailable(context)) {  
            Toast.makeText(context, "network disconnected!", 0).show();  
        }  
    }  
      
    /** 
     * 网络是否可用 
     *  
     * @param context 
     * @return 
     */  
    public static boolean isNetworkAvailable(Context context) {  
        ConnectivityManager mgr = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);  
        NetworkInfo[] info = mgr.getAllNetworkInfo();  
        if (info != null) {  
            for (int i = 0; i < info.length; i++) {  
                if (info[i].getState() == NetworkInfo.State.CONNECTED) {  
                    return true;  
                }  
            }  
        }  
        return false;  
    }  
  
}  

```
```

<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>  
<receiver android:name=".NetworkStateReceiver">  
    <intent-filter>  
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter>  
</receiver>  

```
因为在isNetworkAvailable方法中我们使用到了网络状态相关的API，所以需要声明相关的权限才行
##  示例：电量变化 
```

public class BatteryChangedReceiver extends BroadcastReceiver {  
  
    private static final String TAG = "BatteryChangedReceiver";  
      
    @Override  
    public void onReceive(Context context, Intent intent) {  
        int currLevel = intent.getIntExtra(BatteryManager.EXTRA_LEVEL, 0);  //当前电量  
        int total = intent.getIntExtra(BatteryManager.EXTRA_SCALE, 1);      //总电量  
        int percent = currLevel * 100 / total;  
        Log.i(TAG, "battery: " + percent + "%");  
    }  
  
}  

```
```

<receiver android:name=".BatteryChangedReceiver">  
    <intent-filter>  
        <action android:name="android.intent.action.BATTERY_CHANGED"/>  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter>  
</receiver>  

```
当然，有些时候我们是要**立即获取电量**的，而不是等电量变化的广播，比如当阅读软件打开时立即显示出电池电量。我们可以按以下方式获取：` registerReceiver返回的Intent `
```

Intent batteryIntent = getApplicationContext().registerReceiver(null,  
        new IntentFilter(Intent.ACTION_BATTERY_CHANGED));  
int currLevel = batteryIntent.getIntExtra(BatteryManager.EXTRA_LEVEL, 0);  
int total = batteryIntent.getIntExtra(BatteryManager.EXTRA_SCALE, 1);  
int percent = currLevel * 100 / total;  
Log.i("battery", "battery: " + percent + "%");  

```
参考：http://blog.csdn.net/liuhe688/article/details/6955668