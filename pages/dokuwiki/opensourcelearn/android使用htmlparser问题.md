title: android使用htmlparser问题 

#  android使用htmlparser问题 

直接使用会报错，提示缺少java.beans组件。而这些组件android不给提供。
解决方案：
第一步：
android下的javax.beans组件移植包：https://code.google.com/p/openbeans/加入到项目依赖
第二步：
修改htmlparser源码，将引用javax.beans.*的转换为:引用：com.googlecode.openbeans.*

本人已改好源码下载：![](/data/dokuwiki/opensourcelearn/htmlparser_android_fork.zip|htmlparser_android.zip)