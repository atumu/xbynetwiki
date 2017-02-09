title: androidstudio使用_mipmap_ic_launcher编译报错解决 

#  AndroidStudio使用@mipmap/ic_launcher编译报错解决 
原文链接：http://www.apkfuns.com/error16-9-attribute-applicationicon-valuemipmapic_launcher-from-androidmanifest-xml169.html
![](/data/dokuwiki/tooluse/pasted/20150529-125441.png)
![](/data/dokuwiki/tooluse/pasted/20150529-125455.png)
```

xmlns:tools="http://schemas.android.com/tools"  
<application  
        android:allowBackup="true"  
        android:icon="@mipmap/ic_launcher"  
        android:label="@string/app_name"  
        android:theme="@style/AppTheme"  
        android:name=".app.App"  
        tools:replace="name,icon,label,theme"> 

``` 
