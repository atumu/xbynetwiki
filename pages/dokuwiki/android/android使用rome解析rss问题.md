title: android使用rome解析rss问题 

#  Android使用ROME解析RSS问题 

rome要依赖jdom，所以我们需要两个jar包，
而单单用官网上的jar是不能满足android平台下保存RSS内容的功能，缺少类javax.beans相关类，
因此需要下载android版的rome和jdom的jar包，，
它们可以在下面的地址下载到：https://code.google.com/p/android-rome-feed-reader/downloads/list  好的，下载好之后add到项目的build path就可以用了。
本站提供下载：![](/data/dokuwiki/android/android_rss_lib.zip|android_rss_lib.zip)