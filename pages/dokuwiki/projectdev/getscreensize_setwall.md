title: getscreensize_setwall 

#  Android获取屏幕分辨率和大小与设置壁纸、动态壁纸 

#  获取屏幕分辨率和大小 
```

// 方法1 Android获得屏幕的宽和高      
        WindowManager windowManager = getWindowManager();      
        Display display = windowManager.getDefaultDisplay();      
        int screenWidth = screenWidth = display.getWidth();      
        int screenHeight = screenHeight = display.getHeight();      
              
        // 方法2     
        DisplayMetrics dm = new DisplayMetrics();    
        getWindowManager().getDefaultDisplay().getMetrics(dm);     
        float width=dm.widthPixels*dm.density;     
        float height=dm.heightPixels*dm.density; 

```
#  设置静态壁纸： 

1、别忘记在ApplicationManifest.xml 中加上权限的设置。<uses-permission android:name = "android.permission.SET_WALLPAPER"/>2、设置壁纸的方法总结。壁纸设置方法有三种第一 通过WallpaperManager方法中的 setBitmap（）第二 通过WallpaperManager方法中的 setResource（）第三 通过ContextWrapper 类中提供的setWallpaper（）方法
```

WallpaperManager wallpaperManager = WallpaperManager.getInstance(this);    
 Resources res = getResources();    
 Bitmap bitmap=BitmapFactory.decodeResource(res, getResources().getIdentifier("wallpaper" + imagePosition, "drawable", "com.ch"));     
wallpaperManager.setBitmap(bitmap);  

```
#  动态壁纸 

Livewallpaper，即动态墙纸，是Android的一大3D特色功能，用户可以在桌面选择加载动态墙纸，让自己的手机桌面背景旋动起来。
Android动态墙纸的本质是一个“Service”。
<action Android:name="android.service.wallpaper.WallpaperService" />
系统通过APK的这个action把其当做一个动态墙纸加载进LivePicker列表，用户在LivePicker列表里选择自己喜欢的动态墙纸，进而将动态墙纸显示进Launcher的背后。
动态墙纸和Launcher是完全不同的两个进程，只不过Launcher和动态墙纸的进程可以通过框架里的WallpaperManager进行进程间通信罢了，用户在Launcher桌面滑动、点击屏幕时有的动态墙纸能产生交互效果，实际上就是这个进程通信完成的。
**实现思路：**
1、在res/xml/下添加一个xml文件描述动态壁纸的缩略图，比如livewallpaper.xml:
注意: 这里的android:thumbnail值得是你这个动态壁纸的小图标 会在你选着动态壁纸的时候出现。
android:settingsActivity:添加用于设置的Activity.
```

<?xml version="1.0" encoding="utf-8"?>  
<wallpaper xmlns:android="http://schemas.android.com/apk/res/android" android:thumbnail="@drawable/icon"  android:description="@string/wallpaper_description"   android:settingsActivity="de.vogella.android.wallpaper.MyPreferencesActivity"/>  

```
2、AndroidManifest配置：
```

<?xml version="1.0" encoding="utf-8"?>  
<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
      package="org.crazyit.desktop"  
      android:versionCode="1"  
      android:versionName="1.0">   
      <uses-feature    //添加feature  
        android:name="android.software.live_wallpaper"  
        android:required="true" >  
    </uses-feature>  
    <application  
        android:icon="@drawable/icon"  
        android:label="@string/app_name">  
        <!-- 配置实时壁纸Service -->  
        <service android:label="@string/app_name"  
            android:name=".LiveWallpaper"  
            android:permission="android.permission.BIND_WALLPAPER">  //添加权限  
            <!-- 为实时壁纸配置intent-filter -->  
            <intent-filter>  
                <action  android:name="android.service.wallpaper.WallpaperService" /> //添加intent-filter  
            </intent-filter>  
            <!-- 为实时壁纸配置meta-data -->  
            <meta-data android:name="android.service.wallpaper"  //添加meta-data  
                android:resource="@xml/livewallpaper" />  
        </service>  
        <activity      //添加用于设置的Activity  
            android:name=".MyPreferencesActivity"  
            android:exported="true"   //注意设置导出  
            android:label="@string/app_name"  
            android:theme="@android:style/Theme.Light.WallpaperSettings" >  
        </activity>  
    </application>  
</manifest>   

```
3、实现动态壁纸是不需要使用Activity, 创建LiveWallpaper类，让其继承WallpaperService:必须覆盖 onCreateEngine()方法,返回自己实现的Engine类
在Engine类中的onCreate()方法中进行调用绘制图形方法。Engine类定义了生命周期方法，如onCreate(), onSurfaceCreated(), onVisibilityChanged(), onOffsetsChanged(), onTouchEvent() and onCommand().
例如以下示例：
```

public class LiveWallpaper extends WallpaperService  
{  
    // 实现WallpaperService必须实现的抽象方法  
    public Engine onCreateEngine()  
    {  
        // 返回自定义的Engine  
        return new MyEngine();  
    }  
  
    class MyEngine extends Engine  
    {  
        // 记录程序界面是否可见  
        private boolean mVisible;  
        // 记录当前当前用户动作事件的发生位置  
        private float mTouchX = -1;  
        private float mTouchY = -1;  
        // 记录当前圆圈的绘制位置  
          
        //左上角坐标  
        private float cx1 = 15;  
        private float cy1 = 20;  
          
        //右下角坐标  
        private float cx2 = 300;  
        private float cy2 = 380;  
          
        //右上角坐标  
        private float cx3 = 300;  
        private float cy3 = 20;  
          
        //左下角坐标  
        private float cx4 = 15;  
        private float cy4 = 380;  
          
        // 定义画笔  
        private Paint mPaint = new Paint();  
        // 定义一个Handler  
        Handler mHandler = new Handler();  
        // 定义一个周期性执行的任务  
        private final Runnable drawTarget = new Runnable()  
        {  
            public void run()  
            {  
                // 动态地绘制图形  
                drawFrame();  
            }  
        };  
  
        @Override  
        public void onCreate(SurfaceHolder surfaceHolder)  
        {  
            super.onCreate(surfaceHolder);  
            // 初始化画笔  
            mPaint.setColor(0xffffffff);  
            mPaint.setAntiAlias(true);  
            mPaint.setStrokeWidth(2);  
            mPaint.setStrokeCap(Paint.Cap.ROUND);  
            mPaint.setStyle(Paint.Style.STROKE);  
            // 设置处理触摸事件  
            setTouchEventsEnabled(true);  
        }  
  
        @Override  
        public void onDestroy()  
        {  
            super.onDestroy();  
            // 删除回调  
            mHandler.removeCallbacks(drawTarget);  
        }  
  
        @Override  
        public void onVisibilityChanged(boolean visible)  
        {  
            mVisible = visible;  
            // 当界面可见时候，执行drawFrame()方法。  
            if (visible)  
            {  
                // 动态地绘制图形  
                drawFrame();  
            }  
            else  
            {  
                // 如果界面不可见，删除回调  
                mHandler.removeCallbacks(drawTarget);  
            }  
        }  
  
        public void onOffsetsChanged(float xOffset, float yOffset, float xStep,  
            float yStep, int xPixels, int yPixels)  
        {  
            drawFrame();  
        }  
  
  
        public void onTouchEvent(MotionEvent event)  
        {  
            // 如果检测到滑动操作  
            if (event.getAction() == MotionEvent.ACTION_MOVE)  
            {  
                mTouchX = event.getX();  
                mTouchY = event.getY();  
            }  
            else  
            {  
                mTouchX = -1;  
                mTouchY = -1;  
            }  
            super.onTouchEvent(event);  
        }  
  
        // 定义绘制图形的工具方法  
        private void drawFrame()  
        {  
            // 获取该壁纸的SurfaceHolder  
            final SurfaceHolder holder = getSurfaceHolder();  
            Canvas c = null;  
            try  
            {  
                // 对画布加锁  
                c = holder.lockCanvas();  
                if (c != null)  
                {  
                    c.save();  
                    // 绘制背景色  
                    c.drawColor(0xff000000);  
                    // 在触碰点绘制圆圈  
                    drawTouchPoint(c);  
                      
                    // 绘制圆圈  
                    c.drawCircle(cx1, cy1, 80, mPaint);  
                    c.drawCircle(cx2, cy2, 40, mPaint);  
                    c.drawCircle(cx3, cy3, 50, mPaint);  
                    c.drawCircle(cx4, cy4, 60, mPaint);  
                    c.restore();  
                }  
            }  
            finally  
            {  
                if (c != null)  
                    holder.unlockCanvasAndPost(c);  
            }  
            mHandler.removeCallbacks(drawTarget);  
            // 调度下一次重绘  
            if (mVisible)  
            {  
                cx1 += 6;  
                cy1 += 8;  
                // 如果cx1、cy1移出屏幕后从左上角重新开始  
                if (cx1 > 320)  
                    cx1 = 15;  
                if (cy1 > 400)  
                    cy1 = 20;  
                  
                  
                cx2 -= 6;  
                cy2 -= 8;  
                // 如果cx2、cy2移出屏幕后从右下角重新开始  
                if (cx2 <15)  
                    cx2 = 300;  
                if (cy2 <20)  
                    cy2 = 380;  
                  
                  
                cx3 -= 6;  
                cy3 += 8;  
                // 如果cx3、cy3移出屏幕后从右上角重新开始  
                if (cx3 <0)  
                    cx3 = 300;  
                if (cy3 >400)  
                    cy3 = 20;  
                  
                  
                cx4 += 6;  
                cy4 -= 8;  
                // 如果cx4、cy4移出屏幕后从左下角重新开始  
                if (cx4 >320)  
                    cx4 = 15;  
                if (cy4 <0)  
                    cy4 = 380;  
                  
                // 指定0.1秒后重新执行mDrawCube一次  
                mHandler.postDelayed(drawTarget, 100);  
            }  
        }  
  
        // 在屏幕触碰点绘制圆圈  
        private void drawTouchPoint(Canvas c)  
        {  
            if (mTouchX >= 0 && mTouchY >= 0)  
            {  
                c.drawCircle(mTouchX, mTouchY, 40, mPaint);  
            }  
        }  
    }  
}  

```
4、在Activity中通过Intent调用系统设置livewallpaper界面:
```

Intent intent = new Intent(WallpaperManager.ACTION_CHANGE_LIVE_WALLPAPER);  
   intent.putExtra(WallpaperManager.EXTRA_LIVE_WALLPAPER_COMPONENT,  
       new ComponentName(this, MyWallpaperService.class));  
   startActivity(intent);  
       finish();  

```

参考：
http://www.cnblogs.com/flyme/archive/2011/08/15/2139553.html
http://blog.csdn.net/t12x3456/article/details/7857741
http://www.learn-android-easily.com/2013/07/android-livewallpaer-tutorial.html
http://www.vogella.com/tutorials/AndroidLiveWallpaper/article.html
http://code.tutsplus.com/tutorials/creating-live-wallpapers-on-android--mobile-9516