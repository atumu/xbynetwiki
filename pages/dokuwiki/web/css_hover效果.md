title: css_hover效果 

#  css hover效果 
##  效果一 
![](/data/dokuwiki/web/pasted/20151223-172131.png)
```

.pic-article-main:hover {
	border: 1px solid #ccc;
	box-shadow: 0 0 6px #888888;
	transition: all .4s;
	-moz-transition: all .4s;	/* Firefox 4 */
	-webkit-transition: all .4s;	/* Safari 和 Chrome */
	-o-transition: all .4s;
}

```
##  css按钮hover特效 
```

btn:hover{
	box-shadow:0 2px 2px rgba(0,0,0,.2),0 6px 10px rgba(0,0,0,.3);
	-webkit-transition:all .3s linear;
	-moz-transition:all .3s linear;
	-o-transition:all .3s linear;
	transition:all .3s linear
}

```