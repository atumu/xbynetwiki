title: python学习笔记103 

#  Python学习笔记之PyQt4之菜单和工具栏 
**QtGui.QMainWindow 类提供了一个应用的主窗口。**这使得我们可以创建典型的应用框架，**包括状态栏，工具栏和菜单。**
##  状态栏 
状态栏主要用于显示状态信息。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QMainWindow):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.statusBar().showMessage('Ready')
        self.setGeometry(300, 300, 250, 150)
        self.setWindowTitle('Statusbar')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
状态栏由 QtGui.QMainWindow 帮忙创建。
self.statusBar().showMessage('Ready')
为了得到状态栏，我们调用了 QtGui.QMainWindow 的 statusBar() 方法。第一次调用创建了状态栏，随后返回 statusbar 对象。接着我们调用 showMessage 在状态栏上显示了一条消息。
##  菜单栏 
菜单栏是 GUI 应用中很常用的一部分。它是在多个菜单中命令的集合。在 console 应用中，我们需要记住命令和它们的选项。而这里，我们把很多命令按照逻辑进行分组。这就使得学习使用一个新的应用的时间可以减少。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QMainWindow):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        exitAction = QtGui.QAction(QtGui.QIcon('exit.png'), '&Exit', self)
        exitAction.setShortcut('Ctrl+Q')
        exitAction.setStatusTip('Exit application')
        exitAction.triggered.connect(QtGui.qApp.quit)
        self.statusBar()
        menubar = self.menuBar()
        fileMenu = menubar.addMenu('&File')
        fileMenu.addAction(exitAction)
        self.setGeometry(300, 300, 300, 200)
        self.setWindowTitle('Menubar')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在上面的例子中，我们创建了只有一个菜单的菜单栏。这个菜单中包含了一个 action ，如果选中后就会终止应用。我们还创建了一个状态栏。而且也可以用快捷键 Ctrl + Q 退出。
exitAction = QtGui.QAction(QtGui.QIcon('exit.png'))
exitAction.setShortcut('Ctrl+Q')
exitAction.setStatusTip('Exit application')
QtGui.QAction 是一个 action 的抽象，包括菜单栏，工具栏或者是自定义的快捷键。在上面的三行，我们创建了一个 action ，有一个指定的图标以及 ‘Exit’ 标签。而且，这个 action 定义了快捷键。第三行则是创建一个提示，当我们把鼠标指针移到菜单条目上，将在状态栏中显示相应的提示。
**exitAction.triggered.connect(QtGui.qApp.quit)**
当我们选择了这个特定的 action ，触发的信号就被发送了。这个信号和 QtGui.QApplication 的 quit() 方法联系在一起。这就终止了应用。
menubar = self.menuBar()
fileMenu = menubar.addMenu('&File')
fileMenu.addAction(exitAction)
此处三行，创建了一个菜单栏。我们往菜单栏中添加了一个名为 File 的菜单，而且，我们把 Alt + F 设为了快捷方式。然后我们再把 exitAction 放到了 fileMenu 中。
##  工具栏 
在一个应用中，菜单把所有的命令分组。而工具栏中则提供了常用命令的快捷方式。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QMainWindow):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        exitAction = QtGui.QAction(QtGui.QIcon('exit24.png'), 'Exit', self)
        exitAction.setShortcut('Ctrl+Q')
        exitAction.triggered.connect(QtGui.qApp.quit)
        self.toolbar = self.addToolBar('Exit')
        self.toolbar.addAction(exitAction)
        self.setGeometry(300, 300, 300, 200)
        self.setWindowTitle('Toolbar')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在上面的例子中，我们创建了一个简单的工具栏。这个工具栏有一个退出的 action ，当触发时，就会终止应用。
exitAction = QtGui.QAction(QtGui.QIcon('exit24.png'), 'Exit', self)
exitAction.setShortcut('Ctrl+Q')
exitAction.triggered.connect(QtGui.qApp.quit)
和前面菜单栏的例子一样，我们也创建了一个 action 对象。这个对象有一个标签，图标以及快捷方式。而且 QtGui.QMainWindow 的 quit() 方法和其触发信号关联了起来。
self.toolbar = self.addToolBar('Exit')
self.toolbar.addAction(exitAction)
这里，我们创建了工具栏，并把 action 对象放入。
