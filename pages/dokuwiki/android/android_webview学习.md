title: android_webview学习 

#  Android WebView学习 
##  权限: 
` <uses-permission android:name="android.permission.INTERNET" /> `
##  在WebView中使用JavaScript 
''WebView myWebView = (WebView) findViewById(R.id.webview);
WebSettings webSettings = myWebView.getSettings();webSettings.setJavaScriptEnabled(true);''
##  绑定JavaScript代码到Android源码 
```

public class JavaScriptInterface {
Context mContext;
/** Instantiate the interface and set the context */
JavaScriptInterface(Context c) { 
mContext = c; 
}
/** Show a toast from the web page */ 
public void showToast(String toast) { 
Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show(); 
}
}

WebView webView = (WebView) findViewById(R.id.webview);
webView.addJavascriptInterface(new JavaScriptInterface(this), "Android");

```
这段代码会创建一个**名称为“Android”**的JavaScript接口并运行在WebView当中。在这里，你的Web应用会接入到JavaScriptInterface类。例如，下面为Html何JavaScript代码，它会在用户点击按钮的时候使用新的接口创建一个Toast消息。
```

<input type="button" value="Say hello" onClick="Android.showToast('Hello Android!')" />

```
<note>备注：在JavaScript中绑定的对象会运行在另一个线程，它和创建它的线程是不同的线程。</note>
##  处理页面导航 
''WebView myWebView = (WebView) findViewById(R.id.webview);
myWebView.setWebViewClient(new WebViewClient());''
完成，现在用户点击的所有链接都会在WebView中打开。如果你想要在点击链接并载入网页的时候做更多的操作，请创建自己的WebViewClient 并重写java.lang.String) shouldOverrideUrlLoading()方法。例如：
```

private class MyWebViewClient extends WebViewClient { 
@Override 
public boolean shouldOverrideUrlLoading(WebView view, String url) { 
if (Uri.parse(url).getHost().equals("www.example.com")) { 
//这是我的网页，所以不要重写，让我的WebView载入它
return false; 
} 
// 如果不是我的网页，则启动另外的Activity来处理该URL
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url)); 
startActivity(intent); 
return true; 
}
}

```
##  操作网页历史 
当你的WebView重写URL的载入，它会自动累积访问过的网页历史。你可以使用` goBack()方法和goForward() `方法操纵回退和向前功能。 例如，下面是Activity使用返回按钮操作“回退”功能
```

@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
// Check if the key event was the Back button and if there's history
if ((keyCode == KeyEvent.KEYCODE_BACK) && myWebView.canGoBack() { 
myWebView.goBack(); 
return true; 
} 
// If it wasn't the Back key or there's no web page history, bubble up to the default 
// system behavior (probably exit the activity) 
return super.onKeyDown(keyCode, event);
}

``` 
##  fragment中webview的历史操作 
```


webview.setOnKeyListener(new OnKeyListener() {
                        
 @Override
 public boolean onKey(View v, int keyCode, KeyEvent event) {
    if (event.getAction() == KeyEvent.ACTION_DOWN) {    
      if (keyCode == KeyEvent.KEYCODE_BACK && webview.canGoBack()) {  //表示按返回键 时的操作  
          webview.goBack();   //后退    
          return true;    //已处理    
      }    
  }    
  return false;
 }
});

```
##  webview.loaddata乱码解决 
loadData()中的html data中不能包含'#', '%', '\', '?'四中特殊字符，出现这种字符就会出现解析错误，显示找不到网页还有部分html代码。需要如何处理呢？我们需要用UrlEncoder编码为%23, %25, %27, %3f 。
可以使用以下两种代码，data为string类型的html代码
1、webView.loadData(URLEncoder.encode(data, “utf-8”), “text/html”, “utf-8”);这样一些背景效果什么的都不怎么好看了。不推荐。
` 2、webView.loadDataWithBaseURL(null,data, “text/html”, “utf-8”, null); `
这样就会完美解析了。
 编辑
**webview中文乱码解决：**
```

方式一：
//webView.loadData(htmlData, "text/html", "UTF-8");//API提供的标准用法，无法解决乱码问题  
webView.loadData(htmlData, "text/html; charset=UTF-8", null);//这种写法可以正确解码  
方式二
//先调用这句然后再加载数据
webView.getSettings().setDefaultTextEncodingName("UTF-8");//设置默认为utf-8  
webView.loadData(htmlData, "text/html; charset=UTF-8", null);//这种写法可以正确解码  
方式三：推荐
webView.getSettings().setDefaultTextEncodingName("UTF-8");//设置默认为utf-8  
webView.loadDataWithBaseURL(null,data, "text/html",  "utf-8", null);

```
##  WebView不加载图片问题 

**loadData不能加载图片内容**，如果要加载图片内容或者获得更强大的Web支持请使用` loadDataWithBaseURL `。
##  android_webview中内容获取 

在程序中经常会用到webView来显示网页，但如果能够得到网页中的内容呢
```

//定义java的js交互接口，这个类将会被js操纵。
 class WebViewContent{
        @JavascriptInterface
        public void getContent(String data){
            webViewContent=data;

        }
    }
//定义WebView并开启、绑定与JS的交互，以便获取webview中的html内容。
 	 WebSettings settings = webView.getSettings();
        //开启缩放支持
        settings.setSupportZoom(true);
        settings.setBuiltInZoomControls(true);
		//启用JS
        settings.setJavaScriptEnabled(true);
        //默认对缩放比例有限制，导致用户体验不好，所以需要设置为使用任意比例缩放。
        settings.setUseWideViewPort(true);
        //使页面之间可以点击链接导航
        webView.setWebViewClient(new WebViewClient());
        webView.setWebChromeClient(new WebChromeClient());
        //初始页面一般过大，我们设置为75%
//        webView.setInitialScale(75);
	//设置默认为utf-8，否则会出现乱码现象
        webView.getSettings().setDefaultTextEncodingName("UTF -8");
		//使用loadDataWithBaseURL替代loadData以提供更好的体验和防止乱码。
         webView.loadDataWithBaseURL(null, model.getSummary(), "text/html", "UTF-8", null);
    	//将JS接口类WebViewContent进行绑定,绑定到js中的handler对象
        webView.addJavascriptInterface(new WebViewContent(),"handler");
		//通过WebViewClient操作。
        webView.setWebViewClient(new WebViewClient() {
            @Override
            public void onPageFinished(WebView view, String url) {
//                页面加载完成获取内容，通过js代码即可获取数据到绑定的js接口对象的对应方法中。
                view.loadUrl("javascript:window.handler.getContent('<html>'+document.getElementsByTagName('html')[0].innerHTML+'</html>');");
                super.onPageFinished(view, url);
            }
        });

```
##  Android 中Webview 自适应屏幕 
###  方式一：在HTML页面中使用Viewport Metadata 

```

<head>
 <title>Example</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
</head>

```
上文只是一个只有两个viewport属性例子，下文描述了所有支持的viewport属性及其允许的值类型。
```

<meta name="viewport"
      content="
          height = [pixel_value | device-height] ,
          width = [pixel_value | device-width ] ,
          initial-scale = float_value ,
          minimum-scale = float_value ,
          maximum-scale = float_value ,
          user-scalable = [yes | no] ,
          target-densitydpi = [dpi_value | device-dpi |
                               high-dpi | medium-dpi | low-dpi]
          " />

```
####  自动调整尺寸 
` <meta name="viewport" content="width=device-width" /> `
####  预定义viewport缩放 
viewport的缩放值定义了页面所允许的缩放范围。viewport属性允许您通过以下方式指定您的页面缩放：
**initial-scale**
页面的初始缩放。这个值是一个指示您的页面相对于屏幕大小倍数的float值。例如，如果您设置初始缩放为“1.0”那么页面将根据目标密度1比1的显示。如果设置为“2.0”那么页面将放大2倍。
为了将网页与viewport尺寸匹配，默认初始缩放是计算过的。因为默认viewport 是800像素宽，如果设备屏幕判断为小于800像素宽，初始缩放将以小于1.0的某个值为默认值，以便在屏幕上匹配一个800像素宽的页面。
**minimum-scale**
允许的最小缩放。这个值是一个指示您的页面相对于屏幕大小最小倍数的float值。例如，如果您设置为1.0，那么因为最小大小和目标密度是1:1的，页面就不能缩小。
**maximum-scale**
允许的最大缩放。这个值是一个指示您的页面相对于屏幕大小最大倍数的float值。例如您设置此值为2.0，那么您就不能放大超过2倍。
**user-scalable**
是否允许用户缩放此页面。设置为yes允许缩放，no为不允许缩放。默认值是yes。如果您设置此值为no，那么minimum-scale和maximum-scale将会被忽略，因为缩放不可用。
所以的缩放值必须在0.01到10之间。
` <meta name="viewport" content="initial-scale=1.0" /> `
这个metadata设置初始缩放为相对于viewport目标密度的原始大小。
####  预定义viewport匹配密度 
设备屏幕的密度基于屏幕决定，由每英寸的像素数（dpi）定义。有三种Android支持的密度：高（hdpi）、中（mdpi）和低（ldpi）
您可以通过使用viewport属性target-densitydpi来为您的网页改变目标密度。它允许以下的值：
device-dpi - 使用设备本地的dpi作为目标dpi。默认缩放将不会起作用。
high-dpi - 使用hdpi作为目标dpi。 中和低密度的屏幕将会适当缩小。.
medium-dpi - 使用mdpi作为目标dpi。 高和低密度的屏幕将会分别放大和缩小。这是默认的密度值。
low-dpi - 使用ldpi作为目标dpi。 中和高密度的屏幕将会适当放大。
<value> - 指定一个dpi值作为目标dpi。该值必须在70-400之间。
例如，您可以通过设置viewport的target-densitydpi属性，来阻止Android浏览器和WebView为不同的分辨率缩放网页。而这个页面会以当匹配前屏幕的密度的大小显示。在本例中，您同样应该定义viewport的宽以匹配设备宽度，这样您的页面才会自然适合屏幕大小。例如：
` <meta name="viewport" content="target-densitydpi=device-dpi, width=device-width" /> `

###  方式二:用JavaScript适配设备密度 
Android浏览器和WebView支持允许您请求当前设备的屏幕密度的DOM属性 - DOM属性 window.devicePixelRatio。这个属性的值 指定当前设备使用的比例系数。例如，如果window.devicePixelRatio的值是"1.0"，那么这个设备被认为是中等密度并且默认情况下不缩放；如果这个值是"1.5"，那么这个设备被认为是高密度并且默认情况下放大1.5x；如果这个值是"0.75"，那么这个设备被认为是低密度并且默认情况下缩小0.75x。**默认匹配的是中等密度，但是您可以改变这个密度匹配来改变不同屏幕密度下您网页的缩放。**
```

if (window.devicePixelRatio == 1.5) {
  alert("This is a high-density screen");
} else if (window.devicePixelRatio == 0.75) {
  alert("This is a low-density screen");
}

```
 
###  方式三：使用代码方式 

本人测试还是存在问题
关键代码:
settings.setUseWideViewPort(true); 
settings.setLoadWithOverviewMode(true); 
```

    WebSettings settings = webView.getSettings();
        
        settings.setSupportZoom(true);//开启缩放支持
        settings.setBuiltInZoomControls(true);//开启缩放支持
        settings.setJavaScriptEnabled(true);
        settings.setDisplayZoomControls(false); //隐藏webview缩放按钮
        //默认对缩放比例有限制，导致用户体验不好，所以需要设置为使用任意比例缩放。
        settings.setUseWideViewPort(true);
        //设置webView自适应手机屏幕
        settings.setLoadWithOverviewMode(true);
        settings.setDefaultTextEncodingName("UTF -8");//设置默认为utf-8
        //使页面之间可以点击链接导航
        webView.setWebViewClient(new WebViewClient());
        webView.setWebChromeClient(new WebChromeClient());

```
参考：http://www.cnblogs.com/bluestorm/archive/2013/04/15/3021996.html
###  方式四：综合方式-强烈推荐 
参考：http://www.cppblog.com/WhiteDummy/archive/2013/06/25/201300.aspx
如果想要得到正常的倍率，是需要配合网页端的。（这里仅讨论html5的场合，跨平台嘛）
个人认为，由网页方面写死一个宽，再提供一个js的缩放函数（包括图片，字体），根据不同设备的分辨率来调用，是比较理想的。（当然，也可以用穷举法，一个分辨率进一个网页，用不同css和不同大小资源 =_=!）
假设宽定位1280，则html5方面必须有：
`  <meta name="viewport" content="width=1280, initial-scale=1.0,maximum-scale=2.0, minimum-scale=0.5, user-scalable=no,target-densitydpi=device-dpi" /> `
其中，**target-densitydpi是最重要的**，它将配合android端的以下代码使用。
```

  //use html5 viewport attribute
 settings.setLoadWithOverviewMode(true);
 settings.setUseWideViewPort(true);

```
表示我们的代码支持html5网页自适应。所谓杀什么畜生用什么刀，网页的事情，dpi适应什么的，就交给html5去做好了 = =，不用我们在更外面一层蛋疼。
这样做之后，1280宽的图片无论在什么设备的分辨率都是正常的尺寸，不会被做倍数不明的拉伸，方便我们控制。

最后我采取的策略:
` <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes,target-densitydpi=device-dpi"> `
```

  //use html5 viewport attribute
 settings.setLoadWithOverviewMode(true);
 settings.setUseWideViewPort(true);

```
##  实例：自己开发简易脱机浏览器 

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
参考：
http://stackoverflow.com/questions/8200945/how-to-get-html-content-from-a-webview
http://veikr.com/201106/android_webview_content-html.html