title: jquery.cleditor 

#  jquery.CLEditor小巧简单的编辑器 
官网：http://premiumsoftware.net/cleditor/
安装：
  * jquery.cleditor.min.js - jQuery Plug-in (minified)
  * jquery.cleditor.js - jQuery Plug-in (source)
  * jquery.cleditor.css - Style Sheet
  * images/buttons.gif - Toolbar Button Image Strip
  * images/toolbar.gif - Toolbar Background Image
```

<html>
<head>
    <link rel="stylesheet" href="jquery.cleditor.css" />
    <script src="jquery.min.js"></script>
    <script src="jquery.cleditor.min.js"></script>
    <script>
        $(document).ready(function () { 
        	 $("#input").cleditor({
                width: 500, // width not including margins, borders or padding
                height: 250, // height not including margins, borders or padding
                controls: // controls to add to the toolbar
                    "bold italic underline strikethrough subscript superscript | font size " +
                    "style | color highlight removeformat | bullets numbering | outdent " +
                    "indent | alignleft center alignright justify | undo redo | " +
                    "rule image link unlink | cut copy paste pastetext | print source"
                
        });
          
      	$("#input").cleditor({width:500, height:250});
	$("textarea").cleditor();
	var editor = $("#input").cleditor()[0];
	var $editors = $("textarea").cleditor();
    </script>
</head>
<body>
    <textarea id="input" name="input"></textarea>
</body>
</html>

```
##  Callbacks 
  * ` .updateTextArea(html) ` - This handler is called by the .updateTextArea() method of the cleditor object **to convert HTML into source code**. This handler receives the internal iframe contents as the html parameter and should return the source code to store in the textarea.
  * ` .updateFrame(source) ` - This handler is called by the .updateFrame() method of the cleditor object to **convert source code into HTML**. This handler **receives the textarea contents** as the source parameter and should return the HTML to store in the internal iframe.
取值，很简单：$("#input").val()
赋值有点头痛，其实也很简单：
 var o = $("#input").cleditor()[0];
$("#input").val(“abcdefg”);
o.updateFrame();
注意：不要使用$("#input").html(“abcdefg”);原因是什么呢..你如果赋的值是带有样式的，如果使用html()之后，会有些问题...