title: python学习笔记100 

#  Python学习笔记之PyQt4入门 
官网:http://pyqt.sourceforge.net/Docs/PyQt4/introduction.html
英文教程:http://zetcode.com/gui/pyqt4/
中文翻译:http://www.cppblog.com/mirguest/category/15952.html  http://blog.csdn.net/Kai_gai/article/category/5911577
` 最新的PyQt5教程 `:http://zetcode.com/gui/pyqt5/
` 所有学习资料链接:https://wiki.python.org/moin/PyQt 这里面都可以找到 `
一个很好的教材书:https://www.commandprompt.com/community/pyqt/book1
示例代码下载：http://www.qtrac.eu/pyqtbook.html
协议：PyQt4, unlike Qt, is not available under the LGPL.
PyQt4 是用于创建 GUI 应用的 toolkit 。它是 Python 和 Qt 库的混合。Qt 库是一种很强大的 GUI 库。PyQt4 的官方网站在 www.riverbankcomputing.co.uk/news 。其开发者是 Phil Thompson 。
PyQt4 以一系列 Python 模块实现。它有超过 300 个类，以及几乎 6000 的函数及方法。它是一个多平台的 toolkit。它可以运行于大多数的操作系统。包括 Unix，Windows 和 Mac。
PyQt4 具有双重的证书。开发者可以在 GPL 和商业的证书间选择。
由于有很多类，所以它们被划分为几个模块。
  * QtCore
  * QtGui
  * QtNetwork
  * QtXml
  * QtSvg
  * QtOpenGL
  * QtSql
QtCore 模块包含核心的非GUI的功能。这个模块用于时间，文件和目录，多种数据类型，流，url，mime type，线程或进程。 
QtGui 模块包含图形的组件和相关的类。这里包含例如按钮，窗口，状态栏，工具栏，边栏，位图，颜色，字体等。 
QtNetwork 模块包含用于网络编程的类。这些类允许编写 TCP/IP 和 UDP 客户端及服务端。它们使得网络编程更加简单和可移植。 
QtXml 包含用于处理 xml 文件的类。此模块提供了包括 SAX 和 DOM API 的实现。 
QtSvg 模块提供了用于显示 SVG 文件内容的类。Scalable Vector Graphics（SVG）是一种用于描述二维图形的语言。 
QtOpenGL 模块则用 OpenGL 库表现 3D 及 2D。这个模块可以无缝的把 Qt GUI 库和 OpenGL 库连接起来。
QtSql 库则提供了用于数据库的类。
为了创建图形用户接口，Python 程序员目前会选择几种，PyQt4，PyGTK 和 wxPython。选择哪个依赖于具体的条件。
因为一些证书的问题和技术需求，一种新的 Qt 和 Python 结合的项目被开发，成为 PySide 。PySide 与 PyQt4 有很高的兼容性。
```

import sys  
from PyQt4 import QtGui, QtCore   
# 此处创建一个名为Example的类，并且继承QtGui.QWidget类。
class Example(QtGui.QWidget):  
    def __init__(self):  
        super(Example, self).__init__()  
        # 将GUI的创建委托给initUI()方法。  
        self.initUI()  
        
    def initUI(self):    
        # 这是一个静态方法，用于设置渲染提示框(tooltips)的字体。  
        QtGui.QToolTip.setFont(QtGui.QFont('SanSerif', 10))  
        # setToolTip()方法用于创建一个提示框。  
        self.setToolTip('This is a <b>Qwidget</b> widget')  
        
        # 创建一个按钮(button)部件，并为其设置提示框。第一个参数指定按钮上显示的标签，第二个参数指定父窗口部件。此处父窗口部件是派生自QtGui.QWidget类的Example部件。  
        btn = QtGui.QPushButton('Button', self)  
        btn.setToolTip('This is a <b>QPushButton</b> widget')  
  
        # 设置按钮部件大小，并移动其在窗口上的位置。sizeHint()方法将给出按钮推荐的大小。  
        btn.resize(btn.sizeHint())  
        btn.move(50, 50)  
 
        qbtn = QtGui.QPushButton('Quit', self)  
 
        # PyQt4中的事件处理采用信号与槽(signal & slot)机制。如果点击按钮，  
        # 则会发送clicked信号。槽可以是Qt槽或者任何可调用的Python(直译的)。  
        # QtCore.QCoreApplication包含主事件循环。它处理并分发所有事件。  
        # instance()方法为我们提供了当前程序的实例。注意：QtCore.QCoreApplication  
        # 是由QtGui.QApplication创建的。clicked信号与quit()方法关联，用于结束  
        # 应用程序。通信由两个目标完成：发送者和接收者。发送者是按钮，接收者是应用程序。  
        qbtn.clicked.connect(QtCore.QCoreApplication.instance().quit)  
        qbtn.resize(qbtn.sizeHint())  
        qbtn.move(150, 50)  
  
        # 下面三个方法都继承自QtGui.QWidget类。setGeometry()方法做两件事。  
        # 它定位窗口在屏幕上的位置和窗口自身的大小。前两个参数指定窗口的x和y坐标。  
        # 第三个参数指定窗口的宽度，第四个参数指定窗口的高度。实际上，它将resize()  
        # 方法和move()方法合并到了一个方法中。  
        self.setGeometry(300, 300, 250, 150)  
        self.setWindowTitle('Icon')  
  
        # 移动窗口到屏幕中央  
        self.center()  
  
        # setWindowIcon()方法设置应用程序的图标。该方法需要一个QtGui.QIcon对象。  
        # QtGui.QIcon()接受将要显示图标的路径作为参数。  
        self.setWindowIcon(QtGui.QIcon('web.png'))  
  
        self.show()  
  
  
    # 当关闭QtGui.QWidget时，会产生一个QtGui.QCloseEvent事件。为了修改部件  
    # 行为，需要重新实现closeEvent()事件处理方法。  
    def closeEvent(self, event):  
        # 弹出对话框，该对话框有两个按钮，分别是‘Yes'和'No'。第一个字符串显示在  
        # 标题栏中。第二个字符串是对话框显示的消息内容。第三个参数指定出现在对话  
        # 框中的按钮组合。最后一个参数指定默认选中的按钮。  
        reply = QtGui.QMessageBox.question(self, 'Message',  
                                           'Are you sure to quit?',  
                                           QtGui.QMessageBox.Yes |  
                                           QtGui.QMessageBox.No,  
                                           QtGui.QMessageBox.No)  
  
        # 测试返回值，如果点击'Yes'，则接收事件并终止应用程序。否则，忽略该信号。  
        if reply == QtGui.QMessageBox.Yes:  
            event.accept()  
        else:  
            event.ignore()  
  
    def center(self):  
  
        # 获取指定主窗口矩形，包括所有的窗口框架。  
        qr = self.frameGeometry()  
  
        # QtGui.QDesttopWidget类提供了用户桌面的信息，包括屏幕大小。  
        # 计算出显示器的屏幕分辨率，并获取中心点的位置。  
        cp = QtGui.QDesktopWidget().availableGeometry().center()  
  
        # 窗口自身就是矩形，拥有宽度和高度。现在移动矩形到屏幕中间。矩形的大小保持不变。  
        qr.moveCenter(cp)  
  
        # 将应用程序左上角的点移动到qr矩形的左上角，因此窗口处于屏幕的中间。  
        self.move(qr.topLeft())  
  
        # 这一段其实没怎么弄明白。先暂时就这样吧！以后再慢慢理解。  
          
def main():  
  
    app = QtGui.QApplication(sys.argv)  
    ex = Example()  
    sys.exit(app.exec_())  
  
  
if __name__ == '__main__':  
    main()  

```
##  python+PyQT+Eric安装配置QtIDE开发环境
参考http://www.cnblogs.com/lhj588/archive/2011/10/03/2198472.html
eric ide官网：http://www.eric-ide.python-projects.org/eric-download.html
下载PyQt4
官方网站：http://www.riverbankcomputing.co.uk
下载Eric5
官方网站：http://eric-ide.python-projects.org/
**安装Eric5**
点击eric5-5.1.5中的install.py文件进行安装。
![](/data/dokuwiki/python/pasted/20160315-011628.png)
Pythonw配置：您可以通过点击安装目录eric5.bat（第一次）或eric5-configure.bat进行配置，
点击Editor—>Autocompation—>勾上所有的对号选框。QScintilla—>勾上左右的两个选
框，然后在下面source中，选择from Document and API files. 
![](/data/dokuwiki/python/pasted/20160315-011634.png)
点击Editor—>APIs—>勾上Complie APIs Autocompation，然后在Language中，选择
python。点面下面的Add from installed APIs按钮,选择住需要的.api文件。最后点击
Compile APIs。如图：
![](/data/dokuwiki/python/pasted/20160315-011654.png)
4、制作一个Demo
4、1、用Eric创建Demo项目
Projcet Name（项目名称）：Demo
Projcet Type（项目类型）：QT4 GUI
Projcet Directory（项目保存目录）:选择你计划存放的项目文件目录。
点击OK，会出现版本选择对话框，选择None。 
4、2、在Demo项目中添加Forms,用PyQT4设计  
单击软件界面左面的Projcet-Viewer中的第二个选项卡Forms在下面空白区域中，右键鼠标－>New form... 弹出对话框中选择Dialog，然后OK－给ui文件起个名字(Login.ui),保存后，会自弹出QT4设计窗口
![](/data/dokuwiki/python/pasted/20160315-011733.png)
转到QT设计师窗体设计工具，这时您就可视化的设计您的登录窗体了。这个工具非常简单、中文操作界面，一看就会。
返回到Eric界面后，设计程序。
返回到Eric IDE在Project-Viewer---->Forms 如图：
![](/data/dokuwiki/python/pasted/20160315-011756.png)
选择中Login.ui文件右键点击“compile form”,就会在Project-Viewer--->Sources生成一个UI_Login.py的脚步文件，如图：
![](/data/dokuwiki/python/pasted/20160315-011809.png)
##  PyQt4 中的事件和信号 
在任何的 GUI 程序中，事件是很重要的部分。**事件是由用户或操作系统产生的。**
**当我们调用应用的 exec_() 时，应用就进入了` 主循环 `。主循环会接受事件并且把它们发送给对象。**Trolltech 引入了一个独特的**信号和槽机制。**
事件是任何 GUI 程序中很重要的部分。**所有 GUI 应用都是事件驱动的。**一个应用对其生命期产生的不同的事件类型做出反应。事件是主要由应用的用户产生。但是，也可以通过其他方法产生，比如，网络通信，窗口的管理者，计时器。**在事件模型中，有三个参与者：**
  * 事件源（event source）
  * 事件对象（event object）
  * 事件目标（event target）
event source 是那些状态改变的对象。它产生事件。而 event object （Event）封装了事件源的状态改变。而 event target 则是需要被通知的。事件源把处理事件的任务委托给了事件目标。
**当我们调用了应用的 exec_() 方法，应用就进入主循环了。主循环接受事件然后把他们发送给对象。信号和槽用于对象间的通信。当特定的事件发生时 信号 就被发送了。而 槽 则是任何 Python 中可调用的。当信号发送给了这个槽，槽就被调用了。**

###  新的 API 
这是旧式的 API 。
QtCore.QObject.connect(button, QtCore.SIGNAL('clicked()'), self.onClicked)
PyQt 4.5 引入了新的 API 用于信号和槽。
```

button.clicked.connect(self.onClicked)

```
新式的更接近 Python 的标准。
主要接口介绍：
QtGui.QWidget 是所有用户接口对象的基类。
QtGui.QApplication每个 PyQt4 应用必须创建一个 application 对象。

###  信号与槽 
这是一个简单的例子，描述 PyQt4 中的信号和槽。
```

#!/usr/bin/python
# -*- coding: utf-8 -*-
import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        lcd = QtGui.QLCDNumber(self)
        sld = QtGui.QSlider(QtCore.Qt.Horizontal, self)
        vbox = QtGui.QVBoxLayout()
        vbox.addWidget(lcd)
        vbox.addWidget(sld)
        self.setLayout(vbox)
        sld.valueChanged.connect(lcd.display)
        self.setGeometry(300, 300, 250, 150)
        self.setWindowTitle('Signal & slot')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在我们的例子中，我们显示了 QtGui.QLCDNumber 和 QtGui.QSlider 。通过拖拽滑块，我们就可以更改 lcd 的数字。
sld.valueChanged.connect(lcd.display)
**此处，我们把 slider 的信号 valueChanged 和 lcd 的槽 display 连接了起来。**
sender 是发送信号的对象。 receiver 是接受信号的对象。而 slot 是回馈信号的方法。

##  自定义信号pyqtSignal与槽pyqtSlot 
参考官方文档http://pyqt.sourceforge.net/Docs/PyQt4/new_style_signals_slots.html
PyQT4.5版本以后可以采用新的信号与槽方式
###  1、自定义信号 
**PyQt4.QtCore.pyqtSignal(types[, name])**
Create one or more overloaded unbound signals as a class attribute.
Parameters:	
  * types – the types that define the C++ signature of the signal. Each type may be a Python type object or a string that is the name of a C++ type. Alternatively each may be a sequence of type arguments. In this case each sequence defines the signature of a different signal overload. The first overload will be the default.
  * name – the name of the signal. If it is omitted then the name of the class attribute is used. This may only be given as a keyword argument.
  * Return type:	an unbound signal
` 新的信号定义必须定义在继承QObject类的子类中 `
通过类成员变量定义信号对象(` QtCore的内建类pyqtSignal `。它不是QtCore的方法，而是QtCore的内建类)，如:
```

class MyWidget(QWidget):  
    Signal_NoParameters = PyQt4.QtCore.pyqtSignal()                                  # 无参数信号  
    Signal_OneParameter = PyQt4.QtCore.pyqtSignal(int)                               # 一个参数(整数)的信号  
    Signal_OneParameter_Overload = PyQt4.QtCore.pyqtSignal([int],[str])              # 一个参数(整数或者字符串)重载版本的信号  
    Signal_TwoParameters = PyQt4.QtCore.pyqtSignal(int,str)                          # 二个参数(整数,字符串)的信号  
    Signal_TwoParameters_Overload = PyQt4.QtCore.pyqtSignal([int,int],[int,str])     # 二个参数([整数,整数]或者[整数,字符串])重载版本的信号 

``` 
```

from PyQt4.QtCore import QObject, pyqtSignal
class Foo(QObject):
    # This defines a signal called 'closed' that takes no arguments.
    closed = pyqtSignal()
    # This defines a signal called 'rangeChanged' that takes two
    # integer arguments.
    range_changed = pyqtSignal(int, int, name='rangeChanged')
    # This defines a signal called 'valueChanged' that has two overloads,
    # one that takes an integer argument and one that takes a QString
    # argument.  Note that because we use a string to specify the type of
    # the QString argument then this code will run under Python v2 and v3.
    valueChanged = pyqtSignal([int], ['QString'])

    # This will cause problems because each has the same C++ signature.
    vChanged = pyqtSignal([dict], [list])

```
**connect(slot[, type=PyQt4.QtCore.Qt.AutoConnection[, no_receiver_check=False]])**
Connect a signal to a slot. An exception will be raised if the connection failed.
Parameters:	
  * slot – the slot to connect to, either a Python callable or another bound signal.
  * type – the type of the connection to make.
  * no_receiver_check – suppress the check that the underlying C++ receiver instance still exists and deliver the signal anyway.
Signals are disconnected from slots using the disconnect() method of a bound signal.

**disconnect([slot])**
Disconnect one or more slots from a signal. An exception will be raised if the slot is not connected to the signal or if the signal has no connections at all.
  * Parameters:	slot – the optional slot to disconnect from, either a Python callable or another bound signal. If it is omitted then all slots connected to the signal are disconnected.
Signals are emitted from using the emit() method of a bound signal.

**emit(*args)**
Emit a signal.通过pyqtSignal对象的emit方法发事件
  * Parameters:	args – the optional sequence of arguments to pass to any connected slots.
```

from PyQt4.QtCore import QObject, pyqtSignal
class Foo(QObject):
    # Define a new signal called 'trigger' that has no arguments.
    trigger = pyqtSignal()
    def connect_and_emit_trigger(self):
        # Connect the trigger signal to a slot.
        self.trigger.connect(self.handle_trigger)
        # Emit the signal.
        self.trigger.emit()
    def handle_trigger(self):
        # Show that the slot has been called.
        print "trigger signal received"

```

###  2、槽函数定义 
通过` @PyQt4.QtCore.pyqtSlot装饰 `方法定义槽函数,如:
PyQt4.QtCore.pyqtSlot(types[, name[, result]])
```

from PyQt4.QtCore import QObject, pyqtSlot
class Foo(QObject):

    @pyqtSlot()
    def foo(self):
        """ C++: void foo() """

    @pyqtSlot(int, str)
    def foo(self, arg1, arg2):
        """ C++: void foo(int, QString) """

    @pyqtSlot(int, name='bar')
    def foo(self, arg1):
        """ C++: void bar(int) """

    @pyqtSlot(int, result=int)
    def foo(self, arg1):
        """ C++: int foo(int) """

    @pyqtSlot(int, QObject)
    def foo(self, arg1):
        """ C++: int foo(int, QObject *) """

```
```

from PyQt4.QtCore import QObject, pyqtSlot

class Foo(QObject):

    @pyqtSlot(int)
    @pyqtSlot('QString')
    def valueChanged(self, value):
        """ Two slots will be defined in the QMetaObject. """

```
###  Connecting Signals and Slots 
Connections between signals and slots (and other signals) are made using the QtCore.QObject.connect() method. For example:
```

QtCore.QObject.connect(a, QtCore.SIGNAL('QtSig()'), pyFunction)
QtCore.QObject.connect(a, QtCore.SIGNAL('QtSig()'), pyClass.pyMethod)
QtCore.QObject.connect(a, QtCore.SIGNAL('QtSig()'), b, QtCore.SLOT('QtSlot()'))
QtCore.QObject.connect(a, QtCore.SIGNAL('PySig()'), b, QtCore.SLOT('QtSlot()'))
QtCore.QObject.connect(a, QtCore.SIGNAL('PySig'), pyFunction)

```
Disconnecting signals works in exactly the same way using the QtCore.QObject.disconnect() method. However, not all the variations of that method are supported by PyQt4. Signals must be disconnected one at a time.
###  Emitting Signals 
Any instance of a class that is derived from the QtCore.QObject class can emit a signal using its emit() method. This takes a minimum of one argument which is the signal. Any other arguments are passed to the connected slots as the signal arguments. For example:
```

a.emit(QtCore.SIGNAL('clicked()'))
a.emit(QtCore.SIGNAL('pySig'), "Hello", "World")

```
###  信号(singal)与槽(slot)进阶 
信号(singal)与槽(slot)用于对象相互通信，信号：当某个对象的某个事件发生时，触发一个信号，槽：响应指定信号的所做的反应
PyQt的窗体控件类已经有很多的内置信号，开发者也可以添加自己的自定义信号,信号槽有如下特点：
　　　　- 一个信号可以连接到许多插槽。
　　　　- 一个信号也可以连接到另一个信号。
　　　　- 信号参数可以是任何Python类型。
　　　　- 一个插槽可以连接到许多信号。
　　　　- 连接可能会直接（即同步）或排队（即异步）。
　　　　- 连接可能会跨线程。
　　　　- 信号可能会断开
####  内置信号槽的使用 
```

from PyQt5.QtWidgets import *  
from PyQt5.QtCore import *  
    
def sinTest():  
    btn.setText("按钮文本改变")  
    
app = QApplication([])  
    
main = QWidget()  
main.resize(200,100)  
btn = QPushButton("按钮文本",main)  
##按钮btn的内置信号连接名为sinTest的槽  
btn.clicked.connect(sinTest)  
main.show()  
    
app.exec_()  

```
####  自定义信号槽的使用 
```

class SinClass(QObject):  
        
    ##声明一个无参数的信号  
    sin1 = pyqtSignal()  
        
    ##声明带一个int类型参数的信号  
    sin2 = pyqtSignal(int)  
        
    ##声明带一个int和str类型参数的信号  
    sin3 = pyqtSignal(int,str)  
    
    ##声明带一个列表类型参数的信号  
    sin4 = pyqtSignal(list)  
    
    ##声明带一个字典类型参数的信号  
    sin5 = pyqtSignal(dict)  
    
    ##声明一个多重载版本的信号，包括了一个带int和str类型参数的信号，以及带str参数的信号  
    sin6 = pyqtSignal([int,str], [str])  
        
    def __init__(self,parent=None):  
        super(SinClass,self).__init__(parent)  
    
        ##信号连接到指定槽  
        self.sin1.connect(self.sin1Call)  
        self.sin2.connect(self.sin2Call)  
        self.sin3.connect(self.sin3Call)  
        self.sin4.connect(self.sin4Call)  
        self.sin5.connect(self.sin5Call)  
        self.sin6[int,str].connect(self.sin6Call)  
        self.sin6[str].connect(self.sin6OverLoad)  
    
        ##信号发射  
        self.sin1.emit()  
        self.sin2.emit(1)  
        self.sin3.emit(1,"text")  
        self.sin4.emit([1,2,3,4])  
        self.sin5.emit({"name":"codeio","age":"25"})  
        self.sin6[int,str].emit(1,"text")  
        self.sin6[str].emit("text")  
            
    def sin1Call(self):  
        print("sin1 emit")  
    
    def sin2Call(self,val):  
        print("sin2 emit,value:",val)  
    
    def sin3Call(self,val,text):  
        print("sin3 emit,value:",val,text)  
    
    def sin4Call(self,val):  
        print("sin4 emit,value:",val)  
            
    def sin5Call(self,val):  
        print("sin5 emit,value:",val)  
    
    def sin6Call(self,val,text):  
        print("sin6 emit,value:",val,text)  
    
    def sin6OverLoad(self,val):  
        print("sin6 overload emit,value:",val)  
    
sin = SinClass()  

```
运行结果：
sin1 emit
sin2 emit,value: 1
sin3 emit,value: 1 text
sin4 emit,value: [1, 2, 3, 4]
sin5 emit,value: {'age': '25', 'name': 'codeio'}
sin6 emit,value: 1 text
sin6 overload emit,value: text
####  信号槽N对N连接、断开连接 
```

from PyQt5.QtWidgets import *  
from PyQt5.QtCore import *  
  
class SinClass(QObject):  
  
    ##声明一个无参数的信号  
    sin1 = pyqtSignal()  
  
    ##声明带一个int类型参数的信号  
    sin2 = pyqtSignal(int)  
  
    def __init__(self,parent=None):  
        super(SinClass,self).__init__(parent)  
  
        ##信号sin1连接到sin1Call和sin2Call这两个槽  
        self.sin1.connect(self.sin1Call)  
        self.sin1.connect(self.sin2Call)  
  
        ##信号sin2连接到信号sin1  
        self.sin2.connect(self.sin1)  
  
        ##信号发射  
        self.sin1.emit()  
        self.sin2.emit(1)  
  
        ##断开sin1、sin2信号与各槽的连接  
        self.sin1.disconnect(self.sin1Call)  
        self.sin1.disconnect(self.sin2Call)  
        self.sin2.disconnect(self.sin1)  
  
        ##信号sin1和sin2连接同一个槽sin1Call  
        self.sin1.connect(self.sin1Call)  
        self.sin2.connect(self.sin1Call)  
  
        ##信号再次发射  
        self.sin1.emit()  
        self.sin2.emit(1)  
  
    def sin1Call(self):  
        print("sin1 emit")  
  
    def sin2Call(self):  
        print("sin2 emit")  
  
sin = SinClass()  

```
运行结果：
sin1 emit
sin2 emit
sin1 emit
sin2 emit
sin1 emit
sin1 emit
####  多线程信号槽通信 
```

from PyQt5.QtWidgets import *  
from PyQt5.QtCore import *  
  
class Main(QWidget):  
    def __init__(self, parent = None):  
        super(Main,self).__init__(parent)  
  
        ##创建一个线程实例并设置名称、变量、信号槽  
        self.thread = MyThread()  
        self.thread.setIdentity("thread1")  
        self.thread.sinOut.connect(self.outText)  
        self.thread.setVal(6)  
  
    def outText(self,text):  
        print(text)  
  
class MyThread(QThread):  
  
    sinOut = pyqtSignal(str)  
  
    def __init__(self,parent=None):  
        super(MyThread,self).__init__(parent)  
  
        self.identity = None  
  
    def setIdentity(self,text):  
        self.identity = text  
  
    def setVal(self,val):  
        self.times = int(val)  
  
        ##执行线程的run方法  
        self.start()  
  
    def run(self):  
        while self.times > 0 and self.identity:  
            ##发射信号  
            self.sinOut.emit(self.identity+" "+str(self.times))  
            self.times -= 1  
  
app = QApplication([])  
  
main = Main()  
main.show()  
  
app.exec_()  

```
运行结果：
thread1 6
thread1 5
thread1 4
thread1 3
thread1 2
thread1 1
###  5、实例 
```

from PyQt4 import QtGui, QtCore
import os,sys

"""
自定义信号
"""
class Event(QtCore.QObject):
    progressSignal=QtCore.pyqtSignal(int)

```
```

self.events=Event()
self.events.progressSignal.connect(self.progressHandle)
self.events.progressSignal.emit(math.ceil(i/piccount*100))

```
转载出处：http://blog.csdn.net/hlqyq/article/details/6713828
###  重新实现事件处理句柄¶ 
在 PyQt4 中事件的处理一般通过重新实现事件的句柄。
```

#!/usr/bin/python
# -*- coding: utf-8 -*-
import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.setGeometry(300, 300, 250, 150)
        self.setWindowTitle('Event handler')
        self.show()
    def keyPressEvent(self, e):
        if e.key() == QtCore.Qt.Key_Escape:
            self.close()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在我们的例子中，我们重新实现了 keyPressEvent() 。
```

def keyPressEvent(self, e):
    if e.key() == QtCore.Qt.Key_Escape:
        self.close()

```
如果我们按 escape 键，那么应用就将终止。

###  事件发送者 
有些时候，知道信号的发送者是很方便的。因此，PyQt4 有个 sender() 方法。
```

#!/usr/bin/python
# -*- coding: utf-8 -*-
import sys
from PyQt4 import QtGui, QtCore
class Example(QtGui.QMainWindow):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        btn1 = QtGui.QPushButton("Button 1", self)
        btn1.move(30, 50)
        btn2 = QtGui.QPushButton("Button 2", self)
        btn2.move(150, 50)
        btn1.clicked.connect(self.buttonClicked)
        btn2.clicked.connect(self.buttonClicked)
        self.statusBar()
        self.setGeometry(300, 300, 290, 150)
        self.setWindowTitle('Event sender')
        self.show()
    def buttonClicked(self):
        sender = self.sender()
        self.statusBar().showMessage(sender.text() + ' was pressed')
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这个例子中有两个按钮。在 buttonClicked() 方法中，**我们通过调用 sender() 方法知道了哪个按钮被点击了。**
btn1.clicked.connect(self.buttonClicked)
btn2.clicked.connect(self.buttonClicked)
两个按钮都连接到相同的槽中。
我们通过调用 sender() 方法知道了消息源。在状态栏中，我们显示了被按的按钮的标签。
###  发送信号 
**由`  QtCore.QObject ` 创建的对象可以发送信号。**如果我们点击按钮，一个 clicked() 信号就被生成。在下面的例子中我们将看到如何发送信号。
```

#!/usr/bin/python
# -*- coding: utf-8 -*-
import sys
from PyQt4 import QtGui, QtCore
class Communicate(QtCore.QObject):
    closeApp = QtCore.pyqtSignal()
class Example(QtGui.QMainWindow):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        self.c = Communicate()
        self.c.closeApp.connect(self.close)
        self.setGeometry(300, 300, 290, 150)
        self.setWindowTitle('Emit signal')
        self.show()
    def mousePressEvent(self, event):
        self.c.closeApp.emit()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
我们**创建了一个新的信号称为 closeApp** 。这个信号在鼠标点击时将被发送(调用` emit() `)。而这个信号又和 QtGui.QMainWindow 的 close() 槽相连接。
```

class Communicate(QtCore.QObject):
    closeApp = QtCore.pyqtSignal()

```
我们创建了基于 QtCore.QObject 的类(由`  QtCore.QObject ` 创建的对象可以发送信号。)。当它被实例化后就创建了一个 closeApp 信号。
self.c = Communicate()
self.c.closeApp.connect(self.close)
类 Communicate 的实例就被创建了。我们把 QtGui.QMainWindow 的 close() 槽连接到信号 closeApp 上。
当鼠标指针在窗口中点击，信号 closeApp 就被发送了。
在这个部分，我们涉及了信号与槽。
##  PyQt4的 layout 管理 
layout 管理就是在窗口中布置 widget。管理有两种方式。我们可以使用 绝对定位 或者 layout 类 。
###  绝对定位 
程序员可以指定每个 widget 的位置和大小。**当你使用绝对定位时，你需要理解一些事情**。
  * 如果你改变了窗口的大小，widget 的大小和位置是不会改变的。
  * 在不同平台上应用可能看起来不太一样。
  * 在你的应用中改变字体会毁掉你的 layout。
如果你决定改变你的 layout，那么你必须要重新设计你的 layout，这意味着麻烦及耗时。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        lbl1 = QtGui.QLabel('Zetcode', self)
        lbl1.move(15, 10)
        lbl2 = QtGui.QLabel('tutorials', self)
        lbl2.move(35, 40)
        lbl3 = QtGui.QLabel('for programmers', self)
        lbl3.move(55, 70)
        self.setGeometry(300, 300, 250, 150)
        self.setWindowTitle('Absolute')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
我们简单地` 调用 move() ` 方法来定位我们的 widget 。在我们这个情况下，这些 widget 是标签。我们通过给定 x 和 y 坐标进行定位。坐标系统是从左上角开始。x 的值从左到右增加。y 则是从上到下。
###  Box Layout 
通过 layout 类管理会更加灵活和实用。这是优先考虑的方法。基本的 layout 类是 ` QtGui.QHBoxLayout 和 QtGui.QVBoxLayout ` 。它们分别以水平和垂直方式对 widget 进行布局。
假设我们想把两个按钮放置在右下角。为了创建这样的 layout，我们可以使用一个水平的和垂直的盒子。为了创建必要的空间，我们将会增加一个 缩放比例 。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        okButton = QtGui.QPushButton("OK")
        cancelButton = QtGui.QPushButton("Cancel")
        hbox = QtGui.QHBoxLayout()
        hbox.addStretch(1)
        hbox.addWidget(okButton)
        hbox.addWidget(cancelButton)
        vbox = QtGui.QVBoxLayout()
        vbox.addStretch(1)
        vbox.addLayout(hbox)
        self.setLayout(vbox)
        self.setGeometry(300, 300, 300, 150)
        self.setWindowTitle('Buttons')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
这里在窗口的右下角放置了两个按钮。他们在窗口调整时，会保持处于那个位置。我们使用了 QtGui.HBoxLayout 和 QtGui.QVBoxLayout 。
我们创建了一个水平的 layout 。增加了缩放比例(hbox.addStretch)和按钮。**这个缩放在两个按钮前增加了合适的缩放空白。这就使得按钮在窗口的右边。**
为了创建必要的 layout，我们把水平的 layout 放置到垂直的 layout 中。而**垂直盒子中的缩放因子(vbox.addStretch(1))使得水平的盒子处于窗口的底部。**
最后，我们设置了窗口的 layout 。

###  QtGui.QGridLayout 
最通用的 layout 类是网格 layout 。这个 layout 把空间分为行和列。为了创建一个网格的 layout，我们可以使用** QtGui.QGridLayout** 类。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        names = ['Cls', 'Bck', '', 'Close', '7', '8', '9', '/',
                '4', '5', '6', '*', '1', '2', '3', '-',
                '0', '.', '=', '+']
        grid = QtGui.QGridLayout()
        j = 0
        pos = [(0, 0), (0, 1), (0, 2), (0, 3),
                (1, 0), (1, 1), (1, 2), (1, 3),
                (2, 0), (2, 1), (2, 2), (2, 3),
                (3, 0), (3, 1), (3, 2), (3, 3 ),
                (4, 0), (4, 1), (4, 2), (4, 3)]
        for i in names:
            button = QtGui.QPushButton(i)
            if j == 2:
                grid.addWidget(QtGui.QLabel(''), 0, 2)
            else: grid.addWidget(button, pos[j][0], pos[j][1])
            j = j + 1
        self.setLayout(grid)
        self.move(300, 150)
        self.setWindowTitle('Calculator')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
在我们的例子中，我们创建了一组按钮，并以网格放置。为了填补一个空格，我们增加了一个 QtGui.QLable 的 widget 。
grid = QtGui.QGridLayout()
这里，我们创建了一个网格 layout 。
```

if j == 2:
    grid.addWidget(QtGui.QLable(''), 0, 2)
else: grid.addWidget(button, pos[j][0], pos[j][1])

```
为了把一个 widget 增加到网格中，我们调用了 addWidget() 方法。参数是 widget，行和列的数字。

###  再看例子 
在网格中，widget 可以跨多行或多列。下面的例子我们就将介绍这个。
```

import sys
from PyQt4 import QtGui
class Example(QtGui.QWidget):
    def __init__(self):
        super(Example, self).__init__()
        self.initUI()
    def initUI(self):
        title = QtGui.QLabel('Title')
        author = QtGui.QLabel('Author')
        review = QtGui.QLabel('Review')
        titleEdit = QtGui.QLineEdit()
        authorEdit = QtGui.QLineEdit()
        reviewEdit = QtGui.QTextEdit()
        grid = QtGui.QGridLayout()
        grid.setSpacing(10)
        grid.addWidget(title, 1, 0)
        grid.addWidget(titleEdit, 1, 1)
        grid.addWidget(author, 2, 0)
        grid.addWidget(authorEdit, 2, 1)
        grid.addWidget(review, 3, 0)
        grid.addWidget(reviewEdit, 3, 1, 5, 1)
        self.setLayout(grid)
        self.setGeometry(300, 300, 350, 300)
        self.setWindowTitle('Review')
        self.show()
def main():
    app = QtGui.QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())
if __name__ == '__main__':
    main()

```
我们创建了一个窗口，其中有三个标签，两个行编辑区及一个文本编辑区。layout 由 QtGui.QGridLayout 创建。
grid = QtGui.QGridLayout()
grid.setSpacing(10)
我们创建了网格 layout，**并设置了 widget 间的间隔。**
grid.addWidget(reviewEdit, 3, 1, 5, 1)
如果我们增加一个 widget 到网格中，**我们可以提供行和列的跨度。**我们使得 reviewEdit 的跨度为 5 。
这个部分我们讲述了 layout 管理。


参考http://www.cppblog.com/mirguest/category/15952.html
http://blog.csdn.net/Kai_gai/article/category/5911577