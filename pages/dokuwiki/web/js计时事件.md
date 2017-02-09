title: js计时事件 

#  JS计时事件setInterval() 和 setTimeout() 
通过使用 JavaScript，我们有能力作到在一个设定的时间间隔之后来执行代码，而不是在函数被调用后立即执行。我们称之为计时事件。
在 JavaScritp 中使用计时事件是很容易的，两个关键方法是:
  * **setInterval()** - 间隔指定的毫秒数不停地执行指定的代码。
  * **setTimeout()** - 暂停指定的毫秒数后执行指定的代码
Note: setInterval() 和 setTimeout() 是 HTML DOM Window对象的两个方法。
##  setInterval() 方法 
setInterval() 间隔指定的毫秒数**不停地执行指定的代码**
` window.setInterval("javascript function",milliseconds); `
window.setInterval() 方法可以不使用window前缀，直接使用函数setInterval()。
setInterval() 第一个参数是函数（function）。
第二个参数**间隔的毫秒数**
注意: 1000 毫秒是一秒。
每三秒弹出 "hello" ：
setInterval(function(){alert("Hello")},3000);

**如何停止执行?**
` clearInterval() ` 方法用于停止 setInterval() 方法执行的函数代码。
window.clearInterval(intervalVariable)
window.clearInterval() 方法可以不使用window前缀，直接使用函数clearInterval()。
**要使用 clearInterval() 方法, 在创建计时方法时你必须使用全局变量：**
myVar=setInterval("javascript function",milliseconds);
然后你可以使用clearInterval() 方法来停止执行。
```

<p id="demo"></p>
<button onclick="myStopFunction()">Stop time</button>

<script>
var myVar=setInterval(function(){myTimer()},1000);
function myTimer()
{
var d=new Date();
var t=d.toLocaleTimeString();
document.getElementById("demo").innerHTML=t;
}
function myStopFunction()
{
clearInterval(myVar);
}
</script>

```
##  setTimeout() 方法 
` window.setTimeout("javascript 函数",毫秒数); `
setTimeout() 方法会返回某个值。在上面的语句中，值被储存在名为 t 的变量中。假如你希望取消这个 setTimeout()，你可以使用这个变量名来指定它。
setTimeout() 的第一个参数是含有 JavaScript 语句的字符串。这个语句可能诸如 "alert('5 seconds!')"，或者对函数的调用，诸如 alertMsg()"。
第二个参数指示从当前起多少毫秒后执行第一个参数。
提示：1000 毫秒等于一秒。
等待3秒，然后弹出 "Hello":
```

setTimeout(function(){alert("Hello")},3000);

```
**如何停止执行?**
` clearTimeout() ` 方法用于停止执行setTimeout()方法的函数代码。
语法
window.clearTimeout(timeoutVariable)
window.clearTimeout() 方法可以不使用window 前缀。
要使用clearTimeout() 方法, 你必须在创建超时方法中（setTimeout）使用全局变量:
myVar=setTimeout("javascript function",milliseconds);
如果函数还未被执行，你可以使用 clearTimeout() 方法来停止执行函数代码。

