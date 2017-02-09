title: laravel框架24 

#  Laravel之邮件 
Laravel 基于热门的** SwiftMailer 函数库**提供了一个简洁的 API。Laravel 为 SMTP、Mailgun、Mandrill、Amazon SES、PHP 的 mail 函数及 sendmail 提供驱动，让你可以快速地从所选择的本地或云端服务开始发送邮件。
基于 API 的驱动，例如 Mailgun 或 Mandrill，通常比 SMTP 服务器更简单快速。所有的 API 驱动都需要在应用程序中安装 Guzzle HTTP 函数库。你可在 composer.json 文件中加入下面这一行，以便于在项目中安装 Guzzle：
```

"guzzlehttp/guzzle": "~5.3|~6.0"

```
Mailgun 驱动#
要使用 Mailgun 驱动，首先必须安装 Guzzle，之后将`  config/mail.php ` 配置文件中的** driver 选项**设置为 mailgun。接下来，确认 ` config/services.php ` 配置文件包含下列选项：
```

'mailgun' => [
    'domain' => 'your-mailgun-domain',
    'secret' => 'your-mailgun-key',
],

```
SES 驱动#
要使用 Amazon SES 驱动，必须安装 PHP 的 Amazon AWS SDK。你可在 composer.json 文件的 require 段落加入下面这一行以安装此函数库：
```

"aws/aws-sdk-php": "~3.0"

```
接下来，将 config/mail.php 配置文件中的 driver 选项设置为 ses。然后确认 config/services.php 配置文件包含下列选项：
```

'ses' => [
    'key' => 'your-ses-key',
    'secret' => 'your-ses-secret',
    'region' => 'ses-region',  // 例如 us-east-1
],

```


##  发送邮件# 
Laravel 允许你**在视图中存放电子邮件消息**。例如，要组织你的电子邮件，你可以在`  resource/views ` 目录中创建 emails 目录：
要发送消息，使用`  Mail facade 的 send 方法 `。send 方法接收三个参数。首先是包含邮件消息的视图名称。其次是一个要传递给该视图的数据数组。最后是一个用来接收消息实例的 闭包回调，让你可以自定义收件者、主题，以及邮件消息的其它部分：
```

<?php
namespace App\Http\Controllers;

use Mail;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 发封电子邮件提醒给用户。
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function sendEmailReminder(Request $request, $id)
    {
        $user = User::findOrFail($id);

        Mail::send('emails.reminder', ['user' => $user], function ($m) use ($user) {
            $m->from('hello@app.com', 'Your Application');

            $m->to($user->email, $user->name)->subject('Your Reminder!');
        });
    }
}

```
在上面的例子中，我们传了包含 user 键的数组，因此可以使用下列的 PHP 代码，在邮件视图中显示用户名：
```

<?php echo $user->name; ?>

```
**注意：$message 变量永远会被传递至电子邮件视图，且允许内部嵌入附件。因此，你应该避免将 message 变量传至视图。**

**创建消息#**
如之前讨论过的，传给 send 方法的第三个参数是一个闭包，让你可以对电子邮件消息自身指定不同的选项。使用此闭包，你可以指定消息的其它属性，例如副本、密件副本等等：
这是一份在 ` $message 消息生成器实例中 `可以使用的方法清单：
```

$message->from($address, $name = null);
$message->sender($address, $name = null);
$message->to($address, $name = null);
$message->cc($address, $name = null);
$message->bcc($address, $name = null);
$message->replyTo($address, $name = null);
$message->subject($subject);
$message->priority($level);
$message->attach($pathToFile, array $options = []);
// 以原始 $data 字符串附加一个文件...
$message->attachData($data, $name, array $options = []);
// 获取底层的 SwiftMailer 消息实例...
$message->getSwiftMessage();

```
注意：**传递至 Mail::send 闭包的消息实例继承了 SwiftMailer 消息类**，让你可以调用该类的任何方法来创建你的电子邮件消息。

**发送纯文本#**
传给 send 方法的视图，在默认情况下会假定它包含了 HTML。然而，通过传递数组作为 send 方法的第一个参数，除了 HTML 视图之外，你还可以同时指定发送纯文本视图：
Mail::send(['html.view', 'text.view'], $data, $callback);
或者，若你只需要发送纯文本电子邮件，则可以在数组中使用 text 键来指定：
Mail::send(['text' => 'view'], $data, $callback);
**发送原始字符串#**
若你希望直接发送原始字符串，则可以使用 raw 方法：
Mail::raw('Text to e-mail', function ($message) {
});

##  附件 
要在电子邮件中加入附件，可在传递给闭包的 $message 对象上使用 attach 方法。attach 方法接受文件的完整路径作为它的第一个参数：
```

Mail::send('emails.welcome', $data, function ($message) {
    //

    $message->attach($pathToFile);
});

```
附加文件至消息时，你也可以传递数组给 attach 方法作为第二个参数，**以指定显示名称和／或 MIME 类型**：
```

$message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);

```

##  在电子邮件视图中嵌入图片 
在电子邮件中嵌入内部图片通常是件很麻烦的事；然而 Laravel 提供了一个便利的方法，让你在电子邮件中附加图片并获取适当的** CID**。要嵌入内部图片，可在电子邮件视图的`  $message 变量中使用 embed 方法 `。` 请记得，Laravel 会自动让你所有的电子邮件视图都能获取到 $message 变量 `：
```

<body>
    这是一张图片：
    <img src="<?php echo $message->embed($pathToFile); ?>">
</body>

```
**在电子邮件视图中嵌入原始数据#**
若你已有想嵌入电子邮件消息中的原始数据字符串，则可以在 $message 变量上使用`  embedData 方法 `：
```

<body>
    这是一张从原始数据来的图片：

    <img src="<?php echo $message->embedData($data, $name); ?>">
</body>

```

##  队列邮件 
**将邮件消息加入队列#**
**由于发送电子邮件消息会大幅延长应用程序的响应时间，许多开发者都会选择将邮件消息加入队列并于后台发送。**Laravel 使用其内置的统一的队列 API 来让你轻松地完成此工作。要将邮件消息加入队列，使用 Mail facade 的 queue 方法：
```

Mail::queue('emails.welcome', $data, function ($message) {
    //
});

```
此方法会自动将工作加入队列，以便在后台发送邮件消息。当然，在使用此功能之前，你需要设置你的队列。

**延迟的消息队列#**
若你希望延迟发送已加入队列的电子邮件消息，则可以使用 later 方法。若要开始使用，只需将你想要延迟发送消息的秒数作为第一个参数发送给此方法即可：
```

Mail::later(5, 'emails.welcome', $data, function ($message) {
    //
});

```
**加入特定队列#**
若你想要指定特定的队列来加入消息 ，可使用 queueOn 及 laterOn 方法：
```

Mail::queueOn('queue-name', 'emails.welcome', $data, function ($message) {
    //
});

Mail::laterOn('queue-name', 5, 'emails.welcome', $data, function ($message) {
    //
});

```

##  事件# 
Laravel 会在发送邮件消息之前触发 ` mailer.sending 事件 `。切记，此事件只会在邮件发送时触发，在队列时则不会。你可以在你的`  EventServiceProvider ` 注册一个事件侦听器：
```

/**
 * 注册你应用程序中的其它事件。
 *
 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
 * @return void
 */
public function boot(DispatcherContract $events)
{
    parent::boot($events);

    $events->listen('mailer.sending', function ($message) {
        //
    });
}

```