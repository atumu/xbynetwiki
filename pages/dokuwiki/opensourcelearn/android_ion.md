title: android_ion 

#  Android ion异步网络和图像加载大大简化网络开发强烈推荐 
Ion是一个Android异步网络和图像加载库，优雅得API大大简化了网络操作。
地址：https://github.com/koush/ion
##  特点： 

异步下载：
  * Images into ImageViews or Bitmaps (animated GIFs supported too)
  * JSON (via **Gson**)
  * Strings
  * Files
  * Java types using Gson

易于使用地流式API
  * Automatically **cancels** operations when the calling Activity finishes
  * Manages invocation back onto the UI thread
  * All operations return a Future and can be cancelled

HTTP POST/PUT:
  * text/plain
  * application/json - **both JsonObject and POJO**
  * application/x-www-form-urlencoded
  * multipart/form-data

Transparent usage of HTTP features and optimizations:
  * SPDY and HTTP/2
  *** Caching**
  * **Gzip**/Deflate Compression
  * **Connection pooling/reuse** via HTTP Connection: keep-alive
  * Uses the best/stablest connection from a server if it has multiple IP addresses
  * **Cookies**

View **received** **headers**
**Grouping and cancellation of requests
Download progress callbacks**
**Supports file:/, http(s):/, and content:/ URIs**
Request level **logging** and profiling
Support for** proxy** servers like Charles Proxy to do request analysis
**Based on NIO and AndroidAsync**
Ability to use self signed** SSL** certificates
示例：https://github.com/koush/ion/tree/master/ion-sample

##  安装： 

jar方式：
  * 本身jar:ion.jar
  * Gson:gson.jar
  * AndroidAsync：https://github.com/koush/AndroidAsync
Maven：
```

<dependency>
   <groupId>com.koushikdutta.ion</groupId>
   <artifactId>ion</artifactId>
   <version>2,</version>
</dependency>

```
Gradle：
```

dependencies {
    compile 'com.koushikdutta.ion:ion:2.+'
}

```
#  使用： 
##  Get JSON 
```

Ion.with(context)
.load("http://example.com/thing.json")
.asJsonObject()
.setCallback(new FutureCallback<JsonObject>() {
   @Override
    public void onCompleted(Exception e, JsonObject result) {
        // do stuff with the result or error
    }
});

```
##  Post JSON and read JSON 
```

JsonObject json = new JsonObject();
json.addProperty("foo", "bar");

Ion.with(context)
.load("http://example.com/post")
.setJsonObjectBody(json)
.asJsonObject()
.setCallback(new FutureCallback<JsonObject>() {
   @Override
    public void onCompleted(Exception e, JsonObject result) {
        // do stuff with the result or error
    }
});

```
##  Post application/x-www-form-urlencoded and read a String 
```

Ion.with(getContext())
.load("https://koush.clockworkmod.com/test/echo")
.setBodyParameter("goop", "noop")
.setBodyParameter("foo", "bar")
.asString()
.setCallback(...)

```
##  Post multipart/form-data and read JSON with an upload progress bar 
```

Ion.with(getContext())
.load("https://koush.clockworkmod.com/test/echo")
.uploadProgressBar(uploadProgressBar)
.setMultipartParameter("goop", "noop")
.setMultipartFile("filename.zip", new File("/sdcard/filename.zip"))
.asJsonObject()
.setCallback(...)

```
##  Download a File with a progress bar 
```

Ion.with(context)
.load("http://example.com/really-big-file.zip")
// have a ProgressBar get updated automatically with the percent
.progressBar(progressBar)
// and a ProgressDialog
.progressDialog(progressDialog)
// can also use a custom callback
.progress(new ProgressCallback() {@Override
   public void onProgress(int downloaded, int total) {
       System.out.println("" + downloaded + " / " + total);
   }
})
.write(new File("/sdcard/really-big-file.zip"))
.setCallback(new FutureCallback<File>() {
   @Override
    public void onCompleted(Exception e, File file) {
        // download done...
        // do stuff with the File or error
    }
});

```
##  Setting Headers 
```

Ion.with(context)
.load("http://example.com/test.txt")
// set the header
.setHeader("foo", "bar")
.asString()
.setCallback(...)

```
  
##  Load an image into an ImageView 
```

// This is the "long" way to do build an ImageView request... it allows you to set headers, etc.
Ion.with(context)
.load("http://example.com/image.png")
.withBitmap()
.placeholder(R.drawable.placeholder_image)
.error(R.drawable.error_image)
.animateLoad(spinAnimation)
.animateIn(fadeInAnimation)
.intoImageView(imageView);

// but for brevity, use the ImageView specific builder...
Ion.with(imageView)
.placeholder(R.drawable.placeholder_image)
.error(R.drawable.error_image)
.animateLoad(spinAnimation)
.animateIn(fadeInAnimation)
.load("http://example.com/image.png");

```
##  Ion图像加载API特点 
  * **Disk and memory caching**
  * Bitmaps are held via **weak references** so memory is managed very effeciently
  * ListView Adapter **recycling** support
  * Bitmap **transformations** via the .transform(Transform)
  * **Animate** loading and loaded ImageView **states**
  * **DeepZoom** for extremely large images

##  Futures 
All operations return a custom Future that allows you to specify a **callback** that runs on completion.
```

Future<String> string = Ion.with(context)
.load("http://example.com/string.txt")
.asString();

Future<JsonObject> json = Ion.with(context)
.load("http://example.com/json.json")
.asJsonObject();

Future<File> file = Ion.with(context)
.load("http://example.com/file.zip")
.write(new File("/sdcard/file.zip"));

Future<Bitmap> bitmap = Ion.with(context)
.load("http://example.com/image.png")
.intoImageView(imageView);

```
##  Cancelling Requests 
Futures can be cancelled by calling .cancel():
```

bitmap.cancel();
json.cancel();

```
##  Blocking on Requests 
All Futures have a **Future.get()** method that waits for the result of the request, by blocking if necessary.
```

JsonObject json = Ion.with(context)
.load("http://example.com/thing.json").asJsonObject().get();

```
##  Seamlessly use your own Java classes with Gson（利用Gson无缝使用POJO）: 
```

public static class Tweet {
    public String id;
    public String text;
    public String photo;
}

public void getTweets() throws Exception {
    Ion.with(context)
    .load("http://example.com/api/tweets")
    .as(new TypeToken<List<Tweet>>(){})
    .setCallback(new FutureCallback<List<Tweet>>() {
       @Override
        public void onCompleted(Exception e, List<Tweet> tweets) {
          // chirp chirp
        }
    });
}

```
##  Logging 
Wondering why your app is slow? Ion lets you do **both global and request level logging**.
To enable it globally:
```

Ion.getDefault(getContext()).configure().setLogging("MyLogs", Log.DEBUG);

```
Or to enable it on just a single request:
```

Ion.with(context)
.load("http://example.com/thing.json")
.setLogging("MyLogs", Log.DEBUG)
.asJsonObject();

```
##  Request Groups请求组合 
**By default, Ion automatically places all requests into a group with all the other requests created by that Activity or Service**. Using the** cancelAll(Activity)** call, all requests still pending can be easily **cancelled**:
```

Future<JsonObject> json1 = Ion.with(activity, "http://example.com/test.json").asJsonObject();
Future<JsonObject> json2 = Ion.with(activity, "http://example.com/test2.json").asJsonObject();

// later... in activity.onStop
@Override
protected void onStop() {
    super.onStop();
    Ion.getDefault(activity).cancelAll(activity);
}

```
Ion also lets you tag your requests into** groups** to allow for easy cancellation of requests in that group later:
自定义Group便于管理：
```

Object jsonGroup = new Object();
Object imageGroup = new Object();

Future<JsonObject> json1 = Ion.with(activity)
.load("http://example.com/test.json")
// tag in a custom group
.group(jsonGroup)
.asJsonObject();

Future<JsonObject> json2 = Ion.with(activity)
.load("http://example.com/test2.json")
// use the same custom group as the other json request
.group(jsonGroup)
.asJsonObject();

Future<Bitmap> image1 = Ion.with(activity)
.load("http://example.com/test.png")
// for this image request, use a different group for images
.group(imageGroup)
.intoImageView(imageView1);

Future<Bitmap> image2 = Ion.with(activity)
.load("http://example.com/test2.png")
// same imageGroup as before
.group(imageGroup)
.intoImageView(imageView2);

// later... to cancel only image downloads:
Ion.getDefault(activity).cancelAll(imageGroup);

```
##  Proxy Servers (like Charles Proxy) 
```

// proxy all requests
Ion.getDefault(context).configure().proxy("mycomputer", 8888);

// or... to proxy specific requests
Ion.with(context)
.load("http://example.com/proxied.html")
.proxy("mycomputer", 8888)
.getString();

```
##  Viewing Received Headers 
Ion operations return a **ResponseFuture**, which grant **access to response properties** via the Response object.** The Response object contains the headers, as well as the result:**` ResponseFuture同时包含头部和请求结果 `
```

Ion.with(getContext())
.load("http://example.com/test.txt")
.asString()
.withResponse()
.setCallback(new FutureCallback<Response<String>>() {
    @Override
    public void onCompleted(Exception e, Response<String> result) {
        // print the response code, ie, 200
        System.out.println(result.getHeaders().code());
        // print the String that was downloaded
        System.out.println(result.getResult());
    }
});

```