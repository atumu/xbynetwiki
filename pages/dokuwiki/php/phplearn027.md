title: phplearn027 

#  PHP学习之网络操作 
**基本的文件IO函数，如file(),fopen(),file_get_contents()等都支持http，https,ftp等路径获取内容。**
http://php.net/manual/zh/book.network.php
http://php.net/manual/zh/book.url.php
##  IP域名信息 
gethostbyaddr — 获取指定的IP地址对应的主机名
gethostbyname — 获取指定的主机名对应的IPv4地址
gethostname — Gets the host name

##  URL操作 
  * urlencode() - 编码 URL 字符串
  * urldecode — 解码已编码的 URL 字符串
超全局变量 $_GET 和 $_REQUEST 已经被解码了。对 $_GET 或 $_REQUEST 里的元素使用 urldecode() 将会导致不可预计和危险的结果。
  * base64_decode — 对使用 MIME base64 编码的数据进行解码
  * base64_encode — 使用 MIME base64 对数据进行编码
  * get_headers — 取得服务器**响应**一个 HTTP 请求所发送的所有标头
array get_headers ( string $url [, int $format = 0 ] )
format：如果**将可选的 format 参数设为 1**，则 get_headers() 会解析相应的信息并设定数组的键名。
```

<?php
$url = 'http://www.example.com';
print_r(get_headers($url));//键为1，2，3，4，。。。
print_r(get_headers($url, 1));//键为header头的键，
?>

```

  * get_meta_tags — 从一个文件中提取所有的 meta 标签 content 属性，返回一个数组
  * http_build_query — 生成 URL-encode 之后的请求字符串
string http_build_query ( mixed $query_data [, string $numeric_prefix [, string $arg_separator [, int $enc_type = PHP_QUERY_RFC1738 ]]] )
返回一个 URL 编码后的字符串。
参数query_data可以是数组或包含属性的对象。
一个 query_data 数组可以是简单的一维结构，也可以是由数组组成的数组（其依次可以包含其它数组）。
如果 query_data 是一个对象，只有 public 的属性会加入结果。
```

<?php
$data = array('foo'=>'bar',
              'baz'=>'boom',
              'cow'=>'milk',
              'php'=>'hypertext processor');

echo http_build_query($data) . "\n";
echo http_build_query($data, '', '&amp;');

?>
输出
foo=bar&baz=boom&cow=milk&php=hypertext+processor
foo=bar&amp;baz=boom&amp;cow=milk&amp;php=hypertext+processor

```

  * parse_url — 解析 URL，返回其组成部分
mixed parse_url ( string $url [, int $component = -1 ] )
本函数不是用来验证给定 URL 的合法性的，只是将其分解为下面列出的部分。不完整的 URL 也被接受，parse_url() 会尝试尽量正确地将其解析。
参数component
指定 PHP_URL_SCHEME、 PHP_URL_HOST、 PHP_URL_PORT、 PHP_URL_USER、 PHP_URL_PASS、 PHP_URL_PATH、 PHP_URL_QUERY 或 PHP_URL_FRAGMENT 的其中一个**来获取 URL 中指定的部分的 string。** （除了指定为 PHP_URL_PORT 后，将返回一个 integer 的值）。

对严重不合格的 URL，parse_url() 可能会返回 FALSE。
如果省略了 component 参数，将返回一个关联数组 array，在目前至少会有一个元素在该数组中。数组中可能的键有以下几种：
  * scheme - 如 http
  * host
  * port
  * user
  * pass
  * path
  * query - 在问号 ? 之后
  * fragment - 在散列符号 # 之后
如果指定了 component 参数， parse_url() 返回一个 string （或在指定为 PHP_URL_PORT 时返回一个 integer）而不是 array。如果 URL 中指定的组成部分不存在，将会返回 NULL。
```

<?php
$url = 'http://username:password@hostname/path?arg=value#anchor';

print_r(parse_url($url));

echo parse_url($url, PHP_URL_PATH);
?>

```