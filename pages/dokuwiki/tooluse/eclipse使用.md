title: eclipse使用 

#  Eclipse10个高效的快捷键 

1. ctrl+shift+r：打开资源
这可能是所有快捷键组合中最省时间的了。这组快捷键可以让你打开你的工作区中任何一个文件，而你只需要按下文件名或mask名中的前几个字母，比如applic*.xml。美中不足的是这组快捷键并非在所有视图下都能用。
2. ctrl+o：快速outline
如果想要查看当前类的方法或某个特定方法，但又不想把代码拉上拉下，也不想使用查找功能的话，就用ctrl+o吧。它可以列出当前类中的所有方法及属性，你只需输入你想要查询的方法名，点击enter就能够直接跳转至你想去的位置。
3. ctrl+e：快速转换编辑器
这组快捷键将帮助你在打开的编辑器之间浏览。使用ctrl+page down或ctrl+page up可以浏览前后的选项卡，但是在很多文件打开的状态下，ctrl+e会更加有效率。
4. ctrl+2，L：为本地变量赋值
开发过程中，我常常先编写方法，如Calendar.getInstance()，然后通过ctrl+2快捷键将方法的计算结果赋值于一个本地变量之上。这样我节省了输入类名，变量名以及导入声明的时间。Ctrl+F的效果类似，不过效果是把方法的计算结果赋值于类中的域。
5. alt+shift+r：重命名
重命名属性及方法在几年前还是个很麻烦的事，需要大量使用搜索及替换，以至于代码变得零零散散的。今天的Java IDE提供源码处理功能，Eclipse也是一样。现在，变量和方法的重命名变得十分简单，你会习惯于在每次出现更好替代名称的时候都做一次重命名。要使用这个功能，将鼠标移动至属性名或方法名上，按下alt+shift+r，输入新名称并点击回车。就此完成。如果你重命名的是类中的一个属性，你可以点击alt+shift+r两次，这会呼叫出源码处理对话框，可以实现get及set方法的自动重命名。
6. alt+shift+l以及alt+shift+m：提取本地变量及方法
源码处理还包括从大块的代码中提取变量和方法的功能。比如，要从一个string创建一个常量，那么就选定文本并按下alt+shift+l即可。如果同一个string在同一类中的别处出现，它会被自动替换。方法提取也是个非常方便的功能。将大方法分解成较小的、充分定义的方法会极大的减少复杂度，并提升代码的可测试性。
7. shift+enter及ctrl+shift+enter
Shift+enter在当前行之下创建一个空白行，与光标是否在行末无关。Ctrl+shift+enter则在当前行之前插入空白行。
8. Alt+方向键
这也是个节省时间的法宝。这个组合将当前行的内容往上或下移动。在try/catch部分，这个快捷方式尤其好使。
9. ctrl+m
大显示屏幕能够提高工作效率是大家都知道的。Ctrl+m是编辑器窗口最大化的快捷键。
10. ctrl+.及ctrl+1：下一个错误及快速修改
ctrl+.将光标移动至当前文件中的下一个报错处或警告处。这组快捷键我一般与ctrl+1一并使用，即修改建议的快捷键。新版Eclipse的修改建议做的很不错，可以帮你解决很多问题，如方法中的缺失参数，throw/catch exception，未执行的方法等等。
更多快捷键组合可在Eclipse按下ctrl+shift+L查看。

#  eclipse快捷键总结 
编辑：
 ctrl+Z/Y
 开启智能insert模式：ctrl+shift+insert
 F2显示javadoc提示
 内容帮助：ctrl+space
 上下文信息：ctrl+shift+space
 代码补全:alt+/
 快速修复：ctrl+1
 行注释ctrl+/;
 块注释ctrl+shift+/；取消块注释ctrl+shit+
 代码折叠：ctrl+(shift)++/=(自定义)
 代码格式化：ctrl+shift+F
 打开类型层次：F4
 打开调用层次：ctrl+alt+H
 打开附加的javadoc:shift+F2
 打开文件结构：ctrl+F3
 打开继承层次：ctrl+T
 打开大纲：ctrl+O
 显示位置：shift+alt+B
 添加导入ctlr+shift+M,组织导入：ctrl+shift+O
 围绕：shift+alt+Z
 添加javadoc:shift+alt+J
 返回上次编辑地方：ctrl+Q
Ctrl+Shift+X 把当前选中的文本全部变味小写    Ctrl+Shift+Y 把当前选中的文本全部变为小写

跳转与搜索：
 查询：ctrl+F,查找下一个：ctrl+(shift)+K
 增量查找：ctrl+(shift)+J
 打开搜索对话框：ctrl+H
 搜索ctrl+alt+G;ctrl+shift+G;ctrl+G
 打开搜索资源ctrl+shift+R
 查找类：ctrl+shift+T

跳转到下一个错误：ctrl+,
跳转到行：ctrl+L
跳转到上、下一个成员：ctrl+shit+up/down
跳转到匹配的括号：ctrl+shift+P
跳转到源码声明：F3
全局 后退历史记录 Alt+←  全局 前进历史记录 Alt+→
全局 工作区中的声明 Ctrl+G     全局 工作区中的引用 Ctrl+Shift+G


移动与选择、插入、删除：
 移动ctrl+HOME/END;HOME/END;ctrl+left/Right/up/down
 移动行：alt+up/down:
 选择代码块：shift+Alt+up/down与shift+alt+left/right结合使用效果更好

 选择：shift+home/end;shift+ctrl+left/right
 插入shift+ctrl+enter;shift+enter
 复制行：ctrl+alt+up/down
 删除行：ctrl+D
 删除单词：ctrl+backspace/delete
 删除至行尾：shift+ctrl+del
 链接行：ctrl+alt+J
 大小写转换：shift+ctrl+Y/X

重构：
显示重构菜单shift+alt+T
重命名：shift+alt+R
移动：shift+alt+V
更改方法签名：shift+alt+C
提取方法：shift+alt+M
提取本地变量：shift+alt+L
内联：shift+alt+I
重构undo：shift+alt+Z

运行于调试：
运行：ctrl+F11,运行java程序：shift+alt+X j;
运行junit测试：shift+alt+X T
调试java程序：shift+alt+D J
调试junit测试：shift+alt+D T
设置断点：shift+ctrl+B
恢复：F8,中断：ctrl+F2
单步进入：F5;单步进入选择：ctrl+F5
单步跳过：F6 单步返回：F7
强制返回：shift+alt+F
运行至行：ctrl+R
显示所有实例：shift+ctrl+N

文件与窗口操作：
新建ctrl+N
弹出新建菜单：shift+alt+N
关闭：ctrl+W,关闭全部：ctrl+shift+W
窗口最大化：ctrl+M
保存：ctrl+S,保存全部：ctrl+shift+S
重命名：F2 刷新：F5  属性：alt+enter
切换编辑器,很实用：ctrl+E
切换标签：ctrl+pgUp/pgDn

其他：
构建全部：ctrl+B
显示快捷键帮助：shift+ctrl+L


smart insert mode,智能插入模式，即输入左括号，自动插入右括号；输入左引号自动出现右引号；换行自动缩进；等等；很有用，平时都应启用它。

代码缩写：
java:
try/catch
test/test3
do/else/elseif/for/foreach/switch/while/if/ifelse
main/runnable/synchronized
sysout/syserr
psfi public static final
psfs public static final String
———-
SQL:
table/update
———-
HTML(只适合于myeclipse)
comment注释
dl/ol/ul/table
tr/CD/ahref/html(自定义)
—————-
javascript(只适合于myeclipse)
catch/do/else/elseif/for/forin/
function/if/ifelse/switch/try/while
——————————-
maven:(只适合于myeclipse)
execution
javac plugin/jetty plugin
profile/m2e profile
project
repository
war plugin
dep/deps/pl(自定义)

#  问题记录与解决 
##  Eclipse开启代码自动提示功能 
参考：http://jingyan.baidu.com/article/bad08e1e8d719809c85121f6.html
Eclipse代码里面的代码提示功能默认是关闭的，只有输入“.”的时候才会提示功能，下面说一下如何修改eclipse配置，开启代码自动提示功能。
Eclipse  -> Window -> Perferences
Java -> Editor -> Content Assist
在右边最下面一栏找到 auto-Activation
下面有三个选项，找到第二个“Auto activation triggers for Java：”选项 
在其后的文本框中会看到一个“.”存在。这表示：只有输入“.”之后才会有代码提示和自动补全，我们要修改的地方就是这里。把该文本框中的“.”换掉，换成“abcdefghijklmnopqrstuvwxyz.”，这样，你在Eclipse里面写Java代码就可以做到按“abcdefghijklmnopqrstuvwxyz.”中的任意一个字符都会有代码提示。
![](/data/dokuwiki/tooluse/pasted/20150512-061605.png)
##  关于eclipse工具栏无法找到ADT中AVD,Android Lint等工具图标的解决办法 
最近发现eclipse安装ADT插件后工具栏上没有android相关工具的图标。
解决办法：
Window-customize perspective-Command Groups Availability然后勾选android相关选项即可。
##  关于Eclipse无法生成android R.java的一种情况的解决 
今天导入一个项目时出现了gen目录下只有buildconfig.java文件而没有R.java。project clean以及删除gen目录完全没用。这种情况产生原因有很多。
找了半天，原因就是该项目的target sdk version与eclipse默认的不一致。右键项目properties在android选项中更改默认的Project build path即可。
##  Android Eclipse中Unable to execute dex: Multiple dex files define解决 
解决办法在http://stackoverflow.com/questions/7870265/unable-to-execute-dex-multiple-dex-files-define-lcom-myapp-rarray
也就是Build Path->Order and Export->然后将Android Private Libraries前面那个钩去掉->点击OK。
其他问题记录：

同样的问题在AndroidStudio中也可能出现解决方式：
AndroidStudio中出现 Android Duplicate files copied in APK解决：
打开项目下面的 build.gradle 文件，在 android 代码块中添加下面代码
```

android{  
....  
packagingOptions {  
        exclude 'META-INF/DEPENDENCIES'  
        exclude 'META-INF/NOTICE'  
        exclude 'META-INF/LICENSE'  
        exclude 'META-INF/LICENSE.txt'  
        exclude 'META-INF/NOTICE.txt'  
    }  
}  

```
详情参考：http://stackoverflow.com/questions/27668156/execution-failed-for-task-appdexdebug-not-sure-may-be-a-dependency-issue-bet
##  关于Eclipse无法生成android R.java的一种情况的解决 
今天导入一个项目时出现了gen目录下只有buildconfig.java文件而没有R.java。project clean以及删除gen目录完全没用。这种情况产生原因有很多。
找了半天，原因就是该项目的target sdk version与eclipse默认的不一致。右键项目properties在android选项中更改默认的Project build path即可。