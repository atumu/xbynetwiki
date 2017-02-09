title: android截屏应用开发思路 

#  android截屏应用开发思路 

概述
虽然现在Android系统一般都自带了截屏功能，但是截屏还是很有趣的东西，有个时候需要在应用内提供这种便利功能。
本文不打算开发一个单独的截屏程序，只是介绍一下截屏实现的思路。

思路
其实截屏功能的实现也是靠linux shell命令来实现的。具体实现如下：
前提条件可以没有root：
String string = "/system/bin/screencap -p "+fileName+"\n\n";
Runtime.getRuntime().exec(string);