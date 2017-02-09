location:dokuwiki/android/idea快捷键
createAt:2016-12-22 10:47:14
modifyAt:2016-12-22 10:47:14
author:xbynet
title: idea快捷键 

#  IDEA快捷键 
实用快捷键:
Ctrl+/ 或 Ctrl+Shift+/ 注释（/ / 或者/*...*/ ）
Ctrl+G 定位某一行
Ctrl+D 复制行,Ctrl+X 剪切，Ctrl+Y删除行
CTRL+Z 撤销 ,CTRL+SHIFT+Z  重做 
代码提示 Ctrl+Space
**智能补全：Ctrl+Shift+Space**
自动完成：Ctrl+Shift+Enter
**输入提示：Ctrl+Alt+Space**

新建行 : ctrl+enter ; shift+enter
` alt+enter ` 快速修复(错误提示),导入 
组织导入 Ctr+alt+O
格式化代码 Ctr+alt+L
大小写转化 Ctr+shift+U

**Shift+Esc**，不仅可以把**焦点移到编辑器**上，而且还可以**隐藏当前（或最后活动的）工具窗口**
**F12，把焦点从编辑器移到最近使用的工具窗口**
**Shift+F1**，要**打开**编辑器光标字符处使用的类或者方法 Java **文档的浏览器**

` Alt+Insert `可以新建类、方法等任何东西。在编辑窗口中点击可以生成构造函数、toString、getter/setter、重写父类方法等
Ctrl+W自动按语法选中代码,以及反向的Ctrl+Shift+W
` Ctrl+J ` 自动模版代码，常用的有fori/sout/psvm+Tab即可生成循环、System.out、main方法等boilerplate样板代码，用Ctrl+J可以查看所有模板。
另外，Intellij IDEA 13中加入了后缀自动补全功能(Postfix Completion)，比模板生成更加灵活和强大。例如要输入for(User user : users)只需输入user.for+Tab。再比如，要输入Date birthday = user.getBirthday();只需输入user.getBirthday().var+Tab即可。

ctr+shit+v，打开你当前至少5条的粘贴板
重构
Ctrl+Shift+Alt+T重构一切
Shift+F6 重构-重命名
ctrl+alt+v：提取为局部变量
ctrl+alt+f：提取为成员变量


类与方法大纲：
` Ctrl+Q ` 显示注释文档
**Ctrl+H打开类层次窗口**，在继承层次上跳转则用**Ctrl+B/Ctrl+Alt+B**分别对应父类或父方法定义和子类或子方法实现，
**Ctrl+F12**查看当前类的所有方法。
Ctrl+Alt+Space 类名或接口名提示
**Ctrl+P 方法参数提示**
`Ctrl+Shift+I `查询变量定义
Ctrl+Shift+Alt+N 查找类中的方法或变量
Alt+Shift+C 对比最近修改的代码
**Alt+F7**:找类或方法的使用地方，类似于调用层次。查找一个属性或方法被谁调用
Ctrl+B 快速打开光标处的类或方法 
Ctrl+Alt+B:可以查看一个类的subtype(s)。包括subclass(s) or implementation(s)
` CTRL+ALT+T `  把选中的代码放在 TRY{} IF{} ELSE{} 里
**Ctrl+O，重写override方法**
搜索：
Ctrl+R 替换文本,Ctrl+F 查找文本(再配合F3/Shift+F3前后移动到下一匹配处。)
Ctrl+N/Ctrl+Shift+N可以打开类或资源
按Shift+Shift即可在一个弹出框中搜索任何东西，包括类、资源、配置项、方法等等。
**Ctrl+Shift+F：全文检索**
Ctrl+Shift+A可以查找所有Intellij的命令，并且每个命令后面还有其快捷键
选中文本，按Alt+F3 ，逐个往下查找相同文本，并高亮显示。

移动：
Ctrl+[，]移动到前/后代码块，
Alt+ Up/Down 在方法间快速移动定位
**Ctrl+Shift+ Up/Down移动行**
Ctrl+(shift)+ +，-折叠代码就不多说了。
` Ctrl+Alt+ left/right ` 返回至上次浏览的位置
F2 或Shift+F2 高亮错误或警告快速定位

Ctrl+向上:不移动光标，往上滑屏
Ctrl+向下:不移动光标，往下滑屏
窗口:
**Alt+1 快速打开或隐藏工程面板**
**Alt+4-Run, Alt+5 debug, Alt+6 Android Monitor**
**Alt+7打开文件结构**，Alt+8：打开继承层次(Ctrl+H)
**Alt+F12打开控制台**
**Alt+F1，查找代码所在位置，包括show file in explorer,查看文件结构，定位到导航栏，定位到项目视图栏文件位置**
Ctrl+Tab切换标签页，Ctrl+E/Ctrl+Shift+E打开最近打开过的或编辑过的文件。
CTRL+F4关闭当前窗口。
Alt+left/right，切换代码视图

**Alt+Shift+Insert**进入到列模式进行按列选中。
运行：Alt+Shift+F10运行程序，Shift+F9启动调试，Ctrl+F2停止。
代码标签输入完成后，按Tab，生成代码。
选中文本，按Ctrl+Shift+F7 ，高亮显示所有该文本，按Esc高亮消失。
**Ctrl+Shift+C，复制文件路径**
Shift+F12，还原默认布局
Ctrl+Shift+F12，隐藏/恢复所有窗口
参考:http://www.cnblogs.com/bluestorm/archive/2013/05/20/3087889.html
http://www.360doc.com/content/15/1118/15/18324215_514081702.shtml