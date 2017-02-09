title: chapter5图形 


#   图形 
Created Sunday 08 February 2015
OpenGL EL

##  使用自定义字体 
在assets/fonts中安装字体的TTF或OTF版本。调用View的setTypeface()即可
而Typeface.几种create方法则可以创建Typeface实例。
示例：FontDemo
Typeface t=Typeface.createFromAsset(getAssets(),"fonts/fontdemo.ttf");
v.setTypeface(t,Typeface.BOLD_ITALIC);

##  用OpenGL ES绘制旋转的方块 
略

##  用意图拍照 
Intent intent=new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
/ /建立保存图像的文件
imgFile=new File(MainActivity.getDataDir(),FileNameUtils.getNextFilename(".jpg"));
Uri uri=Uri.fromFile(imgFile);
intent.putExtra(MediaStore.EXTRA_OUTPUT,uri);
intent.putExtra(MediaStore.EXTRA_VIDEO_QUALITY,1);
startActivityForResult(intent,0);

##  用android.media.Camera拍照 
解决方案：创建一个SurfaceView并实现用户拍照时触发的回调，以便控制图像捕捉过程

##  用Google ZXing条码扫描程序扫描条形码或QR代码 

##  用AndroidPlot显示图表和图形 

##  使用Inkscape创建Android启动器图标 

##  从OpenClipArt.org用Paint.Net创建简易启动器图标 
http://www.openclipart.org提供了超过33000个免费的图形

##  使用Nine Patch文件 

##  用Android RGraph创建HTML5图表 

##  添加简单的光栅动画 
用AnimationDrawable类能轻松地排序图像
res/drawable创建一个以animation-list为根的xml文件
android:src
覆盖onWindowFocusChanged()方法添加animationDrawable.start()可以使动画开始之前所有部件均已加载。

##  使用捏合缩放 





