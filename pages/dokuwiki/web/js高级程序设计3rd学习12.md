title: js高级程序设计3rd学习12 

#  JS高级程序设计3rd学习12新兴API 
##  Geolocation API(地理定位geolocation) 
HTML5 Geolocation（地理定位）用于定位用户的位置。
定位用户的位置
HTML5 Geolocation API 用于获得用户的地理位置。
鉴于该特性可能侵犯用户的隐私，除非用户同意，否则用户位置信息是不可用的。
浏览器支持
Internet Explorer 9、Firefox、Chrome、Safari 以及 Opera 支持地理定位。
注释：对于拥有 GPS 的设备，比如 iPhone，地理定位更加精确。
**请使用 getCurrentPosition() 方法来获得用户的位置。**
下例是一个简单的地理定位实例，可返回用户位置的经度和纬度。
```

var x=document.getElementById("demo");
function getLocation()
  {
  if (navigator.geolocation)
    {
    navigator.geolocation.getCurrentPosition(showPosition);
    }
  else{x.innerHTML="Geolocation is not supported by this browser.";}
  }
function showPosition(position)
  {
  x.innerHTML="Latitude: " + position.coords.latitude +
  "<br />Longitude: " + position.coords.longitude;
  }


```
例子解释：
检测是否支持地理定位
如果支持，则运行 getCurrentPosition() 方法。如果不支持，则向用户显示一段消息。
如果getCurrentPosition()运行成功，则向参数showPosition中规定的函数返回一个coordinates对象
showPosition() 函数获得并显示经度和纬度
上面的例子是一个非常基础的地理定位脚本，不含错误处理。
###  处理错误和拒绝 
getCurrentPosition() 方法的第二个参数用于处理错误。它规定当获取用户位置失败时运行的函数：
```

function showError(error)
  {
  switch(error.code)
    {
    case error.PERMISSION_DENIED:
      x.innerHTML="User denied the request for Geolocation."
      break;
    case error.POSITION_UNAVAILABLE:
      x.innerHTML="Location information is unavailable."
      break;
    case error.TIMEOUT:
      x.innerHTML="The request to get user location timed out."
      break;
    case error.UNKNOWN_ERROR:
      x.innerHTML="An unknown error occurred."
      break;
    }
  }

```
错误代码：
Permission denied - 用户不允许地理定位
Position unavailable - 无法获取当前位置
Timeout - 操作超时
###  在地图中显示结果 
如需在地图中显示结果，您需要访问可使用经纬度的地图服务，比如谷歌地图或百度地图：
```

function showPosition(position)
{
var latlon=position.coords.latitude+","+position.coords.longitude;
var img_url="http://maps.googleapis.com/maps/api/staticmap?center="
+latlon+"&zoom=14&size=400x300&sensor=false";

document.getElementById("mapholder").innerHTML="<img src='"+img_url+"' />";
}

```
getCurrentPosition() 方法 - 返回数据
若成功，则 getCurrentPosition() 方法返回对象。始终会返回 latitude、longitude 以及 accuracy 属性。如果可用，则会返回其他下面的属性。
  * coords.latitude	十进制数的纬度
  * coords.longitude	十进制数的经度
  * coords.accuracy	位置精度
  * coords.altitude	海拔，海平面以上以米计
  * coords.altitudeAccuracy	位置的海拔精度
  * coords.heading	方向，从正北开始以度计
  * coords.speed	速度，以米/每秒计
  * timestamp	响应的日期/时间

###  Geolocation跟踪监控 
` watchPosition() ` - 返回一个ID，后续可将这个ID传入clearWatch(ID)来取消跟踪，并继续返回用户移动时的更新位置（就像汽车上的 GPS）。
` clearWatch() ` - 停止 watchPosition() 方法
下面的例子展示 watchPosition() 方法。您需要一台精确的 GPS 设备来测试该例（比如 iPhone）：
```


var x=document.getElementById("demo");
function getLocation()
  {
  if (navigator.geolocation)
    {
    navigator.geolocation.watchPosition(showPosition);
    }
  else{x.innerHTML="Geolocation is not supported by this browser.";}
  }
function showPosition(position)
  {
  x.innerHTML="Latitude: " + position.coords.latitude +
  "<br />Longitude: " + position.coords.longitude;
  }


```

##  File API 
在之前我们操作本地文件都是使用flash、silverlight或者第三方的activeX插件等技术，由于使用了这些技术后就很难进行跨平台、或者跨浏览器、跨设备等情况下实现统一的表现，从另外一个角度来说就是让我们的web应用依赖了第三方的插件，而不是很独立，不够通用。
在HTML5标准中，默认提供了操作文件的API让这一切直接标准化。有了操作文件的API，让我们的Web应用可以很轻松的通过JS来控制文件的读取、写入、文件夹、文件等一系列的操作.但是最新的标准中大部分浏览器都已经实现了文件的读取API，文件的写入，文件和文件夹的最新的标准刚制定完毕，相信后面随着浏览器的升级这些功能肯定会实现的非常好，接下来介绍文件读取的几个API。
###  几个重要的JS对象 
**1):FileList对象**
它是File对象的一个集合，在Html4标准中文件上传控件只接受一个文件，而在新标准中，只需要设置multiple，就支持多文件上传，所以从此标签中获取的**files属性就是FileList对象实例**。
demo：<input type="file" multiple="multiple" name="fileDemo" id="fileDemo" />  
document.getElementById("fileDemo").files返回一个FileList对象实例

**2)Blob对象**
其实就是一个**原始数据对象，它提供了` slice方法 `可以读取原始数据中的某块数据。**另外有两个属性：**size**（数据的大小），**type**（数据的MIME类型）；看下面的是W3C的API原型：
```

    interface Blob {
      readonly attribute unsigned long long size;
      readonly attribute DOMString type;
      //slice Blob into byte-ranged chunks     
      Blob slice(optional long long start,
                 optional long long end,
                 optional DOMString contentType); 
    
    };

```
###  3）File对象 
**继承自Blob对象**，指向一个具体的文件，它还有两个属性：**name**（文件名), **lastModifiedDate**（最后修改时间)；
由于File继承自Blob，所以也有**size**（数据的大小），**type**（数据的MIME类型）属性、slice方法。
我们可以通过监听**change事件**并读取files集合，或者单独设一个按钮监听点击来获取files集合
```

    <input type="file" multiple id="files-list">
    <script>
        window.onload = function(){
            var filesListInput = document.getElementById("files-list");
            filesListInput.addEventListener("change", function(event){
                var info = "",
                    output = document.getElementById("output"),
                    files = event.target.files,//读取files属性
                    i = 0,
                    len = files.length;
                
                while (i < len){
                    info += files[i].name + " (" + files[i].type + ", " + files[i].size + " bytes)<br>";
                    i++;
                }
                
                output.innerHTML = info;
            });
        };
    </script>
    <div id="output"></div>

```
###  4）FileReader对象 
设计用来读取文件里面的数据，提供三个常用的读取文件数据的方法，另外读取文件数据使用了**异步**的方式，非常高效。然后让我们看一些W3C的标准：
这个对象是非常重要第一个对象，它提供了四个读取文件数据的方法，这些方法都是异步的方式读取数据，**读取成功后就直接将结果放到` 属性result `中**。所以**一般就是直接读取数据，然后监听此对象的onload事件，然后在事件里面读取result属性**，再做后续处理。**当然abort就是停止读取的方法**。其他的就是事件和状态不再赘述。
  * **readAsText**(file,encoding);→第一个参数传入Blog对象，然后第二个参数传入编码格式，异步将数据读取成功后放到result属性中，读取的内容是普通的文本字符串的形式。
  * **readAsDataURL**(file);→传入一个Blob对象，读取内容可以做为URL属性，**也就是说可以将一个图片的结果指向给一个img的src属性。 ** 
  * readAsBinaryString(file);  → 传入一个Blob对象(注意File继承自Blob)，然后读取数据的结果作为二进制字符串的形式放到FileReader的result属性中。
  * readAsArrayBuffer(file);
**FileReader特提供了几个事件**。其中最有用的三个事件是progress、load、error
  * progress事件属性:lengthComputable,loaded,total
  * error事件属性code:1表示未找到文件，2表示安全性错误，3表示读取中断，4表示文件不可读，5表示编码错误
  * load事件：文件成功加载后触发。
```

    <input type="file" id="files-list">
    <script>
        window.onload = function(){
            
            var filesList = document.getElementById("files-list");
            EventUtil.addHandler(filesList, "change", function(event){
                var info = "",
                    output = document.getElementById("output"),
                    progress = document.getElementById("progress"),
                    files = EventUtil.getTarget(event).files,
                    type = "default",
                    reader = new FileReader();
                    
                if (/image/.test(files[0].type)){
                    reader.readAsDataURL(files[0]);
                    type = "image";
                } else {
                    reader.readAsText(files[0]);
                    type = "text";
                }
                    
                reader.onerror = function(){
                    output.innerHTML = "Could not read file, error code is " + reader.error.code;
                };
                
                reader.onprogress = function(event){
                    if (event.lengthComputable){
                        progress.innerHTML = event.loaded + "/" + event.total;
                    }
                };
                
                reader.onload = function(){
                
                    var html = "";
                    
                    switch(type){
                        case "image":
                            html = "<img src=\"" + reader.result + "\">";
                            break;
                        case "text":
                            html = reader.result;
                            break;
                            
                    }
                    output.innerHTML = html;
                };
            });
        };
        
    </script>
    <div id="progress"></div>
    <pre id="output"></pre>

```
如果想中断读取过程可以调用abort()方法。这样就会触发abort事件。
###  读取部分内容 
有时我们只是想读取文件的一部分而不是全部内容。为此，File对象还支持一个slice()方法。
```

        function blobSlice(blob, startByte, length){
            if (blob.slice){
                return blob.slice(startByte, length);
            } else if (blob.webkitSlice){
                return blob.webkitSlice(startByte, length);
            } else if (blob.mozSlice){
                return blob.mozSlice(startByte, length);
            } else {
                return null;
            }
        }

```
```

        window.onload = function(){
            
            var filesList = document.getElementById("files-list");
            EventUtil.addHandler(filesList, "change", function(event){
                var info = "",
                    output = document.getElementById("output"),
                    progress = document.getElementById("progress"),
                    files = EventUtil.getTarget(event).files,
                    reader = new FileReader(),
                    blob = blobSlice(files[0], 0, 32);

                if (blob){
                    reader.readAsText(blob);
                    
                    reader.onerror = function(){
                        output.innerHTML = "Could not read file, error code is " + reader.error.code;
                    };

                    reader.onload = function(){
                        output.innerHTML = reader.result;
                    };
                } else {
                    alert("Your browser doesn't support slice().");
                }
            });
        };
        
    </script>
    <pre id="output"></pre>

```

###  对象URL 
略。
##  读取拖放文件并上传 
通过HTML5 File API与拖放支持，当触发drop事件时，可以通过event.dataTransfer.files中读取被放置的文件。
###  读取拖放的文件 
```

    <div id="droptarget" style="width: 500px; height: 200px; background: silver">
        Drop some files here
    </div>
    
    <script>
        window.onload = function(){
            var droptarget = document.getElementById("droptarget");
            function handleEvent(event){
                var info = "",
                    output = document.getElementById("output"),
                    files, i, len;            
            
                EventUtil.preventDefault(event);
                
                if (event.type == "drop"){
                    files = event.dataTransfer.files;
                    i = 0;
                    len = files.length;
                
                    while (i < len){
                        info += files[i].name + " (" + files[i].type + ", " + files[i].size + " bytes)<br>";
                        i++;
                    }
                    
                    output.innerHTML = info;
                }
            }

            EventUtil.addHandler(droptarget, "dragenter", handleEvent);
            EventUtil.addHandler(droptarget, "dragover", handleEvent);
            EventUtil.addHandler(droptarget, "drop", handleEvent);        

        };
        
    </script>

```
###  使用XHR上传文件 
使用FormData对象结合XHR的send()方法就可以实现了。对于服务器而言这仍然是普通的表单提交。
```

    <div id="droptarget" style="width: 500px; height: 200px; background: silver">
        Drop some files here
    </div>
    
    <script>
    
        window.onload = function(){
        
            var droptarget = document.getElementById("droptarget");
            
            function handleEvent(event){
                var info = "",
                    output = document.getElementById("output"),
                    data, xhr,
                    files, i, len;            
            
                EventUtil.preventDefault(event);
                
                if (event.type == "drop"){
                    data = new FormData();
                    files = event.dataTransfer.files;
                    i = 0;
                    len = files.length;
                
                    while (i < len){
                        data.append("file" + i, files[i]);
                        i++;
                    }
                    
                    xhr = new XMLHttpRequest();
                    xhr.open("post", "FileAPIExample06Upload.php", true);
                    xhr.onreadystatechange = function(){
                        if (xhr.readyState == 4){
                            alert(xhr.responseText);
                        }
                    };
                    xhr.send(data);
                }
            }

            EventUtil.addHandler(droptarget, "dragenter", handleEvent);
            EventUtil.addHandler(droptarget, "dragover", handleEvent);
            EventUtil.addHandler(droptarget, "drop", handleEvent);        

        };
        
    </script>
    <pre id="output"></pre>

```
