title: js阻止页面离开或刷新 

#  js阻止页面离开或刷新 
```

		//初始化关闭
        	window.addEventListener("beforeunload", function (e) {
      		  var confirmationMessage = "要记得保存！你确定要离开我吗？";
      		  (e || window.event).returnValue = confirmationMessage;     // 兼容 Gecko + IE
      		  return confirmationMessage;     // 兼容 Gecko + Webkit, Safari, Chrome
        	});

```