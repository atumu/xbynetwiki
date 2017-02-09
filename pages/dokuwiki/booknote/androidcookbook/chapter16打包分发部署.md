title: chapter16打包分发部署 


#   打包、部署和分发应用 
Created Saturday 17 January 2015

##  创建签名证书，签署应用程序 
jdk工具keytool产生自签名证书；jarsigner进行签署
keytool -genkey -v -keystore myapp.keystore -alias myapp -keyalg RSA -validity 10000 （10000天）
jarsigner -verbose -keystore myapp.keystore myapp.apk mykey

##  通过Android Play分发应用程序 

##  将AdMob集成到应用中 

##  用ProGuard进行代码混淆和优化 
示例：
```

-optimizationpasses 5
-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-dontpreverify
-verbose
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class com.android.vending.licensing.ILicensingService

-keepclasseswithmembernames class * {
	native <methods>;
}

-keepclasseswithmembers class * {
	public <init>(android.content.Context, android.util.AttributeSet);
}

-keepclasseswithmembers class * {
	public <init>(android.content.Context, android.util.AttributeSet, int);
}

-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

-keepclassmembers enum * {
	public static **[] values();
	public static ** valueOf(java.lang.String);
}

-keep class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator *;
}
-dontpreverify关闭予校验
keep,keepclassesmembers, keepclasseswithmembernames指必须保留的特殊类。

```