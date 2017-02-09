title: chapter9电话_联网应用程序 

# 电话应用程序 
Created Sunday 10 January 2010

##  在电话响铃时进行某些操作 
实现一个广播接收器，然后监听TelephonyManager.ACTION_PHONE_STATE_CHANGE
manifest文件中注册
```

<receiver android:name="">
	<intent-filter>
		<action android:name="android.intent.action.PHONE_STATE"/>
	</intent-filter>
</receiver>
添加相关权限
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>

```
代码中获取状态：
```

string state=intent.getStringExtra(TelephonyManager.EXTRA_STATE);
if(TelephonyManager.EXTRA_STATE_RINGING.equals(state)){
	string incomingNumber=intent.getStringExtra(TelephonyManager.EXTRA_INCOMING_NUMBER);
}

```
<wrap em>注意：当有多个接收器注册了的时候，系统不会有调用顺序。所以如果需要调用优先权，需要使用有序广播的接收器。
如果BroadcastReceiver在10秒内没有结束，Android框架就会显示ANR对话框。所以如果需要进行超过10s的处理，应该实现一个Service并调用服务方法。
不建议从BroadcastReceiver启动一个Activity，因为它将产生新的屏幕，导致用户从当前运行的应用程序转移注意力。如果你的应用程序在响应意图广播时需要想用户显示信息，</wrap>
####  应用用通知管理器来完成。 

##  处理呼出电话 
问题：你希望阻止某些呼叫，或者修改呼叫电话号码
监听Intent.ACTION_NEW_OUTGOING_CALL广播操作。并且将广播接收器的结果数据设置为新的号码。
可以通过Intent.EXTRA_PHONE_NUMBER提取到拨出的号码。
使用setResultData可以设置新的号码替换。
manifest中注册：
```

<receiver android:name="">
	<intent-filter **android:priority="11"**>数值越大优先级越大，负数为系统保留优先级。
		<action android:name="android.intent.action.NEW_OUTGOING_CALL"/>
	</intent-filter>
</receiver>
添加相关权限
<uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>

```
注意：Intent.ACTION_NEW_OUTGOING_CALL是一个有序广播，这种受保护的意图只有系统能够发送。

####  有序广播与常规广播相比有3个附加特征： 
* 可以使用<intent-filter>元素的android:priority属性。
* 可以调用setResultData方法结果传播给下一个接收器。getResultData获取上一个处理结果。
* 可以调用abortBroadcast()方法终止传播

##  电话自动拨号 
解决方案：启动一个拨叫电话的Intent机制。URI为“tel:”+拨叫号码的android.intent.action.DIAL操作创建和启动一个Intent就行。
权限android.permission.CALL_PHONE
string intentStr="tel:"+"55555";
Intent intent=new Intent("android.intent.action.DIAL",Uri.parse(intentStr));
startActivity(intent);

##  发送单部分和多部分SMS消息 
解决方案：SMSManager
SMS消息有限制。为160个字符。多余的要被分段成一个列表。如果只有一部分可以调用sendTextMessage()直接发送。但是如果有多个部分，则必须将列表传给sendMultipartTextMessage().
权限：android:permission.SEND_SMS
```

public class SendSMS {
	static String TAG = "SendSMS";
	SmsManager mSMSManager = null;
	/* The list of message parts our messge
	 * gets broken up into by SmsManger */
	ArrayList<String> mFragmentList = null;
	/* Service Center - not used */
	String mServiceCentreAddr = null;

	SendSMS() {
		**mSMSManager = SmsManager.getDefault();**
	}

	/* Called from the GUI to send one message to one destination */
	public boolean sendSMSMessage(
			String aDestinationAddress,
			String aMessageText) {
		
		if (mSMSManager == null) {
			return (false);
		}

		mFragmentList = mSMSManager.divideMessage(aMessageText);
		int fragmentCount = mFragmentList.size();
		if (fragmentCount > 1) {
			Log.d(TAG, "Sending " + fragmentCount + " parts");
			mSMSManager.sendMultipartTextMessage(aDestinationAddress, 
					mServiceCentreAddr,
					mFragmentList, null, null);
		} else {
			Log.d(TAG, "Sendine one part");
			mSMSManager.sendTextMessage(aDestinationAddress, 
					mServiceCentreAddr,
					aMessageText, null, null);
		}

		return true;
	}
}

```
##  在Android应用程序中接收SMS消息 
使用广播接收器监听SMS消息，然后提取信息
权限:android.permission.RECEIVE_SMS
注册接收器：
```

<receiver android:name="" android:enabled="true">
	<intent-filter>
		<action android:name="android.provider.Telephony.SMS_RECEIVED"/>
		<category android:name="android.intent.category.DEFAULT"/>
	</intent-filter>
</receiver>

Bundle bundle=intent.getExtras();
SmsMessage[] msgs=null;
String message="";
if(bundle!=null){
	Object[] pdus=(Object[])bundle.get("**pdus**");
	msgs=new SmsMessage[pdus.length];
	for(int i=0;i<msgs.length;i++){
	msgs[i]=SmsMessage.createFromPdu((byte[])pdus[i]);
	message=msgs[i].getMessageBody();
	
	}
}

```
##  使用模拟器控制面板向模拟器发送SMS消息 

##  使用Android的TelephonyManager获得设备信息 
问题：你希望获取设备上的网络相关信息和电话信息
解决：使用TelephonyManager
TelephonyManager还提供关于Android电话系统的信息。它能帮助收集不同的信息，如位置，IMEI号码，网络提供商等。
具体有待研究

#  联网应用程序 
Created Sunday 10 January 2010

##  必须具有的权限： 

###  android.permission.INTERNET 

##  导论 
Web服务主要有两种风格：XML/SOAP和REST风格。

##  使用REST风格的Web服务 
解决：URL和URLConnection或者Android自带的Apache HttpClient程序库
使用URL和URLConnection:
```

URL url=new URL("http",host,port,path);
URLConnection conn=url.openConnection();
//这里采用GET操作，要采用POST,可添加conn.setDoOutput(true)
conn.setDoInput(true)
conn.setAllowUserInteration(true);
conn.connect();
//要使用POST,conn.getOutputStream
StringBuilder sb-=new StringBuilder();
BufferedReader in=new BufferedReader(new InputStreamReader(conn.getInputStream()));
....
使用HttpClient
HttpHost target=new HttpHost(host,port);
HttpClient client=new DefaultHttpClient();
HttpGet get=new HttpGet(path);
HttpEntity results=null;
try{
	HttpResponse respon=client.execute(target,get);
	results=respon.getEntity();
	EntityUtils.toString(results);
}

```
##  使用正则表达式从无结构文本中提取信息 
解决：使用java.net下载HTML页面，并使用正则表达式提取页面中的信息。

##  使用ROME解析RSS/Atom Feed 
问题：你希望解析RSS/Atom Feed,这种格式常用于提供网站新闻的更新列表。
解决：` ROME是一个基于java的RSS聚合feed解析器。 `
需要库：rome,jar；jdom.jar

##  用MD5加密明文 

##  将文本转换为超链接 
解决：TextView的autoLink属性 android:autoLink="all"

##  用WebView访问网页 
解决:在布局中嵌入标准WebView组件，并调用其loadUrl()方法加载及显示网页

##  自定义WebView 
```

 webView = (WebView)findViewById(R.id.webView);
WebSettings settings= webView.getSettings();
//开启缩放支持
settings.setSupportZoom(true);
settings.setBuiltInZoomControls(true);
settings.setJavaScriptEnabled(true);
//默认对缩放比例有限制，导致用户体验不好，所以需要设置为使用任意比例缩放。
settings.setUseWideViewPort(true);
//使页面之间可以点击链接导航
webView.setWebViewClient(new WebViewClient());
webView.setWebChromeClient(new WebChromeClient());
//初始页面一般过大，我们设置为75%
webView.setInitialScale(75);
使用WebSettings类访问内建函数，自定义浏览器
WebSettings webSet=webView.getSettings();

```
WebSettings的一些功能：
* 通知WebView阻止网络图片：webSet.setBlockNetworkImage(true);
* 设置浏览器默认字体大小:webSet.setDefaultFontSize(24);
* 设置WebView的缩放支持：webSet.setSupportZoom(true);
* 通知WebView启用JavaScript运行：webSet.setJavaScriptEnabled(true);
* 控制WebView是否保存密码：webSet.setSavePassword(false);
* 控制WebView是否保存表单数据:webSet.setSaveFormData(true);
等等。

##  处理页面导航：详情查看：file:///D:/android-sdk-windows/docs/guide/webapps/webview.html 
WebClient
WebView myWebView = (WebView) findViewById(R.id.webview);
myWebView.setWebViewClient(new WebViewClient());
That's it. Now all links the user clicks load in your WebView.

If you want more control over where a clicked link load, create your own WebViewClient that overrides the shouldOverrideUrlLoading() method. For example:
```

private class MyWebViewClient extends WebViewClient {
	@Override
	public boolean shouldOverrideUrlLoading(WebView view, String url) {
		if (Uri.parse(url).getHost().equals("www.example.com")) {
			// This is my web site, so do not override; let my WebView load the page
			return false;
		}
		// Otherwise, the link is not for a page on my site, so launch another Activity that handles URLs
		Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
		startActivity(intent);
		return true;
	}
}

Then create an instance of this new WebViewClient for the WebView:

WebView myWebView = (WebView) findViewById(R.id.webview);
myWebView.setWebViewClient(new MyWebViewClient());

```
##  Navigating web page history 

When your WebView overrides URL loading, it automatically accumulates a history of visited web pages. You can navigate backward and forward through the history with goBack() and goForward().

For example, here's how your Activity can use the device Back button to navigate backward:
```

@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
	// Check if the key event was the Back button and if there's history
	if ((keyCode == KeyEvent.KEYCODE_BACK) && myWebView.canGoBack()) {
		myWebView.goBack();
		return true;
	}
	// If it wasn't the Back key or there's no web page history, bubble up to the default
	// system behavior (probably exit the activity)
	return super.onKeyDown(keyCode, event);
}

```
The canGoBack() method returns true if there is actually web page history for the user to visit. Likewise, you can use canGoForward() to check whether there is a forward history. If you don't perform this check, then once the user reaches the end of the history, goBack() or goForward() does nothing.

##  WebView实现下载功能 
 在做美图欣赏Android应用的时候，其中有涉及到Android应用下载的功能，这个应用本身其实也比较简单，就是通过WebView控制调用相应的WEB页面进行展示。刚开始以为和普通的文件下载实现，只需要一个链接，然后点击就可以实现下载了，可是放到手机上试的时候，点击下载链接一点反应都没有，在普通页面里面点击是好的，且点击其它的普通链接是可以正常工作的。原来是因为WebView默认没有开启文件下载的功能，如果要实现文件下载的功能，需要设置WebView的DownloadListener，通过实现自己的DownloadListener来实现文件的下载。具体操作如下：
```

1、设置WebView的DownloadListener：
webView.setDownloadListener(new MyWebViewDownLoadListener());
2、实现MyWebViewDownLoadListener这个类，具体可以如下这样：    

private class MyWebViewDownLoadListener implements DownloadListener {  
  
@Override  
public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype,  
									long contentLength) {  
			Uri uri = Uri.parse(url);  
			Intent intent = new Intent(Intent.ACTION_VIEW, uri);  
			startActivity(intent);  
		}  
  
	}  

```
这只是调用系统中已经内置的浏览器进行下载，还没有WebView本身进行的文件下载，不过，这也基本上满足我们的应用场景了。

