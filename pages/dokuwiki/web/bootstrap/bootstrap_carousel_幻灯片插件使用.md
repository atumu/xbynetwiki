title: bootstrap_carousel_幻灯片插件使用 

#  bootstrap carousel 幻灯片插件使用 
```

<style type="text/css">
  解决宽度问题
 .carousel{
width:600px !important;
}
解决高度问题
.carousel-inner{
  width:auto !important;
  max-height: 400px !important;
  
}
解决图片居中问题
.carousel-inner img {
  margin: auto;
}
</style>


<div id="carousel-example-generic" class="carousel slide" data-ride="carousel">
  <!-- Indicators -->
  <ol class="carousel-indicators">
    <li data-target="#carousel-example-generic" data-slide-to="0" class="active"></li>
    <li data-target="#carousel-example-generic" data-slide-to="1"></li>
    <li data-target="#carousel-example-generic" data-slide-to="2"></li>
  </ol>

  <!-- Wrapper for slides -->
  <div class="carousel-inner" role="listbox">
    <div class="item active">
      <img src="images/baidutts1.png" style="width:auto;height:500px !important" alt="...">
      <div class="carousel-caption">
        ...
      </div>
    </div>
 <div class="item">
      <img src="images/baidutts2.png" style="width:auto;height:500px !important" alt="...">
      <div class="carousel-caption">
        ...
      </div>
    </div>
     <div class="item">
      <img src="images/baidutts3.png" style="width:auto;height:500px !important" alt="...">
      <div class="carousel-caption">
        ...
      </div>
    </div>
     <div class="item">
      <img src="images/baidutts4.png" style="width:auto;height:500px !important" alt="...">
      <div class="carousel-caption">
        ...
      </div>
    </div>
     <div class="item">
      <img src="images/baidutts5.png" style="width:auto;height:500px !important" alt="...">
      <div class="carousel-caption">
        ...
      </div>
    </div>
  </div>

  <!-- Controls -->
  <a class="left carousel-control" href="#carousel-example-generic" role="button" data-slide="prev">
    <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
    <span class="sr-only">Previous</span>
  </a>
  <a class="right carousel-control" href="#carousel-example-generic" role="button" data-slide="next">
    <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
    <span class="sr-only">Next</span>
  </a>
</div>

```