title: python学习笔记114 

#  Python学习笔记之PyQt4之窗体 
##  QSplitter分割窗体 
![](/data/dokuwiki/python/pasted/20160317-105222.png)
整个对话框由3个窗口组成，各个窗口之间的大小可随意拖动改变。此实例使用QSplitter类来实现，实现代码如下所示：
```

1.# -*- coding: utf-8 -*-
2.from PyQt4.QtGui import *
3.from PyQt4.QtCore import *
4.import sys
5.
6.QTextCodec.setCodecForTr(QTextCodec.codecForName("utf8"))
7.
8.class MainWidget(QMainWindow):
9. def __init__(self,parent=None):
10. super(MainWidget,self).__init__(parent)
11. font=QFont(self.tr("黑体"),12)
12. QApplication.setFont(font)
13.
14. mainSplitter=QSplitter(Qt.Horizontal,self)
15. leftText=QTextEdit(self.tr("左窗口"),mainSplitter)
16. leftText.setAlignment(Qt.AlignCenter)
17. rightSplitter=QSplitter(Qt.Vertical,mainSplitter)
18. rightSplitter.setOpaqueResize(False)
19. upText=QTextEdit(self.tr("上窗口"),rightSplitter)
20. upText.setAlignment(Qt.AlignCenter)
21. bottomText=QTextEdit(self.tr("下窗口"),rightSplitter)
22. bottomText.setAlignment(Qt.AlignCenter)
23. mainSplitter.setStretchFactor(1,1)
24. mainSplitter.setWindowTitle(self.tr("分割窗口"))
25.
26. self.setCentralWidget(mainSplitter)
27.
28. app=QApplication(sys.argv)
29. main=MainWidget()
30. main.show()
31. app.exec_()

```
第11-12行指定显示的字体。
第14行定义一个QSplitter类对象，为主分割窗口，设定此分割窗为水平分割窜。
第15行定义一个QTextEdit类对象，并插入主分割窗口中。
第16行调用setAlignment()方法，设定TextEdit中文字的对齐方式，常用的有以下几种。
Qt.AlignLeft：左对齐。
Qt.AlignRight：右对齐。
Qt.AlignCenter：文字居中(Qt.AlignHCenter为水平居中，Qt.AlignVCenter为垂直居中)。
Qt.AlignUp：文字与顶端对齐。
Qt.AlignBottom：文字与底部对齐。
第17行定义一个右部的分割窗口，定义为垂直分割窗，并以主分割窗口为父窗口。
第18行调用的方法**setOpaqueResize(boolean)用由设定分割窗的分割条在拖动时是否为实时更新显示，若设为True则实时更新显示，**若设为False则在拖动时只显示一条灰色的精线条，在拖动到位并弹起鼠标后再显示分割条。默认设为True，这和Qt3正好相反，Qt3中默认为False。
第23行**setStretchFactor()方法用于设定可伸缩控**件，它的第一个参数指定设置的控件序号，控件序号按插入的先后次序从0起依次编号，第二个参数大于0的值表示此控件为可伸缩控件。此实例中设定右部分割窗为可伸缩控件，当整个对话框的宽度发生改变时，左部的文件编辑框宽度保持不变，右部的分割窗宽度随整个对话框大小的改变进行调整。
##  不规则窗体 
常见的窗体通常是各种方形的对话框，如前面实例中实现的所有对话框都是这样的。但有时也会需要用到非方形的窗体，如圆形，椭圆形甚至是不规则形状的对话框。
![](/data/dokuwiki/python/pasted/20160317-102629.png)
在图中所示的哆拉A梦即为一个不规则窗体，实例在不规则窗体中绘制了作为窗体形状的PNG图片，也可在不规则窗体上放置按钮等控件，可以通过鼠标左键拖动窗体，鼠标右键关闭窗体。
具体实现代码如下：
```

1.from PyQt4.QtGui import *
2.from PyQt4.QtCore import *
3.import sys
4.
5.class ShapeWidget(QWidget):
6. def __init__(self,parent=None):
7. super(ShapeWidget,self).__init__(parent)
8.
9. pix=QPixmap("image/21.png","0",Qt.AvoidDither|Qt.ThresholdDither|Qt.ThresholdAlphaDither)
10. self.resize(pix.size())
11. self.setMask(pix.mask())
12. self.dragPosition=None
13.
14. def mousePressEvent(self,event):
15. if event.button()==Qt.LeftButton:
16. self.dragPosition=event.globalPos()-self.frameGeometry().topLeft()
17. event.accept()
18. if event.button()==Qt.RightButton:
19. self.close()
20.
21. def mouseMoveEvent(self,event):
22. if event.buttons() & Qt.LeftButton:
23. self.move(event.globalPos()-self.dragPosition)
24. event.accept()
25.
26. def paintEvent(self,event):
27. painter=QPainter(self)
28. painter.drawPixmap(0,0,QPixmap("image/21.png"))
29.
30. app=QApplication(sys.argv)
31. form=ShapeWidget()
32. form.show()
33. app.exec_()

```
ShapeWidget即为此不规则窗体类，继承自QWidget类。在类中重定义的鼠标事件mousePressEvent()，mouseMoveEvent()以及绘制函数paintEvent()，使不规则窗体能用鼠标随意拖动。
第9行新建一个QPixmap对象。
第10行重设主窗体的尺寸为所读取的图片的大小。
第11行的**setMask()命令是实现不规则窗体的关键，setMask()的作用是为调用它的控件增加一个遮罩，遮住所选区域以外的部分使之看起来是透明的，它的参数可为一个QBitmap对象或一个QRegion对象，此处调用QPixmap的mask()函数获得图片自身的遮罩，为一个QBitmap对象。实例中使用的是png格式的图片，它的透明部分实际上即是一个遮罩。**
重定义鼠标按下响应函数mousePressEvent(QMouseEvent)和鼠标移动响应函数mouseMoveEvent(QMouseEvent)，使不规则窗体能响应鼠标事件，随意拖动。
在mousePressEvent函数中，首先判断按下的是否为鼠标左键，若是则保存当前鼠标点所在的位置相对于窗体左上角的偏移值dragPosition，如果按下鼠标右键，则关闭窗体。
此处的frameGeometry().topLeft()仍然表示整个窗体的左上角，而并不是所见的不规则窗体的左上角。
在mouseMoveEvent函数中，首先判断当前鼠标状态，调用event.buttons()返回鼠标的状态，若为左侧按钮则调用QWidget的move()函数把窗体移动至鼠标当前点，由于move()函数的参数指的是窗体的左上角的位置，因此要用鼠标当前点的位置减去相对窗体左上角的偏移值dragPosition。
ShapeWidget的重画函数主要完成在窗体上绘制图片的工作，此处为方便显示在窗体上绘制的即是用来确定窗体外形的PNG图片。