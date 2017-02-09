title: nginx_php文件上传大小限制 

#  nginx+php文件上传大小限制问题解决方法 
1、修改nginx配置：vim /etc/nginx/config.d/wiki.conf:
```

 location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;

	client_max_body_size 100m; #限制为100M
	#client_body_temp_path /opt/www/tmp 上传文件临时目录
    }

```
2、修改php.ini
vim /etc/php5/fpm/php.ini
在php.ini里面查看如下行：
```

upload_max_filesize=100M
post_max_size=100M
memory_limit=120M
max_execution_time=600
file_uploads=on
upload_tmp_dir=/tmp/www

```
在上传大文件时，你会有上传速度慢的感觉，当超过一定的时间，会报脚本执行超过30秒的错误，这是因为在php.ini配置文件中max_execution_time配置选项在作怪，其表示每个脚本最大允许执行时间(秒)，0 表示没有限制。你可以适当调整max_execution_time的值，不推荐设定为0。