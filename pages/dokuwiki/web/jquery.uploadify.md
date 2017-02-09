title: jquery.uploadify 

#  jQuery.uploadify异步文件上传插件 
官网：http://www.uploadify.com/download/
```

<!DOCTYPE html>
<html>
<head>
    <title>My Uploadify Implementation</title>
    <link rel="stylesheet" type="text/css" href="uploadify.css">
    <script type="text/javascript" src="http://code.jquery.com/jquery-1.7.2.min.js"></script>
    <script type="text/javascript" src="jquery.uploadify-3.1.min.js"></script>
  <script type="text/javascript">
        $(function () {
            $("#uploadify").uploadify({
                //指定swf文件
                'swf': 'js/uploadify/uploadify.swf',
                //后台处理的页面
                'uploader': 'UploadHandler.ashx',
              //浏览按钮的宽度
       		 'width':'100',
       	     //浏览按钮的高度
        	'height':'32',
              	//cancelImage:'js/uploadify/uploadify-cancel.png',
              	//successTimeout:99999,//上传超时时间
                //按钮显示的文字
                'buttonText': '上传图片',
                //显示的高度和宽度，默认 height 30；width 120
                //'height': 15,
                //'width': 80,
                //上传文件的类型  默认为所有文件    'All Files'  ;  '*.*'
                //在浏览窗口底部的文件类型下拉菜单中显示的文本
                'fileTypeDesc': 'Image Files',
                //允许上传的文件后缀
                'fileTypeExts': '*.gif; *.jpg; *.png',
                //发送给后台的其他参数通过formData指定
                //'formData': { 'someKey': 'someValue', 'someOtherKey': 1 },
                //上传文件页面中，你想要用来作为文件队列的元素的id, 默认为false  自动生成,  不带#
                //'queueID': 'fileQueue',
                //选择文件后自动上传
                'auto': true,
                //设置为true将允许多文件上传
                'multi': true，
              	'queueSizeLimit'：9，//当允许多文件上传时，设置选择文件的个数，默认值为999， 
                //fileSizeLimit:'20MB',//上传文件最大值
                'method' : 'post',
             // 'fileObjName':'Filedata' //后台处理时文件的标志。需要与后台中file属性一样。
                //每次更新上载的文件的进展
                 onUploadProgress: function(file, bytesUploaded, bytesTotal, totalBytesUploaded, totalBytesTotal) {}
		  //上传到服务器，服务器返回相应信息到data里
                 onUploadSuccess:function(file, data, response){}
          	//选择上传文件后调用
        	'onSelect' : function(file) {},
                 //返回一个错误，选择文件的时候触发
        	'onSelectError':function(file, errorCode, errorMsg){}
          	 'onUploadStart' : function(file) {
         	  	 alert('Starting to upload ' + file.name);
        	  }
          	'onUploadComplete' : function(file) {
            		alert('The file ' + file.name + ' finished processing.');
        	}
            });
        });
    
    </script>
</head>
<body>
 <div>
        <%--用来作为文件队列区域--%>
        <div id="fileQueue">
        </div>
        <input type="file" name="uploadify" id="uploadify" />
        <p>
            <a href="javascript:$('#uploadify').uploadify('upload','*')">上传</a>| 
            <a href="javascript:$('#uploadify').uploadify('cancel')">取消上传</a>
           <a href="javascript:$('#uploadify').uploadify('stop')">停止上传</a>
        </p>
    </div>
</body>
</html>

```
##  方法调用 
$('#file_upload').uploadify('upload');
$('#file_upload').uplaodify('cancel','*');

##  服务端的书写 
以PHP为例：
```

$('#file_upload').uploadify({
    // Some options
    'method'   : 'post',
    'formData' : { 'someKey' : 'someValue' }
});

```
```

// Set $someVar to 'someValue'
$someVar = $_POST['someKey'];
// Set $someVar to 'someValue'
$someVar = $_POST['someKey'];

```
###  服务器返回数据形式 
```

$targetFolder = '/uploads'; // Relative to the root
if (!empty($_FILES)) {
	$tempFile = $_FILES['Filedata']['tmp_name'];
	$targetPath = $_SERVER['DOCUMENT_ROOT'] . $targetFolder;
	$targetFile = rtrim($targetPath,'/') . '/' . $_FILES['Filedata']['name'];

	// Validate the file type
	$fileTypes = array('jpg','jpeg','gif','png'); // File extensions
	$fileParts = pathinfo($_FILES['Filedata']['name']);

	if (in_array($fileParts['extension'],$fileTypes)) {
		move_uploaded_file($tempFile,$targetFile);
		echo $targetFolder . '/' . $_FILES['Filedata']['name'];
	} else {
		echo 'Invalid file type.';
	}
} 

```
```

$('#file_upload').uploadify({
    // Some options
    'onUploadSuccess' : function(file, data, response) {
        alert('The file was saved to: ' + data);
    }
});

```


参考：http://www.uploadify.com/documentation/
http://www.abc3210.com/2012/js_09/jquery-uploadify.shtml
http://www.cnblogs.com/babycool/archive/2012/08/04/2623137.html
http://zhanzhongchu.iteye.com/blog/2092449
http://blog.csdn.net/huangtao2011/article/details/25238655