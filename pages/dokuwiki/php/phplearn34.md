title: phplearn34 

#  PHP学习之性能调优 
内存:
memory_limit 默认值128M

Zend OPcache字节码缓存:参考[这里](/pages/dokuwiki/php/phplearn32)

文件上传:
```

file_uploads = On
upload_tmp_dir = "d:/fileuploadtmp"
upload_max_filesize = 90M
max_file_uploads=10
post_max_size = 100M
max_execution_time = 3600 ;秒，一个小时
memory_limit=1048576000);Byte 1000 兆，即 1G

``` 

缓冲输出:
output_buffering=4096
implicit_flush=false
realpath_cache_size=64k
