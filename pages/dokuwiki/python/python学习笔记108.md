title: python学习笔记108 

#  Python学习笔记之PyQt4之StyleSheet 
官网示例:http://doc.qt.io/qt-4.8/stylesheet-examples.html
官方教程:http://doc.qt.io/qt-4.8/stylesheet-syntax.html
http://doc.qt.io/qt-4.8/stylesheet-reference.html
作为一个跨平台的UI开发框架，Qt提供了强大而灵活的界面外观设计机制。
Qt样式表是一个可以自定义部件外观的十分强大的机制。Qt样式表的概念、术语和语法都受到了HTML的层叠样式表（Cascading StyleSheets，CSS）的启发，不过与CSS不同的是，Qt样式表应用于部件的世界。
二、在设计模式使用Qt样式表
1.新建Qt Gui应用，项目名称为myStyle，其他保持默认即可。完成后打开mainwindow.ui进入设计模式，然后拖入一个Push Button按钮。
2.在按钮部件上右击，选择“改变样式表”菜单项，在弹出的编辑样式表对话框中点击“添加颜色”下拉框，然后选择background-color，我们为其添加背景颜色。如下图所示。
这时会弹出选择颜色按钮，大家可以随便选择一个颜色，这里选择了红色，然后点击确定按钮关闭对话框。添加好的代码如下图所示。这种方法可以快速设置样式表，当然我们也可以自己手动来添加代码。
3.完成后大家可能发现按钮的颜色并没有改变，不要着急，这时运行程序，发现已经有效果了。如下图所示。
4.其实在设计模式还可以很容易地使用背景图片，这个需要使用Qt资源，大家可以试试，这里就不再介绍。

三、在代码中设置Qt样式表
既然在设计器中可以使用样式表，那么使用代码就一定可以实现。在代码中可以使用` setStyleSheet()函数 `来设置样式表，不过用两种设置方法。
1.设置所有的相同部件都使用相同的样式。我们在mainwindow.cpp的构造函数中添加如下代码：
setStyleSheet("QPushButton { color: white }");
这时运行程序，效果如下图所示。
可以看到按钮的文本颜色变成了白色，不过这种方式是给所有QPushButton类对象设置的样式。也就是说，我们再往界面上拖放其他的Push Button，它的文本颜色也会变成白色。

2.那么怎样才能只给特定的一个按钮设置样式表呢，这就需要使用第二种方式了。我们接着在mainwindow.cpp构造函数中添加代码：
ui->pushButton->setStyleSheet("color: blue");
这样就是只给先前添加的pushButton按钮设置了样式，将文本颜色设置为蓝色。为了有一个对比，大家可以再往界面上拖入一个Push Button按钮，然后运行程序，如下图所示。
也许现在又会问了，怎么按钮的背景不是红色的了？这是因为一个部件只能单独设置一个样式表，我们在代码中为pushButton设置了样式表就会屏蔽设计器中设置的。这里只是说单独为一个部件同时设置了多个样式表会出现这种情况，如果对其父类进行设置，则只会对其有影响，但是不会屏蔽掉自己的样式表，比如前面按钮的红底白字就是这种情况。
下面我们把代码更改如下：
ui->pushButton->setStyleSheet("background-color:red; color: blue");
再次运行程序，可以发现已经是红底蓝字了。效果如下图所示。
现在大家应该可以了解到，我们前面在设计模式中就是只为指定的pushButton按钮设置了背景。

##  四、样式表语法 
###  1、 一般选择器（selector） 
Qt支持所有的CSS2定义的选择器，其祥细内容可以在w3c的网站上查找http://www.w3.org/TR/CSS2/selector.html ， 其中比较常用的selector类型有：
  * 通用类型选择器：* 会对所有控件有效果。
  * 类型选择器：QPushButton匹配所有QPushButton的实例和其子类的实例。
  * 属性选择器：QPushButton[flat=”false”]匹配所有QPushButton属性flat为false的实例，属性分为两种，静态的和动态的，静态属性可以通过Q_PROPERTY() 来指定，来动态属性可以使用setProperty来指定，如：
```

QLineEdit *nameEdit = new QLineEdit(this);
nameEdit->setProperty("mandatoryField", true);

```
如果在设置了qss后Qt属性改变了，需要重新设置qss来使其生效，可以使用先unset再setqss。
  * 类选择器：.QPushButton匹配所有QPushButton的实例，但不包含其子类，这相当于：*[class~="QPushButton"]     ~=的意思是测试一个QStringList类型的属性是否包含给定的QString
  * ID选择器：QPushButton#okButton对应Qt里面的object name设置，使用这条CSS之前要先设置对应控件的object name为okButton，如：Ok->setObjectName(tr(“okButton”));
  * 后裔选择器：QDialog QPushButton匹配所有为QDialog后裔（包含儿子，与儿子的儿子的递归）为QPushButton的实例
  * 子选择器：QDialog > QPushButton匹配所有的QDialog直接子类QPushButton的实例，不包含儿子的儿子的递归。


###  2、子控件选择器 
对于复杂的控件，可能会在其中包含其他子控件，如一个QComboxBox中有一个drop-down的按钮。那么现在如果要设置QComboxBox的下拉按钮的话，就可以这样访问：
QComboBox::drop-down { image: url(dropdown.png) }其标志是（::）
子控件选择器是用位置的引用来代表一个元素，这个元素可以是一个单一控件或是另一个包含子控件的复合控件。使用subcontrol-origin属性可以改变子控件的默认放置位置，如：
```

    QComboBox {
           margin-right: 20px;
    }
    QComboBox::drop-down {
           subcontrol-origin: margin;
    }

```
###  3、伪选择器（pseudo-states） 
伪选择器以冒号（:）表示，与css里的伪选择器相似，是基于控件的一些基本状态来限定程序的规则，如hover规则表示鼠标经过控件时的状态，而press表示按下按钮时的状态。如：
```

    QPushButton:hover {
           Background-color:red;
    }

```
表示鼠标经过时QPushButton背景变红。
**Pseudo还支持否定符号(!)，如：**
QRadioButton:!hover { color: red }
表示没有鼠标移上QRadioButton时他显示的颜色是red。
Pseudo可以被串连在一起，比如：
QPushButton:hover:!pressed { color: blue; }
表示QPushButton在鼠标移上却没有点击时显示blue字，但如果点击的时候就不会显示blue颜色了。
同样可以和之前所讲的子控件选择器一起联合使用，如：
QSpinBox::down-button:hover { image: url(btn-combobox-press.bmp) }
与前面所讲的一样，伪选择器，子控件选择器等都是可以用逗号(,)分隔表示连续相同的设置的，如：
QPushButton:hover,QSpinBox::down-button, QCheckBox:checked { color: white ;image: url(btn-combobox-press.bmp);}

###  1.样式规则 
样式表包含了一系列的样式规则，**一个样式规则由一个选择符（selector）和一个声明（declaration）组成**。选择符指定了受该规则影响的部件；声明指定了这个部件上要设置的属性。例如：
QPushButton{color:red}
在这个样式规则中，**QPushButton是选择符，{color:red}是声明，而color是属性，red是值**。这个规则指定了QPushButton和它的子类应该使用红色作为它们的前景色。**Qt样式表中一般不区分大小写**，例如color、Color、COLOR和COloR表示相同的属性。只有类名，对象名和Qt属性名是区分大小写的。一些选择符可以指定相同的声明，只需要使用逗号隔开，例如：
QPushButton,QLineEdit,QComboBox{color:red}
一个样式规则的声明部分是一些属性：值对组成的列表，它们包含在大括号中，使用分号隔开。例如：
QPushButton{color:red;background-color:white}

2.子控件（Sub-Controls）
对一些复杂的部件修改样式，可能需要访问它们的子控件，例如QComboBox的下拉按钮，还有QSpinBox的向上和向下的箭头等。选择符可以包含子控件来对部件的特定子控件应用规则，例如：
QComboBox::drop-down{image:url(dropdown.png)}
这样的规则可以改变所有的QComboBox部件的下拉按钮的样式。在Qt Style Sheets Reference关键字对应的帮助文档的List of Stylable Widgets一项中列出了所有可以使用样式表来自定义样式的Qt部件，在List of Sub-Controls一项中列出了所有可用的子控件。

3.伪状态（Pseudo-States）
选择符可以包含伪状态来限制规则在部件的指定的状态上应用。伪状态出现在选择符之后，用冒号隔离，例如：
QPushButton:hover{color:white}
这个规则表明当鼠标悬停在一个QPushButton部件上时才被应用。伪状态可以使用感叹号来表示否定，例如要当鼠标没有悬停在一个QRadioButton上时才应用规则，那么这个规则可以写为：
QRadioButton:!hover{color:red}
伪状态还可以多个连用，达到逻辑与效果，例如当鼠标悬停在一个被选中的QCheckBox部件上时才应用规则，那么这个规则可以写为：
QCheckBox:hover:checked{color:white}
如果有需要，也可以使用逗号来表示逻辑或操作，例如：
QCheckBox:hover,QCheckBox:checked{color:white}
在Qt Style Sheets Reference关键字对应的帮助文档的List of Pseudo-States一项中列出了Qt支持的所有的伪状态。

4.例子
大家可以在Qt Style Sheets Examples页面找到很多相关的例子来学习，例如，下面是QSpinBox部件的一段样式表：
```

QSpinBox {
     padding-right: 15px; /* make room for the arrows */
     border-image: url(:/images/frame.png) 4;
     border-width: 3;
}
QSpinBox::up-button {
     subcontrol-origin: border;
     subcontrol-position: top right; /* position at the top right corner */
     width: 16px; /* 16 + 2*1px border-width = 15px padding + 3px parent border */
     border-image: url(:/images/spinup.png) 1;
     border-width: 1px;
}
QSpinBox::up-button:hover {
     border-image: url(:/images/spinup_hover.png) 1;
}
QSpinBox::up-button:pressed {
     border-image: url(:/images/spinup_pressed.png) 1;
}
QSpinBox::up-arrow {
     image: url(:/images/up_arrow.png);
     width: 7px;
     height: 7px;
}
QSpinBox::up-arrow:disabled, QSpinBox::up-arrow:off { /* off state when value is max */
    image: url(:/images/up_arrow_disabled.png);
}
QSpinBox::down-button {
     subcontrol-origin: border;
     subcontrol-position: bottom right; /* position at bottom right corner */
     width: 16px;
     border-image: url(:/images/spindown.png) 1;
     border-width: 1px;
     border-top-width: 0;
}
QSpinBox::down-button:hover {
     border-image: url(:/images/spindown_hover.png) 1;
}
QSpinBox::down-button:pressed {
     border-image: url(:/images/spindown_pressed.png) 1;
}
QSpinBox::down-arrow {
     image: url(:/images/down_arrow.png);
     width: 7px;
     height: 7px;
}
QSpinBox::down-arrow:disabled,
QSpinBox::down-arrow:off { /* off state when value in min */
    image: url(:/images/down_arrow_disabled.png);
}

```
结语
要想为软件设计一个漂亮的界面，需要灵活使用Qt样式表，不过这需要一定的CSS功底，还需要有美工经验。这一节只是简单介绍了下Qt中样式表的应用，只为抛砖引玉。大家也可以参考《QtCreator快速入门》第8章的相关内容，里面还涉及到了换肤、透明窗体、不规矩窗体等内容。

参考:http://bbs.qter.org/forum.php?mod=viewthread&tid=193
http://blog.hehehehehe.cn/a/9751.htm