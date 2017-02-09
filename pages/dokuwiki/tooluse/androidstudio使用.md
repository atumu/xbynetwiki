title: androidstudio使用 

#  使用 
##  显示行号： 

![](/data/dokuwiki/tooluse/pasted/20150514-120614.png)
##  将adroid studio keymap改为eclipse 

##  1.android studio中的自动导入auto import: 

Eclipse自动添加import语句, 使用Ctrl + Shift + o组合, 可以自动查找java的import语句进行添加;
Android默认是Alt+Enter单个添加import语句, 可以修改IDE, 使其自动添加, 所使用的java库;
位置: Files ->Settings-> IDE Settings-> Editor -> Auto Import
选择: Add unambiguous imports on the fly, 即可.
以下为解释：XML
Show import popup，这个是用于编辑XML时，自动会弹出一个import的对话框，问你是否需要导入。
Java
Insert imports on paste:(All Ask None),这个其实就是你在复制代码的时候，对于导入的包是否需要进行询问的一个选项。
All:选择这项的时候，你黏贴的代码，有需要导入的包名时，会自动导入，不会弹提示框
ASK:选择这项的时候，你黏贴的代码，有需要导入的包名时，会弹提示框，问你要不要导入
None:选择这项的时候，你黏贴的代码，有需要导入的包名时，不会弹提示框，也不会自动导入。
Show import popup:这个是和上面的Insert imports on paste是不同的项了哈，不要混一起，这个是指当你输入的类的声明没被导入时，会弹出一个选择的对话框。但是这边需要注意下，这个选项其实是有点问题的。不管你勾还是不勾，反正对话框是不会弹出来的，在你输完类名后，声明都自动导入了。所以我估计这个可能是Android Studio的bug。
Optimize imports on fly：这个其实和快捷键Ctrl+Shift+O/Ctrl+Alt+O是一样的，就是把不用的声明移除掉。
Add unambiguous imports on the fly：这个就是自动导入功能了，当你输入类名后，声明就被自动导入了。
Exclude from Import and Completion：这个其实就是你自定义import。可以不用关注，一般来说你是用不上的。

##  2.修改主题为黑色 

Android Studio默认为白色风格，以下黑色风格设置方式：File->Settings->Appearance->Theme->Darcula
##  3.android studio与eclipse的一些概念的区别： 

Eclipse工程可以导入Android Studio运行，而反过来在Android Studio建立的工程不能在Eclipse中运行；
二者的工程结构不一样，在Eclipse中一个Project就代表一个项目工程，而在Android Studio中就和Intellij一样，一个Project代表一个工作空间，相当于Eclipse中的workspace，而在Android Studio中一个Module就相当于Eclipse中的一个Project，这个概念需要弄明白，不要混了或觉得糊涂了。
在编辑操作上，在Eclipse中编辑修改后必须手动command+s保存文件，而在Android Studio中就和Intellij一样是自动保存的，这一点和第二点和Xcode也是类似的。
工程目录上的区别，在Eclipse中src部分一般是java文件，res部分是资源文件，包括布局文件和多媒体资源等。
包括java文件和资源文件全部放到了src目录下，src目录下包括一个main文件夹，再下面就是java文件夹和res文件夹，其实这里，java文件夹就相当于Eclipse中的src，res还是那个res，
##  4.从Eclipse导入工程到Android Studio 

首先升级ADT到最新版本，目前为版本号为22（注意和ADT相关的组件最好一并升级，避免后期可能出现的错误）
选择需要从Eclipse导出的工程，右键选择Export并选择Android下的Generate Gradle Build Files
随后进入Android Studio并选择Import Project，可以看到刚刚在Eclipse中的项目图标变成了一个Android机器人图标，说明转换成功，这时候选择工程导入即可：

5.android studio中使用android Lint工具进行静态检查。
导航条上Analyze-Inspect Code。

#  问题描述与解决 

##   no debuggable application 
android studio 默认是没有开启debuggable 功能的，在tools里打开该功能即可，Tools->Android->Enable ADB Integration
参考：http://blog.csdn.net/hyr83960944/article/details/38438355
http://www.bianceng.cn/OS/extra/201409/45193.htm
http://blog.csdn.net/iplayvs2008/article/details/17715945