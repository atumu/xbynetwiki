title: android官方文档学习之drawable 

#  android官方文档学习之Drawable 
官方文档：http://developer.android.com/guide/topics/resources/drawable-resource.html
一个drawable资源,就是能被画到屏幕的一个一般的图形概态<简单说一下, drawable就是对图片向上抽取之后的一个抽象名称,图片有多种表现形式,接着看下文.文中一些地方还是直译为drawable>.它可以用getDrawable(int)这个方法api或者将其应用到xml文件资源中,使用android:drawable and android:icon. 这两个属性.
有如下多种图形概态.
**Bitmap File**一个位图文件(.png,.jpg,或.gif), 生成一个` BitmapDrawable `对象.
**Nine-Patch File**就是一张可以基于自动适应内容大小而伸缩区域的png图片(.9.png), 生成一个` NinePatchDrawable `对象
**Layer List**这个Drawable用来管理一个其它多个drawable的数组.既然是一个数组,所以就不难理解索引值最大的元素将画在最高部. 生成一个 LayerDrawable对象.
**State List**这是一个xml文件用于不同的状态来引用不同的位图图形(比如,当一个Button控件按下状态要显示不同的图像).生成一个StateListDrawable对象.
Level List一个xml文件,定义了一个drawable可用于管理几个可以替换的drawable.每一个都会分配一个最大的数值.生成一个LevelListDrawable.
Transition Drawable一个xml文件,定义了一个drawable可用于两张图片形成一个渐变的过渡效果生成一个TransitionDrawable对象
Inset Drawable一个xml文件,定义了一个drawable,跟据指定的距离插入到另一个drawable.当一个View<视图>对象需要一张比其实际边框要小的背景图时,就可以用到这个了.
Clip Drawable一个xml文件,定义了一个drawable, 根据当前对准值作相应的拉伸处理,生成 ClipDrawable对象.
Scale Drawable一个xml文件,定义了一个drawable, 根据当前对准值作相应的平铺处理,生成 ScaleDrawable对象.
Shape Drawable就是通过一个xml文件来定义一个包含颜色和渐变的几何图形, 生成一个 ShapeDrawable对象
另见Animation Resource <动画资源>文档,学习如何创建一个` AnimationDrawable `对像.
<note tip>注：在xml中一个color resource<颜色资源>也可以作为一个drawable. 例如,创建一个state list drawable时,你可以为android:drawable属性引用一个颜色资源(android:drawable="@color/green").</note>
##  State List Drawable 
状态列表Drawable是一个定义在XML中的可绘制对象，使用几个不同的图像，根据对象的状态来呈现同一个图形。例如，一个Button控件，可以处于几种状态中的一种（**按下，聚焦，或都不是**）使用状态列表可绘制，可以为每个状态提供不同的背景图像。
文件位置：
res/drawable/filename.xml
以文件名作为标示资源的ID。
资源引用：
Java: R.drawable.filename
XML: @[package:]drawable/filename
语法：
```

<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"
    android:constantSize=["true" | "false"]
    android:dither=["true" | "false"]
    android:variablePadding=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:state_pressed=["true" | "false"]
        android:state_focused=["true" | "false"]
        android:state_hovered=["true" | "false"]
        android:state_selected=["true" | "false"]
        android:state_checkable=["true" | "false"]
        android:state_checked=["true" | "false"]
        android:state_enabled=["true" | "false"]
        android:state_activated=["true" | "false"]
        android:state_window_focused=["true" | "false"] />
</selector>

示例：
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"
          android:drawable="@drawable/button_pressed" /> <!-- pressed -->
    <item android:state_focused="true"
          android:drawable="@drawable/button_focused" /> <!-- focused -->
    <item android:state_hovered="true"
          android:drawable="@drawable/button_focused" /> <!-- hovered -->
    <item android:drawable="@drawable/button_normal" /> <!-- default -->
</selector>

```
其他：略，参考官方文档或者wiki.eoeandroid.com/Drawable.html