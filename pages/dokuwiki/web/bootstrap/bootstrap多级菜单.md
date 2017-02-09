title: bootstrap多级菜单 

#   bootstrap多级菜单 
项目地址：https://github.com/vsn4ik/bootstrap-submenu
demo:http://vsn4ik.github.io/bootstrap-submenu/#examples
**步骤：**
1、导入：
<script src="bootstrap-submenu.min.js"></script>
<link rel="stylesheet" href="bootstrap-submenu.min.css"></script>
2、` 启用Bootstrap-submenu: `
<script>
$(document).ready(function(){
 $('.dropdown-submenu > a').submenupicker();
});
</script>
3、使用：
<li class="dropdown">
<a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">学习<span class="caret"></span></a>
<ul class="dropdown-menu" role="menu">
<li class="` dropdown-submenu `" ><a ` data-toggle="dropdown" ` tabindex="0">Android</a>
<ul class="dropdown-menu">
<li><a href="http://wiki.xby1993.net/doku.php?id=android">Android</a></li>
<li><a href="http://wiki.xby1993.net/doku.php?id=greendao">GreenDao</a></li>
</ul>
` </li> `
效果:
<html>
<iframe style="width:100%;height:500px;" src="http://www.xby1993.net"></iframe>
</html>