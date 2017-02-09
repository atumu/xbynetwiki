title: python学习笔记105 

#  Python学习笔记之PyQt4之定制组件 
` QWidget类是所有用户界面对象的基类。继承QObject和QPaintDevice。 `
部件（Widget）是用户界面的基本元素，**它不仅负责接受窗口系统传递来的鼠标、键盘以及其他事件信息，还负责在屏幕上把自己给绘制出来**（每一个部件都是矩形的、以Z轴排列，能够被其父部件或上层部件遮盖）。
**顶层部件是一种特殊的部件，它没有父部件**。一般来说，顶层部件拥有框架和标题栏（当然了，通过适当的窗体标识能够创建没有这些玩意的顶层部件）。
` 在Qt中，QMainWindow和各种QDialog的子类是最常见的顶层部件类。 `
**每个部件的构造器接受一个或两个标准参数。**
**QWidget *parent = 0 指定窗口部件的父部件。如果为0（默认），新的窗口部件将是一个顶层部件部件。如果不是，它将会是指定parent部件的一个子部件，并且限制在其父部件（parent部件）的范围内显示**（除非你指定Qt.Window作为视窗标识）。
**Qt.WindowFlags f = 0 (若可用) 设置视窗标识，默认设置对几乎所有窗口部件都是适用，但是在某些情况下，你需要使用其它的窗体标识，比如一个没有使用系统框架的顶层部件，你必须使用特定的标记。**
很奇怪的是，PYQT里面没有QT里面所讲的第三个参数  const char *name = 0 新窗口部件的窗口部件名称。
QWidget有很多成员函数，但是其中一些函数没有直接的功能。如，QWidget的font属性，QWidget并不使用，而是需要继承QWidget的子类提供实际的功能，比如QLabel、 QPushButton、 QListWidget和 QTabWidget。

顶层和子部件
没有父亲的部件总是独立的窗体（顶层部件）。对于这些部件来说，` setWindowTitle()和setWindowIcon() `函数分别设置窗体标题栏和图标。
非窗体部件为子部件，在其父部件内显示。QT提供的大多数部件主要用于子部件类型。例如按钮，你把它把它作为顶级窗口部件也是可能的，但绝大多数人喜欢把他们的按钮放到其它按钮当中，比如QDialog。

复合部件
**当一个部件用于当作容器来组织一定数量的子部件，这个部件就叫做复合部件。复合部件（例如QFrame）可以通过特定的可视属性并添加子部件来创建，并通过层来进行布局管理。**复合部件也可通过标准部件的子类化来创建（例如QFrame 或 QWidget），并在子类的构造器中添加所需要的层和子部件。QT提供的大部分范例使用的是第二种方法。

定制部件与绘制过程
Since QWidget is a subclass of QPaintDevice, subclasses can be used to display custom content that is composed using a series of painting operations with an instance of the QPainter class. This approach contrasts with the canvas-style approach used by the Graphics View Framework where items are added to a scene by the application and are rendered by the framework itself. [这段还没琢磨明白]
` 部件都是用自建的paintEvent()函数来完成所有的绘制工作。 当部件需要重绘时（不论外部变化或应用程序的要求），paintEvent()函数将被调用。 `范例Analog Clock 展示了简单的绘制事件处理方法。

默认尺寸和尺寸约定
**当实现一个新的部件时，最好是重载部件的` sizeHint() `函数为部件合理设置默认尺寸，并通过` setSizePolicy() `函数设置正确的部件尺寸约定。**
默认情况下，不提供尺寸信息的复合部件根据其子部件的空间要求确定大小尺寸。
尺寸约定（Size Police）能让布局管理系统为您提供良好的默认行为，使其他部件轻松的容纳和组织你的部件。
尺寸约定默认sizeHint()告知的尺寸大小是该部件的最佳尺寸（对于许多部件来说，这足够了）
提升：顶层部件的尺寸默认为桌面高度和宽度的2/2，你可以通过resize()手动调整尺寸大小。  
PyQt4 有丰富的组件。但是不可能提供所有的组件。PyQt4 中仅仅提供最常用的组件，像按钮，文本框，滑块等。如果我们需要特殊的组件，我们必须要自己创建。
自定制组件可以使用工具包画制工具创建。有两种可能，一个程序员可以修改或提升一个已存在的工具，或是从零开始创建。

Burning widget
这是一个组件，我们可以在 Nero，K3B 或其它 CD/DVD 刻录软件。
```

import sys
from PyQt4 import QtGui, QtCore
class Communicate(QtCore.QObject):
    updateBW = QtCore.pyqtSignal(int)
class BurningWidget(QtGui.QWidget):
    def __init__(self):
        super(BurningWidget, self).__init__()
        self.initUI()
    def initUI(self):
        self.setMinimumSize(1, 30)
        self.value = 75
        self.num = [75, 150, 225, 300, 375, 450, 525, 600, 675]
    def setValue(self, value):
        self.value = value
    def paintEvent(self, e):
        qp = QtGui.QPainter()
        qp.begin(self)
        self.drawWidget(qp)
        qp.end()
    def drawWidget(self, qp):
        font = QtGui.QFont('Serif', 7, QtGui.QFont.Light)
        qp.setFont(font)
        size = self.size()
        w = size.width()
        h = size.height()
        step = int(round(w / 10.0))
        till = int(((w / 750.0) * self.value))
        full = int(((w / 750.0) * 700))
        if self.value >= 700:
            qp.setPen(QtGui.QColor(255, 255, 255))
            qp.setBrush(QtGui.QColor(255, 255, 184))
            qp.drawRect(0, 0, full, h)
            qp.setPen(QtGui.QColor(255, 175, 175))
            qp.setBrush(QtGui.QColor(255, 175, 175))
            qp.drawRect(full, 0, till-full, h)
        else:
            qp.setPen(QtGui.QColor(255, 255, 255))
            qp.setBrush(QtGui.QColor(255, 255, 184))
            qp.drawRect(0, 0, till, h)
        pen = QtGui.QPen(QtGui.QColor(20, 20, 20), 1,
            QtCore.Qt.SolidLine)
        qp.setPen(pen)
        qp.setBrush(QtCore.Qt.NoBrush)
        qp.drawRect(0, 0, w-1, h-1)
        j = 0
        for i in range(step, 10*step, step):
            qp.drawLine(i, 0, i, 5)
            metrics = qp.fontMetrics()
            fw = metrics.width(str(self.num[j]))
            qp.drawText(i-fw/2, h/2, str(self.num[j]))
            j = j + 1
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        sld = QtGui.QSlider(QtCore.Qt.Horizontal, self)
        sld.setFocusPolicy(QtCore.Qt.NoFocus)
        sld.setRange(1, 750)
        sld.setValue(75)
        sld.setGeometry(30, 40, 150, 30)
        self.c = Communicate()
        self.wid = BurningWidget()
        self.c.updateBW[int].connect(self.wid.setValue)
        sld.valueChanged[int].connect(self.changeValue)
        hbox = QtGui.QHBoxLayout()
        hbox.addWidget(self.wid)
        vbox = QtGui.QVBoxLayout()
        vbox.addStretch(1)
        vbox.addLayout(hbox)
        self.setLayout(vbox)
        self.setGeometry(300, 300, 390, 210)
        self.setWindowTitle('Burning widget')
        self.show()
    def changeValue(self, value):
        self.c.updateBW.emit(value)
        self.wid.repaint()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在这个例子中，我们有一个 QtGui.QSlider 和一个自定制组件。滑块控制了自定制组件。这个组件显示一个媒介总的容量以及剩余的容量。这里最小的值是 1，最大是 750 。如果我们到达 700 ，我们开始画红色。这是用于指示过度烧制。
组件放在窗口的底部。这通过一个 QtGui.QHBoxLayout 和 QtGui.QVBoxLayout 实现。
```

class BurningWidget(QtGui.QWidget):
    def __init__(self):
        super(BurningWidget, self).__init__()

```
这个组件基于 QtGui.QWidget 组件。
self.setMinimumSize(1, 30)
我们更改了组件最小的大小（高度）。默认的值是一点点小。
font = QtGui.QFont('Serif', 7, QtGui.QFont.Light)
qp.setFont(font)
我们用小于默认的字体。这适合于我们的需要。

size = self.size()
w = size.width()
h = size.height()
step = int(round(w / 10.0))
till = int(((w / 750.0) * self.value))
full = int(((w / 750.0) * 700))
我们动态地绘制组件。窗口越大，组件会越大。反之亦然。这就是为何我们要计算组件的大小。参数 till 决定了要画多少。此值来自于滑块组件。这是整个区域的部分值。参数 full 决定了什么时候开始绘制红色部分。注意，此处使用了浮点运算，是为更高的精度。

真正绘制时包含三个部分。我们先绘制黄色或红色和黄色的矩形。然后绘制垂直的线，主要用于分割组件。最后是绘制数字，用于指示媒介的大小。

metrics = qp.fontMetrics()
fw = metrics.width(str(self.num[j]))
qp.drawText(i-fw/2, h/2, str(self.num[j]))
我们使用字体度量来绘制文本。我们必须要知道文本的宽度来居中绘制。

```

def changeValue(self, value):
    self.c.updateBW.emit(value)
    self.wid.repaint()

```
我们移动滑块时， changeValue() 方法就被调用了。在此方法内部，我们发送了自定义的 updateBW 信号，并且带了一个参数。这个参数是当前滑块的值。这个值用于计算在 Burning 这个组件中要绘制多少。自定义的组件然后被重绘。