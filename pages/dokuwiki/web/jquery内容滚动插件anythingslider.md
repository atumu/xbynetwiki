title: jquery内容滚动插件anythingslider 

#  jQuery内容滚动插件AnythingSlider 
官网:https://css-tricks.com/anythingslider-jquery-plugin/
Wiki:https://github.com/CSS-Tricks/AnythingSlider/wiki
demo:http://css-tricks.github.com/AnythingSlider/
参数列表：http://css-tricks.github.io/AnythingSlider/#&panel1-1&panel2-1
引入head:
```

<script src="http://cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
  <!-- FlexSlider -->
  <script defer src="jquery.anythingslider.min.js"></script>
  <link rel="stylesheet" href="anythingslider.css"  />

  
<!-- Anything Slider optional plugins -->
<script src="jquery.easing.1.2.js"></script>
<script src="swfobject.js"></script>

<style>
#slider{ width: 100%;  margin: 0 auto; }
</style>
 <!-- AnythingSlider initialization -->
<script>
// DOM Ready
$(function(){
			
	$('#slider').anythingSlider({
			
		resizeContents : true, 
		easing : 'easeInOutBack',
		//toggleArrows : true,
		//toggleControls  : true,
		autoPlay : true});
		
});
</script>

```
编辑HTML:
```


<ul id="slider">

<li><img src="demos/images/slide-civil-1.jpg" alt=""></li>

<li><img src="demos/images/slide-env-1.jpg" alt=""></li>

<li><img src="demos/images/slide-civil-2.jpg" alt=""></li>

<li><img src="demos/images/slide-env-2.jpg" alt=""></li>

</ul>

```