title: phplearn7 

#  PHP学习之Session与Cookie 
##  Cookies 
什么是 Cookie？
cookie 常用于识别用户。cookie 是服务器留在用户计算机中的小文件。每当相同的计算机通过浏览器请求页面时，它同时会发送 cookie。通过 PHP，您能够创建并取回 cookie 的值。
如何创建 cookie？
` setcookie() 函数 `用于设置 cookie。
bool setcookie ( string $name [, string $value = "" [, int $expire = 0 [, string $path = "" [, string $domain = "" [, bool $secure = false [, bool $httponly = false ]]]]]] )
**setcookie(name, value, expire, path, domain);**
在下面的例子中，我们将创建名为 "user" 的 cookie，把为它赋值 "Alex Porter"。我们也规定了此 cookie 在一小时后过期：
```

<?php 
  //注释：setcookie() 函数必须位于 <html> 标签之前。
setcookie("user", "Alex Porter", time()+3600);
?>
<html>
<body>
</body>
</html>

```
` 注释：setcookie() 函数必须位于 < html> 标签之前。 `
注释：在发送 cookie 时，cookie 的值会自动进行 URL 编码，在取回时进行自动解码（为防止 URL 编码，请使用 ` setrawcookie() ` 取而代之）。

如何取回 Cookie 的值？
PHP 的`  $_COOKIE 变量 `用于取回 cookie 的值。
在下面的例子中，我们取回了名为 "user" 的 cookie 的值，并把它显示在了页面上：
```

<?php
// Print a cookie
echo $_COOKIE["user"];

// A way to view all cookies
print_r($_COOKIE);
?>

```
在下面的例子中，我们使用 ` isset() 函数 `来确认是否已设置了 cookie：
```

<html>
<body>
<?php
if (isset($_COOKIE["user"]))
  echo "Welcome " . $_COOKIE["user"] . "!<br />";
else
  echo "Welcome guest!<br />";
?>
</body>
</html

```
如何删除 cookie？
当删除 cookie 时，您应当使过期日期变更为过去的时间点。
删除的例子：
```

<?php 
// set the expiration date to one hour ago
setcookie("user", "", time()-1);
?>

```

##  Session 
php.ini配置会话控制：
![](/data/dokuwiki/php/pasted/20160411-001036.png)
![](/data/dokuwiki/php/pasted/20160411-001050.png)
Session 的工作机制是：为每个访问者创建一个唯一的 id (UID)，并基于这个 UID 来存储变量。UID 存储在 cookie 中，亦或通过 URL 进行传导。
**SESSIONID的名字默认为PHPSESSID**(如java的为JSESSIONID)
开始 PHP Session
在您把用户信息存储到 PHP session 中之前，**首先必须启动会话**。
` 注释：session_start() 函数必须位于 < html> 标签之前： `
```

<?php session_start(); ?>
<html>
<body>

</body>
</html>

```
上面的代码会向服务器注册用户的会话，以便您可以开始保存用户信息，同时会为用户会话分配一个 UID。
存储 Session 变量
存储和取回 session 变量的正确方法是使用 PHP ` $_SESSION 变量 `：
在下面的例子中，我们创建了一个简单的 page-view 计数器。isset() 函数检测是否已设置 "views" 变量。如果已设置 "views" 变量，我们累加计数器。如果 "views" 不存在，则我们创建 "views" 变量，并把它设置为 1：
```

<?php
session_start();

if(isset($_SESSION['views']))
  $_SESSION['views']=$_SESSION['views']+1;

else
  $_SESSION['views']=1;
echo "Views=". $_SESSION['views'];
?>

```
**终结 Session**
如果您希望删除某些 session 数据，可以使用`  unset() 或 session_destroy() 函数 `。
unset() 函数用于释放指定的 session 变量：
```

<?php
unset($_SESSION['views']);
?>

```
您也可以通过 session_destroy() 函数彻底终结 session：
```

<?php
session_destroy();
?>

```
注释：session_destroy() 将重置 session，您将失去所有已存储的 session 数据。

###  示例 
制作一个简单的登录程序
本程序分为4个页面 login.php checklogin.php function.php login_out.php
**login.php**
```

<? 
 session_start();//开启会话  
 $username=$_SESSION['username'];//从会话中读取用户名  
 $password=$_SESSION['password'];//从会话中读取密码  
?> 
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> 
<html xmlns="http://www.w3.org/1999/xhtml"> 
<head> 
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> 
<title>用户登录</title> 
</head> 
<!--用JS验证表单是否为空--> 
<script language="javascript"> 
 function check_login()  
 {  
     var f=document.form1;  
     var username=f.username.value;  
     var password=f.password.value;  
     if(username=="")  
     {  
         alert("请输入用户名");  
         f.username.focus();//让焦点回到文本域  
         return false;//中断程序  
     }  
     if(password=="")  
     {  
         alert("请输入密码");  
         f.password.focus();  
         return false;  
     }  
     return true;  
}  
</script> 
<body> 
<? 
 if($username==`  and $password== `)//判断会话中用户名和密码是否为空  
 {  
?> 
<form id="form1" name="form1" method="post" action="checklogin.php" onsubmit="return check_login()"> 
  <table width="275" border="0" align="center"> 
    <tr align="center"> 
      <td colspan="2">用户登录</td> 
    </tr> 
    <tr> 
      <td width="71">用户名：</td> 
      <td width="194" height="22"><label for="textfield"></label> 
      <input type="text" name="username" id="textfield" /></td> 
    </tr> 
    <tr> 
      <td>密码：</td> 
      <td height="22"><label for="textfield2"></label> 
      <input type="text" name="password" id="textfield2" /></td> 
    </tr> 
    <tr> 
      <td>&nbsp;</td> 
      <td><input type="submit" name="button" id="button" value="提交" /> 
      <input type="reset" name="button2" id="button2" value="重置" /> 
      <input name="action" type="hidden" id="action" value="login" /></td> 
    </tr> 
  </table> 
    
<? } else {?> 
</form> 
<table width="275" border="0" align="center"> 
    <tr align="center"> 
      <td colspan="2">用户信息</td> 
    </tr> 
    <tr> 
      <td width="71">用户名：</td> 
      <td width="194" height="22"><? echo $username;?> </td> 
    </tr> 
    <tr> 
      <td>密码：</td> 
      <td height="22"><? echo $password; ?></td> 
    </tr> 
    <tr> 
      <td>&nbsp;</td> 
      <td>进入管理页面  
          <a href="login_out.php">退出登录</a></td> 
    </tr> 
  </table> 
  <? }?> 
</body> 
</html> 

```
**checklogin.php**
```

<? 
session_start();//开启会话  
header("Content-Type:text/html; charset=utf-8");  
include('function.php');  
$action=$_REQUEST['action'];  
$username=$_POST['username'];  
$password=$_POST['password'];  
if ($action=='login')//防恶意登录 隐藏域的值是否为login  
{  
    if($username=='prometheus' and $password=='sm520517')//判断会话中用户名和密码  
    {  
          
        $_SESSION['username']=$username;  
        $_SESSION['password']=$password;  
        msg_box('恭喜你，登录成功','login.php');  
    }  
    else  
    {  
        history_box('对不起，你输入的用户名或密码错误');  
          
    }  
}  
else  
{  
    seturl('login.php');  
      
}  
?> 

```
**function.php**
```

<? 
//提示文本  
function msg_box($text,$url) //声明提示框函数  
{  
    $msg="<script>alert('$text');location.href='$url';</script>";  
    echo $msg;  
}  
//后退上一步  
function history_box($text)  
{  
    $msg="<script>alert('$text');history.go(-1);</script>";  
    echo $msg;  
}  
//直接跳转页面  
function seturl($url)  
{  
    $msg="<script>location.href='$url';</script>";  
    echo $msg;  
}  
//注销登录  
function dellogin($text,$url)  
{  
    $msg="<script>alert('$text');location.href='$url';</script>";  
    echo $msg;  
}  
?> 

```
**login_out.php**
```

<? 
session_start();  
header("Content-Type:text/html; charset=utf-8");  
include('function.php');  
session_destroy();  
dellogin('您已成功退出登录','login.php');  
?> 

```
由这四个页面组成了完整的登录程序
注意：1.php与HTML包含的写法
2.script中的写法

http://phpmylove.blog.51cto.com/3389265/641345