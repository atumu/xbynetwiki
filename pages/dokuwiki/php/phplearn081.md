title: phplearn081 

#  PHP学习之SwiftMailer和PHPMailer 
##  PHPMailer 
PHPMailer是PHP界最为流行的邮件发送库.支持直接与 SMTP 服务器通讯。
github:https://github.com/PHPMailer/PHPMailer
安装:composer require phpmailer/phpmailer
特点:
  * 支持直接与 SMTP 服务器通讯。而无需本地mail-server
  * 在邮件中包含多个 TO、CC、BCC 和 REPLY-TO。
  * 平台应用广泛，支持的 SMTP 服务器包括 Sendmail、qmail、Postfix、Gmail、Imail、Exchange 等。
  * 支持嵌入图像，附件，HTML 邮件。
  * 可靠的强大的调试功能。
  * 支持 SMTP 认证。
  * 自定义邮件头。
  * 支持 8bit、base64、binary 和 quoted-printable 编码。
  * Used by many open-source projects: WordPress, Drupal, 1CRM, SugarCRM, Yii, Joomla! and many more
  * Multipart/alternative emails for mail clients that do not read HTML email
  * Support for UTF-8 content and 8bit, base64, binary, and quoted-printable encodings
  * SMTP authentication with LOGIN, PLAIN, NTLM, CRAM-MD5 and Google's XOAUTH2 mechanisms over SSL and TLS transports
  * DKIM and S/MIME signing support
```

<?php
require 'PHPMailerAutoload.php';

$mail = new PHPMailer;

//$mail->SMTPDebug = 3;                               // Enable verbose debug output

$mail->isSMTP();                                      // Set mailer to use SMTP
$mail->Host = 'smtp1.example.com;smtp2.example.com';  // Specify main and backup SMTP servers
$mail->SMTPAuth = true;                               // Enable SMTP authentication
$mail->Username = 'user@example.com';                 // SMTP username
$mail->Password = 'secret';                           // SMTP password
$mail->SMTPSecure = 'tls';                            // Enable TLS encryption, `ssl` also accepted
$mail->Port = 587;                                    // TCP port to connect to

$mail->setFrom('from@example.com', 'Mailer');
$mail->addAddress('joe@example.net', 'Joe User');     // Add a recipient
$mail->addAddress('ellen@example.com');               // Name is optional
$mail->addReplyTo('info@example.com', 'Information');
$mail->addCC('cc@example.com');
$mail->addBCC('bcc@example.com');

$mail->addAttachment('/var/tmp/file.tar.gz');         // Add attachments
$mail->addAttachment('/tmp/image.jpg', 'new.jpg');    // Optional name
$mail->isHTML(true);                                  // Set email format to HTML

$mail->Subject = 'Here is the subject';
$mail->Body    = 'This is the HTML message body <b>in bold!</b>';
$mail->AltBody = 'This is the body in plain text for non-HTML mail clients';

if(!$mail->send()) {
    echo 'Message could not be sent.';
    echo 'Mailer Error: ' . $mail->ErrorInfo;
} else {
    echo 'Message has been sent';
}

```
###  无法发送SSL加密的邮件原因与解决 
https://www.xssfox.com/2015-12-16/php5_6-failed-to-connect-to-server-0/
http://stackoverflow.com/questions/26827192/phpmailer-ssl3-get-server-certificatecertificate-verify-failed
https://github.com/PHPMailer/PHPMailer/wiki/Troubleshooting#php-56-certificate-verification-failure
http://php.net/manual/en/migration56.openssl.php
##  SwiftMailer 
Swift Mailer是一个PHP邮件发送类(类似的有PHPMailer,pear/mail)。它不依赖于 PHP 自带的mail() 函数，因为该函数在发送多个邮件时占用的系统资源很高。
**Swift 直接与 SMTP 服务器通讯，具有非常高的发送速度和效率。**
github:https://github.com/swiftmailer/swiftmailer
官网:http://swiftmailer.org/
examples:https://github.com/PHPMailer/PHPMailer/tree/master/examples
可参考:
http://www.sitepoint.com/sending-email-with-swift-mailer/
安装:composer require swiftmailer/swiftmailer
Swiftmailer的特点：
  * 邮件发送驱动有如下几种:SMTP, sendmail, postfix or a custom Transport implementation of your own
  * 支持用户认证
  * Protect from header injection attacks without stripping request data content
  * Send MIME compliant HTML/multipart emails
  * Use event-driven plugins to customize the library
  * Handle large attachments and inline/embedded images with low memory use

普通的Swift_MailTransport：
```

<?php
require_once 'lib/swift_required.php';

// Create the mail transport configuration
$transport = Swift_MailTransport::newInstance();

// Create the message
$message = Swift_Message::newInstance();
$message->setTo(array(
  "hello@gmail.com" => "Aurelio De Rosa",
  "test@fake.com" => "Audero"
));
$message->setSubject("This email is sent using Swift Mailer");
$message->setBody("You're our best client ever.");
$message->setFrom("account@bank.com", "Your bank");

// Send the email
$mailer = Swift_Mailer::newInstance($transport);
$mailer->send($message);

```
Swift_SmtpTransport，并带附件:
```

<?php
require_once 'lib/swift_required.php';

// Create the SMTP configuration
$transport = Swift_SmtpTransport::newInstance("smtp.fake.com", 25);
$transport->setUsername("Username");
$transport->setPassword("Password");

// Create the message
$message = Swift_Message::newInstance();
$message->setTo(array(
   "hello@gmail.com" => "Aurelio De Rosa",
   "test@fake.com" => "Audero"
));
$message->setCc(array("another@fake.com" => "Aurelio De Rosa"));
$message->setBcc(array("boss@bank.com" => "Bank Boss"));
$message->setSubject("This email is sent using Swift Mailer");
$message->setBody("You're our best client ever.");
$message->setFrom("account@bank.com", "Your bank");
$message->attach(Swift_Attachment::fromPath("path/to/file/file.zip"));

// Send the email
$mailer = Swift_Mailer::newInstance($transport);
$mailer->send($message, $failedRecipients);

// Show failed recipients
print_r($failedRecipients);

```

使用模板:
```

<?php
require_once 'lib/swift_required.php';

// Usually you want to replace the following static array
// with a dynamic one built retrieving users from the database
$users = array(
  array(
    "fullname" => "Aurelio De Rosa",
    "operations" => 100,
    "email" => "hello@gmail.com"
  ),
  array(
    "fullname" => "Audero",
    "operations" => 50,
    "email" => "test@fake.com"
  )
);

// Create the replacements array
$replacements = array();
foreach ($users as $user) {
  $replacements[$user["email"]] = array (
    "{fullname}" => $user["fullname"],
    "{transactions}" => $user["operations"]
  );
}

// Create the mail transport configuration
$transport = Swift_MailTransport::newInstance();

// Create an instance of the plugin and register it
$plugin = new Swift_Plugins_DecoratorPlugin($replacements);
$mailer = Swift_Mailer::newInstance($transport);
$mailer->registerPlugin($plugin);

// Create the message
$message = Swift_Message::newInstance();
$message->setSubject("This email is sent using Swift Mailer");
$message->setBody("You {fullname}, are our best client ever thanks " .
    " to the {transactions} transactions you made with us.");
$message->setFrom("account@bank.com", "Your bank");

// Send the email
foreach($users as $user) {
  $message->setTo($user["email"], $user["fullname"]);
  $mailer->send($message);
}

```

**使用SSL/TLS：**
http://swiftmailer.org/docs/sending.html
```

// Create the Transport
$transport = Swift_SmtpTransport::newInstance('smtp.example.org', 465, 'ssl');
$transport->setUsername("test@163.com");
$transport->setPassword("test");
// Create the Mailer using your created Transport
$mailer = Swift_Mailer::newInstance($transport);

/*
It's also possible to use multiple method calls

$transport = Swift_SmtpTransport::newInstance()
  ->setHost('smtp.example.org')
  ->setPort(587)
  ->setEncryption('ssl')
  ;
*/

```