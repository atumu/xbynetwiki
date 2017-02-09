title: android获取应用信息等实现思路 

#  Android获取应用信息判断网络连接返回桌面及卸载apk结束进程等的实现思路 

#  获取应用信息 

##  获取应用名 

```

/** 
     * 根据应用包名获取应用名 
     * @param context 
     * @param appPackageName 
     * @return 返回应用名，不存在返回null 
     */  
    public static String getAppName(Context context,String appPackageName){  
        PackageManager pm = context.getPackageManager();  
        String appName=null;  
        try {  
            appName = pm.getApplicationInfo(appPackageName, 0).loadLabel(pm)  
            .toString();  
            return appName;  
        } catch (NameNotFoundException e) {//不存在  
            return appName;//返回null  
        }  
    }  

```

##  获取版本号 

```

/** 
     * 根据包名获取版本号 
     * @param appName 
     * @return 
     * @throws NameNotFoundException  
     */  
    private String getVersionName(String packageName) throws NameNotFoundException {  
        // TODO Auto-generated method stub  
        PackageManager pm=getPackageManager();  
        int flags=0;  
        PackageInfo  packageInfo =pm.getPackageInfo(packageName, flags);  
        String versionName=packageInfo.versionName;//版本名  
        int versioncode=packageInfo.versionCode;//版本号  
        return versionName;  
    }  

```
##  获取应用权限 

```

/** 
     * 根据包名获取应用所有权限 
     * @param context 
     * @param packageName 
     * @return 返回权限字符串数组 
     * @throws NameNotFoundException 
     */  
    public static String[] getAppPermissions(Context context, String packageName) throws NameNotFoundException {  
        return context.getPackageManager().getPackageInfo(  
                    packageName,PackageManager.GET_PERMISSIONS).requestedPermissions;  
    }  

```
#  返回桌面 

```

/** 
* 返回到桌面 
*  
* @param context 
* @return 
*/  
public static void returnDesktop(Context context) { //  
Intent intent = new Intent(Intent.ACTION_MAIN);  
intent.addCategory(Intent.CATEGORY_HOME);  
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);  
context.startActivity(intent);  
}  

```
#  判断网络是否可用，以及是3G还是wifi 
```

/** 
     * 判断网络是否可用 
     *  
     * @param context 
     * @return true可用,false不可用 
     */  
    public static boolean checkTheNetworkConnection(Context context) { //  
        ConnectivityManager connectivityManager = (ConnectivityManager) context  
                .getSystemService(Context.CONNECTIVITY_SERVICE);  
        NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();  
        if (networkInfo != null && networkInfo.isConnected()) {  
            return true;  
        } else {  
            return false;  
        }  
    }  

/** 
     * 判断手机当前上网用的是sim卡还是wifi <!-- 获取网络信息状态，如当前的网络连接是否有效 --> <uses-permission 
     * android:name="android.permission.ACCESS_NETWORK_STATE"/> 
     *  
     * @param context 
     *            上下文 
     * @return 返回true是网络类型是wifi网络，返回false网络类型是sim卡网络 
     */  
    public static boolean checkSIMorWifi(Context context) {  
        ConnectivityManager connectivityManager = (ConnectivityManager) context  
                .getSystemService(Context.CONNECTIVITY_SERVICE);  
        NetworkInfo[] networkInfos = connectivityManager.getAllNetworkInfo();  
        boolean isWifiConnect = false;  
        for (int i = 0; i < networkInfos.length; i++) {  
            if (networkInfos[i].getState() == NetworkInfo.State.CONNECTED) {  
                isWifiConnect = false;  
            }  
            if (networkInfos[i].getType() == connectivityManager.TYPE_WIFI) {  
                isWifiConnect = true;  
            }  
        }  
        return isWifiConnect;  
    }  

```

#  卸载程序 
```

/* 调用系统的卸载程序卸载apk */  
    public void uninstallApk(String packageName) {  
        Uri uri = Uri.parse("package:" + packageName);  
        Intent intent = new Intent(Intent.ACTION_DELETE, uri);  
        int requestCode=0x11;//卸载标记  
        startActivityForResult(intent, requestCode);  
    }  

```
#  结束组件 
```

/* 
 * 退出应用总结： finish()：结束当前Activity，不会立即释放内存。遵循android内存管理机制。 
 * exit()：结束当前组件如Activity，并立即释放当前Activity所占资源。 
 * killProcess()：结束当前组件如Activity，并立即释放当前Activity所占资源。 
 * restartPackage()：结束整个App，包括service等其它Activity组件。 
 */  
/** 
 * killProcess()：结束当前组件如Activity，并立即释放当前Activity所占资源。(可用) 
 */  
public static void killProcess() {  
    android.os.Process.killProcess(android.os.Process.myPid());  
}  

```