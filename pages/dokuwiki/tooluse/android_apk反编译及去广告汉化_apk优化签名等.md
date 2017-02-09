title: android_apk反编译及去广告汉化_apk优化签名等 

#  Android APK反编译及去广告汉化，apk优化签名等 


Android APK反编译是个有趣的事情。我们可以对APK反编译进行汉化破解工作等。
Android APK反编译主要涉及三个工具的使用，分别是
apktool
dex2jar
jd-gui(即jad)
最后介绍了一款国人开发的集成反编译工具AndroidKiller

Android APK去广告及汉化：使用的工具apktool及android sdk自带的signapk.jar
Android APK 优化与签名：使用的工具androidsdk 自带的zipalign与signapk.jar

本文附带工具整理下载
Android-ApkTool

功能： 可以反编译成smali的中间代码文件和可人类友好的XML资源文件如AndroidManifest.xml
项目地址：原地址：https://code.google.com/p/android-apktool/，但是该项目已经搬迁至http://ibotpeaches.github.io/Apktool/
使用说明：
官方文档地址：http://ibotpeaches.github.io/Apktool/documentation/
项目最新版本为2.0，与1.x版本的使用差异：
Examples of new usage in 2.0 vs 1.5.x
Old (Apktool 1.5.x)	<span style="white-space:pre">			</span>New (Apktool 2.0.x)
apktool if framework-res.apk tag	<span style="white-space:pre">	</span>apktool if framework-res.apk -t tag
apktool d framework-res.apk output	<span style="white-space:pre">	</span>apktool d framework.res.apk -o output
apktool b output new.apk	<span style="white-space:pre">		</span>apktool b output -o new.apk
下面为具体使用
基本：
apktool d testapp.apk

Decoding反编码：
$ apktool d foo.jar
// decodes foo.jar to foo.jar.out folder

$ apktool decode foo.jar
// decodes foo.jar to foo.jar.out folder

$ apktool d bar.apk
// decodes bar.apk to bar folder

$ apktool decode bar.apk
// decodes bar.apk to bar folder

$ apktool d bar.apk -o baz
// decodes bar.apk to baz folder

构建Building

$ apktool b foo.jar.out
// builds foo.jar.out folder into foo.jar.out/dist/foo.jar file

$ apktool build foo.jar.out
// builds foo.jar.out folder into foo.jar.out/dist/foo.jar file

$ apktool b bar
// builds bar folder into bar/dist/bar.apk file

$ apktool b .
// builds current directory into ./dist

$ apktool b bar -o new_bar.apk
// builds bar folder into new_bar.apk

$ apktool b bar.apk
// WRONG: brut.androlib.AndrolibException: brut.directory.PathNotExist: apktool.yml
// Must use folder, not apk/jar file


dex2jar

简介 ：把dex文件转换成jar文件
项目地址：原地址：https://code.google.com/p/dex2jar/，最新项目地址：http://sourceforge.net/p/dex2jar/
使用：
文档地址：http://sourceforge.net/p/dex2jar/wiki/UserGuide/; http://sourceforge.net/p/dex2jar/wiki/Faq/
版本：原地址版本为0.0.9.5,新地址版本为2.0
要求环境：JDK7
// For Linux, Mac OSX, Cygwin
sh /home/panxiaobo/dex2jar-version/d2j-dex2jar.sh /home/panxiaobo/someApk.apk
// For Windows
C:\dex2jar-version\d2j-dex2jar.bat someApk.apk
然后可以使用jd-gui查看产生的jar文件

jd-gui

介绍：直接通过jar查看源码
项目地址：http://jd.benow.ca/
使用：略

国人开发的反编译集成工具AndroidKiller

t挺强大的，用过就知道了。
地址：http://pan.baidu.com/s/1c06fupE

APK去广告与汉化

APK去广告：
先反编译
apktool d old.apk
修改布局文件，以去谷歌广告为例：（只适用于非UI硬编码）
    <com.google.ads.AdView android:id="@id/adView" android:background="#00000000" android:layout_width="0.0dip" android:layout_height="0.0dip" ads:adSize="BANNER" ads:adUnitId="a676bb9042fc40cf" ads:loadAdOnCreate="true" ads:testDevices="emulator-5554" android:visibility="gone" />
方式一：android:layout_width="0.0dip" android:layout_height="0.0dip" 
方式二：添加android:visibility="gone"属性

重新编译生成apk：
apktool b old.out

` 重新编译生成apk之后需要进行签名才能安装或者使用，否则无法安装，或者出现FC。 `

签名：

java -jar signapk.jar testkey.x509.pem testkey.pk8 update.apk update_signed.apk

汉化：步骤相同，只是需要改改value目录下的相关值文件。
APK优化与签名

因为优化会影响签名，所以我们需要注意执行顺序：先优化后签名。
优化：
zipalign工具使用说明：
Usage: zipalign [-f] [-v] <align> infile.zip outfile.zip
       zipalign -c [-v] <align> infile.zip


  <align>: alignment in bytes, e.g. '4' provides 32-bit alignment
  -c: check alignment only (does not modify file)
  -f: overwrite existing outfile.zip
  -v: verbose output

zipalign -c -v -f 4 new.apk

签名：
java -jar signapk.jar testkey.x509.pem testkey.pk8 update.apk update_signed.apk
