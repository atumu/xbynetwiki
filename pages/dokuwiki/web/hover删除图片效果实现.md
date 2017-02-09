title: hover删除图片效果实现 

#  hover删除图片效果实现 
```

<div id="thumbCloseDiv" class="col-sm-12 content-close">
	<img class="close-image" src="/static/core/common/images/close.png" style="display:none;"/>
	<img id="thumbImg" class="col-sm-12" src="" alt="" style="display:none;margin: 0;padding: 0;max-height: 200px;"/> 
</div>

```
```

.close-image{
    display: block;
    float:right;
    position: absolute;
    top:-10px;
    right: -10px;
    width: 20px;
    height: 20px;
    cursor: pointer;
}
.close-image{
	z-index: 3;
}
.content-close{
	padding: 0;
	margin: 0;
}
.content-close:hover{
	border: solid rgba(0, 0, 0, 0.13);
	
}

```

```

    function thumbHoverClose(){
        //图片hover_close
         $("#thumbCloseDiv").hover(function(){
            if($("#thumbImg").attr("src")){
                $(".close-image").show();
            }
         },function(){
            $(".close-image").hide();
         });
         $(".close-image").click(function() {
            $("#thumbImg").attr("src","");
            $("#articleThumbnailId").val("");
            $("#thumbImg").hide();
            $(".close-image").hide();
        });
    }


```