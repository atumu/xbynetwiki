title: js_dom总结一 

#  JS DOM总结一 
```

第1章 JavaScript简史
◇BOM浏览器对象模型，允许访问和操控浏览器窗口。
◇DOM文件对象模型，是一套对文档的内容进行抽象和概念化的方法。
◇DHTML动态HTML，结合HTML、CSS、JavaScript三种技术。
·利用HTML把网页标记为各种元素。
·利用CSS设计各有关元素的排版样式并确定它们在窗口中的显示位置。
·利用JavaScript实时地操控和改变各有关样式。
◇W3C推出的标准化DOM可以让任何一种程序设计语言对使用任何一种标记语言编写出来的任何一份文档进行操纵。
◇DOM是一种API（应用编程接口）。API：一组已经得到有关各方共同认可的基本约定（相当于现实世界的莫尔斯码、元素周期表）。
◇W3C对DOM的定义是：一个与系统平台和编程语言无关的接口，程序和脚本可以通过这个接口动态地对文档的内容、结构和样式进行访问和修改。

---------------------------------------------------------------------------------------

第2章 JavaScript语法
2.1 准备工作
◇将JS代码嵌在一份HTML/XHTML文档中：
①将js插入<head>部分的<script>标签间：<script type="text/javascript">js代码</script>
②把js代码存入一个独立文件，利用<script type="text/javascript" src="file.js"></script>
◇程序设计语言分为解释型和编译型两大类。Java或C++等语言需要一个编译器（编译型）。JavaScript（解释型），需要Web浏览器当解释器。
2.2 语法
2.3 语句
◇用分号结束语句。
◇注释：
·单行注释：" //注释句子 "
·多行注释：" /*注释句子*/ "
·HTML注释型风格（js中为单行注释）：" <- 注释句子 "；如果在html中嵌入则需要用“->”结束注释句。（不建议使用这种注释）
2.4 变量
◇JavaScript允许程序员直接对变量进行赋值而不提前声明。良好的变成习惯还是应该提前声明。用关键字var声明。
◇变量名区分大小写；变量名不允许包含空格或标点符号；变量名允许包含字母、数字、美元符号$和下划线符号_。
◇JavaScript为弱类型（weakly typed）语言：程序员可以随意改变某个变量的数据类型。
◇变量类型1-字符串：字符串必须放在引号内——单引号或双引号都允许使用。建议使用双引号，其中转移字符为反斜杠" \ " 。
◇变量类型2-数值：JS中为浮点数。
◇变量类型3-布尔值：两种可取值true或false。
◇变量类型4-数组：使用关键字Array声明，可定义长度。关联数组：通过在填充数组时为每个新元素明确地给出下标的方式来对数组值进行引用。
2.5 操作
◇算术符操作。其中符号" + "：可用于字符串进行字符串连接、字符串与数字连接。
2.6 条件语句
◇比较操作符：>、<、>=、<=、==；
◇逻辑操作符：&&、||、! ;
2.7 循环语句
◇while、do...while、for。
2.8 函数
◇变量与函数的命名：书的作者建议——变量用下划线分隔单词，函数用camelCase方法。C#中简单变量用camelCase，比较高级的命名用PascalCase。
◇在函数内部使用变量时，用var定义变量则为局部变量，不用var定义变量而直接使用则默认为使用全局变量。
2.9 对象
◇对象是自我包含的数据集合，包含在对象里的数据可以通过两种形式——即属性（property）和方法（method）访问：
·属性是隶属于某个特定对象的变量。
·方法是只有某个特定对象才能调用的函数。
◇对象就是由一些彼此相关的属性和方法集合在一起而构成的一个数据实体。
◇属性和方法需要使用“点”语法来访问。
◇使用new关键字创建对象的实例。对象是统称，实例是个体。
◇内建对象：js里预先定义好的对象，如Array、Date、Math等，有各自的属性和方法，可以帮助快速、简单地完成许多任务。
◇宿主对象：相对js来说就是Web浏览器提供的预定义对象。包括Form、Image、Element通过这些对象获得关于网页上表单、图像和各种表单元素信息。可以用js的document获得关于某给定网站上的任何一个元素信息。

---------------------------------------------------------------------------------------

第3章 DOM
3.2 对象模型中的"O"
◇对网页内容进行处理，用来实现这一目标的载体是document。
3.3 模型：DOM中的"M"
◇DOM中的"M"代表着"Model"（模型），含义是某种事物的表现形式。
◇DOM代表着被加载到浏览器窗口里的当前网页：浏览器向我们提供了当前网页的地图（或者说模型），而我们可以通过JavaScript去读取这张地图。
◇DOM把一份文档表示为一棵“家谱”树（节点）。
◇节点：
·元素节点：各类的元素标签。
·文本节点：部分元素节点内的文本。
·属性节点：元素内所包含的属性。
3.4 实用DOM方法
◇getElementById(id)方法：返回一个与那个有给定id属性值的元素节点相对应的对象。！注意返回的是对象！
◇getElementByTagName(tag)方法：返回一个对象数组，每个对象分别对应着文档里有给定标签的一个元素。！注意返回的是一个数组！
◇文档中每一个元素节点都是一个对象。
◇getAttribute(attribute)方法：通过上面两种方法查找到元素后，可以使用getAttribute()方法把它的各种属性的值查询出来。
·传入参数：所查询的属性的名字。
·区别：不能通过document对象调用，只能通过一个元素节点对象调用。
◇setAttribute(attribute,value)方法：允许我们对属性节点的值做出修改。
·使用此方法设置属性，查看源文件可以见属性值未被修改。
◇DOM工作模式：先加载文档的静态内容、再以动态方式对它们进行刷新，动态刷新不影响文档的静态内容。
◇DOM的威力：对页面内容的刷新不需要最终用户在它们的浏览器里执行页面刷新操作就可以实现。（AJAX）

---------------------------------------------------------------------------------------

第4章 实例研究：JavaScript美术馆
◇setAttribute()方法是“第1级DOM”（DOM Level 1）的组成部分之一，这个方法可以对任意元素节点的任意属性进行设置。
◇DOM是一种适用于多种环境和多种程序设计语言的通用性API。如果把DOM技巧运用在Web浏览器以外的应用环境里，严格遵守“第1级DOM”能够让你们避免与兼容性有关的任何问题。
◇事件处理函数：在预定义事件发生时让预先安排好的JavaScript代码开始执行，用术语说就是“触发一个动作”。onmourseover、onmourseon、onclick等。
◇this关键字：含义是“这个对象”。
◇事件处理：event = "JavaScript statement(s)"。JS代码包含在一对引号之间，可以放置任意JS语句，各条语句用分号隔开。
◇事件处理函数工作机制：在给某个元素添加了事件处理函数后，一旦发生预定事件，相应的JavaScript代码就会得到执行；那些JavaScript代码可以返回一个结果，而这个结果将被传递回那个事件处理函数。
◇childNodes属性让我们可以从给定文档的节点树里把任何一个元素的所有子元素检索出来。childNodes属性将返回一个数组，这个数组包含给定元素节点的全体子元素。调用：element.childNodes
◇childNodes属性返回的数组包含所有类型的节点，除了所有的元素节点，所有的属性节点和文本节点也包含在其中。
◇nodeType属性：用数字返回所指定的节点的种类。元素节点为1;属性节点为2；文本节点为3。
◇nodeValue属性：用于检索和设置某个文本节点的值。调用：node.nodeValue
◇firstChild和lastChild属性：选择数组第一个或最后一个元素。

---------------------------------------------------------------------------------------

第5章 JavaScript编程原则和良好习惯
5.1
◇预留退路：确保网页在没有JavaScript的情况下也能正常工作。
◇分离JavaScript：把网页的结构和内容与JavaScript脚本的动作行为分开。
◇向后兼容性：确保老版本的浏览器不会因为你的JavaScript脚本而死机。
5.2 预留退路
◇JavaScript使用window对象的open()方法来创建新的浏览器窗口。window.open(url,name,features)，三个可选参数。
  ·url：想在新窗口里打开的那份文档的URL地址。如果省略这个参数，屏幕上将弹出一个空白的浏览器窗口。
  ·name：新窗口的名字。可以在代码里通过这个名字与新窗口进行通信。
  ·features：以逗号分隔的字符串，其内容是新窗口的各种属性。这些属性包括窗口的尺寸（宽度和高度）以及窗口被激活或禁用的各种浏览功能（工具条、菜单条、初始显示位置，等等）。参数使用原则：新窗口的浏览功能要少而精。
◇JavaScript伪协议（不建议使用）：让我们可以通过一个链接来使用JavaScript函数。
◇内嵌的事件处理函数（不建议使用）：html中内嵌onlick等事件处理函数来调用JavaScript。
5.4 分离JavaScript
◇利用Class和id属性作为“挂钩”在分离的js中操纵。
5.5 向后兼容性
◇对象检测（object detection）：对于不能使用js或部分dom方法的古老浏览器，通过返回false推出js，避免崩溃。
◇浏览器嗅探技术（请改用对象检测技术）：查找浏览器提供的信息来解决向后兼容问题的办法。

---------------------------------------------------------------------------------------
第6章 案例研究：JavaScript美术馆改进版
◇预留退路问题：保证js不能执行前提下不影响原本的功能。案例里为href属性添加链接。
◇分离JS代码：设置挂钩。
◇添加事件处理函数。
◇进行必要的检查：对需要用到的DOM方法进行检查(getElementsByTagName等)，对挂钩的存在进行检查，若不存在返回false不执行js。
◇创建必要的变量。
◇创建循环：很多时候用i作为循环的控制变量，含义“increment”。
◇完成必要的操作。
◇把多个JS函数绑定到onload事件处理函数上：
  · window.onload = function1();window.onload = function2();执行时仅执行function2，即每个事件处理函数只能绑定一条指令。
  ·解决方法1：window.onload = function() { firstFunction(); secondFunction();}
  ·解决方法2：使用addLoadEvent()函数，请理解此函数具体的编写代码。
◇JavaScript函数的优化：充分考虑各种检查，同时不要做太多的假设。
◇慎用onkeypress：这个事件很容易引起问题，很多浏览器连tab都引起这个事件。无特殊理由，不要是用onkeypress，onclick可以解决几乎所有问题，包括键盘浏览功能。
6.5 DOM Core和HTML-DOM
◇之前所运用的getElementById()方法、getElementsByTagName()方法、getAttribute()方法、setAttribute()方法都是DOM Core的组成部分，并不专属于JavaScript，支持DOM的任何一种程序设计语言都可以使用它们。不仅限于处理网页，还可以处理任何一种标记语言（XML）编写的文档。
◇HTML-DOM提供了简明的记号来描述各种HTML元素的属性。

---------------------------------------------------------------------------------------

第7章 动态创建HTML内容
◇document.write()方法：可以方便快捷地把字符串插入到文档里，但其使用方法违背了“分离JavaScript”原则，因此避免使用这个方法。
◇innerHTML属性：非W3C DOM标准，可以用来读、写某给定元素里的HTML内容。
◇深入剖析DOM方法：createElement()、createTextNode()、appendChild()和insertBefore()。
◇浏览器呈现的DOM节点树，而不是右键属性查看的源文件。因此，动态创建HTML内容实际上并不是在创建HTML内容，而是在改变DOM节点树。
◇createElement()方法：语法：document.creatElement(nodeName);创建一个元素节点，并且返回一个指向该节点的指针。这个节点已经具有一个nodeType和nodeName值。
◇appendChild()方法：语法：parent.appendChild(child);把某个节点变成另一个节点的子节点。
◇createTextNode()方法：语法：document.creatTextNode(text);创建一个文本节点，返回该节点一个指针。
◇insertBefore()方法：语法：parentElement.insertBefore(newElement,targetElement);在某个元素节点前添加另一个元素节点。
◇DOM没有提供insertAfter()方法，可以利用一下DOM方法和属性编写：parentNode属性、lastChild属性、appendChild()方法、insertBefore()方法、nextSibling属性。
◇nextSibling属性：目标元素的下一个兄弟节点。

---------------------------------------------------------------------------------------

第8章 充实文档的内容
◇不能滥用DOM，仅记“循序渐进”和“预留退路”。
◇自顶向下逐步求精，先搭建js基本实现功能，再进行对象检测，预留退路的设计和必要的注释。
◇for/in循环：它可以把某个数组的下标（键字）临时地赋予给一个变量值。语法：for(variable in array)

---------------------------------------------------------------------------------------
 
第9章 CSS-DOM
◇每个元素节点有一个style属性，即内嵌的CSS样式属性。可以通过：element.style.property进行读写。这个style属性是一个对象。
◇当在js中操纵style属性时，像font-famliy这样的属性不能直接出现在js中，因为js保留-、+这些符号作运算符。js中做法是使用“Camel”记号来表示那些名字里有多个单词的CSS属性。例font-family属性写作fontFamily。
◇DOM style属性只能操纵内嵌式style，不能对外部css文件及嵌在head内的css进行操作。
◇使用elemennt.sytle.property = value;可以为css属性刷新值。如果value是属性值，需要单引号或双引号括起来，如果是变量就不能有引号。
◇事件响应：使用属性设置：onmouseover、onmouseout。用法：para.onmouseover=fucntion(){}。
◇不应该用DOM直接添加/修改style的值，而是应该使用className属性为元素节点添加/追加/刷新属性，再用对应CSS控制表现。
◇所有可以抽象化的具体函数都应该进行抽象化，使其成为通用函数模块化重用。

---------------------------------------------------------------------------------------
 
第10章 用JavaScript实现动画效果
◇JS可以通过按照预定的时间间隔重复调用一个函数，从而随时间推移改变某个元素样式，即创建动画。
◇setTimeout("function",interval)：第一个参数是字符串，内容是将要执行的那个函数的名字。第二个参数是一个值，它以毫秒为单位设定需要经过多长时间才开始执行由第一个参数所给出的函数。
◇clearTimeout(variable)：可以取消某个正在排队等待执行的函数，需要将要取消的setTimeout函数赋值给一个变量，再在clearTimeout里调用这个变量作参数。
◇parseInt()函数：用法parseInt(string)，可以提取字符串里的数值信息，其返回值永远是整数。
◇slideShow中实例应用到的技巧：清除setTimeout的方法——利用元素创建属性。
◇ceil属性的语法：
  ·Math.ceil(number)：把浮点数number向“大于”方向舍入为与之最接近的整数。
  ·Math.floor(number)：把浮点数number向“小于”方向舍入为与之最接近的整数。
  ·Math.round(number)：把浮点数舍入为与之最接近的整数（四舍五入？）。

---------------------------------------------------------------------------------------

第11章 学以致用：JavaScript网站设计实战
◇构建通用模板。
◇用gif保存图像。
◇CSS文件分三个：
  ·页面布局CSS(layout.css)
  ·颜色相关CSS(color.css)
  ·文本字形相关CSS(typography.css)
  ·用@import url(xxx.css)把上面的样式表引入一个基本样式表。
◇如果为某个元素设置了一种前景颜色，就应该再为它设置一种背景颜色。防止某些文本内容变成“隐形”文字。
◇background-attachment 属性设置背景图像是否固定或者随着页面的其余部分滚动。
◇把反复多次地用到的js函数集中全部放入global.js文件。
◇window.location.href可以得到当前浏览器的url。
◇indexOf()方法可以在一个字符串里寻找一个子串的位置：string.indexOf(substring),这里返回的是
子串substring在字符串string里第一次出现的位置，若没有找到匹配此方法将返回-1。
◇element.lastChild.nodeValue.toLowerCase();把某个文本节点转换为小写，这些有利于文本串的判断。
◇算法是js的关键，创建自己的库，并且附带算法。
◇遇到作用域问题时，有时候不在同一作用域里变量调用不了，此时不一定要使用全局变量，可以为相关元素定义属性变量，再于其他函数调用。
◇HTML-DOM最有用的对象之一：Form对象。
   ·form.elements.length：返回某给定的form元素里表单元素的总个数。
   ·form.elements：返回某给定的form元素里包含的所有表单元素的数组。
  ·element.value：包含该表单元素的当前值。
   ·element.defaultValue：包含该表单的初始值。
◇编写用来检查表单数据的JavaScript脚本时，必须记住：
   ·糟糕的表单检查还不如根本没有检查。
   ·不要完全依赖JavaScript，它替代不了服务器端得数据合法性检查。你已经用JavaScript对表单数据进行过检查，绝不意味着你在那些数据到达服务器时用不着再对它们进行检查。

---------------------------------------------------------------------------------------
 
第12章 展望DOM脚本编程技术
◇Ajax

---------------------------------------------------------------------------------------
 
附录一 DOM方法（常用DOM方法，非全体）
◇创建节点
  ·createElement()：按照给定的标签名创建一个新的元素节点。这个方法的返回值是一个指向新建元素节点的引用指针：Reference = document.createElement(element);创建的节点没有添加到文档里，只存在于JavaScript上下文里的DocumentFragment对象。
  ·createTextNode()：创建一个包含着给定文本的新文本节点。这个方法返回值是一个指向新建文本节点的引用指针：reference=document.createTextNode(text);
◇复制节点
  ·cloneNode()：将为给定的节点创建一个副本。这个方法返回值是一个指向新建克隆节点的引用指针：reference=node.cloneNode(deep);参数deep是一个布尔型参数，取值只可能为true或false，这个参数决定是否要把被复制节点的子节点也一同复制到新建节点里去。
◇插入/剪切节点
  ·appendChild()：给元素节点追加一个子节点，这个方法的返回值是一个指向新增子节点的引用指针：reference=element.appendChild(newChild);还可剪切原有节点。
  ·insertBefore()：把一个给定节点插入到一个给定元素节点的给定子节点前面，它返回一个指向新增子节点的引用指针。Reference=element.insertBefore(newNode,targetNode);
◇删除节点
  ·removeChild()：从一个给定元素里删除一个子节点，返回一个指向已被删除的子节点的引用指针。Reference=element.removeChild(node);
◇替换节点
  ·replaceChild()：把一个给定父元素里的一个子节点替换为另外一个节点，返回一个指向已被替换的那个子节点的引用指针。Reference=element.replaceChild(newChild,oldChild);
◇设置属性节点
  ·setAttribute()：为给定元素节点添加一个新的属性值或是改变它的现有属性的值。属性的名字和值必须以字符串形式传递给此方法。Element.setAttribute(attributeName,attributeValue);
◇查找节点
  ·getAttribute()：返回一个给定元素的一个给定属性节点的值，必须以字符串的形式传递给这个方法。attributeValue=element.getAttribute(attributeName);
  ·getElementById()：寻找一个有着给定id属性值的元素，返回一个有着给定id值的节点（对象）：element=document.getElementById(ID);
  ·getElementsByTagName()：寻找有着给定标签名的所有元素，返回一个节点集合数组，数组里每个元素都是一个对象。Elements=document.getElementSByTagName(tagName);
  ·hasChildNodes：检查一个给定元素是否有子节点，返回一个布尔值true或false。booleanValue=element.hasChildNodes;
 
--------------------------------------------------------------------------------------- 

附录二 DOM属性（常用DOM属性，非全体）
◇节点的属性
  ·nodeName：属性返回一个字符串，其内容是给定节点的名字：name=node.nodeName;
  ·nodeType：属性返回一个整数，这个数值代表着给定节点的类型：integer=node.nodeType;
       — 1.  ELEMENT_NODE
       — 2. ATTRIBUTE_NODE
       — 3.  TEXT_NODE
       — 4.  CDATA_SECTION_NODE
       — 5.  ENTITY_REFERENCE_NODE
       — 6.  ENTITY_NODE
       — 7.  PROCESSING_INSTRUCTION_NODE
       — 8.  COMMENT_NODE
       — 9.  DOCUMENT_NODE
       — 10.  DOCUMENT_TYPE_NODE
       — 11.  DOCUMENT_FRAGMENT_NODE
       — 12.  NOTATION_NODE
  ·nodeValue：返回给定节点的当前值，返回的是一个字符串。nodeValue是一个读/写属性。元素节点返回null，不能赋予值。Value=node.nodeValue;
◇遍历节点树
  ·childNodes：属性将返回一个数组，这个数组由给定元素节点的子节点构成。是一个只读属性。nodeList=node.childNodes;
  ·firstChild：属性将返回一个给定元素节点的第一个子节点，返回一个节点对象的引用指针。是一个只读属性。Reference=node.firstChild;
  ·lastChild：属性将返回一个给定元素节点的最后一个子节点，返回一个节点对象的引用指针。是一个只读属性。Reference=node.lastChild;
  ·nextSibling：属性将返回一个给定节点的下一个子节点，返回一个节点对象的引用指针。是一个只读属性。Reference=node.nextSibling;
  ·parentNode：属性将返回一个给定节点的下一个父节点，返回一个节点对象的引用指针。是一个只读属性。Reference=node.parentNode;
  ·previousSibling：属性将返回一个给定节点的前一个子节点，返回一个节点对象的引用指针。是一个只读属性。Reference=node.previousSibling;

```

##  JavaScript内置函数 
javascript函数一共可分为五类：
　　 ·常规函数
　　 ·数组函数
　　 ·日期函数
　　 ·数学函数
　　 ·字符串函数

```

---------------------------------------------------------------------------------------

1.常规函数
　　 javascript常规函数包括以下9个函数：
　　 (1)alert函数：显示一个警告对话框，包括一个OK按钮。
　　 (2)confirm函数：显示一个确认对话框，包括OK、Cancel按钮。
　　 (3)escape函数：将字符转换成Unicode码。
　　 (4)eval函数：计算表达式的结果。
　　 (5)isNaN函数：测试是(true)否(false)不是一个数字。
　　 (6)parseFloat函数：将字符串转换成符点数字形式。
　　 (7)parseInt函数：将符串转换成整数数字形式(可指定几进制)。
　　 (8)prompt函数：显示一个输入对话框，提示等待用户输入。例如：
　　 <script language="javascript">
　　 <!--
　　 alert("输入错误");
　　 prompt("请输入您的姓名","姓名");//（标题，预设值）
　　 confirm("确定否！");
　　 //-->
　　 </script>
　　 (9)unescape函数：解码由escape函数编码的字符。

---------------------------------------------------------------------------------------

2.数组函数
　　 javascript数组函数包括以下4个函数：
　　 (1)join函数：转换并连接数组中的所有元素为一个字符串。例:
　　　　 function JoinDemo()
　　　　 {
　　　　　 var a, b;
　　　　　 a = new Array(0,1,2,3,4);
　　　　　 b = a.join("-");//分隔符
　　　　　 return(b);//返回的b=="0-1-2-3-4"
　　　　 } 
　　 (2)langth函数：返回数组的长度。例：
　　　　 function LengthDemo()
　　　　 {
　　　　　 var a, l;
　　　　　 a = new Array(0,1,2,3,4);
　　　　　 l = a.length;
　　　　　 return(l);//l==5
　　　　 } 
　　 (3)reverse函数：将数组元素顺序颠倒。例：
　　　 function ReverseDemo()
　　　 {
　　　　 var a, l;
　　　　 a = new Array(0,1,2,3,4);
　　　　 l = a.reverse();
　　　　 return(l);
　　　 } 
　　 (4)sort函数：将数组元素重新排序。例：
　　　　 function SortDemo()
　　　　 {
　　　　　 var a, l;
　　　　　 a = new Array("X" ,"y" ,"d", "Z", "v","m","r");
　　　　　 l = a.sort();
　　　　　 return(l);
　　　　 } 

---------------------------------------------------------------------------------------

3.日期函数
　　 javascript日期函数包括以下20个函数：
　　 (1)getDate函数：返回日期的“日”部分，值为1～31。例：
　　　 function DateDemo()
　　　 {
　　　　 var d, s = "Today's date is: ";
　　　　 d = new Date();
　　　　 s += (d.getMonth() + 1) + "/";
　　　　 s += d.getDate() + "/";
　　　　 s += d.getYear();
　　　　 return(s);
　　　 } 
　　 (2)getDay函数：返回星期几，值为0～6，其中0表示星期日，1表示星期一，...，6表示星期六。例：
　　　 function DateDemo()
　　　 {
　　　　 var d, day, x, s = "Today is: ";
　　　　 var x = new Array("Sunday", "Monday", "Tuesday");
　　　　 var x = x.concat("Wednesday","Thursday", "Friday");
　　　　 var x = x.concat("Saturday");
　　　　 d = new Date();
　　　　 day = d.getDay();
　　　　 return(s += x[day]);
　　　 } 
　　 (3)getHours函数：返回日期的“小时”部分，值为0～23。例。
　　　 function TimeDemo()
　　　 {
　　　　 var d, s = "The current local time is: ";
　　　　 var c = ":";
　　　　 d = new Date();
　　　　 s += d.getHours() + c;
　　　　 s += d.getMinutes() + c;
　　　　 s += d.getSeconds() + c;
　　　　 s += d.getMilliseconds();
　　　　 return(s);
　　　 } 
　　 (4)getMinutes函数：返回日期的“分钟”部分，值为0～59。见上例。
　　 (5)getMonth函数：返回日期的“月”部分，值为0～11。其中0表示1月，2表示3月，...，11表示12月。见前面的例子。
　　 (6)getSeconds函数：返回日期的“秒”部分，值为0～59。见前面的例子。
　　 (7)getTime函数：返回系统时间。
　　　 function GetTimeTest()
　　　 {
　　　　 var d, s, t;
　　　　 var MinMilli = 1000 * 60;
　　　　 var HrMilli = MinMilli * 60;
　　　　 var DyMilli = HrMilli * 24;
　　　　 d = new Date();
　　　　 t = d.getTime();
　　　　 s = "It's been "
　　　　 s += Math.round(t / DyMilli) + " days since 1/1/70";
　　　　 return(s);
　　　 } 
　　 (8)getTimezoneOffset函数：返回此地区的时差(当地时间与GMT格林威治标准时间的地区时差)，单位为分钟。
　　　 function TZDemo()
　　　 {
　　　　 var d, tz, s = "The current local time is ";
　　　　 d = new Date();
　　　　 tz = d.getTimezoneOffset();
　　　　 if (tz < 0)
　　　　 s += tz / 60 + " hours before GMT";
　　　　 else if (tz == 0)
　　　　 s += "GMT";
　　　　 else
　　　　 s += tz / 60 + " hours after GMT";
　　　　 return(s);
　　　 } 
　　 (9)getYear函数：返回日期的“年”部分。返回值以1900年为基数，例如1999年为99。前面有例子。
　　 (10)parse函数：返回从1970年1月1日零时整算起的毫秒数(当地时间)。
　　　 function GetTimeTest(testdate)
　　　 {
　　　　 var d, s, t;
　　　　 var MinMilli = 1000 * 60;
　　　　 var HrMilli = MinMilli * 60;
　　　　 var DyMilli = HrMilli * 24;
　　　　 d = new Date();
　　　　 t = Date.parse(testdate);
　　　　 s = "There are "
　　　　 s += Math.round(Math.abs(t / DyMilli)) + " days "
　　　　 s += "between " + testdate + " and 1/1/70";
　　　　 return(s);
　　　 } 
　　 (11)setDate函数：设定日期的“日”部分，值为0～31。
　　 (12)setHours函数：设定日期的“小时”部分，值为0～23。
　　 (13)setMinutes函数：设定日期的“分钟”部分，值为0～59。
　　 (14)setMonth函数：设定日期的“月”部分，值为0～11。其中0表示1月，...，11表示12月。
　　 (15)setSeconds函数：设定日期的“秒”部分，值为0～59。
　　 (16)setTime函数：设定时间。时间数值为1970年1月1日零时整算起的毫秒数。
　　 (17)setYear函数：设定日期的“年”部分。
　　 (18)toGMTString函数：转换日期成为字符串，为GMT格林威治标准时间。
　　 (19)setLocaleString函数：转换日期成为字符串，为当地时间。
　　 (20)UTC函数：返回从1970年1月1日零时整算起的毫秒数，以GMT格林威治标准时间计算。

---------------------------------------------------------------------------------------

4.数学函数
　　 javascript数学函数其实就是Math对象，它包括属性和函数(或称方法)两部分。其中，属性主要有下列内容。
　　 Math.e:e(自然对数)、Math.LN2（2的自然对数)、Math.LN10(10的自然对数)、Math.LOG2E(e的对数，底数为2)、Math.LOG10E(e的对数，底数为10)、Math.PI(π)、Math.SQRT1_2(1/2的平方根值)、Math.SQRT2(2的平方根值)。
　　 函数有以下18个：
　　 (1)abs函数：即Math.abs(以下同)，返回一个数字的绝对值。
　　 (2)acos函数：返回一个数字的反余弦值，结果为0～π弧度(radians)。
　　 (3)asin函数：返回一个数字的反正弦值，结果为-π/2～π/2弧度。
　　 (4)atan函数：返回一个数字的反正切值，结果为-π/2～π/2弧度。
　　 (5)atan2函数：返回一个坐标的极坐标角度值。
　　 (6)ceil函数：返回一个数字的最小整数值(大于或等于)。
　　 (7)cos函数：返回一个数字的余弦值，结果为-1～1。
　　 (8)exp函数：返回e(自然对数)的乘方值。
　　 (9)floor函数：返回一个数字的最大整数值(小于或等于)。
　　 (10)log函数：自然对数函数，返回一个数字的自然对数(e)值。
　　 (11)max函数：返回两个数的最大值。
　　 (12)min函数：返回两个数的最小值。
　　 (13)pow函数：返回一个数字的乘方值。
　　 (14)random函数：返回一个0～1的随机数值。
　　 (15)round函数：返回一个数字的四舍五入值，类型是整数。
　　 (16)sin函数：返回一个数字的正弦值，结果为-1～1。
　　 (17)sqrt函数：返回一个数字的平方根值。
　　 (18)tan函数：返回一个数字的正切值。

---------------------------------------------------------------------------------------

5.字符串函数
　　 javascript字符串函数完成对字符串的字体大小、颜色、长度和查找等文明作，共包括以下20个函数：
　　 (1)anchor函数：产生一个链接点(anchor)以作超级链接用。anchor函数设定<A NAME...>的链接点的名称，另一个函数link设定<A HREF=...>的URL地址。
　　 (2)big函数：将字体加到一号，与<BIG>...</BIG>标签结果相同。
　　 (3)blink函数：使字符串闪烁，与<BLINK>...</BLINK>标签结果相同。
　　 (4)bold函数：使字体加粗，与<B>...</B>标签结果相同。
　　 (5)charAt函数：返回字符串中指定的某个字符。
　　 (6)fixed函数：将字体设定为固定宽度字体，与<TT>...</TT>标签结果相同。
　　 (7)fontcolor函数：设定字体颜色，与<FONT COLOR=color>标签结果相同。
　　 (8)fontsize函数：设定字体大小，与<FONT SIZE=n>标签结果相同。
　　 (9)indexOf函数：返回字符串中第一个查找到的下标index，从左边开始查找。
　　 (10)italics函数：使字体成为斜体字，与<I>...</I>标签结果相同。
　　 (11)lastIndexOf函数：返回字符串中第一个查找到的下标index，从右边开始查找。
　　 (12)length函数：返回字符串的长度。(不用带括号)
　　 (13)link函数：产生一个超级链接，相当于设定<A HREF=...>的URL地址。
　　 (14)small函数：将字体减小一号，与<SMALL>...</SMALL>标签结果相同。
　　 (15)strike函数：在文本的中间加一条横线，与<STRIKE>...</STRIKE>标签结果相同。
　　 (16)sub函数：显示字符串为下标字(subscript)。
　　 (17)substring函数：返回字符串中指定的几个字符。
　　 (18)sup函数：显示字符串为上标字(superscript)。
　　 (19)toLowerCase函数：将字符串转换为小写。
　　 (20)toUpperCase函数：将字符串转换为大写。

```