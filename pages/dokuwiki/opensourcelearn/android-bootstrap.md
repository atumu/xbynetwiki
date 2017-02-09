title: android-bootstrap 

#  Android-Bootstrap:AndroidTextView/Button/EditText/ImageView漂亮组件 
<WRAP center round alert 60%>
` **使用注意事项：必须在XML文件中为BootstrapButton配置android:text属性，否则无法通过setText()改变显示文本。** `
</WRAP>


![](/data/dokuwiki/opensourcelearn/pasted/20150528-090859.png?200*100)


Android-Bootstrap是一个美观的Button/EditText/ImageView/Thumbnail组件
项目地址：https://github.com/Bearded-Hen/Android-Bootstrap
demo地址：https://github.com/Bearded-Hen/AndroidBootstrapSample
wiki:https://github.com/Bearded-Hen/Android-Bootstrap/wiki
安装：Gradle:
```

dependencies {
   compile 'com.beardedhen:androidbootstrap:+'
}

```
基本使用：
1、Paste the following XML into a layout file to create a BootstrapButton:
```

<!-- basic button -->
<com.beardedhen.androidbootstrap.BootstrapButton
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_margin="10dp"
android:text="Success"
bootstrap:bb_icon_right="fa-android"
bootstrap:bb_type="success"
/>

```
2、Add the bootstrap namespace to the root view of your layout:
```

xmlns:bootstrap="http://schemas.android.com/apk/res-auto"

```

参考：[【Android】开源项目汇总-备用](http://www.eoeandroid.com/home.php?mod=space&uid=765778&do=blog&id=47674)