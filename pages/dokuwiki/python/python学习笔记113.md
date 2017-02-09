title: python学习笔记113 

#  Python学习笔记之PyQt4之启动画面SplashScreen 
多大数应用程序启动时都会在程序完全启动时显示一个启动画面，在程序完全启动后消失。程序启动画面可以显示一些有关产品的信息，让用户在等待程序启动的同时了解有关产品的功能，也是一个宣传的方式。
` QSplashScreen类 `提供了在程序启动过程中显示的启动画面的功能。本实例实现一个出现程序启动画面的例子。
当运行程序时，在显示屏的中央出现一个启动画面，经过一段时间，应用程序完成初始化工作后，启动画面隐去，出现程序的主窗口界面，如下图所示。
```

from PyQt4.QtGui import *
from PyQt4.QtCore import *
import sys
class MainWindow(QMainWindow):
def __init__(self,parent=None):
super(MainWindow,self).__init__(parent)
self.setWindowTitle("Splash Example")
edit=QTextEdit()
 edit.setText("Splash Example")
 self.setCentralWidget(edit)

 self.resize(600,450)

 QThread.sleep(3)

 app=QApplication(sys.argv)

 splash=QSplashScreen(QPixmap("image/23.png"))
 splash.show()
 app.processEvents()
 window=MainWindow()
 window.show()
 splash.finish(window)

 app.exec_()

```
第19行利用QPixmap对象创建一个QSplashScreen对象。
第20行调用show()函数显示此启动图片。
第21行调用processEvents()使程序在显示启动画面的同时仍能响应鼠标等其他事件。
第22,23行正常创建主窗体对象，并调用show()函数显示。
第24行调用QSplashScreen类的finish()函数，表示在主窗体对象初始化完成后，结束启动画面。
第26行正常地调用exec_()函数。

