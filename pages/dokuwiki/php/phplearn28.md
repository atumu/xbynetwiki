title: phplearn28 

#  PHP学习之HTTP库Requests 
http://requests.ryanmccue.info/
https://github.com/rmccue/Requests/
API：http://requests.ryanmccue.info/api/
Requests是一个PHP的HTTP类库。相对于cURL等类库来说，它具有简单易用且友好的API，且不依赖于cURL。它支持HEAD、 GET、 POST、 PUT、 DELETE和PATCH等方法，基本能满足任何形式的HTTP请求。
Requests不依赖于任何PHP标准库外的扩展，唯一的要求就是需要PHP5.2+的版本。但是如果PHP的cURL可用，Requests会优先使用它，否则会使用socket。
##  安装和使用 
通过Composer安装
```

{
    "require": {
        "rmccue/requests": ">=1.0"
    }
}

```
```

$headers = array('Accept' => 'application/json');
$options = array('auth' => array('user', 'pass'));
$request = Requests::get('https://api.github.com/gists', $headers, $options);

var_dump($request->status_code);
// int(200)

var_dump($request->headers['content-type']);
// string(31) "application/json; charset=utf-8"

var_dump($request->body);
// string(26891) "[...]"

```
###  自动加载 
可以使用Composer的加载器：
include('/path/to/composer/vendor/autoload.php');
也可以使用Requests自带的：
include('/path/to/library/Requests.php');
Requests::register_autoloader();

###  各种请求 
GET请求
$response = Requests::get('http://httpbin.org/get');
POST请求
$response = Requests::post('http://httpbin.org/post');
需要传数据的话，可以使用第三个参数：
$data = array('key1' => 'value1', 'key2' => 'value2');
$response = Requests::post('http://httpbin.org/post', array(), $data);
如果需要传原始数据的话，第三个参数请传字符串。

其他请求
其他请求方法都大同小异：
$response = Requests::put('http://httpbin.org/put');
$response = Requests::delete('http://httpbin.org/delete');
$response = Requests::patch('http://httpbin.org/patch', array('If-Match' => 'e0023aa4e'));
$response = Requests::head('http://httpbin.org/headers');
需要注意的是equests::patch()方法的第二个参数为必传。

###  Requests::request()方法 
看API文档，你会发现**这些方法接受的参数几乎一样：$url，$headers，$data（只有POST、PUT和PATCH有）**，$options。**事实上它们只是对Requests::request()方法进行了一次封装**：
```

/**
 * Send a GET request
 */
public static function get($url, $headers = array(), $options = array()) {
    return self::request($url, $headers, null, self::GET, $options);
}

/**
 * Send a HEAD request
 */
public static function head($url, $headers = array(), $options = array()) {
    return self::request($url, $headers, null, self::HEAD, $options);
}

/**
 * Send a DELETE request
 */
public static function delete($url, $headers = array(), $options = array()) {
    return self::request($url, $headers, null, self::DELETE, $options);
}

/**
 * Send a POST request
 */
public static function post($url, $headers = array(), $data = array(), $options = array()) {
    return self::request($url, $headers, $data, self::POST, $options);
}
/**
 * Send a PUT request
 */
public static function put($url, $headers = array(), $data = array(), $options = array()) {
    return self::request($url, $headers, $data, self::PUT, $options);
}

```
$header允许我们自定义请求头，例如POST方法的话，我们通常需要指定Content-Type，使服务器知道我们正在发送的的数据是什么格式：
```

$url = 'https://api.github.com/some/endpoint';
$headers = array('Content-Type' => 'application/json');
$data = array('some' => 'data');
$response = Requests::post($url, $headers, json_encode($data));

```
**$options允许我们对请求进行配置**，例如超时时间：
```

$options = array(
    'timeout' => 5
);
$response = Requests::get('https://httpbin.org/', array(), $options);

```
###  Requests_Response对象 
Requests里所有的请求方法（HEAD、 GET、 POST、 PUT、 DELETE和PATCH）返回的都是一个` Requests_Response对象 `，这个对象包含了**响应的各种信息**：
  * $body：响应体。
  * $raw：原始的HTTP响应数据。
  * $headers：响应头。
  * $status_code：状态码。
  * $success：标识请求是否成功。
  * $redirects：请求的重定向次数。
  * $url：请求的URL。
  * $history：请求的历史记录。
  * $cookies：cookie信息。

更多的选项配置请看：http://requests.ryanmccue.info/api/source-class-Requests.html#_request

###  Session处理 
当你需要对同一网站发出多个请求，那么` Requests_Session对象 `可以帮到轻易的设置一些默认参数：
```

$url = 'https://api.github.com/';
$header = array('X-ContactAuthor' => 'rmccue');
$data = array();
$options = array('useragent' => 'My-Awesome-App');
$session = new Requests_Session($url, $header, $data, $options);
$response = $session->get('/zen');

```
**Requests_Session的构造函数接受url、headers、data和options这4个参数，顺序跟Requests::request()方法一致**。同时你也可以通过访问属性的方式去修改options参数：
```

// 设置option属性
$session->useragent = 'My-Awesome-App';

```
更多Requests_Session的信息请看：http://requests.ryanmccue.info/api/source-class-Requests_Session.html

###  HTTPS请求 
Requests会默认帮忙处理HTTPS请求，就跟在浏览器访问HTTPS网站一样：
$response = Requests::get('https://httpbin.org/');
但是如果你想使用其他的证书或者自签证书，你可以指定证书文件（PEM格式）：
```

$options = array(
    'verify' => '/path/to/cacert.pem'
);

```
$response = Requests::get('https://httpbin.org/', array(), $options);
如果你想禁用HTTPS的验证，可以通过设置options：'verify' => false。
###  HTTP基本验证 
HTTP基本验证功能可以通过options的auth实现：
```

$options = array(
    'auth' => array('user', 'password')
);

```
Requests::get('http://httpbin.org/basic-auth/user/password', array(), $options);

##  进阶使用 
###  使用代理 
代理可以通过options的proxy实现：
```

$options = array(
    'proxy' => '127.0.0.1:3128'
);

```
Requests::get('http://httpbin.org/ip', array(), $options);
如果代理需要验证：
```

$options = array(
    'proxy' => array( '127.0.0.1:3128', 'my_username', 'my_password' )
);

```
Requests::get('http://httpbin.org/ip', array(), $options);
###  钩子 
通过Requests的钩子系统，我们可以通过**注册自己的钩子去扩展Requests的功能**：
```

$hooks = new Requests_Hooks();
$hooks->register('requests.after_request', 'mycallback');

$request = Requests::get('http://httpbin.org/get', array(), array('hooks' => $hooks));

```

Request提供的钩子请看：http://requests.ryanmccue.info/docs/hooks.html

###  自定义验证 
通过实现Requests_Auth接口，我们可以为请求添加自定义的验证。假设服务器会检查HTTP请求里的Hotdog请求头的值是不是为Yummy。我们先实现我们的验证类：
```

class MySoftware_Auth_Hotdog implements Requests_Auth {
    protected $password;

    public function __construct($password) {
        $this->password = $password;
    }

    public function register(Requests_Hooks &$hooks) {
        $hooks->register('requests.before_request', array(&$this, 'before_request'));
    }

    public function before_request(&$url, &$headers, &$data, &$type, &$options) {
        $headers['Hotdog'] = $this->password;
    }
}

```
可以看到，类实现了Requests_Auth接口，同时代码实现也用到了钩子。下面我们通过options的auth去调用我们的自定义验证：
```

$options = array(
    'auth' => new MySoftware_Auth_Hotdog('yummy')
);

```
$response = Requests::get('http://hotdogbin.org/admin', array(), $options);

###  Why Requests？ 
文章开始提到了Requests的一些优点，这个官网有个专门的页面进行详细的介绍，同时还提到了Requests跟其他类似类库的对比。通过这个对比，大家对Requests会有进一步的认识，同时也科普下还有哪些HTTP请求相关的类库。
请猛击：http://requests.ryanmccue.info/docs/why-requests.html


参考:https://segmentfault.com/a/1190000002867007