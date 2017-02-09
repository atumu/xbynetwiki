title: notepad_插件 

插件推荐

Explorer是文件浏览插件，可以快速的定位当前正在编辑的文件的位置，支持在搜索目录下文件的内容(Find in files)。收藏夹功能可以保存经常使用的目录或文件。
AutoSave 自动保存插件
CodeAligment代码对齐
Compare, 文件对比插件，可以进行简单快速的对比，不过进行复杂点的对比，我一般用WinMerge。
TagsView，文档的Class, 属性, 方法列表。比另一个叫做FunctionList的插件更好用。
RegEx Helper，在文档的中匹配正则表达式，可以用来测试正则表达式。
JSON Viewer,显示文档中选定JSON的结构。
NppAutoIndent, 自动缩进。
File Switcher一个快速切换窗口的工具，支持通过输入文件名，路径或者tab index来查找切换，可以用来替换默认的Ctrl + Tab。
Finger Text标签代码替换和文本自动完成插件，编辑器配合这个功能可以有效地提升代码的书写速度，提高自己的工作效率，例如我输入if然后按Tab键将会把if替换成一个完整的if结构，可以极大的提高效率，当然具体怎么替换是可以配置的。详细用法参考：　Finger Text

TextFx这个号称是Notepad++上面最好用的plugin，具有超强的文本处理能力，比如文本编码处理等。编程某种程度上就是文本工作，所以这个插件对开发人员应该是非常有帮助的。以前是默认安装的，现在需要自己手动安装。详细用法可以参考：http://zhibin07.iteye.com/blog/1287234

Task List自动扫描当前文档，将所有"TODO:"开头的注释都找出来，列在右边的面板中，双击可以跳转该行。这和Eclipse里的TODO功能很相似，便于标记查找没有完成的工作。

HTML Tag编辑HTML代码时比较有用，它主要的功能是匹配选择的标签，对HTML标签编码及解码，对JS编码及解码，我认为对HTML标签编码及解码是最有用的功能了。
NppExec插件让Notepad 编译运行Java、Python
HEX-Editor此插件主要提供了16进制查看与编辑的功能。
PoorMansTSqlFormatter自动格式化SQL
Quick Color Picker颜色摄取



使用

一、Notepad++自动完成代码

 Notepad++自动填充代码 在Notepad++ 中设置：设置->首选项->备份与自动完成-> 所有输入均启用自动完成->函数自动完成即可

二、配置NppExec编译执行Java代码


1.变量含义
FULL_CURRENT_PATH 文件路径名称 E:/java/HelloNpp.java
CURRENT_DIRECTORY 文件目录（不含文件名） E:/java/
FILE_NAME 文件全名称 HelloNpp.java
NAME_PART 文件名称（不含ext） HelloNpp
EXT_PART 文件扩展名 java

2.配置
写好Java文件，然后点击plugins->NppExec->Execute
输入脚本：
npp_save
cd $(CURRENT_DIRECTORY) 
javac $(NAME_PART).java 
java $(NAME_PART)
依次是：
1、cd到Java文件所在目录
2、编译Java文件
3、执行Java文件
然后点击save，名字为编译运行Java文件

3、高级选项保存操作
进入NppExec->advanced options里
然后在左下方Associated script选择我们之前的编译运行Java文件，Add/Modify然后OK

4、配置快捷键
进入setting->shortcut mapper
在plugin commands面板，双击填写快捷键。我们把上面的编译运行Java文件填写为Ctrl+F12
然后去我们的程序中按Ctrl+F12就会运行这个Java文件。

参考：
http://blog.csdn.net/wang02011/article/details/7743522
http://blog.csdn.net/borishuai/article/details/8510306
http://www.crifan.com/files/doc/docbook/rec_soft_npp/release/htmls/npp_common_plugins.html
http://www.ziliao1.com/Article/Show/91EFE7FEFE9FAD9CAEECEAE6EA9B90E9.html