title: qt_jambi_qt_for_java 

#  Qt Jambi-QT for Java 
Qt Jambi:Qt4是一个跨平台的GUI开发框架，而 QtJambi 则是基于JVM平台的Qt移植。
Qt Jambi比较全面的移植了Qt的类库，同时还包括Qt Desinger，这意味着你可以通过Qt Desinger进行界面设计，并直接转换为Java代码（或直接在JVM平台内调用该设计的XML文件），大大的提升了UI设计的效率。
众所周知，JVM平台的Swing和SWT作为GUI开发，其界面的美观程度，以及开发难度都比较高，而Qt Jambi全面移植了Qt的QCSS系统，你可以在使用CSS3.0的基础上，对软件的界面进行任意的调整，QCSS具有完整的behavior和sub-control，使你的样式可以精确的控制到每个元件的每一个部分的每一个状态，这对比与HTML制作中，仍不完善、标准不统一的CSS现状更加先进。同时他还支持类Swing的整体外观调整，当然，在可定制样式的基础面前，整体外观的调整已不若Swing中的lnf那么重要了。
Qt Jambi还包括“Signals and Slots”的系统，并且具有完整的事件机制，以弥补了Java语言本身的一些缺陷。同时还完整的转移了Qt中许多有用的辅助库，如QHTTP、QSQL等。
Qt Jambi的底层封装方面，类似SWT。

官网：http://qtjambi.org/
官方文档：http://qtjambi.org/documentation
Github地址：https://github.com/qtjambi/qtjambi
以windows平台为例
下载后是一个exe安装包，安装后如图
![](/data/dokuwiki/java/pasted/20150903-102043.png)
两个红色框选中的jar包就是我们实际开发时需要的依赖。对，你没看错，只需要这两个jar包就可以了。
` Class-Path: qtjambi-4.8.6.jar qtjambi-native-win64-msvc2012x64-4.8.6.jar `
不用包含bin和lib下的那些.dll库，因为平台依赖的那个qtjambi-native-win64-msvc2012x64-4.8.6.jar内部已经包含了那些dll。

其余两个jar包死demo和demo-src可以作为学习资料

为demo的那个jar配置META-INF：记住最后一行换行。
Class-Path: qtjambi-4.8.6.jar qtjambi-native-win64-msvc2012x64-4.8.6.jar qtjambi-examples-src-4.8.6.jar
Main-Class: com.trolltech.launcher.Launcher

![](/data/dokuwiki/java/pasted/20150903-105112.png)
然后运行demo
![](/data/dokuwiki/java/pasted/20150903-105149.png?600x300)
可以看到我们只依赖了qtjambi-4.8.6.jar qtjambi-native-win64-msvc2012x64-4.8.6.jar即可。

参考：
http://www.oschina.net/p/qtjambi
