title: nginx配置011 

#  nginx配置之自定义变量 
http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#set
http://blog.sina.com.cn/s/blog_6d579ff40100wi7p.html
语法
<blockquote>
Syntax:	set $variable value;
Default:	—
Context:	server, location, if
</blockquote>
变量值可以是普通字符串，正则表达式的分组某个值，nginx内置变量等。
例如:
```

 set $a "hello world";

```
```

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
}


```

```

 location /test {
            set $foo hello;
            echo "foo: $foo";
        }

```
这里我们使用第三方 [ngx_echo 模块](https://github.com/openresty/echo-nginx-module#readme)的 echo 配置指令将 $foo 变量的值作为当前请求的响应体输出。
