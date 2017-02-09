title: python学习笔记110 

#  Python学习笔记之PyQt4Widget部件2 
参考资料：Qt documentation online(http://doc.qt.io/qt-5/qtexamplesandtutorials.html)（因为这个帮助文档是基于C++做的，里面的语句是C++写的，不过因为PyQt做了很好的移植，方法的名称，参数等等基本都可以在python中套用）
##  Model/View组件 
标准组件
![](/data/dokuwiki/python/pasted/20160316-151356.png)
Model/View组件
![](/data/dokuwiki/python/pasted/20160316-151237.png)![](/data/dokuwiki/python/pasted/20160316-151242.png)

` Model类型一般需要继承 QAbstractItemModel `
**Model/View组件以及对应的标准组件**
  * QListWidget继承于QListView,是QListView对应的标准组件。而QListView本身是Model/View组件
  * QTableWidget继承于	QTableView是QTableView对应的标准组件。而QTableView本身是Model/View组件
  * QTreeWidget继承于	QTreeView是QTreeView对应的标准组件。而QTreeView本身是Model/View组件
  * QColumnView 
  * QComboBox 
##  QListWidget 
QListWidget是一个列表框，使用非常简单：它的继承关系：
![](/data/dokuwiki/python/pasted/20160316-141125.png)
```

# !/usr/bin/python
import sys
from PyQt4.QtGui import *
from PyQt4 import QtCore
 
class ListWidget(QMainWindow):
    def __init__(self, parent=None):
        QWidget.__init__(self, parent)
        self.setWindowTitle('ListWidget')
        self.List = QListWidget(self)
        self.List.setSortingEnabled(1)
        item = ['OaK','Banana','Apple',' Orange','Grapes','Jayesh']
        listItem = []
        for lst in item:
            listItem.append(QListWidgetItem(lst))
        for i in range(len(listItem)):
            self.List.insertItem(i+1,listItem[i])
        self.setCentralWidget(self.List)
       
app = QApplication(sys.argv)
tb = ListWidget()
tb.show()
app.exec_()

```
列表选择变化会listWidget触发SIGNAL("currentRowChanged(int)")
##  QTableWidget 
![](/data/dokuwiki/python/pasted/20160316-142010.png)
如上所示，QtableWidget是继承于QtableView的。所以QtableView的方法也在QtableWidget中继承了。
QTableWidget类提供了一个默认模式的表格，它是基于Item的，这个Item是由QTableWidgetItem提供的。如果你要构建自己的数据模式，请使用QTableView而不是QTableWidget。
一，如何构建一个QtableWidget。
```

# !/usr/bin/python
import sys
from PyQt4.QtGui import *
class TableWidget(QMainWindow):
    def __init__(self,parent=None):
        QWidget.__init__(self,parent)
        self.setWindowTitle('TableWidget')
        self.table = QTableWidget(10,6)
        self.setCentralWidget(self.table)
app = QApplication(sys.argv)
tb = TableWidget()
tb.show()
app.exec_()

```
结果如下图所示：创建了一个10行6列的表格，可编辑可输入。
初始化的时候也可以不设置行数和列数。而等到创建完了以后再设。
比如：
```

    self.table = QTableWidget()
    self.table.setRowCount(10)
    self.table.setColumnCount(6)

```
二，添加表头。
可以添加水平和垂直表头，QtableWidget提供了两个方法来添加表头，非常方便。
self.table = QTableWidget(5,7)
self.table.setHorizontalHeaderLabels(['SUN','MON','TUE','WED','THU','FIR','SAT'])
上面两句就是添加水平表头。假如我们不添加表头，那么表头默认的数字就是代表所在
行或者所在列。
 
三，添加表项。
self.newItem = QTableWidgetItem('Item')
self.table.setItem(1,2,self.newItem)
如下图：可以看出，行列数是指不算标题行，都是从第0行，或者第0列开始计数的。

下面我们通过循环来添加表项的所有内容：
```

self.table = QTableWidget(5,7)       
self.table.setHorizontalHeaderLabels(['SUN','MON','TUE','WED',
                                              'THU','FIR','SAT'])
        for i in range(self.table.rowCount()):
            for j in range(self.table.columnCount()):
                cnt = '(%d,%d)'% (i,j)
                newItem = QTableWidgetItem(cnt)
                self.table.setItem(i,j,newItem)

```
QTableWidget.rowCount()是得到行数，int型。
QTableWidget.columnCount()是得到列数，int型
![](/data/dokuwiki/python/pasted/20160316-142250.png)
四，修改表项内容
  * QTableWidget.clear(self) 清楚所有表项及表头
  * QTableWidget.clearContents(self) 只清楚表项，不清楚表头。
  * QTableWidget.insertColumn(self, int column) 在某一列插入新的一列。
  * QTableWidget.insertRow(self, int row)在某一行插入新的一行。
  * QTableWidget.removeColumn(self, int column) 移除column列及其内容。
  * QTableWidget.removeRow(self, int row)移除第row行及其内容。
 
五，关于显示的一些问题，外观
QTableView.setShowGrid (self, bool show) 从TableView继承而来的，
是否显示表格的横竖线，默认情况下是显示的，如上面的例子，如果设为setShowGrid(False) ，则不显示分割线，横竖都没有。
另外，还可以通过hideRow(),hideColumn(),showRow(),showColumn()等来隐藏或显示特定行和列。
还有一个是否显示表头的问题，比如很多情况下我们只需要水平表头，不需要垂直表头，怎么办呢？我们在上面的例子中加上这么一句:
self.table.verticalHeader().setVisible(False)
setVisible是所有Qwidget都有的方法，而self.table.verticalHeader()是得到了一个表头，表头也是QheaderView继承来的，也是Qwidget的子类，所以也可以调用setVisible()方法来显示或者隐藏表头。

因为继承关系，父类的很多方法都可以调用，所以QTableWidget的方法非常之多，应该有几百个，一一学习是不可能的，只能用到的时候去查。下面介绍几个继承于上面父类的方法。
QabstractItemView 是QTableWidget的父类的父类，他有下面几个方法，我们QTableWidget中经常调用，就是是否项目可编辑，点击选择是选择行，还是可以选择列，是可以选择多行（多列），还是只可以选择单行（单列），等等非常好用，如下的列子：
self.table.setEditTriggers(QTableWidget.NoEditTriggers)
self.table.setSelectionBehavior(QTableWidget.SelectRows)
self.table.setSelectionMode(QTableWidget.SingleSelection)
self.table.setAlternatingRowColors(True)第一句，设为不可编辑状态，第二是选择行，第三是选择单个行，第四是隔行改变颜色。

##  QTreeWidget 
![](/data/dokuwiki/python/pasted/20160316-142438.png)
因为继承关系是QAbstractItemView->QTreeView->QTreeWidget ,所以和QTableWidget很多地方是类似的。
如果需要特殊的模式，比如显示硬盘信息及内部文件的dir模式等，都需要用QTreeView，而不是用QTreeWidget。
和QTableWidget类似，一般步骤是先创建一个QTreeWidget实例，然后设置列数，然后再添加头。
```

# !/usr/bin/python
import sys
from PyQt4.QtGui import *
from PyQt4.QtCore import *
class TreeWidget(QMainWindow):
    def __init__(self,parent=None):
        QWidget.__init__(self,parent)
        self.setWindowTitle('TreeWidget')
        self.tree = QTreeWidget()
        self.tree.setColumnCount(2)
        self.tree.setHeaderLabels(['Key','Value'])
        root= QTreeWidgetItem(self.tree)
        root.setText(0,'root')
        child1 = QTreeWidgetItem(root)
        child1.setText(0,'child1')
        child1.setText(1,'name1')
        child2 = QTreeWidgetItem(root)
        child2.setText(0,'child2')
        child2.setText(1,'name2')
        child3 = QTreeWidgetItem(root)
        child3.setText(0,'child3')
        child4 = QTreeWidgetItem(child3)
        child4.setText(0,'child4')
        child4.setText(1,'name4')
        self.tree.addTopLevelItem(root)
        self.setCentralWidget(self.tree)             
app = QApplication(sys.argv)
tp = TreeWidget()
tp.show()
app.exec_()

```
其中的QtreeWidgetItem就是一层层的添加的，其实还是不太方便的。
**在应用程序中一般不是这样来创建QTreeView的，特别是比较复杂的Tree，一般都是通过QTreeView来实现而不是QTreeWidget来实现的。**
这种与QTreeWidget最大的区别就是，我们自己来定制模式，当然也有些系统提供给我们的模式，比如我们的文件系统盘的树列表，比如下面的：
```

import sys
from PyQt4 import QtCore, QtGui
if __name__ == "__main__":
    app = QtGui.QApplication(sys.argv)
    model = QtGui.QDirModel() #系统给我们提供的
    tree = QtGui.QTreeView()
    tree.setModel(model)
    tree.setWindowTitle(tree.tr("Dir View"))
    tree.resize(640, 480)
    tree.show()
    sys.exit(app.exec_())

```
![](/data/dokuwiki/python/pasted/20160316-142658.png)

所以一般的我们自己定制模式，步骤如下：
```

import sys
from PyQt4 import QtCore, QtGui
if __name__ == "__main__":
    app = QtGui.QApplication(sys.argv)
    model = TreeModel(需要处理的数据)
    view = QtGui.QTreeView()
    view.setModel(model)
    view.setWindowTitle("Simple Tree Model")
    view.show()
sys.exit(app.exec_())

```
其中的TreeModel类是我们自己写的，如何显示我们的数据，可以查看相关资料。
