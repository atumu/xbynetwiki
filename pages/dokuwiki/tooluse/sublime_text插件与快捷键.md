title: sublime_text插件与快捷键 

#  sublime text插件与快捷键 
sublime Text3官方文档中文版：http://feliving.github.io/Sublime-Text-3-Documentation/
sublime Text插件下载网站：https://sublime.wbond.net/
Sublime Text install package control:https://sublime.wbond.net/installation
很好的参考文档：https://github.com/jikeytang/sublime-text

**2.安装Package Control:**
输入ctrl+`或者view->show console:输入
如不清楚，参考：https://sublime.wbond.net/installation
Soda主题：https://github.com/buymeasoda/soda-theme/
##  快捷键 
搜索类
Ctrl+F 打开底部搜索框，查找关键字。
Ctrl+shift+F 在文件夹内查找，与普通编辑器不同的地方是sublime允许添加多个文件夹进行查找，略高端，未研究。
Ctrl+H:替换
Ctrl+i增量查找,Ctrl+shif+i反向增量查找。
F3:find_next, Ctrl+F3:find_undex, shift+F3:find_prev

Ctrl+P 打开搜索框。1、输入当前项目中的文件名，**快速搜索文件**，2、输入@和关键字，查找文件中函数名，3、输入:和数字，跳转到文件中该行代码，4、输入#和关键字，查找变量名。
Ctrl+G 跳转行，自动带：，输入数字跳转到该行代码
Ctrl+R 自动带@，查找函数名。举个栗子：在函数较多的页面快速查找某个函数。
Ctrl+： 搜索变量，自动带#，输入关键字，查找文件中的变量名、属性名等。

Ctrl+Shift+P 打开命令框。场景栗子：打开命名框，输入关键字，调用sublime text或插件的功能，例如使用package安装插件。
Esc 退出光标多行选择，退出搜索框，命令框等。

多行编辑
  * ` Ctrl+Shift+L 先通过Ctrl+L或者Shift+↓选中多行，再按下此快捷键，会在每行行尾插入光标，即可同时编辑这些行，移动所有行的光标。(属于多行编辑) `
  * **Ctrl+Alt+↑ 向上添加多行光标，可同时编辑多行。**
  *** Ctrl+Alt+↓ 向下添加多行光标，可同时编辑多行。**(win下可能被占用，请按此教程解除占用http://jingyan.baidu.com/article/8065f87fdaaa0a23312498fc.html)
  * **ctrl一直按住，然后鼠标光标点，可以添加多个光标同时编辑**

选择类
**` Alt+F3 ` (类似于eclipse中的Alt+Shift+R重命名重构)**选中文本按下快捷键，即可一次性选择全部的相同文本进行同时编辑。举个栗子：快速选中并更改所有相同的变量名、函数名等。
**Ctrl+L 选中整行，继续操作则继续选择下一行**，效果和 **Shift+↓** 效果一样。
**shift+↑ 向上选中多行，shift+↓ 向下选中多行。**

**Ctrl+Shift+↑ 交换行，Ctrl+Shift+↓**
**Ctrl+Enter 在下一行插入新行。Ctrl+Shift+Enter 在上一行插入新行。**

**Ctrl+Shift+[ 先选中代码，按下此快捷键，折叠代码。
Ctrl+Shift+] 先选中代码，按下此快捷键，展开代码。**
**Ctrl+Shift+M 选择括号内的内容（继续选择父括号）**。举个栗子：快速选中删除函数中的代码，重写函数体代码或重写括号内里的内容。
**Ctrl+M 光标移动至括号内结束或开始的位置**。

**Ctrl+Shift+← 向左单位性地选中文本。(Shift+← 向左选中文本。)，Ctrl+Shift+→ 向右单位性地选中文本。（Shift+→ 向右选中文本。）**


Ctrl+K+0 展开所有折叠代码。
Ctrl+← 向左单位性地移动光标，快速移动光标。
Ctrl+→ 向右单位性地移动光标，快速移动光标。
Ctrl+D(被我自定义为ctrl+alt+d) 选中光标所占的文本，继续操作则会选中下一个相同的文本。(**默认，Ctrl+D被我自定义为删除一行**)

编辑类
Home/End移至行首/尾，**已被我自定义为(ctrl+k,ctrl+left和ctrl+k,ctrl+right)**
ctrl+home/end移至文首/末。

**Ctrl+J 合并选中的多行代码为一行**。举个栗子：将多行格式的CSS属性合并为一行。
**Ctrl+Shift+D 如果你已经选中了文本,它会复制你的选中项。否则,把光标放在行上,会复制整行。**。(Eclipse中为Ctrl+Alt+Down)
**Ctrl+/ 注释单行。
Ctrl+Shift+/ 注释多行。**
Ctrl+K,Ctrl+U 转换大写。
Ctrl+K,Ctrl+L 转换小写。
**Ctrl+F2 设置书签**
Ctrl+Z 撤销。
Ctrl+Y 恢复撤销。
Ctrl+U 软撤销，感觉和 Gtrl+Z 一样。
Ctrl+K+K 从光标处开始删除代码至行尾。（已被我废弃）
Ctrl+Shift+K 删除整行。**（已被我自定义为Ctrl+D）**
Ctrl+T 左右字母互换。
F6 单词检测拼写

显示类
ctrl+w关闭文件
Ctrl+Tab 按文件浏览过的顺序，切换当前窗口的标签页。
Ctrl+PageDown 向左切换当前窗口的标签页。
Ctrl+PageUp 向右切换当前窗口的标签页。
ctrl+Up/Down:滚屏
Alt+Shift+1 窗口分屏，恢复默认1屏（非小键盘的数字）
Alt+Shift+2 左右分屏-2列
Alt+Shift+3 左右分屏-3列
Alt+Shift+4 左右分屏-4列
Alt+Shift+5 等分4屏
Alt+Shift+8 垂直分屏-2屏
Alt+Shift+9 垂直分屏-3屏

ctrl+ 数字，可以在分屏之间切换
Ctrl+K+B 开启/关闭侧边栏。
F11 全屏模式
Shift+F11 免打扰模式
##  自定义快捷键推荐 
**首先首推多行编辑功能快捷键**：` ctrl+鼠标单击选择可同时编辑多行。 `
ctrl + Shift + P调出面板
然后你可以定义如下快捷键，eclipse风格
```

[

	{ "keys": ["alt+/"], "command": "auto_complete" }, //智能提示
	{ "keys": ["alt+/"], "command": "replace_completion_with_auto_complete", "context":
	[
	{ "key": "last_command", "operator": "equal", "operand": "insert_best_completion" },
	{ "key": "auto_complete_visible", "operator": "equal", "operand": false },
	{ "key": "setting.tab_completion", "operator": "equal", "operand": true }
	]
	},
	//{ "keys": ["ctrl+alt+d"], "command": "find_under_expand" },
	//{ "keys": ["ctrl+shift+f"], "command": "reindent" , "args": {"single_line": false}}, //代码格式化

	//{ "keys": ["ctrl+d"], "command": "run_macro_file", "args": {"file": "Packages/Default/Delete Line.sublime-macro"} },//删除一行
  	{ "keys": ["ctrl+l"], "command": "expand_selection", "args": {"to": "line"} },
  
	//{ "keys": ["ctrl+k","ctrl+left"], "command": "move_to", "args": {"to": "bol", "extend": false} },
	//{ "keys": ["ctrl+k","ctrl+right"], "command": "move_to", "args": {"to": "eol", "extend": false} }
  	{ "keys": ["alt+ctrl+b"], "command": "goto_python_definition"},
     {"keys": ["f5"], "caption": "SublimeREPL: Python", "command": "run_existing_window_command", "args": {"id": "repl_python", "file": "config/Python/Main.sublime-menu"} }



]

```

##  插件快捷键 
对于Python anaconda：
anaconda_goto :ctrl+alt+g
anaconda_find_usages:ctrl+alt+f
anaconda_doc:ctrl+alt+d
anaconda_auto_format:ctrl+alt+r

对于codeintel:
goto_python_definition:super+alt+ctrl+up
back_to_python_definition:super+alt+ctrl+left

##  sublimerepl配置快捷键 
http://www.cnblogs.com/maoxianfei/p/5709229.html
因为用sublime运行python，如果有input（）函数，ctrl+b是不能输入数据的，所以下载安装了sublimeREPL进行调试。
但是sublimeREPL没有自定义快捷键，所以只有自己设置。
首先找到sublimerepl的配置文件。
步骤：Preferences-->Browse Packages-->SublimeREPL文件夹-->config文件夹-->Python文件夹-->**Default.sublime-commands。**
这是repl的配置文件，找到你需要的命令复制下来。
粘贴到Preferences-->Key Bindings User
```

[
    {
            "keys": ["f5"],//这是自己设的快捷键
            "command": "run_existing_window_command", 
            "args":
            {
                "id": "repl_python_run",
                "file": "config/Python/Main.sublime-menu"
            }
    }
]

```
##  Terminal 调用 cmder 
配置终端路径
默认调用系统自带的PowerShell,我用的是cmder，
去package setting 找到Terminal的user setting
Cmder on Windows 
```

{
	// The command to execute for the terminal, leave blank for the OS default
	// On OS X the terminal can be set to iTerm.sh to execute iTerm
	"terminal": "C:\\Windows\\System32\\cmd.exe",

	// A list of default parameters to pass to the terminal, this can be
	// overridden by passing the "parameters" key with a list value to the args
	// dict when calling the "open_terminal" or "open_terminal_project_folder"
	// commands
	"parameters": ["/START", "%CWD%"]
}

``` 
xterm on GNU/Linux 
```

{ 
"terminal": "xterm"  //gnome-terminal
}

```
##  插件推荐 
  * IMESupport: 输入法等支持,仅windows
  * SideBarEnhancements
  * bookmarks:书签插件。
  * Alignment 对齐
  * AutoFileName .自动补全文件(目录)名
  * BracketHighlighter突出显示括号/引号
  * ColorHighlighter 背景显示16进制颜色
  * ColorPicker
  * DocBlockr 生成注释模板
  * SublimeCodeIntel 智能代码提示
  * [ctags]
  * HTML5
  * HTML-CSS-JS Prettify  , Ctrl+Shift+H格式化代码
  * View In Browser 参考[这里](http://jingyan.baidu.com/article/adc815137f9ae2f723bf73ff.html)
  * JSONLint
  * JavaScript Completions
  * jQuery
  * Bootstrap 3 Snippets
  * AutoDocstring 自动生成python文档注释
  * anaconda #Python 3 python语法支持
  * sublimeREPL #终端支持
  * PackageResourceViewer
  * RecentActiveFiles 最近打开的文件列表
  * LESS, lessCSS代码提示 、scss
  * MarkdownPreviewer – Markdown 实时预览插件
  * Emmet (ex-Zen Coding) 用过Zen-Coding的都知道, 把你节省的时间用于享受生活
  * Sublimelint让SublimeText高亮显示错误代码 参考http://jingyan.baidu.com/article/b907e627b790b146e7891c3c.html
  * Terminal这个插件可以让你在Sublime中直接使用终端打开你的项目文件夹，并支持使用快捷键。

##  Linux下Python3 on Sublime Text 3 
第一种方案：不推荐
http://stackoverflow.com/questions/23257984/python3-4-on-sublime-text-3
Select the menu **Tools > Build > New Build System** :
```

{
    "shell_cmd": "/usr/bin/env python3 ${file}",
    "selector": "source.python",
    "file_regex": "^(...*?):([0-9]*):?([0-9]*)",
    "working_dir": "${file_path}"
}

```
I saved mine in ".../Packages/User/Python3.sublime-build".

第二种方案：
http://www.cnblogs.com/net66/p/5598383.html
安装PackageResourceViewer插件
输入 Ctrl+Shift+P
输入install，选择Package Control： Install Package
选择PackageResourceViewer，安装

设置默认的 Python.sublime-build
输入 Ctrl+Shift+P
输入 resource，选择PackageResourceViewer：Open Resource
再选择Python，再再选择Python.sublime-build
编辑Python.sublime-build将"shell_cmd": "python -u \"$file\"",改为以下之一：
```

"shell_cmd": "python3 -u \"$file\"", //指定python3为.py默认编译器
"shell_cmd": "python2 -u \"$file\"", //指定python2为.py默认编译器
"shell_cmd": "python -u \"$file\"", //根据Ubuntu系统设置，看/usr/bin/python链接哪儿（ln）
"shell_cmd": "指定版本python的绝对路径 -u \"$file\"", //指定路径下的python编译器

```
使用python3的配置文件示例（Python.sublime-build）
```

{
    //"shell_cmd": "python -u \"$file\"",
    "shell_cmd": "python3 -u \"$file\"",                           //指定python3为.py默认编译器
    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    "selector": "source.python",

    "env": {"PYTHONIOENCODING": "utf-8"},

    "variants":
    [
        {
            "name": "Syntax Check",
            "shell_cmd": "python -m py_compile \"${file}\"",
        }
    ]
}

```
Ctrl+S 保存配置文件
注：有关.sublime-build的配置信息说明，可见参见[这儿](http://jaylabs.sinaapp.com/Sublime_unofficial/reference/build_systems.html#troubleshooting-build-systems)
重启Sublime Text 3
打开.py文件，Ctrl + B 即可编译执行
