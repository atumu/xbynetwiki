title: html5_video_src动态修改 

#  HTML5 video src动态修改失效问题解决 
由于浏览器在加载DOM的时候就会请求video的src，然后对video进行渲染，之后才完成加载。**理解这个顺序特别重要，浏览器video的src请求是在DOM加载完全之前。**
完成加载之后，我们动态修改src属性，不会再导致浏览器对video进行渲染,只是替换了一个src值。所以导致修改失效。
这个问题其实可以换一种思路解决，考虑到如果我们在HTML中动态添加html标签内容后就会对局部的添加进行重新渲染.所以我能可以采取先情况容器内容，然后动态append进去，这样就可以让其重新加载渲染video元素及其内容了。如此问题便得以解决。
```

<div id="news-video-show" >
 <video controls='' class='uk-responsive-width' width='600' height='350'  id='news-video'>
   <source src=""  type='video/mp4'/>
  </video>
</div>

```
```

var _data="http://www.xby1993.net/abc.mp4"
$("#news-video-show").empty();
var a = "<video controls='' class='uk-responsive-width' width='600' height='350'  id='news-video'><source src=" + _data+ "  type='video/mp4'/><video>";
$("#news-video-show").append(a);

```
考虑到img可以动态修改src能够进行渲染，而video,audio却不能在动态修改src后渲染，是否是个BUG呢？