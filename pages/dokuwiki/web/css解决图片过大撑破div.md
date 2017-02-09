title: css解决图片过大撑破div 

#  css解决图片过大撑破DIV 
```

.newsImg-thumb{
	text-align:center;
	height: 100%;
}
.newsImg-thumb img{
	display: inline-block;
	max-height: 100%;
}
<div style="height: 222px;>
	<div class="newsImg-thumb img-responsive img-rounded">
		<img src="/resource//uploads/images/20160125/29f53a911f3e4066ba65689640ef5a41.jpg" alt="缩略图">
	</div>
</div>

```
或
```

.newsImg-thumb{
	text-align:center;
	height: 100%;
}
.newsImg-thumb img{
	display: inline-block;
	max-height: 100%;
}
<div style="height: 222px;>
	<div class="newsImg-thumb ">
		<img class="img-responsive img-rounded" src="/resource//uploads/images/20160125/29f53a911f3e4066ba65689640ef5a41.jpg" alt="缩略图">
	</div>
</div>

```

参考：http://www.divcss5.com/wenji/w364.shtml
http://zhidao.baidu.com/link?url=NX6Pmo_3-w8wL3e_qA1mjkMBq6tVNeFF4e8PmPC8X-El5Y5DWQelD-fr1yc6I2It_7l6uv4KBLOlNqXx-qRb0_