title: ajax调用的全局错误处理 

#  Ajax调用的全局错误处理 
当某次 Ajax 调用返回 404 或 500 错误，就会执行错误处理。**但如果没有定义该处理，其他 jQuery 代码或许会停止工作。**可以通过下面这段代码定义一个全局 Ajax 错误处理：
```

$(document).ajaxError(function (e, xhr, settings, error) {
  console.log(error);
});

```