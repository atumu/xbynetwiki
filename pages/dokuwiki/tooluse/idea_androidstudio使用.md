title: idea_androidstudio使用 

#  IDEA&AndroidStudio使用 
https://www.jetbrains.com/
快捷键Ctrl + Alt + S，打开全局配置Settings。
IDEA快捷键设置 
因为IDEA默认为Ctrl + 空格。默认快捷键与Windows输入法快捷键冲突。 
Main menu -> Code -> Completion下面有两个选项： basic：一般用法为输入某个类名开头的几个字母，然后使用此处设置的快捷键，自动补全代码。 
在basic上点击鼠标右键，弹出菜单，选择remove Ctrl + 空格，这个默认快捷键。 再次在basic上点击鼠标右键，弹出菜单，选择add Keyboard Shortcut

显示工具栏及工具按钮
默认状态，IDEA不会显示工具栏及工具按钮。这样一来我相应的操作必须通过相应菜单一级级浏览查找才能使用，非常影响工具使用效率。 
显示工具栏及工具按钮，需要找到顶部视图菜单View，勾选①处的Toolbar、②处的Tool Buttons。
![](/data/dokuwiki/tooluse/pasted/20160930-141021.png)

IDEA 项目怎么删除
先关闭项目(从菜单 **File -> Close Project** 关掉此项目)，然后界面上不会是有项目例表，鼠标移到你想要删除的项目上（不要点击，一点就打开了），然后按DELETE键

**善用to do**
![](/data/dokuwiki/tooluse/pasted/20160930-140337.png)

用标识编辑过的文件
Ctrl+Alt+S- >Editor –> Editor Tabs 
在IDEA中，你需要做以下设置, 这样被修改的文件会以*号标识出来，你可以及时保存相关的文件。”Mark modifyied tabs with asterisk”

显示行号
如何显示行号：Settings->Editor->Appearance标签项，勾选Show line numbers

编码的问题
需要改三处地方为utf-8：
settings-file encoding，设置项目的默认编码 
other settings - default settings - file encoding 
改单个文件的话，打开文件，项目界面右下角有显示当前光标行号列号，右边就是当前文件编码，自己改成想要的类型。

生成序列化serialVersionUID 
Editor -> Inspections -> Java -> Serialization issues，勾选Serializable class without ‘serialVersionUID’，至此以后，在你的Java类实现java.io.Serializable接口时，使用快捷键Alt+Enter就会提示add ‘serialVersionUID’ field，自动创建serialVersionUID了。

配置Java编译版本 
Build，Execution，Deployment -> Compiler -> Java Compiler，设置Use Cimpiler为javac，Project bytecode version(leave blank for JDK default)下拉列表选中1.8(需要设置的JDK编译版本)，点击OK按钮完成设置。

**工程结构配置**
现在，我们通过快捷键Ctrl + Alt + Shift + S，打开工程结构设置。 
Project SDK：选择或创建新的JDK，可在列表中选择已创建的对应版本的JDK。New…按钮可以创建不同版本的JDK，穿件成功后会在列表中出现新创建的JDK供配置选择。如果当前还没有任何JDK被创建，则列表会显示红色的No SDK。 
**Project language level**：选择Java JDK的编译版本。在IDEA进行编译时，会检查低于此处设置的版本的语法给出相应警告或错误提示。 
Project compiler output：设置编译后的.class文件存放目录。

**更改java compiler**
选择Eclipse编译器可大大提高Build Project的效率，重新Build时最后把之前的class文件手工删除：
![](/data/dokuwiki/tooluse/pasted/20160930-144505.png)
自动编译选项：
![](/data/dokuwiki/tooluse/pasted/20160930-144522.png)

**备份IDEA全局配置**
IDEA主界面，点击菜单File，选择Export Settings，选择导出全部配置文件存放目录，点击OK按钮完成导出
导入备份的IDEA全局配置
IDEA主界面，点击菜单File，选择Import Settings，选择导出全部配置文件存放目录的settings.jar配置备份文件，点击OK按钮完成导入。

可选特性：
代码自动补全忽略大小写 
Editor -> General -> Code Completion -> Case sensitive completion -> 下拉选择 None
关闭单词拼写检查 
Editor -> Spelling -> Typo，设置Options，去掉去掉勾选 Process code、Procss literals、Process comments。
参考：http://blog.csdn.net/a910626/article/details/45314457
http://blog.csdn.net/eugeneheen/article/details/50370334
http://blog.csdn.net/uusad/article/details/38872005

##  idea新建maven项目时，mvn archetype:generate 速度缓慢 
原因
IDEA根据maven archetype的本质，其实是执行mvn archetype:generate命令，该命令执行时，需要指定一个archetype-catalog.xml文件。
该命令的参数-DarchetypeCatalog，可选值为：remote，internal  ，local等，用来指定archetype-catalog.xml文件从哪里获取。默认为remote，即从http://repo1.maven.org/maven2/archetype-catalog.xml路径下载archetype-catalog.xml文件。文件约为3-4M，下载速度很慢，导致创建过程卡住。
解决方法
解决办法很简单，即指定-DarchetypeCatalog为internal，即可使用maven默认的archetype-catalog.xml，而不用再remote下载。
首先，**` 关闭IDEA所有项目 `**，以使后续设置为默认项目设置。
然后，找到maven的runner，在VM Options输入框内，加入-DarchetypeCatalog=internal ，保存即可。
PS：` 注意右上角的灰字：for default project，而不是for current project `
![](/data/dokuwiki/tooluse/pasted/20161011-234302.png)