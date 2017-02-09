title: idea使用 

#  使用 

IDEA内存优化：

1. IDEA内存优化

因机器本身的配置而配置：

IntelliJ IDEA 8binidea.exe.vmoptions  
—————————————–  
-Xms64m  
-Xmx256m  
-XX:MaxPermSize=92m  
-ea  
-server  
-Dsun.awt.keepWorkingSetOnMinimize=true

 

快捷键：如果想修改快捷键（setting->keymap：


#  问题与解决 
##  Intellij 14 the supplied javaHome seems to be invalid 
问题原因在于：我使用的是64位jdk，而默认启动的是Intellij 32位版本，所以产生问题，我们只要启动64位版本的Intellij即可：
JetBrains\IntelliJ IDEA Community Edition 14.0.2\bin\idea64.exe

#  实用快捷键: 
Ctrl+/ 或 Ctrl+Shift+/ 注释（// 或者/*…*/ ）
删除当前行 ctr+Y 剪切当前行 ctr+X 快速修复 alt+enter (modify/cast)
代码提示 alt+/ ctr+G 定位某一行 Shift+F6 重构-重命名
Ctrl+R 替换文本 Ctrl+F 查找文本 Ctrl+E 最近打开的文件 Ctrl+J 自动代码 组织导入 ctr+alt+O

格式化代码 ctr+alt+L 大小写转化 ctr+shift+U Ctrl+Shift+Space 自动补全代码 

Ctrl+空格 代码提示 Alt+Insert可以生成构造器/Getter/Setter等 

————————–

##  IntelliJ Idea 常用快捷键列表 
Alt+回车 导入包,自动修正
Ctrl+N 查找类 Ctrl+Shift+N 查找文件
Ctrl+Alt+L 格式化代码 Ctrl+Alt+O 优化导入的类和包

Alt+Insert 生成代码(如get,set方法,构造函数等)
Ctrl+E或者Alt+Shift+C 最近更改的代码 

Ctrl+Shift+Backspace 可以跳转到上次编辑的地方Ctrl+R 替换文本 Ctrl+F 查找文本

Ctrl+Shift+Space 自动补全代码 Ctrl+空格 代码提示

Ctrl+Alt+Space 类名或接口名提示 Ctrl+P 方法参数提示

Ctrl+Shift+Alt+N 查找类中的方法或变量 Alt+Shift+C 对比最近修改的代码

Shift+F6 重构-重命名 Ctrl+Shift+先上键
Ctrl+E 最近打开的文件 Ctrl+H 显示类结构图

Ctrl+Q 显示注释文档 Alt+F1 查找代码所在位置
Alt+1 快速打开或隐藏工程面板 Ctrl+Alt+ left/right 返回至上次浏览的位置

Alt+ left/right 切换代码视图 Alt+ Up/Down 在方法间快速移动定位

Ctrl+Shift+Up/Down 代码向上/下移动。 F2 或Shift+F2 高亮错误或警告快速定位

代码标签输入完成后，按Tab，生成代码。

选中文本，按Ctrl+Shift+F7 ，高亮显示所有该文本，按Esc高亮消失。

Ctrl+W 选中代码，连续按会有其他效果

选中文本，按Alt+F3 ，逐个往下查找相同文本，并高亮显示。

Ctrl+Up/Down 光标跳转到第一行或最后一行下

Ctrl+B 快速打开光标处的类或方法 跳转到定义处这个就不用多说了，Ctrl + Alt + B 跳转到方法实现处

Ctrl + W 按一个word来进行选择操作在IDEA里的这个快捷键功能是先选择光标所在字符处的单词，然后是选择源
代码的扩展区域。举例来说，对下边这个java.text.SimpleDateFormat formatter = new java.text.SimpleDateFormat("yyyy-MM-dd HH:mm");当光标的位置在双引号内的字符串中时，会先选中这个字符串，然后是等号右边的表达式，再是整个句子
Ctrl + ]/[ 跳转到代码块结束/开始处 Ctrl+Shift+E，最近更改的文件 Ctrl+F12 可以显示当前文件的结构

