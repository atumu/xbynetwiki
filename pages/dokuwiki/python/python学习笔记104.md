title: python学习笔记104 

#  Python学习笔记之PyQt4之drag和drop 
在计算机中的图形界面中， drag-and-drop 是例如点击到一个虚拟对象并把它拖到另外的位置上的行为。一般来说，这可以用于很多行为，或创建两个对象间的关联。（Wikipedia）
drag 和 drop 的功能是 GUI 最有用的功能之一。它可以是用户处理复杂的工作。
一般来说，我们可以 drag 和 drop 两种东西，数据或图形对象。如果我们吧一幅图像从一个应用拖到另一个应用，我们处理的是二进制数据。如果我们在 Firefox 中拖动了一个标签，我们拖的则是一个图形组件。
##  简单的 Drag 和 Drop 
第一个例子，我们将有一个 QtGui.QLineEdit 和 QtGui.QPushButton 。我们将从行编辑区拖动文本到按钮上。
```

import sys
from PyQt4 import QtGui
class Button(QtGui.QPushButton):
    def __init__(self, title, parent):
        super(Button, self).__init__(title, parent)
        self.setAcceptDrops(True)
    def dragEnterEvent(self, e):
        if e.mimeData().hasFormat('text/plain'):
            e.accept()
        else:
            e.ignore()
    def dropEvent(self, e):
        self.setText(e.mimeData().text())
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        edit = QtGui.QLineEdit('', self)
        edit.setDragEnabled(True)
        edit.move(30, 65)
        button = Button("Button", self)
        button.move(190, 65)
        self.setWindowTitle('Simple Drag & Drop')
        self.setGeometry(300, 300, 300, 150)
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    ex.show()
    app.exec_()
if __name__ == '__main__':
    main()

```
简单的拖拽操作。
```

class Button(QtGui.QPushButton):
    def __init__(self, title, parent):
        super(Button, self).__init__(title, parent)

```
为了可以把文字拖到 QtGui.QPushButton 组件上，**我们必需要重新实现一些方法。所以我们创建了我们自己的按钮类。它从 QtGui.QPushButton 派生。**
**self.setAcceptDrops(True)**
我们开启允许接受拖入的事件。
```

def dragEnterEvent(self, e):
    if e.mimeDate().hasFormat('text/plain'):
        e.accept()
    else:
        e.ignore()

```
首先，我们重新实现了 drageEnterEvent() 方法。我们将接受特定的数据类型，此处是纯文本。
```

def dropEvent(self, e):
    self.setText(e.mimeDate().text())

```
通过重新实现 dropEvent() 方法，我们定义了放下后处理的事件。我们在这里是改变了按钮中的显示文本。
```

edit = QtGui.QLineEdit('', self)
edit.setDragEnabled(True)

```
QtGui.QLineEdit 组件有内置的拖拽操作。我们只需要调用 setDragEnabled() 激活它就可以了。
##  拖拽一个按钮组件 
下面的例子，我们将介绍如何拖拽一个按钮对象。
```

import sys
from PyQt4 import QtGui
from PyQt4 import QtCore
class Button(QtGui.QPushButton):
    def __init__(self, title, parent):
        super(Button, self).__init__(title, parent)
    def mouseMoveEvent(self, e):
        if e.buttons() != QtCore.Qt.RightButton:
            return
        mimeData = QtCore.QMimeData()
        drag = QtGui.QDrag(self)
        drag.setMimeData(mimeData)
        drag.setHotSpot(e.pos() - self.rect().topLeft())
        dropAction = drag.start(QtCore.Qt.MoveAction)
    def mousePressEvent(self, e):
        QtGui.QPushButton.mousePressEvent(self, e)
        if e.button() == QtCore.Qt.LeftButton:
            print 'press'
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.setAcceptDrops(True)
        self.button = Button('Button', self)
        self.button.move(100, 65)
        self.setWindowTitle('Click or Move')
        self.setGeometry(300, 300, 280, 150)
    def dragEnterEvent(self, e):
        e.accept()
    def dropEvent(self, e):
        position = e.pos()
        self.button.move(position)
        e.setDropAction(QtCore.Qt.MoveAction)
        e.accept()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    ex.show()
    app.exec_()
if __name__ == '__main__':
    main()

```
在这个例子中，我们有一个 QtGui.QPushButton 在窗口上。如果我们点击按钮，将在控制台上输出 ‘press’ 。而如果右击按钮并且移动，我们就可以拖拽这个按钮组件。
```

class Button(QtGui.QPushButton):
    def __init__(self, title, parent):
        super(Button, self).__init__(title, parent)

```
我们创建了一个派生自 QtGui.QPushButton 的按钮类。我们还重新实现了 QtGui.QPushButton 中的两个方法， mouseMoveEvent() 和 mousePressEvent() 。其中， mouseMoveEvent() 方法是开始拖拽处发生的地方。
```

if event.buttons() != QtCore.Qt.RightButton:
    return

```
我们决定只用鼠标右击进行拖拽。左击用于点击按钮。
mimeData = QtCore.QMimeData()
drag = QtGui.QDrag(self)
drag.setMimeData(mimeData)
drag.setHotSpot(event.pos() - self.rect().topLeft())
我们创建了一个 QDrag 对象。
dropAction = drag.start(QtCore.Qt.MoveAction)
start() 方法开始拖拽操作。
```

def mousePressEvent(self, e):
    QtGui.QPushButton.mousePressEvent(self, e)
    if e.button() == QtCore.Qt.LeftButton:
        print 'press'

```
如果点击了鼠标左键，我们在控制台上打印 ‘press’ 。注意，我们还调用了父类的 mousePressEvent() 方法。否则，我们将不会看到按钮被按下。
position = e.pos()
self.button.move(position)
在 dropEvent() 方法中，定义了当我们松开鼠标按钮停止拖拽的行为。我们找到鼠标当前的位置，并把按钮移到合适的位置。
e.setDropAction(QtCore.Qt.MoveAction)
e.accept()
我们指定了拖拽的类型。在此处是移动的行为。
