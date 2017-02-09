title: tomcat问题记录 

#  tomcat问题记录 
**1、非安装版_Tomcat不能手动启动解决**
http://www.cnblogs.com/xia520pi/archive/2011/11/07/2239421.html
这几天玩一下Eclipse开发Jsp，用的免安装版的Tomcat，用的没有问题，今天突然要把一个已经弄好的网站放到Tomcat的webapps里，点击bin下面的startup.bat文件手动启动，但老是出现窗口一闪就过，后来上网一查就找到了解决方案，其实就是没有指明JRE路径，问题很小，但自己记性不好，记录下来以示提醒。
![](/data/dokuwiki/tooluse/pasted/20150902-111955.png)