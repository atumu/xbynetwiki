title: chrome_ie_firefox查看http请求响应头 

#  Google Chrome/IE/FireFox查看HTTP请求头request header响应头 
chrome查看网页header，鼠标**右键打开审查元素，或快捷键Shift+Ctrl+I**或者shift+ctrl+c
当我**打开Network**后，发现里面是空的什么也没有。
**查了下，才知，需要刷新页面才能显示出来。**
想了想也是应该，**只有重新载入网页，chrome才能捕获header信息。**
据说这个功能很好很强大，可以用来找到隐藏的视频文件源地址。很多非专业人士用审查元素好像也就是来干这个。
IE和FireFox查看页面header信息需要插件
IE:HttpWatch,Fiddler2
FireFox:Firebug
————————
##  chrome如何查看网页header信息 
1,**Shift+Ctrl+I** 调出 我们牛逼的,性感代码式的调试工作台~~
**2,然后载入网页.**
3,再然后,去看**Network**信息…
4,亮点来了,点击Network信息的第一个.

使用**chrome**浏览器自带的开发者工具查看**http**头的方法
1.在网页任意地方右击选择**审查元素**或者按下**shift+ctrl+c**打开chrome自带的调试工具;
2.选择**network**标签,**刷新网页(在打开调试工具的情况下刷新);**
3.刷新后在左边**找到该网页url**,点击 后右边**选择headers**,就可以看到当前网页的http头了;


请求Header(HTTP request header )
Host 请求的域名
User-Agent 浏览器端浏览器型号和版本
Accept 可接受的内容类型
Accept-Language 语言
Accept-Encoding 可接受的压缩类型 gzip,deflate
Accept-Charset 可接受的内容编码 UTF-8,*

服务器端的响应Header(response header)
Date 服务器端时间
Server 服务器端的服务器软件 Apache/2.2.6
Etag 文件标识符
Content-Encoding传送启用了GZIP压缩 gzip
Content-Length 内容长度
Content-Type 内容类型
响应Headers，我们应该时刻留意它们。这些信息无法直接获取，需要依靠第三方工具。
