title: phplearn08 

#  PHP学习之SMTPEmail 
https://packagist.org/packages/pear/mail
http://blog.csdn.net/sadfishsc/article/details/7379948
http://email.about.com/od/emailprogrammingtips/qt/PHP_Email_SMTP_Authentication.htm
http://stackoverflow.com/questions/2284468/problem-with-php-pear-mail
https://packagist.org/packages/pear/net_smtp
http://php.net/manual/en/migration56.openssl.php
pear/main:https://github.com/pear/Mail
PHP mail()与SMTP验证
缺乏灵活性是PHP的mail()函数显得过于简单的部分原因。最重要而且令人沮丧的是，死板的mail()函数通常还不允许你使用你选择的SMTP服务器，并且它也根本不支持如今已被众多邮件服务器采用的SMTP验证。
幸运的是，克服PHP本身的缺陷既不困难，也不麻烦，更不痛苦。对于大多数情况下的邮件应用，**免费的PEAR Mail包足够提供全部的所需功能与灵活性**，并且它也能够与你期望的外部邮件服务器进行验证。在提高安全性的方面，它也支持SSL连接。
通过SMTP验证在PHP脚本中发送邮件
在PHP脚本中通过SMTP验证连接外部SMTP服务器并且发送邮件的方法如下：
○  **确保PEAR Mail包已经安装。** composer安装:
```

composer require pear/mail pear/net_smtp

```
Ø  通常，它已经安装到了PHP之中，尤其在PHP 4及以后的版本中。放手一试吧。
○  根据你的需求改写后面的示例。确保你至少改变了以下这些变量：
Ø  from：邮件发送方的email地址。
Ø  to：邮件接收方的email地址。
Ø  host：外部SMTP服务器的地址。
Ø  username：SMTP验证的用户名（通常与发送邮箱的用户名相同）。
Ø  password：SMTP验证的密码。
##  通过SMTP验证从PHP发送邮件的示例(测试成功) 
```

require_once "Mail.php";  
   
 $from = "Sandra Sender <sender@example.com>";  
 $to = "Ramona Recipient <recipient@example.com>";  
 $subject = "Hi!";  
 $body = "Hi,\n\nHow are you?";  
   
 $host = "mail.example.com";  
 $username = "smtp_username";  
 $password = "smtp_password";  
   
 $headers = array ('From' => $from,  
   'To' => $to,  
   'Subject' => $subject);  
 $smtp = Mail::factory('smtp',  
   array ('host' => $host,  
     'auth' => true,
    // 'debug'=>true,
     'username' => $username,  
     'password' => $password));  
   
 $mail = $smtp->send($to, $headers, $body);  
   
 if (PEAR::isError($mail)) {  
   echo("<p>" . $mail->getMessage() . "</p>");  
  } else {  
   echo("<p>Message successfully sent!</p>");  
  }  
 ?>  

```
##  通过SMTP验证和SSL加密从PHP发送邮件的示例（测试没有成功） 
```

<?php  
 require_once "Mail.php";  
   
 $from = "Sandra Sender <sender@example.com>";  
 $to = "Ramona Recipient <recipient@example.com>";  
 $subject = "Hi!";  
 $body = "Hi,\n\nHow are you?";  
   
 $host = "ssl://mail.example.com";  
 $port = "465";  
 $username = "smtp_username";  
 $password = "smtp_password";  
   
 $headers = array ('From' => $from,  
   'To' => $to,  
   'Subject' => $subject);  
 $smtp = Mail::factory('smtp',  
   array ('host' => $host,  
     'port' => $port,  
     'auth' => true,  
     'username' => $username,  
     'password' => $password));  
   
 $mail = $smtp->send($to, $headers, $body);  
   
 if (PEAR::isError($mail)) {  
   echo("<p>" . $mail->getMessage() . "</p>");  
  } else {  
   echo("<p>Message successfully sent!</p>");  
  }  
 ?>  

```

译后补充：
1. 上面的示例运行中会出现如下这种错误：
Strict Standards: Non-static method …
其原因是PEAR Mail包中的有些实现没有按照严格的PHP语法来写，尤其是这样静态函数的调用。这些错误信息是在PHP解释过程中产生的，并不影响运行的结果。
解决的方法是在php.ini文件中将 error_reporting 的 E_STRICT 去掉，改为 error_reporting=E_ALL，重启Apache服务器即可。
2.Mail::Factory 静态函数的第二个参数数组中还可以包含SMTP服务器的端口号port、本地服务器地址localhost、超时timeout等数据。
3. 在本人测试的PHP5.3.2版本中，包括Mail在内的PEAR已经安装到了PHP的路径下，在php/PEAR目录之中。在这里能够找到Mail的主文件Mail.php以及相关的文件夹Mail。其中的文件与从PEAR官方下载的Mail包相差无几。
在调用时，可以直接 require_once(“Mail.php”) 就能引用到 PHP/PEAR/Mail.php 文件，而不再需要在这些文件放到项目目录下。
