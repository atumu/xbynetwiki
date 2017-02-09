title: android_gradle插件学习 

#  android_gradle插件学习 

Java工程有三大构建工具Ant, Maven, Gradle。Ant需要手工编写太多东西，Maven通过使用约定优于配置的策略管理项目依赖于项目构建，而且建有Maven中央仓库。Gradle也是使用约定优于配置的策略基于任务驱动型。` 并且能能管理依赖并从maven中央仓库或者jcenter拉取下载相关依赖（也可以自定义maven仓库地址）推荐Android工程采用jcenter,java工程采用maven仓库。 `自动化项目的构建。
认识Gradle是从AndroidStudio开始的。
采用Gradle作为新构建系统的目标：
  - 让重用代码和资源变得更加容易。
  - 让创建同一应用程序的不同版本变得更加容易，无论是多个apk发布版本还是同一个应用的不同定制版本。
  - 让构建过程变得更加容易配置，扩展和定制。
  - 整合优秀的IDE
#  使用 
```

buildscript {  
    //设置仓库地址  
    repositories {  
        //如果使用maven中央仓库mavenCentral()  
        //自定义maven仓库地址  
        /* 
        * maven { 
        *   url "http://maven.petrikainulainen.net/repo" 
        *} 
        */  
        //推荐Android工程采用jcenter,java工程采用maven仓库  
        jcenter()  
    }  
    //设置gradle版本，不是app应用依赖，  
    dependencies {  
        //如果出现版本不一致问题，可以这么写:  
        //classpath 'com.android.tools.build:gradle:1.2.+'  
        classpath 'com.android.tools.build:gradle:1.2.2'  
  
        // NOTE: Do not place your application dependencies here; they belong  
        // in the individual module build.gradle files  
    }  
    //有时候,我们的代码使用utf-8 保存的,但是,进行gradle build 的环境是gbk这类的,这时候会包如下错误:  
    //编码GBK的不可映射字符.这个时候我们就需要手动的设置编译时编码类型.  
     tasks.withType(JavaCompile) {   
            options.encoding = "UTF-8"   
    }    
}  
  
//apply plugin: 'android'  
//apply plugin: 'android-library'  
apply plugin: 'com.android.application'  
  
android {  
    compileSdkVersion 22  
    buildToolsVersion "22.0.1"  
    //为避免引入第三方库是报错，排除可能重复的license文件  
    packagingOptions {  
        exclude 'META-INF/DEPENDENCIES'  
        exclude 'META-INF/NOTICE'  
        exclude 'META-INF/LICENSE'  
        exclude 'META-INF/LICENSE.txt'  
        exclude 'META-INF/NOTICE.txt'  
    }  
    defaultConfig {  
        applicationId "net.xbynet.setwallpaper"  
        minSdkVersion 14  
        targetSdkVersion 22  
        versionCode 1  
        versionName "1.0"  
    }  
    //构建类型，可以构造如release版本或者debug版本或者多渠道打包等  
    buildTypes {  
        release {  
            minifyEnabled false  
            //下面为proguard文件配置  
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'  
             minifyEnabled true  
            zipAlignEnabled true  
            // 移除无用的resource文件  
            shrinkResources true  
            signingConfig signingConfigs.release  
        }  
        hiapk {  
             packageNameSuffix ".hiapk"  
         }  
         playstore {  
             packageNameSuffix ".playstore"  
        }  
    }  
     // 多渠道打包，以友盟为例，当然在manifest文件中存在一个placeHolder  
    /** 
    *   <meta-data 
    *   android:name="UMENG_CHANNEL" 
    *   android:value="${UMENG_CHANNEL_VALUE}" /> 
    */  
    /* 
    *也可以采用这种方式 
    *  productFlavors { 
    *   xiaomi { 
    *       manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"] 
    *   } 
    *   _360 { 
    *       manifestPlaceholders = [UMENG_CHANNEL_VALUE: "_360"] 
    *   } 
    *   baidu { 
    *       manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu"] 
    *   } 
    *   wandoujia { 
    *       manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"] 
    *   } 
    *}  
    */  
    productFlavors {  
        wandoujia {}  
        _360 {}  
        baidu {}  
        xiaomi {}       
    }  
    productFlavors.all { flavor ->  
        flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]  
    }  
}  
    //签名相关  
       signingConfigs {  
        debug {  
            // No debug config  
        }  
        release {  
             // 不显示Log  
            buildConfigField "boolean", "LOG_DEBUG", "false"  
            storeFile file("../yourapp.keystore")  
            storePassword "your password"  
            keyAlias "your alias"  
            keyPassword "your password"  
        }  
    }  
}  
//依赖设置，不属于android子元素。  
dependencies {  
    //注释掉下面一句，因为这会引起android.support-v4出现多个的问题。  
    //compile fileTree(dir: 'libs', include: '*.jar')  
    //取而代之采用如下形式:在maven仓库下groupId则不同为com.google.android  
    compile 'com.android.support:appcompat-v7:22.0.0'  
    compile files('libs/eventbus-2.4.0.jar') //依赖于本地工程libs目录下的eventbus-2.4.0.jar  
    compile 'com.android.support:appcompat-v7:22.0.0' //从maven中央库下载依赖  
    provided 'org.roboguice:roboblender:3.0'  //从maven中央库下载依赖  
    compile 'org.roboguice:roboguice:3.0'   //从maven中央库下载依赖  
    compile project(':libraries:lib1')   //依赖于某个项目  
}  

```
参考：
http://blog.csdn.net/changemyself/article/details/39927381
http://blog.jobbole.com/72992/
http://www.cnblogs.com/youxilua/p/3348162.html
http://www.cnblogs.com/youxilua/archive/2013/05/20/3087935.html
http://blog.csdn.net/ljchlx/article/details/43059467