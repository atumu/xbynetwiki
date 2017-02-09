title: python学习笔记101 

#  Python学习笔记之PyQt4Widget部件 
组件（Widget）是一个应用最基本的构件。PyQt4 中有大量的组件。按钮，选择框，滑块，列表等等。任何一个程序员都会需要这些组件。这篇中，我们将介绍一些有用的组件，
QtGui.QCheckBox ，ToggleButton ， QtGui.QSlider , QtGui.QProcessBar 和 QtGui.QCalendarWidget、QtGui.QPixmap ， QtGui.QLineEdit ， QtGui.QSplitter 和 QtGui.QComboBox 。
##  QtGui.QCheckBox 
QtGui.QCheckBox是一个组件有两种状态，On 和 Off 。它有一个标签。选择框通常代表那些可以开启或关闭的特性，但是不会影响其他。
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        cb = QtGui.QCheckBox('Show title', self)
        cb.move(20, 20)
        cb.toggle()
        cb.stateChanged.connect(self.changeTitle)
        self.setGeometry(300, 300, 250, 150)
        self.setWindowTitle('QtGui.QCheckBox')
        self.show()
    def changeTitle(self, state):
        if state == QtCore.Qt.Checked:
            self.setWindowTitle('QtGui.QCheckBox')
        else:
            self.setWindowTitle('')
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在我们的例子中，我们创建一个选择框，用于开关窗口的标题。
cb = QtGui.QCheckBox('Show title', self)
这是 QtGui.QCheckBox 的构造器。
cb.toggle()
我们设置了窗口的标题，因此我们必须选中选择框。默认情况下，窗口的标题并没有设置，选择框也没有勾中。
cb.stateChanged.connect(self.changeTitle)
我们接着把 ` stateChanged 这个信号 `与自定义的 changeTitle() 方法连接起来。 changeTitle() 将用于开关窗口的标题。

##  ToggleButton 
**PyQt4 中没有 ToggleButton 。为了创建 ToggleButton，我们使用 QtGui.QPushButton 的特殊模式。ToggleButton 是一个按钮，有两种状态，按了与未按。**你通过点击进行开关。有很多情况下需要这样的功能。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.col = QtGui.QColor(0, 0, 0)
        redb = QtGui.QPushButton('Red', self)
        redb.setCheckable(True)
        redb.move(10, 10)
        redb.clicked[bool].connect(self.setColor)
        redb = QtGui.QPushButton('Green', self)
        redb.setCheckable(True)
        redb.move(10, 60)
        redb.clicked[bool].connect(self.setColor)
        blueb = QtGui.QPushButton('Blue', self)
        blueb.setCheckable(True)
        blueb.move(10, 110)
        blueb.clicked[bool].connect(self.setColor)
        self.square = QtGui.QFrame(self)
        self.square.setGeometry(150, 20, 100, 100)
        self.square.setStyleSheet("QWidget { background-color: %s }" %
            self.col.name())
        self.setGeometry(300, 300, 280, 170)
        self.setWindowTitle('Toggle button')
        self.show()
    def setColor(self, pressed):
        source = self.sender()
        if pressed:
            val = 255
        else: val = 0
        if source.text() == "Red":
            self.col.setRed(val)
        elif source.text() == "Green":
            self.col.setGreen(val)
        else:
            self.col.setBlue(val)
        self.square.setStyleSheet("QFrame { background-color: %s }" %
            self.col.name())
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在这里，我们创建了三个 ToggleButton 。我们还创建了一个 QtGui.QFrame 。我们先把它的颜色设为黑色。而 toggleButton 将会加入或取消相应的颜色组份。背景色取决于我们按了哪些按钮。
self.col = QtGui.QColor(0, 0, 0)
这是颜色的初始值，黑色。
redb = QtGui.QPushButton('Red', self)
redb.setCheckable(True)
redb.move(10, 10)
**为了创建 ToggleButton，我们创建了一个 QtGui.QPushButton ，并通过调用 setCheckable() 让它可以被选中。**
```

redb.clicked[bool].connect(self.setColor)

```
我们把点击的信号和自定义的方法连接起来。
source = self.sender()
我们先获取哪个按钮被开关了。
```

if source.text() == "Red":
    self.col.setRed(val)

```
如果是红色按钮，我们就相应地更改红色的部分。
```

self.square.setStyleSheet("QFrame { background-color: %s }" %
    self.col.name())

```
**为了改变背景色，我们使用了样式表。**

##  QtGui.QSlider 
**QtGui.QSlider 是一个组件，只有一个滑块。这个滑块可以向前向后拖动。这就能让我们选定一个值。**有些时候，使用滑块很自然，相对于简单的提供一个数字或是一个旋钮。** QtGui.QLable 可以显示文字或图片。**
在我们的例子中，我们有一个滑块和标签。标签将会显示图片，而滑块用于控制标签。
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        sld = QtGui.QSlider(QtCore.Qt.Horizontal, self)
        sld.setFocusPolicy(QtCore.Qt.NoFocus)
        sld.setGeometry(30, 40, 100, 30)
        sld.valueChanged[int].connect(self.changeValue)
        self.label = QtGui.QLabel(self)
        self.label.setPixmap(QtGui.QPixmap('mute.png'))
        self.label.setGeometry(160, 40, 80, 30)
        self.setGeometry(300, 300, 280, 170)
        self.setWindowTitle('QtGui.QSlider')
        self.show()
    def changeValue(self, value):
        if value == 0:
            self.label.setPixmap(QtGui.QPixmap('mute.png'))
        elif value > 0 and value <= 30:
            self.label.setPixmap(QtGui.QPixmap('min.png'))
        elif value > 30 and value < 80:
            self.label.setPixmap(QtGui.QPixmap('med.png'))
        else:
            self.label.setPixmap(QtGui.QPixmap('max.png'))
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在这个例子中，我们模拟了音量控制。通过拖拽滑块，我们可以更改标签上的图片。
sld = QtGui.QSlider(QtCore.Qt.Horizontal, self)
这里我们创建的是水平的 QtGui.QSlider 。
self.label = QtGui.QLable(self)
self.label.setPixmap(QtGui.QPixmap('mute.png'))
我们创建了一个 QtGui.QLabel 组件。我们设置一幅静音的图片在上面。
sld.valueChange[int].connect(self.changeValue)
我们把信号 valueChanged 和 changeValue() 方法联系起来。
```

if value == 0:
    self.label.setPixmap(QtGui.QPixmap('mute.png'))

```
基于滑块的值，我们设置图片到标签上。前面的代码，我们在滑块的值为零时，设置静音的图片。

##  QtGui.QProgressBar 
进度条常在处理很长的任务时使用。它是动态的，因此用户可以知道我们的任务正在处理。在 PyQt4 中，进度条可以是水平或垂直。任务被分割成很多步。程序员可以设置进度条的最小值与最大值。默认值值 0，99 。
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.pbar = QtGui.QProgressBar(self)
        self.pbar.setGeometry(30, 40, 200, 25)
        self.btn = QtGui.QPushButton('Start', self)
        self.btn.move(40, 80)
        self.btn.clicked.connect(self.doAction)
        self.timer = QtCore.QBasicTimer()
        self.step = 0
        self.setGeometry(300, 300, 280, 170)
        self.setWindowTitle('QtGui.QProgressBar')
        self.show()
    def timerEvent(self, e):
        if self.step >= 100:
            self.timer.stop()
            self.btn.setText('Finished')
            return
        self.step = self.step + 1
        self.pbar.setValue(self.step)
    def doAction(self):
        if self.timer.isActive():
            self.timer.stop()
            self.btn.setText('Start')
        else:
            self.timer.start(100, self)
            self.btn.setText('Stop')
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在我们的例子中，有一个水平的进度条和一个按钮。按钮开启和关闭进度条。
self.pbar = QtGui.QProgressBar(self)
这是 QtGui.QProgressBar 的构造器。
self.timer = QtCore.QBasicTimer()
为了激活进度条，我们使用了计时器对象。
self.timer.start(100, self)
为了载入计时器的事件，我们调用 start() 方法。这个方法有两个参数，超时与接受事件的对象。
```

def timerEvent(self, e):
    if self.step >= 100:
        self.timer.stop()
        self.btn.setText('Finished')
        return
    self.step = self.step + 1
    self.pbar.setValue(self.step)

```
每个 QtCore.QObject 及其派生类都有 timerEvent() 这个句柄。为了对计时器事件做出回应，我们重新实现事件的句柄。
```

def doAction(self):
    if self.timer.isActive():
        self.timer.stop()
        self.btn.setText('Start')
    else:
        self.timer.start(100, self)
        self.btn.setText('Stop')

```
在 doAction() 方法里，我们开启关闭计时器。

##  QtGui.QCalendarWidget 
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        cal = QtGui.QCalendarWidget(self)
        cal.setGridVisible(True)
        cal.move(20, 20)
        cal.clicked[QtCore.QDate].connect(self.showDate)
        self.lbl = QtGui.QLabel(self)
        date = cal.selectedDate()
        self.lbl.setText(date.toString())
        self.lbl.move(130, 260)
        self.setGeometry(300, 300, 350, 300)
        self.setWindowTitle('Calendar')
        self.show()
    def showDate(self, date):
        self.lbl.setText(date.toString())
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这个例子中有一个日历和一个标签。当前选中的日期会显示在标签中。
cal = QtGui.QCalendarWidget(self)
我们构造了一个日历组件。
cal.clicked[QtCore.QDate].connect(self.showDate)
如果我们从组件上选择了日期， clicked[QtCore.QDate] 信号就将被发送。我们把此信号和用户定义的 showDate() 方法连接。
```

def showDate(self, date):
    self.lbl.setText(date.toString())

```
我们通过调用 selectedDate() 方法获取选中的日期。然后我们把日期对象转为字符串并且设置到标签中。

##  QtGui.QPixmap 
QtGui.QPixmap 是可以处理图片的组件之一。它对显示图片进行了优化。在我们的例子中，我们会用 QtGui.QPixmap 来显示图片。
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        hbox = QtGui.QHBoxLayout(self)
        pixmap = QtGui.QPixmap("redrock.png")
        lbl = QtGui.QLabel(self)
        lbl.setPixmap(pixmap)
        hbox.addWidget(lbl)
        self.setLayout(hbox)
        self.move(300, 200)
        self.setWindowTitle('Red Rock')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在这个例子里，我们显示了一幅图片。
pixmap = QtGui.QPixmap("redrock.png")
我们创建了一个 QtGui.QPixmap 对象。它接受文件名作为参数。
lbl = QtGui.QLabel(self)
lbl.setPixmap(pixmap)
我们把 pixmap 放到了 QtGui.QLabel 中。

##  QtGui.QLineEdit 
QtGui.QLineEdit 是一个允许输入和编辑一行纯文本。这个组件中**可以撤销/恢复，剪切/粘贴以及拖拉**。
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.lbl = QtGui.QLabel(self)
        qle = QtGui.QLineEdit(self)
        qle.move(60, 100)
        self.lbl.move(60, 40)
        qle.textChanged[str].connect(self.onChanged)
        self.setGeometry(300, 300, 280, 170)
        self.setWindowTitle('QtGui.QLineEdit')
        self.show()
    def onChanged(self, text):
        self.lbl.setText(text)
        self.lbl.adjustSize()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这个例子显示一个行编辑区和一个标签。我们输入的就会立即在标签中显示。
qle = QtGui.QLineEdit(self)
创建了 QtGui.QLineEdit 组件。
**qle.textChanged[str].connect(self.onChanged)**
如果文本区的文本改变了，我们就调用 onChanged() 方法。
```

def onChanged(self, text):
    self.lbl.setText(text)
    self.lbl.adjustSize()

```
在 onChanged() 方法中，我们把输入的文本放到了标签中。通过**调用 adjustSize() 方法，我们把标签设置到文本的长度**。

##  QtGui.QSplitter 
QtGui.QSplitter 可以让用户控制子组件的大小，通过拖动子组件的大小。在我们的例子中，**我们的三个 QtGui.QFrame 将由两个 splitter 分割。**
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        hbox = QtGui.QHBoxLayout(self)
        topleft = QtGui.QFrame(self)
        topleft.setFrameShape(QtGui.QFrame.StyledPanel)
        topright = QtGui.QFrame(self)
        topright.setFrameShape(QtGui.QFrame.StyledPanel)
        bottom = QtGui.QFrame(self)
        bottom.setFrameShape(QtGui.QFrame.StyledPanel)
        splitter1 = QtGui.QSplitter(QtCore.Qt.Horizontal)
        splitter1.addWidget(topleft)
        splitter1.addWidget(topright)
        splitter2 = QtGui.QSplitter(QtCore.Qt.Vertical)
        splitter2.addWidget(splitter1)
        splitter2.addWidget(bottom)
        hbox.addWidget(splitter2)
        self.setLayout(hbox)
        QtGui.QApplication.setStyle(QtGui.QStyleFactory.create('Cleanlooks'))
        self.setGeometry(300, 300, 300, 200)
        self.setWindowTitle('QtGui.QSplitter')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在这个例子中，有三个框架组件，两个分割条。
topleft = QtGui.QFrame(self)
topleft.setFrameShape(QtGui.QFrame.StyledPanel)
我们使用了有样式的框架，主要用于看到边框。
splitter1 = QtGui.QSplitter(QtCore.Qt.Horizontal)
splitter1.addWidget(topleft)
splitter1.addWidget(topright)
我们创建了一个 QtGui.QSplitter 组件，并添加到两个框架。
splitter2 = QtGui.QSplitter(QtCore.Qt.Vertical)
splitter2.addWidget(splitter1)
我们也可以把一个 splitter 添加到 splitter 中。
QtGui.QApplication.setStyle(QtGui.QStyleFactory.create('Cleanlooks'))
我们使用一个简洁的样式。在有些样式中，框架是不可见的。

##  QtGui.QComboBox 
QtGui.QComboBox 允许用户从一组选项中选取一个。
```

import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.lbl = QtGui.QLabel("Ubuntu", self)
        combo = QtGui.QComboBox(self)
        combo.addItem("Ubuntu")
        combo.addItem("Mandriva")
        combo.addItem("Fedora")
        combo.addItem("Red Hat")
        combo.addItem("Gentoo")
        combo.move(50, 50)
        self.lbl.move(50, 150)
        combo.activated[str].connect(self.onActivated)
        self.setGeometry(300, 300, 300, 200)
        self.setWindowTitle('QtGui.QComboBox')
        self.show()
    def onActivated(self, text):
        self.lbl.setText(text)
        self.lbl.adjustSize()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这个例子中有一个 QtGui.QComboBox 和 QtGui.QLable 。这里有五个选项。它们是 Linux 的发行版。标签中就会显示选中的项目。
combo = QtGui.QComboBox(self)
combo.addItem("Ubuntu")
combo.addItem("Mandriva")
combo.addItem("Fedora")
combo.addItem("Red Hat")
combo.addItem("Gentoo")
我们创建一个 QtGui.QComboBox 组件并增加了五个选项。
` combo.activated[str].connect(self.onActivated) `
当选择一个选项后，我们调用了 onActivated() 方法。
```

def onActivated(self, text):
    self.lbl.setText(text)
    self.lbl.adjustSize()

```
在这个方法中，我们把选中的选项的文本放到标签中。我们还调整标签的大小。