title: python学习笔记102 

#  Python学习笔记之PyQt4Dialog部件 
Dialog 窗口或 dialog 是现代 GUI 应用必不可少的一部分。一个 dialog 定义为两人或更多人间的会话。在计算机应用中，dialog 就是一个和应用说话的窗口。dialog 可以用于输入数据，修改数据，更改应用的设置等等。对话框在用户和计算机的通信间是重要的手段。
##  QtGui.QInputDialog 
QtGui.QInputDialog 提供了一个简单方便的对话框，用于获取用户输入的一个值。输入值可以是字符串，数字，或者一个列表中的一项。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.btn = QtGui.QPushButton('Dialog', self)
        self.btn.move(20, 20)
        self.btn.clicked.connect(self.showDialog)
        self.le = QtGui.QLineEdit(self)
        self.le.move(130, 22)
        self.setGeometry(300, 300, 290, 150)
        self.setWindowTitle('Input dialog')
        self.show()
    def showDialog(self):
        text, ok = QtGui.QInputDialog.getText(self, 'Input Dialog',
            'Enter your name:')
        if ok:
            self.le.setText(str(text))
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这个例子中用到了一个按钮和一个行编辑组件。按钮会显示一个输入对话框用于得到文本。而输入的文本将在行编辑组件中显示。
text, ok = QtGui.QInputDialog.getText(self, 'Input Dialog', 'Enter your name:')
这一行显示了输入对话框。第一个字符串是对话框的标题，第二个字符串则是对话框中的消息。对话框将返回输入的文本和一个布尔值。如果点击了 ok 按钮，则布尔值为 true ，否则为 false 。
if ok:self.le.setText(str(text))
从对话框中接收到的文本就被设置到行编辑组件中。
##  QtGui.QColorDialog 
QtGui.QColorDialog 用于选取颜色值。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        col = QtGui.QColor(0, 0, 0)
        self.btn = QtGui.QPushButton('Dialog', self)
        self.btn.move(20, 20)
        self.btn.clicked.connect(self.showDialog)
        self.frm = QtGui.QFrame(self)
        self.frm.setStyleSheet("QWidget { background-color: %s }"
            % col.name())
        self.frm.setGeometry(130, 22, 100, 100)
        self.setGeometry(300, 300, 250, 180)
        self.setWindowTitle('Color dialog')
        self.show()
    def showDialog(self):
        col = QtGui.QColorDialog.getColor()
        if col.isValid():
            self.frm.setStyleSheet("QWidget { background-color: %s }"
                % col.name())
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这个例子显示一个按钮和一个 QtGui.QFrame 。这个组件的背景被设为黑色。使用 QtGui.QColorDialog 可以改变其背景。
col = QtGui.QColor(0, 0, 0)
这个是 QtGui.QFrame 的初始颜色。
col = QtGui.QColorDialog.getColor()
这一行将弹出 QtGui.QColorDialog 。
```

if col.isValid():
    self.frm.setStyleSheet("QWidget { background-color: %s }"
        % col.name())

```
我们检查颜色是否合法。如果点击了取消按钮，返回的就不是合法值。如果颜色合法，我们就用样式表更改背景颜色。
##  QtGui.QFontDialog 
QtGui.QFontDialog 用于选取字体。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        vbox = QtGui.QVBoxLayout()
        btn = QtGui.QPushButton('Dialog', self)
        btn.setSizePolicy(QtGui.QSizePolicy.Fixed,
            QtGui.QSizePolicy.Fixed)
        btn.move(20, 20)
        vbox.addWidget(btn)
        btn.clicked.connect(self.showDialog)
        self.lbl = QtGui.QLabel('Knowledge only matters', self)
        self.lbl.move(130, 20)
        vbox.addWidget(self.lbl)
        self.setLayout(vbox)
        self.setGeometry(300, 300, 250, 180)
        self.setWindowTitle('Font dialog')
        self.show()
    def showDialog(self):
        font, ok = QtGui.QFontDialog.getFont()
        if ok:
            self.lbl.setFont(font)
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在我们的例子中，我们有一个按钮和一个标签。通过 QtGui.QFontDialog 我们可以改变标签的字体。
font, ok = QtGui.QFontDialog.getFont()
我们弹出一个字体对话框。 getFont() 方法将返回字体的名称和 ok 参数。如果用户点击了 OK 那么就是 True ，否则为 False 。
if ok:self.label.setFont(font)
如果我们点击了 ok，标签的字体就可能改变。
##  QtGui.QFileDialog 
QtGui.QFileDialog 是允许用户选择文件或目录的对话框。文件可以用于打开或保存。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QMainWindow):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.textEdit = QtGui.QTextEdit()
        self.setCentralWidget(self.textEdit)
        self.statusBar()
        openFile = QtGui.QAction(QtGui.QIcon('open.png'), 'Open', self)
        openFile.setShortcut('Ctrl+O')
        openFile.setStatusTip('Open new File')
        openFile.triggered.connect(self.showDialog)
        menubar = self.menuBar()
        fileMenu = menubar.addMenu('&File')
        fileMenu.addAction(openFile)
        self.setGeometry(300, 300, 350, 300)
        self.setWindowTitle('File dialog')
        self.show()
    def showDialog(self):
        fname = QtGui.QFileDialog.getOpenFileName(self, 'Open file',
                '/home')
        f = open(fname, 'r')
        with f:
            data = f.read()
            self.textEdit.setText(data)
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这个例子中有菜单栏，文本编辑区以及状态栏。菜单中的选项显示 QtGui.QFileDialog 用于选择文件。而文件的内容则载入到文本编辑区。
```

class Example(QtGui.QMainWindow):
    def __init__(self):
        super(Example, self).__init__()

```
这个例子是基于 QtGui.QMainWindow 组件，因为我们要在中心设置文本编辑区。
fname = QtGui.QFileDialog.getOpenFileName(self, 'Open file','/home')
我们弹出 QtGui.QFileDialog 。在 getOpenFileName() 方法中第一个字符串是标题。第二个则是指定对话框的工作目录。默认情况下，文件过滤为所有文件（ * ）。
```

f = open(fname, 'r')
with f:
    data = f.read()
    self.textEdit.setText(data)

```
选择的文件将被读取，并且其文件内容设置到文本编辑区。

