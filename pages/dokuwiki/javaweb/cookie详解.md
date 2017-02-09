title: cookie详解 

#  cookie详解 
会话（Session）跟踪是Web程序中常用的技术，用来跟踪用户的整个会话。
常用的会话跟踪技术是Cookie与Session。Cookie通过在客户端记录信息确定用户身份，Session通过在服务器端记录信息确定用户身份。
一般Cookie与Session结合，Cookie在本地存储SessionID和其他一些信息。
HTTP协议是无状态的协议。一旦数据交换完毕，客户端与服务器端的连接就会关闭，再次交换数据需要建立新的连接。这就意味着服务器无法从连接上跟踪会话。Cookie就是这样的一种机制。它可以弥补HTTP协议无状态的不足。
**注意：Cookie功能需要浏览器的支持。**
##  Cookie的不可跨域名性 
**` Cookie的不可跨域名性。 `**根据Cookie规范，浏览器访问Google只会携带Google的Cookie，而不会携带Baidu的Cookie。Google也只能操作Google的Cookie，而不能操作Baidu的Cookie。
Cookie在客户端是由浏览器来管理的。浏览器能够保证Google只会操作Google的Cookie而不会操作Baidu的Cookie，从而保证用户的隐私安全。浏览器判断一个网站是否能操作另一个网站Cookie的依据是域名。Google与Baidu的域名不一样，因此Google不能操作Baidu的Cookie。
需要注意的是，**虽然网站images.google.com与网站www.google.com同属于Google，但是域名不一样，二者同样不能互相操作彼此的Cookie。**
注意：用户登录网站www.google.com之后会发现访问images.google.com时登录信息仍然有效，而普通的Cookie是做不到的。这是因为Google做了特殊处理。本章后面也会对Cookie做类似的处理。
Unicode编码：保存中文
中文与英文字符不同，中文属于Unicode字符，在内存中占4个字符，而英文属于ASCII字符，内存中只占2个字节。Cookie中使用Unicode字符时需要对Unicode字符进行编码，否则会乱码。

##  设置Cookie的所有属性 
除了name与value之外，Cookie还具有其他几个常用的属性。每个属性对应一个getter方法与一个setter方法。Cookie类的所有属性如表1.1所示。
  * String name 该Cookie的名称。Cookie一旦创建，名称便不可更改
  * Object value 该Cookie的值。如果值为Unicode字符，需要为字符编码。如果值为二进制数据，则需要使用BASE64编码
  * int ` maxAge ` 该Cookie**失效的时间，单位秒**。如果为**正数**，则该Cookie在maxAge秒之后失效。如果为**负数**，该Cookie为**临时Cookie**，关闭浏览器即失效，浏览器也不会以任何形式保存该Cookie。` 如果为0，表示删除该Cookie。默认为–1 `
  * boolean secure 该Cookie是否**仅被使用安全协议传输**。安全协议。安全协议有` HTTPS `，SSL等，在网络上传输数据之前先将数据加密。默认为false
  * String path 该Cookie的使用路径。如果设置为“/sessionWeb/”，则只有contextPath为“/sessionWeb”的程序可以访问该Cookie。如果设置为“/”，则本域名下contextPath都可以访问该Cookie。` 注意最后一个字符必须为“/” `
  * String domain 可以访问该Cookie的域名。如果设置为“.google.com”，**则所有以“google.com”结尾的域名**都可以访问该Cookie。` 注意第一个字符必须为“.” `
  * String comment 该Cookie的用处说明。浏览器显示Cookie信息的时候显示该说明
  * int version 该Cookie使用的版本号。0表示遵循Netscape的Cookie规范，1表示遵循W3C的RFC 2109规范

##  Cookie的有效期 
Cookie的maxAge决定着Cookie的有效期，单位为秒（Second）。Cookie中通过getMaxAge()方法与setMaxAge(int maxAge)方法来读写maxAge属性。
  * **如果maxAge属性为正数，**则表示该Cookie会**在maxAge秒之后自动失效**。浏览器会将maxAge为正数的Cookie持久化，即写到对应的Cookie文件中。无论客户关闭了浏览器还是电脑，只要还在maxAge秒之前，登录网站时该Cookie仍然有效。下面代码中的Cookie信息将永远有效。
  * **如果maxAge为负数**，则表示该Cookie仅在本浏览器窗口以及本窗口打开的子窗口内有效，**关闭窗口后该Cookie即失效**。maxAge为负数的Cookie，为临时性Cookie，不会被持久化，不会被写到Cookie文件中。Cookie信息保存在浏览器内存中，因此关闭浏览器该Cookie就消失了。` Cookie默认的maxAge值为–1。 `
  * **如果maxAge为0**，则表示**删除**该Cookie。Cookie机制没有提供删除Cookie的方法，因此通过设置该Cookie即时失效实现删除Cookie的效果。失效的Cookie会被浏览器从Cookie文件或者内存中删除，
```

Cookie cookie = new Cookie("username","helloweenvsfei");   // 新建Cookie
cookie.setMaxAge(Integer.MAX_VALUE);           // 设置生命周期为MAX_VALUE
response.addCookie(cookie);                    // 输出到客户端

```

##  Cookie的修改、删除 
**Cookie并不提供修改、删除操作。**如果要修改某个Cookie，只需要` 新建一个同名的Cookie，添加到response中覆盖原来的Cookie。 `
**如果要删除某个Cookie**，只需要**新建一个同名的Cookie，并将maxAge设置为0**，并添加到response中覆盖原来的Cookie。注意是0而不是负数。负数代表其他的意义。读者可以通过上例的程序进行验证，设置不同的属性。
注意：修改、删除Cookie时，新建的Cookie除value、maxAge之外的所有属性，例如name、path、domain等，都要与原Cookie完全一样。否则，浏览器将视为两个不同的Cookie不予覆盖，导致修改、删除失败。

##  Cookie的域名 
` Cookie是不可跨域名的 `。域名www.google.com颁发的Cookie不会被提交到域名www.baidu.com去。这是由Cookie的隐私安全机制决定的。隐私安全机制能够禁止网站非法获取其他网站的Cookie。
**正常情况下，同一个一级域名下的两个二级域名如www.helloweenvsfei.com和images.helloweenvsfei.com也不能交互使用Cookie**，因为二者的域名并不严格相同。

如果想所有helloweenvsfei.com名下的二级域名都可以使用该Cookie，**需要设置Cookie的domain参数**，例如：
```

Cookie cookie = new Cookie("time","20080808"); // 新建Cookie
cookie.setDomain(".helloweenvsfei.com");           // 设置域名
cookie.setPath("/");                              // 设置路径
cookie.setMaxAge(Integer.MAX_VALUE);               // 设置有效期
response.addCookie(cookie);                       // 输出到客户端

```
注意：domain参数必须以点(".")开始。另外，name相同但domain不同的两个Cookie是两个不同的Cookie。如果想要两个域名完全不同的网站共有Cookie，可以生成两个Cookie，domain属性分别为两个域名，输出到客户端。

##  Cookie的路径 
domain属性决定运行访问Cookie的域名，而path属性决定允许访问Cookie的路径（ContextPath）。例如，如果只允许/sessionWeb/下的程序使用Cookie，可以这么写：
```

Cookie cookie = new Cookie("time","20080808");     // 新建Cookie
cookie.setPath("/session/");                          // 设置路径
response.addCookie(cookie);                           // 输出到客户端

```
**设置为“/”时允许所有路径使用Cookie。**` path属性需要使用符号“/”结尾 `。
注意：` 页面只能获取它属于的Path的Cookie `。**例如/session/test/a.jsp不能获取到路径为/session/abc/的Cookie。使用时一定要注意。**

##   Cookie的安全属性 
HTTP协议不仅是无状态的，而且是不安全的。使用HTTP协议的数据不经过任何加密就直接在网络上传播，有被截获的可能。使用HTTP协议传输很机密的内容是一种隐患。如果不希望Cookie在HTTP等非安全协议中传输，可以设置Cookie的secure属性为true。浏览器只会在HTTPS和SSL等安全协议中传输此类Cookie。下面的代码设置secure属性为true：
```

Cookie cookie = new Cookie("time", "20080808"); // 新建Cookie
cookie.setSecure(true);                           // 设置安全属性
response.addCookie(cookie);                        // 输出到客户端

```
提示：secure属性并不能对Cookie内容加密，因而不能保证绝对的安全性。如果需要高安全性，需要在程序中对Cookie内容加密、解密，以防泄密。

##   JavaScript操作Cookie 
Cookie是保存在浏览器端的，因此浏览器具有操作Cookie的先决条件。浏览器可以使用脚本程序如JavaScript或者VBScript等操作Cookie。这里以JavaScript为例介绍常用的Cookie操作。例如下面的代码会输出本页面所有的Cookie。
```

<script>document.write(document.cookie);</script>

```
由于JavaScript能够任意地读写Cookie，有些好事者便想使用JavaScript程序去窥探用户在其他网站的Cookie。不过这是徒劳的，W3C组织早就意识到JavaScript对Cookie的读写所带来的安全隐患并加以防备了，**W3C标准的浏览器会阻止JavaScript读写任何不属于自己网站的Cookie。**换句话说，A网站的JavaScript程序读写B网站的Cookie不会有任何结果。

##  Session中禁止使用Cookie 
既然WAP上大部分的客户浏览器都不支持Cookie，索性禁止Session使用Cookie，统一使用URL地址重写会更好一些。Java Web规范支持通过配置的方式禁用Cookie。下面举例说一下怎样通过配置禁止使用Cookie。
编辑内容如下：**` /META-INF/context.xml `**
```

<?xml version='1.0' encoding='UTF-8'?>
<Context path="/sessionWeb"cookies="false">
</Context>

```
或者修改Tomcat全局的conf/context.xml，修改内容如下：
代码1.12  context.xml
```

<!-- The contents of this file will be loaded for eachweb application -->
<Context cookies="false">
    <!-- ... 中间代码略 -->
</Context>

```
部署后TOMCAT便不会自动生成名JSESSIONID的Cookie，Session也不会以Cookie为识别标志，而仅仅以重写后的URL地址为识别标志了。
注意：该配置只是禁止Session使用Cookie作为识别标志，并不能阻止其他的Cookie读写。也就是说服务器不会自动维护名为JSESSIONID的Cookie了，但是程序中仍然可以读写其他的Cookie。

参考：http://blog.csdn.net/fangaoxin/article/details/6952954