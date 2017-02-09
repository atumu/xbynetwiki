title: bitmap_avoid_oom 

#  Android高效加载位图避免OOM 

#  概述 

java.lang.OutofMemoryError: bitmap size exceeds VM budget.这个OOM异常应该是比较熟悉的。那么在小内存设备当中如何高效加载位图Bitmap是一个比较紧迫的问题。
Android中进行图片处理及加载操作一般不能在UI线程中进行。
#  有效地加载大位图文件 

android.graphics.BitmapFactory
##  在不分配内存的前提下读取一个位图的尺寸和类型 

BitmapFactory类提供了几个解码的方法(decodeByteArray(),decodeFile(),decodeResource(),等等)帮助你从多种资源创建位图。每种类型的解码方法都有额外的特征可以让你通过BitMapFactory.Options类指定解码选项。当解码时避免内存分配可以设置inJustDecodeBounds属性为true，位图对象返回null但是设置了outWidth，outHeight和outMimeType。` 这种技术允许你在创建位图(和分配内存)之前去读取图像的尺寸和类型。为了避免java.lang.OutOfMemory异常，在解码位图之前请检查它的尺寸，除非你十分确定资源提供给你的可预见的图像数据正好满足你的内存。 `
```

BitmapFactory.Options options = new BitmapFactory.Options();  
   
options.inJustDecodeBounds = true;  
BitmapFactory.decodeResource(getResources(), R.id.myimage, options);  
int imageHeight = options.outHeight;  
int imageWidth = options.outWidth;  
String imageType = options.outMimeType;  

```

#  加载一个缩小版本位图到内存中 

需要考虑的前提：
  * 估计加载完整图像所需要的内存;
  * 你承诺加载这个图片所需空间带给你的程序的其他内存需求;
  * 准备加载图像的目标ImageView或UI组件尺寸;
  * 当前设备的屏幕尺寸和密度;
告诉解码器去重新采样这个图像，加载一个更小的版本到内存中，在你的**BitmapFactory.Option对象中设置inSampleSize为true。**例如，将一个分辨率为2048*1536的图像用 inSampleSize值为4去编码将产生一个大小为大约512*384的位图。加载这个到内存中仅使用0.75MB，而不是完整的12MB大小的图像
下面是一个示例用于计算SampleSize的值：
```

public static int calculateInSampleSize(  
  
　　BitmapFactory.Options options, int reqWidth, int reqHeight) {  
　　// Raw height and width of image  
　　final int height = options.outHeight;  
　　final int width = options.outWidth;  
　　int inSampleSize = 1;  
　　if (height > reqHeight || width > reqWidth) {  
　　if (width > height) {  
　　inSampleSize = Math.round((float)height / (float)reqHeight);  
　　} else {  
　　inSampleSize = Math.round((float)width / (float)reqWidth);  
　　}  
　　}  
　　return inSampleSize;  
　　}  

```
**要使用这种方法，首先解码，将inJustDecodeBounds设置为true，将选项传递进去，然后再次解码，在使用新的inSampleSize值并将inJustDecodeBounds设置为false:**
```

public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,  
   
　　int reqWidth, int reqHeight) {  
   
　　// First decode with inJustDecodeBounds=true to check dimensions  
   
　　final BitmapFactory.Options options = new BitmapFactory.Options();  
   
　　options.inJustDecodeBounds = true;  
   
　　BitmapFactory.decodeResource(res, resId, options);  
   
　　// Calculate inSampleSize  
   
　　options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);  
   
　　// Decode bitmap with inSampleSize set  
   
　　options.inJustDecodeBounds = false;  
   
　　return BitmapFactory.decodeResource(res, resId, options);  
　　}  

```
这种方法可以很容易地加载任意大小的位图到一个ImageView显示一个100x100像素的缩略图，如下面的示例代码所示：
```

mImageView.setImageBitmap(  
decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));  

```

#  Processing Bitmaps Off the UI Thread 

这节将通过在后台线程中使用AsyncTask处理位图，并处理并发问题.
##  使用一个AsyncTask 

AsyncTask类提供了一种简单的方式来在一个后台线程中执行许多任务，并且把结果反馈给UI线程.使用的方法是，创建一个继承与它的子类并且实现提供的方法.这里是一个使用AsyncTask和decodeSampledBitmapFromResource()加载一个大图片到ImageView中的例子:
```

class BitmapWorkerTask extends AsyncTask {  
    private final WeakReference imageViewReference;  
    private int data = 0;  
   
    public BitmapWorkerTask(ImageView imageView) {  
        // Use a WeakReference to ensure the ImageView can be garbage collected  
        imageViewReference = new WeakReference(imageView);  
    }  
   
    // Decode image in background.  
    @Override  
    protected Bitmap doInBackground(Integer... params) {  
        data = params[0];  
        return decodeSampledBitmapFromResource(getResources(), data, 100, 100));  
    }  
   
    // Once complete, see if ImageView is still around and set bitmap.  
    @Override  
    protected void onPostExecute(Bitmap bitmap) {  
        if (imageViewReference != null && bitmap != null) {  
            final ImageView imageView = imageViewReference.get();  
            if (imageView != null) {  
                imageView.setImageBitmap(bitmap);  
            }  
        }  
    }  
 }  

```
对于ImageView来说WeakReference确保那时AsyncTask并不会阻碍ImageView和任何它的引用被垃圾回收期回收.不能保证ImageView在任务完成后仍然存在，所以你必须在onPostExecute()方法中检查它的引用.ImageView可能不再存在，如果例如，如果在任务完成之前用户退出了活动或者配置发生了变化.
为了异步地加载位图，简单地创建一个新的任务并且执行它:
```

public void loadBitmap(int resId, ImageView imageView) {  
  BitmapWorkerTask task = new BitmapWorkerTask(imageView);  
  task.execute(resId);  

```
#  处理并发 

常见的视图组件例如**ListView和GridView**如在上一节中当和AsyncTask结合使用时引出了另外一个问题.为了优化内存，当用户滚动时这些组件回收了子视图.如果每个子视图触发一个AsyncTask，**当它完成时没法保证，相关的视图还没有被回收时已经用在了别的子视图当中.此外，还有异步任务开始的顺序是不能保证他们完成的顺序.**
创建一个专用的Drawable的子类来存储一个引用**备份工作任务**.在这种情况下，一个BitmapDrawable被使用以便任务完成后一个**占位符图像**可以显示在ImageView中:
```

static class AsyncDrawable extends BitmapDrawable {  
    private final WeakReference bitmapWorkerTaskReference;  
   
    public AsyncDrawable(Resources res, Bitmap bitmap,  
            BitmapWorkerTask bitmapWorkerTask) {  
        super(res, bitmap);  
        bitmapWorkerTaskReference =  
            new WeakReference(bitmapWorkerTask);  
    }  
   
    public BitmapWorkerTask getBitmapWorkerTask() {  
        return bitmapWorkerTaskReference.get();  
    }  
 }  

```
执行BitmapWorkerTask前，你创建一个AsyncDrawable，**并将其绑定到目标ImageView:**
```

public void loadBitmap(int resId, ImageView imageView) {  
   if (cancelPotentialWork(resId, imageView)) {  
       final BitmapWorkerTask task = new BitmapWorkerTask(imageView);  
       final AsyncDrawable asyncDrawable =  
               new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);  
       imageView.setImageDrawable(asyncDrawable);  
       task.execute(resId);  
   }  
}  

```
**如果别的正在运行的任务已经和这个ImageView关联，cancelPotentialWork引用在上面的代码示例检查中**.如果这样，它试图通过调用cancel()取消先前的任务
```

public static boolean cancelPotentialWork(int data, ImageView imageView) {  
    final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);  
   
    if (bitmapWorkerTask != null) {  
        final int bitmapData = bitmapWorkerTask.data;  
        if (bitmapData != data) {  
            // Cancel previous task  
            bitmapWorkerTask.cancel(true);  
        } else {  
            // The same work is already in progress  
            return false;  
        }  
    }  
    // No task associated with the ImageView, or an existing task was cancelled  
    return true;  
[java] view plaincopy
private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {  
  if (imageView != null) {  
      final Drawable drawable = imageView.getDrawable();  
      if (drawable instanceof AsyncDrawable) {  
          final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;  
          return asyncDrawable.getBitmapWorkerTask();  
      }  
   }  
   return null;  
}  

```
这最后一步是在BitmapWorkerTask更新onPostExecute()方法，以便任务取消时并且当前任务和这个ImageView关联时进行检查:
```

class BitmapWorkerTask extends AsyncTask {  
   ...  
  
   @Override  
   protected void onPostExecute(Bitmap bitmap) {  
       if (isCancelled()) {  
           bitmap = null;  
       }  
  
       if (imageViewReference != null && bitmap != null) {  
           final ImageView imageView = imageViewReference.get();  
           final BitmapWorkerTask bitmapWorkerTask =  
                   getBitmapWorkerTask(imageView);  
           if (this == bitmapWorkerTask && imageView != null) {  
               imageView.setImageBitmap(bitmap);  
           }  
       }  
   }  
}  

```

#  缓存位图 

在许多情况下（例如有些组件像ListView,GridView以及ViewPager等），出现在屏幕上的图片总量，其中包括可能马上要滚动显示在屏幕上的那些图片，实际上是无限的.
当加载多个位图时使用一个内存和磁盘的位图缓存来提高响应速度以及提升整个UI界面的流畅性.
##  使用内存缓存LruCache 

其原理是最近被引用的对象保存在一个强引用LinkedHashMap中，以及在缓存超过了其指定的大小之前释放最近很少使用的对象的内存.
<note>注意：在过去，一个常用的内存缓存实现是一个SoftReference或WeakReference的位图缓存，然而现在不推荐使用.从android2.3（API 级别9）开始，垃圾回收器更加注重于回收软/弱引用，这使得使用以上引用很大程度上无效.</note>
为了选择一个合适的的LruCache大小，许多因素应当被予以考虑，例如:
  * 大量的图片是如何立刻出现在屏幕上的？需要多少即将在屏幕上显示的？
  * 设备的屏幕的大小和分辨率是多少？
  * 位图是什么尺寸和配置以及每张图片要占用多少内存？
  * 图片访问频繁嘛？比起别的将会被频繁地访问吗？也许你可能总是想要在内存中保存一定项，甚至对于不同的位图组来说有多个LRUCache对象.
  * 你能在数量和质量之间取得平衡嘛？有时对于存储更多的低质量的位图是更有用的,潜在地在另外的后台任务中加载一个更高质量版本.
缓存太小会导致额外的没有益处的开销，缓存过大会再次导致java.lang.OutOfMemory异常，并且只保留下你的应用程序其余相当少的内存来运行你的应用程序. 这个例子是针对位图来设置一个LruCache:(` 注意：一般需要覆盖sizeOf() `)
```

private LruCache mMemoryCache;  
@Override  
protected void onCreate(Bundle savedInstanceState) {  
   ...  
   // Get memory class of this device, exceeding this amount will throw an  
   // OutOfMemory exception.  
   final int memClass = ((ActivityManager) context.getSystemService(  
           Context.ACTIVITY_SERVICE)).getMemoryClass();  
   // Use 1/8th of the available memory for this memory cache.  
   final int cacheSize = 1024 * 1024 * memClass / 8;  
   mMemoryCache = new LruCache(cacheSize) {  
       @Override  
       protected int sizeOf(String key, Bitmap bitmap) {  
           // The cache size will be measured in bytes rather than number of items.  
           return bitmap.getByteCount();  
       }  
   };  
   ...  
}  
public void addBitmapToMemoryCache(String key, Bitmap bitmap) {  
   if (getBitmapFromMemCache(key) == null) {  
       mMemoryCache.put(key, bitmap);  
   }  
}  
public Bitmap getBitmapFromMemCache(String key) {  
    return mMemoryCache.get(key);  
}  

```
注意:在这个例子中，应用程序中八分之一的内存被用作缓存.在一个普通的/hdpi设备中最低有4MB.在一个800*480分辨率的设备上全屏显示一个充满图片的GridView控件视图将使用1.5MB左右的缓存，所以这将在内存中缓冲至少四分之一的图片.
当将一张位图加载进一个ImageView中，LruCache首先被检查.如果有输入，缓存立刻被使用来更新这个ImageView视图,否则一个后台线程随之诞生来处理这张图片
```

public void loadBitmap(int resId, ImageView imageView) {  
   final String imageKey = String.valueOf(resId);  
   final Bitmap bitmap = getBitmapFromMemCache(imageKey);  
   if (bitmap != null) {  
       mImageView.setImageBitmap(bitmap);  
   } else {  
       mImageView.setImageResource(R.drawable.image_placeholder);  
       BitmapWorkerTask task = new BitmapWorkerTask(mImageView);  
       task.execute(resId);  
   }  
}  

class BitmapWorkerTask extends AsyncTask {  
    ...  
    // Decode image in background.  
    @Override  
    protected Bitmap doInBackground(Integer... params) {  
        final Bitmap bitmap = decodeSampledBitmapFromResource(  
                getResources(), params[0], 100, 100));  
        addBitmapToMemoryCache(String.valueOf(params[0]), bitmap);  
        return bitmap;  
    }  
    ...  
 }  

```
##  使用磁盘缓存DiskLruCache 

一个内存缓冲对于加快访问最近浏览过的位图是很有用的，然而你不能局限图片在缓存中可用.像GridView这种具有更大的数据集的组件很容易地会占用所有内存缓存.你的应用程序会被别的任务像打电话等打断，并且当运行在后台时被进程杀死以及内存缓存被回收.一旦用户重新打开，你的应用程序不得不重新处理每一张图片.
在这种情况下使用磁盘缓存来持续处理位图，并且有助于在图片在内存缓存中不再可用时缩短加载时间.当然，从磁盘获取图片比从内存加载更慢并且应当在后台线程中处理，因为磁盘读取的时间是不可预知的. 注意:如果它们被更频繁地访问，那么一个ContentProvider可能是一个更合适的地方来存储缓存中的图像，例如在一个图片库应用程序里.
` 注意:DiskLruCache没有包含在API中而是包含在4.0源码中， `
下载位置：
https://android.googlesource.com/platform/libcore/+/jb-mr2-release/luni/src/main/java/libcore/io/DiskLruCache.java
```

/*
 * Copyright (C) 2012 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.example.android.displayingbitmaps.util;

import android.annotation.TargetApi;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Bitmap.CompressFormat;
import android.graphics.Bitmap.Config;
import android.graphics.BitmapFactory;
import android.graphics.drawable.BitmapDrawable;
import android.os.Build.VERSION_CODES;
import android.os.Bundle;
import android.os.Environment;
import android.os.StatFs;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.util.LruCache;

import com.example.android.common.logger.Log;
import com.example.android.displayingbitmaps.BuildConfig;

import java.io.File;
import java.io.FileDescriptor;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.lang.ref.SoftReference;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Collections;
import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;

/**
 * This class handles disk and memory caching of bitmaps in conjunction with the
 * {@link ImageWorker} class and its subclasses. Use
 * {@link ImageCache#getInstance(android.support.v4.app.FragmentManager, ImageCacheParams)} to get an instance of this
 * class, although usually a cache should be added directly to an {@link ImageWorker} by calling
 * {@link ImageWorker#addImageCache(android.support.v4.app.FragmentManager, ImageCacheParams)}.
 */
public class ImageCache {
    private static final String TAG = "ImageCache";

    // Default memory cache size in kilobytes
    private static final int DEFAULT_MEM_CACHE_SIZE = 1024 * 5; // 5MB

    // Default disk cache size in bytes
    private static final int DEFAULT_DISK_CACHE_SIZE = 1024 * 1024 * 10; // 10MB

    // Compression settings when writing images to disk cache
    private static final CompressFormat DEFAULT_COMPRESS_FORMAT = CompressFormat.JPEG;
    private static final int DEFAULT_COMPRESS_QUALITY = 70;
    private static final int DISK_CACHE_INDEX = 0;

    // Constants to easily toggle various caches
    private static final boolean DEFAULT_MEM_CACHE_ENABLED = true;
    private static final boolean DEFAULT_DISK_CACHE_ENABLED = true;
    private static final boolean DEFAULT_INIT_DISK_CACHE_ON_CREATE = false;

    private DiskLruCache mDiskLruCache;
    private LruCache<String, BitmapDrawable> mMemoryCache;
    private ImageCacheParams mCacheParams;
    private final Object mDiskCacheLock = new Object();
    private boolean mDiskCacheStarting = true;

    private Set<SoftReference<Bitmap>> mReusableBitmaps;

    /**
     * Create a new ImageCache object using the specified parameters. This should not be
     * called directly by other classes, instead use
     * {@link ImageCache#getInstance(android.support.v4.app.FragmentManager, ImageCacheParams)} to fetch an ImageCache
     * instance.
     *
     * @param cacheParams The cache parameters to use to initialize the cache
     */
    private ImageCache(ImageCacheParams cacheParams) {
        init(cacheParams);
    }

    /**
     * Return an {@link ImageCache} instance. A {@link RetainFragment} is used to retain the
     * ImageCache object across configuration changes such as a change in device orientation.
     *
     * @param fragmentManager The fragment manager to use when dealing with the retained fragment.
     * @param cacheParams The cache parameters to use if the ImageCache needs instantiation.
     * @return An existing retained ImageCache object or a new one if one did not exist
     */
    public static ImageCache getInstance(
            FragmentManager fragmentManager, ImageCacheParams cacheParams) {

        // Search for, or create an instance of the non-UI RetainFragment
        final RetainFragment mRetainFragment = findOrCreateRetainFragment(fragmentManager);

        // See if we already have an ImageCache stored in RetainFragment
        ImageCache imageCache = (ImageCache) mRetainFragment.getObject();

        // No existing ImageCache, create one and store it in RetainFragment
        if (imageCache == null) {
            imageCache = new ImageCache(cacheParams);
            mRetainFragment.setObject(imageCache);
        }

        return imageCache;
    }

    /**
     * Initialize the cache, providing all parameters.
     *
     * @param cacheParams The cache parameters to initialize the cache
     */
    private void init(ImageCacheParams cacheParams) {
        mCacheParams = cacheParams;


        // Set up memory cache
        if (mCacheParams.memoryCacheEnabled) {
            if (BuildConfig.DEBUG) {
                Log.d(TAG, "Memory cache created (size = " + mCacheParams.memCacheSize + ")");
            }

            // If we're running on Honeycomb or newer, create a set of reusable bitmaps that can be
            // populated into the inBitmap field of BitmapFactory.Options. Note that the set is
            // of SoftReferences which will actually not be very effective due to the garbage
            // collector being aggressive clearing Soft/WeakReferences. A better approach
            // would be to use a strongly references bitmaps, however this would require some
            // balancing of memory usage between this set and the bitmap LruCache. It would also
            // require knowledge of the expected size of the bitmaps. From Honeycomb to JellyBean
            // the size would need to be precise, from KitKat onward the size would just need to
            // be the upper bound (due to changes in how inBitmap can re-use bitmaps).
            if (Utils.hasHoneycomb()) {
                mReusableBitmaps =
                        Collections.synchronizedSet(new HashSet<SoftReference<Bitmap>>());
            }

            mMemoryCache = new LruCache<String, BitmapDrawable>(mCacheParams.memCacheSize) {

                /**
                 * Notify the removed entry that is no longer being cached
                 */
                @Override
                protected void entryRemoved(boolean evicted, String key,
                        BitmapDrawable oldValue, BitmapDrawable newValue) {
                    if (RecyclingBitmapDrawable.class.isInstance(oldValue)) {
                        // The removed entry is a recycling drawable, so notify it
                        // that it has been removed from the memory cache
                        ((RecyclingBitmapDrawable) oldValue).setIsCached(false);
                    } else {
                        // The removed entry is a standard BitmapDrawable

                        if (Utils.hasHoneycomb()) {
                            // We're running on Honeycomb or later, so add the bitmap
                            // to a SoftReference set for possible use with inBitmap later
                            mReusableBitmaps.add(new SoftReference<Bitmap>(oldValue.getBitmap()));
                        }
                    }
                }

                /**
                 * Measure item size in kilobytes rather than units which is more practical
                 * for a bitmap cache
                 */
                @Override
                protected int sizeOf(String key, BitmapDrawable value) {
                    final int bitmapSize = getBitmapSize(value) / 1024;
                    return bitmapSize == 0 ? 1 : bitmapSize;
                }
            };
        }


        // By default the disk cache is not initialized here as it should be initialized
        // on a separate thread due to disk access.
        if (cacheParams.initDiskCacheOnCreate) {
            // Set up disk cache
            initDiskCache();
        }
    }

    /**
     * Initializes the disk cache.  Note that this includes disk access so this should not be
     * executed on the main/UI thread. By default an ImageCache does not initialize the disk
     * cache when it is created, instead you should call initDiskCache() to initialize it on a
     * background thread.
     */
    public void initDiskCache() {
        // Set up disk cache
        synchronized (mDiskCacheLock) {
            if (mDiskLruCache == null || mDiskLruCache.isClosed()) {
                File diskCacheDir = mCacheParams.diskCacheDir;
                if (mCacheParams.diskCacheEnabled && diskCacheDir != null) {
                    if (!diskCacheDir.exists()) {
                        diskCacheDir.mkdirs();
                    }
                    if (getUsableSpace(diskCacheDir) > mCacheParams.diskCacheSize) {
                        try {
                            mDiskLruCache = DiskLruCache.open(
                                    diskCacheDir, 1, 1, mCacheParams.diskCacheSize);
                            if (BuildConfig.DEBUG) {
                                Log.d(TAG, "Disk cache initialized");
                            }
                        } catch (final IOException e) {
                            mCacheParams.diskCacheDir = null;
                            Log.e(TAG, "initDiskCache - " + e);
                        }
                    }
                }
            }
            mDiskCacheStarting = false;
            mDiskCacheLock.notifyAll();
        }
    }

    /**
     * Adds a bitmap to both memory and disk cache.
     * @param data Unique identifier for the bitmap to store
     * @param value The bitmap drawable to store
     */
    public void addBitmapToCache(String data, BitmapDrawable value) {

        if (data == null || value == null) {
            return;
        }

        // Add to memory cache
        if (mMemoryCache != null) {
            if (RecyclingBitmapDrawable.class.isInstance(value)) {
                // The removed entry is a recycling drawable, so notify it
                // that it has been added into the memory cache
                ((RecyclingBitmapDrawable) value).setIsCached(true);
            }
            mMemoryCache.put(data, value);
        }

        synchronized (mDiskCacheLock) {
            // Add to disk cache
            if (mDiskLruCache != null) {
                final String key = hashKeyForDisk(data);
                OutputStream out = null;
                try {
                    DiskLruCache.Snapshot snapshot = mDiskLruCache.get(key);
                    if (snapshot == null) {
                        final DiskLruCache.Editor editor = mDiskLruCache.edit(key);
                        if (editor != null) {
                            out = editor.newOutputStream(DISK_CACHE_INDEX);
                            value.getBitmap().compress(
                                    mCacheParams.compressFormat, mCacheParams.compressQuality, out);
                            editor.commit();
                            out.close();
                        }
                    } else {
                        snapshot.getInputStream(DISK_CACHE_INDEX).close();
                    }
                } catch (final IOException e) {
                    Log.e(TAG, "addBitmapToCache - " + e);
                } catch (Exception e) {
                    Log.e(TAG, "addBitmapToCache - " + e);
                } finally {
                    try {
                        if (out != null) {
                            out.close();
                        }
                    } catch (IOException e) {}
                }
            }
        }

    }

    /**
     * Get from memory cache.
     *
     * @param data Unique identifier for which item to get
     * @return The bitmap drawable if found in cache, null otherwise
     */
    public BitmapDrawable getBitmapFromMemCache(String data) {

        BitmapDrawable memValue = null;

        if (mMemoryCache != null) {
            memValue = mMemoryCache.get(data);
        }

        if (BuildConfig.DEBUG && memValue != null) {
            Log.d(TAG, "Memory cache hit");
        }

        return memValue;

    }

    /**
     * Get from disk cache.
     *
     * @param data Unique identifier for which item to get
     * @return The bitmap if found in cache, null otherwise
     */
    public Bitmap getBitmapFromDiskCache(String data) {

        final String key = hashKeyForDisk(data);
        Bitmap bitmap = null;

        synchronized (mDiskCacheLock) {
            while (mDiskCacheStarting) {
                try {
                    mDiskCacheLock.wait();
                } catch (InterruptedException e) {}
            }
            if (mDiskLruCache != null) {
                InputStream inputStream = null;
                try {
                    final DiskLruCache.Snapshot snapshot = mDiskLruCache.get(key);
                    if (snapshot != null) {
                        if (BuildConfig.DEBUG) {
                            Log.d(TAG, "Disk cache hit");
                        }
                        inputStream = snapshot.getInputStream(DISK_CACHE_INDEX);
                        if (inputStream != null) {
                            FileDescriptor fd = ((FileInputStream) inputStream).getFD();

                            // Decode bitmap, but we don't want to sample so give
                            // MAX_VALUE as the target dimensions
                            bitmap = ImageResizer.decodeSampledBitmapFromDescriptor(
                                    fd, Integer.MAX_VALUE, Integer.MAX_VALUE, this);
                        }
                    }
                } catch (final IOException e) {
                    Log.e(TAG, "getBitmapFromDiskCache - " + e);
                } finally {
                    try {
                        if (inputStream != null) {
                            inputStream.close();
                        }
                    } catch (IOException e) {}
                }
            }
            return bitmap;
        }

    }

    /**
     * @param options - BitmapFactory.Options with out* options populated
     * @return Bitmap that case be used for inBitmap
     */
    protected Bitmap getBitmapFromReusableSet(BitmapFactory.Options options) {

        Bitmap bitmap = null;

        if (mReusableBitmaps != null && !mReusableBitmaps.isEmpty()) {
            synchronized (mReusableBitmaps) {
                final Iterator<SoftReference<Bitmap>> iterator = mReusableBitmaps.iterator();
                Bitmap item;

                while (iterator.hasNext()) {
                    item = iterator.next().get();

                    if (null != item && item.isMutable()) {
                        // Check to see it the item can be used for inBitmap
                        if (canUseForInBitmap(item, options)) {
                            bitmap = item;

                            // Remove from reusable set so it can't be used again
                            iterator.remove();
                            break;
                        }
                    } else {
                        // Remove from the set if the reference has been cleared.
                        iterator.remove();
                    }
                }
            }
        }

        return bitmap;

    }

    /**
     * Clears both the memory and disk cache associated with this ImageCache object. Note that
     * this includes disk access so this should not be executed on the main/UI thread.
     */
    public void clearCache() {
        if (mMemoryCache != null) {
            mMemoryCache.evictAll();
            if (BuildConfig.DEBUG) {
                Log.d(TAG, "Memory cache cleared");
            }
        }

        synchronized (mDiskCacheLock) {
            mDiskCacheStarting = true;
            if (mDiskLruCache != null && !mDiskLruCache.isClosed()) {
                try {
                    mDiskLruCache.delete();
                    if (BuildConfig.DEBUG) {
                        Log.d(TAG, "Disk cache cleared");
                    }
                } catch (IOException e) {
                    Log.e(TAG, "clearCache - " + e);
                }
                mDiskLruCache = null;
                initDiskCache();
            }
        }
    }

    /**
     * Flushes the disk cache associated with this ImageCache object. Note that this includes
     * disk access so this should not be executed on the main/UI thread.
     */
    public void flush() {
        synchronized (mDiskCacheLock) {
            if (mDiskLruCache != null) {
                try {
                    mDiskLruCache.flush();
                    if (BuildConfig.DEBUG) {
                        Log.d(TAG, "Disk cache flushed");
                    }
                } catch (IOException e) {
                    Log.e(TAG, "flush - " + e);
                }
            }
        }
    }

    /**
     * Closes the disk cache associated with this ImageCache object. Note that this includes
     * disk access so this should not be executed on the main/UI thread.
     */
    public void close() {
        synchronized (mDiskCacheLock) {
            if (mDiskLruCache != null) {
                try {
                    if (!mDiskLruCache.isClosed()) {
                        mDiskLruCache.close();
                        mDiskLruCache = null;
                        if (BuildConfig.DEBUG) {
                            Log.d(TAG, "Disk cache closed");
                        }
                    }
                } catch (IOException e) {
                    Log.e(TAG, "close - " + e);
                }
            }
        }
    }

    /**
     * A holder class that contains cache parameters.
     */
    public static class ImageCacheParams {
        public int memCacheSize = DEFAULT_MEM_CACHE_SIZE;
        public int diskCacheSize = DEFAULT_DISK_CACHE_SIZE;
        public File diskCacheDir;
        public CompressFormat compressFormat = DEFAULT_COMPRESS_FORMAT;
        public int compressQuality = DEFAULT_COMPRESS_QUALITY;
        public boolean memoryCacheEnabled = DEFAULT_MEM_CACHE_ENABLED;
        public boolean diskCacheEnabled = DEFAULT_DISK_CACHE_ENABLED;
        public boolean initDiskCacheOnCreate = DEFAULT_INIT_DISK_CACHE_ON_CREATE;

        /**
         * Create a set of image cache parameters that can be provided to
         * {@link ImageCache#getInstance(android.support.v4.app.FragmentManager, ImageCacheParams)} or
         * {@link ImageWorker#addImageCache(android.support.v4.app.FragmentManager, ImageCacheParams)}.
         * @param context A context to use.
         * @param diskCacheDirectoryName A unique subdirectory name that will be appended to the
         *                               application cache directory. Usually "cache" or "images"
         *                               is sufficient.
         */
        public ImageCacheParams(Context context, String diskCacheDirectoryName) {
            diskCacheDir = getDiskCacheDir(context, diskCacheDirectoryName);
        }

        /**
         * Sets the memory cache size based on a percentage of the max available VM memory.
         * Eg. setting percent to 0.2 would set the memory cache to one fifth of the available
         * memory. Throws {@link IllegalArgumentException} if percent is < 0.01 or > .8.
         * memCacheSize is stored in kilobytes instead of bytes as this will eventually be passed
         * to construct a LruCache which takes an int in its constructor.
         *
         * This value should be chosen carefully based on a number of factors
         * Refer to the corresponding Android Training class for more discussion:
         * http://developer.android.com/training/displaying-bitmaps/
         *
         * @param percent Percent of available app memory to use to size memory cache
         */
        public void setMemCacheSizePercent(float percent) {
            if (percent < 0.01f || percent > 0.8f) {
                throw new IllegalArgumentException("setMemCacheSizePercent - percent must be "
                        + "between 0.01 and 0.8 (inclusive)");
            }
            memCacheSize = Math.round(percent * Runtime.getRuntime().maxMemory() / 1024);
        }
    }

    /**
     * @param candidate - Bitmap to check
     * @param targetOptions - Options that have the out* value populated
     * @return true if <code>candidate</code> can be used for inBitmap re-use with
     *      <code>targetOptions</code>
     */
    @TargetApi(VERSION_CODES.KITKAT)
    private static boolean canUseForInBitmap(
            Bitmap candidate, BitmapFactory.Options targetOptions) {

        if (!Utils.hasKitKat()) {
            // On earlier versions, the dimensions must match exactly and the inSampleSize must be 1
            return candidate.getWidth() == targetOptions.outWidth
                    && candidate.getHeight() == targetOptions.outHeight
                    && targetOptions.inSampleSize == 1;
        }

        // From Android 4.4 (KitKat) onward we can re-use if the byte size of the new bitmap
        // is smaller than the reusable bitmap candidate allocation byte count.
        int width = targetOptions.outWidth / targetOptions.inSampleSize;
        int height = targetOptions.outHeight / targetOptions.inSampleSize;
        int byteCount = width * height * getBytesPerPixel(candidate.getConfig());
        return byteCount <= candidate.getAllocationByteCount();

    }

    /**
     * Return the byte usage per pixel of a bitmap based on its configuration.
     * @param config The bitmap configuration.
     * @return The byte usage per pixel.
     */
    private static int getBytesPerPixel(Config config) {
        if (config == Config.ARGB_8888) {
            return 4;
        } else if (config == Config.RGB_565) {
            return 2;
        } else if (config == Config.ARGB_4444) {
            return 2;
        } else if (config == Config.ALPHA_8) {
            return 1;
        }
        return 1;
    }

    /**
     * Get a usable cache directory (external if available, internal otherwise).
     *
     * @param context The context to use
     * @param uniqueName A unique directory name to append to the cache dir
     * @return The cache dir
     */
    public static File getDiskCacheDir(Context context, String uniqueName) {
        // Check if media is mounted or storage is built-in, if so, try and use external cache dir
        // otherwise use internal cache dir
        final String cachePath =
                Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||
                        !isExternalStorageRemovable() ? getExternalCacheDir(context).getPath() :
                                context.getCacheDir().getPath();

        return new File(cachePath + File.separator + uniqueName);
    }

    /**
     * A hashing method that changes a string (like a URL) into a hash suitable for using as a
     * disk filename.
     */
    public static String hashKeyForDisk(String key) {
        String cacheKey;
        try {
            final MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update(key.getBytes());
            cacheKey = bytesToHexString(mDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            cacheKey = String.valueOf(key.hashCode());
        }
        return cacheKey;
    }

    private static String bytesToHexString(byte[] bytes) {
        // http://stackoverflow.com/questions/332079
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(0xFF & bytes[i]);
            if (hex.length() == 1) {
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString();
    }

    /**
     * Get the size in bytes of a bitmap in a BitmapDrawable. Note that from Android 4.4 (KitKat)
     * onward this returns the allocated memory size of the bitmap which can be larger than the
     * actual bitmap data byte count (in the case it was re-used).
     *
     * @param value
     * @return size in bytes
     */
    @TargetApi(VERSION_CODES.KITKAT)
    public static int getBitmapSize(BitmapDrawable value) {
        Bitmap bitmap = value.getBitmap();

        // From KitKat onward use getAllocationByteCount() as allocated bytes can potentially be
        // larger than bitmap byte count.
        if (Utils.hasKitKat()) {
            return bitmap.getAllocationByteCount();
        }

        if (Utils.hasHoneycombMR1()) {
            return bitmap.getByteCount();
        }

        // Pre HC-MR1
        return bitmap.getRowBytes() * bitmap.getHeight();
    }

    /**
     * Check if external storage is built-in or removable.
     *
     * @return True if external storage is removable (like an SD card), false
     *         otherwise.
     */
    @TargetApi(VERSION_CODES.GINGERBREAD)
    public static boolean isExternalStorageRemovable() {
        if (Utils.hasGingerbread()) {
            return Environment.isExternalStorageRemovable();
        }
        return true;
    }

    /**
     * Get the external app cache directory.
     *
     * @param context The context to use
     * @return The external cache dir
     */
    @TargetApi(VERSION_CODES.FROYO)
    public static File getExternalCacheDir(Context context) {
        if (Utils.hasFroyo()) {
            return context.getExternalCacheDir();
        }

        // Before Froyo we need to construct the external cache dir ourselves
        final String cacheDir = "/Android/data/" + context.getPackageName() + "/cache/";
        return new File(Environment.getExternalStorageDirectory().getPath() + cacheDir);
    }

    /**
     * Check how much usable space is available at a given path.
     *
     * @param path The path to check
     * @return The space available in bytes
     */
    @TargetApi(VERSION_CODES.GINGERBREAD)
    public static long getUsableSpace(File path) {
        if (Utils.hasGingerbread()) {
            return path.getUsableSpace();
        }
        final StatFs stats = new StatFs(path.getPath());
        return (long) stats.getBlockSize() * (long) stats.getAvailableBlocks();
    }

    /**
     * Locate an existing instance of this Fragment or if not found, create and
     * add it using FragmentManager.
     *
     * @param fm The FragmentManager manager to use.
     * @return The existing instance of the Fragment or the new instance if just
     *         created.
     */
    private static RetainFragment findOrCreateRetainFragment(FragmentManager fm) {

        // Check to see if we have retained the worker fragment.
        RetainFragment mRetainFragment = (RetainFragment) fm.findFragmentByTag(TAG);

        // If not retained (or first time running), we need to create and add it.
        if (mRetainFragment == null) {
            mRetainFragment = new RetainFragment();
            fm.beginTransaction().add(mRetainFragment, TAG).commitAllowingStateLoss();
        }

        return mRetainFragment;

    }

    /**
     * A simple non-UI Fragment that stores a single Object and is retained over configuration
     * changes. It will be used to retain the ImageCache object.
     */
    public static class RetainFragment extends Fragment {
        private Object mObject;

        /**
         * Empty constructor as per the Fragment documentation
         */
        public RetainFragment() {}

        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            // Make sure this Fragment is retained over a configuration change
            setRetainInstance(true);
        }

        /**
         * Store a single object in this Fragment.
         *
         * @param object The object to store
         */
        public void setObject(Object object) {
            mObject = object;
        }

        /**
         * Get the stored object.
         *
         * @return The stored object
         */
        public Object getObject() {
            return mObject;
        }
    }

}


```
` 当内存缓存在UI线程中被检查时，磁盘缓存在后台线程中被检查.磁盘操作应当永远不会发生在UI线程中.当图片处理完成时，最终的位图都将被添加到内存和磁盘缓存中 `
##  处理配置更改 

程序运行时配置改变，例如屏幕的方向改变，导致系统销毁活动并且采用新的配置重新运行活动.你想要避免不得不再次处理所有的图片以使用户在配置发生改变时有一个平稳和快速的体验.
缓存能通过新的活动实例来使用一个Fragment，**这个Fragment是通过调用setRetainInstance(true)方法被保留的**.活动被重新构造后，保留的片段重新连接，并且你获得现有的高速缓存对象的访问权限，使得图片能快速的加载并重新填充到ImageView对象中.
接下来是一个使用Fragment将一个LruCache对象保留在配置更改中的范例:
```

private LruCache mMemoryCache;  
   
@Override  
protected void onCreate(Bundle savedInstanceState) {  
    ...  
    RetainFragment mRetainFragment =  
            RetainFragment.findOrCreateRetainFragment(getFragmentManager());  
    mMemoryCache = RetainFragment.mRetainedCache;  
    if (mMemoryCache == null) {  
        mMemoryCache = new LruCache(cacheSize) {  
            ... // Initialize cache here as usual  
        }  
        mRetainFragment.mRetainedCache = mMemoryCache;  
    }  
    ...  
}  
   
class RetainFragment extends Fragment {  
    private static final String TAG = "RetainFragment";  
    public LruCache mRetainedCache;  
   
    public RetainFragment() {}  
   
    public static RetainFragment findOrCreateRetainFragment(FragmentManager fm) {  
        RetainFragment fragment = (RetainFragment) fm.findFragmentByTag(TAG);  
        if (fragment == null) {  
            fragment = new RetainFragment();  
        }  
        return fragment;  
    }  
   
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setRetainInstance(true);  
    }  
}  

```