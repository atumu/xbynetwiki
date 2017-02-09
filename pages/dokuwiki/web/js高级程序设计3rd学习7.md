title: js高级程序设计3rd学习7 

#  JS高级程序设计3rd学习7DOM、事件、表单 
DOM(文档对象模型)描述了一个层次化的节点树。
##  document对象
**文档信息**
document.title
document.URL
document.domain
document.referer;
**查找元素**
document.getElementById
document.getElementByTagName
**特殊集合**
document.anchors :包含文档中所有带name特性的<a >元素
document.forms
document.images
document.links.:包含文档中所有带href特性的<a >元素
**文档写入:**
write(),writeLen(),open(),close()
**创建元素：**
```

var div=document.createElement("div");
div.id="myId";
div.className="box";
document.body.appendChild(div);

```
##  动态脚本与动态样式 
动态脚本指的是页面加载时不存在，但将来的某一个时刻通过修改DOM动态添加<script >标签以达到添加脚本的目的。
```

var script=document.createElement("script");
script.type="text/javascript";
script.src="client.js";
document.body.appendChild(script);// var head=document.getElementByTagName("head")[0]; head.appendChild(script);

```
动态样式
```

var link=document.createElement("link");
link.rel="stylesheet";
link.type="text/css";
link.href="style.css";
var head=document.getElementByTagName("head")[0]; 
head.appendChild(script);

```
##  HTML5选择器API 
querySelector(),querySelectorAll()支持css选择器，返回元素或元素列表。
##  HTMLDocument变化 
readyState属性.document.readyState有2个值loading,complete
head属性:document.head相当于document.getElementByTagName("head")[0]; 
字符集属性:document.charset
元素的innerHTML属性.div.innerHTML="<p>aaa</p>";(不包含元素本身)
元素的outerHTML属性.div.outerHTML;(包含元素本身)

###  自定义数据属性 
HTML5规定可以为元素添加非标准的属性，但要添加前缀data-,目的是为元素提供与渲染无关的一些信息。这些属性可以任意添加、随便命名，只要以data-开头即可。
```

<div id="myId" data-appId="1234" data-name="name"/>
var div=document.getElementById("myId");
var appId=div.dataset.appId;//可以通过元素的dataset属性来访问自定义属性的值。
var name=div.dataset.name;

//设置值
div.dataset.appId=234

```
###  scrollIntoView滚动内容到顶部 
```

<button id="button1">点击本按钮，图片下面的框框1秒钟变出来</button>
<img src="http://ww1.sinaimg.cn/bmiddle/6aa99f00tw1dkxpxgi4mcj.jpg" height="969" />
<span id="block" class="block"></span>
JS代码：
document.getElementById("button1").onclick = function() {
    document.getElementById("block").scrollIntoView();
};

```
示例:http://www.zhangxinxu.com/study/201109/scroll-into-view.html
###  滚动大小 
scrollHeight、scrollWidth用于确定元素内容的实际大小
scrollLeft
scrollTop
具体参考http://www.jb51.net/article/30303.htm

##  事件 
事件流几种形式:
事件冒泡（event bubbling）、事件捕获(event capturing)、DOM事件流
事件冒泡：即事件开始时由最具体的元素（文档中嵌套层次最深的那个节点）接收，然后**逐级向上传播**到较为不具体的节点（文档）。
事件捕获：**事件捕获的思想是不太具体的节点应该更早的接收到事件，而最具体的节点应该在最后接收到节点。**事件捕获的用意在于事件到达预定目标之前捕获它。
DOM事件流：“DOM2级事件流”规定**的事件流包括三个阶段：事件捕获阶段、处于目标阶段和冒泡阶段。**首先发生的是事件捕获，为截获事件提供了机会。然后是实际的目标接收到事件。最后一个阶段是冒泡阶段，可以在这个阶段对事件作出响应。以简单的HTML页面为例，单击<div>元素会按照下图顺序触发事件
![](/data/dokuwiki/web/pasted/20160306-120656.png)
Opera、Firefox、Chrome和Safari都支持DOM事件流；IE不支持DOM事件流。
###  Event对象 
Event对象在event第一次触发的时候被创建出来，并且一直伴随着事件在DOM结构中流转的整个生命周期。event对象会被作为第一个参数传递给事件监听的回调函数。我们可以通过这个event对象来获取到大量当前事件相关的信息：
  * type (String) — 事件的名称
  * target (node) — 事件起源的DOM节点
  * **currentTarget?(node) — 当前回调函数被触发的DOM节点**，在事件处理程序内部，this始终等于event.currentTarget的值
  * bubbles (boolean) — 指明这个事件是否是一个冒泡事件（接下来会做解释）
  * **preventDefault(function) — 这个方法将阻止浏览器中用户代理对当前事件的相关` 默认行为 `被触发。比如阻止<a>元素的click事件加载一个新的页面**
  * **stopPropagation (function) — 这个方法将` 阻止当前事件链 `上后面的元素的回调函数被触发，当前节点上针对此事件的其他回调函数依然会被触发。（**我们稍后会详细介绍。）
  * stopImmediatePropagation (function) — 这个方法将阻止当前事件链上所有的回调函数被触发，也包括当前节点上针对此事件已绑定的其他回调函数。
  * cancelable (boolean) — 这个变量指明这个事件的默认行为是否可以通过调用event.preventDefault来阻止。也就是说，只有cancelable为true的时候，调用event.preventDefault才能生效。
  * defaultPrevented (boolean) — 这个状态变量表明当前事件对象的preventDefault方法是否被调用过
  * isTrusted (boolean) — 如果一个事件是由设备本身（如浏览器）触发的，而不是通过JavaScript模拟合成的，那个这个事件被称为可信任的(trusted)
  * eventPhase (number) — 这个数字变量表示当前这个事件所处的阶段(phase):none(0), capture(1),target(2),bubbling(3)。我们会在下一个部分介绍事件的各个阶段
  * timestamp (number) — 事件发生的时间
此外事件对象还可能拥有很多其他的属性，但是他们都是针对特定的event的。比如，**鼠标事件包含` clientX和clientY `属性来表明鼠标在当前视窗的位置。**
###  停止传播（Stopping Propagation） 
```

child.addEventListener('click', function(event) {
 event.stopPropagation();
});
parent.addEventListener('click', function(event) {
// If the child element is clicked
// this callback will not fire
});

```
可以通过调用事件对象的stopPropagation方法，在任何阶段（捕获阶段或者冒泡阶段）中断事件的传播。此后，事件不会在后面传播过程中的经过的节点上调用任何的监听函数。
###  阻止浏览器默认行为 
当特定事件发生的时候，浏览器会有一些默认的行为作为反应。**最常见的事件不过于link被点击。当一个click事件在一个<a>元素上被触发时，它会向上冒泡直到DOM结构的最外层document，浏览器会解释href属性，并且在窗口中加载新地址的内容。**
在web应用中，开发人员经常希望能够自行管理导航（navigation）信息，而不是通过刷新页面。为了实现这个目的**，我们需要阻止浏览器针对点击事件的默认行为，而使用我们自己的处理方式。**这时，我们就需要调用` event.preventDefault(). `
```

anchor.addEventListener('click', function(event) {
  event.preventDefault();
// Do our own thing
});

```
我们可以阻止浏览器的很多其他默认行为。比如，我们可以在HTML5游戏中阻止敲击空格时的页面滚动行为，或者阻止文本选择框的点击行为。
**调用event.stopPropagation()只会阻止传播链中后续的回调函数被触发。它不会阻止浏览器的自身的行为。**
###  一些有用的事件 
**load**
load事件可以在任何资源（包括被依赖的资源）被加载完成时被触发，**这些资源可以是图片，css，脚本，视频，音频等文件，也可以是document或者window。**
```

image.addEventListener('load', function(event) {
  image.classList.add('has-loaded');
});

```
**onbeforeunload**
` window.onbeforeunload让开发人员可以在想用户离开一个页面的时候进行确认。 `这个在有些应用中非常有用，比如用户不小心关闭浏览器的tab，我们可以要求用户保存他的修改和数据，否则将会丢失他这次的操作。
```

window.onbeforeunload = function(event) {
  var msg="你确定要离开此页面?";
  event.returnValue=msg; //IE与firefox
  return msg; //safari与chrome
};

```
需要注意的是，对页面添加onbeforeunload处理会导致浏览器不对页面进行缓存?，这样会影响页面的访问响应时间。 同时，onbeforeunload的处理函数必须是同步的（synchronous）。

**resize**
在一些**复杂的响应式布局**中，对window对象监听resize事件是非常常用的一个技巧。仅仅通过css来达到想要的布局效果比较困难。很多时候，我们需要使用JavaScript来计算并设置一个元素的大小。
```

window.addEventListener('resize', function() {
  
// update the layout
});
window.onresize=function(event){}

```
参考:http://blog.jobbole.com/52430/

###  HTML5中的contextmenu事件：上下文菜单 
```

       window.onload=function(event){
            var div = document.getElementById("myDiv");
           div.addEventListener("contextmenu", function(event){
            	//阻止默认行为(显示浏览器右键菜单),我们需要显示上下文菜单
                event.preventDefault();
                var menu = document.getElementById("myMenu");
                menu.style.left = event.clientX + "px";//获取鼠标在视口的位置，并将上下文菜单显示在那里
                menu.style.top = event.clientY + "px";
                menu.style.visibility = "visible";
            });
            //添加一个添加隐藏上下文菜单的事件处理
            document.addEventListener("click", function(event){
                document.getElementById("myMenu").style.visibility = "hidden";
            });
        }

```

###  几个鼠标事件位置的区别 
视口位置:event.clientX,event.clientY
页面坐标位置：页面本身可能会大于视口viewport.所以是有区别的。event.pageX,event.pageY.在页面没有滚动的情况下，视口位置和页面坐标位置含义一直。
屏幕坐标位置:相对与整个电脑屏幕的位置:screenX,scrennY

###  键码与键的字符编码 
event.keyCode,只有在keydown和keyup事件时才会包含
event.charCode,只有在keypress事件是才会包含
常用键码列表:
回车键(Enter)对应于13
可以参考:http://xiaotao-2010.iteye.com/blog/1818668

##  表单 
form元素的属性
  * enctype:请求编码类型.application/x-www-form-urlencoded默认编码，multipart/form-data 在使用包含文件上传控件的表单时，必须使用该值。
  * method:HTTP请求类型.get或post
  * name:表单名
  * action:接受请求的URL
  * reset()
  * submit()
禁止多次提交表单
###  表单序列化为get参数 
```

        function serialize(form){        
            var parts = [],
                field = null,
                i,
                len,
                j,
                optLen,
                option,
                optValue;
            
            for (i=0, len=form.elements.length; i < len; i++){
                field = form.elements[i];
            
                switch(field.type){
                    case "select-one":
                    case "select-multiple":
                    
                        if (field.name.length){
                            for (j=0, optLen = field.options.length; j < optLen; j++){
                                option = field.options[j];
                                if (option.selected){
                                    optValue = "";
                                    if (option.hasAttribute){
                                        optValue = (option.hasAttribute("value") ? option.value : option.text);
                                    } else {
                                        optValue = (option.attributes["value"].specified ? option.value : option.text);
                                    }
                                    parts.push(encodeURIComponent(field.name) + "=" + encodeURIComponent(optValue));
                                }
                            }
                        }
                        break;
                        
                    case undefined:     //fieldset
                    case "file":        //file input
                    case "submit":      //submit button
                    case "reset":       //reset button
                    case "button":      //custom button
                        break;
                        
                    case "radio":       //radio button
                    case "checkbox":    //checkbox
                        if (!field.checked){
                            break;
                        }
                        /* falls through */
                                    
                    default:
                        //don't include form fields without names
                        if (field.name.length){
                            parts.push(encodeURIComponent(field.name) + "=" + encodeURIComponent(field.value));
                        }
                }
            }        
            return parts.join("&");
        }

```