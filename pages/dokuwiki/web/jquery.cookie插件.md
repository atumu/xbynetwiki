title: jquery.cookie插件 

#  jQuery.Cookie插件 
Cookie操作插件 jQuery.Cookie
官网：https://github.com/carhartl/jquery-cookie
jQuery操作cookie的插件,大概的使用方法如下
```

$.cookie('the_cookie'); //读取Cookie值
$.cookie(’the_cookie’, ‘the_value’); //设置cookie的值
$.cookie(’the_cookie’, ‘the_value’, {expires: 7, path: ‘/’, domain: ‘jquery.com’, secure: true});//新建一个cookie 包括有效期 路径 域名等
$.cookie(’the_cookie’, ‘the_value’); //新建cookie
$.cookie(’the_cookie’, null); //删除一个cookie

```
