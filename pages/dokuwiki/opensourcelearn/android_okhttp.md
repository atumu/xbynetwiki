title: android_okhttp 

#  Android 网络框架学习之OKHttp 

okHttp: OKHttp是Android版Http客户端。非常高效，支持SPDY、连接池、GZIP和 HTTP 缓存。默认情况下，OKHttp会自动处理常见的网络问题，像二次连接、SSL的握手问题。如果你的应用程序中集成了OKHttp，Retrofit默认会使用OKHttp处理其他网络层请求。
An HTTP & SPDY client for Android and Java applications
**从Android4.4开始HttpURLConnection的底层实现采用的是okHttp.**
使用要求:对于Android：2.3以上，对于Java:java7以上
两个模块：
okhttp-urlconnection实现.HttpURLConnection API；
okhttp-apache实现Apache HttpClient API.
依赖：okio(https://github.com/square/okio): Okio, which OkHttp uses for fast I/O and resizable buffers.
#  安装： 
```

maven:
<dependency>  
  <groupId>com.squareup.okhttp</groupId>  
  <artifactId>okhttp</artifactId>  
  <version>2.3.0</version>  
</dependency>  
Gradle:
compile 'com.squareup.okhttp:okhttp:2.3.0'  

```

#  GET A URL 

##  同步GET: 
```

private final OkHttpClient client = new OkHttpClient();  
  
public void run() throws Exception {  
  Request request = new Request.Builder()  
      .url("http://publicobject.com/helloworld.txt")  
      .build();  
  
  Response response = client.newCall(request).execute();  
  if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
  Headers responseHeaders = response.headers();  
  for (int i = 0; i < responseHeaders.size(); i++) {  
    System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));  
  }  
  
  System.out.println(response.body().string());  
}  

```
##  异步GET: 

在一个工作线程中下载文件，当响应可读时回调Callback接口。读取响应时会阻塞当前线程。OkHttp现阶段不提供异步api来接收响应体。
```

private final OkHttpClient client = new OkHttpClient();  
  
public void run() throws Exception {  
  Request request = new Request.Builder()  
      .url("http://publicobject.com/helloworld.txt")  
      .build();  
  
  client.newCall(request).enqueue(new Callback() {  
    @Override public void onFailure(Request request, Throwable throwable) {  
      throwable.printStackTrace();  
    }  
  
    @Override public void onResponse(Response response) throws IOException {  
      if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
      Headers responseHeaders = response.headers();  
      for (int i = 0; i < responseHeaders.size(); i++) {  
        System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));  
      }  
  
      System.out.println(response.body().string());  
    }  
  });  
}  

```
##  访问Header: 
```

private final OkHttpClient client = new OkHttpClient();  
  
public void run() throws Exception {  
  Request request = new Request.Builder()  
      .url("https://api.github.com/repos/square/okhttp/issues")  
      .header("User-Agent", "OkHttp Headers.java")  
      .addHeader("Accept", "application/json; q=0.5")  
      .addHeader("Accept", "application/vnd.github.v3+json")  
      .build();  
  
  Response response = client.newCall(request).execute();  
  if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
  System.out.println("Server: " + response.header("Server"));  
  System.out.println("Date: " + response.header("Date"));  
  System.out.println("Vary: " + response.headers("Vary"));  
}  

```
#  POST TO A SERVER 

##  Posting a String: 
```

public static final MediaType jsonReq  
    = MediaType.parse("application/json; charset=utf-8");  
  
OkHttpClient client = new OkHttpClient();  
  
String post(String url, String json) throws IOException {  
  RequestBody body = RequestBody.create(jsonReq, json);  
  Request request = new Request.Builder()  
      .url(url)  
      .post(body)  
      .build();  
  Response response = client.newCall(request).execute();  
  return response.body().string();  
}  

```
##  Posting Streaming: 

```

public static final MediaType MEDIA_TYPE_MARKDOWN  
     = MediaType.parse("text/x-markdown; charset=utf-8");  
  
 private final OkHttpClient client = new OkHttpClient();  
  
 public void run() throws Exception {  
   RequestBody requestBody = new RequestBody() {  
     @Override public MediaType contentType() {  
       return MEDIA_TYPE_MARKDOWN;  
     }  
  
     @Override public void writeTo(BufferedSink sink) throws IOException {  
       sink.writeUtf8("Numbers\n");  
       sink.writeUtf8("-------\n");  
       for (int i = 2; i <= 997; i++) {  
         sink.writeUtf8(String.format(" * %s = %s\n", i, factor(i)));  
       }  
     }  
  
     private String factor(int n) {  
       for (int i = 2; i < n; i++) {  
         int x = n / i;  
         if (x * i == n) return factor(x) + " × " + i;  
       }  
       return Integer.toString(n);  
     }  
   };  
  
   Request request = new Request.Builder()  
       .url("https://api.github.com/markdown/raw")  
       .post(requestBody)  
       .build();  
  
   Response response = client.newCall(request).execute();  
   if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
   System.out.println(response.body().string());  
 }  

```
##  Posting  a File: 

```

public static final MediaType MEDIA_TYPE_MARKDOWN  
     = MediaType.parse("text/x-markdown; charset=utf-8");  
  
 private final OkHttpClient client = new OkHttpClient();  
  
 public void run() throws Exception {  
   File file = new File("README.md");  
  
   Request request = new Request.Builder()  
       .url("https://api.github.com/markdown/raw")  
       .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, file))  
       .build();  
  
   Response response = client.newCall(request).execute();  
   if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
   System.out.println(response.body().string());  
 }  

```
##  Posting from parameters: 

```

private final OkHttpClient client = new OkHttpClient();  
  
  public void run() throws Exception {  
    RequestBody formBody = new FormEncodingBuilder()  
        .add("search", "Jurassic Park")  
        .build();  
    Request request = new Request.Builder()  
        .url("https://en.wikipedia.org/w/index.php")  
        .post(formBody)  
        .build();  
  
    Response response = client.newCall(request).execute();  
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
    System.out.println(response.body().string());  
  }  

```
##  Posting a multipart request: 
```

private static final String IMGUR_CLIENT_ID = "...";  
 private static final MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");  
  
 private final OkHttpClient client = new OkHttpClient();  
  
 public void run() throws Exception {  
   // Use the imgur image upload API as documented at https://api.imgur.com/endpoints/image  
   RequestBody requestBody = new MultipartBuilder()  
       .type(MultipartBuilder.FORM)  
       .addPart(  
           Headers.of("Content-Disposition", "form-data; name=\"title\""),  
           RequestBody.create(null, "Square Logo"))  
       .addPart(  
           Headers.of("Content-Disposition", "form-data; name=\"image\""),  
           RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))  
       .build();  
  
   Request request = new Request.Builder()  
       .header("Authorization", "Client-ID " + IMGUR_CLIENT_ID)  
       .url("https://api.imgur.com/3/image")  
       .post(requestBody)  
       .build();  
  
   Response response = client.newCall(request).execute();  
   if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
   System.out.println(response.body().string());  
 }  

```
##  Posing Json with Gson 

```

private final OkHttpClient client = new OkHttpClient();  
 private final Gson gson = new Gson();  
  
 public void run() throws Exception {  
   Request request = new Request.Builder()  
       .url("https://api.github.com/gists/c2a7c39532239ff261be")  
       .build();  
   Response response = client.newCall(request).execute();  
   if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
   Gist gist = gson.fromJson(response.body().charStream(), Gist.class);  
   for (Map.Entry<String, GistFile> entry : gist.files.entrySet()) {  
     System.out.println(entry.getKey());  
     System.out.println(entry.getValue().content);  
   }  
 }  
  
 static class Gist {  
   Map<String, GistFile> files;  
 }  
  
 static class GistFile {  
   String content;  
 }  

```

#  Response Caching： 

为了缓存响应，你需要一个你可以读写的缓存目录，和缓存大小的限制。这个缓存目录应该是私有的，不信任的程序应不能读取缓存内容。
一个缓存目录同时拥有多个缓存访问是错误的。大多数程序只需要调用一次 new OkHttp() ，在第一次调用时配置好缓存，然后其他地方只需要调用这个实例就可以了。否则两个缓存示例互相干扰，破坏响应缓存，而且有可能会导致程序崩溃。
响应缓存使用HTTP头作为配置。你可以在请求头中添加 Cache-Control: max-stale=3600 ,OkHttp缓存会支持。你的服务通过响应头确定响应缓存多长时间，例如使用 Cache-Control: max-age=9600 。
```

private final OkHttpClient client;  
  
  public CacheResponse(File cacheDirectory) throws Exception {  
    int cacheSize = 10 * 1024 * 1024; // 10 MiB  
    Cache cache = new Cache(cacheDirectory, cacheSize);  
  
    client = new OkHttpClient();  
    client.setCache(cache);  
  }  
  
  public void run() throws Exception {  
    Request request = new Request.Builder()  
        .url("http://publicobject.com/helloworld.txt")  
        .build();  
  
    Response response1 = client.newCall(request).execute();  
    if (!response1.isSuccessful()) throw new IOException("Unexpected code " + response1);  
  
    String response1Body = response1.body().string();  
    System.out.println("Response 1 response:          " + response1);  
    System.out.println("Response 1 cache response:    " + response1.cacheResponse());  
    System.out.println("Response 1 network response:  " + response1.networkResponse());  
  }  

```
#  Canceling a Call 

```

final Call call = client.newCall(request);  
    call.cancel();  

```
#  Timeouts: 
```

private final OkHttpClient client;  
  
  public ConfigureTimeouts() throws Exception {  
    client = new OkHttpClient();  
    client.setConnectTimeout(10, TimeUnit.SECONDS);  
    client.setWriteTimeout(10, TimeUnit.SECONDS);  
    client.setReadTimeout(30, TimeUnit.SECONDS);  
  }  

```
#  Handling Authentication: 

```

private final OkHttpClient client = new OkHttpClient();  
  
  public void run() throws Exception {  
    client.setAuthenticator(new Authenticator() {  
      @Override public Request authenticate(Proxy proxy, Response response) {  
        System.out.println("Authenticating for response: " + response);  
        System.out.println("Challenges: " + response.challenges());  
        String credential = Credentials.basic("jesse", "password1");  
        return response.request().newBuilder()  
            .header("Authorization", credential)  
            .build();  
      }  
  
      @Override public Request authenticateProxy(Proxy proxy, Response response) {  
        return null; // Null indicates no attempt to authenticate.  
      }  
    });  
  
    Request request = new Request.Builder()  
        .url("http://publicobject.com/secrets/hellosecret.txt")  
        .build();  
  
    Response response = client.newCall(request).execute();  
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);  
  
    System.out.println(response.body().string());  
  }  
为避免当验证失败时多次重试，我们可以通过返回null来放弃验证：
 if (responseCount(response) >= 3) {  
    return null; // If we've failed 3 times, give up.  
  }  
//添加以下方法  
private int responseCount(Response response) {  
    int result = 1;  
    while ((response = response.priorResponse()) != null) {  
      result++;  
    }  
    return result;  
  }  

```
#  Interceptors 
```

class LoggingInterceptor implements Interceptor {  
  @Override public Response intercept(Chain chain) throws IOException {  
    Request request = chain.request();  
  
    long t1 = System.nanoTime();  
    logger.info(String.format("Sending request %s on %s%n%s",  
        request.url(), chain.connection(), request.headers()));  
  
    Response response = chain.proceed(request);  
  
    long t2 = System.nanoTime();  
    logger.info(String.format("Received response for %s in %.1fms%n%s",  
        response.request().url(), (t2 - t1) / 1e6d, response.headers()));  
  
    return response;  
  }  
}  

```
![](/data/dokuwiki/opensourcelearn/pasted/20150511-073510.png)

##  Application Interceptors： 
```

OkHttpClient client = new OkHttpClient();  
client.interceptors().add(new LoggingInterceptor());  

```
##  Network Interceptors 

```

OkHttpClient client = new OkHttpClient();  
client.networkInterceptors().add(new LoggingInterceptor()); 

```