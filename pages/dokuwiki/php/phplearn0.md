title: phplearn0 

#  PHP学习之php.ini与环境配置 
PHP核心配置说明:http://php.net/manual/zh/ini.core.php
http://www.laruence.com/2009/07/28/1030.html
找到php.ini-recommended，复制一份，然后将名称修改为：php.ini，然后打开该文件，进行配置。
错误配置php.ini:
;显示错误
display_startup_errors=On
display_errors=On
;报告所有错误
error_reporting=-1
;记录错误
log_errors=On

生产环境配置如下:
;不显示错误
display_startup_errors=Off
display_errors=Off
;除了注意事项之外，报告所有其他错误
error_reporting=E_ALL & ~E_NOTICE
;记录错误
log_errors=On

扩展目录
extension_dir = “ext” ;

; 动态扩展，可以根据需要去掉 extension 前面的注释 ; 
; 如加载 PDO, MySQL
extension=php_pdo_mysql.dll
extension=php_pdo_sqlite.dll
extension=php_mysql.dll
extension=php_mysqli.dll
extension=php_sqlite3.dll

; CGI 设置
;cgi.force_redirect = 1
cgi.fix_pathinfo = 1 (必须需要在PHP-CGI的配置文件（Ubuntu 上此配置文件位于/etc/php5/cgi/php.ini）中，打开cgi.fix_pathinfo选项：这样php-cgi方能正常使用SCRIPT_FILENAME这个变量。)
;cgi.rfc2616_headers = 1

;时区配置
date.timezone = Asia/Shanghai
;字符编码
default_charset="UTF-8"
;常用库配置
extension=php_curl.dll
extension=php_gd2.dll
extension=php_mbstring.dll

文件上传配置:
假设要上传一个90M的大文件。配置 php.ini 如下：
file_uploads = On
upload_tmp_dir = "d:/fileuploadtmp"
upload_max_filesize = 90M
post_max_size = 100M
max_file_uploads=10
max_execution_time = 3600 ;秒，一个小时
memory_limit=1048576000);Byte 1000 兆，即 1G 
提示：除此之外max_execution_time,memory_limit也有影响，需要保持 memory_limit > post_max_size > upload_max_filesize

真实路径缓存
realpath_cache_size=64k

会话处理与memcached:
session.save_handler='memcached'
session.save_path='127.0.0.1:11211'
##  命令行中执行PHP 
php my.php
php -r 'time();'