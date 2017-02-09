title: phplearn8 

#  PHP学习之Email 
##  环境配置 
PHP使用mail()函数发送电子邮件，但是需要配置sendmail来发送，以windows下为例:
1> 在sendmail下载文件http://glob.com.au/sendmail/
2> 按页面描述把解压后的文件放入相应文件夹.
3> 参考http://flowingmotion.jojordan.org/2012/04/26/how-to-set-up-email-with-wamp/ 配置sendmail.ini和php.ini

php.ini配置
```

[mail function] 
sendmail_path = "D:\xampp\sendmail\sendmail.exe -t"

```

sendmail.ini配置 :
```

[sendmail]
smtp_server=smtp.gmail.com
smtp_port=587
smtp_ssl=tls
auth_username=youremailaddresses@gmail.com
auth_password=youremailpassword

```
  
##  mail()函数 
PHP 允许您从脚本直接发送电子邮件。
PHP **mail() 函数**用于从脚本中发送电子邮件。
语法
**mail(to,subject,message,headers,parameters)**
参数	描述
  * to	必需。规定 email 接收者。
  * subject	必需。规定 email 的主题。注释：该参数不能包含任何新行字符。
  * message	必需。定义要发送的消息。应使用 LF (\n) 来分隔各行。
  * headers	可选。规定附加的标题，比如 From、Cc 以及 Bcc。应当使用 CRLF (\r\n) 分隔附加的标题。
  * parameters	可选。对邮件发送程序规定额外的参数。
注释：` PHP 需要一个已安装且正在运行的邮件系统，以便使邮件函数可用。所用的程序通过在 php.ini 文件中的配置设置进行定义。 `请在我们的 PHP Mail 参考手册阅读更多内容。
**邮件函数的行为受 ` php.ini ` 的影响。**
名称	默认	描述	可更改
  * SMTP	"localhost"	Windows 专用：SMTP 服务器的 DNS 名称或 IP 地址。	PHP_INI_ALL
  * smtp_port	"25"	Windows 专用：SMTP 端口号。自 PHP 4.3 起可用。	PHP_INI_ALL
  * sendmail_from	NULL	Windows 专用：规定从 PHP 发送的邮件中使用的 "from" 地址。	PHP_INI_ALL
  * sendmail_path	NULL	Unix 系统专用：规定sendmail 程序的路径（通常 /usr/sbin/sendmail 或 /usr/lib/sendmail）	PHP_INI_SYSTEM

##  PHP 简易 E-Mail 
通过 PHP 发送电子邮件的最简单的方式是发送一封文本 email。
在下面的例子中，我们首先声明变量($to, $subject, $message, $from, $headers)，然后我们在 mail() 函数中使用这些变量来发送了一封 e-mail：
```

<?php
$to = "someone@example.com";
$subject = "Test mail";
$message = "Hello! This is a simple email message.";
$from = "someonelse@example.com";
$headers = "From: $from";
mail($to,$subject,$message,$headers);
echo "Mail Sent.";
?>

```
PHP Mail Form
通过 PHP，您能够在自己的站点制作一个反馈表单。下面的例子向指定的 e-mail 地址发送了一条文本消息：
```

<?php
if (isset($_REQUEST['email']))
//if "email" is filled out, send email
  {
  //send email
  $email = $_REQUEST['email'] ; 
  $subject = $_REQUEST['subject'] ;
  $message = $_REQUEST['message'] ;
  mail( "someone@example.com", "Subject: $subject",
  $message, "From: $email" );
  echo "Thank you for using our mail form";
  }
else
//if "email" is not filled out, display the form
  {
  echo "<form method='post' action='mailform.php'>
  Email: <input name='email' type='text' /><br />
  Subject: <input name='subject' type='text' /><br />
  Message:<br />
  <textarea name='message' rows='15' cols='40'>
  </textarea><br />
  <input type='submit' />
  </form>";
  }
?>

```
例子解释：
首先，检查是否填写了邮件输入框
如果未填写（比如在页面被首次访问时），输出 HTML 表单
如果已填写（在表单被填写后），从表单发送邮件
当点击提交按钮后，重新载入页面，显示邮件发送成功的消息

##  防止 E-mail 注入 
防止 e-mail 注入的最好方法是对输入进行验证。
下面的代码与上一节类似，不过我们已经增加了检测表单中 email 字段的输入验证程序：
```

<html>
<body>
<?php
function spamcheck($field)
  {
  //filter_var() sanitizes the e-mail 
  //address using FILTER_SANITIZE_EMAIL
  $field=filter_var($field, FILTER_SANITIZE_EMAIL);
  
  //filter_var() validates the e-mail
  //address using FILTER_VALIDATE_EMAIL
  if(filter_var($field, FILTER_VALIDATE_EMAIL))
    {
    return TRUE;
    }
  else
    {
    return FALSE;
    }
  }

if (isset($_REQUEST['email']))
  {//if "email" is filled out, proceed

  //check if the email address is invalid
  $mailcheck = spamcheck($_REQUEST['email']);
  if ($mailcheck==FALSE)
    {
    echo "Invalid input";
    }
  else
    {//send email
    $email = $_REQUEST['email'] ; 
    $subject = $_REQUEST['subject'] ;
    $message = $_REQUEST['message'] ;
    mail("someone@example.com", "Subject: $subject",
    $message, "From: $email" );
    echo "Thank you for using our mail form";
    }
  }
else
  {//if "email" is not filled out, display the form
  echo "<form method='post' action='mailform.php'>
  Email: <input name='email' type='text' /><br />
  Subject: <input name='subject' type='text' /><br />
  Message:<br />
  <textarea name='message' rows='15' cols='40'>
  </textarea><br />
  <input type='submit' />
  </form>";
  }
?>

</body>
</html>

```
在上面的代码中，我们使用了** PHP 过滤器**来对输入进行验证：
  * FILTER_SANITIZE_EMAIL 从字符串中删除电子邮件的非法字符
  * FILTER_VALIDATE_EMAIL 验证电子邮件地址

