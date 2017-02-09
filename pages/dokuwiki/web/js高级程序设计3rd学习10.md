title: js高级程序设计3rd学习10 

#  JS高级程序设计3rd学习10离线应用与客户端存储 
支持离线web应用开发是HTML5的另一个重点。
##  离线检测 
```

if(navigator.onLine){
//正常工作
}else{
//执行离线状态时任务
}

```
除了navigator.onLine属性之外，为了更好地确定网络是否可用，HTML5还定义了两个事件:online和offline.当网络状态变化时会分别触发它们。
```

        window.addHandler("online", function(){
            document.getElementById("status").innerHTML = "Online";
        });
        window.addHandler( "offline", function(){
            document.getElementById("status").innerHTML = "Offline";
        });

```
##  应用缓存 
HTML5的应用缓存是专门为开发离线web应用设计的。它是从浏览器缓存中分出来的一块缓存区。通过使用一个描述文件，列出要下载和缓存的资源即可。
API的核心是applicationCache对象。
具体略。
##  数据存储 
###  Cookie 
略。
###  Web Storage 
HTML5定义了Web Storage API目的是克服cookie带来的一些限制(主要是大小限制和共享限制)
IE8+也支持它们。
Web Storage也有分两类sessionStorage和localStorage，这两个差别就跟session和cookie一样，**sessionStorage关闭浏览器后过期**，**localStrage不会过期**，但是跟cookie不一样的是没有过期时间。
  * localStorage - 没有时间限制的数据存储
  * sessionStorage - 针对一个 session 的数据存储
当然web storage也是和域名相关联的。
对比session和cookie的优点主要提现在一下三点：
1、容量大，不同浏览器支持大小不一致。chrome与safari、IOS、Android是2.5M。其余一般是5M
**2、不会随着回话来传输。**
3、读取和写入方便，sessionStorage与localStorage都是Storage的实例，有如下方法:
  * setItem(key,value)
  * getItem(key)
  * clear()
  * removeItem(key)
  * key(index)
还有一个length属性
**Storage只能存储字符串**，非字符串的数据都会被转换成字符串
```

                var i, key, value;
                for (i=0, len = localStorage.length; i < len; i++){
                    key = localStorage.key(i);
                    value = localStorage.getItem(key);
                    alert(key + "=" + value);
                }            
            
            //set some data
            localStorage.setItem("name", "Nicholas");
            localStorage.setItem("book", "Professional JavaScript");

```
####  storage事件 
对Storage对象进行任何修改都会在document上触发storage事件。这个事件的event对象有如下属性:
  * domain
  * key
  * newValue
  * oldValue
```

     document.addHandler("storage", function(event){
                //most browsers haven't implemented this properties yet
                //alert("Storage changed. Name '" + event.key + "' changed from '" + event.oldValue + "' to '" + event.newValue + "'");
                alert("Storage changed for " + event.domain);            
            });

```
##  IndexedDB 
Indexed Database API是在浏览器中保存结构化数据的一种数据库。目前API还不稳定。
略。
