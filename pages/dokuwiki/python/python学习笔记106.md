title: python学习笔记106 

#  Python学习笔记之PyQt4之绘图 
绘图常用于，当我们想改变一个已存在的组件，或者是我们希望自己创建组件。为了实现绘图，我们可以使用 PyQt4 中提供的 API 。
**绘制一般由 paintEvent() 方法处理。绘制的代码放置于 ` QtGui.QPainter ` 对象的 begin() 与 end() 之间。**
##  绘制文本 
我们可以把一些 Unicode 文本绘制到屏幕上。
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.text = u'\u041b\u0435\u0432 \u041d\u0438\u043a\u043e\u043b\u0430\
\u0435\u0432\u0438\u0447 \u0422\u043e\u043b\u0441\u0442\u043e\u0439: \n\
\u0410\u043d\u043d\u0430 \u041a\u0430\u0440\u0435\u043d\u0438\u043d\u0430'
        self.setGeometry(300, 300, 280, 170)
        self.setWindowTitle('Draw text')
        self.show()
    def paintEvent(self, event):
        qp = QtGui.QPainter()
        qp.begin(self)
        self.drawText(event, qp)
        qp.end()
    def drawText(self, event, qp):
        qp.setPen(QtGui.QColor(168, 34, 3))
        qp.setFont(QtGui.QFont('Decorative', 10))
        qp.drawText(event.rect(), QtCore.Qt.AlignCenter, self.text)
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在这里，我们绘制了一些 Azbuka 文字。这段文本垂直和水平上都对齐了。
def paintEvent(self, event):
**绘制在 paint 事件中完成。**
```

qp = QtGui.QPainter()
qp.begin(self)
self.drawText(event, qp)
qp.end()

```
QtGui.QPainter 类可用于底层的绘制。所有绘制的方法要处于 begin() 与 end() 之间。真正的绘制委托给了 drawText() 方法。
qp.setPen(QtGui.QColor(168, 34, 3))
qp.setFont(QtGui.QFont('Decorative', 10))
这里我们定义了画笔及字体。
qp.drawText(event.rect(), QtCore.Qt.AlignCenter, self.text)
drawText() 方法就把文本绘制到窗口上了。 rect() 方法返回所需的矩形。

##  绘制点 
一个点是最简单的图形对象。在窗口上它就是小圆点。
```

import sys, random
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.setGeometry(300, 300, 280, 170)
        self.setWindowTitle('Points')
        self.show()
    def paintEvent(self, e):
        qp = QtGui.QPainter()
        qp.begin(self)
        self.drawPoints(qp)
        qp.end()
    def drawPoints(self, qp):
        qp.setPen(QtCore.Qt.red)
        size = self.size()
        for i in range(1000):
            x = random.randint(1, size.width()-1)
            y = random.randint(1, size.height()-1)
            qp.drawPoint(x, y)
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这个例子中，我们随机画了 1000 个红点。
qp.setPen(QtCore.Qt.red)
这里设置了红色的画笔。我们使用预定义的颜色常量。
size = self.size()
每次我们重新调整了窗口，一个绘图信号就被生成。我们通过窗口的 size() 方法获取当前的大小。我们使用这个大小绘制点的分布。
qp.drawPoint(x, y)
我们使用 drawPoint() 绘制了点。

##  颜色 
一个颜色对象代表红色，绿色和蓝色（RGB）的混合。合法的 RGB 值范围在 0 到 255 。我们可以以多种方式定义颜色。最常见的就是使用 RGB 十进制或十六进制。我们也可以使用 RGBA 值，分别代表 Red，Green，Blue，Alpha。这里额外的信息表示透明度。Alpha值255定义为完全不透明，0 代表完全透明。
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.setGeometry(300, 300, 350, 100)
        self.setWindowTitle('Colors')
        self.show()
    def paintEvent(self, e):
        qp = QtGui.QPainter()
        qp.begin(self)
        self.drawRectangles(qp)
        qp.end()
    def drawRectangles(self, qp):
        color = QtGui.QColor(0, 0, 0)
        color.setNamedColor('#d4d4d4')
        qp.setPen(color)
        qp.setBrush(QtGui.QColor(200, 0, 0))
        qp.drawRect(10, 15, 90, 60)
        qp.setBrush(QtGui.QColor(255, 80, 0, 160))
        qp.drawRect(130, 15, 90, 60)
        qp.setBrush(QtGui.QColor(25, 0, 90, 200))
        qp.drawRect(250, 15, 90, 60)
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在我们的例子里，我们画了三个带色的矩形。
color = QtGui.QColor(0, 0, 0)
color.setNamedColor('#d4d4d4')
这里我们定义了使用十六进制的颜色。
qp.setBrush(QtGui.QColor(200, 0, 0))
qp.drawRect(10, 15, 90, 60)
我们定义了一把刷子和画了一个矩形。一把刷子是一个图像对象，用于画形状的背景。 drawRect() 方法接受四个参数，第一二个为 x 和 y 值，第三四个是矩形的宽度和高度。这个方法用当前的画笔和刷子画了一个矩形。

##  QtGui.QPen 
**QtGui.QPen 是一个图像对象。主要用于画线，曲线，以及矩形，椭圆，多边形或其他形状的外形。**
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.setGeometry(300, 300, 280, 270)
        self.setWindowTitle('Pen styles')
        self.show()
    def paintEvent(self, e):
        qp = QtGui.QPainter()
        qp.begin(self)
        self.drawLines(qp)
        qp.end()
    def drawLines(self, qp):
        pen = QtGui.QPen(QtCore.Qt.black, 2, QtCore.Qt.SolidLine)
        qp.setPen(pen)
        qp.drawLine(20, 40, 250, 40)
        pen.setStyle(QtCore.Qt.DashLine)
        qp.setPen(pen)
        qp.drawLine(20, 80, 250, 80)
        pen.setStyle(QtCore.Qt.DashDotLine)
        qp.setPen(pen)
        qp.drawLine(20, 120, 250, 120)
        pen.setStyle(QtCore.Qt.DotLine)
        qp.setPen(pen)
        qp.drawLine(20, 160, 250, 160)
        pen.setStyle(QtCore.Qt.DashDotDotLine)
        qp.setPen(pen)
        qp.drawLine(20, 200, 250, 200)
        pen.setStyle(QtCore.Qt.CustomDashLine)
        pen.setDashPattern([1, 4, 5, 4])
        qp.setPen(pen)
        qp.drawLine(20, 240, 250, 240)
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在此处，我们画了六条线。这些线以六种不同的样式绘制。五种是预先定义的。最后一条线是自定义的。
pen = QtGui.QPen(QtCore.Qt.black, 2, QtCore.Qt.SolidLine)
我们创建了一个 QtGui.QPen 对象。它的颜色是黑色的，其宽度设为两个像素。所以我们可以看到笔之间的区别。 QtCore.Qt.SolidLine 是一种预定义的样式。
pen.setStyle(QtCore.Qt.CustomDashLine)
pen.setDashPattern([1, 4, 5, 4])
qp.setPen(pen)
这里我们定义了笔的自定义样式。我们设置为 QtCore.Qt.CustomDashLine 样式，并调用了 setDashPattern() 方法。这个有数字的列表定义了样式。列表的长度必须是偶数。奇数位定义了划线，而偶数则是空白。数字越大，那么它们就越大。我们这里是 1px 划线 4px 空白 5px 划线 4px 空白。

##  QtGui.QBrush 
**QtGui.QBrush 是一个图像对象。它用于绘制图像形状的背景，比如矩形，椭圆，或多边形。刷子可以有三种不同的形状。**
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.setGeometry(300, 300, 355, 280)
        self.setWindowTitle('Brushes')
        self.show()
    def paintEvent(self, e):
        qp = QtGui.QPainter()
        qp.begin(self)
        self.drawBrushes(qp)
        qp.end()
    def drawBrushes(self, qp):
        brush = QtGui.QBrush(QtCore.Qt.SolidPattern)
        qp.setBrush(brush)
        qp.drawRect(10, 15, 90, 60)
        brush.setStyle(QtCore.Qt.Dense1Pattern)
        qp.setBrush(brush)
        qp.drawRect(130, 15, 90, 60)
        brush.setStyle(QtCore.Qt.Dense2Pattern)
        qp.setBrush(brush)
        qp.drawRect(250, 15, 90, 60)
        brush.setStyle(QtCore.Qt.Dense3Pattern)
        qp.setBrush(brush)
        qp.drawRect(10, 105, 90, 60)
        brush.setStyle(QtCore.Qt.DiagCrossPattern)
        qp.setBrush(brush)
        qp.drawRect(10, 105, 90, 60)
        brush.setStyle(QtCore.Qt.Dense5Pattern)
        qp.setBrush(brush)
        qp.drawRect(130, 105, 90, 60)
        brush.setStyle(QtCore.Qt.Dense6Pattern)
        qp.setBrush(brush)
        qp.drawRect(250, 105, 90, 60)
        brush.setStyle(QtCore.Qt.HorPattern)
        qp.setBrush(brush)
        qp.drawRect(10, 195, 90, 60)
        brush.setStyle(QtCore.Qt.VerPattern)
        qp.setBrush(brush)
        qp.drawRect(130, 195, 90, 60)
        brush.setStyle(QtCore.Qt.BDiagPattern)
        qp.setBrush(brush)
        qp.drawRect(250, 195, 90, 60)
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在这里，我们画了九个不同的矩形。
brush = QtGui.QBrush(QtCore.Qt.SolidPattern)
qp.setBrush(brush)
qp.drawRect(10, 15, 90, 60)
我们定义了一个刷子对象，并设置给 painter 对象。然后通过 drawRect() 方法画了矩形。
