title: chromeapp1 

#  Chrome扩展与应用开发快速入门 
学习网址:http://www.ituring.com.cn/minibook/950
Github Demo代码：https://github.com/Sneezry/chrome_extensions_and_apps_programming

项目结构:必须文件manifest.json
第一个项目参考:http://www.ituring.com.cn/article/60134
##  Manifest文件格式 
**Chrome扩展的Manifest必须包含name、version和manifest_version属性，目前来说manifest_version属性值只能为数字2，对于应用来说，还必须包含app属性。**

其他常用的可选属性还有browser_action、page_action、background、permissions、options_page、content_scripts，所以我们可以保留一份manifest.json模板，当编写新的扩展时直接填入相应的属性值。如果我们需要的属性不在这个模板中，可以再去查阅官方文档，但我想这样的一份模板可以应对大部分的扩展了.
```

{
    "app": {
        "background": {
            "scripts": ["background.js"]
        }
    },
    "manifest_version": 2,
    "name": "My Extension",
    "version": "versionString",
    "default_locale": "en",
    "description": "A plain text description",
    "icons": {
        "16": "images/icon16.png",
        "48": "images/icon48.png",
        "128": "images/icon128.png"
    },
    "browser_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        },
        "default_title": "Extension Title",
        "default_popup": "popup.html"
    },
    "page_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        },
        "default_title": "Extension Title",
        "default_popup": "popup.html"
    },
    "background": {
        "scripts": ["background.js"]
    },
    "content_scripts": [
        {
            "matches": ["http://www.google.com/*"],
            "css": ["mystyles.css"],
            "js": ["jquery.js", "myscript.js"]
        }
    ],
    "options_page": "options.html",
    "permissions": [
        "*://www.google.com/*"
    ],
    "web_accessible_resources": [
        "images/*.png"
    ]
}

```
在官方文档中可以找到完整的Manifest属性列表，扩展在https://developer.chrome.com/extensions/manifest，应用在https://developer.chrome.com/apps/manifest。由于Google更新得非常频繁，上述页面内容可能会经常变动，但那些比较基本的属性变动的几率不会很大。

##  Chrome扩展基础 
通过Manifest中的` content_scripts `属性可以指定将哪些脚本何时注入到哪些页面中，当用户访问这些页面后，相应脚本即可自动运行，从而对页面DOM进行操作。
Manifest的content_scripts属性值为数组类型，数组的每个元素可以包含matches、exclude_matches、css、js、run_at、all_frames、include_globs和exclude_globs等属性。其中matches属性定义了哪些页面会被注入脚本，exclude_matches则定义了哪些页面不会被注入脚本，css和js对应要注入的样式表和JavaScript，run_at定义了何时进行注入，all_frames定义脚本是否会注入到嵌入式框架中，include_globs和exclude_globs则是全局URL匹配，最终脚本是否会被注入由matches、exclude_matches、include_globs和exclude_globs的值共同决定。简单的说，如果URL匹配mathces值的同时也匹配include_globs的值，会被注入；如果URL匹配exclude_matches的值或者匹配exclude_globs的值，则不会被注入。

下面我们来写一个恶作剧的小扩展，名字就叫做永远点不到的搜索按钮吧 :)
首先创建Manifest文件，内容如下：
```

{
    "manifest_version": 2,
    "name": "永远点不到的搜索按钮",
    "version": "1.0",
    "description": "让你永远也点击不到Google的搜索按钮",
    "content_scripts": [
        {
            "matches": ["*://www.google.com/"],
            "js": ["js/cannot_touch.js"]
        }
    ]
}

```
在content_scripts属性中我们定义了一个匹配规则，当URL符合*:/ /www.google.com/规则的时候，就将js/cannot_touch.js注入到页面中。其中*代表任意字符，这样当用户访问http://www.google.com/和https:/ /www.google.com/时就会触发脚本。

----

**跨域**指的是JavaScript通过XMLHttpRequest请求数据时，调用JavaScript的页面所在的域和被请求页面的域不一致。对于网站来说，浏览器出于安全考虑是不允许跨域。另外，对于域相同，但端口或协议不同时，浏览器也是禁止的。
但这个规则如果同样限制Chrome扩展应用，就会使其能力大打折扣，所以**Google允许Chrome扩展应用不必受限于跨域限制。但出于安全考虑，需要在Manifest的permissions属性中声明需要跨域的权限。**
比如，如果我们想设计一款获取维基百科数据并显示在其他网页中的扩展，就要在Manifest中进行如下声明：
```

{
    ...
    "permissions": [
        "*://*.wikipedia.org/*"
    ]
}

```

**常驻后台**
有时我们希望扩展不仅在用户主动发起时（如开启特定页面或点击扩展图标等）才运行，而是希望扩展自动运行并常驻后台来实现一些特定的功能，比如实时提示未读邮件数量、后台播放音乐等等。
在Manifest中指定background域可以使扩展常驻后台。background可以包含三种属性，分别是**scripts、page和persistent**。如果指定了scripts属性，则Chrome会在扩展启动时自动创建一个包含所有指定脚本的页面；如果指定了page属性，则Chrome会将指定的HTML文件作为后台页面运行。通常我们只需要使用scripts属性即可，除非在后台页面中需要构建特殊的HTML——但一般情况下后台页面的HTML我们是看不到的。persistent属性定义了常驻后台的方式——当其值为true时，表示扩展将一直在后台运行，无论其是否正在工作；当其值为false时，表示扩展在后台按需运行，这就是Chrome后来提出的Event Page。Event Page可以有效减小扩展对内存的消耗，如非必要，请将persistent设置为false。persistent的默认值为true。

下面我们来编写一款实时监视网站在线状态的扩展。思路很简单，每隔5秒就发起一次连接请求，如果请求成功就代表网站在线，将扩展图标显示为绿色，如果请求失败就代表网站不在线，将扩展图标显示为红色。
下面是这个扩展的Manifest文件，此例中以检测www.google.cn为例，你可以根据自己的意愿更改为其他的网站。

```

{
    "manifest_version": 2,
    "name": "Google在线状态",
    "version": "1.0",
    "description": "监视Google是否在线",
    "icons": {
        "16": "images/icon16.png",
        "48": "images/icon48.png",
        "128": "images/icon128.png"
    },
    "browser_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        }
    },
    "background": {
        "scripts": [
            "js/status.js"
        ]
    },
    "permissions": [
        "http://www.google.cn/"
    ]
}

```
用到的chrome接口：chrome.browserAction.setIcon


----
**带选项页面的扩展**
有一些扩展允许用户进行个性化设置，这样就需要向用户提供一个选项页面。Chrome通过Manifest文件的options_page属性为开发者提供了这样的接口，可以为扩展指定一个选项页面。当用户在扩展图标上点击右键，选择菜单中的“选项”后，就会打开这个页面1。

1 对于没有图标的扩展，可以在chrome:/ /extensions页面中单击“选项”。
指定options_page属性后，扩展图标上的右键菜单会包含“选项”链接

对于网站来说，用户的设置通常保存在Cookies中，或者保存在网站服务器的数据库中。对于JavaScript来说，一些数据可以保存在变量中，但如果用户重新启动浏览器，这些数据就会消失。
那么如何在扩展中保存用户的设置呢？我们可以使用HTML5新增的**localStorage**接口。除了localStorage接口以外，还可以使用其他的储存方法。后面将专门拿出一节来讲解数据存储，本节中我们先使用最简单的localStorage方法储存数据。

localStorage是HTML5新增的方法，它允许JavaScript在用户计算机硬盘上永久储存数据（除非用户主动删除）。但localStorage也有一些限制，首先是localStorage和Cookies类似，都有域的限制，运行在不同域的JavaScript无法调用其他域localStorage的数据；其次是单个域在localStorage中存储数据的大小通常有限制（虽然W3C没有给出限制），对于Chrome这个限制是5MB2；最后localStorage只能储存字符串型的数据，无法保存数组和对象，但可以通过join、toString和JSON.stringify等方法先转换成字符串再储存。

2 通过声明` unlimitedStorage `权限，Chrome扩展和应用可以突破这一限制。

下面我们将编写一个天气预报的扩展，这个扩展将提供一个选项页面供用户填写所关注的城市。
有很多网站提供天气预报的API，比如OpenWeatherMap的API。可以通过http://openweathermap.org/API了解更多相关内容。
```

{
    "manifest_version": 2,
    "name": "天气预报",
    "version": "1.0",
    "description": "查看未来两周的天气情况",
    "icons": {
        "16": "images/icon16.png",
        "48": "images/icon48.png",
        "128": "images/icon128.png"
    },
    "browser_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        },
        "default_title": "天气预报",
        "default_popup": "popup.html"
    },
    "options_page": "options.html",
    "permissions": [
        "http://api.openweathermap.org/data/2.5/forecast?q=*"
    ]
}

```
上面是这个扩展的Manifest文件，options.html为设定选项的页面。下面开始编写options.html文件。
```

<html>
    <head>
        <title>设定城市</title>
    </head>
    <body>
        <input type="text" id="city" />
        <input type="button" id="save" value="保存" />
        <script src="js/options.js"></script>
    </body>
</html>

```
这个页面提供了一个id为city的文本框和一个id为save的按钮。由于Chrome不允许将JavaScript内嵌在HTML文件中，所以我们单独编写一个options.js脚本文件，并在HTML文件中引用它。下面来编写options.js文件。
```

var city = localStorage.city || 'beijing';
document.getElementById('city').value = city;
document.getElementById('save').onclick = function(){
    localStorage.city = document.getElementById('city').value;
    alert('保存成功。');
}

```

----

###  扩展页面间的通信 
有时需要让扩展中的多个页面之间，或者不同扩展的多个页面之间相互传输数据，以获得彼此的状态。比如音乐播放器扩展，当用户鼠标点击popup页面中的音乐列表时，popup页面应该将用户这个指令告知后台页面，之后后台页面开始播放相应的音乐。
Chrome提供了4个有关扩展页面间相互通信的接口，分别是**runtime.sendMessage、runtime.onMessage、runtime.connect和runtime.onConnect**。做为一部入门级教程，此节将只讲解runtime.sendMessage和runtime.onMessage接口，runtime.connect和runtime.onConnect做为更高级的接口请读者依据自己的兴趣自行学习，你可以在http://developer.chrome.com/extensions/extension得到有关这两个接口的完整官方文档。
` 请注意，Chrome提供的大部分API是不支持在content_scripts中运行的，但runtime.sendMessage和runtime.onMessage可以在content_scripts中运行，所以扩展的其他页面也可以同content_scripts相互通信。 `
runtime.sendMessage完整的方法为：
```

chrome.runtime.sendMessage(extensionId, message, options, callback)

```
其中extensionId为所发送消息的目标扩展，如果不指定这个值，则默认为发起此消息的扩展本身；message为要发送的内容，类型随意，内容随意，比如可以是'Hello'，也可以是{action: 'play'}、2013和['Jim', 'Tom', 'Kate']等等；options为对象类型，包含一个值为布尔型的includeTlsChannelId属性，此属性的值决定扩展发起此消息时是否要将TLS通道ID发送给监听此消息的外部扩展1，有关TLS的相关内容可以参考http://www.google.com/intl/zh-CN/chrome/browser/privacy/whitepaper.html#tls，这是有关加强用户连接安全性的技术，如果这个参数你捉摸不透，不必理睬它，options是一个可选参数；callback是回调函数，用于接收返回结果，同样是一个可选参数。

runtime.onMessage完整的方法为：
```

chrome.runtime.onMessage.addListener(callback)

```
此处的callback为必选参数，为回调函数。callback接收到的参数有三个，分别是message、sender和sendResponse，即消息内容、消息发送者相关信息和相应函数。其中sender对象包含4个属性，分别是tab、id、url和tlsChannelId，tab是发起消息的标签，有关标签的内容可以参看4.5节的内容。

为了进一步说明，下面举一个例子。
在popup.html中执行如下代码：
```

chrome.runtime.sendMessage('Hello', function(response){
    document.write(response);
});

```
在background中执行如下代码：
```

chrome.runtime.onMessage.addListener(function(message, sender, sendResponse){
    if(message == 'Hello'){
        sendResponse('Hello from background.');
    }
});

```


----
###  储存数据 
一个程序免不了要储存数据，对于Chrome扩展也是这样。通常Chrome扩展使用以下三种方法中的一种来储存数据：
第一种是使用HTML5的localStorage，这种方法在上一节的内容中已经涉及；
第二种是使用Chrome提供的存储API；
第三种是使用Web SQL Database。

由于上节已经讲解了localStorage的使用方法，下面将详细讲解后两种储存数据的方法。

**Chrome存储API**

Chrome为扩展应用提供了存储API，以便将扩展中需要保存的数据写入本地磁盘。Chrome提供的存储API可以说是对localStorage的改进，它与localStorage相比有以下区别：
如果储存区域指定为sync，数据可以自动同步；
content_scripts可以直接读取数据，而不必通过background页面；
在隐身模式下仍然可以读出之前存储的数据；
读写速度更快；
用户数据可以以对象的类型保存。

对于第二点要进一步说明一下。**首先localStorage是基于域名的，这在前面的小节中已经提到过了。而content_scripts是注入到用户当前浏览页面中的，如果content_scripts直接读取localStorage，所读取到的数据是用户当前浏览页面所在域中的。所以通常的解决办法是content_scripts通过runtime.sendMessage和background通信，由background读写扩展所在域（通常是chrome-extension:/ /extension-id/）的localStorage，然后再传递给content_scripts。**

**使用Chrome存储API必须要在Manifest的permissions中声明"storage"，之后才有权限调用。**Chrome存储API提供了2种储存区域，分别是**sync和local**。两种储存区域的区别在于，sync储存的区域会根据用户当前在Chrome上登陆的Google账户自动同步数据，当无可用网络连接可用时，sync区域对数据的读写和local区域对数据的读写行为一致。

对于每种储存区域，Chrome又提供了5个方法，分别是get、getBytesInUse、set、remove和clear。
get方法即为读取数据，完整的方法为：
```

chrome.storage.StorageArea.get(keys, function(result){
    console.log(result);
});

```
keys可以是字符串、包含多个字符串的数组或对象。如果keys是字符串，则和localStorage的用法类似；如果是数组，则相当于一次读取了多个数据；如果keys是对象，则会先读取以这个对象属性名为键值的数据，如果这个数据不存在则返回keys对象的属性值（比如keys为{'name':'Billy'}，如果name这个值存在，就返回name原有的值，如果不存在就返回Billy）。如果keys为一个空数组（[]）或空对象（{}），则返回一个空列表，如果keys为null，则返回所有存储的数据。

getBytesInUse方法为获取一个数据或多个数据所占用的总空间，返回结果的单位是字节
Chrome同时还为存储API提供了一个onChanged事件，当存储区的数据发生改变时，这个事件会被激发。

onChanged的完整方法为：
```

chrome.storage.onChanged.addListener(function(changes, areaName){
    console.log('Value in '+areaName+' has been changed:');
    console.log(changes);
});

```
callback会接收到两个参数，第一个为changes，第二个是StorageArea。changes是词典对象，键为更改的属性名称，值包含两个属性，分别为oldValue和newValue；StorageArea为local或sync

**Web SQL Database**
Web SQL Database的三个核心方法为openDatabase、transaction和executeSql。openDatabase方法的作用是与数据库建立连接，transaction方法的作用是执行查询，executeSql方法的作用是执行SQL语句。
下面举一个简单的例子：
```

db = openDatabase("db_name", "0.1", "This is a test db.", 1024*1024);
if(!db){
    alert('数据库连接失败。');
}
else {
    db.transaction( function(tx) {
        tx.executeSql(
            "SELECT COUNT(*) FROM db_name",
            [],
            function(tx, result){
                console.log(result);
            },
            function(tx, error){
                alert('查询失败：'+error.message);
            }
        );
    }
}

```
更多关于Web SQL Database的资料可以参考http://www.w3.org/TR/webdatabase/。由于原生的Web SQL Database并不算好用，也有一些开源的二次封装的库来简化Web SQL Database的使用，如https://github.com/KenCorbettJr/html5sql。

##  Chrome扩展的UI界面 
###  Browser Action 
Browser Actions将扩展图标置于Chrome浏览器工具栏中，地址栏的右侧。如果声明了popup页面，当用户点击图标时，在图标的下侧会打开这个页面1。同时图标上面还可以附带badge——一个带有显示有限字符空间的区域——用以显示一些有用的信息，如未读邮件数、当前音乐播放时间等。
Browser Actions可以在Manifest中设定一个默认的图标.也可以通过代码设定
chrome.browserAction.setIcon({path: {'19': 'images/icon19_'+index+'.png'}});
###  右键菜单 
http://www.ituring.com.cn/article/65657
###  桌面提醒 
http://www.ituring.com.cn/article/65941
###  Page Actions 
http://www.ituring.com.cn/article/65947

