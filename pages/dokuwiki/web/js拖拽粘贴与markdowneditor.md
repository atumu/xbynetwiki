title: js拖拽粘贴与markdowneditor 

#  js拖拽粘贴与MarkDownEditor改造 
Markdown编辑器选用https://simplemde.com
它是一款纯js实现的markdown编辑器。缺点不支持图片上传。那我们就得改造它。
simplemde是基于[codemirror](http://codemirror.net/)编辑器的.
先介绍基本：
codemirror文档：http://codemirror.net/doc/manual.html
simplemde文档:https://github.com/NextStepWebs/simplemde-markdown-editor
API文档:
拖拽：
https://developer.mozilla.org/en-US/docs/Web/API/DragEvent
https://developer.mozilla.org/en-US/docs/Web/API/DragEvent/dataTransfer

粘贴：
https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent
https://developer.mozilla.org/en-US/docs/Web/Events/paste

注意一点：目前firefox与chrome比较新的版本都实现了这些API。
##  paste事件 
绑定的元素不一定是input，普通的div也是可以绑定的，如果是给document绑定了，就相当于全局了，任何时候的粘贴操作都会触发。
获取事件对象**ClipboardEvent**
先写一下事件绑定的代码
```

pasteEle.addEventListener("paste", function (e){
    if ( !(e.clipboardData && e.clipboardData.items) ) {
        return ;
    }

    for (var i = 0, len = e.clipboardData.items.length; i < len; i++) {
        var item = e.clipboardData.items[i];

        if (item.kind === "string") {
            item.getAsString(function (str) {
                // str 是获取到的字符串
            })
        } else if (item.kind === "file") {
            var pasteFile = item.getAsFile();
            // pasteFile就是获取到的文件
        }
    }
});

```
粘贴事件提供了一个clipboardData的属性，如果该属性有items属性，那么就可以查看items中是否有图片类型的数据了。
**clipboardData介绍**
介绍一下clipboardData对象，它实际上是一个DataTransfer类型的对象，DataTransfer 是拖动产生的一个对象，但实际上粘贴事件也是它。
属性介绍
  * dropEffect	String	默认是 none
  * effectAllowed	String	默认是 uninitialized
  * files	FileList	**在粘贴操作时为空List**
  * **items**	DataTransferItemList	剪切板中的各项数据
  * **types**	Array	剪切板中的数据类型。

**DataTransferItem**
items是一个DataTransferItemList对象，自然里面都是DataTransferItem类型的数据了。
DataTransferItem有两个属性kind和type
  * kind	一般为string或者file
  * type	具体的数据类型，例如具体是哪种类型字符串或者哪种类型的文件，即MIME-Type，**常见的值有text/plain、text/html、Files**。
方法
  * getAsFile	空	如果kind是file，可以用该方法获取到文件
  * getAsString	回调函数	如果kind是string，可以用该方法获取到字符串，**字符串需要用回调函数得到**，回调函数的第一个参数就是剪切板中的字符串

综合
```

// demo 程序将粘贴事件绑定到 document 上
document.addEventListener("paste", function (e) {
    var cbd = e.clipboardData;
    //var ua = window.navigator.userAgent;

    for(var i = 0; i < cbd.items.length; i++) {
        var item = cbd.items[i];
        if(item.kind == "file"){
            var blob = item.getAsFile();
            if (blob.size === 0) {
                return;
            }
            // blob 就是从剪切板获得的文件 可以进行上传或其他操作
        }
    }
}, false);

```

##  drop事件 
**DragEvent**
**DragEvent.dataTransfer**
  * dropEffect	String	默认是 none
  * effectAllowed	String	默认是 uninitialized
  * files	FileList	
  * **items**	DataTransferItemList	剪切板中的各项数据
  * **types**	Array	剪切板中的数据类型。
**DataTransferItem**
items是一个DataTransferItemList对象，自然里面都是DataTransferItem类型的数据了。
DataTransferItem有两个属性kind和type
  * kind	一般为string或者file
  * type	具体的数据类型，例如具体是哪种类型字符串或者哪种类型的文件，即MIME-Type，**常见的值有images/*、text/plain、text/html、Files**。
方法
  * getAsFile	空	如果kind是file，可以用该方法获取到文件
  * getAsString	回调函数	如果kind是string，可以用该方法获取到字符串，**字符串需要用回调函数得到**，回调函数的第一个参数就是剪切板中的字符串

```

dropEle.addEventListener("drop", function (e){
	var data = new FormData();
	var files = event.dataTransfer.files;
	var i = 0;
	var len = files.length;
	while (i < len){
	    data.append("file" + i, files[i]);
	     i++;
	}
	var xhr = new XMLHttpRequest();
	xhr.open("post", "/upload", true);
	xhr.onreadystatechange = function(){
	     if (xhr.readyState == 4){
	         alert(xhr.responseText);
	     }
	 };
	 xhr.send(data);
});

```

阻止浏览器默认打开拖拽文件的行为：参考[这里](http://stackoverflow.com/questions/6756583/prevent-browser-from-loading-a-drag-and-dropped-file)
```

window.addEventListener("drop",function(e){
  e = e || event;
  console.log(e);
  //e.preventDefault();
  if (e.target.tagName != "textarea") {  // check wich element is our target
    e.preventDefault();
  }  
},false);

```

理论知识说完了。下面开始实验改造codemirror
##  改造codemirror支持粘贴和拖拽上传 
```

<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<title>codemirror</title>
	<link rel="stylesheet" href="codemirror.css">
	<script src="codemirror.js"></script>
</head>
<body>
	<textarea name="aaa" id="aaa"></textarea>
	<script>
	var textarea=document.getElementById("aaa")
  	var editor = CodeMirror.fromTextArea(textarea, {
    	lineNumbers: true
  	});
  editor.on("paste",function(editor,e){
  	// console.log(e.clipboardData)
  	if(!(e.clipboardData&&e.clipboardData.items)){
  		alert("该浏览器不支持操作");
  		return;
  	}
  	for (var i = 0, len = e.clipboardData.items.length; i < len; i++) {
        var item = e.clipboardData.items[i];
       // console.log(item.kind+":"+item.type);
        if (item.kind === "string") {
            item.getAsString(function (str) {
                // str 是获取到的字符串
            })
        } else if (item.kind === "file") {
            var blob = item.getAsFile();
            // pasteFile就是获取到的文件
            console.log(blob);
            var file = new File([blob], "image.png", {type:"image/png"});
            fileUpload(file);
        }
    }
  });
  editor.on("drop",function(editor,e){
  	// console.log(e.dataTransfer.files[0]);
  	if(!(e.dataTransfer&&e.dataTransfer.files)){
  		alert("该浏览器不支持操作");
  		return;
  	}
  	for(var i=0;i<e.dataTransfer.files.length;i++){
  		console.log(e.dataTransfer.files[i]);
  		fileUpload(e.dataTransfer.files[i]);
  	}
  	e.preventDefault();
  });
  //文件上传
  function fileUpload(fileObj){
  	var data = new FormData();
  	data.append("file",fileObj);
  	var xhr = new XMLHttpRequest();
	xhr.open("post", "/upload", true);
	xhr.onreadystatechange = function(){
	     if (xhr.readyState == 4){
	         alert(xhr.responseText);
	     }
	 };
	 xhr.send(data);
  }
  //阻止浏览器默认打开拖拽文件的行为
  window.addEventListener("drop",function(e){
  e = e || event;
  e.preventDefault();
  if (e.target.tagName == "textarea") {  // check wich element is our target
    e.preventDefault();
  } 
},false);
</script>
</html>

```
###  附Codemirror常用事件与方法 
参考[这里](http://blog.csdn.net/mafan121/article/details/49178945)
1.onChange(instance,changeObj)：codeMirror文本被修改后触发。
instance是一个当前的codemirror对象，changeObj是一个｛from，to，text[,removed][，origin]｝对象。其中from，to分别表示起始行对象和结束行对象，行对象包括ch：改变位置距离行头的间隔字符，line：改变的行数。text是一个字符串数组表示被修改的文本内容，即你输入的内容。

2.onBeforeChange(instance,changObj):内容改变前被调用
3.onCursorActivity(instance)：当鼠标点击内容区、选中内容、修改内容时被触发
4.onKeyHandled:(instance,name,event):当一个都dom元素的事件触发时调用，name为操作名称。
5.onInputRead(insatance,changeObj):当一个新的input从隐藏的textara读取出时调用
6.onBeforeSelectionChange(instance,obj):当选中的区域被改变时调用，obj对象是选择的范围和改变的内容（本人未测试成功）
7.onUpdate(instance):编辑器内容被改变时触发
8.onFocus(instance):编辑器获得焦点式触发
9.onBlur(instance):编辑器失去焦点时触发

常用方法：
getValue():获取编辑器文本内容
setValue(text):设置编辑器文本内容
getRange({line,ch},{line,ch}):获取指定范围内的文本内容第一个对象是起始坐标，第二个是结束坐标
replaceRange(replaceStr,{line,ch},{line,ch}):替换指定区域的内容
getLine(line)：获取指定行的文本内容
lineCount():统计编辑器内容行数
firstLine():获取第一行行数，默认为0，从开始计数
lastLine():获取最后一行行数
getLineHandle(line):根据行号获取行句柄
getSelection():获取鼠标选中区域的代码
replaceSelection(str):替换选中区域的代码
setSelection({line:num,ch:num1},{line:num2,ch:num3}):设置一个区域被选中
somethingSelected()：判断是否被选择
getEditor()：获取CodeMirror对像
undo()：撤销
redo():回退

##  改造simplemde支持粘贴和拖拽上传 
```

var simplemde = new SimpleMDE({ element: document.getElementById("MyID") });
simplemde.codemirror.on("drop", function(editor,e){
    ...
});
simplemde.codemirror.on("paste",function(editor,e){
...
});

```
##  Blob对象转File对象 
为了使用WebUploader这个文件上传组件，需要将粘贴得到的Blob对象转为File对象。
参考[这里](http://stackoverflow.com/questions/27251953/how-to-create-file-object-from-blob)
Blob 对象是包含有只读原始数据的类文件对象.File 接口基于 Blob，继承了 Blob 的功能,并且扩展支持用户计算机上的本地文件。
```

var blob=new Blob();
var file = new File([blob], "image.png", {type:"image/png"});

```
File构造器的第一个参数必须是数组

##  WebUploader文件上传 
http://fex.baidu.com/webuploader/getting-started.html
创建Uploader对象
```

var uploader = WebUploader.Uploader({
    swf: 'path_of_swf/Uploader.swf',

    // 开起分片上传。
    chunked: true
});

```
监听fileQueued事件来实现进度UI构造:
```

// 当有文件被添加进队列的时候
uploader.on( 'fileQueued', function( file ) {
    var $list=$("#list");
    $list.append( '<div id="' + file.id + '" class="item">' +
        '<h4 class="info">' + file.name + '</h4>' +
        '<p class="state">等待上传...</p>' +
    '</div>' );
});

```
文件上传进度:
```

// 文件上传过程中创建进度条实时显示。
uploader.on( 'uploadProgress', function( file, percentage ) {
    var $li = $( '#'+file.id ),
    $percent = $li.find('.progress .progress-bar');
    // 避免重复创建
    if ( !$percent.length ) {
        $percent = $('<div class="progress progress-striped active">' +
          '<div class="progress-bar" role="progressbar" style="width: 0%">' +
          '</div>' +
        '</div>').appendTo( $li ).find('.progress-bar');
    }

    $li.find('p.state').text('上传中');
    $percent.css( 'width', percentage * 100 + '%' );
});

```
文件成功、失败处理:
```

uploader.on( 'uploadSuccess', function( file,data ) {
    $( '#'+file.id ).find('p.state').text('已上传');
});

uploader.on( 'uploadError', function( file ) {
    $( '#'+file.id ).find('p.state').text('上传出错');
});

uploader.on( 'uploadComplete', function( file ) {
    $( '#'+file.id ).find('.progress').fadeOut();
});

```
添加文件到队列并上传:
uploader.addFiles( file ) ⇒ undefined
uploader.addFiles( [file1, file2 ...] ) ⇒ undefined
开始上传:
uploader.upload() ⇒ undefined
uploader.upload( file | fileId) ⇒ undefined

其他参考：
[js获取剪切板内容，js控制图片粘贴](https://segmentfault.com/a/1190000004288686)
[在线代码编辑器 CODEMIRROR 事件说明](http://www.hyjiacan.com/codemirror-event/)
[javascript.ruanyifeng.com](http://javascript.ruanyifeng.com/htmlapi/file.html)
https://developer.mozilla.org/zh-CN/docs/Web/API/Blob