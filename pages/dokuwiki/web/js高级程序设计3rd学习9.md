title: js高级程序设计3rd学习9 

#  JS高级程序设计3rd学习8高级技巧(重要) 
##  作用域安全的构造函数(最佳实践) 
构造函数其实就是一个使用new操作符调用的函数，当使用new操作符调用时，构造函数内用到的this对象会指向新创建的对象实例。但是如果不使用new操作符，那么this对象将由运行时决定。如下:
```

        function Person(name, age, job){
            this.name = name;
            this.age = age;
            this.job = job;
        }
        //使用new操作符时
        var person1 = new Person("Nicholas", 29, "Software Engineer");
        alert(person1.name);     //"Nicholas"
        alert(person1.age);      //29
        alert(person1.job);      //"Software Engineer"
	
	//不使用new操作符时
        var person2 = Person("Nicholas", 29, "Software Engineer");
        alert(person2);         //undefined
        alert(window.name);     //"Nicholas"
        alert(window.age);      //29
        alert(window.job);      //"Software Engineer"

```
问题出在persion2没有使用new操作符来调用构造函数，这就导致此时Person只是一个普通函数调用，所以this对象是在运行时绑定，此处被绑定到了window对象，污染了全局空间。
解决该问题的方案就是创建一个**作用域安全的构造函数**。
**作用域安全的构造函数**在进行任何更改前，首先确认this对象是正确的类型的实例。如果不是则创建新的实例并返回。
```

       function Person(name, age, job){
            if (this instanceof Person){ //先进行判断
                this.name = name;
                this.age = age;
                this.job = job;
            } else {
                return new Person(name, age, job);
            }
        }
        
        var person1 = Person("Nicholas", 29, "Software Engineer");
        alert(window.name);   //""
        alert(person1.name);  //"Nicholas"
        
        var person2 = new Person("Shelby", 34, "Ergonomist");
        alert(person2.name);  //"Shelby"

```
**最后的结果就是，调用Person构造函数时无论是否使用new操作符，都会返回一个Person的新实例**，这就避免了在全局对象上意外设置属性等问题。确保了this的正确指向。
<WRAP center round alert 60%>
实现这个作用域安全的构造函数模式后，你就锁定了可以调用构造函数的环境。(当用于构造函数窃取模式的继承时会出现问题，解决方案在后面)
</WRAP>
` **当要求构造函数窃取模式的继承时上述代码会出现问题** `。请看下面的分析：
```

        function Polygon(sides){
            if (this instanceof Polygon) {
                this.sides = sides;
                this.getArea = function(){
                    return 0;
                };
            } else {
                return new Polygon(sides);
            }
        }
        
        function Rectangle(width, height){
            Polygon.call(this, 2); 
            this.width = width;
            this.height = height;
            this.getArea = function(){
                return this.width * this.height;
            };
        }
        
        var rect = new Rectangle(5, 10);
        alert(rect.sides);   //undefined

```
Rectangle构造函数中的this对象并没有得到增长，同时Polygon.call()返回的值也没有用到，所以Rectangle实例中就不会有sides属性。
**如果构造函数窃取结合使用原型链或者寄生组合则可以解决这个问题。**如下：
```

        function Polygon(sides){
            if (this instanceof Polygon) {
                this.sides = sides;
                this.getArea = function(){
                    return 0;
                };
            } else {
                return new Polygon(sides);
            }
        }
        
        function Rectangle(width, height){
            Polygon.call(this, 2);
            this.width = width;
            this.height = height;
            this.getArea = function(){
                return this.width * this.height;
            };
        }
        
        Rectangle.prototype = new Polygon();//这样一来就代表Rectangle是Polygon的之类。
        
        var rect = new Rectangle(5, 10);
        alert(rect.sides);   //2

```
上述代码中，**一个Rectangle实例也同时是一个Polygon实例，所以Polygon.call(this, 2);会按照预期执行。**
除非你单纯基于构造函数窃取来实现继承（不提倡，一般采取构造函数窃取与原型链组合模式实现继承），否则` 推荐作用域安全的构造函数作为最佳实践。 `

##  函数绑定bind() 
函数绑定要创建一个函数，可以在特定的**this**环境中以指定参数调用另一个函数。**该技巧通常和回调函数与事件处理程序一起使用，以便在将函数作为变量传递的同时保留代码执行环境。**
先看看没有函数绑定时的例子：
```

       var handler = {
            message: "Event handled",
        
            handleClick: function(event){
                alert(this.message + ":" + event.type);
            }
        };
        
        var btn = document.getElementById("my-btn");
        btn.addEventHandler("click", handler.handleClick);

```
这个this对象最后执行了DOM按钮而非handler。我们可以使用一个闭包来修正它。
```

btn.addEventHandler("click", function(event){
	handler.handleClick(event);
});

```
我们还可以更进一步，定义一个bind函数：
```

        function bind(fn, context){
            return function(){
                return fn.apply(context, arguments);
            };
        }
	btn.addEventHandler("click",bind(handler.handleClick, handler));

```
我们用这个bind函数保持了回调函数中的this执行环境。

` 其实ES5已经为所有函数定义了一个原生的bind()函数(但是IE8及以下不支持) `，所以你不必再像上面那样定义bind()函数了。可以直接这么用:
```

 btn.addHandler("click", handler.handleClick.bind(handler));

```

##  函数柯理化与绑定 
与函数绑定紧密相关的主题是函数柯理化(function currying)
柯理化函数创建步骤：调用另一个函数并为它传入要柯理化的函数和必要参数。
```

        function bind(fn,context){
            var args = Array.prototype.slice.call(arguments, 2);
            return function(){
                var innerArgs = Array.prototype.slice.call(arguments),
                    finalArgs = args.concat(innerArgs);
                return fn.apply(context, finalArgs);
            };
        }
    
        function add(num1, num2){
            return num1 + num2;
        }
        
        var curriedAdd = bind(add,null, 5);
        alert(curriedAdd(3));   //8

        var curriedAdd2 = bind(add,null, 5, 12);
        alert(curriedAdd2());   //17

```
**上面函数的主要工作就是将被返回函数的参数进行排序并保持函数的this执行环境。** 
  * 柯里函数的第一个参数是要进行柯理化的函数引用，其余参数是要传入的值。
  * 为了获取第一个参数之后的所有参数，在arguments对象上调用了slice()方法，并传入参数2表示将被返回的数组包含从第三个参数开始的所有参数。然后args数组包含了来自外部函数(即bind)的参数。
  * 在内部函数中，创建了innerArgs数组用来存放所有传入的参数(又一次用到了slice()).
  * 然后将args与innerArgs组合为finalArgs.
  * 然后调用apply()将结果传递给该函数。
使用示例:
```

        function bind(fn,context){
            var args = Array.prototype.slice.call(arguments, 2);
            return function(){
                var innerArgs = Array.prototype.slice.call(arguments),
                    finalArgs = args.concat(innerArgs);
                return fn.apply(context, finalArgs);
            };
        }
        var handler = {
            message: "Event handled",
        
            handleClick: function(name, event){
                alert(this.message + ":" + name + ":" + event.type);
            }
        };
        
        var btn = document.getElementById("my-btn");
        btn.addHandler("click", bind(handler.handleClick,handler, "my-btn"));

```
上面"my-btn"被当作第一个参数(args中)传入handleClick中，回调中传入event参数(innerArgs中)被当作第二个参数传入handleClick.然后this指向handler对象。
` 其实ES5已经为所有函数定义了一个原生的bind()函数(但是IE8及以下不支持) `，所以你不必再像上面那样定义bind()函数了。可以直接这么用:
```

 btn.addHandler("click", handler.handleClick.bind(handler, "my-btn"));

```

##  防止篡改对象 
为防止JS对象被恶意篡改，一些第三方库一般会采用防止篡改对象技术。
注意：一旦把对象定义为防篡改，就无法撤消。
###  不可扩展对象 
```

        var person = { name: "Nicholas" };
        Object.preventExtensions(person); //关键一行
        
        person.age = 29;
        alert(person.age);//undefined

```
 Object.preventExtensions(person); 之后就不能再给对象添加新的属性和方法了。但是已有成员不受影响，你仍然可以修改和删除已有成员。
使用Object.isExtensible(person)可以用来判断对象是否可以扩展。

###  密封对象 
ES5为对象定义的第二个保护级别是密封对象(sealed object).密封对象不可扩展，而且已有成员的[ [Configurable] ]特性被设置为false。这就意味着不能删除属性和方法。当时已有的属性值仍然可以修改。
```

        var person = { name: "Nicholas" };
        Object.seal(person);
        
        person.age = 29;
        alert(person.age);      //undefined 不能扩展
        
        delete person.name;
        alert(person.name);     //"Nicholas" 不能被删除

```
密封对象=不可扩展对象(不能添加)+不能删除属性和方法
但是可以修改已有属性值。
可以通过Object.isSealed(person)来进行判断。

###  冻结对象(frozen object) 
最严格的防篡改级别是冻结对象。冻结对象不可扩展，是密封的，而且对象已有属性不能修改。
```

        var person = { name: "Nicholas" };
        Object.freeze(person);
        
        person.age = 29;
        alert(person.age);      //undefined
        
        delete person.name;
        alert(person.name);     //"Nicholas"
        
        person.name = "Greg";
        alert(person.name);     //"Nicholas"

```
##  高级定时器Web Workers 
**web worker 是运行在后台的 JavaScript，不会影响页面的性能。**
什么是 Web Worker？
当在 HTML 页面中执行脚本时，页面的状态是不可响应的，直到脚本已完成。
web worker 是运行在后台的 JavaScript，独立于其他脚本，不会影响页面的性能。您可以继续做任何愿意做的事情：点击、选取内容等等，而此时 web worker 在后台运行。
浏览器支持
所有主流浏览器均支持 web worker，除了 Internet Explorer。
###  检测 Web Worker 支持 
在创建 web worker 之前，请检测用户的浏览器是否支持它：
```

if(typeof(Worker)!=="undefined")
  {
  // Yes! Web worker support!
  // Some code.....
  }
else
  {
  // Sorry! No Web Worker support..
  }

```
###  创建 web worker 文件 
现在，让我们在一个**外部** JavaScript 中创建我们的 web worker。
在这里，我们创建了计数脚本。该脚本存储于 "demo_workers.js" 文件中：
```

var i=0;
function timedCount()
{
i=i+1;
postMessage(i);
setTimeout("timedCount()",500);
}
timedCount();

```
以上代码中` 重要的部分是 postMessage() 方法 ` - 它用于向 HTML 页面传回一段消息。postMessage()参数可以是能够序列化为JSON结构的任意JS对象。
注释：web worker 通常不用于如此简单的脚本，而是用于更耗费 CPU 资源的任务。

###  创建 Web Worker 对象 
我们已经有了 web worker 文件，现在我们需要从 HTML 页面调用它。
下面的代码检测是否存在 worker，如果不存在，- 它会创建一个新的 web worker 对象，然后运行 "demo_workers.js" 中的代码：
```

if(typeof(w)=="undefined")
  {
  w=new Worker("demo_workers.js");
  }

```
然后我们就可以从 web worker 发生和接收消息了。
**向 web worker 添加一个 "onmessage" 事件监听器：**
```

w.onmessage=function(event){
document.getElementById("result").innerHTML=event.data;
};

```
当 web worker 传递消息时，会执行事件监听器中的代码。event.data 中存有来自 postMessage( data) 的数据。

可以通过Worker实例的terminate()方法停止Worker工作。


###  终止 Web Worker 
当我们创建 web worker 对象后，它会继续监听消息（即使在外部脚本完成之后）直到其被终止为止。
如需终止 web worker，并释放浏览器/计算机资源，请使用 terminate() 方法：
` w.terminate(); `

###  Web Worker的限制 
**由于 web worker 位于外部文件中，它们无法访问下例 JavaScript 对象：**
  * window 对象
  * document 对象
  * parent 对象

###   Web Worker的全局作用域 
Web Worker中的代码所执行代码所在的全局作用域与页面的全局作用域不是同一个。前者为Worker,而后者为window。
在这个特殊的全局作用域中，this和self引用的都是worker对象。worker对象本身也是一个最小化的运行环境,还包括以下属性:
  * 最小化的navigator对象，包括onLine、appName、appVersion、userAgent、platform属性
  * 只读的location对象
  * setTimeout()、setInterval()等
  * XMLHttpRequest构造函数

页面中：
```

        (function(){
        
            var data = [23,4,7,9,2,14,6,651,87,41,7798,24],
                worker = new Worker("WebWorkerExample01.js");
                                
            worker.onmessage = function(event){
                alert(event.data);
            };
            
            worker.postMessage(data);            
        
        })();
        

```
worker中：
```

self.onmessage = function(event){
    var data = event.data;
    data.sort(function(a, b){
        return a - b;
    });
    
    self.postMessage(data);
};

```
###  worker中包含其他脚本 
正常情况下worker是无法操作页面中的任何东西包括DOM。但是我们可以调用importScripts()来加载外部js文件。这些加载都是异步的但是保证顺序执行。
```

//webwork内部代码
importScript("file1.js","file2.js");

```
注意：worker中的脚本一般都具有特殊的用途，也受到很多限制(如不能操作DOM等)，不会像页面中的脚本那么功能宽泛。
目前Web Workers规范还不是很完善。浏览器支持程度也不一致。

###  完整的 Web Worker 实例代码 
我们已经看到了 .js 文件中的 Worker 代码。下面是 HTML 页面的代码：
实例
```

<!DOCTYPE html>
<html>
<body>

<p>Count numbers: <output id="result"></output></p>
<button onclick="startWorker()">Start Worker</button>
<button onclick="stopWorker()">Stop Worker</button>
<br /><br />

<script>
var w;

function startWorker()
{
if(typeof(Worker)!=="undefined")
{
  if(typeof(w)=="undefined")
    {
    w=new Worker("demo_workers.js");
    }
  w.onmessage = function (event) {
    document.getElementById("result").innerHTML=event.data;
  };
}
else
{
document.getElementById("result").innerHTML="Sorry, your browser
 does not support Web Workers...";
}
}

function stopWorker()
{
w.terminate();
}
</script>

</body>
</html>

```
##  拖放 
拖放（Drag 和 drop）是 HTML5 标准的组成部分。
拖放是一种常见的特性，即抓取对象以后拖到另一个位置。在 HTML5 中，拖放是标准的一部分，任何元素都能够拖放。
浏览器支持
Internet Explorer 9、Firefox、Opera 12、Chrome 以及 Safari 5 支持拖放。
参考:http://www.w3school.com.cn/html5/html_5_draganddrop.asp
HTML5 拖放实例
下面的例子是一个简单的拖放实例：
```

<!DOCTYPE HTML>
<html>
<head>
<style type="text/css">
#div1 {width:488px;height:70px;padding:10px;border:1px solid #aaaaaa;}
</style>
<script type="text/javascript">
function allowDrop(ev)
{
ev.preventDefault();
}

function drag(ev)
{
ev.dataTransfer.setData("Text",ev.target.id);
}

function drop(ev)
{
ev.preventDefault();
var data=ev.dataTransfer.getData("Text");
ev.target.appendChild(document.getElementById(data));
}
</script>
</head>
<body>

<p>请把 W3School 的图片拖放到矩形中：</p>

<div id="div1" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
<br />
<img id="drag1" src="/i/w3school_banner.gif" draggable="true" ondragstart="drag(event)" />

</body>
</html>

```
###  设置元素为可拖放 
首先，为了使元素可拖动，把 draggable 属性设置为 true ：
```

< img draggable="true" / >

```
拖动什么 - ondragstart 和 setData()
然后，规定当元素被拖动时，会发生什么。
在上面的例子中，ondragstart 属性调用了一个函数，drag(event)，它规定了被拖动的数据。
**dataTransfer.setData() 方法设置被拖数据的数据类型和值：**
```

function drag(ev)
{
ev.dataTransfer.setData("Text",ev.target.id);
}

```
在这个例子中，数据类型是 "Text"，值是可拖动元素的 id ("drag1")。
###  放到何处ondragover 
ondragover 事件规定在何处放置被拖动的数据。
**默认地，无法将数据/元素放置到其他元素中。如果需要设置允许放置，我们必须阻止对元素的默认处理方式。**
这要通过调用 ondragover 事件的 event.preventDefault() 方法：
event.preventDefault()
###  进行放置 - ondrop 
当放置被拖数据时，会发生 drop 事件。
在上面的例子中，ondrop 属性调用了一个函数，drop(event)：
```

function drop(ev)
{
ev.preventDefault();
var data=ev.dataTransfer.getData("Text");
ev.target.appendChild(document.getElementById(data));
}

```
代码解释：
调用 preventDefault() 来避免浏览器对数据的默认处理（drop 事件的默认行为是以链接形式打开）
通过 dataTransfer.getData("Text") 方法获得被拖的数据。该方法将返回在 setData() 方法中设置为相同类型的任何数据。
被拖数据是被拖元素的 id ("drag1")
把被拖元素追加到放置元素（目标元素）中

###  在两个div之间拖动图像 
```

<!DOCTYPE HTML>
<html>
<head>
<style type="text/css">
#div1, #div2
{float:left; width:100px; height:35px; margin:10px;padding:10px;border:1px solid #aaaaaa;}
</style>
<script type="text/javascript">
function allowDrop(ev)
{
ev.preventDefault();
}

function drag(ev)
{
ev.dataTransfer.setData("Text",ev.target.id);
}

function drop(ev)
{
ev.preventDefault();
var data=ev.dataTransfer.getData("Text");
ev.target.appendChild(document.getElementById(data));
}
</script>
</head>
<body>

<div id="div1" ondrop="drop(event)" ondragover="allowDrop(event)">
  <img src="/i/w3school_logo_black.gif" draggable="true" ondragstart="drag(event)" id="drag1" />
</div>
<div id="div2" ondrop="drop(event)" ondragover="allowDrop(event)"></div>

</body>
</html>

```