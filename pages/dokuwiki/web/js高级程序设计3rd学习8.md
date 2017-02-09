modifyAt:2016-12-11 13:41:57
location:dokuwiki/web/js高级程序设计3rd学习8
title: js高级程序设计3rd学习8 
author:xbynet
createAt:2016-12-11 13:41:29

#  JS高级程序设计3rd学习8JSON与Ajax 
##  JSON 
JSON是JS的一个严格的子集。形式类似于JS中的对象字面量或数组字面量，但是比它们拥有更多的限制。
JSON的语法可以表示以下三种类型的值：
  * 简单值:字符串、数值、布尔值、null。但是JSON不支持undefined
  * 对象:表示一组无序的键值对。
  * 数组

JSON中的对象要求**给属性名**加上**双引**号(不能是单引号，也不能不加)
```

{
	"name":"xby",
	"age":29
}

```
###  JSON对象解析与序列化 
JSON.stringify()方法用于把JS对象序列化为字符串
JSON.parse()用于从字符串中解析出JS对象
序列化也可以提供一些选项。
```

        var book = {
                        title: "Professional JavaScript",
                        authors: [
                            "Nicholas C. Zakas"
                        ],
                        edition: 3,
                        year: 2011
                   };

        var jsonText = JSON.stringify(book, ["title", "edition"],4); //只有第一个参数是必须的，其余可选。第二个参数是个过滤器，可以是数组，也可以是一个函数function(key,value)返回对应key的值。第三个参数用于控制缩进
        alert(jsonText);

```
JSON.stringify第二个参数是函数的情形:
```

FontPicker({callBack:function(style){
				$("#globalTitleFont").data("value",JSON.stringify(style,function(k,v){
                                  //必须要进行判断k存在，因为第一次传入的k为"",而v为整个对象即style。以后传入的才是里面的每个键
					if(!v&&k){ 
						if(k=="font-size"){
							v="12px";
						}else if(k=="color"){
							v="black";
						}else{
							v="normal";
						}
					}
					return v //由于第一次传入的k为"",而v为整个对象，所以记得返回。否则就不会有下文了。
					
				}));
			}});

```
##  Ajax与跨域请求 
Ajax技术的核心是XMLHttpRequest(简称XHR)
XHR的用法.
```

var xhr=new XMLHttpRequest();
xhr.open("get","example.php",true);第三个参数代表是否异步发送
xhr.send(null);//接受一个参数，参数为http请求的参数

```
xhr的相关属性:
responseText只要返回有值，那么它始终有值
responseXML不一定有值，需要服务器返回的MIME类型是XML的。
status：响应的HTTP状态，如200
statusText
readystate属性:0,1,2,3,4(4代表完成)
onreadystatechange回调监听函数
```

        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function(event){
            if (xhr.readyState == 4){
                if ((xhr.status ==200) || xhr.status == 304){
                    alert(xhr.responseText);
                } else {
                    alert("Request was unsuccessful: " + xhr.status);
                }
            }
        };
        xhr.open("get", "example.txt", true);
        xhr.send(null);

```
请求终止方法abort()

GET请求时需要对URL查询参数进行encodeURIComponent
```

function addURLParam(url,name,value){
url+=(url.indexOf("?")==-1?"?":"&");
url+=encodeURIComponent(name)+"="+encodeURIComponent(value);
return url;
}

```
POST请求时需要设置头部
```

           xhr.open("post", "postexample.php", true);
            xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");//设置头部
            var form = document.getElementById("user-info");            
            xhr.send(serialize(form));

```
不过建议尽量用GET请求，因为传输的数据流少一点

###  XMLHttpRequest2 
目前所有浏览器支持还不是很完善。
FormData。用于表单数据的序列化。甚至支持ajax文件上传。
```

var data=new FormData(document.forms[0]);
data.append("name","xny");
xhr.send(data);

```
###  进度事件 
目前支持还不是很完善：
loadstart
progress
error
abort
load
loadend
目前所有浏览器都支持load，但是其他的不一定支持。
```

       xhr.onload = function(event){
            if ((xhr.status >= 200 && xhr.status < 300) || 
                    xhr.status == 304){
                alert(xhr.responseText);
            } else {
                alert("Request was unsuccessful: " + xhr.status);
            }
        };

```

##  CORS跨域资源共享 
http://www.cnblogs.com/vajoy/p/4295825.html
http://www.ruanyifeng.com/blog/2016/04/cors.html?utm_source=tuicool&utm_medium=referral
http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html
浏览器的**同源策略/SOP**（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。
SOP要求两个通讯地址的**协议、域名、端口号**必须相同，否则两个地址的通讯将被浏览器视为不安全的，并被block下来。比如“http页面”和“https页面”属于不同协议；“qq.com”、“www.qq.com”、“a.qq.com”都属于不同域名（或主机）；“a.com”和“a.com:8000”属于不同端口号。这三种情况常规都是无法直接进行通讯的。

通过XHR实现Ajax通信的一个主要限制就是跨域安全策略。默认情况下XHR对象只能访问同一域名中的资源。
**CORS**（Cross-Origin Resource Sharing,跨源资源共享）是W3C的一个草案。定义了必须访问跨域资源时，浏览器与服务器如何进行沟通。
**基本思想：使用自定义HTTP头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功还是失败。**

例如，对于简单的GET|POST操作，
  * 请求必须加一个Origin头部: Origin:http://www.xby1993.net （` 该步骤由浏览器自动添加，对开发者是透明的 `。IE10及其它浏览器XHR原生支持，IE8、9需要使用XDomainRequest）
  * 服务器响应必须加一个头部Access-Control-Allow-Origin头部:Access-Control-Allow-Origin:http://www.xby1993.net

以上工作是基础。也是必须要做的事情。接下来进行方案选择。下面的方案都是以上面为基础而言的。
注意：请求和响应都不包含cookie信息。(如需包含cookie信息，请看后面)

**CORS需要浏览器和服务器同时支持**。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。(IE8、9需要使用XDomainRequest)
**整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息**，有时还会多出一次附加的请求，但用户不会有感觉。
` 因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口(即在响应中添加相应的Access-Control-xxx头部)，就可以跨源通信。 `

###  两种请求 
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。
1、简单请求：只要同时满足以下两大条件，就属于简单请求。
  * 请求方法是以下三种方法之一：HEAD、GET、POST
  * HTTP的头信息不超出以下几种字段：
Accept、Accept-Language、Content-Language、Last-Event-ID、Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
2、凡是不同时满足上面两个条件，就属于非简单请求。
浏览器对这两种请求的处理，是不一样的。

**简单请求**
3.1 基本流程
对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。**(浏览器自动添加的，对开发者透明)**
下面是一个例子，浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个Origin字段。
```

GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

```
上面的头信息中，` Origin字段 `用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。**服务器根据这个值，决定是否同意这次请求。**
如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被**XMLHttpRequest的onerror回调函数捕获。**注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。
如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。
```

Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true 可选，如果输出了该头，说明服务器允许CORS带cookie传输。
Access-Control-Expose-Headers: FooBar 可选，如果输出了该头，说明服务器允许CORS传输自定义的头部。
Content-Type: text/html; charset=utf-8

```
上面的头信息之中，有三个与CORS请求相关的字段，都以Access-Control-开头。
（1）Access-Control-Allow-Origin
该字段是必须的。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求。
（2）Access-Control-Allow-Credentials
该字段可选。它的值是一个布尔值，**表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。**设为true时，Cookie就会包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。
（3）Access-Control-Expose-Headers
该字段可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。**如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定**。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值。

3.2 **withCredentials 属性**
上面说到，**CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段。**
Access-Control-Allow-Credentials: true
**另一方面，开发者必须在AJAX请求中打开withCredentials属性。**
```

var xhr = new XMLHttpRequest();
xhr.withCredentials = true;

```
**否则，即使服务器同意发送Cookie，浏览器也不会发送。**
需要注意的是，如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。**同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。**

对于非简单请求，请参考：http://www.ruanyifeng.com/blog/2016/04/cors.html?utm_source=tuicool&utm_medium=referral

###  IE8、9对CORS的实现 
` IE8、9对CORS的实现是XDR，而IE10以后废弃了XDR，与主流浏览器一样支持标准的XHR CORS `
XDR（XDomainRequest）
注意：所有的XDR请求都是异步执行的，不能用它来创建同步请求。请求返回之后会触发load事件，响应的数据也会保存在responseText中。
```

        var xdr = new XDomainRequest();
        xdr.onload = function(){
            alert(xdr.responseText);
        };
        xdr.onerror = function(){
            alert("Error!");
        };
        
        //you'll need to replace this URL with something that works
        xdr.open("get", "http://www.somewhere-else.com/xdr.php");
        xdr.send(null);    

```
###  其余浏览器对CORS的实现 
其余浏览器都通过XMLHttpRequest对象实现了**对CORS的原生支持**(包括IE10及以后)。要请求另一个域中的资源，使用标准的XHR对象并在open()方法中传入**绝对URL**即可。
 xhr.open("get", "http://www.somewhere-else.com/xdr.php",true);
跨域带来的限制:不能设置头部，不能发送和接受cookie
###  跨浏览器的CORS 
```

        function createCORSRequest(method, url){
            var xhr = new XMLHttpRequest();
            if ("withCredentials" in xhr){
                xhr.open(method, url, true);
            } else if (typeof XDomainRequest != "undefined"){
                xhr = new XDomainRequest();
                xhr.open(method, url);
            } else {
                xhr = null;
            }
            return xhr;
        }

        var request = createCORSRequest("get", "http://www.somewhere-else.com/xdr.php");
        if (request){
            request.onload = function(){
                //do something with request.responseText
            };
            request.send(null);
        }

```
##  其余跨域技术 
在CORS出现以前，要实现跨域Ajax通信需要费一些周折。利用DOM中能够执行跨域请求的能力，在不依赖于XHR对象的情况下也能发送某种请求。
###  图像Ping 
图像Ping跨域请求技术使用的是<img >标签。
动态创建图像常用于图像Ping。
特定：单向，简单通信。
确定：无法处理响应
```

        img.onload = img.onerror = function(){
            alert("Done!");
        };
        img.src = "http://www.example.com/test?name=Nicholas"; 

```
###  JSONP 
JSONP(JSON with padding)由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数。回调函数的名字一般在请求中指定的。而数据就是传入回调函数的JSON数据。
下面是一个典型的JSONP请求：
http://sss.net/json/?callback=handleResponse
指定了回调函数，名为handleResponse
1. 服务端JSONP格式数据
  * 如客户想访问 : http://www.runoob.com/try/ajax/jsonp.php?callback=callbackFunction。
  * 假设客户期望返回JSON数据：["customername1","customername2"]。
  * 真正返回到客户端的数据显示为: callbackFunction(["customername1","customername2"])。
例如服务端文件jsonp.php代码为：

``` php

<?php
header('Content-type: application/json');
//获取回调函数名
$jsoncallback = htmlspecialchars($_REQUEST ['jsoncallback']);
//json数据
$json_data = '["customername1","customername2"]';
//输出jsonp格式的数据
echo $jsoncallback . "(" . $json_data . ")";
?>

```
  
JSONP是通过动态< script >元素来使用的。
```

    
        function handleResponse(response){
            alert("You're at IP address " + response.ip + ", which is in " + response.city + ", " + response.region_name);
        }
        var script = document.createElement("script");
        script.src = "http://freegeoip.net/json/?callback=handleResponse";
        document.body.insertBefore(script, document.body.firstChild);

```
优点：可以直接访问相应数据。
缺点：由于返回的数据是JS代码并且会被立即执行的，所以需要保证其他域绝对的安全，否则可能会在响应中夹带一些恶意代码。

###  websocket 
后面介绍。此处略。

###  HTML跨文档通信 
http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html
#### = window.name 
浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。
父窗口先打开一个子窗口，载入一个不同源的网页，该网页将信息写入window.name属性。
window.name = data;
接着，子窗口跳回一个与主窗口同域的网址。
location = 'http://parent.url.com/xxx.html';
然后，主窗口就可以读取子窗口的window.name了。
var data = document.getElementById('myFrame').contentWindow.name;
这种方法的优点是，window.name容量很大，可以放置非常长的字符串；缺点是必须监听子窗口window.name属性的变化，影响网页性能。
####  跨文档通信 API-window.postMessage 
HTML5为了解决这个问题，引入了一个全新的API：**跨文档通信 API（Cross-document messaging）。**
这个API为window对象新增了一个` window.postMessage `方法，**允许跨窗口通信，不论这两个窗口是否同源。**
举例来说，父窗口http://aaa.com向子窗口http://bbb.com发消息，调用postMessage方法就可以了。
```

var popup = window.open('http://aaa.com', 'title');
popup.postMessage('Hello World!', 'http://aaa.com');

```
**postMessage方法的第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（origin）**，即"协议 + 域名 + 端口"。也可以设为*，表示不限制域名，向所有窗口发送。
子窗口向父窗口发送消息的写法类似。
```

window.opener.postMessage('Nice to see you', 'http://bbb.com');

```
**父窗口和子窗口都可以通过message事件，监听对方的消息。**
```

window.addEventListener('message', function(e) {
  console.log(e.data);
},false);

```
message事件的事件对象event，提供以下三个属性。
  * event.source：发送消息的窗口
  * event.origin: 消息发向的网址
  * event.data: 消息内容
下面的例子是，**子窗口通过event.source属性引用父窗口，然后发送消息。**
```

window.addEventListener('message', receiveMessage);
function receiveMessage(event) {
  event.source.postMessage('Nice to see you!', '*');
}

```
event.origin属性可以过滤不是发给本窗口的消息。
```

window.addEventListener('message', receiveMessage);
function receiveMessage(event) {
  if (event.origin !== 'http://bbb.com') return;
  if (event.data === 'Hello World') {
      event.source.postMessage('Hello', event.origin);
  } else {
    console.log(event.data);
  }
}

```
####  LocalStorage 
通过window.postMessage，读写其他窗口的 LocalStorage 也成为了可能。
下面是一个例子，主窗口写入iframe子窗口的localStorage。
```

window.onmessage = function(e) {
  if (e.origin !== 'http://bbb.com') {
    return;
  }
  var payload = JSON.parse(e.data);
  localStorage.setItem(payload.key, JSON.stringify(payload.data));
};

```
上面代码中，子窗口将父窗口发来的消息，写入自己的LocalStorage。

##  Comet 
略

##  SSE服务器发送事件 
[HTML5学习之SSE(服务器推送事件)](/pages/dokuwiki/web/HTML5学习2)
##  WebSockets 
略
##  浏览器同源政策与跨域说明 
http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html