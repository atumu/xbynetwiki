title: android官方文档学习之theme_style 

#  android官方文档学习之Theme、Style 
Android上的Style分为了两个方面：
  * Theme是针对窗体级别的，改变窗体样式；
  * Style是针对窗体元素级别的，改变指定控件或者Layout的样式。

PS:为了研究Android的Style和Theme，强烈建议下载Android的base.git！
Android系统的themes.xml和style.xml(位于` /base/core/res/res/values/ `)包含了很多系统定义好的style，建议在里面挑个合适的，然后再继承修改。以下属性是在Themes中比较常见的，源自Android系统本身的themes.xml：
附件下载：![](/data/dokuwiki/android/android_stylesandthemes.xml.zip|)
```

<!-- Window attributes -->  
<item name="windowBackground">@android:drawable/screen_background_dark</item>  
<item name="windowFrame">@null</item>  
<item name="windowNoTitle">false</item>  
<item name="windowFullscreen">false</item>  
<item name="windowIsFloating">false</item>  
<item name="windowContentOverlay">@android:drawable/title_bar_shadow</item>  
<item name="windowTitleStyle">@android:style/WindowTitle</item>  
<item name="windowTitleSize">25dip</item>  
<item name="windowTitleBackgroundStyle">@android:style/WindowTitleBackground</item>  
<item name="android:windowAnimationStyle">@android:style/Animation.Activity</item>  

```
  
本文程序的themes.xml代码如下，自定义了WindowTitle,
```

<?xml version="1.0" encoding="UTF-8"?>
<resources>
 <!--继承Android内置的Theme.Light，位于/base/core/res/res/values/themes.xml -->
 <style name="Theme" parent="android:Theme.Light">
  <item name="android:windowFullscreen">true</item>
  <item name="android:windowTitleSize">60dip</item>
  <item name="android:windowTitleStyle">@style/WindowTitle</item>
 </style>

 <style name="WindowTitle" parent="android:WindowTitle">
  <item name="android:singleLine">true</item>
  <item name="android:shadowColor">#BB000000</item>
  <item name="android:shadowRadius">2.75</item>
 </style>
</resources> 

``` 

本文程序的styles.xml代码如下,background默认使用的是9.png,xml定义在/base/core/res/res/drawable/之下：
```

<?xml version="1.0" encoding="UTF-8"?>
<resources>
 <style name="TextView">
  <item name="android:textSize">18sp</item>
  <item name="android:textColor">#008</item>
  <item name="android:shadowColor">@android:color/black</item>
  <item name="android:shadowRadius">2.0</item>
 </style>

 <style name="EditText">
  <item name="android:shadowColor">@android:color/black</item>
  <item name="android:shadowRadius">1.0</item>
  <item name="android:background">@android:drawable/btn_default</item>
  <item name="android:textAppearance">?android:attr/textAppearanceMedium</item>
 </style>

    <style name="Button">
        <item name="android:background">@android:drawable/edit_text</item>
        <item name="android:textAppearance">?android:attr/textAppearanceMedium</item>
    </style>
</resources>

```
c参考：http://blog.csdn.net/hellogv/article/details/6128594