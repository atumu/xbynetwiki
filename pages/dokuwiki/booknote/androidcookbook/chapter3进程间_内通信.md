title: chapter3进程间_内通信 

#  进程间、内通信 
Created Sunday 08 February 2015

##  Android提供了一组独特的应用程序间/内通信机制： 
	意图（Intent）:指定你下一步的意图：调用应用程序内的特定类，或者调用用户配置的任何应用程序，处理特定数据类型的特定请求
	广播接收器（BroadcastReceiver）:与意图过滤器（IntentFilter）结合使用。可以定义能够处理特定类型数据的特定请求（即意图的目标）的应用程序。
	AsyncTask:允许指定在非GUI线程或者主事件线程上不应该执行的长期运行代码，避免ANR错误。
	Handler:允许你对来自后台线程、交由另一个线程处理的消息进行排队，通常能够生成安全地更新屏幕的信息。


##  用Intent打开网页、电话号码、其他内容 
解决方案：Intent+startActivity/startActivityForResult(intent.requestCode)
讨论：要打开网页使用Intent.ACTION_VIEW(android.intent.action.VIEW)
	Intent intent=new Intent(Intent.ACTION_VIEW,Uri.parse(data));
	data包含的是以http:开头的网页URL、tel:开头的电话号码、或者其他内容。
	当时用startActivityForResult关心反馈时我们需要覆盖Activity的onActivityResult(int requestCode,int resultCode,Intent data)
	resultCode一般为Activity.RESULT_OK、Activity.RESULT_CANCELED等。在处理Intent的程序或者Activity结束之前都不会知道结果。


##  从视图发送邮件 
	解决方案：用一个意图将电子邮件数据作为参数发送给邮件应用


###  1.文本邮件 
	讨论：记得修改AndroidManifest.xml允许互联网权限<uses-permission android:name="android.permission.INTERNET" [[/>]]
											<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" [[/>]]
		意图使用：android.content.Intent.ACTION_SEND
	
	关键代码：
						Intent emailIntent = new Intent(android.content.Intent.ACTION_SEND);
						emailIntent.setType("text/html");
						emailIntent.putExtra(android.content.Intent.EXTRA_TITLE, "My Title");
						emailIntent.putExtra(android.content.Intent.EXTRA_SUBJECT, "My Subject");
	
						// Obtain reference to String and pass it to Intent
						emailIntent.putExtra(android.content.Intent.EXTRA_TEXT, getString(R.string.my_text));
						startActivity(emailIntent);


###  2.带有附件的电子邮件： 
	关键代码：除了与文本邮件相同的地方之外还要添加。
			intent.putExtra(Intent.EXTRA_STREAM,Uri.fromFile(new File("/path/to/myfile")));
			intent.setType(text/plain);
			MIME类型可以始终设置为text/plain,但是如果希望更明确的指定如包含jpeg图像应该设置为image/jpeg
	
	== 多个附件： ==
		
		== Intent intent=new Intent(Intent.ACTION_SEND_MULTIPLE); ==
		intent.setType("text/plain");
		intent.putExtra(Intent.EXTRA_TITLE, "My Title");
		intent..putExtra(Intent.EXTRA_SUBJECT, "My Subject");
		intent.putExtra(android.content.Intent.EXTRA_TEXT, getString(R.string.my_text));
		intent.putExtra(Intent.EXTRA_EMAIL,new String[]{recipient_address};
		ArrayList<Uri> uris=new ArrayList<Uri>();
		uris.add(uri.fromFile(new File("/path/to/file1")));
		uris.add(uri.fromFile(new File("/path/to/file2")));
		
		== intent.putParcelableArrayListExtra(Intent.EXTRA_STREAM,uris); ==
		如果发送的是不同类型的文件，则MIME类型必须设置为multipart/mixed
		最后在两种情况下，你都可以用如下代码启动一个新的Activity:
			startActivity(Intent.createChooser(intent,"send email"));
		Intent.createChooser的使用是可选的，但是它使得用户可以选择自己喜欢的应用来发送电子邮件。


##  用Intent.putExtra()推送字符串值 
	解决方案是使用Intent.putExtra()推送数据，然后使用getIntent().getExtras().getString()获取数据。
	也可以使用SharedPreferences代替。


##  从子活动中获取数据到主活动中 
	在主活动中使用startActivityForResult()和onActivityResult(),在子活动中使用setResult()
	主活动的onActivityForResult()在子活动的finish()之后调用。
	子活动的关键代码：
			Intent idata=new Intent();
			idata.putExtra(...);
			
			== setResult(Activity.RESULT_OK,idata); ==
			
			== finish(); ==
			

##  保持服务运行同时显示其他应用 
解决方案：Service类。（android.app.Service）
	
###  service的生命周期： 
![](/data/dokuwiki/booknote/androidcookbook/pasted/20150521-045902.png)
Activity的startService和stopService()
扩展Service类必须实现抽象方法onBind().这个方法在类直接启动时不使用，可以作为一个存根方法。通常至少覆盖onStartCommand()和onUnbind();以开始和结束某些活动。
关于onStartCommand()的不同返回值，如果返回START_STICKY将在服务终止时重启这个服务，如果返回START_NO_STICKY该服务不会被自动重启。
service类需要在AndroidManifest文件中进行声明：<service android:name=""/>


##  发送和接收广播消息 
	解决方案：创建并实例化一个广播接收器，并创建一个IntentFilter然后用接收广播消息的活动注册接收器。registerReceiver(myReceiver,intentFilter);
	android.content.BroadcastReceiver,子类必覆盖onReceiver(Context context,Intent intent);
	发送广播消息则为：sendBroadcast(intent);


##  在设备重启之后启动服务 
解决方案：监听用于自引导事件的意图，在事件发生时启动服务
android.intent.action.BOOT_COMPLETED意图开机完成时发出，故而我们只需注册这个意图的接收器。在AndroidManifest中添加：
	<receiver android:name="">
			<intent-filter>
				<action android:name="android.intent.action.BOOT_COMPLETED"/>
			</intent-filter>
	</receiver>


##  用线程创建响应式应用程序 

##  用AsyncTask进行后台处理 
解决方案：使用android.os.AsyncTask<Params, Progress, Result>和ProgressDialog
AsyncTask must be subclassed to be used. The subclass will override at least one method (doInBackground(Params...)), and most often will override a second one (onPostExecute(Result).)

 private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
	 protected Long doInBackground(URL... urls) {
		 int count = urls.length;
		 long totalSize = 0;
		 for (int i = 0; i < count; i++) {
			 totalSize += Downloader.downloadFile(urls[i]);
			 publishProgress((int) ((i / (float) count) * 100));
			 // Escape early if cancel() is called
			 if (isCancelled()) break;
		 }
		 return totalSize;
	 }

	 protected void onProgressUpdate(Integer... progress) {
		 setProgressPercent(progress[0]);
	 }

	 protected void onPostExecute(Long result) {
		 showDialog("Downloaded " + result + " bytes");
	 }
 }
 
Once created, a task is executed very simply:

 new DownloadFilesTask().execute(url1, url2, url3);

##  用活动线程队列和处理器在线程之间发送消息 
问题：在其他线程中安全地更新GUI
 解决方案：
	* Handler类中的handleMessage()、sendMessage()方法。与Message类结合使用。将事件包装在消息中，通过消息队列发送这些消息。
			* Handler类、View类的post(),postDelay()
			* Activity.runOnUiThread
			* AsyncTask类

##  创建Android Epoch HTML/JAVASCript日历 
webView使用部分：
		// Get access to the WebView holder
				webview = (WebView) this.findViewById(R.id.webview);
				// Get the settings
				WebSettings settings = webview.getSettings();
				// Enable JavaScript
				settings.setJavaScriptEnabled(true);
				// Enable ZoomControls visibility
				settings.setSupportZoom(true);
			   // Add JavaScript Interface
				webview.addJavascriptInterface(new MyJavaScriptInterface(), "android");        
				// Set the Chrome Client
				webview.setWebChromeClient(new MyWebChromeClient());
				// Load the URL of the HTML file
				webview.loadUrl("file:///android_asset/calendarview.html");
Handler安全地更新UI:
	  jsHandler.post(new Runnable()
								{
									public void run()
										{
											// Java telling JavaScript to do things
											webview.loadUrl("javascript: popup();");
										}
								});




##  技巧 
Activity中onCreate()方法中使用requestWindowFeature(int featrue)参数为Window中的几个final常数
如：FEATUR_INDETERMINATE_PROGRESS;  FEATURE_NO_TITLE155555555555555555`

具体有待进一步研读
