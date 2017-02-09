title: css_div宽度随内部img的宽度 

#  css设置div宽度随内部img的宽度 
```

   <div class="test-div img-responsive img-rounded">
         <img  src="http://ccm.ddcdn.com/ext/photo-s/01/15/b7/80/caption.jpg" alt="First slide">
         <div class="title">标题1111111 1</div>
   </div>

```
```

.test-div{
	position: relative;
    	overflow: hidden;
	display:inline-block; /**最关键的一句,让div宽度随内部img的宽度*/
  	/**width: 100%; */
}
.test-div img {
	width: 100%;
}
.title{
    position: absolute;
    z-index: 3;
    bottom: 0;
    left: 0;
    overflow: hidden;
    width: 100%;
    height: 30px;
    text-align:center;
    color:white;
    background-color: rgba(255, 255, 255, 0.28);
}
 img:hover{ //鼠标悬浮放大效果
      	    transition: all 1s ease-in-out;
   	    -moz-transition: all 1s ease-in-out;
   	     -o-transition: all 1s ease-in-out;
   	     -webkit-transition: all 1s ease-in-out;
	    transform:scale(1.2,1.2);//1.2指的是X或Y轴的放大的倍数
   	    -webkit-transform:scale(1.2,1.2);
   	    -moz-transform:scale(1.2,1.2);
     	-o-transform:scale(1.2,1.2);
	}
img{ //鼠标移出恢复效果
	transition: all 1s ease-in-out;
   	    -moz-transition: all 1s ease-in-out;
   	     -o-transition: all 1s ease-in-out;
   	     -webkit-transition: all 1s ease-in-out;
	    transform:scale(1.0,1.0);
   	    -webkit-transform:scale(1.0,1.0);
   	    -moz-transform:scale(1.0,1.0);
     	-o-transform:scale(1.0,1.0);
}

```         