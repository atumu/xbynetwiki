title: android官方文档学习之volley 

#  android官方文档学习之Volley网络框架 
建议参考：http://blog.csdn.net/guolin_blog/article/details/17482095  Android Volley完全解析系列文章学习。

Volley 是一个HTTP库，它能够帮助Android apps更方便的执行网络操作，最重要的是，它更快速高效。可以通过开源的 AOSP 仓库获取到Volley 。
Volley 有如下的优点：
  * 自动调度网络请求。
  * 高并发网络连接。
  * 通过标准的HTTP的cache coherence(高速缓存一致性)使得磁盘与内存缓存不可见(Transparent)。
  * 支持指定请求的优先级。
  * 支持取消已经发出的请求。你可以取消单个请求，或者指定取消请求队列中的一个区域。
  * 框架容易被定制，例如，定制重试或者回退功能。
  * 强大的指令(Strong ordering)可以使得**异步加载**网络数据并显示到UI的操作更加简单。
  * 包含了Debugging与tracing工具。

Volley擅长执行用来显示UI的RPC操作， 例如获取搜索结果的数据。它轻松的整合了任何协议，并输出操作结果的数据，可以是raw strings，也可以是images，或者是JSON。Volley可以使得你免去重复编写样板代码，使你可以把关注点放在你的app的功能逻辑上。
**Volley不适合用来下载大的数据文件**。因为Volley会在解析的过程中保留持有所有的响应数据在内存中。对于下载大量的数据操作，请考虑使用DownloadManager。
Volley框架的核心代码是托管在AOSP仓库的frameworks/volley中.
下载Volley代码：
` git clone https://android.googlesource.com/platform/frameworks/volle `
附上Volley源码：![](/data/dokuwiki/android/volley.zip|)
不过已经有人将volley的代码放到github上了：
https://github.com/mcxiaoke/android-volley，你可以使用更加简单的方式来使用volley：
Gradle
```

compile 'com.mcxiaoke.volley:library:1.0.+'

```
Maven
```

<dependency>
    <groupId>com.mcxiaoke.volley</groupId>
    <artifactId>library</artifactId>
    <version>1.0.16</version>
</dependency>

```
以一个Android library project的方式导入下载的源代码到你的项目中。(如果你是使用Eclipse，请参考Managing Projects from Eclipse with ADT)，或者编译成一个.jar文件。
##  发送简单的网络请求 
通过创建一个` RequestQueue `并传递` Request `对象给它。RequestQueue管理用来执行网络操作的工作线程，从Cache中读写数据，并解析Http的响应内容。` Requests `执行raw responses的解析，` Volley会把响应的数据分发给主线程。 `
1、添加权限：` android.permission.INTERNET `
2、Use newRequestQueue
Volley提供了一个简便的方法：` Volley.newRequestQueue `用来为你建立一个RequestQueue，使用默认值，并启动这个队列。例如：
```

// Instantiate the RequestQueue.
RequestQueue queue = Volley.newRequestQueue(this);
String url ="http://www.google.com";

// Request a string response from the provided URL.
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
            new Response.Listener() {
    @Override
    public void onResponse(String response) {
        // Display the first 500 characters of the response string.
        mTextView.setText("Response is: "+ response.substring(0,500));
    }
}, new Response.ErrorListener() {
    @Override
    public void onErrorResponse(VolleyError error) {
        mTextView.setText("That didn't work!");
    }
});
// Add the request to the RequestQueue.
queue.add(stringRequest);

```
` Volley总是将解析后的数据返回至主线程中 `。在主线程中更加合适使用接收到的数据用来操作UI控件，这样你可以在响应的handler中轻松的修改UI，但是对于库提供的一些其他方法是有些特殊的，例如与取消有关的。
##  Send a Request 
为了发送一个请求，你只需要构造一个请求并通过add()方法添加到RequestQueue中。一旦你添加了这个请求，它会通过队列，得到处理，然后得到原始的响应数据并返回。
当你执行add()方法时，Volley触发执行一个缓存处理线程以及一系列网络处理线程。当你添加一个请求到队列中，它将被缓存线程所捕获并触发：如果这个请求可以被缓存处理，那么会在缓存线程中执行响应数据的解析并返回到主线程。如果请求不能被缓存所处理，它会被放到网络队列中。网络线程池中的第一个可用的网络线程会从队列中获取到这个请求并执行HTTP操作，解析工作线程的响应数据，把数据写到缓存中并把解析之后的数据返回到主线程。
` 请注意那些比较耗时的操作，例如I/O与解析parsing/decoding都是执行在工作线程。你可以在任何线程中添加一个请求，但是响应结果都是返回到主线程的。 `
![](/data/dokuwiki/android/pasted/20150608-151322.png)
##  Cancel a Request 
**对请求Request对象调用cancel()方法取消一个请求**。一旦取消，Volley会确保你的响应Handler不会被执行。这意味着在实际操作中你可以在activity的onStop()方法中取消所有pending在队列中的请求。
为了利用这种优势，你应该跟踪所有已经发送的请求，以便在需要的时候可以取消他们。有一个简便的方法：**你可以为每一个请求对象都绑定一个tag对象**。然后你可以使用这个tag来提供取消的范围。例如，**你可以为你的所有请求都绑定到执行的Activity上，然后你可以在onStop()方法执行requestQueue.cancelAll(this) 。**同样的，你可以为ViewPager中的所有请求缩略图Request对象分别打上对应Tab的tag。并在滑动时取消这些请求，用来确保新生成的tab不会被前面tab的请求任务所卡到。
下面一个使用String来打Tag的例子：
1、定义你的tag并添加到你的请求任务中。
```

public static final String TAG = "MyTag";
StringRequest stringRequest; // Assume this exists.
RequestQueue mRequestQueue;  // Assume this exists.

// Set the tag on the request.
stringRequest.setTag(TAG);

// Add the request to the RequestQueue.
mRequestQueue.add(stringRequest);

```
2、在activity的onStop()方法里面，取消所有的包含这个tag的请求任务。
```

@Override
protected void onStop () {
    super.onStop();
    if (mRequestQueue != null) {
        mRequestQueue.cancelAll(TAG);
    }
}

```
##  建立请求队列 
###  Set Up a Network and Cache 
一个` RequestQueue `需要两部分来支持它的工作：**一部分是网络操作，用来传输请求，另外一个是用来处理缓存操作的Cache。**在Volley的工具箱中包含了标准的实现方式：` DiskBasedCache `提供了每个文件与对应响应数据一一映射的缓存实现。 ` BasicNetwork `提供了一个网络传输的实现，连接方式可以是AndroidHttpClient 或者是 HttpURLConnection.
下面的代码片段演示了如何一步步建立一个RequestQueue:
```

RequestQueue mRequestQueue;

// Instantiate the cache
Cache cache = new DiskBasedCache(getCacheDir(), 1024 * 1024); // 1MB cap

// Set up the network to use HttpURLConnection as the HTTP client.
Network network = new BasicNetwork(new HurlStack());

// Instantiate the RequestQueue with the cache and network.
mRequestQueue = new RequestQueue(cache, network);

// Start the queue
mRequestQueue.start();

String url ="http://www.myurl.com";

// Formulate the request and handle the response.
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
        new Response.Listener<String>() {
    @Override
    public void onResponse(String response) {
        // Do something with the response
    }
},
    new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            // Handle error
    }
});

// Add the request to the RequestQueue.
mRequestQueue.add(stringRequest);

```
##  Use a Singleton Pattern 
` 如果你的程序需要持续的使用网络，更加高效的方式应该是建立一个RequestQueue的单例 `，这样它能够持续保持在整个app的生命周期中。你可以通过多种方式来实现这个单例。推荐的方式是实现一个单例类，` 里面封装了RequestQueue对象与其他Volley的方法 `。另外一个方法是继承Application类，并在Application.OnCreate()方法里面建立RequestQueue。但是这个方法是不推荐的。因为一个static的单例能够以一种更加模块化的方式提供同样的功能。
` 一个关键的概念是RequestQueue必须和Application context所关联的。而不是Activity的context。 `**这确保了RequestQueue在你的app生命周期中一直存活，而不会因为activity的重新创建而重新创建RequestQueue。(例如，当用户旋转设备时)。**
下面是一个单例类，提供了RequestQueue与ImageLoader的功能：
```

private static MySingleton mInstance;
    private RequestQueue mRequestQueue;
    private ImageLoader mImageLoader;
    private static Context mCtx;

    private MySingleton(Context context) {
        mCtx = context;
        mRequestQueue = getRequestQueue();

        mImageLoader = new ImageLoader(mRequestQueue,
                new ImageLoader.ImageCache() {
            private final LruCache<String, Bitmap>
                    cache = new LruCache<String, Bitmap>(20);

            @Override
            public Bitmap getBitmap(String url) {
                return cache.get(url);
            }

            @Override
            public void putBitmap(String url, Bitmap bitmap) {
                cache.put(url, bitmap);
            }
        });
    }

    public static synchronized MySingleton getInstance(Context context) {
        if (mInstance == null) {
            mInstance = new MySingleton(context);
        }
        return mInstance;
    }

    public RequestQueue getRequestQueue() {
        if (mRequestQueue == null) {
            // getApplicationContext() is key, it keeps you from leaking the
            // Activity or BroadcastReceiver if someone passes one in.
            mRequestQueue = Volley.newRequestQueue(mCtx.getApplicationContext());
        }
        return mRequestQueue;
    }

    public <T> void addToRequestQueue(Request<T> req) {
        getRequestQueue().add(req);
    }

    public ImageLoader getImageLoader() {
        return mImageLoader;
    }
}

```
##  创建标准的网络请求 
Volley支持的常用请求类型：

  * StringRequest。指定一个URL并在相应回调中接受一个原始的raw string数据。请参考前一课的示例。
  * ImageRequest。指定一个URL并在相应回调中接受一个image。
  * JsonObjectRequest与JsonArrayRequest (均为JsonRequest的子类)。指定一个URL并在相应回调中获取到一个JSON对象或者JSON数组。

###  Request an Image 
Volley为请求图片提供了如下的类。这些类依次有着依赖关系，用来支持在不同的层级进行图片处理：
` ImageRequest ` - 一个封装好的，用来处理URL请求图片并且返回一张decode好的bitmap的类。它同样提供了一些简便的接口方法，例如指定一个大小进行重新裁剪。它的主要好处是Volley会确保类似decode，resize等耗时的操作执行在工作线程中。

` ImageLoader ` - 一个用来处理加载与缓存从网络上获取到的图片的帮助类。ImageLoader是管理协调大量的ImageRequest的类。例如，在ListView中需要显示大量缩略图的时候。ImageLoader为通常的Volley cache提供了更加前瞻的内存缓存，这个缓存对于防止图片抖动非常有用。这还使得能够在避免阻挡或者延迟主线程的前提下在缓存中能够被Hit到。ImageLoader还能够实现响应联合Coalescing，每一个响应回调里面都可以设置bitmap到view上面。联合Coalescing使得能够同时提交多个响应，这提升了性能。

` NetworkImageView ` - 在ImageLoader的基础上建立，**替换ImageView进行使用**。对于需要对ImageView设置网络图片的情况下使用很有效。NetworkImageView同样可以在view被detached的时候取消pending的请求。
```

ImageView mImageView;
String url = "http://i.imgur.com/7spzG.png";
mImageView = (ImageView) findViewById(R.id.myImage);
...

// Retrieves an image specified by the URL, displays it in the UI.
ImageRequest request = new ImageRequest(url,
    new Response.Listener() {
        @Override
        public void onResponse(Bitmap bitmap) {
            mImageView.setImageBitmap(bitmap);
        }
    }, 0, 0, null,
    new Response.ErrorListener() {
        public void onErrorResponse(VolleyError error) {
            mImageView.setImageResource(R.drawable.image_load_error);
        }
    });
// Access the RequestQueue through your singleton class.
MySingleton.getInstance(this).addToRequestQueue(request);

```
你可以使用` ImageLoader与NetworkImageView `用来处理类似ListView等**大量显示图片**的情况。在你的layout XML文件中，你可以` 使用NetworkImageView来替代通常的ImageView， ` 
```

<com.android.volley.toolbox.NetworkImageView
        android:id="@+id/networkImageView"
        android:layout_width="150dp"
        android:layout_height="170dp"
        android:layout_centerHorizontal="true" />

```
你可以使用ImageLoader来显示一张图片，例如：
```

ImageLoader mImageLoader;
ImageView mImageView;
// The URL for the image that is being loaded.
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
...
mImageView = (ImageView) findViewById(R.id.regularImageView);

// Get the ImageLoader through your singleton class.
mImageLoader = MySingleton.getInstance(this).getImageLoader();
mImageLoader.get(IMAGE_URL, ImageLoader.getImageListener(mImageView,
         R.drawable.def_image, R.drawable.err_image));

```
然而，如果你要做得是为ImageView进行图片设置，你可以使用NetworkImageView来实现，例如:
```

ImageLoader mImageLoader;
NetworkImageView mNetworkImageView;
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
...

// Get the NetworkImageView that will display the image.
mNetworkImageView = (NetworkImageView) findViewById(R.id.networkImageView);

// Get the ImageLoader through your singleton class.
mImageLoader = MySingleton.getInstance(this).getImageLoader();

// Set the URL of the image that should be loaded into this view, and
// specify the ImageLoader that will be used to make the request.
mNetworkImageView.setImageUrl(IMAGE_URL, mImageLoader);

```
上面的代码是通过前一节课的单例模式来实现访问到RequestQueue与ImageLoader的。之所以这样做得原因是：对于ImageLoader(一个用来处理加载与缓存图片的帮助类)来说，单例模式可以避免旋转所带来的抖动。使用单例模式可以使得bitmap的缓存与activity的生命周期无关。如果你在activity中创建ImageLoader，这个ImageLoader有可能会在手机进行旋转的时候被重新创建。这可能会导致抖动。
##  Example LRU cache 
Volley工具箱中提供了通过` DiskBasedCache `实现的一种标准缓存。这个类能够缓存文件到磁盘的制定目录。**但是为了使用ImageLoader，你应该提供一个自定义的内存LRC缓存**，这个缓存需要实现` ImageLoader.ImageCache `的接口。**你可能想把你的缓存设置成一个` 单例 `**
下面是一个内存LRU Cache的实例。它` 继承自LruCache并实现了ImageLoader.ImageCache的接口 `：
```

import android.graphics.Bitmap;
import android.support.v4.util.LruCache;
import android.util.DisplayMetrics;
import com.android.volley.toolbox.ImageLoader.ImageCache;

public class LruBitmapCache extends LruCache<String, Bitmap>
        implements ImageCache {

    public LruBitmapCache(int maxSize) {
        super(maxSize);
    }

    public LruBitmapCache(Context ctx) {
        this(getCacheSize(ctx));
    }

    @Override
    protected int sizeOf(String key, Bitmap value) {
        return value.getRowBytes() * value.getHeight();
    }

    @Override
    public Bitmap getBitmap(String url) {
        return get(url);
    }

    @Override
    public void putBitmap(String url, Bitmap bitmap) {
        put(url, bitmap);
    }

    // Returns a cache size equal to approximately three screens worth of images.
    public static int getCacheSize(Context ctx) {
        final DisplayMetrics displayMetrics = ctx.getResources().
                getDisplayMetrics();
        final int screenWidth = displayMetrics.widthPixels;
        final int screenHeight = displayMetrics.heightPixels;
        // 4 bytes per pixel
        final int screenBytes = screenWidth * screenHeight * 4;

        return screenBytes * 3;
    }
}

```
下面是如何初始化ImageLoader并使用cache的实例:
```

RequestQueue mRequestQueue; // assume this exists.
ImageLoader mImageLoader = new ImageLoader(mRequestQueue, new LruBitmapCache(LruBitmapCache.getCacheSize()));

```
##  Request JSON 
Volley提供了以下的类用来执行JSON请求：

  * ` JsonArrayRequest ` - 一个为了获取JSONArray返回数据的请求。
  * ` JsonObjectRequest ` - 一个为了获取JSONObject返回数据的请求。允许把一个JSONObject作为请求参数。
这两个类都是继承自` JsonRequest `的。你可以使用类似的方法来处理这两种类型的请求。如下演示了如果获取一个JSON feed并显示到UI上：
```

TextView mTxtDisplay;
ImageView mImageView;
mTxtDisplay = (TextView) findViewById(R.id.txtDisplay);
String url = "http://my-json-feed";

JsonObjectRequest jsObjRequest = new JsonObjectRequest
        (Request.Method.GET, url, null, new Response.Listener() {

    @Override
    public void onResponse(JSONObject response) {
        mTxtDisplay.setText("Response: " + response.toString());
    }
}, new Response.ErrorListener() {

    @Override
    public void onErrorResponse(VolleyError error) {
        // TODO Auto-generated method stub

    }
});

// Access the RequestQueue through your singleton class.
MySingleton.getInstance(this).addToRequestQueue(jsObjRequest);

```
##  实现自定义的网络请求 
对于那些你需要自定义的请求类型，你需要执行以下操作：

  * 继承Request<T>类，<T>表示请求返回的数据类型。因此如果你需要解析的响应类型是一个String，可以通过继承Request<String>来创建你自定义的请求。请参考Volley工具类中的StringRequest与 ImageRequest来学习如何继承Request。
  * 实现抽象方法parseNetworkResponse()与deliverResponse()，下面会详细介绍。

```

public class GsonRequest<T> extends Request<T> {
    private final Gson gson = new Gson();
    private final Class<T> clazz;
    private final Map<String, String> headers;
    private final Listener<T> listener;

    /**
     * Make a GET request and return a parsed object from JSON.
     *
     * @param url URL of the request to make
     * @param clazz Relevant class object, for Gson's reflection
     * @param headers Map of request headers
     */
    public GsonRequest(String url, Class<T> clazz, Map<String, String> headers,
            Listener<T> listener, ErrorListener errorListener) {
        super(Method.GET, url, errorListener);
        this.clazz = clazz;
        this.headers = headers;
        this.listener = listener;
    }

    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        return headers != null ? headers : super.getHeaders();
    }

    @Override
    protected void deliverResponse(T response) {
        listener.onResponse(response);
    }

    @Override
    protected Response<T> parseNetworkResponse(NetworkResponse response) {
        try {
            String json = new String(
                    response.data,
                    HttpHeaderParser.parseCharset(response.headers));
            return Response.success(
                    gson.fromJson(json, clazz),
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        }
    }
}

```
