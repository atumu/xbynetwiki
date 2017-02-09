title: ajax学 

#  Ajax学习 
XMLHttpRequest
setTimeout("sendEmptyRequest()" , 800);是js的计时器，指定800ms之后再执行sendEmptyRequest()函数。
```

XMLHttpReq = new XMLHttpRequest();
// 发送请求函数
function sendRequest()
{
	// input是个全局变量，就是用户输入聊天信息的单行文本框
	var chatMsg = input.value;
	// 完成XMLHttpRequest对象的初始化
	createXMLHttpRequest();
	// 定义发送请求的目标URL
	var url = "chat.do";
	// 通过open方法取得与服务器的连接
	// 发送POST请求
	XMLHttpReq.open("POST", url, true);
	// 设置请求头-发送POST请求时需要该请求头
	XMLHttpReq.setRequestHeader("Content-Type",
		"application/x-www-form-urlencoded");
	// 指定XMLHttpRequest状态改变时的处理函数
	XMLHttpReq.onreadystatechange = processResponse;
	// 清空输入框的内容
	input.value = "";
	// 发送请求，send的参数包含许多的key-value对。
	// 即以：请求参数名=请求参数值 的形式发送请求参数。
	XMLHttpReq.send("chatMsg=" + chatMsg); 
}

function sendEmptyRequest()
{
	// 完成XMLHttpRequest对象的初始化
	createXMLHttpRequest();
	// 定义发送请求的目标URL
	var url = "chat.do";
	// 发送POST请求
	XMLHttpReq.open("POST", url, true);
	// 设置请求头-发送POST请求时需要该请求头
	XMLHttpReq.setRequestHeader("Content-Type",
		"application/x-www-form-urlencoded");
	// 指定XMLHttpRequest状态改变时的处理函数
	XMLHttpReq.onreadystatechange = processResponse;
	// 发送请求,，不发送任何参数
	XMLHttpReq.send(null);
	// 指定0.8s之后再次发送请求
	setTimeout("sendEmptyRequest()" , 800);
}
// 处理返回信息函数
function processResponse()
{
	// 当XMLHttpRequest读取服务器响应完成
	if (XMLHttpReq.readyState == 4)
	{
		// 服务器响应正确（当服务器响应正确时，返回值为200的状态码）
		if (XMLHttpReq.status == 200)
		{
			// 使用chatArea多行文本域显示服务器响应的文本
			document.getElementById("chatArea").value 
				= XMLHttpReq.responseText;
		}
		else
		{
			// 提示页面不正常
			window.alert("您所请求的页面有异常。");
		}
	}
}
function enterHandler(event)
{
	// 获取用户单击键盘的“键值”
	var keyCode = event.keyCode ? event.keyCode 
		: event.which ? event.which : event.charCode;
	// 如果是回车键
	if (keyCode == 13)
	{
		sendRequest();
	}
}

```
##  XMLHttpRequest介绍 
**方法：**
![](/data/dokuwiki/web/pasted/20150724-023600.png)
![](/data/dokuwiki/web/pasted/20150724-023611.png)
abort():终止请求发送
getAllResponseHeader()
getResponseHeader(key)

**属性：**
responseText	获得字符串形式的响应数据。
responseXML	获得 XML 形式的响应数据。
![](/data/dokuwiki/web/pasted/20150724-023653.png)
```

<%@ page contentType="text/html; charset=GBK" language="java" %>
<%
// 处理POST请求，POST请求通过request.setCharacterEncoding("UTF-8");解决乱码问题。
if (request.getMethod().equalsIgnoreCase("POST"))
{
	request.setCharacterEncoding("UTF-8");
	System.out.println(request.getParameter("value"));
}
// 处理GET请求,对于中文请求参数，发送过来的时候是url编码ISO-8859-1,所以我们需要转换编码为utf-8否则服务器端就会出现乱码。
else if (request.getMethod().equalsIgnoreCase("GET"))
{
	String tmp = request.getParameter("value");
	String a = new String(tmp.getBytes("ISO-8859-1") , "UTF-8");
	System.out.println(a);
}
%>

```
##  解析XML响应 
使用JS内置的eval()函数解析JSON可能有一些潜在的安全问题，为了避免这个问题，可以使用一些更安全的库提供的函数来解析JSON.
```

xmlhttp.onreadystatechange=function()
  {
  if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
    xmlDoc=xmlhttp.responseXML;
    txt="";
    x=xmlDoc.getElementsByTagName("title");
    for (i=0;i<x.length;i++)
      {
      txt=txt + x[i].childNodes[0].nodeValue + "<br />";
      }
    document.getElementById("myDiv").innerHTML=txt;
    }
  }
xmlhttp.open("GET","/example/xmle/books.xml",true);
xmlhttp.send();
}


```
##  获取json响应 
```

// 获取服务器的JSON响应
			// 并调用eval()函数将服务器响应解析成JavaScript数组
			var books = eval(xmlrequest.responseText);
			// 遍历数组，每个数组元素生成一个表格行
			for (var i = 0 , len = books.length ; i < len ; i++)
			{
				var tr = bookTb.insertRow(i);
				// 依次创建4个单元格，并为单元格设置内容
				var cell0 = tr.insertCell(0);
				cell0.innerHTML = books[i].id;
				var cell1 = tr.insertCell(1);
				cell1.innerHTML = books[i].name;
				var cell2 = tr.insertCell(2);
				cell2.innerHTML = books[i].author;
				var cell3 = tr.insertCell(3);
				cell3.innerHTML = books[i].price;
			}

```

