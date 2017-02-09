title: android代码混淆工具proguard学习 

#  android代码混淆工具proguard学习 

Proguard代码混淆工具：可以对代码进行去冗余压缩，代码优化，代码混淆等。在Android中的主要应用就是对代码混淆：就是将类名，方法名，Field名变成如a,b,c或者1,2,3等难以阅读和理解的名字，以防止逆向工程和被反编译阅读源码。

#  使用Proguard 

##  启用 

Eclipse下：
项目根路径下有两个文件：project.properties和proguard-project.txt
在project.properties中有这样一段话：
# To enable ProGuard to shrink and obfuscate your code, uncomment this (available properties: sdk.dir, user.home):
#proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt:proguard-project.txt
设置sdk.dir变量然后取消proguard.config注释即可启用。

AndroidStudio下：
项目根路径下有两个文件:build.gradle和proguard-rules.pro
在build.gradle中有如下片段，即启用proguard。
```

android{  
......  
  buildTypes {  
        release {  
            minifyEnabled false  
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'  
        }  
    }  
}  

```
##  不能混淆的代码 

系统接口
Jni接口
其他如四大组件、View等
第三方类库如android-support
##  配置语法 
```

-include {filename}    从给定的文件中读取配置参数   
-basedirectory {directoryname}    指定基础目录为以后相对的档案名称   
-injars {class_path}    指定要处理的应用程序jar,war,ear和目录   
-outjars {class_path}    指定处理完后要输出的jar,war,ear和目录的名称   
-libraryjars {classpath}    指定要处理的应用程序jar,war,ear和目录所需要的程序库文件   
-dontskipnonpubliclibraryclasses    指定不去忽略非公共的库类。   
-dontskipnonpubliclibraryclassmembers    指定不去忽略包可见的库类的成员。  
  
保留选项   
-keep {Modifier} {class_specification}    保护指定的类文件和类的成员   
-keepclassmembers {modifier} {class_specification}    保护指定类的成员，如果此类受到保护他们会保护的更好  
-keepclasseswithmembers {class_specification}    保护指定的类和类的成员，但条件是所有指定的类和类成员是要存在。   
-keepnames {class_specification}    保护指定的类和类的成员的名称（如果他们不会压缩步骤中删除）   
-keepclassmembernames {class_specification}    保护指定的类的成员的名称（如果他们不会压缩步骤中删除）   
-keepclasseswithmembernames {class_specification}    保护指定的类和类的成员的名称，如果所有指定的类成员出席（在压缩步骤之后）   
-printseeds {filename}    列出类和类的成员-keep选项的清单，标准输出到给定的文件   
  
压缩   
-dontshrink    不压缩输入的类文件   
-printusage {filename}   
-whyareyoukeeping {class_specification}       
  
优化   
-dontoptimize    不优化输入的类文件   
-assumenosideeffects {class_specification}    优化时假设指定的方法，没有任何副作用   
-allowaccessmodification    优化时允许访问并修改有修饰符的类和类的成员   
  
混淆   
-dontobfuscate    不混淆输入的类文件   
-printmapping {filename}   
-applymapping {filename}    重用映射增加混淆   
-obfuscationdictionary {filename}    使用给定文件中的关键字作为要混淆方法的名称   
-overloadaggressively    混淆时应用侵入式重载   
-useuniqueclassmembernames    确定统一的混淆类的成员名称来增加混淆   
-flattenpackagehierarchy {package_name}    重新包装所有重命名的包并放在给定的单一包中   
-repackageclass {package_name}    重新包装所有重命名的类文件中放在给定的单一包中   
-dontusemixedcaseclassnames    混淆时不会产生形形色色的类名   
-keepattributes {attribute_name,...}    保护给定的可选属性，例如LineNumberTable, LocalVariableTable, SourceFile, Deprecated, Synthetic, Signature, and   
  
InnerClasses.   
-renamesourcefileattribute {string}    设置源文件中给定的字符串常量  

```
#  应用实例（ 
更多的配置例子可以查看sdk的proguard文件夹下面的examples目录）
保留所有的本地native方法不被混淆。这点在我们做ndk开发的时候很重要。
-keepclasseswithmembernames class * {
native <methods>;
}
保留所有的set和get开头的方法。
-keepclassmembers public class * extends android.view.View {
void set*(***);
*** get*();
}
# We want to keep methods in Activity that could be used in the XML attribute onClick
我们保留在activity中的方法参数是view的方法，这样的话，我们在xml里面编写onClick就不会被影响了。
-keepclassmembers class * extends android.app.Activity {
public void *(android.view.View);
}
-keepclassmembernames class *******************.Model{
java.lang.Long mId;
}
上面的一连串*号是我的包名。
笔者一开始是这么写的。
-keepclassmembernames class *******************.Model{
Long mId;
}
` 后来查看了proguard文件夹下面examples文件夹下面的许多pro文件推荐参照android.pro这个文件写，发现了他们的共性，发现不是基本类型是不能这么写的。 `
#  保留第三方类库： 

[java] view plaincopy
-libraryjars /libs/achartengine-1.1.0.jar  
-dontwarn achartengine.**  
-keep class achartengine.** { *;}  
  
-libraryjars /libs/android-support-v4.jar  
-dontwarn android.support.v4.**  
-keep class android.support.v4.** { *;}  
#  使用Proguard删除Log代码 

```

-assumenosideeffects  
class android.util.Log  
 {  
    public static ***  
 e(...);  
    public static ***  
 w(...);  
    public static ***  
 wtf(...);  
    public static ***  
 d(...);  
    public static ***  
 v(...);  
}  

```
#  demo实例： 

```

-ignorewarnings                     # 忽略警告，避免打包时某些警告出现  
-optimizationpasses 5               # 指定代码的压缩级别  
-dontusemixedcaseclassnames         # 是否使用大小写混合  
-dontskipnonpubliclibraryclasses    # 是否混淆第三方jar  
-dontpreverify                      # 混淆时是否做预校验  
-verbose                            # 混淆时是否记录日志  
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*        # 混淆时所采用的算法  
  
-libraryjars   libs/treecore.jar  
  
-dontwarn android.support.v4.**     #缺省proguard 会检查每一个引用是否正确，但是第三方库里面往往有些不会用到的类，没有正确引用。如果不配置的话，系统就会报错。  
-dontwarn android.os.**  
-keep class android.support.v4.** { *; }        # 保持哪些类不被混淆  
-keep class com.baidu.** { *; }    
-keep class vi.com.gdi.bgl.android.**{*;}  
-keep class android.os.**{*;}  
  
-keep interface android.support.v4.app.** { *; }    
-keep public class * extends android.support.v4.**    
-keep public class * extends android.app.Fragment  
  
-keep public class * extends android.app.Activity  
-keep public class * extends android.app.Application  
-keep public class * extends android.app.Service  
-keep public class * extends android.content.BroadcastReceiver  
-keep public class * extends android.content.ContentProvider  
-keep public class * extends android.support.v4.widget  
-keep public class * extends com.sqlcrypt.database  
-keep public class * extends com.sqlcrypt.database.sqlite  
-keep public class * extends com.treecore.**  
-keep public class * extends de.greenrobot.dao.**  
  
  
-keepclasseswithmembernames class * {       # 保持 native 方法不被混淆  
    native <methods>;  
}  
  
-keepclasseswithmembers class * {            # 保持自定义控件类不被混淆  
    public <init>(android.content.Context, android.util.AttributeSet);  
}  
  
-keepclasseswithmembers class * {            # 保持自定义控件类不被混淆  
    public <init>(android.content.Context, android.util.AttributeSet, int);  
}  
  
-keepclassmembers class * extends android.app.Activity { //保持类成员  
   public void *(android.view.View);  
}  
  
-keepclassmembers enum * {                  # 保持枚举 enum 类不被混淆  
    public static **[] values();  
    public static ** valueOf(java.lang.String);  
}  
  
-keep class * implements android.os.Parcelable {    # 保持 Parcelable 不被混淆  
  public static final android.os.Parcelable$Creator *;  
}  
  
-keep class MyClass;                              # 保持自己定义的类不被混淆  

```
#  proguard运行后分析异常追踪 
1、<project_root>/bin/proguard文件夹分析：
mapping.txt
表示混淆前后代码的对照表，这个文件非常重要。如果你的代码混淆后会产生bug的话，log提示中是混淆后的代码，希望定位到源代码的话就可以根据mapping.txt反推。
` 使用mapping.txt还原异常栈：使用retrace.bat位于<sdk_root>/tools/proguard/bin `
<blockquote>retrace.bat|retrace.sh [-verbose] mapping.txt [<stacktrace_file>]  
For example:  
retrace.bat -verbose mapping.txt obfuscated_trace.txt </blockquote> 
每次发布都要保留它方便该版本出现问题时调出日志进行排查，它可以根据版本号或是发布时间命名来保存或是放进代码版本控制中。
dump.txt
描述apk内所有class文件的内部结构。
seeds.txt
列出了没有被混淆的类和成员。
usage.txt
列出了源代码中被删除在apk中不存在的代码。


参考：
http://blog.csdn.net/banketree/article/details/41928175
http://my.oschina.net/zxcholmes/blog/312627
http://blog.csdn.net/binyao02123202/article/details/18940715