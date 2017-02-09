title: phplearn32 

#  PHP学习之Zend OPcache与内置HTTP服务器 
##  Zend OPcache 
字节码缓存不是PHP的新特性，有很多独立的扩展可以实现缓存，例如Alternative PHP Cache(APC),eAccelerator,Xcache等。但是从PHP5.5.0开始，PHP内置了字节码缓存功能，名为Zend OPcache.
字节码缓存能够存储预先编译好的PHP字节码。
原理：把PHP编译生成的执行字节码OPcodes缓冲到内存中从而避免重复的编译过程，能够直接使用缓冲区已编译的代码从而提高速度，降低服务器负载，它们的效率是显而易见的，像drupal这种庞大的CMS，每次打开一个页面要调用数十个PHP文件，执行数万行代码，效率可想而知，在安装APC等加速器后打开页面的速度明显加快。

启用Zend OPcache:
默认情况下，Zend OPcache没有启用如果是自己编译PHP，执行./configure命令时必须包含以下选项: - -enable-opcache
之后在php.ini中指定Zend OPcache扩展的路径。如下:
```

zend_extension=/path/to/opcache.so

```
注意：Zend OPcache必须在Xdebug之前。
推荐配置:http://www.laruence.com/2013/11/11/2928.html
```

zend_extension=/path/to/opcache.so
opcache.enable_cli=1
opcache.memory_consumption=128      //共享内存大小, 这个根据你们的需求可调
opcache.interned_strings_buffer=8   //interned string的内存大小, 也可调
opcache.max_accelerated_files=4000  //最大缓存的文件数目
opcache.revalidate_freq=60          //60s检查一次文件更新
opcache.fast_shutdown=1             //打开快速关闭, 打开这个在PHP Request Shutdown的时候
                                    //   会收内存的速度会提高
opcache.save_comments=0             //不保存文件/函数的注释

#opcache.validate_timestamps=1 //默认为1，如果设为0，则表示不自动那个检查文件更新，那我们就需要手动清空缓存的字节码。从而opcache.revalidate_freq也不会有效。

```

##  内置的HTTP服务器 
从PHP5.4.0开始，PHP内置了简单的Web服务器。你可以进入项目的根目录。然后执行
```

php -S localhost:4000

```
如果不想当前目录作为根，那么可以通过-t选项指定
```

php -S localhost:4000 -t your_www_dir

```
-c选项可以指定配置文件:
```

php -S localhost:4000 -c /path/to/php.ini

```
指定路由脚本(前端控制器，用于转发所有HTTP请求，apache通过.htacces文件控制，Nginx通过try_files指令控制):
```

php -S localhost:8000 router.php

```
查看是否使用的是内置服务器
```

<?php
if(php_sapi_name=="cli-server"){
//内置服务器
}else{
}

```

