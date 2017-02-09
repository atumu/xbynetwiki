title: phplearn026 

#  PHP学习之密码哈希与日期处理 
##  密码哈希 
加密与哈希不是一回事。加密是双向算法，哈希是单向算法。
哈希算法有很多,如MD5,SHA1,SHA512,bcrypt等.有些算法很快，用于验证数据完整性，有些算法速度很慢，致在提高安全性。生成和存储密码时需要使用速度慢，安全性高的算法。
最安全的哈希算法是bcrypt.处理数据的次数叫做工作因子(work factor).
内置关键函数
**password_hash — 创建密码的哈希（hash）**
string password_hash ( string $password , integer $algo [, array $options ] )
当前支持的算法：
  * PASSWORD_DEFAULT - 使用 bcrypt 算法 (PHP 5.5.0 默认)。数据库里储存结果的列可超过60个字符（最好是255个字符）。
  * PASSWORD_BCRYPT - 使用 CRYPT_BLOWFISH 算法创建哈希。这会产生兼容使用 "$2y$" 的 crypt()。 
支持的选项：
  * salt - 手动提供哈希密码的盐值（salt）。**盐值（salt）选项从 PHP 7.0.0 开始被废弃（deprecated）了。 现在最好选择简单的使用默认产生的盐值。**
  * cost - 代表算法使用的工作因子。省略时，**默认值是 10**。 这个 cost 是个不错的底线，但也许可以根据自己硬件的情况，加大这个值。

**password_verify() - 验证密码是否和哈希匹配**
boolean password_verify ( string $password , string $hash )
注意 password_hash() 返回的哈希包含了算法、 cost 和盐值。 因此，所有需要的信息都包含内。使得验证函数不需要储存额外盐值等信息即可验证哈希。

password_needs_rehash — 确认用户的密码是否符合最新的算法选项。
boolean password_needs_rehash ( string $hash , integer $algo [, array $options ] )

下面用一个用户注册登录的例子来进行说明:
注册用户的脚本:
```

<?php
try {
    // Validate email
    $email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
    if (!$email) {
        throw new Exception('Invalid email');
    }

    // Validate password
    $password = filter_input(INPUT_POST, 'password');
    if (!$password || mb_strlen($password) < 8) {
        throw new Exception('Password must contain 8+ characters');
    }

    // Create password hash
    $passwordHash = password_hash(
       $password,
       PASSWORD_DEFAULT,
       ['cost' => 12]
    );
    if ($passwordHash === false) {
        throw new Exception('Password hash failed');
    }

    // Create user account (THIS IS PSUEDO-CODE)
    // $user = new User();
    // $user->email = $email;
    // $user->password_hash = $passwordHash;
    // $user->save();

    // Redirect to login page
    header('HTTP/1.1 302 Redirect');
    header('Location: /login.php');
} catch (Exception $e) {
    // Report error
    header('HTTP/1.1 400 Bad request');
    echo $e->getMessage();
}


```
登录用户的脚本:
```

<?php
session_start();
try {
    // Get email address from request body
    $email = filter_input(INPUT_POST, 'email');

    // Get password from request body
    $password = filter_input(INPUT_POST, 'password');

    // Find account with email address (THIS IS PSUEDO-CODE)
    $user = User::findByEmail($email);

    // Verify password with account password hash
    if (password_verify($password, $user->password_hash) === false) {
        throw new Exception('Invalid password');
    }

    // Re-hash password if necessary (see note below)
    $currentHashAlgorithm = PASSWORD_DEFAULT;
    $currentHashOptions = array('cost' => 15);
    $passwordNeedsRehash = password_needs_rehash(
        $user->password_hash,
        $currentHashAlgorithm,
        $currentHashOptions
    );
    if ($passwordNeedsRehash === true) {
        // Save new password hash (THIS IS PSUEDO-CODE)
        $user->password_hash = password_hash(
            $password,
            $currentHashAlgorithm,
            $currentHashOptions
        );
        $user->save();
    }

    // Save login status to session
    $_SESSION['user_logged_in'] = 'yes';
    $_SESSION['user_email'] = $email;

    // Redirect to profile page
    header('HTTP/1.1 302 Redirect');
    header('Location: /user-profile.php');
} catch (Exception $e) {
    header('HTTP/1.1 401 Unauthorized');
    echo $e->getMessage();
}


```

###  PHP5.5.0之前的密码哈希API 
使用组件ircmaxell/password-compat

##  日期、时间和时区 
PHP5.2引入DateTime,DateInterval、DateTimeZone、DatePeriod类
第三方日期组件:nesbot/carbon
设置默认时区:
php.ini设置data.timezone='Asia/ShangHai'
脚本中设置:date_default_timezone_set('Asia/ShangHai')

DateTime类:
```

// Constructor
$datetime1 = new DateTime();
// Constructor with argument
$datetime2 = new DateTime('2014-04-27 5:03 AM');
// Static constructor with format
$datetime3 = DateTime::createFromFormat('M j, Y H:i:s', 'Jan 2, 2014 23:04:12');

```
日期格式:http://php.net/manual/datetime.createfromformat.php

DateInterval类：
间隔以P开头的字符串。标志如下Y,M,D,W(星期),H,M,S
```

<?php
$datetime = new DateTime('2014-01-01 14:00:00');
// Create two weeks interval
$interval = new DateInterval('P2W');
// Modify DateTime instance
$datetime->add($interval);
echo $datetime->format('Y-m-d H:i:s');

```
```

<?php
date_default_timezone_set('America/New_York');

$dateStart = new \DateTime();
$dateInterval = \DateInterval::createFromDateString('-1 day');
$datePeriod = new \DatePeriod($dateStart, $dateInterval, 3);
foreach ($datePeriod as $date) {
    echo $date->format('Y-m-d'), PHP_EOL;
}


```

DateTimeZone类
```

<?php
$timezone = new DateTimeZone('America/New_York');
$datetime = new \DateTime('2014-08-20', $timezone);
$datetime->setTimezone(new DateTimeZone('Asia/Hong_Kong'));


```

DatePeriod类：
```

<?php
date_default_timezone_set('America/New_York');

$start = new DateTime();
$interval = new DateInterval('P2W');
$period = new DatePeriod($start, $interval, 3);

foreach ($period as $nextDateTime) {
    echo $nextDateTime->format('Y-m-d H:i:s'), PHP_EOL;
}


```
```

<?php
date_default_timezone_set('America/New_York');

$start = new DateTime();
$interval = new DateInterval('P2W');
$period = new DatePeriod(
    $start,
    $interval,
    3,
    DatePeriod::EXCLUDE_START_DATE //排除开始时间
);

foreach ($period as $nextDateTime) {
    echo $nextDateTime->format('Y-m-d H:i:s'), PHP_EOL;
}


```