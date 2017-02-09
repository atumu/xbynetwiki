title: chapter13系统与设备控制 


#   系统与设备控制 
Created Monday 11 January 2010

##  访问电话网络/连接性信息 
问题：关于网络连接性的信息
解决：ConnectivityManager和NetworkInfo对象确定手机是否连接到网络，连接的类型，手机是否处于漫游区域。
```

ConnectivityManager cm=(ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);
NetworkInfo ni=cm.getActiveNetworkInfo();
boolean available=ni.isAvailable();
boolean connected=ni.isConnected();
boolean roaming=ni.isRoaming();
//获取网络类型，目前是移动网络或者wifi
int networkType=ni.getType();

```
##  从清单文件获取信息（如版本号） 
解决：android.content.pm.PackageManager与PackageInfo
PackageManager pm=getPackageManager();
PackageInfo info=pm.getPackageInfo(this.getPackageName(),0);
String versionName=info.versionName;

##  将来电通知改为静音、振动或者普通 
解决：使用Android的AudioManager系统服务。
AudioManager am=(AudioManager)getSystemService(Context.AUDIO_SERVICE);;
am.setRingerMode(AudioManager.RINGER_MODE_SILENT);

##  复制文本以及从剪贴板获取文本 
解决：ClipboardManager.setText(),getText()

##  使用基于LED的通知 
解决：NotificationManager及Notification

##  使设备振动 
android.permission.VIBRATE
notification.vibrate=new long[]{1000,1000,1000,1000,1000};

##  从应用程序运行Shell命令 
解决：使用Runtime类的exec()方法返回Process对象。然后调用Process.waitFor()等待命令结束执行。
```

Process p=Runtime.getRuntime().exec("/systemt/bin/ps");
InputStreamReader reader=new InputStreamReader(p.getInputStream());
....

```
##  确定指定应用程序是否运行 
解决：ActivityManager以及RunningAppProcessInfo
```

ActivityManager am=getSytemService(Context.ACTIVITY_SERVICE);
List<RunningAppProcessInfo> procInfos=am.getRunningAppProcesses();
for(int i=0;i<procInfos.size();i++){
	if(procInfos.get(i).**processName.**equals(appPackageName)){
	...
	}
}

```

