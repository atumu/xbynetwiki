title: android_httpurl 

# Android 网络操作学习HttpURLConnection与HttpClient及网络框架选择 

谷歌在官方文档已经建议在2.3以及以上版本使用 HttpConnection。具体原因呢，是因为对2.1和2.2版本，HttpURLConnection有那么几个Bug，所以建议用Apache 的HTTP Client；之后的版本，建议用HttpURLConnection。Apache的HTTP Client比较强大，拥有庞大而灵活的API，这个实现很稳定，并且Bug很少。然而，也就是因为太庞大了，以至于很难在保证兼容性的情况下改进它，故 android 开发团队不应该维护该库而是转投更为轻量级的httpurlconnection。

而当我们开发企业级应用的时候，一般都会选择使用已经封装好的http框架。开源的比较流行的有：
  * 1. Apache 的 HttpClient（Android2.3之前使用）
  * 2. Android 简化扩展版 HttpUrlConnection
  * 3. Google 推出的 Volley（在Android2.3之前使用HttpClient，之后使用HttpUrlConnection）
  * 4. Git开源项目Okhttp (使用http+SPDY协议)
  * 5. Android-async-http(不推荐)
  * 6. Retrofit(默认使用Okhttp作为传输层)
  * 7. Android Query
  * 8. Android AsyncTask
**OkHttp、Volley、Retrofit三者对比：**
**Volley的特点：**
  * 1. Volley的优势在于处理小文件的http请求；
  * 2. 在Volley中也是可以使用Okhttp作为传输层；参考：https://plus.google.com/+JakeWharton/posts/eJJxhkTQ4yU
  * 3. Volley在处理高分辨率的图像压缩上有很好的支持；
  * 4. NetworkImageView在GC的使用模式上更加保守，在请求清理上也更加积极，networkimageview仅仅依赖于强大的内存引用，并当一个新请求是来自ImageView或ImageView离开屏幕时 会清理掉所有的请求数据。
  * 5. Volley比Retrofit在内存错误处理上要更好。
**Retrofit的特点：性能最好，处理最快。如果**
  * 1. 使用REST API时非常方便；
  * 2. 传输层默认就使用OkHttp；
  * 3. 支持NIO；
  * 4. 拥有出色的API文档和社区支持
  * 5. 速度上比volley更快；
**OkHttp的特点：**
支持SPDY（请求头压缩、并行请求、强制SSL、服务端推送）；
` 如果你的应用程序中集成了OKHttp，Retrofit默认会使用OKHttp处理其他网络层请求。 `
#  连接网络： 

ConnectivityManager: 响应网络连接状态查询，同时能在网络连接状态发生改变时通知程序。
NetworkInfo:描述给定类型的网络接口状态(无论是移动网络还是Wi-Fi)
添加权限
```

<uses-permission android:name="android.permission.INTERNET" />  
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />  

```
##  检测网络是否连接 
```

public void myClickHandler(View view) {  
    ...  
    ConnectivityManager connMgr = (ConnectivityManager)   
        getSystemService(Context.CONNECTIVITY_SERVICE);  
    NetworkInfo networkInfo = connMgr.getActiveNetworkInfo();  
    if (networkInfo != null && networkInfo.isConnected()) {  
        // 获取数据  
    } else {  
        // 显示错误  
    }  
    ...  
}  

try {  
        URL url = new URL(myurl);  
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();  
        conn.setReadTimeout(10000 /* milliseconds */);  
        conn.setConnectTimeout(15000 /* milliseconds */);  
        conn.setRequestMethod("GET");  
        conn.setDoInput(true);  
        // 开始查询  
        conn.connect();  
        int response = conn.getResponseCode();  
        Log.d(DEBUG_TAG, "The response is: " + response);  
        is = conn.getInputStream();  
   
        // 将InputStream转化为string  
        String contentAsString = readIt(is, len);  
        return contentAsString;  
   
    // 确保当app用完InputStream对象后关闭它。  
    } finally {  
        if (is != null) {  
            is.close();  
        }   

  HttpClient client = new DefaultHttpClient();  
HttpGet getRequest = new HttpGet("http://blog.isming.me");  
try {    
    HttpResponse response = httpClient.execute(getMethod); //发起GET请求    
  
    Log.i(TAG, "resCode = " + response.getStatusLine().getStatusCode()); //获取响应码    
    Log.i(TAG, "result = " + EntityUtils.toString(response.getEntity(), "utf-8"));//获取服务器响应内容    
}  
  
//先将要传的数据放入List    
params = new LinkedList<BasicNameValuePair>();    
params.add(new BasicNameValuePair("param1", "Post方法"));    
params.add(new BasicNameValuePair("param2", "第二个参数"));    
  
try {    
    HttpPost postMethod = new HttpPost(baseUrl);    
    postMethod.setEntity(new UrlEncodedFormEntity(params, "utf-8")); //将参数填入POST Entity中    
  
    HttpResponse response = httpClient.execute(postMethod); //执行POST方法    
    Log.i(TAG, "resCode = " + response.getStatusLine().getStatusCode()); //获取响应码    
    Log.i(TAG, "result = " + EntityUtils.toString(response.getEntity(), "utf-8")); //获取响应内容    
  </code>
##  检测网络连接类型 
<code java>
// 检查网络连接并设置wifiConnected和mobileConnected变量.   
   public void updateConnectedFlags() {  
       ConnectivityManager connMgr = (ConnectivityManager)   
               getSystemService(Context.CONNECTIVITY_SERVICE);  
  
       NetworkInfo activeInfo = connMgr.getActiveNetworkInfo();  
       if (activeInfo != null && activeInfo.isConnected()) {  
           wifiConnected = activeInfo.getType() == ConnectivityManager.TYPE_WIFI;  
           mobileConnected = activeInfo.getType() == ConnectivityManager.TYPE_MOBILE;  
       } else {  
           wifiConnected = false;  
           mobileConnected = false;  
       }    
   }  

```
#  关于HttpURLConnection需要注意的几点： 

HttpURLConnection没有close方法，只有disconnct()方法，所以conn可以reuse
Android4.0之后增加了响应缓存功能，启用它：
```

private void enableHttpResponseCache() {      
    try {      
        long httpCacheSize = 10 * 1024 * 1024; // 10 MiB      
        File httpCacheDir = new File(getCacheDir(), "http");      
        Class.forName("android.net.http.HttpResponseCache")      
            .getMethod("install", File.class, long.class)      
            .invoke(null, httpCacheDir, httpCacheSize);      
    } catch (Exception httpResponseCacheNotAvailable) {      
    }      
}    

```



参考：
http://blog.csdn.net/hguang_zjh/article/details/33743249
http://www.open-open.com/lib/view/open1394607350395.html
http://my.oschina.net/sammy1990/blog/267718
http://yanmingming.sinaapp.com/?p=1276
http://blog.csdn.net/guolin_blog/article/details/12452307