title: jquery滚动监听刷新 

#  jquery滚动监听底部刷新 
modal-body 其中是overflow:scroll
```

					                $(".modal-body").has('.matImgs').scroll(function(){
					                	var $this=$(this);
					                	var viewH=$this.height(); //可视高度
					                	var contentH=$this.get(0).scrollHeight;//内容高度  
					                    var scrollTop= $this.scrollTop();//滚动高度  
					                    if((contentH-scrollTop)/(viewH+30)>=0.92){//到达底部时,加载新内容 
					                    	 _this.loadMore();
					                    }
					                });

```



```

$(window).scroll(function(){
			var factor=$(window).height()/($(document).height() -$(window).scrollTop());
			if(factor>0.9 &&curPage<maxPage){
}});

```