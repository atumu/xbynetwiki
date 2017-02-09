title: android一键锁屏小程序开发 

#  Android一键锁屏小程序开发

有时候想想要是有个一键锁屏的小程序那就可以解放电源键了。但是市场上的一键锁屏程序要么有很多广告，要么获取设备管理权限后进行流氓行为。所以我们还是自己开发一个一键锁屏小程序。
主要涉及的API: android.app.admin.DeviceAdminReceiver;
   android.app.admin.DevicePolicyManager;

主要参考：Android Developer文档中的APIGuides下的Administor部分。以及Android SDK samples中的ApiDemos下的com\example\android\apis\app\DeviceAdminSample.java

#  二、设备管理介绍 

Android 2.2 通过提供Android设备管理API来向企业级应用提供支持。设备管理API在操作系统级别提供设备管理特性。
可能会用到设备管理API的应用程序：
  - 电子邮件客户端。
  - 远程删除数据的安全应用程序。
  - 设备管理服务和应用程序。
##  设备管理API支持的策略 
file:///E:/api/eoeandroid_wiki/wiki.eoeandroid.com/Device_Policies.html
![](/data/dokuwiki/projectdev/pasted/20150511-021206.png)

除了支持上面列出来的策略外，设备管理API还允许你做下面的事情：
  - 提示用户设置新密码
  - 立即锁屏
  - 擦除数据(也就是，恢复出厂设置)
  - 而本次一键锁屏应用的开发则需要立即锁屏这个策略


#  三、开始开发 

##  AndroidManifest配置 

要使用设备管理API，程序的Manifest文件要包含下面的内容：
DeviceAdminReceiver 的子类包含以下内容.
BIND_DEVICE_ADMIN 权限
响应 DEVICE_ADMIN_ENABLED intent 对象的能力，在Manifest文件中由 intent filter 完成。
在元数据中使用安全策略的声明
```

<activity  
            android:name="net.xby1993.lockscreen.MainActivity"  
            android:label="@string/app_name" >  
            <intent-filter>  
                <action android:name="android.intent.action.MAIN" />  
  
                <category android:name="android.intent.category.LAUNCHER" />  
            </intent-filter>  
        </activity>  
  
        <receiver  
            android:name=".MyAdminReceiver"  
            android:description="@string/app_name"  
            android:label="@string/app_name"  
            android:permission="android.permission.BIND_DEVICE_ADMIN" >  
            <meta-data  
                android:name="android.app.device_admin"  
                android:resource="@xml/lock" />  
  
            <intent-filter>  
                <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />  
            </intent-filter>  
        </receiver>  

```
##  关于策略元数据的说明 

android:resource="@xml/lock"声明了在元数据中使用的安全策略，元数据为设备管理原提供了特定的信息。元数据由DeviceAdminInfo 类来解析。下面是device_admin_sample.xml的内容：元数据中的策略可以选择性定制。在一键锁屏中我们只使用<force-lock/>策略
```

   
<device-admin xmlns:android="http://schemas.android.com/apk/res/android">  
  <uses-policies>  
    <limit-password />  
    <watch-login />  
    <reset-password />  
    <force-lock />  
    <wipe-data />  
    <expire-password />  
    <encrypted-storage />  
    <disable-camera />  
  </uses-policies>  
</device-admin>  

```


下面是我们的lock.xml元数据：
```

<device-admin xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  
  
    <uses-policies>  
  
        <!-- 强制锁屏 -->  
  
        <force-lock />  
    </uses-policies>  
  
</device-admin>  

DeviceAminReceiver开发：
[java] view plaincopy
import android.app.admin.DeviceAdminReceiver;  
  
public class MyAdminReceiver extends DeviceAdminReceiver {  
  
}  

```
这里我们没有必要实现任何回调方法。

##  关于MainActivity的开发（涉及DevicePolicyManager） 

```

import android.os.Bundle;  
import android.app.Activity;  
import android.app.admin.DevicePolicyManager;  
import android.content.ComponentName;  
import android.content.Context;  
import android.content.Intent;  
import android.view.Menu;  
  
public class MainActivity extends Activity {  
  
    private DevicePolicyManager policyManager;  
    private ComponentName componentName;  
    private static final int MY_REQUEST_CODE = 123;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        //获取设备策略管理器系统服务  
        policyManager = (DevicePolicyManager) getSystemService(Context.DEVICE_POLICY_SERVICE);  
        //构建组件封装，用于封装和识别四大组件Activity, Service, BroadcastReceiver, or ContentProvider。  
        componentName = new ComponentName(this, MyAdminReceiver.class);  
          
        // 判断是否有设备管理权限，若有则立即锁屏并结束自己，若没有则获取权限  
        if (policyManager.isAdminActive(componentName)) {  
            policyManager.lockNow();  
            finish();  
        } else {  
            //激活设备管理器申请  
            activeManage();  
        }  
  
        // 这个需要放在最后  
        setContentView(R.layout.activity_main);  
  
    }  
  
    // 获取设备管理器权限的申请。  
    private void activeManage() {  
          
        // 启动设备管理(隐式Intent) - 在AndroidManifest.xml中设定相应过滤器  
        Intent intent = new Intent(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN);  
        // 权限列表  
        intent.putExtra(DevicePolicyManager.EXTRA_DEVICE_ADMIN, componentName);  
  
        // 描述(additional explanation)  
        intent.putExtra(DevicePolicyManager.EXTRA_ADD_EXPLANATION,  
                "请激活设备管理权限才能锁屏。");  
  
        startActivityForResult(intent, MY_REQUEST_CODE);  
  
    }  
  
    @Override  
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {  
        // 获取权限成功，立即锁屏并finish自己，否则继续获取权限  
        if (requestCode == MY_REQUEST_CODE && resultCode == Activity.RESULT_OK) {  
            policyManager.lockNow();  
            finish();  
        } else {  
            activeManage();  
        }  
        super.onActivityResult(requestCode, resultCode, data);  
    }  
  
    @Override  
    public boolean onCreateOptionsMenu(Menu menu) {  
        // Inflate the menu; this adds items to the action bar if it is present.  
        getMenuInflater().inflate(R.menu.main, menu);  
        return true;  
    }  
  
}  

```

##  关于一键锁屏会闪白屏的解决办法 

开发完成后一键锁屏会闪白屏。也就是会闪现Activity界面。
我们可以通过设置主题为透明解决该问题：
values/styles.xml主题文件如下：
```

<resources>  
    <style name="AppBaseTheme" parent="android:Theme.Light">  
        <!--  
            Theme customizations available in newer API levels can go in  
            res/values-vXX/styles.xml, while customizations related to  
            backward-compatibility can go here.  
        -->  
    </style>  
  
    <!-- Application theme. -->  
    <style name="AppTheme" parent="AppBaseTheme">  
  
     <!-- 设置主题为透明，这样我们就不会闪白屏了-->  
        <!-- All customizations that are NOT specific to a particular API-level can go here. -->  
        <item name="android:windowNoTitle">true</item>  
        <item name="android:windowIsTranslucent">true</item>  
    </style>  
  
</resources>  

```

项目源码地址：http://git.oschina.net/xby1993/LockScreen/tree/master