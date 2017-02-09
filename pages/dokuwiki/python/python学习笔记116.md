title: python学习笔记116 

#  Python学习笔记之PyQt4Widget部件3Events 
PyQt中各种Widget（部件）所响应的Event（事件）通常由用户的交互行为产生。
QT中通过调用特定的事件处理函数将包含事件信息的QEvents子类实例传递给Widget（部件），如果该部件仅作为容器包含子部件而没有什么事件响应的动作，那该容器部件无需编写任何事件处理代码。
如果你希望检测到容器内对于子部件的鼠标单击行为,则需要在容器部件的mousePressEvent()中调用子部件的underMouse() 函数。The Scribble example implements a wider set of events to handle mouse movement, button presses, and window resizing。
以下从最常见的事件开始，简要介绍Qwidget的主要事件：
**Widget重绘时调用painEvent()函数。**显示定制内容的部件必须重载此函数。绘制使用的QPainter类只能在paintEvent()或paintEvent()的子函数内使用。
**部件大小调整后调用resizeEvent() 函数。**
光标位于Widget（部件）内或Widget（部件）通过grabMouse()获取鼠标焦点后，**一旦按下鼠标按钮，将调用mousePressEvent()函数**。Pressing the mouse without releasing it is effectively the same as calling grabMouse().
**鼠标按钮放开后调用mouseReleaseEvent()函数。**部件只有在接受鼠标单击事件的情况下，才会接受到该单击的释放事件。也就是说若有用户在部件中按下鼠标按钮不放，移动到部件外后再放开，这个部件也将会收到该鼠标按钮释放信息。这里有一个例外：当弹出菜单在鼠标按住时已显示，弹出菜单将接受鼠标释放事件。
**双击部件时调用mouseDoubleClickEvent()函数**。双击时部件先后接收到鼠标按下、放下事件，并将第二次的按下事件转化为双击事件（双击时鼠标移动的话还会发生鼠标移动事件）。很难从第一个单击鼠标的动作就能区分出双击事件（这也是为什么绝大多数的GUI界面设计的书籍建议双击最好设计成为单击事件扩展操作，而不是触发完全不同的操作。）
接受键盘输入的部件需要重载更多的事件处理函数：

**某个键位按下或按着不放的持续时间超过自重复时间的，将调用keyPressEvent()函数。**只有当部件不使用焦点转移（focus-change mechanisms）时，Tab或Shift+Tab的按键信息才会传递给部件。若你想把这些按键信息传递给部件，你需要重载 QWidget.event()。
部件获得键盘焦点（假定你调用了setFocusPolicy()函数）时调用**focusInEvent() 函数**。功能良好的部件在获取键盘焦点上通常采用一种简单明了的方式。
部件失去焦点时调用**focusOutEvent() 函数。**
除此以外，你可能需要重载一些不那么常用的事件处理函数。

**鼠标按键按下时，鼠标移动会调用mouseMoveEvent()函数。**多用于拖拽操作。若你调用了setMouseTracking(true)，那么即使鼠标按钮未被按下， 鼠标移动事件亦会被唤起(参阅 Drag and Drop 指南.)
键盘按键弹起后或按键被按着不放（若按键设置有自重复）会调用**keyReleaseEvent()函数**。在按住不放的情况向下，每个循环会收到两个弹起、一个按键事件。只有当部件不使用焦点转移（focus-change mechanisms）时，Tab或Shift+Tab的按键信息才会传递给部件。若你想把这些按键信息传递给部件，你需要重载 QWidget.event()。
**部件拥有焦点时，用户推动鼠标转轮将调用wheelEvent()函数。**
**当光标进入部件的屏幕空间范围时调用enterEvent()函数**。 (不包含子部件空间)
**当光标离开部件的屏幕空间范围时调用了leaveEvent()函数,光标进入子部件空间范围时不引起该事件。**
部件相对于其父部件移动后,调用moveEvent()
部件关闭或调用了close()函数后,调用closeEvent()
QEvent.Type的文档中还描述了一些相当隐蔽的事件,要处理这些事件,需要直接重载event().
event()函数的原始实现用于处理Tab和Shift+Tab (移动键盘焦点),并将其他绝大多数的事件传递给以上提高的专门处理函数. Events and the mechanism used to deliver them are covered in The Event System.

参考http://www.2sos.net/essay/PyQtGuide-Event.html