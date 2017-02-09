title: softandweakreference 

#  Android开发优化系列之使用软引用和弱引用 

Java从JDK1.2版本开始，就把对象的引用分为四种级别，从而使程序能更加灵活的控制对象的生命周期。这四种级别由高到低依次为：强引用、软引用、弱引用和虚引用。
这里重点介绍一下软引用和弱引用。
**如果一个对象只具有软引用，那么如果内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。**只要垃圾回收器没有回收它，该对象就可以被程序使用。**软引用可用来实现内存敏感的高速缓存**。软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。
**如果一个对象只具有弱引用，那么在垃圾回收器线程扫描的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。**弱引用也可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。
**弱引用与软引用的根本区别在于：只具有弱引用的对象拥有更短暂的生命周期，可能随时被回收。而只具有软引用的对象只有当内存不够的时候才被回收，在内存足够的时候，通常不被回收。**
在java.lang.ref包中提供了几个类：SoftReference类、WeakReference类和PhantomReference类，它们分别代表软引用、弱引用和虚引用。ReferenceQueue类表示引用队列，它可以和这三种引用类联合使用，以便跟踪Java虚拟机回收所引用的对象的活动。

` 在Android应用的开发中，为了防止内存溢出，在处理一些占用内存大而且声明周期较长的对象时候，可以尽量应用软引用和弱引用技术。 `
下面以使用软引用为例来详细说明。弱引用的使用方式与软引用是类似的。
假设我们的应用会用到大量的默认图片，比如应用中有默认的头像，默认游戏图标等等，这些图片很多地方会用到。如果每次都去读取图片，由于读取文件需要硬件操作，速度较慢，会导致性能较低。所以我们考虑将图片缓存起来，需要的时候直接从内存中读取。但是，由于图片占用内存空间比较大，缓存很多图片需要很多的内存，就可能比较容易发生OutOfMemory异常。这时，我们可以考虑使用软引用技术来避免这个问题发生。
首先定义一个HashMap，保存软引用对象。
```

private Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();  
public void addBitmapToCache(String path) {  
        // 强引用的Bitmap对象  
        Bitmap bitmap = BitmapFactory.decodeFile(path);  
        // 软引用的Bitmap对象  
        SoftReference<Bitmap> softBitmap = new SoftReference<Bitmap>(bitmap);  
        // 添加该对象到Map中使其缓存  
        imageCache.put(path, softBitmap);  
    }  
public Bitmap getBitmapByPath(String path) {  
       // 从缓存中取软引用的Bitmap对象  
       SoftReference<Bitmap> softBitmap = imageCache.get(path);  
       // 判断是否存在软引用  
       if (softBitmap == null) {  
           return null;  
       }  
       // 取出Bitmap对象，如果由于内存不足Bitmap被回收，将取得空  
       Bitmap bitmap = softBitmap.get();  
       return bitmap;  
   }  

```
<note important>注意：一定要判断是否为null，以免出现NullPointerException异常导致应用崩溃</note>

经验分享：
到底什么时候使用软引用，什么时候使用弱引用呢？
个人认为，**如果只是想避免OutOfMemory异常的发生，则可以使用软引用。**如果对于应用的性能更在意，想尽快回收一些占用内存比较大的对象，则可以使用弱引用。
还有就是可以根据对象是否经常使用来判断。如果该对象可能会经常使用的，就尽量用软引用。如果该对象不被使用的可能性更大些，就可以用弱引用。
另外，和弱引用功能类似的是WeakHashMap。WeakHashMap对于一个给定的键，其映射的存在并不阻止垃圾回收器对该键的回收，回收以后，其条目从映射中有效地移除。WeakHashMap使用ReferenceQueue实现的这种机制。


原文：http://blog.csdn.net/arui319/article/details/8489451