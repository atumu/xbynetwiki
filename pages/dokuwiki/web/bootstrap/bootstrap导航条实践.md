title: bootstrap导航条实践 

#  bootstrap导航条实践 
```

   <header class="navbar navbar-inverse navbar-static-top" id="top" role="banner">
    <div class="navbar-header">
      <button class="navbar-toggle collapsed" type="button" data-toggle="collapse" data-target="nav">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
    </div>
  <nav class="collapse navbar-collapse  ">
      <ul class="nav navbar-nav">
        <li class="active"><a href="#">首页</a></li>
        <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">下载 <span class="caret"></span></a>
          <ul class="dropdown-menu" role="menu">
            <li><a href="#baidutts">资讯听读器</a></li>
            <li class="divider"></li>
          </ul>
        </li>
      </ul>
</nav>
</header>

```