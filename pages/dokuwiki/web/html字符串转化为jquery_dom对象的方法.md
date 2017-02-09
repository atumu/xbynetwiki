title: html字符串转化为jquery_dom对象的方法 

#  html字符串转化为jquery dom对象的方法 
原html字符串如下：
```

var text="<div id='overLay' style='width:50px;height:60px;background:url(imgs/back.png) left top no-repeat; position: absolute;'>"
        + "<img style='margin-left:4px;margin-top: 3px;' src='ima.png' width='43px' height='43px'/>"
        + "</div>";

```

**1、下面使用Jquery库将text字符串变量转为Jquery对象。**
Jquery代码如下：
alert($(text).html());
其中$(text)就text字符串转为了一个Jquery对象，最后将该Jquery对象的html()将html内容以字符串的形式输出，结果如下：
<img style='margin-left:4px;margin-top: 3px;' src='ima.png' width='43px' height='43px'/>
明了，$(text)Jquery对象代表的是最外层的html元素div。

**2、将Jquery对象和DOM对象之间互转。**
代码如下：
```

var element= $(text).get(0) //element就是一个dom对象
var jqueryobj=$(element);//jqueryobj就是一个Jquery对象。
$("<body></body>").append($(text));

```
` 注意：DOM对象和Jquery对象区别 `
在我理解，Jquery对象和DOM对象都是封装的html元素，可以对html元素节点进行操作，方便编程，但是他们之间的方法有些是不能共用的，如Jquery对象的html()方法，DOM对象就使用不了；而DOM对象的GetElementById()，Jquery对象也不能使用。所以在必须要的时候可以进行相互转换。

**3、使用js代码将text字符串变量转为DOM对象**
```

/*字符串转dom对象*/
function loadXMLString(txt) 
{
  try //Internet Explorer
   {
     xmlDoc=new ActiveXObject("Microsoft.XMLDOM");
     xmlDoc.async="false";
     xmlDoc.loadXML(txt);
     //alert('IE');
     return(xmlDoc); 
   }
  catch(e)
   {
     try //Firefox, Mozilla, Opera, etc.
      {
        parser=new DOMParser();
        xmlDoc=parser.parseFromString(txt,"text/xml");
       //alert('FMO');
        return(xmlDoc);
      }
     catch(e) {alert(e.message)}
   }
  return(null);
}

```
其中js代码将text字符串转为DOM对象与浏览器有关，所以。。。。。。分开写。　　
这样就实现了html字符串向Jquery对象和DOM对象的转换。

**jQuery对象与dom对象相互转换方法介绍**
刚开始学习jQuery，可能一时会分不清楚哪些是jQuery对象，哪些是DOM对象。至于DOM对象不多解释，我们接触的太多了，下面重点介绍一下jQuery，以及两者相互间的转换。
什么是jQuery对象？
就是通过jQuery包装DOM对象后产生的对象。jQuery对象是jQuery独有的，其可以使用jQuery里的方法。
比如：
$("#test").html() 意思是指：获取ID为test的元素内的html代码。其中html()是jQuery里的方法
这段代码等同于用DOM实现代码：
document.getElementById("id").innerHTML;
**虽然jQuery对象是包装DOM对象后产生的，但是jQuery无法使用DOM对象的任何方法，同理DOM对象也不能使用jQuery里的方法.乱使用会报错。比如：$("#test").innerHTML、document.getElementById("id").html()之类的写法都是错误的。**
还有一个要注意的是：用#id作为选择符取得的是jQuery对象与document.getElementById("id")得到的DOM对象，这两者并不等价。请参看如下说的两者间的转换。
既然jQuery有区别但也有联系，那么jQuery对象与DOM对象也可以相互转换。**在再两者转换前首先我们给一个约定：如果一个获取的是 jQuery对象，那么我们在变量前面加上$，如：var $variab = jQuery对象；如果获取的是DOM对象，则与习惯普通一样：var variab = DOM对象；` 这么约定只是便于讲解与区别，实际使用中并不规定。 `**
jQuery对象转成DOM对象：
两种转换方式将一个jQuery对象转换成DOM对象：[index]和.get(index);
(1)jQuery对象是一个数据对象，可以通过[index]的方法，来得到相应的DOM对象。
如：
```

var $v =$("#v") ; //jQuery对象
var v=$v[0]; //DOM对象
alert(v.checked) //检测这个checkbox是否被选中

```
(2)jQuery本身提供，通过.get(index)方法，得到相应的DOM对象
如：
```

var $v=$("#v"); //jQuery对象
var v=$v.get(0); //DOM对象
alert(v.checked) //检测这个checkbox是否被选中

```
DOM对象转成jQuery对象:
对于已经是一个DOM对象，只需要用$()把DOM对象包装起来，就可以获得一个jQuery对象了。$(DOM对象)
如：
```

var v=document.getElementById("v"); //DOM对象
var $v=$(v); //jQuery对象

```
转换后，就可以任意使用jQuery的方法了。
通过以上方法，可以任意的相互转换jQuery对象和DOM对象。需要再强调注意的是：DOM对象才能使用DOM中的方法，jQuery对象是不可以用DOM中的方法。
参考：http://www.jb51.net/article/71605.htm