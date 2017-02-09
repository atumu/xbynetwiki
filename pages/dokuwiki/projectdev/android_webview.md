title: android_webview 

#  自己开发简易脱机浏览器 

google android sdk离线文档打开的时候特别慢，据说是要从谷歌官网拉取一些东西导致的。脱机浏览可以解决该问题。PC端可以使用firefox。
但是Android端貌似没有支持脱机工作的浏览器。这让我很伤心。决定开发一个简易的脱机浏览器以便在手机端快速查看sdk文档。
设计到的知识点主要为：WebView的初始化以及缩放问题；将应用程序添加到文件打开方式中。
废话不多说：以下为代码部分：
```

package net.xby1993.simpleexplorer;  
  
import android.app.Activity;  
import android.content.Intent;  
import android.net.Uri;  
import android.os.Bundle;  
import android.util.Log;  
import android.view.KeyEvent;  
import android.view.Window;  
import android.view.WindowManager;  
import android.webkit.WebChromeClient;  
import android.webkit.WebSettings;  
import android.webkit.WebView;  
import android.webkit.WebViewClient;  
  
  
public class MainActivity extends Activity {  
    private static final String TAG=MainActivity.class.getSimpleName();  
    private WebView webView;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        //设置全屏无标题栏  
        requestWindowFeature(Window.FEATURE_NO_TITLE);  
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);  
        setContentView(R.layout.activity_main);  
        webView = (WebView)findViewById(R.id.webView);  
        WebSettings settings= webView.getSettings();  
        //开启缩放支持  
        settings.setSupportZoom(true);  
        settings.setBuiltInZoomControls(true);  
        settings.setJavaScriptEnabled(true);  
        //默认对缩放比例有限制，导致用户体验不好，所以需要设置为使用任意比例缩放。  
        settings.setUseWideViewPort(true);  
      	//设置编码，防止乱码
       settings.setDefaultTextEncodingName("UTF-8");
        //使页面之间可以点击链接导航  
        webView.setWebViewClient(new WebViewClient());  
        webView.setWebChromeClient(new WebChromeClient());  
        //初始页面一般过大，我们设置为75%  
        webView.setInitialScale(75);  
        Intent intent=getIntent();  
        //提取文件管理器打开方式传送的文件地址  
        if(intent.getAction().equals(Intent.ACTION_VIEW)){  
            String strUri=intent.getDataString();  
            Log.d(TAG,TAG);  
            Log.d(TAG,strUri);  
            Log.d(TAG,Uri.encode(strUri));  
            //webView.loadUrl(strUri);  为避免乱码别使用它
          webView.loadDataWithBaseURL(strUri,null, "text/html",  "utf-8", null);
        }  
  
    }  
  
    @Override  
    public boolean onKeyDown(int keyCode,KeyEvent event){  
        //确保可以通过返回键浏览历史页面栈  
        if(keyCode==event.KEYCODE_BACK&&webView.canGoBack()){  
            webView.goBack();  
            return true;  
        }  
        return super.onKeyDown(keyCode,event);  
    }  
  
  
}  

```
```

<?xml version="1.0" encoding="utf-8"?>  
<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
    package="net.xby1993.simpleexplorer" >  
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>  
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>  
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>  
    <!-- 删除联网权限的代码 <uses-permission android:name="android.permission.INTERNET"/> -->  
    <application  
        android:allowBackup="true"  
        android:icon="@mipmap/ic_launcher"  
        android:label="@string/app_name"  
        android:theme="@style/AppTheme" >  
        <activity  
            android:name=".MainActivity"  
            android:label="@string/app_name"  
            >  
            <intent-filter>  
                <action android:name="android.intent.action.MAIN" />  
  
                <category android:name="android.intent.category.LAUNCHER" />  
            </intent-filter>  
            <!-- 这里是为了在文件打开方式中添加本应用 -->  
            <intent-filter>  
                <action android:name="android.intent.action.VIEW"/>  
                <category android:name="android.intent.category.DEFAULT"/>  
                <data android:mimeType="text/html"/>  
            </intent-filter>  
        </activity>  
    </application>  
  
</manifest>  

```                  